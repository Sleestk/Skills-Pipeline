# Obsidian — Bases Reference

Bases is Obsidian's native database system (core plugin). Files are saved as `.base` and use YAML-based syntax. Always output **complete, valid `.base` YAML** — never a description.

## What Bases Does

- Creates database-like views of vault notes using their frontmatter properties
- Replaces most Dataview `TABLE` and `LIST` use cases natively
- Supports filtering, sorting, grouping, and formula properties

## Views Available

| View Type | Description |
|---|---|
| `table` | Spreadsheet-style with columns |
| `cards` | Kanban-style card layout |
| `list` | Simple list view |

## Full `.base` File Structure

```yaml
filters:
  and:
    - file.inFolder("Projects")
    - 'status != "done"'
formulas:
  days_old: "now() - file.ctime"
display:
  status: Status
  priority: Priority
  formula.days_old: Days Old
views:
  - type: table
    name: Active Projects
    filters:
      and:
        - 'status == "active"'
    order:
      - file.name
      - status
  - type: cards
    name: Kanban Board
    groupBy: status
```

## Filters

### Logic Operators

```yaml
filters:
  and:
    - condition1
    - condition2
  or:
    - condition1
    - condition2
  not:
    - condition
```

### File Functions

```yaml
file.inFolder("Projects")          ← notes inside a folder
file.inFolder("Projects", true)    ← includes subfolders
file.ext == "md"                   ← by file extension
```

### File Properties (implicit)

```yaml
file.name          ← note name without extension
file.path          ← full vault-relative path
file.ctime         ← creation timestamp
file.mtime         ← modified timestamp
file.size          ← file size in bytes
```

### Tag & Link Functions

```yaml
taggedWith(file.file, "project")
linksTo(file.file, "Note Name")
```

### Property Comparisons

```yaml
'status == "done"'
'status != "done"'
'priority > 2'
'priority >= 3'
'due < now()'
```

## Formulas

Defined in the `formulas:` block and referenced as `formula.name` in `display:`.

| Category | Functions |
|---|---|
| Arithmetic | `+`, `-`, `*`, `/` |
| String | `concat()`, `upper()`, `lower()`, `contains()`, `startsWith()`, `endsWith()` |
| Date | `date()`, `now()`, `datetime.format("YYYY-MM-DD")` |
| Conditional | `if(condition, trueValue, falseValue)` |
| List | `list().map()`, `list().filter()`, `list().length` |

**Examples:**
```yaml
formulas:
  days_old: "now() - file.ctime"
  is_overdue: "if(due < now(), true, false)"
  full_title: "concat(type, ': ', file.name)"
  tag_count: "tags.length"
```

## Display Block

Maps property keys to human-readable column headers:
```yaml
display:
  file.name: Title
  status: Status
  priority: Priority
  due: Due Date
  formula.days_old: Age (Days)
```

## Example — Project Dashboard

```yaml
filters:
  and:
    - file.inFolder("Projects")
formulas:
  days_active: "now() - file.ctime"
  is_overdue: "if(due < now() and status != 'done', true, false)"
display:
  file.name: Project
  status: Status
  priority: Priority
  due: Due Date
  formula.days_active: Days Active
  formula.is_overdue: Overdue?
views:
  - type: table
    name: All Projects
    order:
      - priority
      - due
  - type: table
    name: Active Only
    filters:
      and:
        - 'status == "active"'
    order:
      - due
  - type: cards
    name: Kanban
    groupBy: status
```

## Example — Tagged Notes Database

```yaml
filters:
  and:
    - taggedWith(file.file, "project")
    - 'status != "done"'
display:
  file.name: Title
  status: Status
  tags: Tags
  due: Due
views:
  - type: table
    name: Active Projects
    order:
      - due
  - type: list
    name: Quick List
```

## Output Rule

All `.base` outputs must be complete, valid YAML in a fenced code block. The user should be able to save it directly as a `.base` file in their vault with zero edits.