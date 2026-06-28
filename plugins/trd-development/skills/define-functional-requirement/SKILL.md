---
name: define-functional-requirement
description: Use when turning a rough feature idea or bug-fix request — given directly in chat or as a ticket reference (Jira, GitHub issue, or other tracker) — into a complete, structured functional requirement. Drives clarifying questions, edge-case discovery, optional HTML mockups for approval, and a final HTML requirement document saved under docs/<feature-slug>/.
---

# define-functional-requirement — Turn a Rough Idea into a Requirement

Take a loosely-described feature or bug fix and produce a well-structured
functional requirement document. The work is conversational: clarify the intent,
hunt for edge cases, optionally mock up the UI for approval, then write the doc
to `docs/<feature-slug>/`.

Do **not** rush to the document. Most of the value is in the questions asked
before it is written.

## Inputs

The rough idea arrives in one of two ways:

| Source | What you get | What to do |
| --- | --- | --- |
| **In chat** | A free-text description of the feature / bug. | Use it directly as the starting point. |
| **Ticket reference** | A tracker (Jira, GitHub issues, other) + a ticket number. | Fetch the ticket first, then treat its contents as the starting point. At the end, **rewrite the ticket** with the complete definition and **link the HTML requirement** back to it. |

To fetch a ticket, use whatever is available in this environment: the `gh` CLI
for GitHub issues (`gh issue view <num>`), a Jira MCP server or CLI, or ask the
user to paste the ticket if no tooling can reach it. Never guess the ticket's
contents.

## Feature slug

The output folder is `docs/<feature-slug>/`. Choose the slug like this:

- **Ticket given** → the slug is the ticket identifier (e.g. `PROJ-123`,
  `issue-456`).
- **Only a description given** → autonomously create a slug of 3–4 words joined
  by `-` (dash) that summarize the feature (e.g. `bulk-export-csv`,
  `fix-login-redirect-loop`). Lowercase, no spaces.

## Procedure

1. **Intake.** Read the rough idea (from chat or by fetching the ticket).
   Determine the feature slug.
2. **Clarify.** Ask the user as many questions as needed to genuinely pin down
   the feature: who it's for, the trigger, expected behavior, data involved,
   constraints, and what "done" looks like. Group questions logically rather than
   dumping a flat list; ask follow-ups based on answers. Keep going until the
   feature is unambiguous.
3. **Probe edge cases.** Explicitly ask about edge cases that matter for a
   *complete* implementation — empty/missing data, permissions, concurrency,
   failure/error states, limits, unusual inputs. Surface the ones the user
   hasn't considered.
4. **Decide on mockups.** Judge whether an HTML mockup would help:
   - **UI-facing features** → almost always yes.
   - **Pure backend, data, or bug fixes with no visible UI** → usually skip;
     say so briefly.
   When mocking up, create one or more self-contained HTML mockups — offer a few
   distinct approaches or visual choices when it's worth comparing. Present them
   and get the user to either pick one or confirm everything is okay before
   moving on.
5. **Write the requirement.** Copy `requirement-template.html` (ships next to
   this SKILL.md), fill in every `{{PLACEHOLDER}}`, and write the result to
   `docs/<feature-slug>/requirement.html`. Store any approved mockups in the same
   folder and reference them from the **Mockups** section. Confirm the path with
   the user when done.
6. **Update the ticket (only if one was provided).** When the rough idea came
   from a ticket, rewrite the ticket with the complete, clarified definition of
   the feature to build or bug to fix (replacing the rough description), and add
   a link from the ticket to the generated HTML requirement document. Use the
   same tooling used to fetch it (`gh issue edit <num>`, the Jira MCP/CLI, etc.).
   If the requirement doc lives in the repo rather than at a public URL, link to
   its path. Skip this step entirely when the idea came only from chat.

## requirement-template.html

A deliberately minimal, early-web HTML file with a tiny embedded style block
(main font, section spacing, three title levels). Replace the `{{...}}`
placeholders; the requirement lists/criteria are HTML fragments (`<ul>`, `<p>`,
etc.) you generate. Don't add external dependencies or a heavier design — the
user maintains the template's styling.

## Common mistakes

- **Jumping to the document** before the feature is actually clarified — the
  questions are the point.
- **Skipping edge cases** the user didn't raise — probe them explicitly.
- **Mocking up backend-only work** — only mock when there's a UI to see.
- **Guessing a ticket's contents** — fetch it, or ask the user to paste it.
- **Wrong slug** — ticket id when a ticket exists; otherwise a 3–4 word dashed
  summary.
