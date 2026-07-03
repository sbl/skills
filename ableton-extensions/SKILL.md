---
name: ableton-extensions
description: Write Ableton Live extensions in TypeScript with @ableton-extensions/sdk. Covers the activate/initialize lifecycle, the Live data model (Song/Track/Clip/Device via handles), commands + context menus, modal webview dialogs, progress dialogs, transactions, and building/packaging (.ablx). Triggers on Ableton extension, Ableton Live extension, .ablx, extensions-cli, ActivationContext, ExtensionContext, registerContextMenuAction, showModalDialog, getObjectFromHandle, withinTransaction, @ableton-extensions/sdk.
---

# Ableton Live Extensions

Write extensions for Ableton Live using `@ableton-extensions/sdk` (v1.0.0-beta.0). Default language is TypeScript, bundled with esbuild, run via `extensions-cli`.

## Mental model

An extension is **Node.js/TypeScript code running inside Live's Extension Host process**, alongside Live. It has full Node + npm access. It reacts to commands (wired to context-menu items) and drives Live's object model.

It is **not** for: realtime audio/MIDI DSP, MIDI routing, drawing into Live's native UI, control surfaces, or headless/background work. Those are Max for Live's domain.

Requirements: a Live **Beta** build that supports Extensions, **Developer Mode** enabled (Preferences → Extensions), and recent Node (SDK needs `>=22.11.0`; the scaffold pins `>=24.14.1`).

## Entry-point contract

Every extension exports `activate` and calls `initialize` first:

```ts
import { initialize, type ActivationContext } from "@ableton-extensions/sdk";

export function activate(activation: ActivationContext) {
  const context = initialize(activation, "1.0.0");
  // context.commands / context.ui / context.application / context.resources / context.environment
}
```

The version string `"1.0.0"` shows up in three coupled places: the `initialize` call, `manifest.json`'s `minimumApiVersion`, and generic type params (`Track<"1.0.0">`). Pass the **lowest** version that has the features you need — older versions stay compatible with more Live releases. `initialize` throws if the host doesn't support the requested version.

## Core idioms

| Task | Idiom |
|------|-------|
| Expose an action | `commands.registerCommand(id, cb)` + `ui.registerContextMenuAction(scope, title, id)`, linked by a namespaced string `id` (e.g. `"myext.rename"`) |
| Command argument | Typed `(arg: unknown)`; cast per scope to `Handle` (object scopes), `ArrangementSelection`, or `ClipSlotSelection` |
| Resolve a handle | `context.getObjectFromHandle(handle, SomeClass)`; use `Clip` to accept audio+midi, or `DataModelObject` + `instanceof` when the type is unknown |
| Read/write a property | **Synchronous** getters/setters: `clip.name`, `clip.warpMode`, `track.mute`, `song.tempo` |
| Create/delete/import/render | **Async** — returns a `Promise`, must be `await`ed |
| Async inside a sync callback | `void (async (x: T) => { ... })(arg as T).catch((e) => console.error(e))` |
| One undo step for many mutations | `withinTransaction(() => Promise.all([...]))` then await the result |
| Long-running work | `ui.withinProgressDialog(text, {}, async (update, signal) => { ... })` |
| Custom UI / user input | `ui.showModalDialog(dataUrl, w, h)` → returns a string |

Positions and durations are in **beats**. Convert to seconds with `const beatsPerSecond = 60 / song.tempo;`.

## Minimal working example

```ts
import { initialize, type ActivationContext } from "@ableton-extensions/sdk";

export function activate(activation: ActivationContext) {
  const context = initialize(activation, "1.0.0");

  context.commands.registerCommand("myext.hello", () => {
    console.log("You right-clicked a ClipSlot!");
  });

  context.ui.registerContextMenuAction(
    "ClipSlot",
    "Say hello",
    "myext.hello",
  );
}
```

`console.*` output goes to `ExtensionHost.txt` (see project-setup reference for paths) — that's the main dev-time feedback channel.

## References

Load the reference that matches the task:

| Working on | Read |
|------------|------|
| Any data-model class, enum, or type signature (Song, Track, Clip, Device, DeviceParameter, WarpMode, NoteDescription, ClipLoopSettings, ContextMenuScope) | [api-reference](references/api-reference.md) |
| Idiomatic code recipes: handle resolution, `instanceof` narrowing, selections, transactions, progress, audio clips, MIDI notes | [patterns](references/patterns.md) |
| Project scaffold, manifest, build.ts/esbuild, tsconfig, CLI, `.ablx` packaging, filesystem permission model, logging, handle invalidation | [project-setup](references/project-setup.md) |
| Modal dialog HTML: the host message bridge, esbuild HTML inlining, Ableton-style theming, design-guide rules | [webview-ui](references/webview-ui.md) |
