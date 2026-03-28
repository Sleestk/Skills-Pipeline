# Obsidian — Linking Notes & Files Reference

## Internal Links (Wikilinks)

```markdown
[[Note Name]]                    ← link to a note
[[Note Name#Heading]]            ← link to a specific heading
[[Note Name#^block-id]]          ← link to a specific block
[[Note Name|Display Text]]       ← link with custom display text
[[Note Name#Heading|Short]]      ← heading link with alias
```

## Aliases

**Inline alias:**
```markdown
[[Real Note Name|My Alias]]
```

**Frontmatter aliases:**
```yaml
---
aliases:
  - Short Name
  - Another Alias
---
```

When you type `[[` and search, aliases appear alongside note names.

## Embedding Files

```markdown
![[Note Name]]               ← embed entire note
![[Note Name#Heading]]       ← embed from a specific heading
![[Note Name#^block-id]]     ← embed a specific block
![[image.png]]               ← embed image (full size)
![[image.png|400]]           ← embed image with pixel width
![[image.png|400x300]]       ← embed with width and height
![[audio.mp3]]               ← embed audio player
![[video.mp4]]               ← embed video player
![[document.pdf]]            ← embed PDF viewer
![[document.pdf#page=3]]     ← embed PDF opened to page 3
```

## Block References

**Creating a block ID:**
Add `^block-id` at the end of a paragraph (no heading needed):
```markdown
This is my important paragraph. ^my-block-id
```

**Linking to it:**
```markdown
[[Note Name#^my-block-id]]
```

**Embedding it:**
```markdown
![[Note Name#^my-block-id]]
```

Block IDs must be alphanumeric with hyphens. They are placed at the end of the block line.

## Link Resolution

Obsidian uses **shortest path that uniquely identifies the file**:
- If `project-notes.md` exists in only one folder: `[[project-notes]]`
- If multiple files share a name, use path: `[[Folder/project-notes]]`

**Relative paths:**
```markdown
[[./sibling-note]]          ← same folder
[[../parent-folder/note]]   ← one level up
```

## Unlinked Mentions

Found in the **Backlinks** or **Outgoing Links** panel — Obsidian detects text that matches a note name but hasn't been linked yet. Use to discover connection opportunities.

## Backlinks

Backlinks panel shows all notes that link **to** the current note. Available:
- As a sidebar panel
- As an inline section at the bottom of the note (toggle in Backlinks plugin settings)

## Link Formats — Standard vs. Wikilink

| Format | Syntax | Notes |
|---|---|---|
| Wikilink (Obsidian default) | `[[Note Name]]` | Shorter, Obsidian-native |
| Markdown link | `[Display](path/to/note.md)` | Standard MD, portable |

To switch between formats: Settings → Files → "Use [[Wikilinks]]" toggle.