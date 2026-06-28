---
name: define-implementation-plan
description: Use to turn a feature's or bug fix's requirement into a concrete implementation plan — a list of discrete, independently testable and releasable tasks — given its ticket id or its feature slug. Analyzes the codebase against the functional and non-functional requirements, then either creates child implementation tickets under the requirement ticket, comments the approach on a simple ticket, or writes an implementation-plan HTML document next to the requirement.
---

# define-implementation-plan — Break a Requirement into Implementation Tasks

Turn an already-defined requirement into a concrete plan for building it: a set of
discrete implementation tasks, sized so each one can be **tested independently and
released independently** — even if it ships disabled behind a flag.

This skill runs after `define-functional-requirement` (and usually
`define-nfr`). It needs, at minimum, the functional requirements in the HTML
requirement document.

## Input

Resolve the feature this run is about, exactly as the other `define-*` skills do:

| Source | Feature slug | Requirement file |
| --- | --- | --- |
| **Ticket id** (Jira, GitHub issue, other) | the ticket identifier (e.g. `PROJ-123`) | `docs/<slug>/requirement.html` |
| **Feature slug** given directly | the slug as given (e.g. `bulk-export-csv`) | `docs/<slug>/requirement.html` |

**Hard stop:** if no HTML requirement document can be located at
`docs/<feature-slug>/requirement.html`, stop and ask the user to point you to the
HTML requirement document. Do not invent a plan without it.

## Where the plan goes

Two cases, decided by whether the requirement is tracked in a ticketing system:

- **Ticketing system in use** (a ticket id was given / a tracker is reachable) →
  the plan goes into the tracker (a comment, or child tickets — see below). Do
  **not** write the HTML plan document in this case.
- **No ticketing system** → the plan goes into
  `docs/<feature-slug>/implementation-plan.html`, built from
  `implementation-plan-template.html` (ships next to this SKILL.md).

If it is unclear whether a tracker is in use, ask the user.

## Procedure

1. **Resolve target.** Determine the feature slug and locate
   `docs/<feature-slug>/requirement.html`. If it is missing, hard-stop (above).
2. **Dispatch the analysis agent (in the background).** Spin off one
   general-purpose agent to fully analyze the current project — its source code
   and its documentation (`README`, `CLAUDE.md`, the `./nfr/` documents,
   architecture/design docs) — and report the structure, the relevant existing
   code, and the integration points the work will touch. Let it run while you
   read the requirement.
3. **Read the requirements.** Read the functional and non-functional requirements
   from `docs/<feature-slug>/requirement.html`. Then wait for the analysis agent
   and read its report.
4. **Judge simple vs. complex.** Based on the analysis and the requirements,
   decide whether this is a *simple* change (one cohesive, self-contained piece of
   work) or a *complex* one (needs to be split).
5. **Design the tasks.** Define the discrete tasks that fully implement the
   requirement, honoring both the functional and non-functional requirements.
   **Splitting rule:** each task must be independently **testable** and
   independently **releasable** — even if it ships in a disabled state (e.g.
   behind a feature flag). Order the tasks so dependencies come first.
   **Shared interfaces:** when two tasks share a common interface (a function
   signature, API shape, schema, event, or data structure that one produces and
   another consumes), define that interface explicitly and identically in **both**
   tickets, so neither agent has to infer it from the other.
6. **Record the plan** per the matrix below.
7. **Check each ticket for ambiguity, then fix it.** After the tickets are
   drafted, dispatch **one fresh general-purpose agent per ticket**. Give each
   agent **only** that single ticket's text plus the linked requirement document —
   not the other tickets, and not this conversation. Instruct it to list every
   question it would need answered and every assumption it would have to make in
   order to implement the ticket. Then **rewrite the ticket so that every listed
   question and assumption is resolved in the ticket text itself.** Repeat for any
   ticket still producing questions until each ticket is self-sufficient — an
   agent can implement it from the ticket plus the requirement alone, with no open
   questions.

### Output matrix

| | **Ticketing system** | **No ticketing system** |
| --- | --- | --- |
| **Simple** | Add a **comment** to the requirement ticket describing the best approach to implement the feature / fix, taking the functional and non-functional requirements into account. Do **not** create child tickets. | Write `implementation-plan.html` with a filled **Chosen Approach** and a single task. |
| **Complex** | Create one **child ticket per task**, each linked as a child of the requirement ticket. | Write `implementation-plan.html` with the **Chosen Approach** plus one task block per task. |

**Child ticket rules (complex + ticketing):**

- Create the tickets with the same tooling used to read the requirement ticket
  (`gh` for GitHub issues, the Jira MCP/CLI for Jira, etc.) and link each one as a
  child of the requirement ticket (Jira sub-task, GitHub sub-issue, or a tracked
  "Part of #<id>" reference if native linking is unavailable).
- Each child ticket's **title states its goal and ends with its order** in the
  sequence, e.g. `Add CSV export endpoint (1/3)`, `Wire export button into the
  toolbar (2/3)`, `Enable export flag by default (3/3)`.
- Each ticket body states scope, how it is tested independently, how it is
  released independently (the flag / disabled state if relevant), what it depends
  on, and its acceptance check.
- When two tickets share a common interface, spell that interface out fully and
  identically in both ticket bodies — do not make one ticket point at the other
  for it.

## implementation-plan-template.html

Minimal early-web HTML, same compact style block as the requirement template.
Replace the `{{PLACEHOLDER}}`s; `{{TASKS}}` is one `<h3>` + block per task. Every
task heading **must start with a sequential number, beginning at 1 and increasing
by 1 with no gaps** (`1.`, `2.`, `3.`, …) — including when there is only one task
(`1.`). The template's HTML comment shows the per-task shape. Don't add external
dependencies or heavier styling.

## Writing standards

The same standards as `define-functional-requirement` apply to every comment,
ticket, and HTML section you write: no AI-slop phrasing, no corporate jargon or
buzzwords, no vague nouns ("the project", "the repo", "the script", "the system")
unless made specific — name the project, name the repo, and give file paths
relative to the project root. Every task must be checkable; if something is
unknown, record it as an open question rather than guessing.

## Common mistakes

- **No requirement document** — hard-stop and ask the user to locate it; never
  plan without it.
- **Creating tickets for trivial work** — a simple change gets a comment on the
  requirement ticket (or a single-task plan), not a pile of child tickets.
- **Tasks that can't ship alone** — every task must be independently testable and
  releasable, even if disabled behind a flag; if a task can't, merge or re-split.
- **Titles without order** — every child ticket title ends with its order
  (`1/3`, `2/3`, …).
- **Writing the HTML plan when a tracker is in use** — with a ticketing system the
  plan lives in the tracker, not in `docs/<slug>/`.
- **Skipping the codebase analysis** — the split depends on the real structure and
  integration points; dispatch the agent, don't assume.
- **Shipping tickets without the ambiguity check** — every ticket must survive a
  fresh agent reading it in isolation with no questions; rewrite until it does.
- **A shared interface defined in only one ticket** — define it fully and
  identically in both, so neither agent has to infer it.
- **HTML tasks not sequentially numbered** — in `implementation-plan.html` every
  task heading must start with a sequential number from 1 with no gaps.
