# RNBO .maxpat JSON Structure

How to generate valid `.maxpat` files containing `rnbo~` objects programmatically.
Based on reverse-engineering working Max 9.1 patchers.

## Architecture

```
Top-level .maxpat (classnamespace: "box")
  └─ rnbo~ object (embeds subpatcher under "patcher" key)
       └─ RNBO subpatcher (classnamespace: "rnbo")
            ├─ buffer~ softcut_buf 524288
            ├─ in~ 1
            ├─ param rate 1 @min -4 @max 4
            ├─ param play 0 @min 0 @max 1
            ├─ codebox~ (code embedded inline)
            ├─ out~ 1
            └─ out~ 2
```

## Top-Level Patcher

```json
{
  "patcher": {
    "fileversion": 1,
    "appversion": { "major": 9, "minor": 1, "revision": 2, "architecture": "x64", "modernui": 1 },
    "classnamespace": "box",
    "rect": [100, 100, 900, 700],
    "boxes": [],
    "lines": [],
    "parameters": {
      "obj-rnbo": ["rnbo~", "rnbo~", 0],
      "parameterbanks": {
        "0": { "index": 0, "name": "",
               "parameters": ["-","-","-","-","-","-","-","-"],
               "buttons": ["-","-","-","-","-","-","-","-"] }
      },
      "inherited_shortname": 1
    },
    "autosave": 0
  }
}
```

## rnbo~ Object

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
    "patcher": { ... }
  }
}
```

**Required fields:**
- `autosave: 1`
- `text: "rnbo~"` (no attributes)
- `varname: "rnbo~"`
- `rnboversion` (e.g. `"1.4.2"`)
- `saved_object_attributes` with `parameter_enable: 1` and unique `uuid`
- `saved_attribute_attributes` with parameter metadata
- `patcher` containing the RNBO subpatcher (NOT `"rnbo"` key)
- `rnboattrcache` listing param names (for top-level visibility)

**Outlets:** Last outlet is always `"list"` (dump outlet). For N signal outlets: `outlettype: ["signal", ..., "list"]`.

**Does NOT load external files.** No `@patching_filename`. Subpatcher is always inline.

## RNBO Subpatcher

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

## RNBO Object Common Fields

Every object inside the RNBO subpatcher needs:
- `rnbo_classname` -- e.g. `"in~"`, `"out~"`, `"param"`, `"codebox~"`, `"buffer~"`
- `rnbo_serial` -- unique integer per object
- `rnbo_uniqueid` -- unique string (convention: `"classname_objid"` or just param name)

Optional: `rnbo_extra_attributes` (class-specific metadata)

## Object Templates

### in~
```json
{
  "box": {
    "id": "obj-in1", "maxclass": "newobj",
    "numinlets": 0, "numoutlets": 1, "outlettype": ["signal"],
    "rnbo_classname": "in~",
    "rnbo_extra_attributes": { "comment": "", "meta": "" },
    "rnbo_serial": 1, "rnbo_uniqueid": "in~_obj-in1",
    "text": "in~ 1"
  }
}
```
**`numinlets: 0`** -- it's a source.

### out~
```json
{
  "box": {
    "id": "obj-out1", "maxclass": "newobj",
    "numinlets": 1, "numoutlets": 0,
    "rnbo_classname": "out~",
    "rnbo_extra_attributes": { "comment": "", "meta": "" },
    "rnbo_serial": 1, "rnbo_uniqueid": "out~_obj-out1",
    "text": "out~ 1"
  }
}
```
**`numoutlets: 0`** -- it's a sink.

### codebox~
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
    "rnbo_serial": 2, "rnbo_uniqueid": "codebox~_obj-cb"
  }
}
```

**Critical:**
- `maxclass: "codebox~"` (NOT `"newobj"` with nested patcher)
- Code in TWO places: `"code"` AND `"rnbo_extra_attributes"."code"`
- `safemath: 1` prevents crashes from division by zero
- `numinlets`/`numoutlets` must match code's `inN`/`outN` usage
- No external `.codebox` files -- code is always inline in the JSON

### param
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
    "rnbo_serial": 3, "rnbo_uniqueid": "rate",
    "text": "param rate 1 @min -4 @max 4",
    "varname": "rate"
  }
}
```

**Syntax:** `param name default @min X @max Y`
- Default is **positional** (2nd argument). No `@initial` attribute.
- `varname` must match the param name
- `rnbo_uniqueid` is typically just the param name

### buffer~
```json
{
  "box": {
    "id": "obj-buf", "maxclass": "newobj",
    "numinlets": 1, "numoutlets": 2, "outlettype": ["float", "bang"],
    "rnbo_classname": "buffer~",
    "rnbo_serial": 1, "rnbo_uniqueid": "buffer~_obj-buf",
    "text": "buffer~ mybuf 524288"
  }
}
```

## Connections

```json
{ "patchline": { "source": ["obj-id", outlet_index], "destination": ["obj-id", inlet_index] } }
```

## Param → Codebox Wiring Pattern

To expose params to the top-level Max patcher:

1. Create `param` objects in the RNBO subpatcher
2. Connect each param's outlet 0 to the corresponding codebox~ inlet
3. In codebox~, read param values from `in2`, `in3`, etc. (`in1` = audio)
4. Add param names to `rnboattrcache` on the rnbo~ object
5. From top-level Max, send `paramname value` messages to rnbo~

```
[param rate 1 @min -4 @max 4] ──outlet 0──> [codebox~ inlet 1]
[param play 0 @min 0 @max 1]  ──outlet 0──> [codebox~ inlet 2]
```

## Python Generation Pattern

```python
import json, uuid

def make_rnbo_param(name, default, pmin, pmax, serial, inlet_idx):
    obj_id = f"obj-p-{name}"
    return {
        "box": {
            "id": obj_id, "maxclass": "newobj", "numinlets": 2, "numoutlets": 2,
            "outlettype": ["", ""], "rnbo_classname": "param",
            "rnbo_extra_attributes": {"preset": 1, "sendinit": 1, "steps": 0.0,
                "displayname": "", "displayorder": "-", "exponent": 1.0, "ctlin": -1.0,
                "fromnormalized": "", "unit": "", "meta": "", "order": "0",
                "tonormalized": "", "enum": ""},
            "rnbo_serial": serial, "rnbo_uniqueid": name,
            "text": f"param {name} {default} @min {pmin} @max {pmax}",
            "varname": name
        }
    }, {"patchline": {"source": [obj_id, 0], "destination": ["obj-cb", inlet_idx - 1]}}
```
