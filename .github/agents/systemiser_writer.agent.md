---
name: systemiser_writer
description: Writes Systemiser readable JSON files based on information in the project repository.
argument-hint: A description of the desired file output in terms of the mapping between systemiser frames and objects defined in the project repository, including any specific information to include or exclude. Also relationships between objects and how these should be represented as flows between objects in Systemiser.
# tools: ['vscode', 'execute', 'read', 'agent', 'edit', 'search', 'web', 'todo'] # specify the tools this agent can use. If not set, all enabled tools are allowed.
---

<!-- Tip: Use /create-agent in chat to generate content with agent assistance -->

Here is the guide for writing Systemiser readable JSON files based on the information in the project repository. The output should be a JSON file that can be ingested by Systemiser, representing the relevant objects and relationships defined in the project repository.

# Systemiser JSON <-> GUI Structure Guide

This document explains how `Simple_test.json` maps to what you see in the Systemiser UI, and how to safely make AI-generated edits (for example: adding a frame) so the file can be re-opened correctly.

## 1. Mental model

The file is a graph model with two primary entity types:

- `Blocks`: frame-like containers (cards in the UI).
- `Lines`: items inside frames (rows in a card).

Relationships are explicit via IDs:

- Every block/line has an ID key in its collection (`Blocks`, `Lines`).
- Parent-child structure is represented with `RL.P` (parent pointer).
- Child references are listed from the parent block (`RL.BK`, `RL.LN`).
- Cross-frame flow arrows between line items are represented in each line's `FW` object.

## 2. Top-level sections

In `Simple_test.json` the root object contains:

- `Info`: file metadata (title, authors, version).
- `Settings`: global display/format options (for example code style and default card width).
- `Blocks`: all frames/cards.
- `Lines`: all line items.
- `Labels`: label counters/metadata.
- `Editing`: bookkeeping of edits and counters.

For UI structure and rendering, the critical sections are `Blocks` and `Lines`.

## 3. How frames map (`Blocks`)

Each entry in `Blocks` is a frame/card. Example IDs in this file:

- `ROOT` (special virtual root)
- `4Zk3c7Zo` ("Main frame")
- `dOx9tag6` ("Another frame")
- `jO7aUkaL` ("Frame in another frame")

### 3.1 Core block fields

Inside each block:

- `CT` (content-ish data)
  - `T`: frame title shown in UI.
  - `CD`: frame code/number shown near the frame (for example `1`, `2`, `2.1`).
  - `W`: width (matches `Settings.DefaultCardWidth` in this sample).
  - `MFL`, `MLL`: display/collapse-related flags.
  - `LB`: label text (empty in this sample).
- `RL` (relationship data)
  - `P`: parent block ID (`ROOT` for top-level frames, another block ID for nested frames).
  - `BK`: map of child block IDs to ordering/position values.
  - `LN`: map of line IDs to ordering/position values.
- `LY` (layout coordinates and spans)
  - `LC`, `LR`: logical column/row position.
  - `LCS`, `LRS`: span.
  - `CLM`, `LLM`, `LLO`: layout mode/options.
- `ED` (edit metadata)
  - `LEA`, `LED`: last-edit author/date.

### 3.2 Parent/child nesting behavior

Nested frame behavior is represented twice (both are important):

1. Child points to parent with `child.RL.P = parentId`.
2. Parent lists child in `parent.RL.BK[childId] = <order/position number>`.

In this file:

- `dOx9tag6` is under `ROOT`.
- `jO7aUkaL` has `RL.P = "dOx9tag6"`.
- `dOx9tag6.RL.BK` includes `"jO7aUkaL": 1024`.

That corresponds to the nested card "Frame in another frame" inside "Another frame" in the screenshot.

## 4. How items map (`Lines`)

Each entry in `Lines` is a row/item shown inside a frame.

Example IDs in this file:

- `4gPGhoOV` with text "An item"
- `KOn8CBLj` with text "A receiving item"

### 4.1 Core line fields

Inside each line:

- `CT`
  - `LH`: line text displayed in UI.
  - `AL`: activity level/line type flag (numeric).
  - `LB`: label text (empty here).
- `RL`
  - `P`: parent block ID (which frame the line belongs to).
- `FW` (flow connections)
  - `FOut` / `FlwOut`: outgoing flow present and target line IDs.
  - `FIn` / `FlwIn`: incoming flow present and source line IDs.
  - `TO` / `TI`: displayed flow tag/index (both are `"1"` in this sample flow).
- `ED`
  - edit metadata.

## 5. How the arrow/flow in UI maps

The visible arrow from "An item" to "A receiving item" is represented bidirectionally:

- Source line `4gPGhoOV`
  - `FW.FOut = true`
  - `FW.FlwOut = ["KOn8CBLj"]`
  - `FW.TO = "1"`
- Target line `KOn8CBLj`
  - `FW.FIn = true`
  - `FW.FlwIn = ["4gPGhoOV"]`
  - `FW.TI = "1"`

To add/remove a flow reliably, update both ends.

## 6. Practical AI edit protocol

When using AI to modify this JSON, enforce these invariants:

1. Preserve valid JSON (no trailing commas, no duplicate keys).
2. Use unique IDs for new `Blocks`/`Lines` keys.
3. Keep parent links symmetric:
   - set `child.RL.P`, and
   - add child ID to parent `RL.BK` (for block child) or `RL.LN` (for line child).
4. Keep flow links symmetric:
   - source `FlwOut` must match target `FlwIn`.
5. Keep code/title coherence:
   - set `CT.T` and `CT.CD` consistent with intended numbering shown in UI.
6. Preserve `ROOT` block and its role as parent of top-level frames.
7. Do minimal edits:
   - only touch objects needed for the requested change.

## 7. Example: Add another top-level frame

Goal: add a new frame "Third frame" as a top-level card under `ROOT`.

### 7.1 Required JSON changes

1. Create a new block under `Blocks`, for example ID `newFrame123`.
2. Set:
   - `Blocks.newFrame123.CT.T = "Third frame"`
   - `Blocks.newFrame123.CT.CD = "3"` (or next code you want)
   - `Blocks.newFrame123.RL.P = "ROOT"`
3. Insert this new block ID into `Blocks.ROOT.RL.BK` with an ordering number.
4. Set layout fields (`LY`) to a sensible position (`LC`, `LR`) so it appears where expected.

### 7.2 Minimal template

```json
"newFrame123": {
  "CT": {
    "__key": "newFrame123",
    "CD": "3",
    "T": "Third frame",
    "LB": "",
    "MLL": 1,
    "MFL": 0,
    "W": 400
  },
  "RL": {
    "P": "ROOT"
  },
  "ED": {
    "LEA": "BGYt5gmu3Qg5HyBxGlTjxZOBgyW2",
    "LED": 1776927297776
  },
  "LY": {
    "CLM": "simple",
    "LLM": "simple",
    "LLO": { "numberTracks": 4, "trackType": "row" },
    "LC": 3,
    "LR": 1,
    "LCS": 1,
    "LRS": 1
  }
}
```

And update `Blocks.ROOT.RL.BK`, for example:

```json
"BK": {
  "4Zk3c7Zo": 64,
  "dOx9tag6": 80,
  "newFrame123": 96
}
```

## 8. Example AI prompt pattern

Use prompts that force structural consistency, for example:

"Edit `Simple_test.json` to add a new top-level frame called 'Third frame'. Keep JSON valid. Add one new block ID only, set parent to ROOT, add that ID to ROOT.RL.BK, keep existing blocks/lines unchanged, and do not remove any existing flow links."

## 9. Unknowns and safe assumptions

Some numeric values appear to be internal positioning/order tokens (`64`, `80`, `1024`) and some counters in `Editing` may be derived. Based on this sample, the most important thing for reliable reload is structural consistency of IDs and relationships.

If unsure, copy patterns from existing sibling objects in the same file and change only the required fields.

---

## 10. Function-as-card layout pattern

This section documents the pattern for representing code/function graphs, where each function is its own card (frame/block) containing description lines and call-link lines.

### 10.1 Hierarchy

```
ROOT
└── folder block  (e.g. "bivios_functions", CD="1", MFL=1)
    └── file block  (e.g. "build_network.py", CD="1.1", MFL=1)
        └── function block  (e.g. "_get_grid_ppa_price_raw_value", CD="1.1.1", MFL=0)
            ├── line: blank
            ├── line: "called by"  ← flow source (FOut)
            ├── line: "calls"      ← flow target (FIn)
            ├── line: blank
            ├── line: <i>description</i>
            ├── line: blank
            ├── line: <b>Inputs: </b>arg1, arg2
            └── line: <b>Outputs: </b>returns a value.
```

- Folder and file blocks use `RL.BK` (child block map), not `RL.LN`.
- Function blocks use `RL.LN` (child line map). They are leaf-level cards.
- Set `MFL=1` on folder/file blocks (they contain nested frames). Set `MFL=0` on function blocks.

### 10.2 Line content per function card

| Position (ordering) | Line text | Flow role |
|---|---|---|
| 64  | `` (blank) | none |
| 128 | `called by` | **FOut = True** (source) → tag on RIGHT |
| 192 | `calls` | **FIn = True** (target) → tag on LEFT |
| 256 | `` (blank) | none |
| 320 | `<i>description text</i>` | none |
| 384 | `` (blank) | none |
| 448 | `<b>Inputs: </b>arg1, arg2` | none |
| 512 | `<b>Outputs: </b>returns a value.` | none |

HTML bold/italic tags (`<b>`, `<i>`) are rendered as formatting in the UI.

### 10.3 Flow direction — reversed call semantics

**Conceptual rule:** arrows point FROM callee TO caller.
- The callee's `called by` line is the **source** (`FOut=True`, `FlwOut=[...]`).
- The caller's `calls` line is the **target** (`FIn=True`, `FlwIn=[...]`).

This means the visible arrow flows left-to-right from the called function toward the calling function, letting you read "function A (right) calls function B (left)" by following the arrow in reverse.

**For a call relationship where function A calls function B:**

```
B.called_by:  FOut=True,  FlwOut=["A.calls_line_id"],  TO="n"
A.calls:      FIn=True,   FlwIn=["B.called_by_line_id"], TI="n"
```

Both ends use the same tag index `n`. Update both ends to keep flow symmetry.

### 10.4 Minimal template for a function block

```json
"bFuncId01": {
  "CT": {
    "__key": "bFuncId01",
    "CD": "1.1.1",
    "T": "my_function",
    "LB": "",
    "MLL": 1,
    "MFL": 0,
    "W": 400
  },
  "RL": {
    "P": "bFileId01",
    "LN": {
      "lBlank1xx": 64,
      "lCalledBy": 128,
      "lCallsxxx": 192,
      "lBlank2xx": 256,
      "lDescxxxx": 320,
      "lBlank3xx": 384,
      "lInputsxx": 448,
      "lOutputsx": 512
    }
  },
  "ED": { "LEA": "BGYt5gmu3Qg5HyBxGlTjxZOBgyW2", "LED": 1776927297776 },
  "LY": {
    "CLM": "simple", "LLM": "simple",
    "LLO": { "numberTracks": 4, "trackType": "row" },
    "LC": 3, "LR": 1, "LCS": 1, "LRS": 1
  }
}
```

And the `called by` / `calls` lines within it:

```json
"lCalledBy": {
  "CT": { "__key": "lCalledBy", "AL": 1, "LB": "", "LH": "called by" },
  "RL": { "P": "bFuncId01" },
  "FW": { "FOut": true, "FlwOut": ["lCallsOfCallerFunc"], "TO": "5" },
  "ED": { "LEA": "BGYt5gmu3Qg5HyBxGlTjxZOBgyW2", "LED": 1776927297776 }
},
"lCallsxxx": {
  "CT": { "__key": "lCallsxxx", "AL": 1, "LB": "", "LH": "calls" },
  "RL": { "P": "bFuncId01" },
  "FW": { "FIn": true, "FlwIn": ["lCalledByOfCalleeFunc"], "TI": "7" },
  "ED": { "LEA": "BGYt5gmu3Qg5HyBxGlTjxZOBgyW2", "LED": 1776927297776 }
}
```

### 10.5 File block — must use BK not LN

When functions are cards, the file block lists them as **child blocks** (`BK`), not lines:

```json
"bFileId01": {
  "CT": { "__key": "bFileId01", "CD": "1.1", "T": "build_network.py", "LB": "", "MLL": 1, "MFL": 1, "W": 400 },
  "RL": {
    "P": "bFolderBlock",
    "BK": {
      "bFuncId01": 64,
      "bFuncId02": 128
    }
  },
  ...
}
```

`MFL=1` on the file block signals it contains nested frames rather than lines.


## 11. Where to write files to

For now, output files into the bivios_projects/systemiser directory, with a name like `systemiser_output_<timestamp>.json` to avoid overwriting existing files. 