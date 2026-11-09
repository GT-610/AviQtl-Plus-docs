---
title: Effects and animation
description: Modify clip parameters, order effects, and animate values with keyframes.
---

::: warning

This page is not finished yet.

:::

# Effects and animation

## Parameters and effects

Every selected clip exposes parameters that describe its content and transform.
Additional effects can be added from the shared catalog. An effect remains part
of the clip and is evaluated according to its position in the effect stack.

Use the enable control to compare the result with and without an effect. Reorder
effects when the output of one should become the input of another.

## Keyframes

A keyframe stores a parameter value at a particular time inside a clip. With two
or more keyframes, Freya NLE calculates intermediate values according to the
selected interpolation.

1. Select a clip and parameter.
2. Move the playhead to the first time.
3. Set the value and create a keyframe.
4. Move the playhead to another time.
5. Change the value and create or update the second keyframe.
6. Preview the interval between them.

<!-- Add screenshot: ![Two keyframes for an animated transform parameter](/images/en/guide/effects-animation/keyframes.webp) -->

Keyframes belong to a clip parameter and are evaluated relative to the clip.

## Good animation practice

- Start with position, scale, rotation, or opacity before combining many effects.
- Preview at normal playback speed as well as frame by frame.
- Keep keyframes inside the clip duration.
- Prefer a small number of intentional keyframes over many corrective ones.
- Check effect order when an animation looks different from what you expected.
