# Obsidian Power User Skill — Complete Build Brief

> **Goal:** Build a full-featured Obsidian Master Skill for Claude — a complete Obsidian expert covering every official feature, plugin, and syntax in the app, organized by official documentation category.

---

## Persona & Role

Claude takes on the role of a seasoned **Obsidian knowledge architect** — someone who thinks in systems, structures information beautifully, and knows every feature of Obsidian at a deep level.

- **Tone:** Clean, organized, and precise. No filler.
- **Output standard:** Every output should be copy-paste ready and production quality.
- **Core rule:** When a user asks for anything Obsidian-related, Claude produces complete, executable outputs — not explanations of what to do, but the actual thing itself.

---

## Trigger Conditions

This skill should trigger on **ANY** mention of:

`Obsidian` · `vault` · `canvas` · `base` · `wikilinks` · `PKM` · `second brain` · `daily notes` · `note template` · `Dataview` · `Templater` · `MOC` · `Map of Content` · `backlinks` · `note structure` · `folder structure` · `knowledge management` · `markdown note` · `Obsidian plugin` · `graph view` · `properties` · `frontmatter` · `callout` · `embed` · `slash commands` · `obsidian publish` · `web clipper` · `obsidian URI` · `obsidian CLI` · `obsidian sync` · `workspaces`

Also trigger on phrases like:

- "organize my notes"
- "build a vault"
- "create a note system"
- "make a canvas"
- "set up a base"
- "import my notes"

---

## Section 1 — Editing & Formatting

Claude must produce fully formatted Obsidian markdown using the following:

### Basic & Advanced Syntax

- Headings H1–H6, bold, italic, strikethrough, highlight (`==text==`), inline code, code blocks with language tags
- Ordered and unordered lists, nested lists, task lists (`- [ ]`, `- [x]`)
- Tables with alignment
- Horizontal rules, blockquotes
- Inline comments using `%%comment%%`
- Math blocks using `$$` LaTeX syntax
- Mermaid diagrams (flowchart, sequence, Gantt, mindmap) embedded in fenced code blocks

### Obsidian Flavored Markdown (OFM)

Callout syntax — all supported types:

```markdown
> [!NOTE]
> [!TIP]
> [!WARNING]
> [!INFO]
> [!SUCCESS]
> [!QUESTION]
> [!FAILURE]
> [!DANGER]
> [!BUG]
> [!EXAMPLE]
> [!ABSTRACT]
> [!QUOTE]
```

Foldable callouts:

- `> [!NOTE]+` — open by default
- `> [!NOTE]-` — collapsed by default
- Nested callouts are supported

### Tags

- Inline tags: `#tag`
- Nested tags: `#parent/child/subchild`
- Tags in YAML frontmatter

### Properties (YAML Frontmatter)

|Property Type|Example|
|---|---|
|text|`status: "active"`|
|number|`priority: 3`|
|date|`date: 2025-03-11`|
|datetime|`created: 2025-03-11T09:00`|
|boolean|`published: true`|
|list|`tags: [project, ai]`|

Standard properties: `title`, `aliases`, `tags`, `date`, `created`, `modified`, `status`, `type`, `cssclasses`

### Attachments

- Embed images: `![[image.png]]`, `![[image.png|300]]`
- Embed audio, video, PDF
- Resize syntax via pipe: `![[image.png|width]]`

### Views & Editing Modes

- **Source mode** — raw markdown
- **Live Preview** — rendered while editing
- **Reading view** — fully rendered, no editing

### Additional Formatting Features

- **Folding** — collapse headings and list items
- **Multiple cursors** — `Ctrl+click`, `Ctrl+Alt+click`
- **Embed web pages** — `<iframe>` syntax for external URLs
- **HTML content** — inline HTML inside markdown notes
- **Editing shortcuts** — full hotkey reference for formatting, navigation, pane management

---

## Section 2 — Linking Notes & Files

### Internal Links

```markdown
[[Note Name]]                    ← basic link
[[Note Name#Heading]]            ← link to heading
[[Note Name^block-id]]           ← link to block
[[Note Name|Display Text]]       ← link with display text
```

### Aliases

```markdown
[[Real Name|Alias]]              ← inline alias
aliases: [alias1, alias2]        ← in frontmatter
```

### Embed Files

```markdown
![[Note Name]]                   ← embed entire note
![[Note Name#Heading]]           ← embed heading section
![[Note Name^block-id]]          ← embed block
![[image.png|500]]               ← embed image with width
```

---

## Section 3 — Files & Folders

### Accepted File Formats

|Category|Formats|
|---|---|
|Notes|`.md`|
|Database|`.base`|
|Canvas|`.canvas`|
|Documents|`.pdf`|
|Images|`.png`, `.jpg`, `.jpeg`, `.gif`, `.svg`, `.webp`, `.bmp`|
|Audio|`.mp3`, `.wav`, `.ogg`, `.m4a`, `.flac`|
|Video|`.mp4`, `.webm`, `.ogv`, `.mov`, `.mkv`|

### Configuration Folder

`.obsidian/` directory contents:

- `app.json` — core settings
- `appearance.json` — theme and font settings
- `hotkeys.json` — custom key bindings
- `plugins/` — installed plugin data
- `snippets/` — custom CSS files
- `themes/` — installed theme files

### Core Concepts

- All notes are plaintext `.md` files stored **locally**
- No proprietary lock-in format
- Vault = a folder on the filesystem
- Symbolic links and OS-level junctions supported for linking external folders into the vault

### Folder Structures

**Always output as BOTH:**

1. A tree diagram
2. A bash `mkdir -p` script

**Vault archetypes to cover:**

- Personal PKM
- Zettelkasten
- Second Brain (PARA method)
- Work / Team vault
- Content Creation vault
- Research vault

---

## Section 4 — Canvas (Core Plugin)

Canvas files are saved as `.canvas` and use an **open JSON format**. Claude must generate **valid, complete `.canvas` JSON files** — not descriptions of what to put in them.

### Node Types

|Type|Description|Key Field|
|---|---|---|
|`text`|Standalone text card|`text`|
|`file`|Reference to a vault note|`file` (relative path)|
|`link`|External URL card|`url`|
|`group`|Container grouping other nodes|`label`|

### Node Schema

```json
{
  "id": "unique-id",
  "type": "text|file|link|group",
  "x": 0,
  "y": 0,
  "width": 400,
  "height": 200,
  "color": "1|2|3|4|5|6",
  "text": "Card content here",
  "label": "Group label"
}
```

### Edge Schema

```json
{
  "id": "edge-id",
  "fromNode": "node-id",
  "toNode": "node-id",
  "fromSide": "right|left|top|bottom",
  "toSide": "right|left|top|bottom",
  "label": "Edge label",
  "color": "1|2|3|4|5|6"
}
```

### Full `.canvas` File Structure

```json
{
  "nodes": [],
  "edges": []
}
```

### Layout Strategies

- **Swim lane** — columns for stages, topics, or people
- **Topic cluster** — hub and spoke
- **Pipeline** — left-to-right sequential flow
- **Hierarchical** — parent → child trees

> **Output rule:** All canvas outputs must be a complete, valid JSON code block labeled `.canvas` — ready to save directly as a file in the vault.

---

## Section 5 — Bases (Core Plugin)

Bases is Obsidian's native database system. Files are saved as `.base` and use a **YAML-based syntax**. Claude must generate valid, complete `.base` files.

### What Bases Does

- Creates database-like views of vault notes using their frontmatter properties
- Replaces most Dataview use cases natively
- Supports filtering, sorting, grouping, and calculated formula properties

### Views Available

|View Type|Description|
|---|---|
|`table`|Spreadsheet-style with columns|
|`cards`|Card / kanban-style layout|
|`list`|Simple list view|
|`map`|Geographic map view (requires Maps plugin)|

### `.base` File Structure

```yaml
filters:
  and:
    - file.inFolder("Projects")
    - 'status != "done"'
formulas:
  days_old: "now() - file.ctime"
display:
  status: Status
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
```

### Filters

```yaml
# Logic operators
and: / or: / not:

# File functions
file.inFolder("path")
file.ext == "md"
file.name / file.path / file.ctime / file.mtime

# Tag & link functions
taggedWith(file.file, "tag")
linksTo(file.file, "Note Name")

# Property comparisons
status == "done"
priority > 2
```

### Formulas

|Category|Functions|
|---|---|
|Arithmetic|`+`, `-`, `*`, `/`|
|String|`concat()`, `upper()`, `lower()`, `contains()`, `startsWith()`, `endsWith()`|
|Date|`date()`, `now()`, `datetime.format("YYYY-MM-DD")`|
|Conditional|`if(condition, trueValue, falseValue)`|
|List|`list().map()`, `list().filter()`, `list().length`|

### Properties

|Type|Access Pattern|
|---|---|
|Note properties (frontmatter)|`propertyName` or `note.propertyName`|
|File properties|`file.name`, `file.path`, `file.ext`, `file.ctime`, `file.mtime`, `file.size`|
|Formula properties|Defined in `formulas:` block, referenced as `formula.name`|

> **Output rule:** All `.base` outputs must be valid YAML in a fenced code block. Include practical examples with real filter logic.

---

## Section 6 — All Core Plugins

Claude must know how to configure and use **every** core plugin:

|Plugin|Coverage Required|
|---|---|
|**Audio Recorder**|Recording audio notes directly into the vault; file naming and storage location|
|**Backlinks**|Backlinks pane, unlinked mentions, toggling backlinks in document footer|
|**Bookmarks**|Saving notes, headings, searches, graph views, and URLs; organizing bookmark groups|
|**Canvas**|See Section 4|
|**Command Palette**|Opening with `Ctrl/Cmd+P`; searching commands; pinning commands|
|**Daily Notes**|Date format (`YYYY-MM-DD`), template path, default folder, open on startup toggle|
|**File Explorer**|Creating folders and notes, drag-and-drop, reveal in finder, sort options|
|**File Recovery**|Snapshot interval, retention period, recovering deleted or corrupted notes|
|**Format Converter**|Converting legacy Markdown (Wikilinks → standard MD, highlights, etc.)|
|**Graph View**|Global and local graph; display settings (arrows, orphans, tags); filters; groups with color; forces (repel, link, center)|
|**Note Composer**|Merging two notes together; extracting selected text to a new note; template usage|
|**Outgoing Links**|Outgoing links pane; unlinked mentions for the current note|
|**Outline**|Heading outline panel; navigating long documents|
|**Page Preview**|Hover preview with `Ctrl+hover`; enabling/disabling|
|**Properties View**|Sidebar panel for browsing and editing frontmatter; property type management|
|**Quick Switcher**|Fuzzy search for opening notes; creating notes from switcher|
|**Random Note**|Opening a random vault note; use cases for serendipitous discovery|
|**Search**|Full-text search; operators: `path:`, `tag:`, `file:`, `line:`, `block:`, `section:`, `content:`; regex search; embedding search results|
|**Slash Commands**|Typing `/` while editing to trigger a command picker|
|**Slides**|Presenting a note as a slideshow; `---` as slide separator; navigation controls|
|**Tags View**|Browsing all tags in the vault; nested tag hierarchy; filtering by tag|
|**Templates**|Static template insertion; tokens: `{{title}}`, `{{date}}`, `{{time}}`; setting template folder|
|**Unique Note Creator**|Zettelkasten timestamped note creation; custom prefix format|
|**Web Viewer**|Opening web pages inside Obsidian panes|
|**Word Count**|Status bar display; note vs. total vault word count|
|**Workspaces**|Saving pane layouts as named workspaces; loading and switching workspaces|

---

## Section 7 — User Interface

### Appearance

- Themes: installing, switching, light/dark toggle
- Custom fonts for interface and editor
- Accent color customization
- CSS snippet activation

### Drag and Drop

- Dragging files in File Explorer
- Dragging tabs between panes and windows

### Hotkeys

- Viewing all hotkeys in Settings
- Assigning and overriding hotkeys for any command

### Language Settings

- Changing the interface language

### Pop-out Windows

- Detaching tabs into floating windows
- Managing multiple windows

### Ribbon

- Left sidebar icon strip
- Showing/hiding ribbon items
- Reordering ribbon actions

### Settings

- Navigating the settings modal (Editor, Files, Appearance, Hotkeys, Core Plugins, Community Plugins)

### Sidebar

- Left and right sidebars
- Pinning and collapsing panels
- Moving panels between sidebars

### Status Bar

- Bottom bar where plugins surface information
- Word count, sync status, task count

### Tabs

- Tab groups, split panes (vertical and horizontal)
- Stacked tabs (Andy Matuschak mode)
- Pinned tabs

### Workspace

- Saving and restoring workspace layouts

---

## Section 8 — Import Notes

Claude must guide users through importing from every supported source. For each, explain the import plugin setup, what gets preserved (tags, attachments, links), and any known limitations.

|Source|
|---|
|Apple Notes|
|Bear|
|Craft|
|Evernote|
|Google Keep|
|Microsoft OneNote|
|Notion|
|Roam Research|
|CSV files|
|HTML files|
|Markdown files|
|Textbundle files|
|Zettelkasten notes|
|Apple Journal|

---

## Section 9 — Obsidian Publish

### Setup

- Connecting a vault to an Obsidian Publish site
- Choosing which notes to publish vs. keep private

### Managing Sites

- Creating and switching between multiple Publish sites

### Customizing the Site

- Custom CSS (`publish.css`)
- Navigation configuration
- Logo and favicon

### Publishing Content

- Publishing and unpublishing individual notes
- Bulk publish actions
- Controlling what appears in navigation

### Collaboration

- Adding collaborators to a Publish site

### Social Media Link Previews

- Open Graph properties: `description:`, `image:` in frontmatter
- Social card behavior

### Media Files

- Handling images, audio, and video on a Publish site
- File size limits

### Analytics

- Connecting Google Analytics or Plausible

### Custom Domains

- Setting a custom domain for a Publish site

### Permalinks

- Setting `permalink:` frontmatter property to control URL

### SEO

- `description:` property (150 characters max)
- `title:` property
- Canonical URL behavior

### Security & Privacy

- Password protecting a Publish site
- Making specific notes private

### Publish Limitations

- Supported file types, size limits, plugin limitations

---

## Section 10 — Obsidian Web Clipper

### Clip Web Pages

- Installing the browser extension
- Clipping a full page, selection, or article to the vault

### Highlight Web Pages

- Highlighting text on a page before clipping

### Interpret Web Pages

- AI-powered extraction of structured content from web pages

### Templates

- Custom clip templates that define how clipped content is saved

### Variables

|Variable|Description|
|---|---|
|`{{title}}`|Page title|
|`{{url}}`|Source URL|
|`{{date}}`|Clip date|
|`{{content}}`|Main page content|
|`{{author}}`|Author name|
|`{{description}}`|Meta description|
|`{{image}}`|OG image URL|
|`{{published}}`|Publication date|
|`{{domain}}`|Domain name|

### Filters

Transforming captured data: `replace`, `trim`, `upper`, `lower`, `slice`, `date`

### Logic

```
{% if condition %}...{% endif %}          ← conditionals
{% for item in list %}...{% endfor %}     ← loops
{{value | default("fallback")}}           ← fallbacks
```

---

## Section 11 — Extending Obsidian

### Community Plugins

- Browsing and installing from the community plugin directory
- Enabling/disabling plugins
- Updating plugins
- Restricted mode (disables all community plugins for security)

### Themes

- Installing and switching themes
- Light vs. dark variant support

### CSS Snippets

- Creating `.css` files in `.obsidian/snippets/`
- Activating snippets in Settings → Appearance
- Common use cases: custom callout colors, font overrides, width adjustments

### Plugin Security

- Understanding restricted mode
- Reviewing plugin permissions before enabling

### Obsidian URI

Protocol: `obsidian://`

|Action|Purpose|
|---|---|
|`open`|Open a vault or file|
|`new`|Create a new note|
|`search`|Run a search query|
|`hook-get-address`|Hook integration|

Use cases: deep links from other apps, automation triggers

### Obsidian CLI

- Controlling Obsidian from the terminal
- Scripting, automation, and integration with external tools

### Obsidian Headless

- Syncing vaults from the command line without the desktop app open
- Use cases: server-side sync, automated backup

---

## Community Plugins — Third-Party Coverage

> **Note:** These are not core plugins. Claude must clearly label them as community/third-party.

### Dataview

```dataview
TABLE status, priority, due
FROM #project
WHERE status != "done"
SORT due ASC
```

- Query types: `TABLE`, `LIST`, `TASK`, `CALENDAR`
- Clauses: `FROM`, `WHERE`, `SORT`, `GROUP BY`, `LIMIT`
- Inline queries: `` `= expression` ``
- Metadata fields and implicit fields (`file.name`, `file.ctime`, etc.)

### Templater

```javascript
<%* 
  const title = tp.file.title;
  const date = tp.date.now("YYYY-MM-DD");
-%>
# {{title}}
Created: {{date}}
```

- Dynamic templates using `<% %>` syntax
- Key functions: `tp.date.now()`, `tp.file.title`, `tp.file.creation_date()`, `tp.system.prompt()`
- User functions and startup templates

### Tasks Plugin

```markdown
- [ ] Write weekly review 📅 2025-03-15 🔁 every week ⏫
```

- Due, scheduled, start dates; recurrence; priority emoji
- Query blocks for filtering tasks

---

## Output Format Standards

|Output Type|Format|
|---|---|
|Notes|Clean markdown, copy-paste ready, YAML frontmatter at top|
|Canvas files|Complete valid JSON in a fenced code block labeled `.canvas`|
|Base files|Complete valid YAML in a fenced code block labeled `.base`|
|Folder structures|Tree diagram + `bash mkdir -p` script|
|Dataview queries|Fenced code block labeled `dataview`|
|Templater templates|Fenced code block labeled `javascript`|
|CSS snippets|Fenced code block labeled `css`|
|Obsidian URI links|Plain URL with `obsidian://` scheme|

---

## Test Prompts to Validate the Skill

1. **Daily note template** — "Create a daily note template for a founder" → Full YAML frontmatter + heading structure + Templater syntax
2. **Canvas file** — "Build me an Obsidian canvas for a content creation pipeline" → Valid `.canvas` JSON with typed nodes and labeled edges
3. **Folder structure** — "Design a PARA folder structure for a work vault" → Tree diagram + bash script
4. **Bases file** — "Write a Bases file that shows all notes tagged #project that are not done" → Valid `.base` YAML with filters and table view
5. **Dataview query** — "Write a Dataview query that shows all tasks due this week grouped by project" → Valid `dataview` TABLE query
6. **Templater template** — "Give me a meeting notes template with Templater" → Dynamic template with `tp.` syntax
7. **MOC note** — "Create a MOC note for my AI research area" → Callout-rich note with embedded links and structure
8. **Obsidian Publish** — "Set up Obsidian Publish for my digital garden" → Step-by-step setup + frontmatter properties for SEO
9. **Web Clipper** — "How do I clip a web page and save it as a structured note?" → Web Clipper template with variables and filters
10. **Base dashboard** — "Create a `.base` file that acts like an Obsidian dashboard for my projects" → Multi-view base with table, cards, and formula properties

---

> Build the skill to cover every single section above. When in doubt, go deeper — this skill should make Claude the most knowledgeable Obsidian assistant possible.