---
name: rnbo
description: Expert RNBO patcher and codebox~ code generation for Cycling '74 Max/MSP. Use when creating RNBO patches, writing codebox~ DSP code, generating .maxpat files with embedded rnbo~ objects, porting C/C++ audio code to RNBO, or when the user mentions RNBO, codebox~, rnbo~, or RNBO web/VST export. For gen~ codebox (GenExpr syntax), use the genexpr skill instead. RNBO codebox~ has DIFFERENT syntax from gen~ codebox.
---

# RNBO for Max/MSP

Generate RNBO patchers and codebox~ code for Cycling '74 Max/MSP. RNBO enables exporting Max patches as VST/AU plugins, web apps, or embedded targets.

## Reference Files

- [Codebox Syntax](references/codebox-syntax.md) - Complete RNBO codebox~ language reference (params, state, buffers, arrays, functions)
- [Maxpat Structure](references/maxpat-structure.md) - JSON structure for generating .maxpat files with rnbo~ objects programmatically
- [C++ Translation](references/cpp-translation.md) - Patterns for porting C/C++ audio DSP to RNBO codebox~

## CRITICAL: RNBO codebox~ is NOT gen~ codebox

RNBO codebox~ and gen~ codebox look similar but have **different syntax**:

| Feature | gen~ codebox (GenExpr) | RNBO codebox~ |
|---------|----------------------|---------------|
| Parameters | `Param name(default);` | `@param({min: 0, max: 1}) name = default;` |
| State | `History name(initial);` | `@state name = initial;` |
| Arrays | `Data name(size);` | `@state name = new FixedSampleArray(size);` |
| Buffers | `Buffer name("ref");` | `@state name = new buffer("ref");` |
| Buffer read | `peek(buf, idx, chan)` returns scalar | `peek(buf, idx)` returns **[value, index] array** -- use `[0]` |
| Buffer write | `poke(buf, value, idx, chan)` | `poke(buf, value, idx)` -- **value before index** |
| Local vars | `x = 0;` (implicit) | `let x = 0;` (explicit) |
| Debugging | N/A | `post("msg"); post(value);` |

**Never mix up the syntaxes.** If the user asks for gen~ code, use the genexpr skill.

## Quick Reference

### Codebox~ Declarations
```javascript
// Parameters (visible within RNBO patcher only)
@param({min: -4.0, max: 4.0}) rate = 1.0;
@param({enum: ["saw", "sine", "square"]}) waveform = 0;

// Persistent state
@state phase = 0.0;
@state writeIdx : Int = 0;          // : Int for bitwise ops
@state counter : Int = 0;

// Constants
const bufSize : Int = 131072;

// Arrays
@state myArray = new FixedSampleArray(bufSize);   // bracket access: myArray[i]

// External buffer reference
@state buf = new buffer("bufname");               // MUST use "new"
```

### Buffer Access
```javascript
// Read: peek returns [value, index] -- MUST use [0]
let val = peek(buf, idx)[0];

// Write: value BEFORE index
poke(buf, value, idx);

// Helper pattern (recommended):
function bufRead(idx) { return peek(buf, idx)[0]; }
function bufWrite(idx, val) { poke(buf, val, idx); }
```

### Param Visibility
`@param` inside codebox~ is **only visible within the RNBO patcher**, not the top-level Max patcher. To expose params externally:

1. Add `param` objects in the RNBO patcher (these are visible to Max)
2. Connect param outputs to codebox~ inlets
3. Read values from `in2`, `in3`, etc. inside codebox~

### RNBO param object syntax
```
param name default @min X @max Y
```
Default is a **positional** argument (2nd arg). There is no `@initial` attribute.

### .maxpat Generation
When generating patchers programmatically, see [Maxpat Structure](references/maxpat-structure.md) for the exact JSON format. Key rules:
- `codebox~` uses `maxclass: "codebox~"` (not `"newobj"`)
- Code goes in `"code"` AND `"rnbo_extra_attributes"."code"`
- `in~` has `numinlets: 0`
- `rnbo~` embeds subpatcher under `"patcher"` key
- No external `.codebox` files exist -- code lives in the maxpat JSON
