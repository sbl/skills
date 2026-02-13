# GenExpr Best Practices

Patterns and idioms from "Generating Sound and Organizing Time" by Graham Wakefield and Gregory Taylor.

For complete algorithm implementations, see [dsp-algorithms.md](dsp-algorithms.md).

---

## Design Principles

### 1. Modular Design
Break complex algorithms into reusable components:
- **Coefficient calculators** - Separate subpatch for filter coefficients
- **Core processing** - The actual filtering/processing
- **Derived outputs** - Additional outputs (e.g., HP from LP, allpass from one-pole)

### 2. Normalized Frequencies
Use normalized frequency (0-1 = 0-Nyquist) internally:
```
normalized = hz / samplerate;
omega = twopi * hz / samplerate;
g = tan(omega / 2);  // bilinear transform prewarp
```

### 3. Safe Parameter Handling
```
Q = max(Q, 0.00001);         // prevent divide by zero
freq = clip(freq, 0, 1);     // clamp normalized frequency
time = max(time_samples, 1); // at least 1 sample
coef = clip(coef, 0, 0.9999); // prevent unstable feedback
```

### 4. Trapezoidal Integration
Prefer bilinear transform for filters - better frequency response near Nyquist.

---

## Reusable Function Library

From go.lib.genexpr - define in a `require`'d file:

### Waveshaping
```
// Cubic interpolation (n=0 triangle, n>15 overshoots)
cubic(p, u) {
  n = max(u*45, 0);
  return interp(p, -n, 0, 1, n+1, mode="cubic");
}

// Spline shaping (smoother)
spline(p, u) {
  n = max(u*15, 0);
  return interp(p, -n, 0, 1, n+1, mode="spline");
}

// Overdrive-style saturation
overdrive(x, shape) {
  y = 1 - exp(shape * log(1 - x));
  return clip(y, 0, 1);
}

// Soft knee: linear center, curved edges
softknee(p, u) {
  v = 1 - u;
  lin = clip(p, 0, v);
  non = (p - lin) / u;
  non = sin(non * pow(halfpi, non));
  return lin + non * u;
}
```

### Window Functions
```
unitHann(p) { return 0.5 - 0.5 * cos(pi * p); }
unitHamming(p) { return 0.53836 - 0.46164 * cos(pi * p); }

unitBlackman(p) {
  r = pi * p;
  return 0.42 - 0.5 * cos(r) + 0.08 * cos(2 * r);
}

unitGauss(p, u) {
  u = 1 / (1 - clip(u, 0, 0.999999));
  return exp(-0.5 * pow(2 * u * (p - 1), 2));
}

unitWelch(p) { x = p - 1; return 1 - x * x; }
```

### Pulse and Quantization
```
// Pulse shaping from ramp
pulse(p, u) {
  u = clip(u, 0, 0.999999);
  n = floor(1 / (1 - u)) - 0.5;
  return p * pow(sin(p * n * pi), 2);
}

// Smooth quantization (b: -1=triangle, 0=sine, +1=square)
quantize(p, b, u) {
  u = clip(u, 0, 0.999999);
  b = clip(b, -1, 0.999999);
  sharp = 2 / (1 - b) - 1;
  n = floor(1 / (1 - u));
  q = p * n;
  r = round(q);
  m = mix(r, q, pow(absdiff(r, q) * 2, sharp * sharp));
  return m / n;
}
```

---

## Common Idioms

### Phasor Wraparound Detection
```
History lastPhase(0);
trigger = in1 < lastPhase;  // true when phasor wraps
lastPhase = in1;
```

### Latch on Trigger
```
History latched(0);
latched = trigger ? newValue : latched;
```

### Sync Parameters to Phasor Cycle
Prevent clicks by only updating parameters at cycle boundaries:
```
History latchedParam(0);
trigger = phasor < history(phasor);
latchedParam = trigger ? param : latchedParam;
```

### Asymmetric Slew (Vactrol-style)
Fast attack, slow decay based on signal level:
```
time_ms = mix(fall_ms, rise_ms, control);
coef = t60(max(mstosamps(time_ms), 1));
```

### Skewed Triangle from Phasor
```
rising = phasor < skew;
triangle = rising ? phasor / skew : (1 - phasor) / (1 - skew);
```

### Decay Coefficient from Time
Use `t60()` for natural exponential decay:
```
coef = t60(max(time_samples, 1));
output = output * coef;  // decays to -60dB in time_samples
```

---

## Chaos Oscillator Template

```
Param speed(0.01, min=0, max=0.1);
History x(0.1); History y(0); History z(0);

// System equations (Lorenz example)
dx = 10 * (y - x);
dy = x * (28 - z) - y;
dz = x * y - 2.666 * z;

// Integrate
x += dx * speed; y += dy * speed; z += dz * speed;

// Normalize to -1..1
out1 = x / 20; out2 = y / 30; out3 = z / 40;
```

---

## Naming Conventions

| Prefix | Meaning |
|--------|---------|
| `go.unit.` | Unipolar (0-1) shaping |
| `go.chaos.` | Chaos oscillator |
| `go.biquad.` | Biquad filter variants |
| `go.onepole.` | One-pole filter variants |
| `go.ramp` | Phasor/ramp utilities |
| `go.sigmoid.` | Sigmoid/saturation |

### Parameters
- Descriptive inlet names: `in 1 input`, `in 2 Hz`, `in 3 Q`
- Units in names: `fall_ms`, `slide_samples`
- Constraints: `@min 0 @max 1`, `@default 0.5`

---

## Performance Tips

1. **`fixdenorm()`** in feedback paths prevents CPU spikes from denormals
2. **Branchless code** - use `switch`/`?` instead of `if`
3. **`fastsin`/`fastcos`** when precision isn't critical
4. **`t60()`** for natural-sounding decay coefficients
5. **Normalize early** - work in -1..1 or 0..1 internally
6. **Pre-compute** - move constant calculations outside feedback loops
