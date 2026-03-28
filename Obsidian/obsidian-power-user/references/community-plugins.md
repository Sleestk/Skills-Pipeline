# Obsidian — Community Plugins Reference

> **Important:** Label all outputs from this file as using **community/third-party plugins**. These require installation from Settings → Community Plugins.

---

## Dataview

Dataview lets you query your vault like a database using note frontmatter and metadata.

### Query Types

```dataview
TABLE col1, col2, col3
FROM source
WHERE condition
SORT field ASC/DESC
LIMIT n
```

| Query Type | Output |
|---|---|
| `TABLE` | Spreadsheet-style table |
| `LIST` | Bullet list of matching notes |
| `TASK` | Task list across the vault |
| `CALENDAR` | Calendar heatmap by date field |

### FROM Clause

```dataview
FROM #tag                     ← notes with a tag
FROM "Folder/Path"            ← notes in a folder
FROM [[Note Name]]            ← notes linked from a note
FROM outgoing([[Note Name]])  ← outgoing links
FROM #project AND #active     ← multiple tags
FROM #project OR "Archive"    ← union
FROM -#archive                ← exclude tag
```

### WHERE Clause

```dataview
WHERE status = "active"
WHERE status != "done"
WHERE priority >= 3
WHERE due <= date(today)
WHERE contains(tags, "project")
WHERE !completed
WHERE file.name != this.file.name
```

### SORT & GROUP BY & LIMIT

```dataview
SORT due ASC
SORT priority DESC, due ASC
GROUP BY status
LIMIT 10
FLATTEN tags AS tag
```

### Implicit File Metadata

| Field | Description |
|---|---|
| `file.name` | Note name without extension |
| `file.path` | Full path within vault |
| `file.folder` | Parent folder |
| `file.ext` | File extension |
| `file.size` | File size in bytes |
| `file.ctime` | Creation datetime |
| `file.mtime` | Modified datetime |
| `file.tags` | All tags in the note |
| `file.inlinks` | Notes that link to this note |
| `file.outlinks` | Notes this note links to |
| `file.tasks` | All tasks in the note |
| `file.day` | Daily note date (if applicable) |

### Inline Queries

Use inside a note with backtick syntax:

```markdown
Today is `= date(today)`.
This note has `= length(file.outlinks)` outgoing links.
Status: `= this.status`
```

### Date Functions

```dataview
WHERE due <= date(today) + dur(7 days)
WHERE file.ctime >= date(today) - dur(30 days)
SORT file.mtime DESC
```

### Example Queries

**All active projects sorted by due date:**
```dataview
TABLE status, priority, due, file.mtime AS "Last Modified"
FROM #project
WHERE status != "done"
SORT due ASC
```

**Tasks due this week grouped by project:**
```dataview
TASK
FROM "Projects"
WHERE !completed AND due <= date(today) + dur(7 days)
GROUP BY file.link
SORT due ASC
```

**Recently modified notes:**
```dataview
LIST
FROM ""
SORT file.mtime DESC
LIMIT 10
```

**Calendar of notes by date field:**
```dataview
CALENDAR date
FROM #meeting
```

---

## Templater

Templater provides dynamic template execution using JavaScript-like syntax. **Must be installed** from Community Plugins.

### Syntax

```javascript
<% expression %>         ← executed, no output
<%= expression %>        ← executed, output inserted
<%* statement %>         ← statement (if, for, etc.)
<%- expression -%>       ← trim whitespace around
```

### Key tp. Functions

**Date & Time:**
```javascript
<%= tp.date.now() %>                          ← today (YYYY-MM-DD)
<%= tp.date.now("MMMM Do, YYYY") %>           ← "March 11th, 2025"
<%= tp.date.now("YYYY-MM-DD", 1) %>           ← tomorrow
<%= tp.date.now("YYYY-MM-DD", -7) %>          ← 7 days ago
<%= tp.date.now("YYYY-MM-DD", 0, tp.file.title, "YYYY-MM-DD") %> ← relative to note title
```

**File:**
```javascript
<%= tp.file.title %>                          ← note title
<%= tp.file.creation_date("YYYY-MM-DD") %>    ← file creation date
<%= tp.file.folder() %>                       ← parent folder name
<%= tp.file.path() %>                         ← full file path
<%= tp.file.move("/New/Path/" + tp.file.title) %>  ← move file on creation
```

**System:**
```javascript
<%= tp.system.prompt("Enter title") %>        ← user input dialog
<%= tp.system.suggester(["Option A", "Option B"], ["a", "b"]) %>  ← dropdown picker
<%= tp.system.clipboard() %>                  ← clipboard contents
```

**Frontmatter:**
```javascript
<%= tp.frontmatter.status %>                  ← read existing property
```

### Example Templates

**Daily Note Template:**
```javascript
---
date: <%= tp.date.now("YYYY-MM-DD") %>
day: <%* const days = ["Sunday","Monday","Tuesday","Wednesday","Thursday","Friday","Saturday"]; tR += days[moment().day()]; %>
week: W<%= tp.date.now("WW") %>
tags: [daily]
mood:
energy:
---

# <%= tp.date.now("MMMM D, YYYY") %>

## 🎯 Today's Priorities
- [ ] 
- [ ] 
- [ ] 

## 📝 Notes


## 🔗 Log


---

## 🔄 Yesterday — [[<%= tp.date.now("YYYY-MM-DD", -1) %>]]
```

**Meeting Notes Template:**
```javascript
---
date: <%= tp.date.now("YYYY-MM-DD") %>
time: <%= tp.date.now("HH:mm") %>
attendees: []
type: meeting
status: active
tags: [meeting]
---

# <%* const title = await tp.system.prompt("Meeting title"); tR += title; %>

**Date:** <%= tp.date.now("MMMM D, YYYY") %>  
**Time:** <%= tp.date.now("HH:mm") %>

## Agenda


## Notes


## Action Items
- [ ] 

## Decisions


---
*Created: <%= tp.file.creation_date("YYYY-MM-DD HH:mm") %>*
```

**New Project Template:**
```javascript
---
created: <%= tp.date.now("YYYY-MM-DD") %>
status: "planning"
priority: <%* tR += await tp.system.suggester(["1 - Low", "2 - Medium", "3 - High"], ["1", "2", "3"]); %>
tags: [project]
due: 
---

# <%= tp.file.title %>

## Overview


## Goals
- [ ] 

## Resources


## Notes
```

### User Functions

Define reusable JavaScript functions in a `.js` file, register them in Templater settings, and call them as `<%= tp.user.functionName() %>`.

### Startup Templates

In Templater settings, set a "Startup template" path — it runs every time Obsidian opens. Useful for updating a daily note or refreshing dashboard metadata.

---

## Tasks Plugin

The Tasks plugin adds powerful due dates, recurrence, and priority to standard Obsidian tasks.

### Task Syntax

```markdown
- [ ] Task description 📅 2025-03-15
- [ ] Task with start date 🛫 2025-03-10 📅 2025-03-15
- [ ] Recurring task 🔁 every week 📅 2025-03-15
- [ ] High priority task ⏫ 📅 2025-03-15
- [x] Completed task ✅ 2025-03-11
```

### Date Emojis

| Emoji | Meaning |
|---|---|
| 📅 | Due date |
| 🛫 | Start date |
| ⏳ | Scheduled date |
| ✅ | Done date (auto-added on completion) |
| ❌ | Cancelled date |
| 🔁 | Recurrence rule |

### Priority Emojis

| Emoji | Priority |
|---|---|
| 🔺 | Highest |
| ⏫ | High |
| 🔼 | Medium |
| 🔽 | Low |
| ⏬ | Lowest |

### Recurrence Examples

```markdown
🔁 every day
🔁 every week
🔁 every month
🔁 every year
🔁 every 2 weeks
🔁 every weekday
🔁 every Monday
🔁 every Monday and Friday
```

### Query Blocks

Embed task queries anywhere in your vault:

````markdown
```tasks
not done
due before next week
sort by due
```
````

**Common filters:**
```
not done
done
due before 2025-03-15
due this week
scheduled today
has due date
no due date
priority is high
tags include #project
path includes Projects/
heading includes Action Items
```

**Sort options:**
```
sort by due
sort by priority
sort by scheduled
sort by created
sort by path
```

**Display options:**
```
hide due date
hide priority
show urgency
limit 20
group by due
group by path
group by tags
```