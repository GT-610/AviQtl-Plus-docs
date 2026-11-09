---
title: Interface overview
description: Understand the preview, timeline, settings, menus, and transport controls.
---

::: warning

This page is not finished yet.

:::

# Interface overview

Freya NLE uses separate editing surfaces rather than a single fixed panel
layout. The main preview, timeline, and object settings can be shown from the
application menus and arranged to fit your screen.

<!-- Add screenshot: ![Freya NLE preview, timeline, and object settings windows](/images/en/guide/interface/workspace-overview.webp) -->

A typical workspace keeps the preview visible while the timeline and selected-object settings remain accessible.

## Preview

The preview shows the current frame after compositing visible layers, generated
objects, transforms, and effects. Use it to judge visual results, not to infer
the exact timing of a clip; timing is controlled on the timeline.

## Timeline

Time runs from left to right and layers are stacked vertically. The playhead
marks the frame displayed by the preview. Clips own their start frame, duration,
layer, parameters, effects, and animation data.

## Object settings

Selecting a clip exposes its editable parameters and effect stack. Parameter
controls may include numeric fields, sliders, colors, files, enumerations, and
keyframe controls. Numeric entry is useful when a value must be exact.

## Menus and commands

The menus provide project creation, open/save, media export, project settings,
undo/redo, playback, clip operations, scene operations, and commands that show
the timeline or settings windows.

::: tip Focus matters
Keyboard commands such as Space may be captured by a text field while you are
typing. Click the timeline or preview before using playback shortcuts.
:::
