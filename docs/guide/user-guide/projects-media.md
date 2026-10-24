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

## Media library

The timeline window has a docked **Media Library** that lists files the project
knows about, whether or not a clip currently uses them. **View → Media Library**
opens it; the **Media** button in the scene strip toggles it.

- **Add Media…** probes the files once and stores their kind and source length in
  the project. Adding the same file twice updates one row instead of creating a
  duplicate.
- Each row keeps an **In** and **Out** point. Those points are the default range
  for every insert; an image has no seekable source, so its range only describes
  the default clip length.
- **Insert at Playhead** places the selected range at the current playhead and
  selected layer, searching forward for a free place when something is already
  there. **Overwrite** places it at the playhead exactly and replaces whatever
  occupies that range on the target layer (and the linked audio layer for a
  video). Both are one undo step.
- **Group** is a flat, project-scoped label for finding media. An entry belongs to
  at most one group. A group can be created, renamed, removed, or cleared from an
  entry; removing or clearing a group never removes media or touches the timeline.
- The search field matches both the file name and the stored path, and the group
  picker narrows the list further.

The library never owns the timeline: removing an entry leaves every clip alone,
and a clip keeps working when its entry is gone.

## Fragments

A fragment (`.aviqtlfrag`) is a block of clips saved on its own, so it can be used
again in another scene or project. **File → Export Fragment…** writes the selected
clips, or the whole scene when nothing is selected; **File → Insert Fragment…** adds
them back, and dropping the file on the timeline places it at the pointer.

A fragment keeps everything the project keeps, including effects and parameters that
have no AviUtl equivalent, so it is the way work moves between AviQtl-Plus projects.

AviUtl object files (`.exo`) can be imported, but not written: **File → Import AviUtl
Object File (Compatibility)…** brings AviUtl material in, and the report says what it
translated and what it had to keep as a disabled placeholder.

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
