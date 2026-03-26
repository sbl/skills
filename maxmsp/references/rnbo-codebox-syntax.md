# RNBO Codebox~ Syntax Reference

Official docs: https://rnbo.cycling74.com/codebox

## Declarations

### @param -- Parameters
```javascript
@param({min: 0.0, max: 4.0}) pchratio = 2.0;
@param({min: 0, max: 1}) play = 0;
@param({enum: ["saw", "sine", "square"]}) waveform = 0;
```
- Decorator `({...})` is **required** -- bare `@param x = 0;` fails with "Expression default needs a type"
- Only visible within the RNBO patcher, NOT the top-level Max patcher
- To expose to Max: use separate `param` objects in the RNBO patcher wired to codebox inlets

### @state -- Persistent State
```javascript
@state myCounter = 0;
@state writeIdx : Int = 0;     // : Int annotation for bitwise ops
@state slope = 0.0;            // no annotation needed for floats (default type)
@state myOsc = new phasor();   // stateful operator instance
@state count;                  // no initial value (defaults to 0)
```
- Only `@state` persists between sample ticks. `let`/`var` reset each tick.
- Add `: Int` ONLY when the variable is used in bitwise ops (`&`, `|`, `>>`, `<<`, `^`, `~`)
- No need for `: number` on floats -- it's the default type

### const -- Constants
```javascript
const bufSize : Int = 131072;
const PI_HALF = 1.5707963267949;
```
- Can be used in array size declarations and `@param` expressions
- Use `: Int` when the const is used for integer/bitwise operations

### let / var -- Local Variables
```javascript
let x = 0;                // preferred
let idx : Int = 0;        // : Int only when needed for bitwise
var y = 0;                // also valid
```

## Arrays

### FixedSampleArray (preferred for audio)
```javascript
const size : Int = 131072;
@state arr = new FixedSampleArray(size);

// Direct bracket access
arr[writeIdx] = input;
let val = arr[readIdx];
```

### Other array types
`FixedFloat32Array`, `FixedFloat64Array`, `FixedIntArray`, `FixedInt32Array`, `FixedUint32Array`, etc.

### Initialization in dspsetup
```javascript
function dspsetup() {
    for (let i : Int = 0; i < size; ++i) {
        arr[i] = 0.0;
    }
}
```

## Buffers (external buffer~ references)

### Declaration
```javascript
@state buf = new buffer("externalBufferName");    // MUST use "new"
@state anon = new buffer(samplerate() * 2);       // anonymous, sized
@state named = new buffer("myBuf", 44100);        // named, sized
```

### Reading -- peek returns [value, index] array
```javascript
let val = peek(buf, sampleIndex)[0];    // [0] extracts the sample value
```

### Writing -- value BEFORE index
```javascript
poke(buf, valueToWrite, sampleIndex);
```

### Recommended helper pattern
```javascript
function bufRead(idx) { return peek(buf, idx)[0]; }
function bufWrite(idx, val) { poke(buf, val, idx); }
```

### Buffer info
```javascript
let frames = dim(buf);         // buffer length in samples
let chans = channels(buf);     // number of channels
```

## I/O

```javascript
let input = in1;     // first signal inlet
out1 = value;        // first signal outlet
out2 = value;        // second outlet
```
Inlet/outlet count is determined by usage in the code. Each `inN`/`outN` creates one.

## Functions

```javascript
function myFunc(a, b) {
    // can access @state directly
    return a + b;
}

function multiReturn(): list {
    return [1, 2];
}
```

## Special Functions

| Function | When called | Notes |
|----------|------------|-------|
| `init()` | Object creation | Cannot access buffers/data |
| `startup()` | After all objects created | Buffers available |
| `dspsetup()` | Audio on / settings change | `codebox~` only |

## Control Flow

```javascript
if (cond) { ... } else if (cond2) { ... } else { ... }
while (i < n && i < 64) { ... }            // MUST be bounded
for (let i : Int = 0; i < 100; ++i) { ... }
switch (stage) { case 0: ...; break; }
let x = cond ? a : b;
```

## Built-in Functions

### Math
`sin cos tan atan atan2 exp log log2 log10 pow sqrt abs`
`floor ceil trunc round min max clamp`

### Utility
`random(min, max)` -- random float in range
`safediv(a, b)` -- safe division (no crash on b=0)
`samplerate()` -- current sample rate
`vectorsize()` -- signal vector size
`post("msg")` / `post(value)` -- print to Max console

### Constants
`PI` `TWOPI` `HALFPI` `E` `LN2` `LN10` `SQRT2`

## Common Gotchas

1. **`@param` needs decorator**: `@param({min:0, max:1}) x = 0;` not `@param x = 0;`
2. **`@param` not visible to top-level Max** -- use RNBO `param` objects
3. **`new` required for buffer**: `new buffer("name")` not `buffer("name")`
4. **peek returns array**: `peek(buf, idx)[0]` to get the value
5. **poke arg order**: `poke(buf, value, idx)` -- value before index
6. **`: Int` for bitwise ops only** -- don't annotate everything
7. **Loops must be bounded** -- `while (i < n && i < 64)`
8. **No `.codebox` files** -- code lives embedded in the .maxpat JSON
9. **`let` is valid and preferred** over `var` for local variables
10. **No `@initial` on param objects** -- default is positional: `param name default @min X @max Y`
