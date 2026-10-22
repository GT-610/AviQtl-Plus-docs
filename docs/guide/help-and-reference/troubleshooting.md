---
title: Troubleshooting
description: Diagnose startup, media, preview, performance, plugin, and export problems.
---

::: warning

This page is not finished yet.

:::

# Troubleshooting

Start by saving a copy of the project and recording the application version,
operating system, graphics hardware, and exact steps that reproduce the issue.

## The application does not start

- Confirm that the archive was fully extracted.
- On Linux, verify the Vulkan driver and runtime, the FFmpeg shared libraries,
  libxkbcommon, and the X11 or Wayland client libraries.
- If startup reports that no GPU-backed adapter was found, update the graphics
  driver; a software renderer cannot run AviQtl.
- Update the graphics driver or Vulkan/Metal/Direct3D runtime.
- Start from a terminal and keep the diagnostic output.

## Media is missing

Open the missing-media manager, replace each item with a compatible source, and
save the project. Check timing after replacement because media properties may
differ.

## Preview is blank or incorrect

- Move the playhead inside the clip duration.
- Confirm that the clip and its layer are visible.
- Temporarily disable recent effects.
- Check the transform, opacity, and layer order.
- Test the source file in another media player.

## Editing becomes slow

- Close unnecessary applications and reduce competing GPU workloads.
- Test whether one media file, effect, or audio plugin triggers the slowdown.
- Reduce the number of simultaneously visible heavy effects while editing.
- Save, restart AviQtl-Plus, and reopen the project before collecting a report.

## Export fails

- Test a short frame range and a simple output path.
- Check free disk space and write permission.
- Confirm that project and export frame rates agree.
- Disable the most recently added effect or plugin to isolate the cause.

## Report a reproducible issue

Use the [AviQtl-Plus issue tracker](https://github.com/GT-610/AviQtl-Plus/issues).
Include the smallest project or steps that reproduce the problem, but remove
private media and personal paths before uploading files or logs.
