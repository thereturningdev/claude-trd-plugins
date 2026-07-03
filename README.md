# TRD Plugin Marketplace

The Returning Dev's [Claude Code](https://docs.claude.com/en/docs/claude-code) plugin marketplace. Catalog: [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json).

## Available plugins

### trd-dev

Software development skills and agents. Lives in [`plugins/trd-dev/`](plugins/trd-dev/).

Skills:

- **nfr-init** — Initializes a project's non-functional requirements, or updates them when the project has changed significantly. Detects the project type, confirms it with you, then copies only the applicable requirement docs into `./nfr/` and references them as mandatory in `CLAUDE.md`. Selection: `general` (always), `development` (software projects), `python` (Python projects), `web` (projects with a web component).
- **define-functional-requirement** — Turns a rough feature idea or bug-fix request (from chat or a ticket) into a structured functional requirement. Asks clarifying and edge-case questions, optionally builds HTML mockups for approval, then writes an HTML requirement under `docs/<feature-slug>/`. When the idea came from a ticket, it rewrites the ticket with the complete definition and links the requirement back to it.
- **define-nfr** — Twin of `define-functional-requirement`, usually run right after it. Given a ticket id or feature slug, it analyzes the project's code and docs, ensures project-level NFRs exist via `nfr-init`, asks you to confirm the non-functional constraints the feature needs (performance, security, accessibility, and so on), then fills the "Non Functional Requirements" section of `docs/<feature-slug>/requirement.html`.
- **define-implementation-plan** — Turns a defined requirement into a concrete implementation plan: discrete tasks each independently testable and releasable (even if shipped disabled behind a flag). Given a ticket id or feature slug, it analyzes the codebase against the functional and non-functional requirements, then creates child implementation tickets under the requirement ticket, comments the approach on a simple ticket, or writes `implementation-plan.html` next to the requirement when no tracker is in use. Stops if it can't find the HTML requirement document.
- **implement-task** — Implements one or more defined implementation tasks. Given a single ticket id, a comma-separated list of ticket ids, or task number(s) from an HTML implementation plan (path provided by you), it dispatches one agent per task in order — each building the task with test-driven development, KISS/YAGNI, and an adversarial pass via `adversarial-tester`, then verifying it with two independent reviewers (one for unrequested or dangerous behavior, one for side effects elsewhere in the product) before fixing any confirmed issues. When no requirement document or implementation plan exists, it works straight from the ticket or the description you provide, keeping the same standards.
- **adversarial-tester** — Dispatches a tester agent to break a chosen function or module. It writes a large batch of edge-case tests (kept in the repo as a regression suite), runs them against the target's *intended* behavior, and produces a self-contained `adversarial-test-report.html` in the working directory listing every passing and failing case.

## Connecting

```
/plugin marketplace add thereturningdev/claude-trd-plugins
```

Then browse and install:

```
/plugin                                    # browse
/plugin install trd-dev@trd                # install a plugin
/plugin marketplace update trd             # refresh the catalog later
```

For local development against this checkout, point at the path instead:

```
/plugin marketplace add /Users/fdiotalevi/TRDWorkspace/claude-trd-plugins
```

## Adding a plugin

1. Create `plugins/<name>/.claude-plugin/plugin.json`.
2. Add components under `plugins/<name>/` — Claude Code auto-discovers `skills/`, `agents/`, `commands/`, `hooks/hooks.json`, `.mcp.json`.
3. Register it in [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json) with `name`, `source` (`./plugins/<name>`), and `description`.
4. Reload: `/plugin marketplace update trd`.

## Structure

```
.
├── .claude-plugin/
│   └── marketplace.json   # the catalog
├── plugins/
│   └── trd-dev/            # software development skills & agents
├── README.md
└── LICENSE
```
