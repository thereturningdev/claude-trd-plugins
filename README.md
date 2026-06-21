# TRD Plugin Marketplace

The Returning Dev's [Claude Code](https://docs.claude.com/en/docs/claude-code) plugin marketplace. Catalog: [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json).

## Available plugins

### trd-development

Software development skills and agents. Lives in [`plugins/trd-development/`](plugins/trd-development/).

Skills:

- **nfr** — Applies The Returning Dev's non-functional requirements to a project. Detects the project type, confirms it with you, then copies only the applicable requirement docs into `./nfr/` and references them as mandatory in `CLAUDE.md`. Selection: `general` (always), `development` (software projects), `python` (Python projects), `web` (projects with a web component).
- **adversarial-tester** — Dispatches a tester agent to break a chosen function or module. It writes a large batch of edge-case tests (kept in the repo as a regression suite), runs them against the target's *intended* behavior, and produces a self-contained `adversarial-test-report.html` in the working directory listing every passing and failing case.

## Connecting

```
/plugin marketplace add thereturningdev/claude-trd-plugins
```

Then browse and install:

```
/plugin                                    # browse
/plugin install trd-development@trd        # install a plugin
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
│   └── trd-development/    # software development skills & agents
├── README.md
└── LICENSE
```
