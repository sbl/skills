# Max/MSP Dialect Comparison

Side-by-side reference for GenExpr (gen~ codebox), RNBO (codebox~), and js (v8).

**Do not mix syntaxes.** gen~ and RNBO look similar but are incompatible.

## Declarations

| Feature | gen~ (GenExpr) | RNBO (codebox~) | js (v8) |
|---------|---------------|-----------------|---------|
| Parameter | `Param name(default, min=0, max=1);` | `@param({min: 0, max: 1}) name = default;` | N/A (use inlets) |
| State variable | `History name(initial);` | `@state name = initial;` | Instance property: `this.x = 0;` |
| Local variable | `x = 0;` (implicit) | `let x = 0;` (explicit) | `let x = 0;` |
| Constant | `pi`, `twopi`, `samplerate` | `const X = val;`, `PI`, `samplerate()` | Standard JS |
| Integer annotation | N/A | `: Int` (for bitwise ops only) | Standard JS |

## Arrays & Buffers

| Feature | gen~ (GenExpr) | RNBO (codebox~) | js (v8) |
|---------|---------------|-----------------|---------|
| Internal array | `Data name(size);` | `@state arr = new FixedSampleArray(size);` | `var arr = new Array(size);` |
| Array read | `peek(name, chan, idx)` | `arr[idx]` | `arr[idx]` |
| Array write | `poke(name, val, chan, idx)` | `arr[idx] = val;` | `arr[idx] = val;` |
| External buffer | `Buffer name("ref");` | `@state buf = new buffer("ref");` | `var buf = new Buffer("ref");` |
| Buffer read | `peek(buf, chan, idx)` returns scalar | `peek(buf, idx)[0]` returns [value, index] | `buf.peek(chan, idx)` |
| Buffer write | `poke(buf, val, chan, idx)` | `poke(buf, val, idx)` value before index | `buf.poke(chan, idx, val)` |
| Buffer length | `dim(name)` | `dim(buf)` | `buf.framecount()` |

## I/O

| Feature | gen~ (GenExpr) | RNBO (codebox~) | js (v8) |
|---------|---------------|-----------------|---------|
| Signal input | `in1`, `in2` | `in1`, `in2` | N/A (message-rate only) |
| Signal output | `out1 = val;` | `out1 = val;` | N/A |
| Message input | N/A | N/A | `function bang()`, `function msg_int(v)` |
| Message output | N/A | N/A | `outlet(0, value)` |
| Inlet count | Determined by `inN` usage | Determined by `inN` usage | `inlets = 2;` |
| Outlet count | Determined by `outN` usage | Determined by `outN` usage | `outlets = 2;` |

## Functions

| Feature | gen~ (GenExpr) | RNBO (codebox~) | js (v8) |
|---------|---------------|-----------------|---------|
| Definition | `myFunc(a, b) { return a+b; }` | `function myFunc(a, b) { return a+b; }` | `function myFunc(a, b) { return a+b; }` |
| Special funcs | N/A | `init()`, `dspsetup()`, `startup()` | `loadbang()`, `bang()`, `msg_int()` |
| Debugging | N/A | `post("msg");` | `post("msg");` |

## Control Flow

| Feature | gen~ (GenExpr) | RNBO (codebox~) | js (v8) |
|---------|---------------|-----------------|---------|
| Conditional | `y = cond ? a : b;` | `if/else`, ternary | Standard JS |
| Loop | Not supported | `for`, `while` (must be bounded) | Standard JS |
| Switch | Not supported | `switch/case` | Standard JS |
| Branchless | `switch(cond, a, b)` | ternary | ternary |

## Threading & Performance

| Aspect | gen~ | RNBO | js |
|--------|------|------|----|
| Thread | Audio (high priority) | Audio (high priority) | Low priority (scheduler) |
| Processing | Per-sample | Per-sample | Message-rate |
| Denormals | Use `fixdenorm()` | Use `safediv()` | N/A |
| Performance | Branchless preferred | Bounded loops required | GC pauses possible |

## Common Gotchas

| Mistake | Consequence |
|---------|-------------|
| Using `History` in RNBO codebox~ | Syntax error. Use `@state`. |
| Using `@state` in gen~ codebox | Syntax error. Use `History`. |
| `peek()` in RNBO without `[0]` | Gets [value, index] array, not scalar |
| `Param` in RNBO codebox~ | Syntax error. Use `@param({...})` or inlet from `param` object |
| `let` in gen~ codebox | Syntax error. Variables are implicit. |
| Bare `@param x = 0;` in RNBO | Fails. Decorator `({...})` is required. |
| Using js for audio-rate DSP | js runs in low-priority thread, not suitable for audio |
