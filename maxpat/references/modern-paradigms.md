# Modern Max/MSP Paradigms (Max 8+)

## MC Multichannel (Max 8)

MC is the multichannel audio system introduced in Max 8. It enables processing many audio channels through a single patch cord.

### When to Use MC

- **Unison/detuned sounds**: Spread oscillators across channels with slightly different frequencies
- **Polyphony without poly~**: Run N voices as N channels, mix down at output
- **Spatial audio**: Route channels to multi-speaker setups
- **Parallel processing**: Apply same effect chain to many signals simultaneously
- **Additive synthesis**: Each channel = one partial

### MC Patterns

**Unison oscillator (8 detuned saws):**
```
mc.saw~ @chans 8 → mc.mixdown~ 2
```
Use `mc.assign~` to spread frequency slightly across channels.

**Multichannel filter sweep:**
```
mc.cycle~ @chans 16 → mc.svf~ @chans 16 → mc.mixdown~ 2
```

### Key MC Attributes

- `@chans N` — set number of channels
- `@automanage 1` — auto-adjust channel count from input
- Most MSP objects have MC equivalents via `mc.` prefix

---

## Ramp-Based Signal-Rate Music (Max 8.3+)

The core modern Max paradigm: **use phasor~ ramps instead of bangs** for musical timing. Ramps are continuous 0-1 signals that provide sample-accurate timing without scheduler jitter. This replaces the traditional `metro` → `bang` → message approach.

### The Paradigm

Instead of:
```
metro 500 → counter → select → noteout  (message-rate, ~ms precision)
```

Use:
```
phasor~ 2 → subdiv~ 4 → shape~ → cycle~  (signal-rate, sample-accurate)
```

### Ramp Generation

- **`phasor~`** — fundamental: ramps 0→1 at frequency in Hz
- **`sync~`** — ramps 0→1 at BPM (musical tempo)
- **`ramp~`** — flexible ramp with trigger modes
- **`mc.snowphasor~`** — probabilistic/stochastic ramps (granular, particle-like)

### Ramp Subdivision and Timing

- **`subdiv~`** — divide ramp into N steps with probability and duration patterns (e.g., `subdiv~ 4` turns one bar into 4 beats; patterns like `1 1 2` create uneven durations)
- **`rate~`** — time-scale ramps (0.5 = double speed, 2.0 = half speed)
- **`swing~`** — apply swing feel to ramp subdivisions

### Ramp Shaping (Envelopes from Ramps)

- **`shape~`** — ramp-driven envelope generator (like `line~` driven by ramp instead of messages). Connect a `function` object to define the curve shape
- **`updown~`** — trapezoidal envelope where attack/release are in ms (independent of ramp speed)
- **`twist~`** — apply nonlinear curve to ramps (easing functions)

### Ramp-Driven Sequencing

- **`techno~`** — step sequencer: set pitch/amp/attack/decay/curve per step, outputs signals directly to oscillators
- **`what~`** — emit impulses when ramp crosses specified thresholds (give it a list of 0.-1. values)
- **`where~`** — detect position within a ramp
- **`stash~`** — store/recall signal values
- **`table~`** — signal-rate table lookup for ramp remapping
- **`phasegroove~`** — ramp-driven buffer playback

### Ramp-to-Event Conversion

When you need to trigger message-rate events from ramps:
```
rate~ → delta~ → <~ 0 → edge~ → bang
```
`delta~` detects ramp resets (negative spike), `<~ 0` creates a logical trigger, `edge~` converts to bang.

### MC Signal Music

For multichannel generative music:
- **`mc.pattern~`** — multichannel pattern sequencer
- **`mc.chord~`** — multichannel chord generator
- **`mc.generate~`** — signal generation with easing/randomness
- **`mc.snowphasor~`** + **`mc.snowfall~`** — particle-based probabilistic ramp engine
- **`mc.midiplayer~`** — convert MC signals to sample-accurate MIDI notes for VST/AU

### When to Use Ramps vs Messages

**Use ramps when:**
- Building sequencers, drum machines, generative music
- Timing precision matters (no scheduler jitter)
- Driving envelopes and modulation from a shared clock
- Working with MC multichannel patterns

**Use messages when:**
- Responding to user interaction (UI, MIDI input)
- One-shot events (file loading, preset recall)
- Non-time-critical operations

---

## Sample-Accurate Timing (Max 8+)

Max 8.3+ also improved message-to-signal conversion. Previously, events arriving mid-vector were collapsed to the vector boundary (up to ~5.8ms imprecision at 256 samples/44.1kHz).

### Required Settings

Both must be enabled in Audio Status:
- **Overdrive** — scheduler runs at high priority
- **Scheduler in Audio Interrupt** — scheduler runs in lockstep with audio processing

### Events-to-Signal (sample-accurate)

- **`sig~`** — number placed at precise sample offset within vector
- **`click~`** — single-sample impulse at precise timing
- **`line~`** / **`curve~`** — ramp/envelope starts at precise sample
- **`vst~`** — receives MIDI events sample-accurately

### Signal-to-Events (sample-accurate)

- **`snapshot~`** — samples signal value at precise intervals
- **`edge~`** — detects signal transitions, outputs bang
- **`spike~`** — measures inter-event time at sample resolution
- **`peakamp~`** — records peak amplitudes with timed events

### Best Practices

- Prefer ramp-based signal-rate approach (above) over message-rate with sample-accurate conversion
- Use `line~` or `slide~` for parameter smoothing (avoids clicks)
- Use `sig~` to convert message-rate control to signal when connecting to `~` objects

---

## ABL Objects (Max 9)

The Ableton DSP library brings production-quality components from Live into Max.

### When to Use ABL vs Standard MSP

**Use ABL when:**
- You want production-quality sound out of the box
- You need a complete device (compressor, reverb, synth) quickly
- You want the exact sound of Ableton Live devices
- Building instruments where Meld oscillator types give you variety
- You need high-quality reverbs (darkhall, shimmer, prism, quartz, tides)

**Use standard MSP when:**
- You need fine-grained control over every parameter
- You're building something ABL doesn't cover
- You need to understand/teach the DSP fundamentals
- You want minimal CPU overhead for simple operations
- You need deterministic, well-documented behavior

### ABL Patterns

**Quick polyphonic synth:**
Use `abl.device.drift~` — it's a complete polyphonic synth with modulation matrix.

**High-quality reverb:**
Choose from 5 algorithms: `abl.dsp.darkhall~`, `abl.dsp.prism~`, `abl.dsp.quartz~`, `abl.dsp.shimmer~`, `abl.dsp.tides~`

**Oscillator variety:**
Use `abl.dsp.meldosc~` with its 24 oscillator types, controlled by frequency + 2 macros. Or use individual oscillator objects for more control.

**Modulation sources:**
`abl.dsp.modulator~`, `abl.dsp.euclid~`, `abl.dsp.wander~`, `abl.dsp.pulsate~` — all output signals suitable for modulating parameters.

### ABL Object Conventions

- All abl objects end with `~` (they're signal-rate)
- `abl.device.*` objects are full devices with many parameters set via attributes
- `abl.dsp.*` objects are building blocks with fewer, more focused parameters
- Parameters are set via `@attribute value` in the object text or via messages
- ABL objects support `@ins` attribute for assigning float-type attributes to inlets (Max 9.1+)

---

## Step Objects (Max 9.1)

New signal-rate step sequencing objects for sample-accurate rhythmic patterns.

| Object | Description |
|--------|-------------|
| `stepfun~` | Step function generator |
| `stepdiv~` | Step divider |
| `stepcounter~` | Signal-rate step counter |

---

## When to Use gen~

gen~ provides sample-level DSP processing with a specialized language (GenExpr). Use the existing **genexpr** skill for gen~ code.

### Use gen~ When

- **Single-sample feedback** — MSP minimum feedback is one vector (64+ samples). gen~ uses `history` for 1-sample delay, enabling proper IIR filters, waveguide synthesis, recursive algorithms
- **Custom DSP algorithms** — waveshaping, novel filter topologies, granular engines, unusual modulation
- **Performance** — gen~ JIT-compiles the entire patch to native code, optimizing across operators. Much lower CPU for complex interconnected math
- **Implementing DSP papers** — direct translation from equations to code
- **Exportability** — gen~ patches export via RNBO to C++, web audio, hardware

### Use MSP Objects When

- Standard objects cover the need (cycle~, svf~, etc.)
- You need dynamic reconfiguration (MSP objects can be created/reconnected at runtime; gen~ is compiled and fixed)
- You need buffer~/groove~/play~ (gen~ buffer access works differently via `peek`/`poke`/`data`)
- Building a prototype quickly
- Mixing messages and signals freely (gen~ is pure signal-rate)

### Key Differences

| Aspect | MSP | gen~ |
|--------|-----|------|
| Processing | Signal vector (64+ samples) | Single sample |
| Min feedback delay | One vector | One sample (`history`) |
| Compilation | Each object separate | Entire patch JIT-compiled together |
| Naming | `cycle~` (with tilde) | `cycle` (no tilde) |
| State | Internal per-object | Explicit via `history`, `data`, `buffer` |
| Messages | Supported | Not available — everything is signal |

### gen~ in Patcher JSON

gen~ appears as a `newobj` with `classnamespace: "dsp.gen"` in its subpatcher. For codebox-based gen~, use `gen~` with an embedded `codebox` object. See the genexpr skill for the code syntax.

---

## JavaScript V8 (Max 9)

Max 9 replaced the legacy JS engine with V8, supporting modern ES6+ features.

### When to Use V8 vs Patching

- **Data transformation**: Complex list/dict processing
- **Algorithmic composition**: Generative MIDI patterns, scales, chord logic
- **State machines**: Complex sequencer logic
- **File I/O**: Reading/writing data files
- **NOT for audio**: V8 runs in the UI thread, not audio thread. Use gen~ for DSP.

### v8 vs js

- `v8` — new V8 engine (Max 9+, recommended)
- `js` — legacy SpiderMonkey engine (Max 7/8 compat)
- `v8.codebox` — inline V8 code in patcher

---

## Codebox Objects (Max 9)

Codebox objects allow inline code editing without opening subpatchers.

| Object | Use For |
|--------|---------|
| `v8.codebox` | JavaScript data processing |
| `node.codebox` | Node.js / async / file I/O |
| `gen.codebox~` | Sample-rate DSP (GenExpr) |
| `dict.codebox` | Dictionary transformations |
| `coll.codebox` | Collection processing |

These are useful for small algorithms that don't warrant a full subpatcher.

---

## Common Patcher Patterns

### Synth Voice (for poly~)

```
notein → poly~ my_synth @voices 8
```

Inside poly~:
```
in 1 (note) → mtof → cycle~ → *~ → adsr~ → out~ 1
in 2 (velocity) → / 127. → *~ (apply velocity)
thispoly~ (voice management)
```

### Effect Chain

```
adc~ → [effect 1] → [effect 2] → dac~
```

With dry/wet:
```
adc~ → *~ (dry) ─────────────────→ +~ → dac~
     → [effect] → *~ (wet) ──────↗
```

### LFO Modulation

```
cycle~ 0.5 (LFO rate) → *~ 100 (depth) → +~ 440 (center) → cycle~ (carrier)
```

### Envelope Following

```
adc~ → abs~ → slide~ 1 100 (fast attack, slow release) → snapshot~ 50
```
