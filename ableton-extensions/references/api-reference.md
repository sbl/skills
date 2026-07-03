# API Reference

Ground truth is the SDK's `index.d.cts` (v1.0.0-beta.0). All signatures below are transcribed from it. Classes are generic over the API version (`Track<"1.0.0">` etc.); the version param is a phantom type you rarely write by hand except in type predicates.

Import everything from the package root:

```ts
import {
  initialize, type ActivationContext, type Handle,
  Song, Track, AudioTrack, MidiTrack, Clip, AudioClip, MidiClip, ClipSlot, TakeLane,
  Scene, CuePoint, Device, RackDevice, DrumRack, Chain, DrumChain, Simpler, Sample,
  DeviceParameter, TrackMixer, ChainMixer, DataModelObject,
  WarpMode, GridQuantization,
  type NoteDescription, type WarpMarker, type ClipLoopSettings,
  type ArrangementSelection, type ClipSlotSelection, type ContextMenuScope,
} from "@ableton-extensions/sdk";
```

## ExtensionContext (returned by `initialize`)

```ts
interface ExtensionContext<Version> {
  application: Application<Version>;
  commands: Commands<Version>;
  environment: Environment<Version>;
  resources: Resources<Version>;
  ui: Ui<Version>;
  getObjectFromHandle<T extends DataModelObject>(handle: Handle, type: abstract new (...args) => T): T;
  withinTransaction<T>(fn: () => T): T;
}
```

- `getObjectFromHandle` — resolves a handle to a typed object. Pass the narrowest class you can, or `DataModelObject` then narrow with `instanceof`. **Throws** if the object was deleted, is a different type than `type`, or its type is unrecognized. Results are cached by handle id (same Live object → same instance).
- `withinTransaction` — groups mutations into one undo step. Callback **must be synchronous**; return `Promise.all([...])` to batch async ops. See patterns reference.

## Handles

```ts
interface Handle { id: bigint; }
```

Opaque host-assigned reference. Never construct one. Only handles received from the host (as command args) are valid. Resolve via `getObjectFromHandle`. Handles invalidate on delete / move / session change — don't cache them long-term (see project-setup).

## Data model

Base class — everything extends it:

```ts
class DataModelObject<Version> {
  readonly handle: Handle;
  get parent(): DataModelObject<Version> | null;   // canonical parent in Live's hierarchy
  static readonly className: string;               // used for host-side type checks
}
```

Inheritance:

```
DataModelObject
├── Application         └ song
├── Song
├── Track ── AudioTrack, MidiTrack
├── ClipSlot
├── TakeLane
├── Clip ── AudioClip, MidiClip
├── Scene
├── CuePoint
├── Device ── RackDevice ── DrumRack;  Simpler
├── DeviceParameter
├── Sample
├── TrackMixer, ChainMixer
└── Chain ── DrumChain
```

### Application
```ts
get song(): Song;
```

### Song  (the current Live Set)
Synchronous properties:
```ts
get tracks(): Track[];          // regular tracks only (no returns/main)
get returnTracks(): Track[];
get mainTrack(): Track;
get scenes(): Scene[];
get cuePoints(): CuePoint[];
get tempo(): number; set tempo(v);
get gridQuantization(): GridQuantization;   // combine with gridIsTriplet
get gridIsTriplet(): boolean;
get rootNote(): number;         // 0=C .. 11=B
get scaleName(): string;
get scaleMode(): boolean;       // Scale Mode on/off
get scaleIntervals(): number[]; // semitone offsets from root
```
Async methods (all return Promises):
```ts
createAudioTrack(): Promise<AudioTrack>;        // inserted after last selected track, else appended
createMidiTrack(): Promise<MidiTrack>;
createScene(index: number): Promise<Scene>;     // 0..scenes.length; -1 appends
deleteTrack(track): Promise<void>;
deleteScene(scene): Promise<void>;
duplicateTrack(track): Promise<Track>;          // duplicate inserted after original
duplicateScene(scene): Promise<Scene>;
createCuePoint(time: number): Promise<CuePoint>;   // time in beats
deleteCuePoint(cuePoint): Promise<void>;
```

### Track  (base; AudioTrack / MidiTrack inherit)
```ts
get name(): string; set name(v);
get mute(): boolean; set mute(v);
get solo(): boolean; set solo(v);
get mutedViaSolo(): boolean;    // read-only
get arm(): boolean; set arm(v);
get clipSlots(): ClipSlot[];
get takeLanes(): TakeLane[];
get arrangementClips(): Clip[];
get groupTrack(): Track | null;
get devices(): Device[];
get mixer(): TrackMixer;
createTakeLane(): Promise<TakeLane>;                       // appended
insertDevice(deviceName: string, index: number): Promise<Device>;  // built-in Live devices ONLY (e.g. "Reverb", "Auto Filter"); no 3rd-party plugins
deleteDevice(device): Promise<void>;
duplicateDevice(device): Promise<Device>;                 // inserted after original
deleteClip(clip): Promise<void>;                          // arrangement clips; for session clips use ClipSlot.deleteClip
clearClipsInRange(startTime: number, endTime: number): Promise<void>;  // beats; clips overlapping a boundary are truncated
```
- **AudioTrack** adds `createAudioClip(args)` — see the arg contract below.
- **MidiTrack** adds `createMidiClip(startTime: number, duration: number): Promise<MidiClip>` (beats).

### ClipSlot  (a Session View grid cell)
```ts
get clip(): Clip | null;
deleteClip(): Promise<void>;
createMidiClip(length: number): Promise<MidiClip>;        // length in beats
createAudioClip(args: { filePath: string; isWarped?: boolean; loopSettings?: ClipLoopSettings }): Promise<AudioClip>;
```

### TakeLane
```ts
get clips(): Clip[];
get name(): string; set name(v);
createMidiClip(startTime: number, duration: number): Promise<MidiClip>;
createAudioClip(args: { filePath: string; startTime: number; duration?: number; isWarped?: boolean; loopSettings?: ClipLoopSettings }): Promise<AudioClip>;
```

### Clip  (base; AudioClip / MidiClip inherit)
```ts
get name(): string; set name(v);
get startTime(): number;  get endTime(): number;  get duration(): number;   // read-only
get startMarker(): number;  get endMarker(): number;                        // read-only
get looping(): boolean; set looping(v);   // enabling loop on unwarped audio auto-enables warp
get loopStart(): number;  get loopEnd(): number;                            // read-only
get color(): number; set color(v);
get muted(): boolean; set muted(v);
```
- **AudioClip** adds:
```ts
get filePath(): string;
get warping(): boolean; set warping(v);
get warpMode(): WarpMode; set warpMode(v);
get warpMarkers(): WarpMarker[];   // read-only
```
- **MidiClip** adds:
```ts
get notes(): NoteDescription[]; set notes(v);   // read + replace the whole note set
```

### Scene
```ts
get name(): string; set name(v);
get tempo(): number;                  // read-only
get signatureNumerator(): number;     // read-only
get signatureDenominator(): number;   // read-only
```

### CuePoint
```ts
get time(): number;   // beats, read-only
get name(): string; set name(v);
```

### Device / RackDevice / DrumRack / Simpler
```ts
class Device {
  get name(): string;
  get parameters(): DeviceParameter[];
}
class RackDevice extends Device {
  get chains(): Chain[];
  insertChain(index: number): Promise<Chain>;   // 0..chains.length
}
class DrumRack extends RackDevice {  // className "DrumRackDevice"
  get chains(): DrumChain[];         // overrides to drum chains
}
class Simpler extends Device {
  get sample(): Sample | null;
  replaceSample(filePath: string): Promise<Sample>;
}
```

### Chain / DrumChain
```ts
class Chain {
  get devices(): Device[];
  get mixer(): ChainMixer;
  insertDevice(deviceName: string, index: number): Promise<Device>;
  deleteDevice(device): Promise<void>;
  duplicateDevice(device): Promise<Device>;
}
class DrumChain extends Chain {
  get receivingNote(): number; set receivingNote(v);
}
```

### DeviceParameter  (value read/write is async, unlike other getters)
```ts
get name(): string;
get min(): number;  get max(): number;
get isQuantized(): boolean;
get defaultValue(): number;
get valueItems(): DeviceParameterValueItem[];   // { name, shortName }[]
getValue(): Promise<number>;
setValue(value: number): Promise<void>;
```

### Mixers
```ts
class TrackMixer {  // className "MixerDevice"
  get volume(): DeviceParameter;
  get panning(): DeviceParameter;
  get sends(): DeviceParameter[];
}
class ChainMixer { /* same three members; className "ChainMixerDevice" */ }
```

### Sample
```ts
get filePath(): string;
```

## Services

### Commands  (`context.commands`)
```ts
registerCommand(commandId: string, callback: (...args: unknown[]) => void): void;
executeCommand(commandId: string, ...args: unknown[]): void;
```
Callback return type is **synchronous** (`void`) — wrap async work in an IIFE (see patterns).

### Ui  (`context.ui`)  — all Promise-based
```ts
registerContextMenuAction(scope: ContextMenuScope, title: string, commandId: string): Promise<() => Promise<void>>;  // resolves to an async unregister fn
showModalDialog(url: string, width: number, height: number): Promise<string>;  // schemes: file:, data:, https:, http://localhost
withinProgressDialog(
  text: string,
  options: { progress?: number },
  callback: (update: (text: string, progress?: number) => Promise<void>, signal: AbortSignal) => Promise<unknown>,
): Promise<unknown>;
```
`update` progress is a percentage **0–100**; pass `undefined` for indeterminate. Check `signal.aborted` / `signal.throwIfAborted()` to honor cancellation. Dialog auto-closes when the callback settles.

### Resources  (`context.resources`)
```ts
renderPreFxAudio(track: AudioTrack, startTime: number, endTime: number): Promise<string>;  // beats → WAV path in temp dir
importIntoProject(filePath: string): Promise<string>;  // copies into project; USE THE RETURNED PATH afterwards
```

### Environment  (`context.environment`)
```ts
get storageDirectory(): string | undefined;  // persistent per-extension dir (config, credentials, cache)
get tempDirectory(): string | undefined;     // temp per-extension dir (may be cleaned between sessions)
get language(): string | undefined;          // uppercase ISO 639-1, e.g. "EN", "DE", "JA"
```

## Enums

```ts
enum WarpMode { Beats=0, Tones=1, Texture=2, Repitch=3, Complex=4, ComplexPro=6 }  // NOTE: no 5
enum GridQuantization {
  NoGrid=0, EightBars=1, FourBars=2, TwoBars=3, Bar=4,
  Half=5, Quarter=6, Eighth=7, Sixteenth=8, ThirtySecond=9
}  // combine with Song.gridIsTriplet for the full grid setting
```

## Types

```ts
type NoteDescription = {
  pitch: number; startTime: number; duration: number;   // required
  velocity?: number; muted?: boolean; probability?: number;
  velocityDeviation?: number; releaseVelocity?: number; selected?: boolean;
};

interface WarpMarker { sampleTime: number; beatTime: number; }

interface ClipLoopSettings {   // all values in beats
  looping: boolean;
  startMarker: number; endMarker: number;
  loopStart: number; loopEnd: number;
}
// Validation the API enforces:
//  - startMarker ≤ endMarker
//  - loop ≥ 0.25 beats (one 16th note)
//  - looping === false  ⇒  loopStart === startMarker  &&  loopEnd === endMarker
//  - isWarped === false ⇒  positions non-negative  &&  looping === false

interface ArrangementSelection {   // note snake_case — comes straight from the host
  time_selection_start: number;    // beats
  time_selection_end: number;      // beats
  selected_lanes: Handle[];
}
interface ClipSlotSelection { selected_clip_slots: Handle[]; }

interface DeviceParameterValueItem { name: string; shortName: string; }
```

## ContextMenuScope

The `scope` string passed to `registerContextMenuAction`. It determines the command's first argument:

**Object scopes** — pass the triggered object's `Handle`:
`"AudioClip"`, `"AudioTrack"`, `"ClipSlot"`, `"DrumRack"`, `"MidiClip"`, `"MidiTrack"`, `"Sample"`, `"Scene"`, `"Simpler"`.

**Selection scopes**:
- `"ClipSlotSelection"` → passes a `ClipSlotSelection`
- `"AudioTrack.ArrangementSelection"` and `"MidiTrack.ArrangementSelection"` → pass an `ArrangementSelection`

## createAudioClip arg contract

`AudioTrack.createAudioClip` and `TakeLane.createAudioClip` take `startTime` (arrangement position); `ClipSlot.createAudioClip` does not (session slot). Full arg shape:

```ts
{ filePath: string; startTime: number; duration?: number; isWarped?: boolean; loopSettings?: ClipLoopSettings }
```
- `filePath` — absolute path (import first via `resources.importIntoProject` and use its return value).
- `duration` — capped at sample length for non-looping clips; looping clips repeat to fill; defaults to natural length.
- `isWarped` — defaults to the file's `.asd` / Live's Auto-Warp pref; **must be provided when `loopSettings` is provided**.
- `loopSettings` — requires `isWarped` defined; if `isWarped === false`, `loopSettings.looping` must be `false`.

> Prefer this **object form**. Some prose docs show an older positional `createAudioClip(path, false)` — that is stale; the type defs and examples use the object form.
