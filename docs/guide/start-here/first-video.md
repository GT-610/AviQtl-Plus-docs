---
title: Make your first video
description: Create, edit, save, preview, and export a small Freya NLE project.
---

# Make your first video

Time to make your first video with Freya NLE. Not sure what to make? Download [this sample video](https://static.fsf.org/nosvn/videos/escape-to-freedom/videos/escape-to-freedom-1080p.mp4) and follow along.

::: info
The sample video comes from https://www.fsf.org/blogs/community/help-others-find-free-software-watch-and-share-escape-to-freedom and is licensed under [CC-BY-SA 4.0](http://creativecommons.org/licenses/by-sa/4.0/).

The tutorial below uses the sample video, but it works the same way with any other footage.
:::

Launch Freya NLE.

## 1. Create a project

Set the width, height, framerate, and audio sample rate.

![Project settings for a 1280 by 720 project at 30 frames per second](/images/guide/first-video/zh-CN/project-launcher.png)

Then choose "Create a new project".

## 2. Add media

Once the project is created, you enter the main window.

![Main window](/images/guide/first-video/zh-CN/main-window.png)

The horizontal position controls time; the vertical position controls the layer.

::: info
The concept of a layer is similar to the "tracks" found in most video editing software, but you can place objects of different types on the same layer. As shown in the figure, a single layer can contain images, video, audio, and even transitions.

![Timeline with different kinds of objects on several layers](/images/guide/first-video/zh-CN/layer-note.png)
:::

::: warning
In Freya NLE, **higher-numbered layers render on top**: for example, layer 3 appears above layer 2. This matches AviUtl, but may be the opposite of some other video editors.
:::

Import an image, video, or audio file, or drag it into the timeline. You can place your first visual clip at frame `0` on a visible layer, or anywhere else you like.

![Dragging a media file into the timeline](/images/guide/first-video/zh-CN/drag.png)

![A media clip placed at the start of the timeline](/images/guide/first-video/zh-CN/import-media.png)

## 3. Add a text object

Right-click the empty area at the position you want on the timeline, choose "Add Object", and click "Text". A text object is placed there.

![Text object settings and the resulting title in the preview](/images/guide/first-video/zh-CN/text-object.png)

You can click and drag the object to move it, or drag its edges to adjust the start and end frames.

Double-click the text object to open its settings, where you can change its motion parameters in detail. We will cover this in the next section.

Scroll down to the "Text" section — these properties are unique to text objects — and change the text content, font, size, and so on. Pick a font you like and replace the content with your own text.

![Rendered text in the preview](/images/guide/first-video/zh-CN/text-preview.png)

::: info
You might wonder why video objects are blue and audio objects are red. In Freya NLE, blue indicates *visual objects* — things you see, which all share the basic "standard drawing" logic, such as text, images, video, and shapes. Red indicates *audio objects* — things you hear, such as the audio attached to a video or a standalone audio file. Purple indicates *transition objects*, which are covered later.
:::

## 4. Animate the text

First, let's understand how motion works in Freya NLE.

![The "standard drawing" panel](/images/guide/first-video/zh-CN/standard-draw.png)

Below every parameter there is a keyframe axis. The object has a keyframe at its start and end; the highlighted blue area in between is the *active region*. The thin blue line on the axis marks the current playhead position. The left half of the control area above shows the value of the keyframe on the left side of the active region, and the right half shows the value of the keyframe on the right side. Click the parameter name in the middle to configure the interpolation type.

![The interpolation window](/images/guide/first-video/zh-CN/interpolation-type.png)

In other words, you only need to define where the value starts within the active region (start keyframe), where it ends (end keyframe), and how it moves (interpolation type). Freya NLE fills in everything in between.

Let's add horizontal constant-velocity motion to the text. Double-click the axis below the X parameter at the position where you want the motion to start, creating a keyframe there. Then double-click again at the position where you want it to end, creating a second keyframe.

::: info
Right-clicking an existing keyframe deletes it, along with the motion it defines.
:::

The axis is now divided into three segments. Click the middle segment to select it as the active region.

![The X parameter state](/images/guide/first-video/zh-CN/x-status.png)

Then drag the sliders or edit the value fields to set the start and end values of the active region. The example here uses `-500` and `500`.

Click the keyframes at either end of the active region to jump to that time and inspect the frame. When the playhead sits inside the active region, the preview shows **the interpolated position at the current time**, not the values you set at the two ends.

![Setting the X start and end values](/images/guide/first-video/zh-CN/x-value-set.png)

::: info
In Freya NLE, the default coordinate origin `x=0, y=0, z=0` is the **center of the canvas**. Think of the preview as a **camera looking straight down** along the Z axis: X is positive to the right, Y is positive upward, and positive Z brings the object closer, making it appear larger.

This is a bit like Blender, isn't it? One of Freya NLE's defining features is that its world is three-dimensional. You may not yet understand why a video editor needs a Z axis, but you will once you reach the "Camera control" chapter!
:::

Click the parameter name `X`, choose "Linear" in the interpolation settings, then close the window.

![The result after setting linear motion](/images/guide/first-video/zh-CN/linear-set.png)

Now click the keyframe on the left side of the active region, then press the play button in the preview window (or press Space) and enjoy the first text animation you made!

![The finished horizontal motion](/images/guide/first-video/zh-CN/final-effect.png)

Likewise, you can add keyframes to the X, Y, Z, scale, rotation, and other parameters of any text or other visual object to control motion along different axes.

## 5. Add an effect

You now know how to control an object's motion. Next, let's learn how to add effects.

First add a shape object, and in its "Shape" section make its width and height a bit larger.

![Preparing the shape object](/images/guide/first-video/zh-CN/graphic-init.png)

Click "Add Effect" in the top-left corner of the shape settings, then choose "Basic Effects" → "Vibration".

![Adding the "Vibration" effect](/images/guide/first-video/zh-CN/add-vibe-effect.png)

Play it — isn't the shape vibrating? Just like any other property, you can adjust the vibration's strength in different directions, its speed, and even the random seed. The same applies to all other effects.

## 6. Preview and save

The main preview shows every effect. Choose "File" → "Save" (or press `Ctrl` + `S`) to save the whole project. Freya NLE also supports autosave, so you can recover your progress after an unexpected crash — but we still recommend saving manually on a regular basis, just in case.

## 7. Export

In the main preview, choose "File" → "Export Media", set the output path, a valid frame range, and the codec, then start the export.

![The "Export Media" window](/images/guide/first-video/zh-CN/export-media.png)

You can watch the live export progress in the main preview window. Once it's done, enjoy your finished video!

![The finished exported video](/images/guide/first-video/zh-CN/final-video.png)

You have now learned the core Freya NLE workflow.
