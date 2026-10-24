---
title: Timeline editing
description: Select, move, trim, split, layer, copy, and arrange clips precisely.
---

::: warning

This page is not finished yet.

:::

# Timeline editing

The timeline is where you decide **when** an item appears and **which layer** it
occupies. AviQtl-Plus combines AviUtl-style layered objects with common modern
editing commands.

## Selection and edit targets

Selection answers “what will be edited.” Pointer actions use the frame and layer
under the pointer. Keyboard and menu commands generally use the playhead and the
selected layer, while parameter animation uses the selected object and current
playhead frame.

## Core operations

- **Move:** drag a clip horizontally to change time or vertically to change its
  layer.
- **Trim:** change a clip edge to adjust its visible duration.
- **Split:** place the playhead inside a clip and use **Split Clip**.
- **Delete:** remove the selected clips.
- **Copy, cut, paste, duplicate:** reuse clips and their settings.
- **Nudge:** move selected clips by one frame for precise alignment.

<!-- Add screenshot: ![Several clips arranged across multiple AviQtl-Plus timeline layers](/images/en/guide/timeline/core-operations.webp) -->

Horizontal placement controls timing; higher layers can composite over lower layers.

## Trim tools

The tool picker at the right of the scene strip changes what a drag on a clip
edge or body means. Each tool is one-shot: after the gesture the pointer returns
to **Selection Tool**, so the next click cannot repeat a trim by accident.

- **Ripple Trim:** drag either edge. Everything after the clip on the same layer
  moves with the new edge, so no gap opens and no later clip is covered.
- **Roll Edit:** drag the cut between two touching clips. One grows while the
  other shrinks; the pair keeps its outer edges exactly where they were.
- **Slide Clip:** drag the clip body. The clip keeps its duration and source
  range while its two neighbours absorb the move, so the row stays closed.
- **Slip Media:** drag the clip body. The clip keeps its timeline span; only the
  source in-point moves, bounded by the media's real length. A source-in badge
  follows the pointer, and a video or audio object with no known source is
  refused instead of guessing.

Every gesture previews with the same planner that commits it, and one gesture is
one undo step. A locked layer refuses the edit. A trim follows the pointer in
whole frames instead of snapping, because these tools move a boundary the user is
already placing by hand.

## Object midpoints

Midpoints mark frames inside one object where its motion value changes. Right
click a clip and choose **Add Midpoint Here**, or use the clip menu at the
clicked frame. Midpoints appear as small handles on the clip:

- drag a handle to move it; neighbouring handles and the clip end bound it, and a
  move that would cross a neighbour is rejected;
- double-click a handle to remove it;
- handles follow the clip when it is moved, trimmed, split, or split apart, and a
  trim below a midpoint drops that midpoint.

## Layers

Layers determine stacking and organization. A visual item on an upper layer can
appear over an item below it. Layers can also be locked to prevent accidental
changes or hidden to simplify previewing.

## Playhead and skimming

The playhead is the authoritative frame for preview, splitting, and keyframe
work. Some pointer interactions may expose a temporary hover or skimmer target;
use the playhead when an operation must occur at an exact frame.

## Undo and redo

Timeline changes are undoable. When performing a complicated edit, make one
logical change at a time and check the result before continuing. Saving remains
important: undo history is not a backup strategy.
