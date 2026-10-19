---
title: AviUtl object files (.exo)
description: How AviQtl-Plus reads and writes AviUtl ExEdit object files, and exactly what is translated.
---

# AviUtl object files (`.exo`)

AviUtl ExEdit can save a selection of objects as an **object file** with the `.exo`
extension: a small text document that carries object positions, layer numbers, the
filter stack of each object, and every parameter value. AviQtl-Plus reads and writes
that format, so existing AviUtl material can be brought into a project and a finished
scene can be handed back to AviUtl.

- **File → Import AviUtl Object File…** appends the file's objects after the current
  scene's last object, as a single undoable edit.
- **Dropping an `.exo` file on the timeline** places the file at the pointer, anchored at its
  earliest object, the way AviUtl does.
- **File → New Project from AviUtl Object File…** starts a project whose resolution, frame
  rate, audio rate, and length come from the file, then imports its objects into the first
  scene, which is what AviUtl's "new project from object file" does.
- **File → Export AviUtl Object File…** writes the selected objects, or the whole scene
  when nothing is selected. ExEdit's export command works on the selection, so this matches
  it; the status line says which of the two happened.

Both commands work on the object file only. They do not execute ExEdit scripts, and they
do not make AviQtl-Plus load AviUtl plugins.

## What the format contains

```text
[exedit]
width=1280
height=720
rate=30
length=120
...
[0]              ; one object: 1-based start, end, and layer
start=11
end=40
layer=3
[0.0]            ; the object itself: text, shape, image, video, audio, ...
_name=文本
text=604f7d59
[0.1]            ; an effect in stack order
_name=模糊
范围=8
[0.2]            ; the drawing/transform filter, always last
_name=标准属性
X=100.0
...
```

Keys are not quoted and a value may itself contain `=`, so a line splits on its first
`=`. Frames and layers are 1-based on disk, which is why an object at `start=11` begins at
frame 10 in AviQtl-Plus. A trackbar that animates is stored as
`start,end,motion,parameter`, for example `X=100.0,300.0,15@easing_普通@uf_easing,16`.

## Text encoding

AviUtl writes strings in the operating system's ANSI code page, not in Unicode. The
Chinese translation packs therefore produce GBK, a stock Japanese install produces
Shift-JIS, and hand-edited files are often UTF-8. The importer detects the encoding per
file and remembers it, so a file that is imported and exported again keeps its bytes
readable by the same AviUtl build.

## What is translated

| ExEdit object or filter | AviQtl-Plus result |
| --- | --- |
| 文本 / テキスト | Text object: text, font, size, colour, outline colour, bold, italic, letter and line spacing, alignment |
| 图像 / 画像, 视频 / 動画, 音频 / 音声 | Image, video, or audio object, with the referenced path taken from the object's binary data |
| 图形 / 図形 | Shape object: the built-in figure types become regular polygons (filled rectangle, triangle, pentagon, hexagon, ten-vertex star; the circle becomes a dense polygon), `大小` the drawn half-extent, `长宽比` the aspect, `线宽` the stroke |
| 标准属性 / 標準描画 (drawing) | Transform effect: position, scale, opacity, Z rotation, blend mode |
| 模糊, 边缘模糊, 马赛克, 单色化, 裁剪遮罩, 斜向裁剪, 色调修正, 淡化, 锐化, 振动, 镜像, 极坐标变换, 光栅, 方向模糊, 放射模糊, 镜头模糊, 发光, 辉光, 扩散光, 提取边缘, 噪波, 描边, 阴影, 色度键, 亮度键, 渐变, 图像拼贴 | The matching built-in effect, with parameters converted to our units |

An animated trackbar becomes a two-point keyframe track: the start value at frame 0 and
the end value at the object's last frame.

### Filter names are localized

AviUtl resolves a filter by its localized `_name`, so the CN pack writes `标准属性` where
stock ExEdit writes `標準描画`. The importer accepts both spellings and records the one the
file used, and the exporter writes that string back, so a file from a Japanese install is
still readable by the same build after a round trip.

## What is preserved but not rendered

Everything else — ExEdit script objects (自定义物件, 动画效果, カスタムオブジェクト), shape
objects, cameras, group controls, audio filters, and any effect without a confirmed
equivalent — is imported as a **disabled placeholder**: the object keeps its name, its
script identifier, its script parameter block, its track and check values, and its binary
data. It appears in the effect stack, does nothing to the picture, and is written back
unchanged when the scene is exported.

A placeholder is inspectable: selecting it shows read-only rows for everything the importer
kept, including the script identifier and each preserved key. The rows are marked unsupported
rather than editable, because the effect does not run yet.

When an import leaves placeholders behind, AviQtl-Plus shows a report naming each object or
filter it preserved. For a script-backed effect the note names the script, because that is the
file the user would have to install for the object to work. A clean import stays silent, so the dialog only appears when the picture
may differ from AviUtl. Exporting reports the objects it had to skip the same way.

The importer never guesses. A filter whose parameter meanings are unconfirmed would
otherwise silently change the picture, and a wrong render is much harder to notice than a
disabled entry in the stack.

Each object also carries routing flags: `group` binds it to a group control, `clipping` clips
it to the object above, `audio` marks it as an audio object, and `chain` and `camera` change
what it applies to. AviQtl-Plus has no field for them, so they are stored on the clip in its
extra fields and written back unchanged.

ExEdit also appends bookkeeping filters of its own — 上层物件缓存 (an upper-object buffer) and
the frame-buffer capture. They carry no artist intent and AviUtl never shows them as editable
entries, so they are kept on the clip rather than in the effect stack and written back to the
file in their original position.

## Limitations worth knowing

- **Scripts are not executed.** ExEdit scripts are Lua programs with their own API; running
  them is a separate feature, so their output cannot be reproduced yet.
- **Motion identity is approximate.** A trackbar's motion is stored as a label inside an
  easing script, not as a curve identifier. Labels that clearly mean linear, accelerate,
  decelerate, or both are converted; anything else becomes linear and keeps its original
  text on the keyframe.
- **Bitmap figures are not converted.** A 図形 can also reference a monochrome PNG in the
  `figure` folder; those arrive as placeholders, only the built-in types are translated.
- **Text alignment is preserved, not translated.** ExEdit stores `align` as a packed setting
  whose encoding is not established, so an imported value is written back verbatim and our
  own alignment number is left at its default.
- **Media paths are recovered heuristically.** ExEdit stores the referenced file inside a
  per-object binary blob whose layout differs between object types. The importer scans that
  blob for a NUL-terminated path with a media extension. A file that cannot be found is
  reported by **File → Manage Missing Media**, not silently dropped.
- **Export re-serializes values.** A value is written from the project's current state, so
  an exported file carries canonical number forms (`100.0` where AviUtl wrote `100.00`) and a
  text buffer without its fixed-size NUL padding. AviUtl reads both identically.
- **Some object types cannot be exported.** A scene reference needs a target scene number
  and a camera control needs a target layer; the exporter does not write those, so such
  clips are reported as skipped instead of being written as objects that point nowhere.
- **An import grows the scene.** If the file reaches past the end of the scene, the scene is
  extended as part of the same undoable edit, so the tail of the import is not hidden behind
  the end of the timeline. A longer scene is never shortened by an import.
- **Layer and frame offsets are fixed at zero.** Imported objects keep their relative
  layout, which matches AviUtl's own "append to the end of the current data" behaviour.

## Related

- [AviUtl operability targets](./operability-targets) — the editing-model goals this
  feature serves.
- [Effects and objects](./effects) — the catalogue an imported object is translated into.
