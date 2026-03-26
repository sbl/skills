# C++ to RNBO Codebox~ Translation Guide

Patterns for porting C/C++ audio DSP code to RNBO codebox~.

## Type Mapping

| C++ | RNBO codebox~ |
|-----|---------------|
| `float`, `double` | default (no annotation needed) |
| `int`, `long`, `unsigned int` | `: Int` annotation |
| `bool` | `: Int` (0/1) |
| `float*` (buffer) | `new buffer("name")` or `new FixedSampleArray(size)` |
| `class` / `struct` | Flatten to prefixed `@state` variables |
| `enum` | Integer constants: `// State: Playing=0, Stopped=1, FadeIn=2, FadeOut=3` |
| `static constexpr` | `const name : Int = value;` |

## Class Flattening

C++ classes become prefixed `@state` variables and standalone functions:

```cpp
// C++
class SubHead {
    float phase_;
    int state_;
    void updatePhase(float start, float end);
};
```
```javascript
// RNBO codebox~
@state sh0_phase = 0.0;
@state sh0_state : Int = 1;
function sh0_updatePhase(start, end) { ... }
```

For two instances (e.g. dual crossfade heads), duplicate with `sh0_` and `sh1_` prefixes.

## Buffer Access

### Internal data (no sharing needed)
Use `FixedSampleArray` with bracket access -- fastest, simplest:
```javascript
const size : Int = 131072;
@state buf = new FixedSampleArray(size);
buf[idx] = value;           // write
let val = buf[idx];          // read
```

### External buffer (shared with Max/RNBO patcher)
Use `buffer()` with peek/poke:
```javascript
@state buf = new buffer("name");
let val = peek(buf, idx)[0];     // peek returns [value, index]
poke(buf, value, idx);           // value BEFORE index
```

**Choose FixedSampleArray** for internal delay lines, LUTs, ring buffers.
**Choose buffer()** when the buffer needs to be shared, loaded with audio files, or sized from the patcher.

## Common C++ Patterns

### Circular buffer with power-of-2 masking
```cpp
// C++
buf_[wrIdx_ & bufMask_] = value;
```
```javascript
// RNBO
@state wrIdx : Int = 0;
@state bufMask : Int = 524287;
function wrapBuf(x) { return (x + bufFrames) & bufMask; }
bufWrite(wrapBuf(wrIdx), value);
```

### One-pole lowpass (LogRamp)
```cpp
// C++: y = x + (y0 - x) * b where b = exp(-6.9 / (t * sr))
float smooth1pole(float x, float x0, float b) { return x + (x0 - x) * b; }
```
```javascript
// RNBO
@state ramp_b = 1.0;
@state ramp_x0 = 0.0;
@state ramp_y0 = 0.0;
function ramp_setTime(t) { ramp_b = exp(-6.9078 / (t * sr)); }
function ramp_update() { ramp_y0 = ramp_x0 + (ramp_y0 - ramp_x0) * ramp_b; return ramp_y0; }
```

### Hermite interpolation
```cpp
// C++
template<typename T>
static T hermite(T x, T y0, T y1, T y2, T y3) {
    return (((0.5*(y3-y0) + 1.5*(y1-y2))*x + (y0-2.5*y1+2.*y2-0.5*y3))*x + 0.5*(y2-y0))*x + y1;
}
```
```javascript
// RNBO (identical math, just different declaration)
function hermite(x, y0, y1, y2, y3) {
    return (((0.5*(y3-y0) + 1.5*(y1-y2))*x + (y0-2.5*y1+2.0*y2-0.5*y3))*x + 0.5*(y2-y0))*x + y1;
}
```

### State variable filter (Chamberlin)
```cpp
// C++ -- Svf::update(float in)
v0 = in; v1z = v1; v2z = v2;
v3 = v0 + v0z - 2.f * v2z;
v1 += g1 * v3 - g2 * v1z;
v2 += g3 * v3 + g4 * v1z;
v0z = v0;
lp = v2; bp = v1;
hp = v0 - rq * v1 - v2;
```
```javascript
// RNBO -- identical math, @state for persistence
@state svf_v0z = 0.0;
@state svf_v1 = 0.0;
@state svf_v2 = 0.0;
// ... g1-g4 coefficients as @state ...
function svf_process(inv) {
    let v0 = inv;
    let v1z = svf_v1;
    let v2z = svf_v2;
    let v3 = v0 + svf_v0z - 2.0 * v2z;
    svf_v1 += svf_g1 * v3 - svf_g2 * v1z;
    svf_v2 += svf_g3 * v3 + svf_g4 * v1z;
    svf_v0z = v0;
    // mix outputs as needed
    return svf_v2 * lpMix + svf_v1 * bpMix + ...;
}
```

### Resampler (variable-rate)
In C++, resamplers return variable output counts (0 to N frames per input). In codebox~ (per-sample processing), inline the resampler into the write function with a bounded while loop:
```javascript
function poke_with_resamp(in_sample, pre, rv) {
    // push to ring buffer
    resampIdx = (resampIdx + 1) & 3;
    resampBuf[resampIdx] = in_sample;
    // compute output frames
    let p = resampPhase + resampRate;
    let nf : Int = trunc(p);
    if (resampRate > 1.0) {
        if (nf < 1) { nf = 1; }
        let f = (1.0 - resampPhase) * resampPhi;
        let i : Int = 0;
        while (i < nf && i < 64) {       // bounded!
            f += resampPhi;
            let yv = hermite_interpolate(f);
            // read-modify-write buffer
            bufWrite(wrIdx, bufRead(wrIdx) * pre + yv * rv);
            wrIdx = wrapBuf(wrIdx + incDir);
            i += 1;
        }
        resampPhase = p - trunc(p);
    } else { /* downsampling: 0 or 1 output */ }
}
```

## What to Simplify

| C++ Feature | RNBO Approach |
|-------------|---------------|
| `std::atomic` | Not needed (single-threaded) |
| Template classes | Duplicate with prefixes (`sh0_`, `sh1_`) |
| Virtual functions | Just use regular functions |
| Memory allocation | `FixedSampleArray` or `buffer()` (fixed at init) |
| Block processing | Per-sample in codebox~ (no block loops) |
| Thread-safe phase | Just read `@state` directly |
| Debug/test buffers | Remove (use `post()` for debugging) |

## Parameter Exposure

To make codebox params accessible from the top-level Max patcher:

1. Add `param` objects in the RNBO patcher: `param rate 1 @min -4 @max 4`
2. Wire param outlet 0 to codebox~ inlet (in2, in3, etc.)
3. In codebox~, read from inlets: `let p_rate = in2;`
4. From Max, send messages: `rate 0.5` to rnbo~
