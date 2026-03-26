# C/C++ to GenExpr Translation Patterns

## Key Differences

| C/C++ | GenExpr | Notes |
|-------|---------|-------|
| `float x = 0;` | `x = 0;` | No type declarations |
| `static float x;` | `History x;` | State persists between samples |
| `x++` | `x += 1` | No increment operator |
| `float* buf` | `Data buf(size)` | Use Data for arrays |
| `buf[i]` | `peek(buf, 0, i)` | Buffer access via peek/poke |
| `#define PI 3.14159` | `pi` | Built-in constants |
| `sampleRate` | `samplerate` | Built-in constant |
| `1.0f / sampleRate` | `1/samplerate` | Division works directly |

## Variable Types

### Local Variables
```
// C++
float x = 0.5;
int n = 10;

// GenExpr
x = 0.5;
n = 10;
```

### State Variables (persist between samples)
```
// C++
static float phase = 0;
static float lastSample = 0;

// GenExpr
History phase(0);
History lastSample(0);
```

### Parameters
```
// C++
float cutoff = 1000.0f;  // external param

// GenExpr
Param cutoff(1000, min=20, max=20000);
```

## Arrays and Buffers

### Fixed Arrays
```
// C++
float buffer[1024];
buffer[i] = value;
float x = buffer[i];

// GenExpr
Data buffer(1024);
poke(buffer, value, 0, i);
x = peek(buffer, 0, i);
```

### Delay Lines
```
// C++
float delayLine[maxDelay];
int writeIdx = 0;
delayLine[writeIdx] = input;
writeIdx = (writeIdx + 1) % maxDelay;
float output = delayLine[(writeIdx - delaySamples + maxDelay) % maxDelay];

// GenExpr
delayed = delay(input, delaySamples);  // Built-in delay operator
```

## Control Flow

### Conditionals
```
// C++
if (x > 0.5f) {
    y = 1.0f;
} else {
    y = 0.0f;
}

// GenExpr
y = x > 0.5 ? 1 : 0;
// or
y = (x > 0.5) * 1 + (x <= 0.5) * 0;
```

### Clamping
```
// C++
if (x < 0) x = 0;
if (x > 1) x = 1;
// or
x = std::clamp(x, 0.0f, 1.0f);

// GenExpr
x = clamp(x, 0, 1);
```

### Wrapping
```
// C++
while (phase >= 1.0f) phase -= 1.0f;
while (phase < 0.0f) phase += 1.0f;

// GenExpr
phase = wrap(phase, 0, 1);
```

## Common DSP Patterns

### One-Pole Lowpass
```
// C++
static float y1 = 0;
float a = exp(-2.0 * M_PI * cutoff / sampleRate);
float output = input + a * (y1 - input);
y1 = output;

// GenExpr
History y1(0);
a = exp(-twopi * cutoff / samplerate);
output = input + a * (y1 - input);
y1 = output;
out1 = output;
```

### Biquad Filter
```
// C++
static float x1 = 0, x2 = 0, y1 = 0, y2 = 0;
float output = b0*input + b1*x1 + b2*x2 - a1*y1 - a2*y2;
x2 = x1; x1 = input;
y2 = y1; y1 = output;

// GenExpr
History x1, x2, y1, y2;
output = b0*in1 + b1*x1 + b2*x2 - a1*y1 - a2*y2;
x2 = x1; x1 = in1;
y2 = y1; y1 = output;
out1 = output;
```

### Phasor/Oscillator
```
// C++
static float phase = 0;
phase += freq / sampleRate;
if (phase >= 1.0f) phase -= 1.0f;
float output = sin(phase * 2.0f * M_PI);

// GenExpr
History phase(0);
phase += freq / samplerate;
phase = wrap(phase, 0, 1);
out1 = sin(phase * twopi);

// Or use built-in phasor
out1 = sin(phasor(freq) * twopi);
```

### Envelope Follower
```
// C++
static float env = 0;
float attackCoef = exp(-1.0 / (attackTime * sampleRate));
float releaseCoef = exp(-1.0 / (releaseTime * sampleRate));
float inputAbs = fabs(input);
if (inputAbs > env) {
    env = attackCoef * (env - inputAbs) + inputAbs;
} else {
    env = releaseCoef * (env - inputAbs) + inputAbs;
}

// GenExpr
History env(0);
attackCoef = exp(-1 / (attackTime * samplerate));
releaseCoef = exp(-1 / (releaseTime * samplerate));
inputAbs = abs(in1);
coef = inputAbs > env ? attackCoef : releaseCoef;
env = coef * (env - inputAbs) + inputAbs;
out1 = env;

// Or use built-in slide
out1 = slide(abs(in1), attackSamps, releaseSamps);
```

## Math Function Equivalents

| C/C++ | GenExpr |
|-------|---------|
| `fabs(x)` | `abs(x)` |
| `fmod(x, y)` | `x % y` or `mod(x, y)` |
| `fmin(a, b)` | `min(a, b)` |
| `fmax(a, b)` | `max(a, b)` |
| `M_PI` | `pi` |
| `2.0 * M_PI` | `twopi` |
| `powf(x, y)` | `pow(x, y)` |
| `sqrtf(x)` | `sqrt(x)` |
| `expf(x)` | `exp(x)` |
| `logf(x)` | `log(x)` or `ln(x)` |
| `sinf(x)` | `sin(x)` |
| `cosf(x)` | `cos(x)` |
| `tanhf(x)` | `tanh(x)` |
| `floorf(x)` | `floor(x)` |
| `ceilf(x)` | `ceil(x)` |
| `roundf(x)` | `round(x)` |

## Tips

1. **Semicolons required** - End every statement with `;`
2. **History is your friend** - Use for any state that persists
3. **Use built-in operators** - `delay`, `slide`, `phasor` etc. are optimized
4. **Avoid loops when possible** - Use vectorized operations
5. **`fixdenorm()`** - Use in feedback paths to avoid denormal slowdown
6. **Comments** - Use `//` for single-line comments
