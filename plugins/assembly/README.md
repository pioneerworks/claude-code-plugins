# Assembly Plugin

Automates end-to-end implementation of small Jira tickets (≤2 SP) through a three-agent workflow: plan → execute → review.

## Installation

1. **Add the plugin marketplace:**
   ```bash
   /plugin marketplace add pioneerworks/claude-code-plugins
   ```

2. **Install the assembly plugin:**
   ```bash
   /plugin install assembly@claude-code-plugins
   ```

3. **Configure Atlassian MCP:**
   ```bash
   /mcp
   ```
   Then authenticate with your Atlassian credentials when prompted.

## Quick Start

```bash
/assembly PROJ-123
```

## Usage

```bash
/assembly ticket_key=PROJ-123 [mode=auto|interactive] [max_cycles=3]
```

**Parameters:**
- `ticket_key` (required): Jira ticket key
- `mode` (optional): `auto` (default) or `interactive` (preview before applying)
- `max_cycles` (optional): Max revision attempts (default: 3)

## How It Works

```
Jira Ticket → Planner → Executor → Reviewer → [Tests/Linters] → Done
                ↑                        ↓
                └──── Replan (if REVISE) ─┘
```

### Workflow

1. **Preflight**: Fetch Jira ticket, validate ≤2 SP
2. **Plan**: Generate `TODO.V{n}` with 3-7 tasks, file paths, and test strategy
3. **Execute**: Apply file changes directly (no Git operations)
4. **Review**: Validate against acceptance criteria
5. **Finalize**: Run packwerk, rubocop, eslint, and tests
6. **Replan**: If review fails or tests fail, iterate with feedback

### Three Agents

1. **assembly-planner** (Opus): Converts Jira tickets into structured plans
2. **assembly-executor** (Sonnet): Implements changes via direct file edits
3. **assembly-reviewer** (Sonnet): Validates implementation and provides feedback

## Architectural Constraints

The plugin enforces:
- **Packs-first**: New domain code under `/packs/<domain>/`
- **No business logic in `/app/`**
- **FE imports**: Use `fe-design-base/...` (no `@` aliases)

## Requirements

- Atlassian MCP server configured for Jira access
- Project structure: packs-based architecture with Rails backend
- Tools: RSpec, Rubocop, Packwerk, ESLint, Jest, Playwright

## Examples

**Auto mode** (apply immediately):
```bash
/assembly SHOP-456
```

**Interactive mode** (preview first):
```bash
/assembly SHOP-456 mode=interactive
```

**Custom max cycles**:
```bash
/assembly SHOP-456 max_cycles=5
```

## Limitations

- Only supports ≤2 SP tickets
- Assumes specific project structure (packs, Rails, etc.)
- No Git operations (direct file editing only)
- Requires Atlassian MCP configuration

## Troubleshooting

**Ticket too large**: Split into smaller tickets if >2 SP

**Architectural violations**: Check `REVIEW.MD` for feedback on constraint violations

**Max cycles exceeded**: Review `REASONS.V1` feedback history, may need ticket refinement

**Test failures**: Check `APPLY_LOG.MD` for details, failures trigger automatic replan
