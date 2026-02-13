---
name: genexpr
description: Expert GenExpr code generation for Max/MSP gen~ objects. Use when creating DSP algorithms, translating C/C++ audio code to GenExpr, implementing filters, oscillators, distortion, delays, reverbs, envelopes, or granular synthesis in gen~. Triggers on requests involving gen~, codebox, or GenExpr syntax.
---

# GenExpr for Max/MSP

Generate expert GenExpr code for gen~ objects in Max/MSP.

## Reference Files

- [Operators Reference](references/operators.md) - Complete gen~ and common operators
- [C++ Translation](references/cpp-translation.md) - Patterns for converting C/C++ to GenExpr
- [DSP Algorithms](references/dsp-algorithms.md) - Complete, ready-to-use implementations (filters, distortion, delay, oscillators, envelopes, granular)
- [Best Practices](references/best-practices.md) - Design patterns, idioms, reusable functions, naming conventions (from "Generating Sound and Organizing Time")

## GenExpr Syntax Quick Reference

### State Variables
```
History variableName(initialValue);
```

### Parameters
```
Param name(default, min=0, max=1);
```

### Input/Output
```
input = in1;      // Read inlet 1
out1 = output;    // Write to outlet 1
```

### Essential Operators
- `delay(signal, samples)` - Delay line
- `phasor(freq)` - Ramp oscillator 0..1
- `cycle(freq)` - Sine oscillator
- `slide(in, up, down)` - Slew limiter / envelope follower
- `wrap(x, lo, hi)` - Wrap to range
- `clamp(x, lo, hi)` - Clamp to range
- `fixdenorm(x)` - Prevent denormal slowdown

### Constants
`samplerate`, `twopi`, `pi`, `halfpi`

## Output Format

Provide GenExpr code with:
1. Clear parameter declarations with sensible ranges
2. Comments explaining the algorithm
3. Usage example showing how to use in Max/MSP

**Example output structure:**
```
// [Algorithm Name]
// [Brief description]

Param paramName(default, min=X, max=Y);
History stateName(0);

// [Algorithm implementation with inline comments]
...

out1 = output;
```

**Usage in Max/MSP:**
1. Create `gen~` object
2. Open and add `codebox`
3. Paste the code
4. Connect inlets/outlets as described

## C++ Translation Guidelines

When translating C/C++ code:

1. Replace `static float` with `History`
2. Replace array access with `peek`/`poke` on `Data` objects
3. Replace `while` phase wrapping with `wrap()`
4. Use built-in operators (`delay`, `slide`) when they match the algorithm
5. Add `fixdenorm()` in feedback paths
6. See [C++ Translation](references/cpp-translation.md) for detailed patterns

## Key Design Patterns

### Safe Parameter Handling
```
Q = max(Q, 0.00001);           // Prevent divide by zero
freq = clip(freq, 0, 1);       // Clamp normalized frequency
time = max(time_samples, 1);   // At least 1 sample
```

### Filter Coefficient Calculation
For bilinear transform filters, calculate coefficients from normalized frequency:
```
omega = twopi * hz / samplerate;
g = tan(omega / 2);            // Prewarp for bilinear transform
```

### Phasor-Based Modulation
Sync parameter changes to phasor cycles:
```
History latched(0);
trigger = in1 < history(in1);  // Detect wraparound
latched = trigger ? newValue : latched;
```

### Decay Coefficients
Use `t60()` for natural-sounding exponential decay:
```
time_samples = mstosamps(decay_ms);
coef = t60(max(time_samples, 1));
```

See [Best Practices](references/best-practices.md) for comprehensive patterns.
