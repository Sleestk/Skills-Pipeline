# Obsidian — Core Plugins Reference

All plugins listed are **core plugins** built into Obsidian — no installation required. Enable/disable in Settings → Core Plugins.

---

## Audio Recorder
- Records audio directly into the vault
- Stored as `.webm` by default in vault root (configurable to a subfolder)
- Access via ribbon icon or Command Palette: "Audio Recorder: Start recording"
- Files are automatically named with timestamp

---

## Backlinks
- **Backlinks pane:** Shows all notes that link to the current note
- **Unlinked mentions:** Detects text matching note names that haven't been linked yet — click to convert to a link
- **Inline backlinks:** Toggle to show backlinks at the bottom of the current note (Settings → Backlinks → "Show backlinks in document")

---

## Bookmarks
- Save notes, headings, blocks, searches, graph views, and URLs
- Accessible from the left sidebar bookmark icon
- Organize into **bookmark groups** (folders within the bookmarks panel)
- Right-click any item in the file explorer → "Bookmark"

---

## Canvas
*See `references/canvas.md` for full syntax and examples.*

---

## Command Palette
- Open: `Ctrl/Cmd+P`
- Fuzzy search across all available commands
- **Pin commands:** Click the pin icon next to a command to pin it to the top
- Recent commands appear at the top automatically

---

## Daily Notes
Settings → Core Plugins → Daily Notes:

| Setting | Description | Example |
|---|---|---|
| Date format | Moment.js format string | `YYYY-MM-DD` |
| New file location | Folder for daily notes | `Journal/Daily` |
| Template file | Path to template note | `Templates/Daily Template` |
| Open on startup | Auto-open today's note | toggle on/off |

- Open today's note: `Ctrl/Cmd+P` → "Daily Notes: Open today's daily note"
- Previous/next day navigation via command palette or ribbon

---

## File Explorer
- Left sidebar panel listing all vault files and folders
- **Create note:** Click the new note icon or right-click folder → "New note"
- **Create folder:** Right-click → "New folder"
- **Drag and drop:** Drag files between folders to move them
- **Reveal in Finder/Explorer:** Right-click → "Reveal in Finder" (macOS) / "Show in Explorer" (Windows)
- **Sort options:** Name, modified time, created time (ascending/descending)
- **Collapse all:** Click the collapse icon in the panel header

---

## File Recovery
- Takes automatic snapshots of notes at a configurable interval
- Settings → File Recovery:
  - **Snapshot interval:** How often to take snapshots (default: 5 minutes)
  - **History length:** How long to retain snapshots (default: 7 days)
- Access: Command Palette → "File Recovery: Open saved snapshots"
- Browse snapshots by date and restore any version

---

## Format Converter
- Converts legacy Markdown formats to Obsidian standard
- Handles: Wikilinks → standard MD links, highlights, strikethrough, and more
- Use: Command Palette → "Format Converter: Open format converter"
- Preview changes before applying — non-destructive until confirmed

---

## Graph View
- Visualizes all notes and their link relationships
- **Global graph:** `Ctrl/Cmd+G`
- **Local graph:** Shows connections for the current note only

**Display settings:**
- Show arrows (link direction)
- Show orphans (unlinked notes)
- Show tags as nodes
- Show attachments

**Filters:** Include/exclude paths, tags, or specific notes

**Groups:** Assign colors by path, tag, or link:
```
path:Projects → color: red
tag:#ai → color: blue
```

**Forces (physics simulation):**
- Repel strength — how much nodes push apart
- Link strength — tension of link lines
- Center force — pulls all nodes toward center

---

## Note Composer
- **Merge notes:** Merge the content of another note into the current note
- **Extract to new note:** Select text → Command Palette → "Note Composer: Extract current selection..."
  - Creates a new note with the selected text
  - Replaces selection with a link to the new note
  - Optionally applies a template to the new note

---

## Outgoing Links
- Sidebar panel showing all links **from** the current note
- **Unlinked mentions:** Text that matches other note names but isn't linked — click to convert

---

## Outline
- Sidebar panel displaying the heading structure of the current note
- Click any heading to jump directly to it
- Works with H1–H6
- Useful for navigating long documents

---

## Page Preview
- Hover over any wikilink while holding `Ctrl` (or `Cmd` on macOS) to see a popup preview of the target note
- Enable/disable in Settings → Core Plugins → Page Preview
- Works for notes, headings, and block references

---

## Properties View
- Sidebar panel for browsing and editing frontmatter properties
- Shows all properties across the entire vault (global view)
- Click a property to see all notes that use it
- Manage property types (text, number, date, boolean, list) from here

---

## Quick Switcher
- Open: `Ctrl/Cmd+O`
- Fuzzy search to open any note by name or alias
- **Create new note:** Type a name that doesn't exist → press Enter
- Searches note titles and aliases defined in frontmatter

---

## Random Note
- Opens a random note from the vault
- Command: "Random Note: Open random note"
- Use case: serendipitous review, resurface forgotten notes, build unexpected connections

---

## Search
- Open: `Ctrl/Cmd+Shift+F`
- Full-text search across the entire vault

**Search operators:**

| Operator | Example | Description |
|---|---|---|
| `path:` | `path:Projects` | Search within a path |
| `tag:` | `tag:#project` | Notes with a specific tag |
| `file:` | `file:meeting` | Match by filename |
| `line:` | `line:action item` | Terms on the same line |
| `block:` | `block:summary` | Terms in the same block |
| `section:` | `section:results` | Terms in the same section |
| `content:` | `content:revenue` | Match note content only |
| `/regex/` | `/\d{4}-\d{2}-\d{2}/` | Regex pattern |

**Embed search results in a note:**
```markdown
```query
tag:#project status:active
```
```

---

## Slash Commands
- Type `/` while editing a note to open the command picker
- Fuzzy search all commands without leaving the editor
- Insert templates, add properties, create headings, etc.

---

## Slides
- Present a note as a slideshow
- Use `---` as the slide separator
- Command: "Slides: Start presentation"
- Navigation: arrow keys or click controls
- Full-screen mode supported

**Example slide note:**
```markdown
# Title Slide

---

## Slide 2

Content here.

---

## Slide 3

More content.
```

---

## Tags View
- Sidebar panel showing all tags across the vault
- Nested tag hierarchy displayed as a tree
- Click any tag to filter notes by that tag
- Shows tag count per tag

---

## Templates
- **Static** template insertion (no code execution — use Templater for dynamic content)
- Settings → Templates:
  - **Template folder:** Path to folder containing template files
  - **Date format:** Moment.js format (e.g., `YYYY-MM-DD`)
  - **Time format:** Moment.js format (e.g., `HH:mm`)

**Available tokens:**
```markdown
{{title}}    ← current note title
{{date}}     ← today's date (per date format setting)
{{time}}     ← current time (per time format setting)
```

**Insert template:** `Ctrl/Cmd+P` → "Templates: Insert template"

---

## Unique Note Creator
- Creates notes with a **Zettelkasten-style** timestamped ID as the filename
- Settings → Unique Note Creator:
  - **Prefix format:** Moment.js format (e.g., `YYYYMMDDHHmm`)
- Command: "Unique Note Creator: Create new unique note"
- Example output filename: `202503111045 My Note Title.md`

---

## Web Viewer
- Opens web pages inside Obsidian panes
- Command: "Web Viewer: Open web page"
- Enter any URL to load it in a pane
- Useful for keeping references alongside notes

---

## Word Count
- Displayed in the status bar (bottom of the window)
- Shows word count and character count for the current note
- Some configurations also show total vault word count

---

## Workspaces
- Save entire pane layouts as named workspaces
- **Save:** Command Palette → "Workspaces: Save layout as..."
- **Load:** Command Palette → "Workspaces: Open..." → select workspace name
- Saves: open files, pane arrangement, sidebar state, active panels
- Useful for switching between "writing mode", "review mode", "GTD mode", etc.