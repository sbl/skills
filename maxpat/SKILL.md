---
name: maxpat
description: Expert Max/MSP patcher creation. Generate .maxpat JSON files with proper structure, connections, and layout. Knows all built-in Max, MSP, MC, and ABL objects. Covers modern paradigms from Max 8+ (MC multichannel, ramp-based signal-rate sequencing with subdiv~/shape~/phasor~, sample-accurate timing) and Max 9 (ABL Ableton DSP objects, V8 JavaScript, codebox objects, step sequencer objects). Prefers ramp-based signal-rate approach over message-rate metro/bang patterns. Use when creating Max/MSP patches, building audio signal chains, designing synthesizers/effects/sequencers in Max, or when the user mentions .maxpat, Max/MSP patching, or Max objects. Knows when to drop down into gen~ (delegates to the genexpr skill for gen~ code). For RNBO web export workflows, use the rnbo skill instead.
---

# Max/MSP Patcher Expert

Generate valid .maxpat files that open directly in Max/MSP. Save to `./patchers/`.

## Reference Files

- [Patcher Format](references/maxpat-format.md) — .maxpat JSON structure, boxes, connections, subpatchers, layout
- [Objects Inventory](references/objects-inventory.md) — Complete categorized list of all built-in objects (MSP, Max, MC, ABL)
- [Modern Paradigms](references/modern-paradigms.md) — Max 8+ MC multichannel, ABL objects, gen~ decision guide, V8 JavaScript, codebox, common patterns

## Workflow

1. Determine what the user wants to build (synth, effect, utility, sequencer, etc.)
2. Choose appropriate objects: standard MSP for fundamentals, ABL for production-quality sound, gen~ for custom sample-level DSP
3. Generate .maxpat JSON with proper structure, connections, and readable layout
4. Save to `./patchers/[name].maxpat`

## Patcher Generation Rules

- **Always include `ezdac~`** at the bottom of audio patches for quick testing
- **Use descriptive object IDs** (e.g., `"osc_carrier"`, `"filter_lp"`) not just `"obj-1"`
- **Layout top-to-bottom**: inputs at top, processing in middle, outputs at bottom
- **Add comments** to label major sections
- **Set sensible defaults**: frequency 440, amplitude 0.5, filter cutoff 1000, etc.
- **Include UI controls**: `flonum`, `slider`, `live.dial`, or `toggle` connected to parameters
- **Smooth parameters**: use `line~` or `slide~` between UI and audio to avoid clicks
- **Only use objects from the inventory** — consult [Objects Inventory](references/objects-inventory.md) when unsure

## Object Selection Guide

| Need | Use | Why |
|------|-----|-----|
| Basic oscillator | `cycle~`, `saw~`, `rect~`, `tri~` | Standard, well-documented |
| Varied timbres | `abl.dsp.meldosc~` | 24 oscillator types in one object |
| Filter | `svf~` (versatile) or `lores~` (simple LP) | Standard, multi-mode |
| Production filter | `abl.dsp.filther~` or `abl.dsp.meldfilter~` | Ableton quality |
| Reverb | `abl.dsp.darkhall~`, `shimmer~`, etc. | Far superior to DIY |
| Delay | `tapin~` / `tapout~` | Standard delay line |
| Distortion | `overdrive~` (simple) or `abl.dsp.saturator~` (quality) | Choose by need |
| Compression | `omx.comp~` (simple) or `abl.device.compressor~` (full) | Choose by need |
| Envelope | `adsr~` or `function` → `line~` | Standard |
| Polyphony | `poly~` with voice subpatcher | Standard approach |
| Unison/spread | MC objects (`mc.saw~`, `mc.mixdown~`) | Modern Max 8+ approach |
| Custom DSP | `gen~` (use genexpr skill) | Sample-level processing |
| LFO | `cycle~` at sub-audio rate, or `abl.dsp.modulator~` | Standard or ABL |
| Sample playback | `groove~` (looping) or `play~` (one-shot) | Depends on use case |
| Sequencing | `subdiv~` + `shape~` (ramp-based) or `techno~` | Signal-rate, sample-accurate |
| Musical timing | `phasor~` or `sync~` (BPM) as ramp clock | Prefer ramps over `metro` |
| Generative/stochastic | `mc.snowphasor~` + `mc.snowfall~` | Particle-based MC timing |

## Quick Reference: Patcher Skeleton

```json
{
  "patcher": {
    "fileversion": 1,
    "appversion": { "major": 8, "minor": 6, "revision": 0, "architecture": "x64", "modernui": 1 },
    "classnamespace": "box",
    "rect": [100, 100, 900, 700],
    "bglocked": 0, "openinpresentation": 0,
    "default_fontsize": 12.0, "default_fontface": 0, "default_fontname": "Arial",
    "gridonopen": 1, "gridsize": [15.0, 15.0], "gridsnaponopen": 1, "objectsnaponopen": 1,
    "statusbarvisible": 2, "toolbarvisible": 1,
    "lefttoolbarpinned": 0, "toptoolbarpinned": 0, "righttoolbarpinned": 0, "bottomtoolbarpinned": 0,
    "toolbars_unpinned_last_save": 0, "tallnewobj": 0, "boxanimatetime": 200,
    "enablehscroll": 1, "enablevscroll": 1, "devicewidth": 0.0,
    "description": "", "digest": "", "tags": "", "style": "",
    "subpatcher_template": "", "assistshowspatchername": 0,
    "boxes": [],
    "lines": []
  }
}
```

## Handling gen~

When the user needs custom sample-level DSP, create a `gen~` object in the patcher and note that the genexpr skill should be used for the internal code. For the patcher JSON, gen~ appears as:

```json
{
  "box": {
    "maxclass": "newobj",
    "text": "gen~",
    "id": "gen_processor",
    "numinlets": 1,
    "numoutlets": 1,
    "outlettype": ["signal"],
    "patching_rect": [100, 200, 40, 22],
    "patcher": {
      "fileversion": 1,
      "appversion": { "major": 8, "minor": 6, "revision": 0, "architecture": "x64", "modernui": 1 },
      "classnamespace": "dsp.gen",
      "rect": [0, 0, 600, 400],
      "boxes": [],
      "lines": []
    }
  }
}
```

## Official Documentation

- Max/MSP docs: https://docs.cycling74.com/
- MSP objects: https://docs.cycling74.com/max8/vignettes/msp_functional
- ABL objects: https://docs.cycling74.com/userguide/abl/
