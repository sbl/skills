# Project Setup, Build & Packaging

## Scaffold a new extension

```sh
mkdir my-extension && cd my-extension
npx @ableton-extensions/create-extension     # or: npx file:/path/to/ableton-create-extension-*.tgz
```
Prompts for name, author, the Live application path, and whether you need a UI (adds a webview setup). Generated layout:

```
.env                 # EXTENSION_HOST_PATH=… (gitignored, per-machine)
build.ts             # esbuild bundler
manifest.json        # extension metadata (read by build + Live)
package.json         # scripts + file: tarball deps
tsconfig.json
src/extension.ts     # entry point exporting activate()
vendor/              # ableton-extensions-{sdk,cli}-*.tgz  (beta: not on npm)
```

During beta the SDK and CLI ship as **local tarballs** referenced with `file:` deps, not from a registry.

## manifest.json

```json
{
  "name": "my-extension",
  "author": "Your Name",
  "version": "1.0.0",
  "entry": "dist/extension.js",
  "minimumApiVersion": "1.0.0"
}
```
`entry` is the bundle output path (build.ts reads it). `minimumApiVersion` must match the string passed to `initialize`.

## package.json scripts

```json
{
  "type": "module",
  "main": "dist/extension.js",
  "scripts": {
    "build": "tsc --noEmit && tsx build.ts --production",
    "build:dev": "tsc --noEmit && tsx build.ts",
    "start": "npm run build:dev && extensions-cli run",
    "package": "npm run build && extensions-cli package"
  },
  "dependencies": {
    "@ableton-extensions/sdk": "file:./vendor/ableton-extensions-sdk-1.0.0-beta.0.tgz"
  },
  "devDependencies": {
    "@ableton-extensions/cli": "file:./vendor/ableton-extensions-cli-1.0.0-beta.0.tgz",
    "esbuild": "0.28.0", "tsx": "^4.19.0", "typescript": "^5.9.3", "@types/node": "^24.1.0"
  }
}
```
- `build` typechecks then bundles; there is **no separate test runner or linter** — `tsc --noEmit` is the correctness gate.
- `start` = dev build then launch in Live's Extension Host.
- `package` = production build then produce a `.ablx`.

## tsconfig.json

The scaffold uses a strict config. Key flags:

```json
{
  "compilerOptions": {
    "module": "nodenext", "moduleResolution": "nodenext", "target": "esnext",
    "strict": true,
    "noUncheckedIndexedAccess": true,     // ← array indexing yields T | undefined
    "exactOptionalPropertyTypes": true,
    "esModuleInterop": true,
    "noEmit": true,
    "rootDir": "./src"
  },
  "include": ["src/**/*"]
}
```
`noUncheckedIndexedAccess` is why examples use the `!` non-null assertion on indexing (`tracks[i]!`, `selection.selected_lanes[0]!`). Match that.

## build.ts (esbuild)

The Extension Host loads a **single standalone JS file** and does not resolve `node_modules` at runtime, so bundling is mandatory once you use npm packages or multiple files.

```ts
import * as esbuild from "esbuild";
import * as fs from "node:fs";

const manifest = JSON.parse(fs.readFileSync("manifest.json", "utf8"));
const production = process.argv.includes("--production");

await esbuild.build({
  entryPoints: ["src/extension.ts"],
  outfile: manifest.entry,      // reads dist path from the manifest
  bundle: true,
  format: "cjs",                // CommonJS for the host
  platform: "node",
  sourcesContent: false,
  logLevel: "info",
  minify: production,
  sourcemap: !production,
  loader: { ".html": "text" },  // lets you `import html from "./interface.html"` (see webview-ui)
});
```
Any bundler works — the host and CLI only care about the file named in `manifest.entry`.

## Running (dev loop)

1. Enable **Developer Mode**: Live → Preferences → Extensions. This releases the Extension Host so the CLI can attach.
2. `npm start`.

`extensions-cli run` options:
- `--live <path>` — path to the Live `.app`/`.exe`/install root/`ExtensionHostNodeModule.node`. Overrides `EXTENSION_HOST_PATH`.
- `--storage-directory <path>`, `--temp-directory <path>` — override the env dirs.
- `--inspect` — attach a debugger (`--inspect-brk`, e.g. from VS Code).

If `--live` is omitted the CLI reads `EXTENSION_HOST_PATH` from the environment or a `.env` file. The bundled examples have no `.env`, so run them with an explicit path: `npm start -- --live "/Applications/Ableton Live 12 Beta.app"`.

## Packaging (.ablx)

```sh
npm run build                              # ALWAYS build first — package does not build for you
npx extensions-cli package                 # → <name>.ablx
npx extensions-cli package -o dist/x.ablx  # custom output path
npx extensions-cli package -i assets templates/index.html   # include extra files/dirs (recursive)
```
A `.ablx` is the bundled JS + `manifest.json` + any explicitly included assets. Included paths must be relative to the extension dir and cannot point outside it. Users install by dropping the `.ablx` onto Live's Extensions settings page.

## Filesystem permission model

Enforced now, with a stricter OS sandbox coming. Applies to child processes and native addons too — no workarounds.

- Read/write **only** `context.environment.storageDirectory` (persistent) and `context.environment.tempDirectory` (temporary, may be cleared between sessions). Both can be `undefined`.
- Do **not** touch arbitrary paths (Documents, Downloads, Desktop, project folder directly).
- To use a file that lives outside the allowed dirs, call `context.resources.importIntoProject(path)` — it runs host-side, can reach the file, and returns a managed path. Use that returned path in later API calls.

## Logging & debugging

`console.log/info/warn/error` and uncaught-exception stack traces are written to `ExtensionHost.txt`:
- macOS: `~/Library/Preferences/Ableton/Live x.x.x/ExtensionHost.txt`
- Windows: `%APPDATA%\Ableton\Live x.x.x\Preferences\ExtensionHost.txt`

This is the primary dev-time feedback channel. Use `--inspect` for step debugging.

## Handle invalidation (gotcha)

A `Handle` is a reference to an object *at the moment you received it*. It becomes invalid when:
- the object is deleted,
- the object is moved (Live allocates a new handle; old ones die),
- the session changes (closing/loading a Set invalidates all handles).

Using a resolved object backed by a stale handle throws. **Don't cache SDK objects or handles long-term** — resolve on demand or re-query the model.
