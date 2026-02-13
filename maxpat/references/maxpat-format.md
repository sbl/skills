# Max Patcher JSON Format (.maxpat)

Reference for generating valid Max patcher files that open directly in Max/MSP.

## Top-Level Structure

```json
{
  "patcher": {
    "fileversion": 1,
    "appversion": {
      "major": 8,
      "minor": 6,
      "revision": 0,
      "architecture": "x64",
      "modernui": 1
    },
    "classnamespace": "box",
    "rect": [100, 100, 900, 700],
    "bglocked": 0,
    "openinpresentation": 0,
    "default_fontsize": 12.0,
    "default_fontface": 0,
    "default_fontname": "Arial",
    "gridonopen": 1,
    "gridsize": [15.0, 15.0],
    "gridsnaponopen": 1,
    "objectsnaponopen": 1,
    "statusbarvisible": 2,
    "toolbarvisible": 1,
    "lefttoolbarpinned": 0,
    "toptoolbarpinned": 0,
    "righttoolbarpinned": 0,
    "bottomtoolbarpinned": 0,
    "toolbars_unpinned_last_save": 0,
    "tallnewobj": 0,
    "boxanimatetime": 200,
    "enablehscroll": 1,
    "enablevscroll": 1,
    "devicewidth": 0.0,
    "description": "",
    "digest": "",
    "tags": "",
    "style": "",
    "subpatcher_template": "",
    "assistshowspatchername": 0,
    "boxes": [],
    "lines": []
  }
}
```

## Boxes

Every object is wrapped in a `{ "box": { ... } }` container inside the `boxes` array.

### Standard Object (newobj)

```json
{
  "box": {
    "maxclass": "newobj",
    "text": "cycle~ 440",
    "id": "obj-1",
    "numinlets": 2,
    "numoutlets": 1,
    "outlettype": ["signal"],
    "patching_rect": [100, 200, 80, 22]
  }
}
```

### Key Fields

| Field | Description |
|-------|-------------|
| `maxclass` | Object type: `"newobj"`, `"toggle"`, `"number~"`, `"ezdac~"`, `"slider"`, `"comment"`, `"message"`, `"flonum"`, `"number"`, `"button"`, `"dial"`, `"multislider"`, `"live.dial"`, `"live.slider"`, etc. |
| `text` | Object text (name + arguments + attributes) |
| `id` | Unique identifier (e.g., `"obj-1"`) |
| `numinlets` | Number of inlets |
| `numoutlets` | Number of outlets |
| `outlettype` | Array of outlet types: `""` (message), `"signal"`, `"int"`, `"float"`, `"bang"`, `"list"`, `"multichannelsignal"` |
| `patching_rect` | `[x, y, width, height]` in pixels |

### Common UI Objects

**Toggle**
```json
{
  "box": {
    "maxclass": "toggle",
    "id": "obj-10",
    "numinlets": 1,
    "numoutlets": 1,
    "outlettype": ["int"],
    "patching_rect": [100, 100, 24, 24],
    "parameter_enable": 0
  }
}
```

**Number Box (float)**
```json
{
  "box": {
    "maxclass": "flonum",
    "id": "obj-11",
    "numinlets": 1,
    "numoutlets": 2,
    "outlettype": ["", "bang"],
    "patching_rect": [100, 100, 50, 22],
    "parameter_enable": 0
  }
}
```

**Message Box**
```json
{
  "box": {
    "maxclass": "message",
    "text": "440",
    "id": "obj-12",
    "numinlets": 2,
    "numoutlets": 1,
    "outlettype": [""],
    "patching_rect": [100, 100, 50, 22]
  }
}
```

**Comment**
```json
{
  "box": {
    "maxclass": "comment",
    "text": "Frequency control",
    "id": "obj-20",
    "numinlets": 1,
    "numoutlets": 0,
    "patching_rect": [100, 80, 120, 20]
  }
}
```

**ezdac~ (audio output toggle)**
```json
{
  "box": {
    "maxclass": "ezdac~",
    "id": "obj-99",
    "numinlets": 2,
    "numoutlets": 0,
    "patching_rect": [300, 600, 45, 45]
  }
}
```

**live.dial (for RNBO and Live-style UI)**
```json
{
  "box": {
    "maxclass": "live.dial",
    "varname": "frequency",
    "id": "obj-30",
    "numinlets": 1,
    "numoutlets": 2,
    "outlettype": ["", "float"],
    "patching_rect": [100, 100, 41, 48],
    "parameter_enable": 1,
    "saved_attribute_attributes": {
      "valueof": {
        "parameter_longname": "frequency",
        "parameter_shortname": "freq",
        "parameter_type": 0,
        "parameter_mmin": 20.0,
        "parameter_mmax": 20000.0,
        "parameter_initial_enable": 1,
        "parameter_initial": [440],
        "parameter_unitstyle": 3
      }
    }
  }
}
```

## Connections (lines)

```json
{
  "patchline": {
    "source": ["obj-1", 0],
    "destination": ["obj-2", 0]
  }
}
```

- `source`: `[object_id, outlet_index]`
- `destination`: `[object_id, inlet_index]`

## Subpatchers

### Standard Subpatcher (p)

```json
{
  "box": {
    "maxclass": "newobj",
    "text": "p my_subpatch",
    "id": "obj-5",
    "numinlets": 1,
    "numoutlets": 1,
    "outlettype": ["signal"],
    "patching_rect": [100, 200, 100, 22],
    "patcher": {
      "fileversion": 1,
      "appversion": { "major": 8, "minor": 6, "revision": 0, "architecture": "x64", "modernui": 1 },
      "classnamespace": "box",
      "rect": [0, 0, 600, 400],
      "boxes": [
        {
          "box": {
            "maxclass": "newobj",
            "text": "inlet~",
            "id": "sub-1",
            "numinlets": 1,
            "numoutlets": 1,
            "outlettype": ["signal"],
            "patching_rect": [50, 30, 45, 22]
          }
        },
        {
          "box": {
            "maxclass": "newobj",
            "text": "outlet~",
            "id": "sub-2",
            "numinlets": 1,
            "numoutlets": 0,
            "patching_rect": [50, 300, 55, 22]
          }
        }
      ],
      "lines": []
    }
  }
}
```

### rnbo~ Subpatcher

For RNBO objects, the subpatcher lives under the `"rnbo"` key with a different internal format. See the RNBO skill's patcher-format.md for RNBO-specific JSON.

```json
{
  "box": {
    "maxclass": "newobj",
    "text": "rnbo~",
    "id": "obj-2",
    "numinlets": 0,
    "numoutlets": 2,
    "outlettype": ["signal", "signal"],
    "patching_rect": [300, 500, 80, 22],
    "rnbo": {
      "patcher": {
        "fileversion": 1,
        "appversion": { "name": "rnbo", "version": "1.4.2" },
        "boxes": [],
        "lines": []
      }
    }
  }
}
```

### gen~ Object

```json
{
  "box": {
    "maxclass": "newobj",
    "text": "gen~",
    "id": "obj-gen",
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

Note `classnamespace` is `"dsp.gen"` for gen~ subpatchers.

## Layout Guidelines

- Standard object height: 22px
- Grid snap: 15px
- Vertical spacing between connected objects: 40-60px
- Horizontal spacing between parallel chains: 120-200px
- Place inputs at top, outputs at bottom
- Group related objects visually
- Add comments to label sections

## patching_rect Tips

- `[x, y, width, height]` — origin is top-left of patcher window
- Start layout around `[30, 30, ...]` for top-left
- Objects typically 80-280px wide depending on text length
- Use consistent vertical spacing for readability
