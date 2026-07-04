---
name: create-html
description: Use whenever you are asked to create an HTML document or page. Produces clean, minimal HTML using a bundled compact template with simple, readable styling — instead of inventing bespoke markup or heavy CSS each time.
---

# create-html — Create HTML documents from a simple template

Whenever the user asks you to create an HTML document or page, base it on the
bundled `template.html`. The template gives a compact, readable serif style out
of the box so every HTML document you produce looks consistent and clean.

## How to use

1. Read `template.html` in this skill's directory.
2. Copy it as the starting point for the new file.
3. Replace the placeholders:
   - `{{TITLE}}` — the document title (used in both `<title>` and the `<h1>`).
   - `{{CONTENT}}` — the body content as HTML.
4. Write the result to the file the user asked for.

## Rules

- **Keep the `<style>` block as-is.** Its whole point is to be compact and
  consistent. Do not expand it, swap fonts, or add frameworks unless the user
  explicitly asks for different styling.
- **Structure the content** with `<section>`, `<h2>`, `<h3>`, `<p>`, `<ul>`,
  `<table>`, etc. — the template already styles these.
- **No external dependencies.** No CDN links, remote fonts, or external
  stylesheets — keep the document self-contained.
- Only deviate from the template when the user gives a specific styling or
  layout requirement that conflicts with it.
