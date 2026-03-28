# Obsidian — Canvas Reference

Canvas files are saved as `.canvas` and use an open JSON format. Always output **complete, valid `.canvas` JSON** — never a description.

## Full File Structure

```json
{
  "nodes": [],
  "edges": []
}
```

## Node Types

| Type | Description | Required Fields |
|---|---|---|
| `text` | Standalone text card | `text` |
| `file` | Reference to vault note | `file` (relative path) |
| `link` | External URL card | `url` |
| `group` | Container for other nodes | `label` |

## Node Schema

```json
{
  "id": "unique-string-id",
  "type": "text",
  "x": 0,
  "y": 0,
  "width": 400,
  "height": 200,
  "color": "1",
  "text": "Card content here (supports markdown)"
}
```

**For `file` nodes:**
```json
{
  "id": "node-2",
  "type": "file",
  "x": 500,
  "y": 0,
  "width": 400,
  "height": 300,
  "file": "Projects/My Note.md"
}
```

**For `link` nodes:**
```json
{
  "id": "node-3",
  "type": "link",
  "x": 1000,
  "y": 0,
  "width": 400,
  "height": 200,
  "url": "https://example.com"
}
```

**For `group` nodes:**
```json
{
  "id": "group-1",
  "type": "group",
  "x": -50,
  "y": -50,
  "width": 900,
  "height": 400,
  "label": "Phase 1",
  "color": "3"
}
```

## Edge Schema

```json
{
  "id": "edge-unique-id",
  "fromNode": "node-id-source",
  "toNode": "node-id-target",
  "fromSide": "right",
  "toSide": "left",
  "label": "Optional edge label",
  "color": "2"
}
```

**Side values:** `right` `left` `top` `bottom`

## Color Values

| Value | Color |
|---|---|
| `"1"` | Red |
| `"2"` | Orange |
| `"3"` | Yellow |
| `"4"` | Green |
| `"5"` | Cyan |
| `"6"` | Purple |

Omit `color` for the default vault accent color.

## Layout Strategies

**Swim lane** — columns for stages, topics, or people:
- Group nodes in vertical columns with group nodes as headers
- x-positions spaced ~600px apart per column

**Topic cluster** — hub and spoke:
- Central node at (0, 0)
- Spokes radiate outward at ~600px radius, edges from center to each

**Pipeline** — left-to-right sequential flow:
- Nodes spaced evenly along x-axis
- Edges from right side of each node to left side of next

**Hierarchical** — parent → child trees:
- Root at top center
- Children spaced below with y increasing per level

## Example — Content Creation Pipeline Canvas

```json
{
  "nodes": [
    {
      "id": "idea",
      "type": "text",
      "x": 0,
      "y": 0,
      "width": 300,
      "height": 120,
      "color": "5",
      "text": "## 💡 Idea\nRaw topic or concept"
    },
    {
      "id": "research",
      "type": "text",
      "x": 400,
      "y": 0,
      "width": 300,
      "height": 120,
      "color": "3",
      "text": "## 🔍 Research\nSources, angles, data"
    },
    {
      "id": "script",
      "type": "text",
      "x": 800,
      "y": 0,
      "width": 300,
      "height": 120,
      "color": "4",
      "text": "## 📝 Script\nFull draft"
    },
    {
      "id": "edit",
      "type": "text",
      "x": 1200,
      "y": 0,
      "width": 300,
      "height": 120,
      "color": "2",
      "text": "## ✂️ Edit\nTimeline cut"
    },
    {
      "id": "publish",
      "type": "text",
      "x": 1600,
      "y": 0,
      "width": 300,
      "height": 120,
      "color": "1",
      "text": "## 🚀 Publish\nUpload + promote"
    }
  ],
  "edges": [
    { "id": "e1", "fromNode": "idea", "toNode": "research", "fromSide": "right", "toSide": "left" },
    { "id": "e2", "fromNode": "research", "toNode": "script", "fromSide": "right", "toSide": "left" },
    { "id": "e3", "fromNode": "script", "toNode": "edit", "fromSide": "right", "toSide": "left" },
    { "id": "e4", "fromNode": "edit", "toNode": "publish", "fromSide": "right", "toSide": "left" }
  ]
}
```

## Output Rule

All canvas outputs **must** be a complete, valid JSON block. The user should be able to save it directly as a `.canvas` file in their vault with zero edits.