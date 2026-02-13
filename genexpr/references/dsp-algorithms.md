# DSP Algorithm Implementations

## Table of Contents
1. [Filters](#filters)
2. [Distortion & Waveshaping](#distortion--waveshaping)
3. [Delays & Reverb](#delays--reverb)
4. [Oscillators & Modulation](#oscillators--modulation)
5. [Envelopes](#envelopes)
6. [Granular Synthesis](#granular-synthesis)

---

## Filters

### One-Pole Lowpass
```
Param cutoff(1000, min=20, max=20000);
History y1(0);

coef = exp(-twopi * cutoff / samplerate);
y1 = in1 + coef * (y1 - in1);
out1 = y1;
```

### One-Pole Highpass
```
Param cutoff(1000, min=20, max=20000);
History y1(0);
History x1(0);

coef = exp(-twopi * cutoff / samplerate);
y1 = coef * (y1 + in1 - x1);
x1 = in1;
out1 = y1;
```

### Biquad (Direct Form II Transposed)
```
// Coefficients calculated externally or via params
Param b0(1); Param b1(0); Param b2(0);
Param a1(0); Param a2(0);
History z1(0); History z2(0);

output = in1 * b0 + z1;
z1 = in1 * b1 - output * a1 + z2;
z2 = in1 * b2 - output * a2;
out1 = output;
```

### State Variable Filter (SVF)
```
Param cutoff(1000, min=20, max=20000);
Param resonance(0.5, min=0, max=1);
History lp(0); History bp(0);

f = 2 * sin(pi * cutoff / samplerate);
q = 1 - resonance;
hp = in1 - lp - q * bp;
bp = bp + f * hp;
lp = lp + f * bp;

out1 = lp;  // lowpass
out2 = hp;  // highpass
out3 = bp;  // bandpass
out4 = lp + hp;  // notch
```

### Moog Ladder Filter (4-pole)
```
Param cutoff(1000, min=20, max=20000);
Param resonance(0, min=0, max=4);
History s1(0); History s2(0); History s3(0); History s4(0);

fc = cutoff / samplerate;
f = fc * 1.16;
fb = resonance * (1.0 - 0.15 * f * f);

input = in1 - fb * s4;
input = input * 0.35013 * f * f * f * f;

s1 = input + 0.3 * s1;
s2 = s1 + 0.3 * s2;
s3 = s2 + 0.3 * s3;
s4 = s3 + 0.3 * s4;

s1 = fixdenorm(s1); s2 = fixdenorm(s2);
s3 = fixdenorm(s3); s4 = fixdenorm(s4);

out1 = s4;
```

### DC Blocker
```
History x1(0); History y1(0);

R = 0.995;  // Pole radius
y1 = in1 - x1 + R * y1;
x1 = in1;
out1 = y1;
```

---

## Distortion & Waveshaping

### Soft Clip (Tanh)
```
Param drive(1, min=0.1, max=10);
Param mix(1, min=0, max=1);

driven = in1 * drive;
saturated = tanh(driven);
out1 = in1 * (1 - mix) + saturated * mix;
```

### Hard Clip
```
Param threshold(0.8, min=0.1, max=1);

out1 = clamp(in1, -threshold, threshold);
```

### Asymmetric Soft Clip
```
Param drive(2, min=1, max=10);
Param asymmetry(0.3, min=0, max=0.9);

x = in1 * drive;
pos = x > 0 ? tanh(x * (1 + asymmetry)) : 0;
neg = x <= 0 ? tanh(x * (1 - asymmetry)) : 0;
out1 = pos + neg;
```

### Tube-Style Saturation
```
Param drive(1, min=0.5, max=5);
Param bias(0.1, min=0, max=0.5);

x = in1 * drive + bias;
y = x > 0 ? 1 - exp(-x) : -1 + exp(x);
out1 = y * 0.9;
```

### Wavefolder
```
Param gain(1, min=0.5, max=10);
Param folds(4, min=1, max=8);

x = in1 * gain;
out1 = sin(x * folds);
```

### Chebyshev Waveshaper (adds harmonics)
```
// T2(x) = 2x² - 1 (adds 2nd harmonic)
// T3(x) = 4x³ - 3x (adds 3rd harmonic)
Param harmonic2(0.3, min=0, max=1);
Param harmonic3(0.2, min=0, max=1);

x = clamp(in1, -1, 1);
t2 = 2*x*x - 1;
t3 = 4*x*x*x - 3*x;
out1 = x + harmonic2 * t2 + harmonic3 * t3;
out1 = clamp(out1, -1, 1);
```

---

## Delays & Reverb

### Simple Delay
```
Param delayMs(250, min=1, max=2000);
Param feedback(0.5, min=0, max=0.99);
Param mix(0.5, min=0, max=1);

delaySamps = delayMs * samplerate / 1000;
delayed = delay(in1 + delayed * feedback, delaySamps);
out1 = in1 * (1 - mix) + delayed * mix;
```

### Ping Pong Delay
```
Param delayMs(250, min=1, max=1000);
Param feedback(0.6, min=0, max=0.95);

delaySamps = delayMs * samplerate / 1000;
History fb1(0); History fb2(0);

left = delay(in1 + fb2 * feedback, delaySamps);
right = delay(in2 + fb1 * feedback, delaySamps);
fb1 = left; fb2 = right;

out1 = in1 + right * 0.7;
out2 = in2 + left * 0.7;
```

### Allpass Filter (for reverb)
```
// g = feedback coefficient
Param g(0.7, min=0, max=0.9);
Param delaySamps(100, min=10, max=5000);

History buffer(0);
delayed = delay(buffer, delaySamps);
buffer = in1 + delayed * g;
out1 = delayed - in1 * g;
```

### Schroeder Reverb (4 comb + 2 allpass)
```
Param roomSize(0.8, min=0.1, max=0.99);
Param damping(0.5, min=0, max=1);

// Comb filter delays (in samples at 44100)
d1 = 1557; d2 = 1617; d3 = 1491; d4 = 1422;
// Allpass delays
ap1 = 225; ap2 = 556;

History c1(0); History c2(0); History c3(0); History c4(0);
History lp1(0); History lp2(0); History lp3(0); History lp4(0);

// Comb filters with lowpass damping
c1out = delay(c1, d1);
lp1 = lp1 + damping * (c1out - lp1);
c1 = in1 + lp1 * roomSize;

c2out = delay(c2, d2);
lp2 = lp2 + damping * (c2out - lp2);
c2 = in1 + lp2 * roomSize;

c3out = delay(c3, d3);
lp3 = lp3 + damping * (c3out - lp3);
c3 = in1 + lp3 * roomSize;

c4out = delay(c4, d4);
lp4 = lp4 + damping * (c4out - lp4);
c4 = in1 + lp4 * roomSize;

combSum = (c1out + c2out + c3out + c4out) * 0.25;

// Allpass filters
History ap1buf(0); History ap2buf(0);
g = 0.7;

ap1del = delay(ap1buf, ap1);
ap1buf = combSum + ap1del * g;
ap1out = ap1del - combSum * g;

ap2del = delay(ap2buf, ap2);
ap2buf = ap1out + ap2del * g;
ap2out = ap2del - ap1out * g;

out1 = ap2out;
```

---

## Oscillators & Modulation

### Band-Limited Sawtooth (PolyBLEP)
```
Param freq(440, min=20, max=20000);
History phase(0);

dt = freq / samplerate;
phase += dt;
phase = wrap(phase, 0, 1);

// Naive saw
saw = 2 * phase - 1;

// PolyBLEP correction at discontinuity
t = phase;
blep = t < dt ? (t = t/dt, t+t - t*t - 1) :
       t > 1-dt ? (t = (t-1)/dt, t*t + t+t + 1) : 0;

out1 = saw - blep;
```

### Band-Limited Square (PolyBLEP)
```
Param freq(440, min=20, max=20000);
Param pulseWidth(0.5, min=0.01, max=0.99);
History phase(0);

dt = freq / samplerate;
phase += dt;
phase = wrap(phase, 0, 1);

// Naive square
square = phase < pulseWidth ? 1 : -1;

// PolyBLEP at rising edge
t = phase;
blep1 = t < dt ? (t = t/dt, t+t - t*t - 1) :
        t > 1-dt ? (t = (t-1)/dt, t*t + t+t + 1) : 0;

// PolyBLEP at falling edge
t2 = wrap(phase - pulseWidth, 0, 1);
blep2 = t2 < dt ? (t2 = t2/dt, t2+t2 - t2*t2 - 1) :
        t2 > 1-dt ? (t2 = (t2-1)/dt, t2*t2 + t2+t2 + 1) : 0;

out1 = square + blep1 - blep2;
```

### FM Synthesis
```
Param carrierFreq(440, min=20, max=2000);
Param modFreq(220, min=1, max=2000);
Param modIndex(2, min=0, max=10);
History carrierPhase(0);
History modPhase(0);

// Modulator
modPhase += modFreq / samplerate;
modPhase = wrap(modPhase, 0, 1);
modulator = sin(modPhase * twopi);

// Carrier with FM
carrierPhase += (carrierFreq + modulator * modIndex * modFreq) / samplerate;
carrierPhase = wrap(carrierPhase, 0, 1);

out1 = sin(carrierPhase * twopi);
```

### LFO with Multiple Shapes
```
Param rate(1, min=0.01, max=20);
Param shape(0, min=0, max=4);  // 0=sine, 1=tri, 2=saw, 3=square, 4=s&h
History phase(0);
History shValue(0);

phase += rate / samplerate;
phase = wrap(phase, 0, 1);

sine = sin(phase * twopi);
tri = 1 - 4 * abs(phase - 0.5);
saw = 2 * phase - 1;
square = phase < 0.5 ? 1 : -1;

// Sample & hold
shTrig = change(floor(phase * 4));
shValue = shTrig != 0 ? noise() : shValue;

// Select shape
out1 = shape < 0.5 ? sine :
       shape < 1.5 ? tri :
       shape < 2.5 ? saw :
       shape < 3.5 ? square : shValue;
```

---

## Envelopes

### ADSR Envelope
```
Param attack(10, min=1, max=5000);   // ms
Param decay(100, min=1, max=5000);   // ms
Param sustain(0.7, min=0, max=1);
Param release(200, min=1, max=5000); // ms
History env(0);
History stage(0);  // 0=idle, 1=attack, 2=decay, 3=sustain, 4=release
History lastGate(0);

gate = in1;

// Gate on
gateOn = gate > 0.5 && lastGate <= 0.5;
// Gate off
gateOff = gate <= 0.5 && lastGate > 0.5;
lastGate = gate;

stage = gateOn ? 1 : gateOff && stage < 4 ? 4 : stage;

// Convert ms to coefficient
attackCoef = 1 / (attack * samplerate / 1000);
decayCoef = 1 / (decay * samplerate / 1000);
releaseCoef = 1 / (release * samplerate / 1000);

// State machine
env = stage == 1 ? env + attackCoef :
      stage == 2 ? env - decayCoef * (env - sustain) :
      stage == 4 ? env - releaseCoef * env : env;

// Stage transitions
stage = stage == 1 && env >= 1 ? 2 : stage;
stage = stage == 2 && env <= sustain + 0.001 ? 3 : stage;
stage = stage == 4 && env <= 0.001 ? 0 : stage;

env = clamp(env, 0, 1);
out1 = env;
```

### Envelope Follower
```
Param attackMs(5, min=0.1, max=100);
Param releaseMs(50, min=1, max=1000);
History env(0);

attackCoef = exp(-1 / (attackMs * samplerate / 1000));
releaseCoef = exp(-1 / (releaseMs * samplerate / 1000));

inputAbs = abs(in1);
coef = inputAbs > env ? attackCoef : releaseCoef;
env = inputAbs + coef * (env - inputAbs);
out1 = env;
```

### AR Envelope (Attack-Release)
```
Param attack(10, min=1, max=1000);
Param release(100, min=1, max=2000);
History env(0);
History triggered(0);

trigger = in1 > 0.5;
triggered = trigger ? 1 : triggered;

attackRate = 1 / (attack * samplerate / 1000);
releaseRate = 1 / (release * samplerate / 1000);

env = triggered && env < 1 ? env + attackRate :
      !trigger ? env - releaseRate : env;

triggered = env <= 0 ? 0 : triggered;
env = clamp(env, 0, 1);
out1 = env;
```

---

## Granular Synthesis

### Simple Grain Player
```
// Plays grains from buffer
Param grainSize(50, min=10, max=500);     // ms
Param grainDensity(10, min=1, max=100);   // grains/sec
Param position(0.5, min=0, max=1);        // buffer position
Param positionRand(0.1, min=0, max=0.5);  // randomization
Param pitch(1, min=0.25, max=4);
History grainPhase(0);
History grainStart(0);
History grainActive(0);
History trigPhase(0);

bufferName = "grainbuf";  // External buffer reference
bufLen = dim(bufferName);

// Trigger new grains
trigPhase += grainDensity / samplerate;
newGrain = trigPhase >= 1;
trigPhase = newGrain ? 0 : trigPhase;

// Start new grain
randOffset = noise() * positionRand;
grainStart = newGrain ? (position + randOffset) * bufLen : grainStart;
grainPhase = newGrain ? 0 : grainPhase;
grainActive = newGrain ? 1 : grainActive;

// Grain playback
grainSamps = grainSize * samplerate / 1000;
grainPhase += pitch;
grainActive = grainPhase >= grainSamps ? 0 : grainActive;

// Read from buffer with envelope
readPos = grainStart + grainPhase;
readPos = wrap(readPos, 0, bufLen);
grainEnv = sin(pi * grainPhase / grainSamps);  // Hann window
sample = peek(bufferName, 0, readPos);

out1 = grainActive ? sample * grainEnv : 0;
```

### Polyphonic Grain Cloud (4 voices)
```
Param grainSize(80, min=20, max=500);
Param density(8, min=1, max=50);
Param position(0.5, min=0, max=1);
Param spread(0.2, min=0, max=1);
Param pitch(1, min=0.25, max=4);
Param pitchRand(0.1, min=0, max=1);

// 4 grain voices
History ph1(0); History ph2(0); History ph3(0); History ph4(0);
History st1(0); History st2(0); History st3(0); History st4(0);
History pt1(1); History pt2(1); History pt3(1); History pt4(1);
History trig(0);

bufLen = dim("grainbuf");
grainSamps = grainSize * samplerate / 1000;

// Trigger rotation
trig += density / samplerate;
trigNow = trig >= 1;
trig = trigNow ? 0 : trig;

// Voice counter for round-robin
History voice(0);
voice = trigNow ? (voice + 1) % 4 : voice;

// Calculate new grain params
newStart = (position + (noise() - 0.5) * spread) * bufLen;
newPitch = pitch * (1 + (noise() - 0.5) * pitchRand);

// Update voices on trigger
st1 = trigNow && voice == 0 ? newStart : st1;
st2 = trigNow && voice == 1 ? newStart : st2;
st3 = trigNow && voice == 2 ? newStart : st3;
st4 = trigNow && voice == 3 ? newStart : st4;

pt1 = trigNow && voice == 0 ? newPitch : pt1;
pt2 = trigNow && voice == 1 ? newPitch : pt2;
pt3 = trigNow && voice == 2 ? newPitch : pt3;
pt4 = trigNow && voice == 3 ? newPitch : pt4;

ph1 = trigNow && voice == 0 ? 0 : ph1;
ph2 = trigNow && voice == 1 ? 0 : ph2;
ph3 = trigNow && voice == 2 ? 0 : ph3;
ph4 = trigNow && voice == 3 ? 0 : ph4;

// Advance phases
ph1 += pt1; ph2 += pt2; ph3 += pt3; ph4 += pt4;

// Read and envelope each voice
env1 = ph1 < grainSamps ? sin(pi * ph1 / grainSamps) : 0;
env2 = ph2 < grainSamps ? sin(pi * ph2 / grainSamps) : 0;
env3 = ph3 < grainSamps ? sin(pi * ph3 / grainSamps) : 0;
env4 = ph4 < grainSamps ? sin(pi * ph4 / grainSamps) : 0;

s1 = peek("grainbuf", 0, wrap(st1 + ph1, 0, bufLen)) * env1;
s2 = peek("grainbuf", 0, wrap(st2 + ph2, 0, bufLen)) * env2;
s3 = peek("grainbuf", 0, wrap(st3 + ph3, 0, bufLen)) * env3;
s4 = peek("grainbuf", 0, wrap(st4 + ph4, 0, bufLen)) * env4;

out1 = (s1 + s2 + s3 + s4) * 0.5;
```
