# Claude Code Plugins Marketplace

A curated collection of plugins extending Claude Code's capabilities with specialized workflows and automation tools.

## Quick Start

### 1. Add the Marketplace

```bash
/plugin marketplace add pioneerworks/claude-code-plugins
```

### 2. Browse Available Plugins

```bash
/plugin list claude-code-plugins
```

### 3. Install a Plugin

```bash
/plugin install <plugin-name>@claude-code-plugins
```

For example:
```bash
/plugin install assembly@claude-code-plugins
```

## Available Plugins

### [Assembly](./plugins/assembly)
Automates end-to-end implementation of small Jira tickets (≤2 SP) through a three-agent workflow: plan → execute → review.

**Use case:** Sprint automation for small, well-defined tickets
**Requirements:** Atlassian MCP server, packs-based Rails project

```bash
/plugin install assembly@claude-code-plugins
/assembly PROJ-123
```

[Full documentation →](./plugins/assembly/README.md)

## Plugin Usage

After installation, plugins add new slash commands to Claude Code. Each plugin's README contains:
- Detailed usage instructions
- Configuration requirements
- Examples and troubleshooting

Access plugin-specific help:
```bash
/<plugin-command> --help
```

## Requirements

- **Claude Code CLI** (latest version)
- Plugin-specific dependencies (see individual plugin READMEs)
- MCP servers as needed per plugin

## Updating Plugins

Check for updates:
```bash
/plugin update claude-code-plugins
```

Update a specific plugin:
```bash
/plugin update assembly@claude-code-plugins
```

## Removing Plugins

Uninstall a plugin:
```bash
/plugin uninstall assembly
```

Remove the marketplace:
```bash
/plugin marketplace remove pioneerworks/claude-code-plugins
```

## Contributing

### Adding a New Plugin

1. Create a directory under `plugins/<your-plugin-name>/`
2. Required files:
   - `README.md` - Plugin documentation
   - `plugin.json` - Plugin metadata
   - Command files (`.md` or scripts)
3. Follow naming conventions:
   - Lowercase with hyphens (e.g., `my-plugin`)
   - Clear, descriptive names
4. Submit a pull request

### Plugin Structure

```
plugins/
└── your-plugin/
    ├── README.md          # User-facing documentation
    ├── plugin.json        # Metadata and configuration
    ├── commands/          # Slash command implementations
    │   └── command.md
    └── assets/           # Supporting files (optional)
```

### Guidelines

- **Single responsibility**: Each plugin should solve one clear problem
- **Documentation**: Include usage examples, requirements, and troubleshooting
- **Dependencies**: Clearly document all MCP servers and tools required
- **Testing**: Test plugins in realistic project environments
- **Versioning**: Follow semantic versioning

## Support

- **Issues**: [GitHub Issues](https://github.com/pioneerworks/claude-code-plugins/issues)
- **Claude Code Help**: `/help` or [docs.claude.com](https://docs.claude.com/claude-code)
- **Feedback**: Open a discussion or issue in this repository

## License

Individual plugins may have their own licenses. See plugin-specific READMEs.

## Changelog

### v0.1.0 (Initial Release)
- Added Assembly plugin for Jira ticket automation
- Established marketplace structure
