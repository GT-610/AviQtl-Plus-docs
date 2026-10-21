---
title: AviUtl object files (.exo)
description: How AviQtl-Plus reads AviUtl ExEdit object files, and exactly what it translates.
---

# AviUtl object files (`.exo`)

AviUtl ExEdit can save a selection of objects as an **object file** with the `.exo`
extension: a small text document that carries object positions, layer numbers, the
filter stack of each object, and every parameter value. AviQtl-Plus **reads** that format,
so existing AviUtl material can be brought into a project.

This is the **compatibility** path, and it runs one way. ExEdit's object model is narrower
than this project's, so an import translates what it can prove and turns anything else into a
disabled placeholder rather than guessing. Writing an object file back out would have to drop
whatever AviQtl-Plus can express and ExEdit cannot, so AviQtl-Plus does not write one: use a
[fragment](./fragment-files) to move work between projects.

- **File → Import AviUtl Object File (Compatibility)…** appends the file's objects after the current
  scene's last object, as a single undoable edit.
- **Dropping an `.exo` file on the timeline** places the file at the pointer, anchored at its
  earliest object, the way AviUtl does.
- **File → New Project from AviUtl Object File…** starts a project whose resolution, frame
  rate, audio rate, and length come from the file, then imports its objects into the first
  scene, which is what AviUtl's "new project from object file" does.

These commands work on the object file only. They do not execute ExEdit scripts, and they
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
CP932 (the code page usually called Shift-JIS), and hand-edited files are often UTF-8. The importer detects the encoding per file,
so a file from either pack reads correctly without asking the user which one it is.

## What is translated

| ExEdit object or filter | AviQtl-Plus result |
| --- | --- |
| 文本 / テキスト | Text object: text, font, size, colour, outline colour, bold, italic, letter and line spacing, alignment |
| 图像 / 画像, 视频 / 動画, 音频 / 音声 | Image, video, or audio object, with the referenced path taken from the object's binary data |
| 图形 / 図形 | Shape object: the built-in figure type becomes a regular polygon. The stored value counts from zero in the order the dialog lists them — 背景 (rectangle), 円 (a dense polygon), 四角形, 三角形, 五角形, 六角形, 星型 — and `大小` / サイズ is the drawn half-extent, `长宽比` / 縦横比 the aspect, `线宽` / ライン幅 the stroke |
| 标准属性 / 標準描画 (drawing) | Transform effect: position, scale, opacity, Z rotation, blend mode |
| 模糊, 边缘模糊, 马赛克, 单色化, 裁剪遮罩, 斜向裁剪, 色调修正, 淡化, 锐化, 振动, 镜像, 极坐标变换, 光栅, 方向模糊, 放射模糊, 镜头模糊, 发光, 辉光, 扩散光, 提取边缘, 噪波, 描边, 阴影, 色度键, 亮度键, 渐变, 图像拼贴 | The matching built-in effect, with parameters converted to our units |

An animated trackbar becomes a two-point keyframe track: the start value at frame 0 and
the end value at the object's last frame.

### Filter names are localized

AviUtl resolves a filter by its localized `_name`, so the CN pack writes `标准属性` where
stock ExEdit writes `標準描画`. The importer accepts every spelling it knows and treats them
as the same filter.

## Adding a translation

Mapping is data, not code. An object or filter is translated only when both its name and its
parameter semantics are known, so a new entry needs evidence for each.

1. Find the filter in ExEdit's own table. It lists the localized display name, every trackbar
   name with its default, every check name, and the flags word; `exo_mapping.rs` mirrors the
   parts the importer needs.
2. Match the parameter names against the catalogue effect's own parameters, and use the
   stored defaults as a check on the scale: a percentage stored as `1000` in ExEdit and `100`
   in our catalogue means a scale of `0.1`. A default that disagrees is not automatically a
   problem - two effects can use different default blur radii - but it means the scale has to
   be justified some other way, usually by the same quantity appearing at the same scale
   elsewhere in the table. When nothing justifies it, the filter stays a placeholder.
3. Add a `FilterMapping` (or an `ObjectMapping` / figure type) with the Chinese, Japanese, and
   English spellings. The CN pack, stock ExEdit, and some tools each spell names differently.
4. Import a file that contains the filter, with `AVIUTL_SAMPLES` pointing at an AviUtl
   package: `cargo test -p aviqtl-app --test exo_samples -- --ignored`. The test imports every
   shipped file and checks that each object becomes a clip with the placement the file gave it
   and that nothing preserved was dropped, so a filter that stops translating shows up here
   rather than in someone's project.

Objects created in AviQtl-Plus use the first alias when no source spelling exists, so put the
name the pack writes first.

## What is preserved but not rendered

Everything else — ExEdit script objects (自定义物件, 动画效果, カスタムオブジェクト), shape
objects, cameras, group controls, audio filters, and any effect without a confirmed
equivalent — is imported as a **disabled placeholder**: the object keeps its name, its
script identifier, its script parameter block, its track and check values, and its binary
data. It appears in the effect stack, and does nothing to the picture.

A placeholder is a record rather than a working effect: its job is to keep what the file said
visible and intact inside the project, so the same material can be carried on later — by saving
it as a fragment, or by whatever runs ExEdit scripts in the future.

A placeholder is inspectable: selecting it shows read-only rows for everything the importer
kept, including the script identifier and each preserved key. The rows are marked unsupported
rather than editable, because the effect does not run yet.

When an import leaves placeholders behind, AviQtl-Plus shows a report naming each object or
filter it preserved. For a script-backed effect the note names the script, because that is the
file the user would have to install for the object to work. A clean import stays silent, so the dialog only appears when the picture
may differ from AviUtl.

The importer never guesses. A filter whose parameter meanings are unconfirmed would
otherwise silently change the picture, and a wrong render is much harder to notice than a
disabled entry in the stack.

Each object also carries routing flags: `group` binds it to a group control, `clipping` clips
it to the object above, `audio` marks it as an audio object, and `chain` and `camera` change
what it applies to. AviQtl-Plus has no field for them, so they are stored on the clip in its
extra fields rather than being dropped.

ExEdit also appends bookkeeping filters of its own — 上层物件缓存 (an upper-object buffer) and
the frame-buffer capture. They carry no artist intent and AviUtl never shows them as editable
entries. They are skipped: adding them to the stack would show a filter AviUtl never
displayed, and they carry nothing a renderer needs.

## Limitations worth knowing

- **Scripts are not executed.** ExEdit scripts are Lua programs with their own API; running
  them is a separate feature, so their output cannot be reproduced yet.
- **Motion identity is approximate.** A trackbar's motion is stored as a label inside an
  easing script, not as a curve identifier. Labels that clearly mean linear, accelerate,
  decelerate, or both are converted; anything else becomes linear and keeps its original
  text on the keyframe.
- **Bitmap figures are not converted.** A 図形 can also reference a monochrome PNG in the
  `figure` folder; those arrive as placeholders, only the built-in types are translated.
- **Text alignment covers the horizontal choice and drops the vertical one.** ExEdit numbers
  `align` as an index into its 文字揃え list: 0/1/2 are left/centre/right on the top row, 3/4/5
  the same three centred vertically, and so on, with 9 and above meaning 縦書き. The horizontal
  choice is translated; the vertical one and the vertical-writing modes have no counterpart
  here, so those keep the default and the original value stays on the object.
- **Media blobs are read for the path only.** ExEdit stores the referenced file inside a
  per-object binary blob whose layout differs between object types. The importer scans that
  blob for a NUL-terminated path with a media extension and keeps the blob alongside it; the
  path is what the clip actually uses. A file that cannot be found is reported by
  **File → Manage Missing Media**, not silently dropped.
- **An import grows the scene.** If the file reaches past the end of the scene, the scene is
  extended as part of the same undoable edit, so the tail of the import is not hidden behind
  the end of the timeline. A longer scene is never shortened by an import.
- **An import moves rather than overlaps.** When the file would land on frames that
  already hold objects, the whole block moves forward and the report says by how much.
  ExEdit refuses to load onto occupied positions, so laying an import on top would leave
  two objects sharing a layer and a frame.

## Related

- [Fragment files](./fragment-files) — the native format for saving a block of clips.
- [AviUtl operability targets](./operability-targets) — the editing-model goals this
  feature serves.
- [Effects and objects](./effects) — the catalogue an imported object is translated into.
