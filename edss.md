---
name: edss
description: Everything for DeepSeek Harness — comprehensive command reference, configuration patterns, plugin management, and troubleshooting guide for the DeepSeek Harness (DSH) platform. Use when working with DSH profiles, agents, skills, plugins, cordis compositions, sessions, tooling, or deployment configuration.
whenToUse: When working with DeepSeek Harness, DSH commands, profile configuration, agent presets, skill management, cordis compositions, or harness deployment questions.
---

# EDS: Everything for DeepSeek Harness

This skill provides a comprehensive reference for the DeepSeek Harness (DSH) platform.

## Quick Start

```bash
# Boot the web profile
dsh web

# Boot with a specific profile
dsh --profile <name>

# Check available profiles
dsh --help
```

## Core Concepts

### Profiles
A DSH profile is an ordered stack of plugin-bundle patch layers. Each profile defines what capabilities are available to an agent.

- **Web profile**: The default GUI-based session (`dsh web`)
- **Headless profile**: For automated/scripted tasks
- **TUI profile**: Terminal UI session

### Cordis Compositions
Every capability in DSH is defined by a Cordis composition — a YAML file listing plugin rows:

```yaml
- id: skill
  name: '@deepseek-ai/dsh-skill'

- id: tool-filesystem
  name: '@deepseek-ai/dsh-tool-filesystem'
```

### Skills
Skills are Markdown files with YAML frontmatter that provide context-specific instructions to agents.

**Location hierarchy** (priority order):
1. `.dsh/skills/` — Project-local
2. `.agents/skills/` — Project-local agent
3. `$DSH_HOME/skills/` — User-level
4. `$AGENTS_HOME/skills/` — Agent-level
5. Bundled skills from the deployment

**Skill file format**:
```markdown
---
name: my-skill
description: What this skill does
whenToUse: When to trigger this skill
---

# Skill Content

Instructions for the agent...
```

### Agent Presets
Agent presets define per-session behavior:

- `standard` — Full coding agent
- `code` — Code-focused agent
- `minimal` — Stripped-down agent
- `cordis` — Cordis plugin development agent

## DSH CLI Commands

```bash
# Boot profiles
dsh web                              # Start web GUI
dsh --profile headless "task"        # Run headless task
dsh --profile tui                    # Terminal UI

# Profile management
dsh plugin --profile <name> add <package>    # Install plugin
dsh plugin --profile <name> remove <package> # Remove plugin
dsh plugin --profile <name> ls               # List installed plugins

# Configuration inspection
dsh --dump-config              # Show composed profile tree
dsh --dump-default-config      # Show default without user layer
dsh --profile <name> --help    # Show profile-specific help

# Resume sessions
dsh --profile <name> --resume <session-id>

# With overlays
dsh --profile web --patch ./extra.yml
```

## Profile Configuration

### Directory Structure
```
~/.dsh/
├── profiles/
│   └── web/
│       ├── package.json          # Plugin dependencies
│       ├── cordis.yml            # Base composition (read-only)
│       ├── cordis.patch.yml      # User patches
│       └── node_modules/         # Installed plugins
├── .agent-presets/               # User-defined presets
└── skills/                       # User-defined skills
```

### package.json
```json
{
  "name": "dsh-profile-web",
  "private": true,
  "dependencies": {},
  "dsh": {
    "profile": {
      "bundles": [
        "@deepseek-ai/dsh-base",
        "@deepseek-ai/dsh-web-app"
      ]
    }
  }
}
```

### cordis.patch.yml
```yaml
# User patches applied after bundle layers
- id: my-skill
  name: '@deepseek-ai/dsh-skill-my'
  config:
    someOption: value
```

## Built-in Skills

| Skill | Description |
|-------|-------------|
| `@deepseek-ai/dsh-skill` | Core skill registry |
| `@deepseek-ai/dsh-skill-filesystem` | Filesystem skill provider |
| `@deepseek-ai/dsh-skill-badge` | DSH branding badge (disabled by default) |

## Built-in Tools

### Agent Tools
| Tool | Description |
|------|-------------|
| `ask_user_question` | Ask the user a question |
| `bash` | Execute bash commands |
| `edit` | Edit text files |
| `write` | Create/replace files |
| `glob` | Find files by pattern |
| `grep` | Search file contents |
| `read` | Read text files |
| `read_image` | Read image files |
| `todo_write` | Update task list |
| `web_search` | Search the web |

### Advanced Tools
| Tool | Description |
|------|-------------|
| `subagent` | Delegate to background subagent |
| `subagent_fork` | Fork with conversation history |
| `workflow` | Multi-agent workflow orchestration |
| `ralph` | Fresh-agent iterative execution |
| `goal` | Create/update persisted goals |
| `interrupt_agent` | Interrupt running subagent |

## Cordis Inspection

Use `cordis_inspect` to query runtime state:

```javascript
// List all providers
cordis_inspect what:"list"

// Query a specific service
cordis_inspect what:"api" name:"serviceName"

// Inspect current agent preset
cordis_inspect what:"presets"
```

## Managing Plugins

```bash
# Install into profile
dsh plugin --profile web add @scope/package-name

# Remove from profile
dsh plugin --profile web remove @scope/package-name

# List installed
dsh plugin --profile web ls
```

## Creating Custom Skills

### Basic Skill
```yaml
# skills/my-skill/SKILL.md
---
name: my-skill
description: Brief description of what this skill does
whenToUse: When to activate this skill
---

# My Skill

Instructions here...
```

### Skill with Options
```yaml
---
name: my-skill
description: My custom skill
disable-model-invocation: false
user-invocable: true
metadata:
  version: 1.0
  author: your-name
---

# Content
```

## Agent Preset Development

### Create a Preset
```bash
# Copy from existing preset
# (use cordis API to copy)
```

### Preset Structure
```
~/.dsh/.agent-presets/
└── my-preset/
    ├── agent.cordis.yml    # Main composition
    └── preset.yml          # Metadata
```

### preset.yml
```yaml
name: "My Preset"
description: "Custom agent preset"
```

### agent.cordis.yml
```yaml
- id: my-tool
  name: '@deepseek-ai/dsh-tool-my'
  config:
    option: value
```

## Troubleshooting

### Plugin Not Found
```
Error: Cannot find module '@scope/package'
```
**Fix**: Ensure the package is installed in the profile's node_modules:
```bash
dsh plugin --profile <name> add @scope/package
```

### Mount Validation Failed
```
N row(s) did not activate: <id>: waiting for <service>
```
**Fix**: Check that the required service is available and the plugin name is correct.

### Skill Not Loading
1. Verify skill file is in one of the skill directories
2. Check YAML frontmatter has `name` and `description`
3. Ensure skill name matches `[a-z0-9][a-z0-9-]*`

### Permission Issues
```
EPERM: operation not permitted
```
**Fix**: May require `danger-full-access` sandbox escalation for paths outside workspace.

## Environment Variables

| Variable | Description |
|----------|-------------|
| `DSH_HOME` | Home directory for DSH (default: `~/.dsh`) |
| `DSH_AGENTS_HOME` | Agents home directory (default: `~/.agents`) |
| `DSH_BUNDLED_SKILL_DIR` | Directory for bundled skills |

## Common Patterns

### Adding a New Tool to Profile
```yaml
# In cordis.patch.yml
- id: tool-myservice
  name: '@deepseek-ai/dsh-tool-myservice'
```

### Creating a Custom Agent Preset
1. Copy existing preset
2. Edit `agent.cordis.yml`
3. Mount-validate with `standingKeyFor`
4. Test in new session

### Skill Organization
```
.dsh/skills/
├── my-workflow.md      # Flat skill file
└── project-guide/
    └── SKILL.md        # Directory-bundle skill
```

## Package Naming Conventions

DSH packages follow this pattern:
- `@deepseek-ai/dsh-base` — Core base
- `@deepseek-ai/dsh-web-app` — Web GUI
- `@deepseek-ai/dsh-skill-*` — Skill providers
- `@deepseek-ai/dsh-tool-*` — Tool implementations
- `@deepseek-ai/dsh-agent-*` — Agent configurations

## Resources

- DSH Repository: `github.com/deepseek-ai/deepseek-harness`
- Skill filesystem docs: See `@deepseek-ai/dsh-skill-filesystem`
- Cordis docs: See `@deepseek-ai/cordis`
