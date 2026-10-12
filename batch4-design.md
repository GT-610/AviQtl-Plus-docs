# Batch 4 design: object midpoints, media bin, multi-object properties, trim tools

This document is the independent design required by the fourth delivery batch of the
[UI research](../audits/2026-09-24-slint-ui-aviutl-research.zh-CN.md). It fixes the semantics,
persistence, undo, and compatibility rules for four editing features before any of them are
wired to a window. Each section states the rule the code must obey, not an aspiration.

Scope note: this batch is the *editing model* layer only. It is deliberately framework-neutral
and lives in `aviqtl-core` and `aviqtl-app`, so none of it requires a GPU-backed adapter. The
preview transform handles, the docked workspace, and every pixel-level affordance stay in the
deferred native GUI suite.

## 1. Object midpoints

### 1.1 What a midpoint is

AviUtl ExEdit's midpoint is an object-internal time boundary added with `P`. It splits the object
into two independently parameterised halves: before the midpoint the object reads one set of
parameter values, after it another. ExEdit presents this by adding a keyframe at that frame to
*every* animatable parameter at once, so the whole object changes its parameterisation at one
instant.

The existing `KeyframeTrack` operates on a single `effect`/`param` pair. Midpoints are object
level: they must affect every animatable parameter of the clip together, while leaving the
parameter tracks and the easing catalogue untouched.

### 1.2 Storage

Midpoints are stored on the clip, not on each track, so that adding one is a single clip-level
write and so that reading them does not require walking every effect.

```json
{
  "id": 7,
  "type": "rect",
  "midpoints": [30, 90]
}
```

`midpoints` is a strictly increasing array of clip-relative frames with
`0 < frame < duration`. It is stored through `ClipDocument::extra`, so the key survives a
round trip through `parse_project_document` without a schema bump: unknown keys are preserved by
the `#[serde(flatten)]` catch-all on every document type.

### 1.3 Rules

1. **Explicit opt-in.** The key is absent until the first midpoint command. A clip without
   `midpoints` is a clip without midpoints; loading must not synthesise an empty array.
2. **Frame 0 and the duration endpoint are fixed.** A midpoint must satisfy
   `0 < frame < clip.duration`. Adding, moving, or inserting never produces a midpoint on a
   fixed frame.
3. **No duplicates.** Adding a midpoint at a frame that already has one is a no-op and reports
   no change, so an Undo is never pushed for a rejected request.
4. **Moving one midpoint never crosses another.** Frames stay strictly increasing. A move that
   would land on or cross a neighbour is rejected; it is not silently reordered.
5. **Removing the last midpoint removes the key.** A clip returns to its pre-midpoint
   representation so a project that never used midpoints stays byte-comparable with its Qt form.
6. **Clip resize, move, split, and merge preserve midpoints.** All are frame-domain operations:
   - resize from the start shifts every midpoint by the same delta, clamped into the open
     interval;
   - a duration change clamps and drops midpoints that fall outside the new interval;
   - split at frame `f` gives the first half every midpoint below `f`; the second half keeps
     the rest rebased onto its own origin. A midpoint exactly at the cut becomes frame 0 of the
     second half, which is a fixed frame, so it is dropped rather than stored;
   - merged or moved clips keep midpoints unchanged, because they are clip-relative.
7. **Undo is one step per command.** Adding several midpoints is one grouped transaction, as
   with the existing `begin_undo_group` / `end_undo_group` convention.
8. **Midpoints are clip metadata, not animation data.** They do not create keyframe tracks, and
   the evaluator is unchanged: a midpoint is a *marker* for the UI and for ExEdit-compatible
   import/export until the object-level parameterisation itself is designed. This is stated
   explicitly because the research warns against renaming "one keyframe per parameter" a
   complete AviUtl midpoint compatibility.

Rule 8 is the honest boundary. Storing the boundaries, keeping them correct across every
existing geometry and split command, and giving them tested undo semantics is the part that can
be finished without a GPU and without a parameterisation redesign. The visual "edit both halves
at once" behaviour belongs to a later step that needs the desktop suite to verify.

### 1.4 Compatibility

- Documents written before this feature contain no `midpoints` key and load unchanged.
- Documents written after it are rejected by a Qt build that predates it, exactly as any other
  unknown key would be. The application is unreleased and has no external extension
  compatibility requirement, so this is accepted rather than emulated.
- Cloning a clip (`Duplicate`) copies the midpoints; undoing the duplicate removes them.

## 2. Media bin

### 2.1 What it is

A project-scoped list of source media, independent of whether a clip currently references it.
Each entry records the probed kind and duration, and carries user-set in/out frames used as the
default range for insert and overwrite. Bins are a flat, ordered grouping; nesting is
deliberately out of scope.

### 2.2 Storage

The project document gains one optional top-level key.

```json
{
  "version": 3,
  "mediaBin": {
    "entries": [
      {"id": 1, "path": "clips/a.mp4", "kind": "video",
       "durationFrames": 240, "inFrame": 30, "outFrame": 210, "name": "a"}
    ],
    "bins": [{"id": 1, "name": "Footage", "entryIds": [1]}]
  }
}
```

`mediaBin` is absent in every pre-existing document and is added to
`ProjectDocument::extra`, which is again a `#[serde(flatten)]` map. No version bump is required:
version 3 is still readable and writable, and the key round-trips.

### 2.3 Rules

1. **Probing is the only source of `kind` and `durationFrames`.** They come from
   `plan_media_import`, which already owns the Qt extension rules and the FFmpeg duration probe.
   The bin never guesses.
2. **In/out are always a valid, non-degenerate range.** `0 <= inFrame < outFrame <= durationFrames`.
   Setting in or out clamps into that range rather than failing.
3. **Entries are deduplicated by resolved path**, so dropping the same file twice updates one
   entry instead of creating a second row. Comparison is on the project-relative path when the
   file is inside the project directory, and on the absolute path otherwise.
4. **The bin never owns the timeline.** Removing a bin entry does not remove clips that use it.
   Clips keep their own resolved media reference; the bin is a source list, not a dependency
   graph. This is why removing an entry cannot silently destroy an edit.
5. **A missing file is not removed.** An entry whose file has disappeared is still listed and is
   reported through the existing missing-media path, so the user can relink it.
6. **Bins never nest** and a bin cannot contain itself. Reordering is a permutation, matching
   `plan_effect_reorder`.

### 2.4 Compatibility

- No `mediaBin` key: the bin is empty and every command is a no-op that reports no change.
- A `mediaBin` key with unknown fields keeps them, because the bin is stored as
  `serde_json::Value` inside the flattened extras until a dedicated type is justified.
- The document is not written with an empty `mediaBin`; the key appears on first add, matching
  the midpoint rule that absence means "never used".

## 3. Multi-object property editing

### 3.1 Semantics

When several clips are selected, one control may address the same parameter on all of them. The
difficulty is that the objects may disagree, and the UI must be able to say so.

The model exposes three states for a property across a selection:

- **Uniform** — every object has the same value. Editing sets that value on every object.
- **Mixed** — the objects disagree. Editing sets the value on every object. The UI shows the
  mixed state before the edit and the chosen value after; it never displays one object's value
  as if it applied to all.
- **Absent** — some object does not have the parameter at all (a clip without that effect, or a
  parameter the metadata does not define). An edit applies only to the objects that have it, and
  the absent objects are skipped rather than gaining a parameter they never had.

### 3.2 Rules

1. **Locked layers are skipped, not an error.** A locked clip keeps its value and is excluded
   from the edit, matching the existing move/resize behaviour where a locked clip rejects
   initiation. A selection consisting only of locked clips changes nothing.
2. **Effects are matched by catalog id, not by index.** Two clips usually hold the effect at
   different sidebar positions; matching by index would edit the wrong effect.
3. **A keyframed parameter is edited at the playhead**, on the objects whose parameter has a
   track, and as a plain parameter on the objects whose parameter has none. This mirrors the
   existing single-object `set_effect_param_for_clip` rule.
4. **One edit is one undo step.** The whole selection is applied through a single batched
   transaction; the compacting of adjacent clip replacements already collapses it to one entry.
5. **Selection order does not change the result.** The applied set is the selection intersected
   with editable objects, in document order, so a Ctrl-click in a different order produces an
   identical edit.

### 3.3 Compatibility

Multi-object editing is a presentation-layer capability over existing per-object commands. It
introduces no new persisted key, so a project edited through it is byte-identical to the same
edit performed one object at a time.

## 4. Trim tools

### 4.1 What the tools do

The four standard trims are pure geometry, so they are planned in `timeline_edit` alongside the
existing move, resize, and paste planners and applied through `BatchUpdateClipGeometry`. No new
`EditRequest` variant is required.

| Tool | Effect | Adjacent clip |
| --- | --- | --- |
| **Trim** (existing resize) | Moves the dragged edge only | unaffected |
| **Ripple** | Trim, then close the gap by shifting later clips | shifts |
| **Roll** | Moves the shared cut; both neighbours' other edges stay put | both |
| **Slip** | Keeps the clip's timeline span, changes its source in-point | unaffected |
| **Slide** | Keeps both source and duration, moves the clip in time | shifts |

### 4.2 Rules

1. **Every tool is a pure planner plus the shared geometry commit.** `preview_*` and the commit
   call the same function, which is the preview/commit symmetry the drag path already uses.
2. **Durations stay positive and frames stay non-negative.** Where a gesture has a natural limit
   (the anchor's own minimum duration, the row's first frame, the end of the source), the planner
   clamps to it, so dragging past the limit is forgiving rather than an error. Where the limit
   belongs to *another* clip (a roll whose neighbour would be consumed, a ripple whose tail would
   collide), the planner rejects, because silently deleting a neighbour's frames is not something
   a drag should decide.
3. **Locked layers reject initiation**, as move and resize already do.
4. **Ripple only shifts clips that are strictly after the edited edge** on the affected layer.
   The tail is validated in order against a running boundary, so a shift that would run a clip
   into its predecessor is rejected rather than silently overlapping. A ripple is the user's
   explicit request for a specific new span, so asking for an impossible one is a rejection, not
   an invitation to hunt for a nearby free frame the way the move planner does.
5. **Ripple is opt-in per command, never a timeline mode.** The research is explicit that a
   magnetic or ripple default would break AviUtl's layered-subtitle muscle memory, so nothing
   changes until the user picks the tool.
6. **Slip is bounded by the source media**, when the source duration is known, and is a no-op
   with no change reported when it is not. Guessing a source range would silently corrupt a
   clip.
7. **Every tool is one undo step.**
8. **A slide needs both neighbours and leaves each of them at least one frame.** A zero-length
   clip cannot be placed on the timeline at all, so a slide that would consume a neighbour
   entirely clamps to the range that leaves one frame behind. A row with no room at all is a
   no-op, not an error, so a wedged row does not push an undo entry for a gesture that did
   nothing.

### 4.3 Compatibility

No new persisted keys. The tools are alternative planners over the same clip geometry the
project already stores.

## 5. Why this is the GPU-independent part of the batch

Preview transform handles and the integrated workspace are excluded on purpose. Handle hit
testing and coordinate mapping need a real window to verify, and free docking changes window
ownership, GPU preview hosting, focus, and shortcut scope — all of which the deferred native GUI
suite must accept. The four areas above are pure model rules and are complete when their
planners, persistence, undo behaviour, and behaviour tests exist.
