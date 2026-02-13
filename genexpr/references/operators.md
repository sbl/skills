# GenExpr Operators Reference

## Gen~ DSP Operators

### Buffer Operations
| Operator | Syntax | Description |
|----------|--------|-------------|
| `buffer` | `buffer(name)` | Reference external buffer~ object |
| `channels` | `channels(name)` | Get channel count of buffer/data |
| `cycle` | `cycle(freq)` | Sine oscillator with wavetable lookup |
| `data` | `data(name, len, chans)` | Internal sample array |
| `dim` | `dim(name)` | Get length in samples |
| `lookup` | `lookup(name, chans)` | Waveshaping lookup |
| `nearest` | `nearest(name, chans)` | Non-interpolated lookup |
| `peek` | `peek(name, chans, idx)` | Read from buffer/data |
| `poke` | `poke(name, chan, val, idx)` | Write to buffer/data |
| `sample` | `sample(name, chans, idx)` | Linear interpolated lookup |
| `splat` | `splat(name, chan, val, phase)` | Interpolated overdub write |
| `wave` | `wave(name, chans, phase, start, end)` | Wavetable with range |

### Conversion
| Operator | Syntax | Description |
|----------|--------|-------------|
| `atodb` | `atodb(amp)` | Amplitude to decibels |
| `dbtoa` | `dbtoa(db)` | Decibels to amplitude |
| `ftom` | `ftom(freq)` | Frequency to MIDI note |
| `mtof` | `mtof(note)` | MIDI note to frequency |
| `mstosamps` | `mstosamps(ms)` | Milliseconds to samples |
| `sampstoms` | `sampstoms(samps)` | Samples to milliseconds |

### Constants
| Operator | Description |
|----------|-------------|
| `samplerate` / `SAMPLERATE` | Current sample rate |
| `vectorsize` / `VECTORSIZE` | DSP vector size |
| `fftsize` / `FFTSIZE` | FFT frame size (pfft~) |
| `ffthop` / `FFTHOP` | FFT hop size |

### Feedback & Delay
| Operator | Syntax | Description |
|----------|--------|-------------|
| `delay` | `delay(maxtime, taps)` | Multi-tap delay line |
| `history` | `history(name, init)` | Single-sample delay (state variable) |

### Filters
| Operator | Syntax | Description |
|----------|--------|-------------|
| `change` | `change(in)` | Sign of difference from previous |
| `dcblock` | `dcblock(in)` | DC blocking filter |
| `delta` | `delta(in)` | Difference from previous sample |
| `interp` | `interp(t, a, b)` | Interpolate between inputs |
| `latch` | `latch(in, ctrl)` | Sample and hold |
| `phasewrap` | `phasewrap(in)` | Wrap to -pi..+pi |
| `sah` | `sah(in, ctrl, thresh)` | Sample and hold on trigger |
| `slide` | `slide(in, up, down)` | Envelope follower / slew limiter |

### Waveforms
| Operator | Syntax | Description |
|----------|--------|-------------|
| `phasor` | `phasor(freq)` | Sawtooth ramp 0..1 |
| `rate` | `rate(phase, mult)` | Phase time-scaling |
| `train` | `train(period, width, phase)` | Pulse train |
| `triangle` | `triangle(phase, duty)` | Triangle/ramp wave |
| `noise` | `noise()` | White noise |

### Integrators
| Operator | Syntax | Description |
|----------|--------|-------------|
| `+=` / `accum` | `+=(val, reset)` | Accumulator |
| `*=` / `mulequals` | `*=(mult, reset)` | Multiplying accumulator |
| `counter` | `counter(inc, reset, max)` | Counter with reset |

### Utilities
| Operator | Syntax | Description |
|----------|--------|-------------|
| `fixdenorm` | `fixdenorm(in)` | Replace denormals with zero |
| `fixnan` | `fixnan(in)` | Replace NaN with zero |
| `isdenorm` | `isdenorm(in)` | Detect denormals |
| `isnan` | `isnan(in)` | Detect NaN |
| `t60` | `t60(time)` | Calculate decay coefficient |
| `t60time` | `t60time(coeff)` | Estimate decay time |

### Voice/Channel
| Operator | Description |
|----------|-------------|
| `elapsed` | Samples since DSP started |
| `voice` | Current poly~ voice index |
| `voicecount` | Total poly~ voices |
| `mc_channel` | Current mc.gen~ channel |
| `mc_channelcount` | Total mc.gen~ channels |

---

## Common Operators

### Comparison
| Operator | Alt | Description |
|----------|-----|-------------|
| `>` | `gt` | Greater than (returns 0 or 1) |
| `<` | `lt` | Less than |
| `==` | `eq` | Equal |
| `!=` | `neq` | Not equal |
| `>=` | `gte` | Greater or equal |
| `<=` | `lte` | Less or equal |
| `>p` | `gtp` | Pass if greater, else 0 |
| `<p` | `ltp` | Pass if less, else 0 |
| `max` | `maximum` | Maximum of inputs |
| `min` | `minimum` | Minimum of inputs |
| `step` | | 0 if in1 < in2, else 1 |

### Logic
| Operator | Alt | Description |
|----------|-----|-------------|
| `!` | `not` | Logical NOT |
| `&&` | `and` | Logical AND |
| `\|\|` | `or` | Logical OR |
| `^^` | `xor` | Logical XOR |
| `bool` | | Convert nonzero to 1 |

### Math
| Operator | Alt | Description |
|----------|-----|-------------|
| `+` | `add` | Addition |
| `-` | `sub` | Subtraction |
| `*` | `mul` | Multiplication |
| `/` | `div` | Division |
| `%` | `mod` | Modulo |
| `neg` | | Negate |
| `absdiff` | | `abs(a - b)` |

### Numeric
| Operator | Description |
|----------|-------------|
| `abs` | Absolute value |
| `ceil` | Round up |
| `floor` | Round down |
| `trunc` | Round toward zero |
| `fract` | Fractional part |
| `sign` | -1, 0, or 1 |
| `round` | Round to nearest |

### Powers & Exponentials
| Operator | Description |
|----------|-------------|
| `exp` | e^x |
| `exp2` | 2^x |
| `fastexp` | Approximated e^x |
| `ln` / `log` | Natural log |
| `log10` | Log base 10 |
| `log2` | Log base 2 |
| `pow` | x^y |
| `sqrt` | Square root |
| `fastpow` | Approximated power |

### Trigonometry
| Operator | Description |
|----------|-------------|
| `sin` | Sine (radians) |
| `cos` | Cosine |
| `tan` | Tangent |
| `asin` | Arc sine |
| `acos` | Arc cosine |
| `atan` | Arc tangent |
| `atan2` | Two-argument arc tangent |
| `sinh` / `cosh` / `tanh` | Hyperbolic |
| `asinh` / `acosh` / `atanh` | Inverse hyperbolic |
| `fastsin` / `fastcos` / `fasttan` | Approximated trig |
| `hypot` | sqrt(x² + y²) |

### Range
| Operator | Description |
|----------|-------------|
| `clamp` / `clip` | Clamp to min..max |
| `fold` | Fold at boundaries |
| `wrap` | Wrap at boundaries |
| `scale` | Map input range to output range |

### Routing
| Operator | Description |
|----------|-------------|
| `?` / `switch` | Ternary: cond ? a : b |
| `gate` | Route to selected outlet |
| `selector` | Select from multiple inputs |
| `mix` | Linear interpolation |
| `smoothstep` | Smooth interpolation (S-curve) |
| `s` / `send` | Send to named receive |
| `r` / `receive` | Receive from named send |

### Constants
| Constant | Value |
|----------|-------|
| `pi` / `PI` | 3.14159... |
| `twopi` / `TWOPI` | 6.28318... |
| `halfpi` / `HALFPI` | 1.5708... |
| `e` / `E` | 2.71828... |
| `phi` / `PHI` | 1.61803... (golden ratio) |
| `sqrt2` / `SQRT2` | 1.41421... |
| `ln2` / `LN2` | 0.69315... |
| `ln10` / `LN10` | 2.30259... |

### Declaration
| Operator | Syntax | Description |
|----------|--------|-------------|
| `param` | `param name(default, min, max)` | External parameter |
| `in` | `in 1` | Patcher input |
| `out` | `out 1` | Patcher output |
