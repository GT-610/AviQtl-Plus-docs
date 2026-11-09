---
title: Exporting
description: Turn an Freya NLE timeline into a video or image sequence.
---

::: warning

This page is not finished yet.

:::

# Exporting

Export creates a deliverable from the current project. Save the project and
preview the intended frame range before opening **Export Media**.

## Before exporting

- Confirm project width, height, frame rate, and audio sample rate.
- Check that no media is missing.
- Play or seek through the complete export range.
- Choose an output folder with enough free space.
- Avoid overwriting the only copy of an earlier export.

<!-- Add screenshot: ![Freya NLE export dialog with output path and frame range](/images/en/guide/exporting/export-dialog.webp) -->

The exact encoder choices depend on the platform and available multimedia components.

## Frame range

The start frame must not be after the end frame. Export a short range first when
testing a new effect, encoder, or project configuration; this is faster than
discovering a problem after a full render.

## Validate the result

Open the exported file in a separate media player and verify:

- image dimensions and orientation;
- duration and frame rate;
- animated text, effects, and transitions;
- audio presence, synchronization, and level;
- the first and last intended frames.

If export fails, record the exact error message and see
[Troubleshooting](../help-and-reference/troubleshooting).
