# Obsidian — Publish & Web Clipper Reference

---

## Obsidian Publish

Obsidian Publish hosts selected vault notes as a public website.

### Setup

1. Settings → Obsidian Publish → "Connect"
2. Sign in to your Obsidian account
3. Create a new site (or connect to an existing one)
4. Select which notes to publish from the "Publish changes" panel

### Managing Sites

- Multiple Publish sites are supported per account
- Switch between sites in Settings → Obsidian Publish → site selector
- Each site has its own domain and publish settings

### Publishing Notes

**Publish a note:** Open the publish panel (ribbon icon) → select note → click "Publish"

**Control via frontmatter:**
```yaml
---
publish: true        ← explicitly publish this note
publish: false       ← keep private even if selected
---
```

**Bulk publish:** Select all modified notes in the publish panel → "Publish all"

### Customization

**Custom CSS:**
- Create `publish.css` in vault root
- It applies globally to the published site

**Navigation:**
- Notes appear in navigation if they're published and have no `publish: false`
- Control order with frontmatter: `nav_order: 1`
- Exclude from nav: `nav_exclude: true`

**Logo/Favicon:**
- Upload via site settings in Obsidian Publish dashboard

### Frontmatter Properties for Publish

```yaml
---
publish: true
title: "Custom Page Title"
description: "Meta description for SEO (max 150 chars)"
permalink: /my-custom-url
image: "https://example.com/og-image.jpg"
nav_order: 3
nav_exclude: false
---
```

### SEO

```yaml
description: "Short description under 150 characters for search engines"
title: "Custom title (overrides note name in browser tab and OG cards)"
```

- Canonical URLs are set automatically based on the note path or `permalink`
- Open Graph cards use `description:` and `image:` properties

### Social Media Link Previews

For rich previews when sharing on X, LinkedIn, etc.:
```yaml
---
description: "The description that appears in link previews"
image: "https://your-domain.com/path/to/image.jpg"
---
```

### Analytics

- Google Analytics: enter tracking ID in Publish site settings
- Plausible: enter domain in Publish site settings

### Custom Domains

1. In Obsidian Publish dashboard → "Custom domain"
2. Add a CNAME record in your DNS pointing to `publish-main.obsidian.md`
3. Enter the domain in Publish settings and verify

### Security & Privacy

**Password protection:**
- Settings → Obsidian Publish → "Site options" → "Password protect"
- Entire site is protected by a single password

**Private notes:**
- Set `publish: false` in frontmatter, or simply don't publish the note
- Linked but unpublished notes appear as plain text (not hyperlinks)

### Collaboration

- Add collaborators in Publish site settings (requires their Obsidian account email)
- Collaborators can publish and manage notes but not change site settings

### Publish Limitations

- Community plugins do not run on published sites
- Interactive features (like Dataview queries) do not execute — only static content is shown
- Supported file types: `.md`, images (`.png`, `.jpg`, `.gif`, `.svg`, `.webp`), `.pdf`
- File size limit: 100MB per file
- Canvas and Bases files not published as interactive components

---

## Obsidian Web Clipper

Web Clipper is a **browser extension** that saves web content directly to your vault.

### Installation

Install the Obsidian Web Clipper extension from:
- Chrome Web Store
- Firefox Add-ons
- Safari Extensions

### Clip Modes

| Mode | Description |
|---|---|
| **Article** | Extracts main article content, strips nav/ads |
| **Full page** | Captures entire page content |
| **Selection** | Clips only highlighted/selected text |
| **Highlight** | Saves highlighted text with context |

### Variables

Use these in clip templates to insert captured data:

| Variable | Description |
|---|---|
| `{{title}}` | Page title |
| `{{url}}` | Source URL |
| `{{date}}` | Date clipped (format: `YYYY-MM-DD`) |
| `{{time}}` | Time clipped |
| `{{content}}` | Main page content |
| `{{author}}` | Author name |
| `{{description}}` | Meta description |
| `{{image}}` | Open Graph image URL |
| `{{published}}` | Publication date |
| `{{domain}}` | Domain name (e.g., `nytimes.com`) |
| `{{favicon}}` | Site favicon URL |
| `{{highlights}}` | Highlighted text selections |
| `{{comments}}` | Comments added during clipping |

### Filters

Apply to variables with a pipe:

```
{{title | upper}}                         ← UPPERCASE
{{title | lower}}                         ← lowercase
{{date | date:"YYYY-MM-DD"}}             ← format date
{{content | replace:"foo","bar"}}         ← replace text
{{description | trim}}                    ← remove whitespace
{{title | slice:0,50}}                    ← first 50 chars
{{content | default:"No content found"}} ← fallback value
```

### Logic

```
{% if author %}By {{author}}{% endif %}
{% for highlight in highlights %}
- {{highlight.text}}
{% endfor %}
{{value | default("Unknown")}}
```

### Example Clip Template

```markdown
---
title: "{{title}}"
url: "{{url}}"
author: "{{author | default("Unknown")}}"
date: {{date}}
published: {{published}}
domain: {{domain}}
tags: [clipped, read-later]
status: unread
---

# {{title}}

> Source: [{{domain}}]({{url}})
> Author: {{author | default("Unknown")}} | Published: {{published | default("Unknown")}}

## Content

{{content}}

{% if highlights %}
## Highlights

{% for highlight in highlights %}
> {{highlight.text}}
{% if highlight.comment %}*{{highlight.comment}}*{% endif %}

{% endfor %}
{% endif %}
```

### AI Interpret Mode

- Uses AI to extract structured data from the page
- Define what fields to extract in the template
- Output is parsed into frontmatter properties automatically
- Useful for structured research: extracting author, key claims, dates, entities

### Multiple Vault Support

- Configure which vault to clip into per template
- Default vault is set in extension settings
- Can prompt to choose vault on each clip