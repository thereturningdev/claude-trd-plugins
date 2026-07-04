---
name: explain-in-html
description: Use when the user asks you to explain a topic in HTML — typically something from the current Claude Code session, such as a requirement, a plan, or the work just done. Produces a step-by-step, logically ordered explanation as an HTML document (via the create-html skill), then hardens it against illogical ordering, vague terms, AI-slop wording, and missing diagrams before it reaches the user.
---

# explain-in-html — Explain a topic as a clean HTML document

The user gives a topic to explain. Usually the topic refers to work in the
current session (for example "explain the requirement", "explain the work you
just did", "explain how this feature works"). Find the relevant material in the
conversation context, write the explanation, and deliver it as an HTML file.

## Step 1 — Write the explanation

- Identify what the user wants explained. Pull the facts from the session
  context (the requirement document, the implementation plan, the code you
  changed, the decisions made). Do not invent details.
- Write the explanation as a **step-by-step logical sequence**: start at the
  beginning, end at the end, each step following from the one before it.
- Produce the HTML by invoking the **create-html** skill. Pass the topic as the
  title and the explanation as the content.

## Step 2 — Mandatory review passes (before sending to the user)

Run every pass below on the drafted explanation. Rewrite until each passes.
Do not send the explanation to the user until all four pass.

### 1. Logical-sequence review
Read the explanation from beginning to end. Confirm each step follows logically
from the previous one, with no gaps or jumps. If the sequence is not logical
start to finish, rewrite it so it is a single coherent progression.

### 2. Precision-of-terms review — zero tolerance for vague terms
Scan for vague nouns such as "file", "service", "repository", "feature",
"component", "module", "system", "the code", "the function". Every one must be
replaced with the specific name it refers to — the exact file path, the exact
service name, the exact repository, the exact feature. If you cannot name the
specific thing, find it in the context first; do not leave the vague term.

### 3. AI-slop review
Remove writing that reads as AI-generated. Rewrite any sentence that contains:
- em dashes (—)
- uncommon or inflated vocabulary
- corporate jargon and buzzwords (e.g. "leverage", "streamline", "seamless",
  "robust", "delve", "elevate", "unlock", "empower", "holistic", "synergy")
Replace them with plain, common words.

### 4. Mermaid-diagram review
If the explanation contains a sequence of steps, or processes that run in
parallel or in sequence, add a **Mermaid diagram** that shows those steps
visually. A step-by-step explanation almost always needs at least one diagram.

## Adding Mermaid diagrams

Only add Mermaid to the document when it actually contains at least one
Mermaid diagram. If the explanation has no diagram, do not include the
`<script>` reference, the `assets/` copy, or any Mermaid markup at all.

The Mermaid rendering library is vendored with this skill at
`assets/mermaid.min.js`. It is a single self-contained JavaScript file (no
external dependencies, no network needed). Never fetch Mermaid from a CDN and
never download or execute Mermaid tooling on the fly — use the vendored file.

For each diagram, write the Mermaid markup inside a `<div class="mermaid">`
block in `{{CONTENT}}`. Reference the vendored library with a local relative
path and initialize Mermaid. Add the two scripts once per document:

```html
<div class="mermaid">
flowchart TD
  A[Step 1] --> B[Step 2] --> C[Step 3]
</div>

<script src="./assets/mermaid.min.js"></script>
<script>
  mermaid.initialize({ startOnLoad: true });
</script>
```

Because the HTML references `./assets/mermaid.min.js` relative to itself, copy
this skill's `assets/mermaid.min.js` into an `assets/` folder next to the
generated HTML file so the diagrams render. Do not fetch Mermaid from a CDN.

Lay the steps out in the same order they appear in the explanation, and use
branches for parallel or conditional paths.

## Deliver

Only after all four review passes succeed, tell the user the file is ready and
where it is.
