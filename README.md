# TRD Plugin Marketplace

A Claude Code plugin marketplace.

## What this is

A marketplace is a git repository that catalogs Claude Code plugins. The catalog lives in `.claude-plugin/marketplace.json`. Each plugin lives in its own directory under `plugins/` and is listed in that manifest.

## Connecting to this marketplace

Local development:

```
/plugin marketplace add /Users/fdiotalevi/TRDWorkspace/claude-trd-plugins
```

From GitHub once pushed:

```
/plugin marketplace add <owner>/claude-trd-plugins
```

Browse and install plugins with the `/plugin` command. Refresh the catalog later with:

```
/plugin marketplace update trd
```

## Adding a new plugin

1. Create `plugins/<name>/.claude-plugin/plugin.json` with a minimal manifest:

   ```json
   {
     "name": "<name>",
     "description": "What this plugin does",
     "version": "0.1.0"
   }
   ```

2. Add components inside `plugins/<name>/`, which Claude Code auto-discovers:
   - `commands/` for slash commands
   - `agents/` for subagents
   - `skills/` for skills (each a `SKILL.md` in its own subdirectory)
   - `hooks/hooks.json` for event hooks
   - `.mcp.json` for MCP servers

3. Register the plugin in `.claude-plugin/marketplace.json` by adding an object to the `plugins` array:

   ```json
   {
     "name": "<name>",
     "source": "./plugins/<name>",
     "description": "What this plugin does"
   }
   ```

4. Reload and install:

   ```
   /plugin marketplace update trd
   /plugin install <name>@trd
   ```

## Structure

```
.
├── .claude-plugin/
│   └── marketplace.json   # the catalog
├── plugins/               # plugin directories live here
├── README.md
└── LICENSE
```
