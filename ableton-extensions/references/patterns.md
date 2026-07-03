# Idiomatic Patterns

Recipes distilled from the SDK's own examples. These are the conventions to follow.

## Command + context menu

Register a command and wire a context-menu item to it by shared string id. Namespace ids (`"myext.action"`).

```ts
context.commands.registerCommand("myext.process", () => {
  console.log("triggered");
});
context.ui.registerContextMenuAction("ClipSlot", "Process this slot", "myext.process");
```

Register one command on **multiple scopes** by looping an `as const` array:

```ts
context.commands.registerCommand("myext.rename", (arg: unknown) => { /* ... */ });

(["MidiClip", "AudioClip"] as const).forEach((scope) => {
  context.ui.registerContextMenuAction(scope, "Rename this clip", "myext.rename");
});
```

## Handle resolution + instanceof narrowing

Command args arrive as `unknown`. For object scopes, cast to `Handle`, resolve, then narrow.

```ts
context.commands.registerCommand("myext.warp", (arg: unknown) => {
  const clip = context.getObjectFromHandle(arg as Handle, Clip);  // Clip covers audio + midi
  if (!(clip instanceof AudioClip)) {
    console.error("Not an audio clip.");
    return;
  }
  clip.warpMode = clip.warpMode === WarpMode.Beats ? WarpMode.Complex : WarpMode.Beats;
});
```

Resolve as the base `DataModelObject` when the type is unknown, then branch:

```ts
const obj = context.getObjectFromHandle(handle, DataModelObject);
if (obj instanceof ClipSlot) { /* ... */ }
else if (obj instanceof AudioTrack) { /* ... */ }
```

## Async inside a synchronous command callback

`registerCommand`'s callback returns `void`. Two ways to run async work:

Declare the callback `async`:
```ts
context.commands.registerCommand("myext.a", async (arg: unknown) => {
  try {
    const slot = context.getObjectFromHandle(arg as Handle, ClipSlot);
    const path = await context.resources.importIntoProject("/abs/path.wav");
    await slot.createAudioClip({ filePath: path, isWarped: true,
      loopSettings: { looping: true, startMarker: 0, endMarker: 4, loopStart: 0, loopEnd: 4 } });
  } catch (e) { console.error(e); }
});
```

Or the typed-IIFE pattern (keeps the arg cast local, guarantees a `.catch`):
```ts
context.commands.registerCommand("myext.b", (arg: unknown) =>
  void (async (handle: Handle) => {
    const clip = context.getObjectFromHandle(handle, Clip);
    // ...
  })(arg as Handle).catch((e) => console.error(e)),
);
```

## Selections: map → filter → batch

Selection-scope args are `ArrangementSelection` / `ClipSlotSelection`. Resolve every handle, filter with typed predicates, batch async work with `Promise.all`, set sync props after.

```ts
context.ui.registerContextMenuAction(
  "MidiTrack.ArrangementSelection", "Fill selection", "myext.fill");

context.commands.registerCommand("myext.fill", async (arg: unknown) => {
  const selection = arg as ArrangementSelection;

  const objects = selection.selected_lanes.map((h) =>
    context.getObjectFromHandle(h, DataModelObject));

  const midiLanes = objects.filter(
    (o): o is MidiTrack<"1.0.0"> | TakeLane<"1.0.0"> =>
      o instanceof MidiTrack ||
      (o instanceof TakeLane && o.parent instanceof MidiTrack));

  const length = selection.time_selection_end - selection.time_selection_start;
  const clips = await Promise.all(
    midiLanes.map((lane) => lane.createMidiClip(selection.time_selection_start, length)));
  clips.forEach((clip, i) => { clip.name = `Clip ${i + 1}`; });
});
```

Note the typed predicate form `(o): o is MidiTrack<"1.0.0"> => ...` — this is where you write the version generic explicitly so TypeScript narrows the array element type.

## Transactions (single undo step)

Every mutation is already undoable on its own. Use `withinTransaction` only to **group several into one undo entry**. The callback is **synchronous** — you cannot `await` inside it. Collect promises synchronously and await the whole thing:

```ts
const promises = context.withinTransaction(() =>
  ranges.map((r) => track.clearClipsInRange(r.start, r.end)));
await Promise.all(promises);
```

Or return `Promise.all` directly and await the transaction call:

```ts
const [a, b] = await context.withinTransaction(() =>
  Promise.all([song.createAudioTrack(), song.createAudioTrack()]));
```

Rules: nested transactions collapse into the outermost. You **cannot create an object and then modify it in the same transaction** (you need the resolved instance first) — do create in one, modify after. Changes apply atomically (no intermediate states shown).

## Progress dialog + cancellation

For long work. `update(text, percent)` where percent is 0–100 (`undefined` = indeterminate). Check the `AbortSignal` between steps.

```ts
context.commands.registerCommand("myext.long", () => {
  void context.ui.withinProgressDialog("Working…", {}, async (update, signal) => {
    for (let i = 0; i < items.length; i++) {
      signal.throwIfAborted();                    // bail if the user cancelled
      await update(`Item ${i + 1}/${items.length}`, (i / items.length) * 100);
      await doWork(items[i]!);                     // ! because noUncheckedIndexedAccess
    }
    await update("Cleaning up", undefined);        // indeterminate
  });
});
```

`signal.throwIfAborted()` throws on cancel (wrap in try/catch if you need cleanup); alternatively poll `if (signal.aborted) return;`.

## Audio: import → create clip

Files outside the allowed dirs must be imported first; always use the returned path.

```ts
const imported = await context.resources.importIntoProject("/abs/sample.wav");
const slot = context.getObjectFromHandle(arg as Handle, ClipSlot);
await slot.createAudioClip({
  filePath: imported,
  isWarped: true,
  loopSettings: { looping: true, startMarker: 1, endMarker: 5, loopStart: 1, loopEnd: 5 },
});
```

Arrangement version needs `startTime` (and optional `duration`):

```ts
const track = context.getObjectFromHandle(selection.selected_lanes[0]!, AudioTrack);
await track.createAudioClip({
  filePath: imported,
  startTime: selection.time_selection_start,
  duration: selection.time_selection_end - selection.time_selection_start,
  isWarped: true,
});
```

## Render → analyze → mutate (the "strip silence" shape)

The reference workflow for audio analysis: render pre-FX audio to a WAV, decode it with an npm lib, compute something, then mutate inside one transaction. Node built-ins and npm packages work (esbuild bundles them).

```ts
import * as fs from "fs/promises";
import decodeAudio from "audio-decode";

// inside a progress dialog callback (update, signal):
const wavPath = await context.resources.renderPreFxAudio(track, startBeat, endBeat);
if (signal.aborted) return;
const decoded = await decodeAudio(await fs.readFile(wavPath));
const channels = Array.from({ length: decoded.numberOfChannels },
  (_, i) => decoded.getChannelData(i));
const ranges = computeSilenceRanges(channels, decoded.sampleRate /* ... */);

// beats ↔ seconds
const beatsPerSecond = 60 / context.application.song.tempo;

const promises = context.withinTransaction(() =>
  ranges.map((r) =>
    track.clearClipsInRange(startBeat + r.start / beatsPerSecond,
                            startBeat + r.end / beatsPerSecond)));
await Promise.all(promises);
```

## MIDI notes

`MidiClip.notes` is a whole-array get/set. To edit, read, transform, assign back:

```ts
const clip = context.getObjectFromHandle(handle, MidiClip);
clip.notes = clip.notes.map((n) => ({ ...n, pitch: n.pitch + 12 }));  // octave up
```

Create notes with the required fields (`pitch`, `startTime`, `duration` in beats):

```ts
const midi = await slot.createMidiClip(4);
midi.notes = [
  { pitch: 60, startTime: 0, duration: 1, velocity: 100 },
  { pitch: 64, startTime: 1, duration: 1 },
];
```

## Device parameters (async value access)

```ts
const device = track.devices[0];
if (device) {
  const p = device.parameters.find((x) => x.name === "Dry/Wet");
  if (p) {
    const current = await p.getValue();
    await p.setValue(Math.min(p.max, current + 0.1));
  }
}
```
