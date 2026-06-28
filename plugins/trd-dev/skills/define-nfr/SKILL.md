---
name: define-nfr
description: Use to work out the non-functional requirements for one specific feature or bug fix — given its ticket id or its feature slug. Twin of define-functional-requirement, usually run right after it (optional). Analyzes the project's code and docs, ensures project-level NFRs exist via nfr-init, asks the user to confirm the constraints the feature needs, then fills the "Non Functional Requirements" section of docs/<feature-slug>/requirement.html.
---

# define-nfr — Define the Non-Functional Requirements for a Feature or Bug Fix

Twin of `define-functional-requirement`. That skill captures *what* to build; this
one captures the non-functional constraints the work must satisfy — performance,
security, privacy, accessibility, reliability, observability, and so on — for one
specific feature or bug fix, and records them in that feature's requirement
document.

Optional. Usually run right after `define-functional-requirement`, once the
functional requirement already exists.

## Input

Resolve the feature this run is about, exactly as `define-functional-requirement`
does:

| Source | Feature slug | Requirement file |
| --- | --- | --- |
| **Ticket id** (Jira, GitHub issue, other) | the ticket identifier (e.g. `PROJ-123`) | `docs/<slug>/requirement.html` |
| **Feature slug** given directly | the slug as given (e.g. `bulk-export-csv`) | `docs/<slug>/requirement.html` |

If `docs/<feature-slug>/requirement.html` does not exist, stop and tell the user
to run `define-functional-requirement` for this feature first — there is no
document to complete.

## Procedure

1. **Resolve target.** Determine the feature slug and confirm
   `docs/<feature-slug>/requirement.html` exists.
2. **Dispatch the analysis agent (in the background).** Spin off one
   general-purpose agent to fully analyze the current project — its source code
   and its documentation (`README`, `CLAUDE.md`, the `./nfr/` documents,
   architecture/design docs). Its job is to report the tech in use, the project's
   existing non-functional posture, and which NFR categories realistically apply.
   Let it run while you do step 3.
3. **Ensure project-level NFRs exist.** Invoke the `nfr-init` skill. If the
   project's `./nfr/` set is already defined and matches the project, nothing
   changes — do nothing further there. If it is missing or out of date, follow
   `nfr-init` to add the applicable documents before continuing.
4. **Wait for the analysis agent** to finish and read its report.
5. **Derive the feature's NFRs.** Cross-read three things: the functional
   requirement in `docs/<feature-slug>/requirement.html`, the project-level NFR
   documents now in `./nfr/`, and the agent's analysis. Identify which
   non-functional constraints *this specific* feature or bug fix triggers, and
   which ones need a decision from the user — e.g. latency / throughput targets,
   expected data volume, authentication & authorization, handling of personal or
   sensitive data, accessibility level, supported browsers / devices, concurrency,
   rate limits, error budgets, logging & monitoring.
6. **Confirm with the user.** Ask every open question from step 5. Then explicitly
   ask whether they want any additional non-functional requirements added. The
   user's answers are authoritative.
7. **Write the section.** Fill the **Non Functional Requirements** section of
   `docs/<feature-slug>/requirement.html` (the `{{NON FUNCTIONAL REQUIREMENTS}}`
   placeholder) with the feature-specific constraints and their agreed values.
   Reference the relevant `./nfr/` documents rather than restating their content —
   capture only what is specific to this feature. **If that section is already
   populated, rewrite it, integrating your findings** rather than appending.

## Writing standards

The same standards as `define-functional-requirement` apply to everything you
write into the document: no AI-slop phrasing, no corporate jargon or buzzwords,
no vague nouns ("the project", "the repo", "the script") unless made specific —
name the project, name the repo, and give file paths relative to the project
root. Every constraint must be checkable; if a value is unknown, record it as an
open question rather than guessing.

## Common mistakes

- **Running before the functional requirement exists** — there is no
  `requirement.html` section to fill; run `define-functional-requirement` first.
- **Skipping the project analysis** — the feature's NFRs depend on the project's
  actual tech and posture; dispatch the analysis agent, don't assume.
- **Re-deriving project-wide NFRs** — those live in `./nfr/` and are owned by
  `nfr-init`; this skill only captures the *feature-specific* constraints and
  references the project set.
- **Appending to an already-populated section** — rewrite it, integrating old and
  new, so it stays coherent.
- **Inventing thresholds** — confirm latency, volume, accessibility level, and
  similar values with the user instead of guessing.
