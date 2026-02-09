# Claude Code Power User Cheat Sheet

> **Version:** February 2026 | **Claude Code:** v1.0.x+ | **Models:** Opus 4.6, Sonnet 4.5, Haiku 4.5

---

## Quick Reference Card

| Action | How |
|--------|-----|
| Start interactive session | `claude` |
| Start with prompt | `claude "fix the login bug"` |
| Headless (pipe-friendly) | `claude -p "explain this"` |
| Continue last conversation | `claude -c` |
| Resume specific session | `claude -r <session-id>` |
| Switch model mid-session | `/model opus` |
| Check context usage | `/context` |
| Compress context | `/compact` |
| Rewind to previous state | `Esc Esc` or `/rewind` |
| Stop generation | `Esc` |
| Toggle permission mode | `Shift+Tab` |
| Show all commands | `/help` |
| Run diagnostics | `/doctor` |
| Exit | `Ctrl+D` or `/exit` |

---

## 1. CLI Flags & Invocation

### Core Commands

```bash
claude                           # Interactive REPL
claude "query"                   # REPL with initial prompt
claude -p "query"                # Print mode (headless), then exit
claude -c                        # Continue most recent conversation
claude -c -p "follow up"         # Continue in print mode
claude -r <id>                   # Resume session by ID or name
claude update                    # Update to latest version
claude mcp                       # Configure MCP servers
claude doctor                    # Run diagnostics
claude install                   # Migrate npm install to native
```

### All Flags

| Flag | Description |
|------|-------------|
| `-p, --print` | Non-interactive mode (for CI/CD, scripts) |
| `-c, --continue` | Resume most recent conversation |
| `-r, --resume <id>` | Resume specific session by ID or name |
| `--model <name>` | Set model (`sonnet`, `opus`, `haiku`, `opusplan`, or full ID) |
| `--permission-mode <mode>` | `default`, `plan`, `acceptEdits`, `bypassPermissions` |
| `--allowedTools <tools...>` | Auto-allow specific tools (e.g. `"Bash(git *)" "Read"`) |
| `--disallowedTools <tools...>` | Remove tools entirely (e.g. `"Bash(rm *)"`) |
| `--add-dir <path>` | Add additional working directories |
| `--output-format <fmt>` | `text`, `json`, `stream-json` |
| `--input-format <fmt>` | `text`, `stream-json` |
| `--max-turns <n>` | Limit agentic turns (print mode) |
| `--max-budget-usd <n>` | Spending cap per session (print mode) |
| `--verbose` | Full turn-by-turn logging |
| `--debug [categories]` | Debug mode (e.g. `"api,hooks"` or `"!statsig"`) |
| `--system-prompt <text>` | Replace default system prompt |
| `--append-system-prompt <text>` | Append to default prompt (recommended) |
| `--json-schema <schema>` | Validate output against JSON Schema (print mode) |
| `--mcp-config <path>` | Load MCP servers from JSON |
| `--strict-mcp-config` | Only use MCP servers from `--mcp-config` |
| `--agents <json>` | Define custom subagents inline |
| `--session-id <uuid>` | Use specific session ID |
| `--fork-session` | Create new session ID when resuming |
| `--from-pr <pr>` | Resume sessions linked to a GitHub PR |
| `--no-session-persistence` | Don't save session (print mode) |
| `--fallback-model` | Auto-fallback when model overloaded (print mode) |
| `--setting-sources <list>` | Comma-separated: `user`, `project`, `local` |
| `--settings <path-or-json>` | Additional settings file or JSON string |
| `--chrome` / `--no-chrome` | Enable/disable Chrome integration |
| `--ide` | Auto-connect to IDE if exactly one available |
| `--init` | Run init hooks, start interactive |
| `--init-only` | Run init hooks, then exit |
| `--remote` | Create new web session on claude.ai |
| `--teleport` | Resume web session locally |
| `--disable-slash-commands` | Disable all skills/commands |
| `-v, --version` | Print version |

### Piping & Composition

```bash
cat file.py | claude -p "review this code"
git diff | claude -p "summarize changes"
claude -p "list all TODOs" --output-format json | jq '.result'
echo '{"task":"review"}' | claude --input-format stream-json -p "go"
```

---

## 2. Slash Commands

### Built-in Commands

| Command | Description |
|---------|-------------|
| `/help` | Show all available commands (built-in + custom) |
| `/compact` | Compress conversation, free context window |
| `/context` | Visualize context window usage (colored grid) |
| `/cost` | Show token usage stats for session |
| `/model [name]` | Switch model (e.g. `/model opus`) |
| `/config` | Open configuration interface |
| `/keybindings` | Open keybinding config (`~/.claude/keybindings.json`) |
| `/init` | Generate CLAUDE.md project guide |
| `/memory` | Edit project memory files (CLAUDE.md) |
| `/status` | Account info, version, connectivity |
| `/doctor` | Diagnose installation/config issues |
| `/debug` | Troubleshoot current session |
| `/rewind` | Restore code/conversation to previous state |
| `/clear` | Clear conversation history |
| `/login` | Switch Anthropic accounts |
| `/install-github-app` | Set up GitHub app for PR automation |
| `/exit` | Exit the session |

### Custom Commands

Custom commands appear in `/help` and are invoked as `/command-name`:

```
.claude/commands/review.md      → /review       (project, shared)
~/.claude/commands/standup.md   → /standup      (user, personal)
```

---

## 3. Keyboard Shortcuts

### Essential

| Shortcut | Action |
|----------|--------|
| `Esc` | Stop/interrupt Claude |
| `Esc Esc` | Open rewind menu (all previous messages) |
| `Ctrl+C` | Cancel current operation |
| `Ctrl+D` | Exit Claude Code |
| `Tab` | Autocomplete / toggle thinking |
| `Shift+Tab` | Cycle permission modes (default → acceptEdits → plan → default) |
| `Up / Down` | Navigate command history |
| `Ctrl+R` | Search command history |
| `Ctrl+T` | Toggle task list view |
| `Ctrl+B` | Move Bash command to background (tmux: press twice) |

### Text Editing (Bash/Readline-style)

| Shortcut | Action |
|----------|--------|
| `Ctrl+A` | Move to start of line |
| `Ctrl+E` | Move to end of line |
| `Option+F` / `Option+B` | Move word forward / back (macOS: set Option as Meta) |
| `Ctrl+W` | Delete previous word |

### Multiline Input

| Terminal | How to Enter Newline |
|----------|---------------------|
| iTerm2 | `Shift+Enter` (works out of box) |
| WezTerm | `Shift+Enter` |
| Ghostty | `Shift+Enter` |
| Kitty | `Shift+Enter` |
| Others | Paste multiline text, or configure keybindings |

### Customization

Edit `~/.claude/keybindings.json` (create via `/keybindings`):
```json
{
  "Chat": { "toggleThinking": "Tab" },
  "Terminal": { "interrupt": "Escape" }
}
```

---

## 4. Permission Modes & Rules

### Modes

| Mode | Behavior |
|------|----------|
| `default` | Reads allowed; asks before writes/commands |
| `plan` | Read-only — analyze but not modify anything |
| `acceptEdits` | File edits auto-approved; commands still ask |
| `bypassPermissions` | Everything auto-approved (containers/VMs only!) |

Switch at launch: `claude --permission-mode plan`
Switch mid-session: `Shift+Tab` to cycle

### Permission Rules in Settings

```json
{
  "allowedTools": [
    "Read",
    "Bash(git log *)",
    "Bash(npm test *)",
    "Write"
  ],
  "disallowedTools": [
    "Bash(rm -rf *)",
    "Read(./.env)"
  ]
}
```

### Rule Syntax

| Pattern | Matches |
|---------|---------|
| `"Read"` | Read tool only |
| `"Write\|Edit"` | Write or Edit |
| `"Bash(git *)"` | Bash with any git command |
| `"Bash(npm test*)"` | Bash with npm test commands |
| `"Read(./.env)"` | Read on .env files |

### Auto-Approval Methods

| Method | Scope | How |
|--------|-------|-----|
| `allowedTools` in settings | Permanent (all sessions) | `"Write(src/generated/*)"` in settings.json |
| "Always allow" prompt | Current session only | Choose when Claude asks for permission |
| `Shift+Tab` mode toggle | Until you toggle back | Cycle to `acceptEdits` while working, flip back when done |
| Command `allowed-tools` | Single command execution | `allowed-tools: Write, Bash(npm run *)` in frontmatter |
| PreToolUse hook | Custom logic (file path, pattern, etc.) | Return `permissionDecision: "allow"` from hook script |

Path-scoped example: `"Write(src/generated/*)"` — auto-approves writes only to generated files.
See `claude-basics/CLAUDE_CODE_DEEP_DIVES.md` → Permissions & Auto-Approval for full details.

### Settings Precedence (highest → lowest)

1. **Enterprise** managed settings
2. **CLI flags** (`--allowedTools`, etc.)
3. **Local project** `.claude/settings.local.json`
4. **Shared project** `.claude/settings.json`
5. **User global** `~/.claude/settings.json`
6. **Defaults**

---

## 5. Context Management

### CLAUDE.md Hierarchy

| File | Scope | Shared? |
|------|-------|---------|
| `~/.claude/CLAUDE.md` | Global, all projects | No (personal) |
| `.claude/CLAUDE.md` or `CLAUDE.md` | Project root | Yes (git) |
| `CLAUDE.local.md` | Project root | No (auto-gitignored) |
| `subdir/CLAUDE.md` | Subdirectory-specific | Yes |

- Claude searches upward from CWD, loads every CLAUDE.md it finds
- Use `@path/to/file` inside CLAUDE.md to import other files
- `.claude/rules/*.md` — split instructions into focused rule files (all auto-loaded)

### Auto Memory

- Claude records learnings at `~/.claude/projects/<project-hash>/memory/`
- `MEMORY.md` always loaded into system prompt (keep under 200 lines)
- Create topic files (`debugging.md`, `patterns.md`) for details

### Context Window

| Metric | Value |
|--------|-------|
| Window size | 200K tokens |
| Auto-compaction trigger | ~75-92% usage |
| Quality degradation zone | 80-90% usage |
| Recommended manual compact | 65-70% |

**Commands:**
- `/context` — visualize usage as colored grid
- `/compact` — compress conversation history
- `/clear` — nuclear option: wipe history, start fresh

**Pro tip:** Prefer `/clear` + re-stating goals over `/compact`. Compaction is lossy and can drop important context. Use CLAUDE.md for anything that must persist.

---

## 6. Model Selection

### Aliases

| Alias | Model | Best For |
|-------|-------|----------|
| `sonnet` | Claude Sonnet 4.5 | General coding (best cost/perf balance) |
| `opus` | Claude Opus 4.6 | Complex reasoning, architecture |
| `haiku` | Claude Haiku 4.5 | Simple tasks, speed priority |
| `opusplan` | Opus→planning, Sonnet→execution | Complex multi-step projects |

### Full Model IDs

```
claude-opus-4-6
claude-sonnet-4-5-20250929
claude-haiku-4-5-20251001
```

### Extended Context

Append `[1m]` for 1M token context:
```bash
claude --model sonnet[1m]
claude --model claude-sonnet-4-5-20250929[1m]
```

### How to Switch

| Method | Example |
|--------|---------|
| CLI flag | `claude --model opus` |
| Mid-session | `/model opus` |
| Env var | `ANTHROPIC_MODEL=opus` |
| Settings | `"model": "opus"` in settings.json |
| Command frontmatter | `model: haiku` in `.claude/commands/` |

### Effort Levels (Opus 4.6)

Controls thinking depth: `low`, `medium`, `high` (default), `max`

Higher effort = more thinking tokens, more tool calls, more thorough explanations.

---

## 7. Plan Mode & Extended Thinking

### Plan Mode Workflow

1. Enter plan mode: `Shift+Tab` (cycle to plan) or `claude --permission-mode plan`
2. Claude explores codebase with read-only tools
3. Creates implementation plan
4. Uses `AskUserQuestion` to clarify requirements
5. Presents plan for approval
6. On approval: exits plan mode, begins implementation

### Extended Thinking (Opus 4.6)

- Adaptive reasoning: dynamically allocates thinking tokens based on problem complexity
- Toggle thinking visibility with `Tab`
- Higher effort levels produce more thorough analysis

### Opusplan Hybrid Mode

```bash
claude --model opusplan
```

- Planning phase: Opus (complex reasoning, ~10-20% of work)
- Execution phase: Auto-switches to Sonnet (~80-90% of work)
- Best balance of quality and cost for large tasks

---

## 8. Custom Commands & Skills

### Custom Slash Commands

**Location:** `.claude/commands/` (project) or `~/.claude/commands/` (user)

**Example:** `.claude/commands/review.md`
```markdown
---
description: Review code changes for issues
argument-hint: [file-or-directory]
model: opus
allowed-tools: Read, Grep, Glob
---

Review the code at $ARGUMENTS for:
- Security vulnerabilities
- Performance issues
- Code style violations

Provide a summary with severity ratings.
```

### Frontmatter Fields

| Field | Description |
|-------|-------------|
| `description` | Shown in `/help` |
| `argument-hint` | Hint for expected args (e.g. `[file]`) |
| `model` | Force specific model for this command |
| `allowed-tools` | Auto-allow these tools |
| `agent` | Specify subagent to use |

### Argument Substitution

| Variable | Meaning |
|----------|---------|
| `$ARGUMENTS` | All arguments passed to command |
| `$1`, `$2`, etc. | Positional arguments |

### Skills (.claude/skills/)

Skills are enhanced commands with bundled resources:

```
.claude/skills/
  deploy/
    SKILL.md          # Required: frontmatter + instructions
    scripts/deploy.sh # Optional: bundled files
```

**SKILL.md example:**
```markdown
---
name: deploy
description: Deploy application to staging or production
---

Follow the deployment checklist:
1. Run tests
2. Build assets
3. Deploy using `scripts/deploy.sh $ARGUMENTS`
```

Skills support progressive disclosure: metadata loads first (~100 tokens), full instructions load only when matched (~5K tokens), bundled resources load on demand.

---

## 9. Hooks System

### Hook Events

| Event | When It Fires | Uses Matcher? |
|-------|---------------|---------------|
| `PreToolUse` | Before tool executes | Yes |
| `PostToolUse` | After tool succeeds | Yes |
| `PostToolUseFailure` | After tool fails | Yes |
| `PermissionRequest` | When permission asked | Yes |
| `Stop` | Main agent finishes | No |
| `SubagentStop` | Subagent finishes | No |
| `TaskCompleted` | Task completed | No |
| `UserPromptSubmit` | User submits prompt | No |
| `SessionStart` | Session begins | No |
| `Setup` | Repository setup/maintenance | No |
| `Notification` | Permission request or idle 60s+ | No |

### Configuration (in settings.json)

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "prettier --write $CLAUDE_FILE_PATH",
            "timeout": 30
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Claude finished a response'"
          }
        ]
      }
    ]
  }
}
```

### Matcher Patterns

| Pattern | Matches |
|---------|---------|
| `"Write"` | Write tool only |
| `"Write\|Edit"` | Write or Edit |
| `"Bash(npm test*)"` | Bash with npm test commands |
| `"*"` or omit | All tools |

### Hook Control (stdout JSON)

Hooks can return JSON to control Claude's behavior:

```json
{ "continue": false, "stopReason": "Formatting failed" }
```

| Field | Effect |
|-------|--------|
| `continue: false` | Block the tool / stop Claude |
| `stopReason` | Message shown when blocked |
| `suppressOutput` | Hide stdout from transcript |

### Environment Variables in Hooks

- `$CLAUDE_PROJECT_DIR` — project directory path

---

## 10. MCP Servers

### Setup

```bash
claude mcp add                    # Interactive wizard
claude mcp add <name> <command>   # Direct add
claude mcp list                   # List configured servers
claude mcp remove <name>          # Remove server
```

### Configuration Scopes

| Scope | File | Shared? |
|-------|------|---------|
| User (global) | `~/.claude/settings.json` | No |
| Project (shared) | `.claude/settings.json` | Yes |
| Local (private) | `.claude/settings.local.json` | No |

### CLI Flags

```bash
claude --mcp-config servers.json         # Load from file
claude --strict-mcp-config               # Only use --mcp-config servers
claude --verbose --mcp-debug             # Debug MCP connections
```

### In Settings JSON

```json
{
  "mcpServers": {
    "my-server": {
      "command": "npx",
      "args": ["my-mcp-server"],
      "env": { "API_KEY": "..." }
    }
  }
}
```

### Debugging MCP

```bash
claude doctor               # Health check includes MCP
claude --mcp-debug           # Protocol-level diagnostics
MCP_TIMEOUT=10000 claude     # Increase startup timeout to 10s
```

---

## 11. Settings & Configuration

### File Locations

| File | Location | Purpose |
|------|----------|---------|
| User settings | `~/.claude/settings.json` | Global preferences |
| User memory | `~/.claude/CLAUDE.md` | Global instructions |
| User keybindings | `~/.claude/keybindings.json` | Keyboard customization |
| User commands | `~/.claude/commands/*.md` | Personal slash commands |
| Project settings | `.claude/settings.json` | Team-shared config |
| Local settings | `.claude/settings.local.json` | Private project config (gitignored) |
| Project commands | `.claude/commands/*.md` | Project slash commands |
| Project agents | `.claude/agents/*.md` | Custom subagent definitions |
| Project skills | `.claude/skills/*/SKILL.md` | Project skills |
| Project rules | `.claude/rules/*.md` | Split instructions (all auto-load) |
| Enterprise | managed-settings.json | Org-wide policy |

### Key Environment Variables

| Variable | Purpose |
|----------|---------|
| `ANTHROPIC_API_KEY` | API authentication (unset when using subscription) |
| `ANTHROPIC_MODEL` | Default model |
| `CLAUDE_CODE_USE_BEDROCK=1` | Enable AWS Bedrock |
| `AWS_REGION` | Required with Bedrock |
| `CLAUDE_CODE_USE_VERTEX=1` | Enable Google Vertex AI |
| `CLOUD_ML_REGION` | Required with Vertex |
| `ANTHROPIC_VERTEX_PROJECT_ID` | Required with Vertex |
| `CLAUDE_CODE_TASK_LIST_ID` | Share task list across sessions |
| `MCP_TIMEOUT` | MCP server startup timeout (ms) |
| `CLAUDE_PROJECT_DIR` | Available in hooks — project directory |

### .claude/ Directory Structure

```
.claude/
├── settings.json          # Config, hooks, permissions (shared)
├── settings.local.json    # Local overrides (gitignored)
├── CLAUDE.md              # Project instructions (shared)
├── commands/              # Slash commands (.md files)
│   ├── review.md
│   └── deploy.md
├── agents/                # Subagent definitions (.md files)
│   └── code-reviewer.md
├── skills/                # Skills (directories)
│   └── deploy/
│       └── SKILL.md
├── rules/                 # Split instructions (all .md auto-load)
│   ├── style.md
│   └── security.md
└── hooks/                 # Hook scripts
    └── format.sh
```

---

## 12. Subagents & Agent Teams

### Built-in Subagent Types

| Type | Purpose | Tools Available |
|------|---------|-----------------|
| `Explore` | Codebase exploration, research | Read, Glob, Grep, WebSearch, WebFetch |
| `Plan` | Architecture design, planning | Read, Glob, Grep (no editing) |
| `Bash` | Command execution | Bash |
| `general-purpose` | Multi-step tasks | All tools |

### Custom Subagents

**Via `.claude/agents/code-reviewer.md`:**

```markdown
---
description: Expert code reviewer. Use proactively after code changes.
model: sonnet
tools:
  - Read
  - Grep
  - Glob
---

You are a senior code reviewer. Focus on:
- Security vulnerabilities
- Performance issues
- Code quality and maintainability
```

**Via CLI flag:**

```bash
claude --agents '{
  "reviewer": {
    "description": "Code reviewer",
    "prompt": "Review code for issues",
    "tools": ["Read", "Grep", "Glob"],
    "model": "sonnet"
  }
}'
```

### Agent Definition Fields

| Field | Required | Description |
|-------|----------|-------------|
| `description` | Yes | When to invoke this agent |
| `prompt` | Yes (file body or field) | System prompt |
| `tools` | No | Tool allowlist (inherits all if omitted) |
| `disallowedTools` | No | Tools to deny |
| `model` | No | `sonnet`, `opus`, `haiku`, `inherit` |
| `mcpServers` | No | MCP servers for this agent |
| `maxTurns` | No | Max agentic turns |
| `skills` | No | Skills to preload |

### Task System (Multi-Session Coordination)

```bash
# Share tasks across terminal sessions:
export CLAUDE_CODE_TASK_LIST_ID=my-project
claude    # Session 1
claude    # Session 2 — sees same task list
```

Task tools: `TaskCreate`, `TaskUpdate`, `TaskList`, `TaskGet`
Status flow: `pending` → `in_progress` → `completed` (or `deleted`)
Dependencies: `addBlockedBy`, `addBlocks`

---

## 13. Git & GitHub Integration

### GitHub Actions Setup

Run `/install-github-app` once to permanently install the Claude GitHub App + workflow. After setup, `@claude` mentions in PRs/issues trigger automatically on GitHub's infrastructure — no local CLI needed. See `claude-basics/CLAUDE_CODE_DEEP_DIVES.md` for full details.

### PR Sessions

```bash
claude --from-pr 123        # Resume session linked to PR #123
claude --from-pr https://github.com/org/repo/pull/123
```

### Git in Claude Code

Claude uses `gh` CLI for all GitHub operations:
```
gh pr create, gh pr view, gh issue view, gh api repos/org/repo/pulls/123/comments
```

---

## 14. IDE Integrations

### VS Code

- Dedicated sidebar panel with GUI
- Inline diffs in editor
- Visual plan review
- Selection context sharing
- Launch: `Cmd+Esc` (Mac) / `Ctrl+Esc` (Win/Linux)

### JetBrains (IntelliJ, PyCharm, WebStorm, etc.)

- Smart terminal wrapper
- Uses IDE's built-in diff viewer
- Selection context sharing
- Launch: `Cmd+Esc` (Mac) / `Ctrl+Esc` (Win/Linux) or click Claude Code button

### CLI Flag

```bash
claude --ide    # Auto-connect to IDE if exactly one is available
```

---

## 15. Headless / SDK Mode

### Print Mode Essentials

```bash
# Basic headless execution
claude -p "explain this function"

# With output format
claude -p "list endpoints" --output-format json

# Streaming JSON
claude -p "refactor this" --output-format stream-json --include-partial-messages

# Budget controls
claude -p "big refactor" --max-turns 50 --max-budget-usd 5.00

# Structured output
claude -p "extract API schema" --json-schema '{"type":"object","properties":{"endpoints":{"type":"array"}}}'

# Fallback model
claude -p "task" --fallback-model    # Use backup if primary overloaded
```

### Output Formats

| Format | Use Case |
|--------|----------|
| `text` | Human-readable (default) |
| `json` | Structured data with metadata |
| `stream-json` | Real-time streaming for progressive processing |

### Agent SDK

Build custom AI agents with Claude Code's tools and agent loop:

```bash
npm install @anthropic-ai/claude-agent-sdk      # TypeScript
pip install claude-agent-sdk-python              # Python
```

Features: file I/O, shell execution, web search, MCP integration, streaming, multi-turn conversations.

---

## 16. Checkpointing & Rewind

### How It Works

- Every user prompt that triggers a code edit creates a checkpoint
- Only edits via Claude's tools (Write, Edit) are tracked
- External file changes are NOT tracked

### Using Rewind

| Method | Action |
|--------|--------|
| `Esc Esc` | Open rewind menu (scrollable list of all prompts) |
| `/rewind` | Same as above |

### Rewind Options

1. **Restore code and conversation** — revert both to that checkpoint
2. **Restore conversation only** — rewind messages but keep current code

### Limitations

- Only tracks Claude's own file edits
- Does NOT persist across sessions (in-memory only)
- Cannot undo Bash commands (e.g. `rm`, `git reset`)
- `--fork-session` creates a new branch when resuming without modifying original session

---

## 17. Debugging & Troubleshooting

### Diagnostic Commands

```bash
claude doctor                        # Full health check
/doctor                              # Same, inside session
/debug                               # Debug current session
/status                              # Account info, connectivity
```

### Debug Flags

```bash
claude --verbose                     # Full turn-by-turn output
claude --debug "api,hooks"           # Debug specific categories
claude --debug "!statsig,!file"      # Exclude noisy categories
claude --verbose --mcp-debug         # Deep MCP diagnostics
```

### What `/doctor` Checks

- API key presence and format
- Node.js version compatibility
- Network connectivity to Anthropic APIs
- MCP server health and connections
- Settings file syntax validation
- Permission rule syntax
- Log file locations

### Common Fixes

| Problem | Solution |
|---------|----------|
| API key issues | `/doctor`, check `ANTHROPIC_API_KEY` |
| MCP disconnects | `claude --mcp-debug`, check `MCP_TIMEOUT` |
| Tools failing | Check permission rules in settings |
| Session hanging | `--verbose` to see what's stuck |
| Outdated version | `claude update` |
| Context exhausted | `/compact` or `/clear` |
| CLAUDE.md not loading | Check file location, use `/context` |

---

## 18. Power-User Workflows

### Repo Onboarding

```bash
claude --model opus
> /init                              # Generate CLAUDE.md
> Explore the codebase architecture, document key patterns in CLAUDE.md
```

### Large Refactoring (Safe)

```bash
claude --permission-mode plan --model opusplan
> Analyze all API endpoints and propose a migration to REST v2
# Review plan → approve → Claude switches to execution mode
```

### CI/CD Code Review

```bash
git diff main...HEAD | claude -p "Review these changes. Flag security issues, bugs, and style violations." --output-format json
```

### Multi-Session Task Coordination

```bash
# Terminal 1
export CLAUDE_CODE_TASK_LIST_ID=sprint-42
claude
> Create tasks for the auth refactor: database schema, API layer, frontend, tests

# Terminal 2
export CLAUDE_CODE_TASK_LIST_ID=sprint-42
claude
> Pick up the next available task and start working on it
```

### Quick Bug Investigation

```bash
claude -c -p "The login page throws a 500 error after the last deploy. Investigate."
```

### Auto-Format on Save (Hook)

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "prettier --write \"$CLAUDE_FILE_PATH\"",
        "timeout": 10
      }]
    }]
  }
}
```

### Test-After-Edit (Hook)

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "npm test --silent 2>&1 | tail -5",
        "timeout": 60
      }]
    }]
  }
}
```

### Custom Subagent for Reviews

`.claude/agents/security-reviewer.md`:
```markdown
---
description: Security-focused code reviewer. Use after any code changes.
model: opus
tools:
  - Read
  - Grep
  - Glob
---

You are a security engineer. Review all code changes for:
- OWASP Top 10 vulnerabilities
- Injection attacks (SQL, command, XSS)
- Authentication/authorization flaws
- Secrets or credentials in code
- Insecure dependencies

Rate each finding: Critical / High / Medium / Low.
```

### Headless Pipeline with Structured Output

```bash
claude -p "Analyze the test coverage gaps" \
  --output-format json \
  --json-schema '{
    "type": "object",
    "properties": {
      "uncovered_files": {"type": "array", "items": {"type": "string"}},
      "coverage_percentage": {"type": "number"},
      "recommendations": {"type": "array", "items": {"type": "string"}}
    }
  }'
```

### Context-Efficient Session Recovery

Instead of `/compact` (lossy), use `/clear` + a custom `/catchup` command:

`.claude/commands/catchup.md`:
```markdown
---
description: Reload context after /clear
---

Read the following files to re-establish context:
1. CLAUDE.md for project rules
2. The STATUS.md file for current progress
3. Recent git log (last 10 commits)

Summarize what we were working on and what's next.
```

---

*Generated February 2026. For latest docs: [code.claude.com/docs](https://code.claude.com/docs)*
