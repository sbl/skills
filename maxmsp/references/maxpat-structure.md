# .maxpat JSON Structure

How to generate valid `.maxpat` files programmatically. Based on reverse-engineering Max 9.1 patchers.

No official schema exists. This reference covers plain Max, gen~, and RNBO patchers.

---

## Top-Level Patcher (all modes)

```json
{
  "patcher": {
    "fileversion": 1,
    "appversion": { "major": 9, "minor": 1, "revision": 2, "architecture": "x64", "modernui": 1 },
    "classnamespace": "box",
    "rect": [100, 100, 900, 700],
    "gridsize": [15.0, 15.0],
    "boxes": [],
    "lines": [],
    "autosave": 0
  }
}
```

## Connections

All modes use the same format:
```json
{ "patchline": { "source": ["obj-id", outlet_index], "destination": ["obj-id", inlet_index] } }
```

---

## Plain Max Objects

### newobj (generic)
```json
{
  "box": {
    "id": "obj-1", "maxclass": "newobj",
    "numinlets": 1, "numoutlets": 1, "outlettype": [""],
    "patching_rect": [100, 100, 80, 22],
    "text": "loadbang"
  }
}
```

### message
```json
{
  "box": {
    "id": "obj-2", "maxclass": "message",
    "numinlets": 2, "numoutlets": 1, "outlettype": [""],
    "patching_rect": [100, 140, 60, 22],
    "text": "bang"
  }
}
```

### comment
```json
{
  "box": {
    "id": "obj-3", "maxclass": "comment",
    "numinlets": 1, "numoutlets": 0,
    "patching_rect": [100, 60, 200, 20],
    "text": "My comment"
  }
}
```

### number (int)
```json
{
  "box": {
    "id": "obj-4", "maxclass": "number",
    "numinlets": 1, "numoutlets": 2, "outlettype": ["", "bang"],
    "patching_rect": [100, 100, 50, 22],
    "parameter_enable": 0
  }
}
```

### flonum (float)
```json
{
  "box": {
    "id": "obj-5", "maxclass": "flonum",
    "numinlets": 1, "numoutlets": 2, "outlettype": ["", "bang"],
    "patching_rect": [100, 100, 50, 22],
    "parameter_enable": 0
  }
}
```

### toggle
```json
{
  "box": {
    "id": "obj-6", "maxclass": "toggle",
    "numinlets": 1, "numoutlets": 1, "outlettype": ["int"],
    "patching_rect": [100, 100, 24, 24],
    "parameter_enable": 0
  }
}
```

### slider
```json
{
  "box": {
    "id": "obj-7", "maxclass": "slider",
    "numinlets": 1, "numoutlets": 1, "outlettype": [""],
    "patching_rect": [100, 100, 20, 140],
    "parameter_enable": 0,
    "size": 128
  }
}
```

### gain~ (signal level)
```json
{
  "box": {
    "id": "obj-8", "maxclass": "gain~",
    "numinlets": 1, "numoutlets": 2, "outlettype": ["signal", ""],
    "patching_rect": [100, 200, 22, 140],
    "parameter_enable": 0
  }
}
```

### ezdac~ (audio output)
```json
{
  "box": {
    "id": "obj-9", "maxclass": "ezdac~",
    "numinlets": 2, "numoutlets": 0,
    "patching_rect": [100, 400, 45, 45]
  }
}
```

### ezadc~ (audio input)
```json
{
  "box": {
    "id": "obj-10", "maxclass": "ezadc~",
    "numinlets": 0, "numoutlets": 2, "outlettype": ["signal", "signal"],
    "patching_rect": [100, 50, 45, 45]
  }
}
```

### live.dial (parameter UI)
```json
{
  "box": {
    "id": "obj-11", "maxclass": "live.dial",
    "numinlets": 1, "numoutlets": 2, "outlettype": ["", "float"],
    "patching_rect": [100, 100, 41, 48],
    "parameter_enable": 1,
    "saved_attribute_attributes": {
      "valueof": {
        "parameter_longname": "cutoff",
        "parameter_shortname": "cutoff",
        "parameter_type": 0,
        "parameter_mmin": 20.0,
        "parameter_mmax": 20000.0,
        "parameter_initial": [1000.0],
        "parameter_unitstyle": 3
      }
    },
    "varname": "cutoff"
  }
}
```

---

## gen~ Patcher Embedding

### Architecture
```
Top-level .maxpat (classnamespace: "box")
  └─ gen~ object (embeds subpatcher under "patcher" key)
       └─ gen subpatcher (classnamespace: "dsp.gen")
            ├─ in 1
            ├─ codebox (GenExpr code)
            └─ out 1
```

### gen~ object
```json
{
  "box": {
    "id": "obj-gen", "maxclass": "newobj",
    "numinlets": 1, "numoutlets": 1, "outlettype": ["signal"],
    "patching_rect": [100, 200, 80, 22],
    "text": "gen~",
    "patcher": {
      "fileversion": 1,
      "appversion": { "major": 9, "minor": 1, "revision": 2, "architecture": "x64", "modernui": 1 },
      "classnamespace": "dsp.gen",
      "rect": [130, 461, 800, 600],
      "boxes": [],
      "lines": []
    }
  }
}
```

### gen~ in / out
```json
{ "box": { "id": "obj-gin1", "maxclass": "newobj", "numinlets": 0, "numoutlets": 1, "outlettype": [""], "text": "in 1" } }
{ "box": { "id": "obj-gout1", "maxclass": "newobj", "numinlets": 1, "numoutlets": 0, "text": "out 1" } }
```

### gen~ codebox (GenExpr)
```json
{
  "box": {
    "id": "obj-gcb", "maxclass": "newobj",
    "numinlets": 1, "numoutlets": 1, "outlettype": [""],
    "patching_rect": [50, 80, 600, 400],
    "text": "codebox",
    "patcher": {
      "fileversion": 1,
      "appversion": { "major": 9, "minor": 1, "revision": 2, "architecture": "x64", "modernui": 1 },
      "classnamespace": "dsp.gen",
      "rect": [50, 80, 600, 400],
      "boxes": [],
      "lines": [],
      "code": "// GenExpr code here\nParam freq(440, min=20, max=20000);\nout1 = cycle(freq);"
    }
  }
}
```

**Note:** gen~ codebox stores code in the nested `"patcher"."code"` field. The `text` is `"codebox"` (not `"codebox~"`).

### gen~ param
```json
{ "box": { "id": "obj-gp1", "maxclass": "newobj", "numinlets": 0, "numoutlets": 1, "outlettype": [""], "text": "param freq @default 440 @min 20 @max 20000" } }
```

---

## RNBO Patcher Embedding

### Architecture
```
Top-level .maxpat (classnamespace: "box")
  └─ rnbo~ object (embeds subpatcher under "patcher" key)
       └─ RNBO subpatcher (classnamespace: "rnbo")
            ├─ buffer~ buf 524288
            ├─ in~ 1
            ├─ param rate 1 @min -4 @max 4
            ├─ codebox~ (RNBO script inline)
            ├─ out~ 1
            └─ out~ 2
```

### rnbo~ object
```json
{
  "box": {
    "autosave": 1,
    "id": "obj-rnbo",
    "maxclass": "newobj",
    "numinlets": 1,
    "numoutlets": 2,
    "outlettype": ["signal", "list"],
    "text": "rnbo~",
    "varname": "rnbo~",
    "patching_rect": [100, 200, 80, 22],
    "inletInfo": {
      "IOInfo": [{ "type": "signal", "index": 1, "tag": "in1", "comment": "" }]
    },
    "outletInfo": {
      "IOInfo": [{ "type": "signal", "index": 1, "tag": "out1", "comment": "" }]
    },
    "rnboversion": "1.4.2",
    "rnboattrcache": {
      "rate": { "label": "rate", "isEnum": 0, "parsestring": "" }
    },
    "saved_attribute_attributes": {
      "valueof": {
        "parameter_invisible": 1,
        "parameter_longname": "rnbo~",
        "parameter_modmode": 0,
        "parameter_shortname": "rnbo~",
        "parameter_type": 3
      }
    },
    "saved_object_attributes": {
      "optimization": "O1",
      "parameter_enable": 1,
      "uuid": "unique-uuid-here"
    },
    "patcher": { }
  }
}
```

**Required fields:**
- `autosave: 1`
- `text: "rnbo~"` (no attributes)
- `varname: "rnbo~"`
- `rnboversion` (e.g. `"1.4.2"`)
- `saved_object_attributes` with `parameter_enable: 1` and unique `uuid`
- `rnboattrcache` listing param names for top-level visibility
- `patcher` containing the RNBO subpatcher

**Outlets:** Last outlet is always `"list"` (dump outlet). For N signal outlets: `outlettype: ["signal", ..., "list"]`.

### RNBO Subpatcher
```json
{
  "fileversion": 1,
  "appversion": { "major": 9, "minor": 1, "revision": 2, "architecture": "x64", "modernui": 1 },
  "classnamespace": "rnbo",
  "rect": [130, 461, 1000, 780],
  "default_fontname": "Lato",
  "title": "untitled",
  "boxes": [],
  "lines": []
}
```

### RNBO common fields

Every object inside the RNBO subpatcher needs:
- `rnbo_classname` -- e.g. `"in~"`, `"out~"`, `"param"`, `"codebox~"`, `"buffer~"`
- `rnbo_serial` -- unique integer per object
- `rnbo_uniqueid` -- unique string

### RNBO in~ / out~
```json
{ "box": { "id": "obj-in1", "maxclass": "newobj", "numinlets": 0, "numoutlets": 1, "outlettype": ["signal"], "rnbo_classname": "in~", "rnbo_extra_attributes": { "comment": "", "meta": "" }, "rnbo_serial": 1, "rnbo_uniqueid": "in~_obj-in1", "text": "in~ 1" } }
{ "box": { "id": "obj-out1", "maxclass": "newobj", "numinlets": 1, "numoutlets": 0, "rnbo_classname": "out~", "rnbo_extra_attributes": { "comment": "", "meta": "" }, "rnbo_serial": 2, "rnbo_uniqueid": "out~_obj-out1", "text": "out~ 1" } }
```

### RNBO codebox~
```json
{
  "box": {
    "code": "out1 = in1 * in2;",
    "fontface": 0, "fontname": "<Monospaced>", "fontsize": 12.0,
    "id": "obj-cb", "maxclass": "codebox~",
    "numinlets": 2, "numoutlets": 1, "outlettype": ["signal"],
    "patching_rect": [50, 80, 700, 500],
    "rnbo_classname": "codebox~",
    "rnbo_extra_attributes": {
      "expr": "", "code": "out1 = in1 * in2;", "safemath": 1, "nocache": 0
    },
    "rnbo_serial": 3, "rnbo_uniqueid": "codebox~_obj-cb"
  }
}
```

**Critical:**
- `maxclass: "codebox~"` (NOT `"newobj"`)
- Code in TWO places: `"code"` AND `"rnbo_extra_attributes"."code"`
- `safemath: 1` prevents crashes from division by zero
- `numinlets`/`numoutlets` must match code's `inN`/`outN` usage

### RNBO param
```json
{
  "box": {
    "id": "obj-p-rate", "maxclass": "newobj",
    "numinlets": 2, "numoutlets": 2, "outlettype": ["", ""],
    "rnbo_classname": "param",
    "rnbo_extra_attributes": {
      "preset": 1, "sendinit": 1, "steps": 0.0,
      "displayname": "", "displayorder": "-", "exponent": 1.0,
      "ctlin": -1.0, "fromnormalized": "", "unit": "", "meta": "",
      "order": "0", "tonormalized": "", "enum": ""
    },
    "rnbo_serial": 4, "rnbo_uniqueid": "rate",
    "text": "param rate 1 @min -4 @max 4",
    "varname": "rate"
  }
}
```

**Syntax:** `param name default @min X @max Y` -- default is positional (2nd arg). No `@initial`.

### RNBO buffer~
```json
{
  "box": {
    "id": "obj-buf", "maxclass": "newobj",
    "numinlets": 1, "numoutlets": 2, "outlettype": ["float", "bang"],
    "rnbo_classname": "buffer~",
    "rnbo_serial": 5, "rnbo_uniqueid": "buffer~_obj-buf",
    "text": "buffer~ mybuf 524288"
  }
}
```

### RNBO param wiring pattern

To expose params to the top-level Max patcher:

1. Create `param` objects in the RNBO subpatcher
2. Connect each param outlet 0 to codebox~ inlet
3. In codebox~, read from `in2`, `in3`, etc. (`in1` = audio)
4. Add param names to `rnboattrcache` on the rnbo~ object

```
[param rate 1 @min -4 @max 4] --outlet 0--> [codebox~ inlet 1]
[param play 0 @min 0 @max 1]  --outlet 0--> [codebox~ inlet 2]
```
