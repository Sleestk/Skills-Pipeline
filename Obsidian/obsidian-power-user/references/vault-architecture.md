# Obsidian — Vault Architecture, UI & Extensions Reference

---

## Vault Architecture Fundamentals

- A **vault** is any folder on the local filesystem
- All notes are plaintext `.md` files — no proprietary lock-in
- The `.obsidian/` folder holds all configuration:

```
.obsidian/
├── app.json           ← core settings
├── appearance.json    ← theme, fonts, accent color
├── hotkeys.json       ← custom key bindings
├── plugins/           ← installed plugin data and settings
├── snippets/          ← custom CSS files (activate in Settings → Appearance)
└── themes/            ← installed theme files
```

- Symbolic links and OS-level junctions supported for linking external folders
- Vault can be synced via Obsidian Sync, iCloud, Dropbox, or any cloud file system

---

## Folder Structures — Vault Archetypes

Always output as **both** a tree diagram and a `bash mkdir -p` script.

---

### Personal PKM Vault

```
Personal/
├── Inbox/             ← quick capture, process daily
├── Notes/
│   ├── Areas/         ← ongoing responsibilities
│   ├── Resources/     ← topics and references
│   └── Archive/       ← inactive material
├── Daily/             ← daily notes
├── Templates/
└── Attachments/
```

```bash
mkdir -p Personal/{Inbox,"Notes/Areas","Notes/Resources","Notes/Archive",Daily,Templates,Attachments}
```

---

### Zettelkasten Vault

```
Zettelkasten/
├── Fleeting/          ← quick raw captures
├── Literature/        ← notes from sources (books, articles, videos)
├── Permanent/         ← atomic concept notes with IDs (e.g., 202503111045)
├── Structure/         ← MOC and index notes
├── Templates/
└── Attachments/
```

```bash
mkdir -p Zettelkasten/{Fleeting,Literature,Permanent,Structure,Templates,Attachments}
```

---

### PARA Second Brain

```
PARA/
├── 1-Projects/        ← active, outcome-driven work
├── 2-Areas/           ← ongoing responsibilities with no end date
├── 3-Resources/       ← topics and references for future use
├── 4-Archive/         ← completed or inactive items
├── Inbox/
├── Daily/
├── Templates/
└── Attachments/
```

```bash
mkdir -p PARA/{"1-Projects","2-Areas","3-Resources","4-Archive",Inbox,Daily,Templates,Attachments}
```

---

### Work / Team Vault

```
Work/
├── Projects/
│   ├── Active/
│   └── Archive/
├── Meetings/
├── People/            ← one note per team member or stakeholder
├── Areas/             ← ongoing departments or functions
├── Resources/         ← docs, runbooks, references
├── Daily/
├── Templates/
└── Attachments/
```

```bash
mkdir -p Work/{"Projects/Active","Projects/Archive",Meetings,People,Areas,Resources,Daily,Templates,Attachments}
```

---

### Content Creation Vault

```
Content/
├── Ideas/             ← raw video/post ideas
├── Research/          ← source material per topic
├── Scripts/           ← full scripts per video
├── Production/        ← shoot logs, editor briefs, b-roll notes
├── Published/         ← archive of completed content
├── MOCs/              ← topic Maps of Content
├── Daily/
├── Templates/
└── Attachments/
```

```bash
mkdir -p Content/{Ideas,Research,Scripts,Production,Published,MOCs,Daily,Templates,Attachments}
```

---

### Research Vault

```
Research/
├── Literature/        ← papers, books, articles
├── Notes/             ← atomic concept notes
├── Experiments/       ← methodology, results, logs
├── Figures/           ← charts, images, diagrams
├── Writing/           ← drafts and papers
├── MOCs/              ← index notes per topic
├── Templates/
└── Attachments/
```

```bash
mkdir -p Research/{Literature,Notes,Experiments,Figures,Writing,MOCs,Templates,Attachments}
```

---

## Maps of Content (MOC)

An MOC is an index note that organizes related notes by theme. It doesn't hold primary content — it links to it.

**Standard MOC structure:**
```markdown
---
type: MOC
topic: "AI Research"
tags: [MOC, ai]
created: 2025-03-11
---

# 🗺️ AI Research — Map of Content

> [!ABSTRACT] Overview
> This MOC organizes all notes related to AI research, tooling, and applications.

## 📌 Core Concepts
- [[Large Language Models]]
- [[Attention Mechanism]]
- [[Transformer Architecture]]

## 🔬 Research Papers
- [[Attention Is All You Need]]
- [[GPT-4 Technical Report]]

## 🛠️ Tools & Frameworks
- [[LangChain Overview]]
- [[Ollama Setup Guide]]

## 📅 Recent Notes
```dataview
LIST
FROM [[]]
SORT file.mtime DESC
LIMIT 10
```

## 🔗 Related MOCs
- [[Machine Learning MOC]]
- [[Software Engineering MOC]]
```

---

## User Interface Reference

### Appearance
- **Themes:** Settings → Appearance → "Manage" → browse and install; toggle light/dark
- **Custom fonts:** Settings → Appearance → Interface font / Text font / Monospace font
- **Accent color:** Settings → Appearance → Accent color picker
- **CSS snippets:** Drop `.css` files into `.obsidian/snippets/` → toggle in Settings → Appearance → CSS snippets

### Sidebar
- Left and right sidebars toggle with the sidebar icons in the header
- Panels can be dragged between sidebars
- Panels can be pinned (keep open) or collapsed

### Tabs
- Open multiple notes in tabs; drag tabs to reorder
- **Split pane:** Right-click tab → "Split right" / "Split down"
- **Stacked tabs (Andy Matuschak mode):** Enable in Settings → Editor → "Stack tabs"
- **Pinned tabs:** Right-click tab → "Pin" — tab stays open even when switching notes

### Pop-out Windows
- Right-click tab → "Move to new window" to detach into floating window
- Supports multiple independent windows simultaneously

### Hotkeys
- View all: Settings → Hotkeys
- Assign custom hotkeys to any command
- Common defaults:
  - `Ctrl+P` — Command Palette
  - `Ctrl+O` — Quick Switcher
  - `Ctrl+G` — Graph View
  - `Ctrl+Shift+F` — Search
  - `Ctrl+E` — Toggle reading/editing view
  - `Ctrl+,` — Settings

### Ribbon
- Left sidebar icon strip for quick access to plugins
- Right-click ribbon icon to hide it
- Reorder icons by dragging (in some versions)

### Status Bar
- Bottom bar displaying plugin-surfaced info: word count, sync status, task count
- Plugins can add their own indicators here

---

## Import Sources

Guide users through importing from each source using the **Importer plugin** (Settings → Community Plugins → search "Importer").

| Source | Notes |
|---|---|
| **Apple Notes** | Export as HTML; Importer converts to MD. Attachments preserved. |
| **Bear** | Export `.bear2bk` bundle; Importer handles it. Tags and links preserved. |
| **Craft** | Export as Markdown zip; mostly clean. Internal links need review. |
| **Evernote** | Export `.enex`; Importer handles. Attachments, tags, notebooks preserved. Known issues: complex tables may degrade. |
| **Google Keep** | Export via Google Takeout (`.json`); Importer converts. Labels → tags. |
| **Microsoft OneNote** | Export via OneNote app as `.docx` per section; Importer converts. Images preserved. |
| **Notion** | Export as Markdown & CSV zip; Importer handles. Databases become MD files + CSVs. |
| **Roam Research** | Export as JSON; Importer converts. Block references partially preserved. |
| **CSV files** | Importer creates one note per row, with column headers as frontmatter. |
| **HTML files** | Importer converts to Markdown. Useful for web archive imports. |
| **Markdown files** | Drag into vault folder directly. Run Format Converter if syntax differs. |
| **Textbundle** | Importer handles `.textbundle` and `.textpack` formats. |
| **Zettelkasten notes** | Import as Markdown files. Use Unique Note Creator for ID continuity. |
| **Apple Journal** | Export via Shortcuts; community plugins for direct import exist. |

---

## Obsidian URI

Protocol: `obsidian://`

| Action | URI | Description |
|---|---|---|
| Open vault | `obsidian://open?vault=VaultName` | Opens a specific vault |
| Open note | `obsidian://open?vault=VaultName&file=Note%20Name` | Opens a specific note |
| Create note | `obsidian://new?vault=VaultName&name=Title&content=Body` | Creates a new note |
| Search | `obsidian://search?vault=VaultName&query=search+term` | Opens search with query |
| Hook integration | `obsidian://hook-get-address` | Returns current note address for Hook app |

**URL-encode spaces as `%20` or `+`.**

Use cases:
- Deep links from other apps (Notion, Craft, email, etc.)
- Automation triggers (Shortcuts, Alfred, Raycast)
- Linking to specific notes from external tools

---

## Obsidian CLI & Headless

**Obsidian CLI** (community tool `obsidian-cli`):
- Control Obsidian from the terminal
- Open vaults, create notes, run commands via shell scripts
- Install via npm: `npm install -g obsidian-cli`

**Obsidian Headless Sync:**
- Sync vaults from command line without the Obsidian desktop app open
- Use case: server-side vault sync, automated backups, CI/CD pipelines
- Requires Obsidian Sync subscription
- Run: `obsidian-sync --vault /path/to/vault`

---

## CSS Snippets — Common Use Cases

```css
/* Wider note width */
.markdown-preview-view,
.markdown-source-view {
  max-width: 900px;
  margin: 0 auto;
}

/* Custom callout color */
.callout[data-callout="custom"] {
  --callout-color: 100, 200, 150;
  --callout-icon: lucide-star;
}

/* Hide frontmatter in reading view */
.markdown-preview-view .metadata-container {
  display: none;
}

/* Bigger headings */
.markdown-preview-view h1 {
  font-size: 2.2em;
}

/* Colored tags */
.tag[href="#project"] {
  background-color: #4a90d9;
  color: white;
}
```

Activate snippets: Settings → Appearance → CSS Snippets → toggle on/off.