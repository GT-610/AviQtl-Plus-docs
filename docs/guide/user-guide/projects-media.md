---
title: Projects and media
description: Set up projects, organize source media, save safely, and relink missing files.
---

::: warning

This page is not finished yet.

:::

# Projects and media

## Project settings

Project width, height, frame rate, and audio sample rate define the editing and
export context. Choose them before detailed editing whenever possible. Changing
settings later can alter framing, timing, or output expectations.

## Project files and source media

An `.aviqtl` project stores editing decisions and references source media by
path. It does not automatically embed every source file. Keep media in a stable
project folder and avoid renaming or moving it during an edit.

```text
my-video/
├── project.aviqtl
├── video/
├── audio/
├── images/
└── exports/
```

## Fragments

A fragment (`.aviqtlfrag`) is a block of clips saved on its own, so it can be used
again in another scene or project. **File → Export Fragment…** writes the selected
clips, or the whole scene when nothing is selected; **File → Insert Fragment…** adds
them back, and dropping the file on the timeline places it at the pointer.

A fragment keeps everything the project keeps, including effects and parameters that
have no AviUtl equivalent. Use it to move work between AviQtl-Plus projects. Use
**File → Export AviUtl Object File (Compatibility)…** only when the other end is
AviUtl: that format cannot hold everything, and the export reports what it had to skip.

## Save habits

- Save as soon as a project is created.
- Save before a large edit, package change, or export.
- Keep an additional copy before moving the project to another computer.
- Do not treat an exported video as a replacement for the project and sources.

## Missing media

A project can still open when referenced files are missing. AviQtl-Plus reports
the missing items and provides a management dialog where each item can be
replaced with a compatible file.

<!-- Add screenshot: ![Missing media notification and replacement controls](/images/en/guide/projects-media/missing-media.webp) -->

Relink missing files, verify the timeline, and save the updated project.

After relinking, seek through every affected clip. A replacement file may have
a different duration, resolution, frame rate, or audio layout even when its file
type is compatible.
