---
name: maxmsp
description: Max/MSP patching and code generation. Covers plain Max patching, gen~ (GenExpr DSP), RNBO (codebox~ / exportable DSP), and js (v8 scripting). Default mode is Max/MSP. Sub-dialects activate when requested. Triggers on Max/MSP, gen~, RNBO, codebox, codebox~, GenExpr, maxpat, rnbo~, js in Max.
---

# Max/MSP

Generate valid Max/MSP patchers and code. Default mode is plain Max patching. Sub-dialects (gen~, RNBO, js) activate only when the user requests them.

## Mode Routing

| User mentions | Mode | References |
|---------------|------|------------|
| patcher, maxpat, objects, connections, Max | **Max/MSP** | [maxpat-structure](references/maxpat-structure.md) |
| gen~, codebox, GenExpr, DSP algorithm | **gen~ (GenExpr)** | [genexpr-syntax](references/genexpr-syntax.md), [genexpr-algorithms](references/genexpr-algorithms.md), [genexpr-best-practices](references/genexpr-best-practices.md) |
| RNBO, codebox~, rnbo~, VST/web export | **RNBO** | [rnbo-codebox-syntax](references/rnbo-codebox-syntax.md), [rnbo-cpp-translation](references/rnbo-cpp-translation.md) |
| js, v8, JavaScript in Max | **js** | See js section below |
| C++ translation | Check target dialect | [genexpr-cpp-translation](references/genexpr-cpp-translation.md) or [rnbo-cpp-translation](references/rnbo-cpp-translation.md) |

## .maxpat Generation

Always output valid JSON. See [maxpat-structure](references/maxpat-structure.md) for complete templates.

Minimal patcher skeleton:
```json
{
  "patcher": {
    "fileversion": 1,
    "appversion": { "major": 9, "minor": 1, "revision": 2, "architecture": "x64", "modernui": 1 },
    "classnamespace": "box",
    "rect": [100, 100, 900, 700],
    "boxes": [],
    "lines": [],
    "autosave": 0
  }
}
```

Connection format:
```json
{ "patchline": { "source": ["obj-id", outlet_idx], "destination": ["obj-id", inlet_idx] } }
```

Embedding rules:
- **gen~**: `classnamespace: "dsp.gen"`, codebox code in `patcher.code`
- **RNBO**: `classnamespace: "rnbo"`, codebox~ code in both `"code"` and `"rnbo_extra_attributes"."code"`

## Dialect Quick Diffs

**Do not mix syntaxes.** See [dialect-comparison](references/dialect-comparison.md) for the full table.

| | gen~ (GenExpr) | RNBO (codebox~) | js (v8) |
|-|---------------|-----------------|---------|
| State | `History x(0);` | `@state x = 0;` | `this.x = 0;` |
| Params | `Param p(0.5, min=0, max=1);` | `@param({min:0, max:1}) p = 0.5;` | Use inlets |
| Vars | `x = 0;` (implicit) | `let x = 0;` | `let x = 0;` |
| Buffers | `peek(buf, chan, idx)` scalar | `peek(buf, idx)[0]` array | `buf.peek(chan, idx)` |
| Thread | Audio rate | Audio rate | Low priority |

## gen~ (GenExpr) Summary

For DSP processing inside `gen~` objects. Code goes in `codebox`.

```
Param freq(440, min=20, max=20000);
History phase(0);

phase += freq / samplerate;
phase = wrap(phase, 0, 1);
out1 = sin(phase * twopi);
```

Key operators: `delay`, `phasor`, `cycle`, `slide`, `wrap`, `clamp`, `fixdenorm`, `t60`
Constants: `samplerate`, `twopi`, `pi`, `halfpi`

References:
- [Syntax & operators](references/genexpr-syntax.md)
- [DSP algorithms](references/genexpr-algorithms.md) (filters, distortion, delay, oscillators, envelopes, granular)
- [Best practices](references/genexpr-best-practices.md) (patterns from "Generating Sound and Organizing Time")
- [C++ translation](references/genexpr-cpp-translation.md)

## RNBO Summary

For exportable DSP inside `rnbo~` objects. Code goes in `codebox~`. Exports to VST/AU, web, embedded.

```javascript
@param({min: 20, max: 20000}) freq = 440;
@state phase = 0;

let inc = freq / samplerate();
phase += inc;
if (phase >= 1.0) phase -= 1.0;
out1 = sin(phase * TWOPI);
```

Key differences from GenExpr: explicit `let`/`var`, `@param` needs decorator `({...})`, `peek()` returns `[value, index]`, loops must be bounded.

`@param` inside codebox~ is only visible within the RNBO patcher. To expose to Max, use `param` objects wired to codebox~ inlets.

References:
- [Codebox~ syntax](references/rnbo-codebox-syntax.md)
- [C++ translation](references/rnbo-cpp-translation.md)
- [Maxpat structure](references/maxpat-structure.md) (RNBO section)

## js Summary

The `js` object runs a v8 JavaScript engine in the **low-priority thread** (not audio rate). Use for control logic, UI scripting, patcher manipulation.

```javascript
inlets = 2;
outlets = 2;

function bang() {
    outlet(0, "hello");
}

function msg_int(v) {
    outlet(0, v * 2);
}

function msg_float(v) {
    outlet(0, v * 0.5);
}

function list() {
    var a = arrayfromargs(arguments);
    outlet(0, a.reverse());
}
```

Key APIs:
- `post("msg")` -- print to Max console
- `outlet(idx, value)` -- send from outlet
- `this.patcher` -- access parent patcher for dynamic patching
- `messnamed("receive-name", value)` -- send to named receive
- `var d = new Dict("name")` -- access named dictionaries
- `var buf = new Buffer("name")` -- access buffer~ objects

Gotchas:
- Runs in scheduler thread, not audio thread. Not for DSP.
- File saved as `.js`, referenced via `js filename.js` in Max
- `autowatch = 1;` at top of file enables live reload on save
- No ES modules. Use `include("other.js")` for code reuse.

## C++ Translation

When translating C/C++ audio code, first determine the target dialect:
- For gen~: [genexpr-cpp-translation](references/genexpr-cpp-translation.md)
- For RNBO: [rnbo-cpp-translation](references/rnbo-cpp-translation.md)

Core pattern: `static float` becomes `History` (gen~) or `@state` (RNBO). Arrays become `Data`/`peek`/`poke` (gen~) or `FixedSampleArray`/brackets (RNBO). Use built-in operators where they match the algorithm.
