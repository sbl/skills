# Max/MSP Object Inventory

Comprehensive reference of built-in objects. Only use objects listed here.

## Table of Contents

- [MSP Audio Objects](#msp-audio-objects)
- [Max Control Objects](#max-control-objects)
- [MC Multichannel Objects](#mc-multichannel-objects)
- [ABL Objects (Max 9)](#abl-objects-max-9)
- [Max 9 New Objects](#max-9-new-objects)

---

## MSP Audio Objects

### Synthesis

| Object | Description |
|--------|-------------|
| `cycle~` | Wavetable oscillator (default sine) |
| `phasor~` | Sawtooth ramp generator (0-1) |
| `saw~` | Band-limited sawtooth |
| `rect~` | Band-limited rectangle/pulse |
| `tri~` | Band-limited triangle |
| `cos~` | Cosine waveshaper |
| `noise~` | White noise generator |
| `pink~` | Pink noise generator |
| `click~` | Single-sample impulse |
| `train~` | Pulse train generator |
| `ioscbank~` | Incremental oscillator bank |
| `oscbank~` | Oscillator bank |
| `kink~` | Phasor waveshaper (adds harmonics) |

### Filters

| Object | Description |
|--------|-------------|
| `svf~` | State-variable filter (LP/HP/BP/notch simultaneous) |
| `biquad~` | Biquad filter (coefficients) |
| `lores~` | Resonant lowpass filter |
| `onepole~` | One-pole lowpass/highpass |
| `reson~` | Resonant bandpass filter |
| `allpass~` | Allpass filter |
| `cascade~` | Cascaded biquad filter |
| `comb~` | Comb filter |
| `cross~` | Crossover filter |
| `fffb~` | Fast fixed filter bank |
| `hilbert~` | Hilbert transform (90-degree phase shift) |
| `teeth~` | Comb filter bank |
| `filtercoeff~` | Generate biquad coefficients |
| `filtergraph~` | Filter design UI |
| `phaseshift~` | Phase shift filter |
| `slide~` | Exponential smoothing (slew limiter) |
| `rampsmooth~` | Linear smoothing (ramp) |
| `buffir~` | FIR filter using buffer |

### Delays

| Object | Description |
|--------|-------------|
| `tapin~` | Delay line write |
| `tapout~` | Delay line read (use with tapin~) |
| `delay~` | Simple signal delay (samples) |

### Dynamics

| Object | Description |
|--------|-------------|
| `omx.comp~` | Compressor |
| `omx.peaklim~` | Peak limiter |
| `omx.4band~` | 4-band EQ |
| `omx.5band~` | 5-band EQ |
| `overdrive~` | Soft saturation/overdrive |

### Envelopes and Functions

| Object | Description |
|--------|-------------|
| `adsr~` | ADSR envelope generator |
| `line~` | Signal-rate line segment generator |
| `curve~` | Exponential curve generator |
| `trapezoid~` | Trapezoidal envelope shaper |
| `triangle~` | Triangle function generator |
| `sig~` | Convert message to constant signal |

### Math (Signal Rate)

| Object | Description |
|--------|-------------|
| `+~` `*~` `-~` `/~` `!-~` `!/~` | Arithmetic |
| `%~` | Modulo |
| `abs~` | Absolute value |
| `clip~` | Clamp to range |
| `pong~` | Wrap/fold/clamp |
| `round~` `trunc~` | Rounding |
| `pow~` `sqrt~` `log~` | Power/root/log |
| `sin~` `cos~` `tan~` `asin~` `acos~` `atan~` `atan2~` | Trig |
| `sinh~` `cosh~` `tanh~` | Hyperbolic |
| `sinx~` `cosx~` `tanx~` | -1 to 1 input range trig |
| `maximum~` `minimum~` | Signal max/min |
| `>~` `<~` `>=~` `<=~` `==~` `!=~` | Comparison |
| `bitand~` `bitor~` `bitxor~` `bitnot~` `bitshift~` | Bitwise |
| `+=~` | Accumulator |

### Conversion

| Object | Description |
|--------|-------------|
| `mtof~` | MIDI note to frequency |
| `ftom~` | Frequency to MIDI note |
| `atodb~` | Amplitude to dB |
| `dbtoa~` | dB to amplitude |
| `mstosamps~` | Milliseconds to samples |
| `sampstoms~` | Samples to milliseconds |
| `cartopol~` | Cartesian to polar |
| `poltocar~` | Polar to Cartesian |
| `freqshift~` | Frequency shift |

### Sampling and Buffers

| Object | Description |
|--------|-------------|
| `buffer~` | Audio buffer (memory) |
| `play~` | Play buffer contents |
| `groove~` | Variable-rate buffer playback with loop |
| `record~` | Record into buffer |
| `wave~` | Wavetable lookup from buffer |
| `2d.wave~` | 2D wavetable lookup |
| `index~` | Sample-accurate buffer read |
| `peek~` | Read/write buffer by sample number |
| `poke~` | Write single sample to buffer |
| `lookup~` | Transfer function from buffer |
| `info~` | Buffer information (length, channels, sr) |
| `polybuffer~` | Manage multiple buffers |
| `stutter~` | Buffer stutter effect |
| `zigzag~` | Arbitrary buffer playback |
| `chucker~` | Audio chunk capture |
| `rate~` | Playback rate for groove~ |
| `normalize~` | Normalize buffer contents |

### FFT

| Object | Description |
|--------|-------------|
| `pfft~` | FFT processing subpatcher host |
| `fft~` / `ifft~` | Forward/inverse FFT |
| `fftin~` / `fftout~` | FFT I/O (inside pfft~) |
| `fftinfo~` | FFT size info |
| `gizmo~` | FFT pitch shifter |
| `frameaccum~` | Accumulate FFT frames |
| `framedelta~` | Frame-to-frame difference |
| `fbinshift~` | Shift FFT bins |
| `vectral~` | Spectral processing |
| `cartopol~` / `poltocar~` | Coordinate conversion |
| `phasewrap~` | Wrap phase values |

### Routing and I/O

| Object | Description |
|--------|-------------|
| `adc~` / `dac~` | Audio device input/output |
| `ezadc~` / `ezdac~` | Audio I/O with toggle |
| `in~` / `out~` | Subpatcher signal I/O |
| `send~` / `receive~` | Wireless signal routing |
| `gate~` | Signal gate/switch |
| `selector~` | Signal selector |
| `matrix~` | Signal routing matrix |
| `pass~` | Pass signal through |
| `mute~` | Mute signal subpatcher |

### Analysis

| Object | Description |
|--------|-------------|
| `snapshot~` | Sample signal value to message |
| `peakamp~` | Peak amplitude measurement |
| `avg~` / `average~` | Signal averaging |
| `meter~` / `levelmeter~` | Level meter UI |
| `scope~` / `spectroscope~` | Signal visualization |
| `number~` | Signal value display |
| `fzero~` | Fundamental frequency detection |
| `zerox~` | Zero-crossing counter |
| `change~` | Report signal changes |
| `edge~` | Detect signal transitions |
| `count~` | Sample counter |
| `delta~` | Sample-to-sample difference |
| `thresh~` | Signal threshold detection |
| `spike~` | Signal spike detection |
| `minmax~` | Track min/max values |

### Signal-Rate Music (Ramp-Based Sequencing, Max 8.3+)

Core paradigm: use `phasor~` ramps (0-1) as timing source instead of message-rate bangs. All objects below operate at signal rate for sample-accurate musical timing.

**Timing and Subdivision:**

| Object | Description |
|--------|-------------|
| `sync~` | BPM-based ramp generator (like phasor~ but in BPM) |
| `ramp~` | Flexible ramp generator with trigger modes |
| `subdiv~` | Subdivide ramp into steps with probability and duration patterns |
| `swing~` | Apply swing to ramp signals |
| `rate~` | Time-scale ramps (2.0 = double duration, 0.5 = half) |

**Ramp Shaping:**

| Object | Description |
|--------|-------------|
| `shape~` | Remap ramp through function curves (ramp-driven line~) |
| `twist~` | Apply nonlinear curve to ramps/envelopes |
| `updown~` | Trapezoidal envelope with ms-based attack/release |

**Pattern and Sequencing:**

| Object | Description |
|--------|-------------|
| `techno~` | Sample-accurate step sequencer (pitch/amp/envelope per step) |
| `seq~` | Arbitrary event sequencer with signal-driven timestamps |
| `what~` | Output impulses when ramp passes specified threshold values |
| `where~` | Detect position within a ramp |
| `stash~` | Store and recall signal values |
| `table~` | Signal-rate table lookup (remap values, random from ramps) |
| `phasegroove~` | Ramp-driven buffer playback |

**Ramp Event Detection Chain:**
`rate~` → `delta~` → `<~ 0` → `edge~` → bang (detects ramp resets as events)

**MC Signal Music Objects:**

| Object | Description |
|--------|-------------|
| `mc.pattern~` | Multichannel pattern sequencer |
| `mc.chord~` | Multichannel chord generator |
| `mc.snowphasor~` | Probabilistic ramp generator (particle-inspired) |
| `mc.snowfall~` | Particle engine (works with mc.snowphasor~) |
| `mc.generate~` | Multichannel signal generation with easing/randomness |
| `mc.midiplayer~` | Convert MC signals to sample-accurate MIDI notes |
| `mc.apply~` | Apply function across MC channels |

### Polyphony

| Object | Description |
|--------|-------------|
| `poly~` | Polyphonic voice manager |
| `thispoly~` | Query/control poly~ voice |
| `in` / `out` / `in~` / `out~` | Subpatcher I/O |

### System

| Object | Description |
|--------|-------------|
| `dspstate~` | DSP engine state info |
| `dsptime~` | Current DSP time |
| `adstatus` | Audio driver status/control |
| `gen~` | Gen code subpatcher (sample-level DSP) |
| `sah~` | Sample and hold |
| `downsamp~` | Signal downsampling |
| `degrade~` | Bit/sample rate reduction |

### Plug-in

| Object | Description |
|--------|-------------|
| `plugin~` / `plugout~` | Audio plug-in I/O |
| `vst~` | Host VST/AU plug-ins |
| `plugphasor~` | Plug-in sync phasor |
| `plugsend~` / `plugreceive~` | Plug-in bus send/receive |
| `plugsync~` | Plug-in transport sync |

### File I/O

| Object | Description |
|--------|-------------|
| `sfplay~` | Sound file playback |
| `sfrecord~` | Sound file recording |
| `sflist~` | Sound file cue list |
| `sfinfo~` | Sound file info |

---

## Max Control Objects

### Math and Logic

| Object | Description |
|--------|-------------|
| `+` `-` `*` `/` `%` | Arithmetic |
| `expr` | Expression evaluator |
| `abs` `sqrt` `pow` | Math functions |
| `sin` `cos` `tan` `atan` `atan2` | Trigonometry |
| `scale` | Map value from one range to another |
| `clip` | Clamp value to range |
| `split` | Route by numeric range |
| `minimum` `maximum` | Min/max of two values |
| `random` | Random integer |
| `drunk` | Random walk |
| `decide` | Random 0 or 1 |
| `urn` | Random without repeat |
| `counter` | Count events |
| `accum` | Accumulate values |

### Timing

| Object | Description |
|--------|-------------|
| `metro` | Periodic bang |
| `qmetro` | Queue-safe metro |
| `delay` | Delayed bang |
| `pipe` | Delayed message |
| `timer` | Measure time between bangs |
| `clocker` | Continuous time reporting |
| `tempo` | Tempo-relative timing |
| `transport` | Global transport control |
| `translate` | Convert time units |
| `when` | Report transport position |
| `timepoint` | Bang at transport position |
| `schedule` | Schedule messages (Max 9) |

### Routing

| Object | Description |
|--------|-------------|
| `trigger` (`t`) | Multi-outlet trigger (right-to-left) |
| `route` | Route by first element |
| `routepass` | Route with passthrough |
| `select` (`sel`) | Match and route values |
| `gate` | Open/close message gate |
| `gswitch` / `gswitch2` | Graphical switch |
| `switch` | Message switch |
| `if` | Conditional routing |
| `send` / `receive` (`s` / `r`) | Wireless message passing |
| `forward` | Dynamic send |
| `patcherargs` | Get patcher arguments |
| `loadbang` | Bang on patcher load |
| `closebang` | Bang on patcher close |
| `deferlow` | Defer to low priority |
| `defer` | Defer message |
| `buddy` | Synchronize inputs |
| `thresh` | Group rapid messages |

### Data

| Object | Description |
|--------|-------------|
| `pack` / `unpack` | Bundle/separate values |
| `append` / `prepend` | Add to message |
| `zl` | List operations (len, rev, sort, slice, group, etc.) |
| `iter` | Iterate through list |
| `join` | Join lists |
| `spray` | Distribute list elements |
| `funnel` | Collect to list |
| `coll` | Named collection (key-value store) |
| `dict` | Dictionary (JSON-like data) |
| `table` | Numeric lookup table |
| `value` (`v`) | Shared named value |
| `pattr` | Parameter storage |
| `preset` | Preset management |
| `autopattr` | Auto-bind parameters |
| `pattrstorage` | Parameter storage manager |
| `bag` | Store set of numbers |
| `borax` | Note analysis |
| `regexp` | Regular expressions |

### MIDI

| Object | Description |
|--------|-------------|
| `notein` / `noteout` | MIDI note I/O |
| `ctlin` / `ctlout` | MIDI CC I/O |
| `bendin` / `bendout` | Pitch bend I/O |
| `pgmin` / `pgmout` | Program change I/O |
| `midiin` / `midiout` | Raw MIDI I/O |
| `midiparse` / `midiformat` | MIDI byte parsing/formatting |
| `poly` | MIDI voice allocation |
| `makenote` | Generate note-on + scheduled note-off |
| `stripnote` | Filter out note-offs |
| `sustain` | MIDI sustain pedal behavior |
| `flush` | Flush all pending notes |
| `ddg.mono` | Monophonic MIDI processor |
| `kslider` | Keyboard UI |
| `midiinfo` | MIDI device info |

### UI Objects

| Object | Description |
|--------|-------------|
| `number` / `flonum` | Integer/float number box |
| `slider` / `rslider` | Slider / range slider |
| `dial` | Rotary dial |
| `multislider` | Array of sliders |
| `toggle` | On/off toggle |
| `button` | Bang button |
| `message` | Message box |
| `comment` | Text comment |
| `umenu` | Pop-up menu |
| `textedit` | Text editor |
| `panel` | Background panel |
| `tab` | Tab selector |
| `radiogroup` | Radio button group |
| `matrixctrl` | Matrix control grid |
| `nslider` | Note display staff |
| `swatch` | Color picker |
| `pictslider` | Picture slider |
| `live.dial` | Live-style dial |
| `live.slider` | Live-style slider |
| `live.toggle` | Live-style toggle |
| `live.button` | Live-style button |
| `live.numbox` | Live-style number box |
| `live.menu` | Live-style menu |
| `live.text` | Live-style text button |
| `live.tab` | Live-style tab |

### Subpatchers and Abstractions

| Object | Description |
|--------|-------------|
| `p` | Subpatcher |
| `bpatcher` | Embedded visible subpatcher |
| `poly~` | Polyphonic subpatcher |
| `patcher` | Create patcher dynamically |
| `thispatcher` | Control containing patcher |
| `js` | JavaScript (legacy) |
| `v8` | JavaScript V8 engine (Max 9) |

---

## MC Multichannel Objects

MC (multichannel) wraps most MSP objects for parallel multichannel processing. Prefix any MSP object with `mc.` to get its multichannel version.

### MC-Specific Objects

| Object | Description |
|--------|-------------|
| `mc.pack~` | Pack mono signals into multichannel |
| `mc.unpack~` | Unpack multichannel to mono signals |
| `mc.mixdown~` | Mix all channels to mono/stereo |
| `mc.stereo~` | Mix multichannel down to stereo |
| `mc.separate~` | Separate specific channels |
| `mc.combine~` | Combine multichannel signals |
| `mc.dup~` | Duplicate single channel to N copies |
| `mc.op~` | Perform operations across channels (sum, mean) |
| `mc.interleave~` | Reorganize channel ordering |
| `mc.deinterleave~` | Reverse interleaving |
| `mc.target` | Set target channel for messages |
| `mc.targetlist` | Set multiple target channels |
| `mc.assign~` | Assign parameter values across channels |
| `mc.sig~` | Multichannel constant signal |
| `mc.list~` | Create multichannel signal from list |
| `mc.index~` | Index into multichannel signals |
| `mc.matrix~` | Multichannel routing matrix |
| `mc.voiceallocator~` | Voice allocation for polyphonic MC |
| `mc.send~` / `mc.receive~` | Multichannel wireless routing |

### Common MC-Wrapped Objects

Most MSP objects have MC equivalents via `mc.` prefix: `mc.cycle~`, `mc.svf~`, `mc.lores~`, `mc.biquad~`, `mc.phasor~`, `mc.noise~`, `mc.tapin~`, `mc.tapout~`, `mc.groove~`, `mc.play~`, `mc.adsr~`, `mc.line~`, `mc.gain~`, `mc.gen~`, `mc.vst~`, etc.

The `mcs.` prefix variant gives each channel independent parameter control (vs `mc.` where parameters are shared).

Key attributes for MC objects:
- `@chans N` — number of channels
- `@automanage 1` — auto-manage channel count
- `@op` — operation mode for parameter distribution

---

## ABL Objects (Max 9)

Ableton DSP objects from Live's internal audio engine. High-quality, production-ready components.

### abl.device Objects (Full Devices)

Complete Ableton Live devices as patchable Max objects.

| Object | Description |
|--------|-------------|
| `abl.device.drift~` | Polyphonic analog-style synthesizer |
| `abl.device.roar~` | 3-stage saturation/distortion effect |
| `abl.device.compressor~` | Compressor |
| `abl.device.limiter~` | Limiter |
| `abl.device.reverb~` | Plate reverb |
| `abl.device.delay~` | Stereo delay |
| `abl.device.echo~` | Modulation delay effect |
| `abl.device.spectralresonator~` | Spectral resonator |
| `abl.device.spectraltime~` | Spectral time-stretching |
| `abl.device.redux~` | Downsampling and bit reduction |
| `abl.device.channeleq~` | Channel EQ |
| `abl.device.utility~` | Utility (gain, pan, phase, mono) |
| `abl.device.autofilter~` | Auto-filter (9.1) |
| `abl.device.drumbuss~` | Drum bus processing (9.1) |
| `abl.device.drumsampler~` | Drum sampler (9.1) |

### abl.dsp Oscillators

| Object | Description |
|--------|-------------|
| `abl.dsp.meldosc~` | Meta-oscillator — 24 types with macro control (9.1) |
| `abl.dsp.basicshapes~` | Classic waveform morphing (saw/square/tri/sine) |
| `abl.dsp.bubble~` | Synthesized bubble |
| `abl.dsp.chip~` | Chiptune oscillator |
| `abl.dsp.crackle~` | Crackle/noise oscillator |
| `abl.dsp.extratone~` | Granular-esque tonal |
| `abl.dsp.rain~` | Rain synthesis |
| `abl.dsp.subosc~` | Sub oscillator with waveform morphing |
| `abl.dsp.swarm~` | Swarm of detuned oscillators |
| `abl.dsp.tarp~` | Tarp oscillator |
| `abl.dsp.simplefm~` | Two-operator FM |
| `abl.dsp.harmonicfm~` | Harmonic FM (ratio-locked) |
| `abl.dsp.fmbass~` | Three-operator FM bass |
| `abl.dsp.foldfm~` | Wavefolding FM |

`abl.dsp.meldosc~` types (set via `@type` 0-23): Basic Shapes, Bitgrunge, Bubble, Chip, Crackle, Dual Basic Shapes, Extratone, Filtered Noise, Fold FM, Harmonic FM, Noise Loop, Noisy Shapes, Rain, Shepard's Pi, Simple FM, Square 5th, Square Sync, FM Bass (Squelch), Sub, Swarm Saw, Swarm Sine, Swarm Square, Swarm Triangle, Tarp.

### abl.dsp Reverbs

| Object | Description |
|--------|-------------|
| `abl.dsp.darkhall~` | Dark hall reverb |
| `abl.dsp.prism~` | Prism reverb (from Hybrid Reverb) |
| `abl.dsp.quartz~` | Quartz reverb (from Hybrid Reverb) |
| `abl.dsp.shimmer~` | Shimmer reverb (from Hybrid Reverb) |
| `abl.dsp.tides~` | Modulating algorithmic reverb (from Hybrid Reverb) |

### abl.dsp Distortion and Saturation

| Object | Description |
|--------|-------------|
| `abl.dsp.saturator~` | Waveshaping saturator |
| `abl.dsp.waveshaper~` | Waveshaper |
| `abl.dsp.distortion~` | Guitar pedal distortion |
| `abl.dsp.fuzz~` | Fuzz guitar pedal |
| `abl.dsp.overdrive~` | Overdrive guitar pedal |
| `abl.dsp.filther~` | Filter with diode clipper + soft saturation |

### abl.dsp Modulation Effects

| Object | Description |
|--------|-------------|
| `abl.dsp.chorus~` | Chorus |
| `abl.dsp.ensemble~` | Ensemble effect |
| `abl.dsp.flanger~` | Stereo flanger |
| `abl.dsp.phaser~` | Phaser |
| `abl.dsp.vibrato~` | Stereo vibrato |
| `abl.dsp.doubler~` | Doubler effect |
| `abl.dsp.ringmod~` | Stereo ring modulator |
| `abl.dsp.pitchshifter~` | Pitch shifter |

### abl.dsp Filters

| Object | Description |
|--------|-------------|
| `abl.dsp.meldfilter~` | Meta-filter (9.1) |

### abl.dsp Modulators

| Object | Description |
|--------|-------------|
| `abl.dsp.ramp~` | Ramp generator |
| `abl.dsp.modulator~` | Modulation signal generator |
| `abl.dsp.alternate~` | Alternating modulator |
| `abl.dsp.euclid~` | Euclidean rhythm ramp generator |
| `abl.dsp.pulsate~` | Random pulse emitter |
| `abl.dsp.wander~` | Wandering modulator |
| `abl.dsp.stereolfo~` | Stereo LFO (9.1) |
| `abl.dsp.transform~` | Signal transformer (9.1) |

### abl.dsp Analysis

| Object | Description |
|--------|-------------|
| `abl.dsp.pitchestimator~` | Pitch detection (9.1) |

---

## Max 9 New Objects

### Signal-Rate Timing (Max 9.1)

| Object | Description |
|--------|-------------|
| `stepfun~` | Step function generator |
| `stepdiv~` | Step divider |
| `stepcounter~` | Signal-rate step counter |
| `stepcounter` | Message-rate step counter |

### Array Operations (Max 9)

| Object | Description |
|--------|-------------|
| `array.expr` | Array expression evaluation |
| `array.fill` | Fill array with values |
| `array.random` | Random array generation |
| `array.min` / `array.max` | Array min/max |
| `array.mean` / `array.median` / `array.mode` | Array statistics |
| `array.stddev` | Standard deviation |
| `array.deserialize` / `array.replace` | Array manipulation (9.1) |

### Codebox Objects (Max 9)

| Object | Description |
|--------|-------------|
| `v8.codebox` | JavaScript V8 inline code |
| `node.codebox` | Node.js inline code |
| `dict.codebox` | Dictionary processing code |
| `coll.codebox` | Collection processing code |
| `gen.codebox` / `gen.codebox~` | Gen code inline |
| `text.codebox` | Text processing code (9.1) |
| `osc.codebox` | OSC processing code |

### Other Max 9 Objects

| Object | Description |
|--------|-------------|
| `loudness~` | EBU R 128 loudness measurement |
| `jweb~` | Web browser with audio capture |
| `v8` | JavaScript V8 engine |
| `hid` | Human Interface Device input |
| `schedule` | Message scheduling |
| `param` (poly~) | Named parameters in poly~ |
| `param.osc` | OSC parameter binding |
