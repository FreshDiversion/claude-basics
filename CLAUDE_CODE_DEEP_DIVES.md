# Claude Code Deep Dives

> In-depth explanations of Claude Code features that go beyond the cheat sheet.
> For quick reference, see `claude-basics/CLAUDE_CODE_CHEATSHEET.md`.

---

## Table of Contents

- [Agent Teams](#agent-teams)
- [CLAUDE.md Best Practices](#claudemd-best-practices)
- [Custom Commands & Skills](#custom-commands--skills)
- [/install-github-app](#install-github-app)
- [Hooks System](#hooks-system)
- [MCP Servers](#mcp-servers)
- [Permissions & Auto-Approval](#permissions--auto-approval)
- [Subagents](#subagents)
- [Tokenization & How LLMs Read Text](#tokenization--how-llms-read-text)

---

## Agent Teams

**Status:** Experimental (Research Preview) — disabled by default, requires opt-in.

### What Are Agent Teams?

Multiple Claude Code instances working in parallel, each with its own full context window, coordinating via peer-to-peer messaging and a shared task list. One session is the **team lead**, the others are **teammates**.

### Enable

```json
// .claude/settings.json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Or via environment variable:
```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

### Agent Teams vs. Subagents

| | Subagents | Agent Teams |
|--|-----------|-------------|
| Communication | Report back to caller only | Peer-to-peer messaging between teammates |
| Context | Share caller's context budget | Each gets own full context window |
| Coordination | Caller manages everything | Shared task list with auto-dependency tracking |
| Cost | ~1.2-1.5x baseline | 2-5x baseline (each teammate is a full instance) |
| Nesting | Can be spawned by any agent | Only the lead can spawn teammates |
| Best for | Quick, focused tasks | Complex parallel work requiring discussion |
| Status | Production | Experimental |

### Core Concepts

**Team Lead** — the session that creates the team. Coordinates work, assigns tasks, synthesizes results. Cannot be transferred.

**Teammates** — independent Claude instances working in parallel. Each has its own full context window, tool access, and permissions.

**Delegate Mode** — restricts the lead to coordination-only tools (can't edit code, run commands, or read files). Forces proper work decomposition. Toggle with `Shift+Tab` or set in agent config:

```markdown
---
name: coordinator
description: Orchestrates work across specialized agents
permissionMode: delegate
---
```

**Mailbox System** — each teammate has an inbox file at `~/.claude/teams/{team-name}/inboxes/{agent-id}.json`. Messages are delivered automatically (not polled). The lead's text output is NOT visible to teammates — must use `write` to communicate.

**Shared Task List** — tasks with status tracking and dependency chains. Tasks auto-unblock as dependencies complete. Stored at `~/.claude/tasks/{team-name}/`. File locking prevents race conditions when teammates claim tasks.

### Display Modes

| Mode | Behavior | Requirement |
|------|----------|-------------|
| `auto` (default) | Uses tmux if available, otherwise in-process | — |
| `in-process` | All teammates in main terminal | `Shift+Up/Down` to select |
| `tmux` | Separate panes per teammate | tmux or iTerm2 |

Set via CLI: `claude --teammate-mode in-process`
Set via settings: `"teammateMode": "in-process"` in settings.json

**In-process controls:**
- `Shift+Up/Down` — select a teammate
- Type — send them a message
- `Enter` — view their full session
- `Escape` — interrupt their current turn

### TeammateTool Operations

| Operation | Purpose |
|-----------|---------|
| `spawnTeam` | Create a team (you become leader) |
| `write` | Direct message to a specific teammate |
| `broadcast` | Message all teammates (use sparingly — scales with team size) |
| `approvePlan` | Approve a teammate's plan before they execute |
| `requestShutdown` | Ask teammate to gracefully shut down |
| `approveShutdown` | Teammate confirms and terminates |
| `cleanup` | Remove team directories after all teammates exit |

### Task Dependency System

Tasks flow through: `pending` → `in_progress` → `completed` (or `deleted`)

Blocked tasks auto-unblock when their dependencies complete:

```
Task 1: Research OAuth libraries          → owner: researcher
Task 2: Design implementation (blocked by 1) → owner: architect
Task 3: Implement OAuth (blocked by 2)       → owner: implementer
Task 4: Write tests (blocked by 3)           → owner: tester
Task 5: Code review (blocked by 4)           → owner: reviewer
```

Researcher finishes → architect auto-unblocks → and so on. No manual orchestration needed.

### Hook Events for Teams

```json
{
  "hooks": {
    "TeammateIdle": [{
      "hooks": [{ "type": "command", "command": "./scripts/check-progress.sh" }]
    }],
    "TaskCompleted": [{
      "hooks": [{ "type": "command", "command": "./scripts/verify-quality.sh" }]
    }]
  }
}
```

- `TeammateIdle` — fires when a teammate finishes work. Exit code 2 sends feedback.
- `TaskCompleted` — fires when a task is marked complete. Exit code 2 prevents completion.

### Practical Examples

#### Parallel Security Review

```
Create an agent team to review PR #142 for security vulnerabilities.

Spawn three security specialists:
1. Auth & Authorization reviewer — token handling, session mgmt, RBAC (src/auth/)
2. Input Validation reviewer — XSS, SQL injection, sanitization (src/handlers/)
3. Secrets & Config reviewer — hardcoded credentials, env handling (config/)

Have them message each other about overlapping findings.
```

#### Competing Hypothesis Debugging

```
Users report the app exits after one message instead of staying connected.

Create an agent team with 5 teammates, each investigating a different hypothesis:
1. Network/Connection issues
2. Memory leak
3. Event loop/Async handling
4. Protocol/Message parsing
5. OS-level resource depletion

Have them debate and try to disprove each other's theories.
```

This fights anchoring bias — multiple independent investigators actively challenging each other converge on the root cause faster than sequential investigation.

#### Custom Team Lead Agent

`.claude/agents/feature-lead.md`:
```markdown
---
name: feature-lead
description: Orchestrate feature development team
tools: Task, Read, Bash
permissionMode: delegate
---

You are the lead for feature development.

Responsibilities:
1. Create tasks with clear dependencies
2. Spawn specialized teammates for their domains
3. Communicate progress and blockers via write
4. Review and approve plans before implementation
5. Synthesize findings into actionable decisions
```

### When to Use Teams

**Good fit:**
- Parallel code reviews across different domains
- Competing-hypothesis debugging (scientific debate structure)
- Multi-specialist feature development with discussion between specialists
- Large codebase exploration from multiple angles

**Bad fit:**
- Sequential/dependent work (single session is simpler and cheaper)
- Simple tasks (overhead isn't worth it)
- Cost-sensitive work (each teammate is a full Claude instance)

### Limitations

| Limitation | Detail |
|-----------|--------|
| No nested teams | Only the lead can spawn teammates |
| No session resumption | In-process teammates lost on `/resume` or `/rewind` |
| File conflict risk | Two teammates editing the same file causes overwrites — assign different files |
| Fixed leadership | Can't transfer or promote |
| One team per session | Clean up current team before starting another |
| VS Code terminal | Split panes don't work — use in-process mode |
| No per-teammate permissions at spawn | Change modes after spawning if needed |
| Context inheritance | Teammates don't get lead's history — include context in spawn prompt |

---

## CLAUDE.md Best Practices

CLAUDE.md is how you give Claude persistent instructions — project conventions, build commands, code style, architecture notes. It's loaded into context every turn, so it's always "remembered" even after compaction. But that also means every line costs tokens on every message.

### The Golden Rule

**CLAUDE.md is reference material, not a conversation.** Write it like a style guide, not a chat message. Claude reads it, it doesn't execute it. "You MUST always run tests first" won't reliably trigger behavior — use a [hook](#hooks-system) or [custom command](#) for that.

### File Types & Where They Go

| File | Location | Shared? | Purpose |
|------|----------|---------|---------|
| `~/.claude/CLAUDE.md` | Home directory | No | Personal preferences across all projects |
| `CLAUDE.md` or `.claude/CLAUDE.md` | Project root | Yes (git) | Team conventions, build commands, architecture |
| `CLAUDE.local.md` | Project root | No (auto-gitignored) | Your private project notes |
| `subdir/CLAUDE.md` | Any subdirectory | Yes | Context specific to that part of the codebase |

Claude searches upward from the current working directory and loads every CLAUDE.md it finds. Deeper files add specificity on top of broader ones.

### Keep It Lean

Every line of CLAUDE.md is loaded into context on **every single turn**. A 500-line CLAUDE.md wastes ~1,500 tokens per message — that adds up fast across a session.

**Target sizes:**
- Root CLAUDE.md: **~20-30 lines** (essentials only)
- Sub-project CLAUDE.md: **~30-50 lines** (component-specific detail)
- Total across all loaded files: **under 100 lines**

### What Belongs in CLAUDE.md

**Yes — put these in:**
- Build/test/run commands (`npm test`, `cargo build --release`)
- Code style rules ("use snake_case for Python, camelCase for TypeScript")
- Architecture overview (3-5 sentences, not a novel)
- File structure guidance ("API routes are in `src/routes/`, tests mirror `src/` in `tests/`")
- Common gotchas ("always use `BigDecimal` for currency, never `float`")
- Links to important docs (PLAN.md, STATUS.md, design docs)

**No — don't put these in:**
- "You MUST do X at session start" (unreliable — use `/commands` or hooks instead)
- Detailed API documentation (too many tokens — link to it or let Claude read the file)
- Full file contents or large code samples (Claude can read the actual files)
- Step-by-step workflows (use custom commands in `.claude/commands/`)
- Temporary notes (use CLAUDE.local.md for scratch, or auto memory)

### Example: Good Root CLAUDE.md

```markdown
# MyApp

Web app: React frontend + Python FastAPI backend.

## Build & Test
- Frontend: `cd frontend && npm test`
- Backend: `cd backend && pytest`
- Full stack: `docker compose up`

## Code Style
- Python: Black formatter, snake_case, type hints on public functions
- TypeScript: Prettier, camelCase, strict mode
- Commits: conventional commits (feat:, fix:, chore:)

## Architecture
- `frontend/` — React SPA, pages in `src/pages/`, API client in `src/api/`
- `backend/` — FastAPI, routes in `app/routes/`, models in `app/models/`
- `infra/` — Terraform for AWS

## Key Docs
- `docs/PLAN.md` — implementation roadmap
- `docs/API.md` — API specification
```

~20 lines. Everything Claude needs to work effectively. Nothing that wastes tokens.

### Example: Good Sub-Project CLAUDE.md

`backend/CLAUDE.md`:
```markdown
# Backend (FastAPI)

## Commands
- Run: `uvicorn app.main:app --reload`
- Test: `pytest -x --tb=short`
- Migrate: `alembic upgrade head`

## Conventions
- All routes return Pydantic models, never raw dicts
- Use `Depends()` for auth — see `app/deps/auth.py`
- Database access through repository pattern: `app/repos/`
- Exceptions go through `app/exceptions.py` handlers

## Don't
- Don't use `db.execute()` directly in routes — use repos
- Don't add new deps without checking `pyproject.toml` for existing alternatives
```

### Splitting Large Instructions: `.claude/rules/`

If you have many rules that don't all apply everywhere, split them into `.claude/rules/`:

```
.claude/rules/
├── style.md          # Code style rules
├── security.md       # Security requirements
├── testing.md        # Test conventions
└── deployment.md     # Deploy guidelines
```

All `.md` files in `.claude/rules/` auto-load. This keeps the root CLAUDE.md lean while still providing comprehensive guidance.

### Importing Files

Use `@path/to/file` inside CLAUDE.md to import other files:

```markdown
# MyProject

@docs/architecture.md
@docs/coding-standards.md
```

Imported files count toward your token budget, so import selectively.

### CLAUDE.local.md — Your Private Notes

Auto-gitignored. Use for:
- Personal environment setup notes
- Temporary task context ("currently refactoring the auth module")
- Preferences that differ from the team ("I prefer verbose test output")

### Common Mistakes

| Mistake | Why It's Bad | Do This Instead |
|---------|-------------|-----------------|
| 500-line CLAUDE.md | Wastes ~1,500 tokens every turn | Keep under 30 lines at root |
| "Always run tests after changes" | Claude doesn't reliably follow imperative instructions | Use a PostToolUse hook |
| Pasting full API docs | Massive token cost loaded every turn | Link to file, let Claude read when needed |
| Duplicating info in CLAUDE.md and sub-project CLAUDE.md | Tokens wasted twice | Root = general, sub = specific |
| Putting secrets or credentials | May be committed to git | Use `.env` files + `.gitignore` |
| Detailed step-by-step workflows | Belongs in runnable commands | Use `.claude/commands/workflow.md` |

---

## Custom Commands & Skills

Custom commands and skills let you create reusable, shareable workflows that run inside Claude Code. Think of them as your own slash commands — `/review`, `/deploy`, `/fix-issue` — tailored to your project.

### Commands vs. Skills

As of January 2026, commands and skills were **merged**. Commands still work, skills are a superset with extra features. No migration required.

| Feature | Commands (`.claude/commands/`) | Skills (`.claude/skills/`) |
|---------|-------------------------------|---------------------------|
| File format | Single `.md` file | Directory with `SKILL.md` + supporting files |
| Supporting files | No | Yes (templates, scripts, examples) |
| Auto-invocation | No (manual `/name` only) | Yes (Claude triggers when relevant) |
| Subagent execution | No | Yes (`context: fork`) |
| Dynamic context | No | Yes (`!`command`` preprocessing) |
| Monorepo discovery | No | Yes (nested `.claude/skills/` found automatically) |

**Bottom line:** Use commands for quick personal workflows. Use skills when you need supporting files, auto-invocation, or subagent execution.

### Creating a Command

Drop a `.md` file in `.claude/commands/` (project) or `~/.claude/commands/` (personal):

`.claude/commands/review.md`:
```markdown
---
description: Review code for quality and security issues
argument-hint: [file-path]
allowed-tools: Read, Grep, Glob
model: opus
---

Review the code at $ARGUMENTS for:
1. Security vulnerabilities
2. Performance issues
3. Code style violations

Provide a summary with severity ratings.
```

Invoke with: `/review src/auth.ts`

### Creating a Skill

Create a directory in `.claude/skills/` with a `SKILL.md`:

```
.claude/skills/
  pr-review/
    SKILL.md           # Required: instructions + frontmatter
    CHECKLIST.md        # Optional: detailed review checklist
    examples/
      good-review.md   # Optional: example output
```

`.claude/skills/pr-review/SKILL.md`:
```markdown
---
name: pr-review
description: Review pull requests for code quality, tests, and docs
argument-hint: [pr-number]
allowed-tools: Read, Grep, Glob, Bash(gh *)
context: fork
agent: Explore
---

Review PR #$0 comprehensively.

## Context
- Changes: !`gh pr diff $0 --stat`
- Files: !`gh pr diff $0 --name-only`

## Process
1. Analyze the diff
2. Check test coverage
3. Review documentation changes

For detailed checklist, see [CHECKLIST.md](CHECKLIST.md).
```

### All Frontmatter Fields

| Field | Default | Description |
|-------|---------|-------------|
| `name` | Filename/directory name | Skill identifier (lowercase, hyphens, max 64 chars) |
| `description` | First paragraph of markdown | What it does. Claude uses this for auto-invocation decisions |
| `argument-hint` | None | Shown in autocomplete: `[file] [format]` |
| `allowed-tools` | None | Tools auto-approved while skill is active |
| `model` | Project default | Force a specific model |
| `disable-model-invocation` | `false` | If `true`, only manual `/name` invocation — Claude can't auto-trigger |
| `user-invocable` | `true` | If `false`, hides from `/` menu but Claude can still auto-invoke |
| `context` | None | Set to `fork` to run in isolated subagent context |
| `agent` | `general-purpose` | Subagent type when `context: fork` (`Explore`, `Plan`, or custom) |
| `hooks` | None | Skill-scoped hooks (same format as global hooks) |

### Argument Substitution

| Variable | Meaning | Example |
|----------|---------|---------|
| `$ARGUMENTS` | All args as one string | `Fix issue $ARGUMENTS` |
| `$0`, `$1`, `$2` | Positional (0-indexed) | `Migrate $0 from $1 to $2` |
| `$ARGUMENTS[0]` | Explicit indexing | Same as `$0` |
| `${CLAUDE_SESSION_ID}` | Current session ID | For logging or correlation |

If you don't include `$ARGUMENTS` anywhere, Claude Code appends `ARGUMENTS: <user input>` at the end automatically.

### Dynamic Context (`` !`command` ``)

Run shell commands **before** Claude sees the prompt. Output replaces the placeholder:

```markdown
---
name: standup
description: Generate standup summary
---

## What I did yesterday
!`git log --oneline --since="yesterday" --author="$(git config user.name)"`

## Open PRs
!`gh pr list --author=@me`

Summarize the above into a standup update.
```

The `!` commands are **preprocessing** — they run immediately, and Claude only sees the output (not the commands).

### Subagent Execution (`context: fork`)

Run a skill in an isolated context window so it doesn't clutter your main conversation:

```markdown
---
name: deep-research
description: Research codebase architecture thoroughly
context: fork
agent: Explore
---

Research $ARGUMENTS:
1. Find relevant files with Glob and Grep
2. Read and analyze key components
3. Map dependencies and relationships
4. Summarize findings with file references
```

Results are summarized back to the main conversation. The subagent's full exploration stays in its own context.

### Controlling Who Can Invoke

| Setting | You invoke | Claude invokes | When to use |
|---------|-----------|---------------|-------------|
| Default (both fields unset) | Yes | Yes | Most skills — available to everyone |
| `disable-model-invocation: true` | Yes | No | Side-effect workflows (deploy, commit) |
| `user-invocable: false` | No | Yes | Background knowledge Claude draws on |

### Progressive Disclosure (How Skills Save Tokens)

Skills don't load everything at once:

1. **Always in context:** Skill name + description only (~100 tokens each)
2. **On invocation:** Full SKILL.md content loads (~500-5,000 tokens)
3. **On reference:** Supporting files load only when Claude reads them

With 50 skills defined, only descriptions are in context. The full instructions load only when a skill is actually used.

### Scopes

| Scope | Location | Shared? |
|-------|----------|---------|
| Project | `.claude/commands/` or `.claude/skills/` | Yes (git) |
| User | `~/.claude/commands/` or `~/.claude/skills/` | No (personal, all projects) |
| Plugin | `my-plugin/skills/` | Yes (distributed) |

Plugin skills are namespaced: `/my-plugin:skill-name`

### Monorepo Discovery (Skills Only)

Claude discovers skills from nested directories:

```
project/
├── .claude/skills/shared-conventions/SKILL.md     # Always available
├── packages/
│   ├── frontend/.claude/skills/
│   │   └── component-patterns/SKILL.md            # Available in frontend/
│   └── backend/.claude/skills/
│       └── api-patterns/SKILL.md                  # Available in backend/
```

### Practical Examples

#### Daily Standup Generator

`.claude/commands/standup.md`:
```markdown
---
description: Generate standup summary from git activity
---

## Yesterday's commits
!`git log --oneline --since="yesterday" --author="$(git config user.name)"`

## Open PRs
!`gh pr list --author=@me --state=open`

Write a concise standup update from the above.
```

#### Context Reload After `/clear`

`.claude/commands/catchup.md`:
```markdown
---
description: Reload context after clearing conversation
allowed-tools: Read, Bash(git log *)
---

Read these files to re-establish context:
1. CLAUDE.md
2. The project STATUS.md file
3. Last 10 git commits: !`git log --oneline -10`

Summarize what we're working on and what's next.
```

#### Architecture Explorer (Subagent)

`.claude/skills/explore-architecture/SKILL.md`:
```markdown
---
name: explore-architecture
description: Map the architecture of a codebase component
argument-hint: [component-or-directory]
context: fork
agent: Explore
---

Explore the architecture of $ARGUMENTS:
1. Find all source files with Glob
2. Identify entry points and exports
3. Map internal dependencies
4. Document key patterns and data flow

Return a concise architecture summary with file references.
```

#### Safe Deploy (Manual Only)

`.claude/skills/deploy/SKILL.md`:
```markdown
---
name: deploy
description: Deploy to production
argument-hint: [environment]
disable-model-invocation: true
allowed-tools: Bash(npm *), Bash(git *), Read
---

Deploy to $0:
1. Verify tests pass: `npm test`
2. Check we're on main: `git branch --show-current`
3. Build: `npm run build`
4. Deploy: `npm run deploy:$0`
5. Verify deployment health
```

`disable-model-invocation: true` means Claude will never trigger this on its own — you must explicitly type `/deploy production`.

### Using Commands in Headless Mode

```bash
claude -p "/review src/auth.ts" --output-format json
claude -p "/standup" --output-format text
```

Built-in commands (`/compact`, `/clear`) don't work in headless mode. Skills with `disable-model-invocation: true` also don't work headless.

### Gotchas

| Gotcha | Detail |
|--------|--------|
| Subdirectories don't namespace | `.claude/commands/frontend/test.md` is `/test`, not `/frontend/test` |
| Missing `$ARGUMENTS` | If not included, args are appended at the end — may be confusing |
| `` !`commands` `` run immediately | They're preprocessing, not something Claude controls |
| Description budget | Many skills = some descriptions excluded from context. Keep them concise |
| `allowed-tools` doesn't override denies | Global deny rules still apply within a skill |
| Commands can't have supporting files | Migrate to skills if you need templates/scripts alongside |

---

## /install-github-app

### What It Is

A **one-time interactive setup wizard** that permanently installs Claude's GitHub integration on your repository. After running it once, the integration works independently — no local Claude Code session needed.

### What It Does (Step by Step)

1. Verifies you're a **repo admin** (required)
2. Installs the official **Claude GitHub App** (`github.com/apps/claude`)
3. Stores your `ANTHROPIC_API_KEY` (or OAuth token) as an **encrypted GitHub repo secret**
4. Creates `.github/workflows/claude.yml` using `anthropics/claude-code-action@v1`

### Requirements

- Repository admin access
- Direct Anthropic API (not Bedrock or Vertex — those require manual setup)

### What You Get

After setup, `@claude` mentions in PRs and issues trigger the GitHub Action automatically on GitHub's infrastructure:

```
@claude review this PR for security issues
@claude fix the TypeError on line 42
@claude explain the auth flow in this codebase
```

### App Permissions

| Permission | Level | Why |
|-----------|-------|-----|
| Contents | Read & Write | Read code, push commits |
| Issues | Read & Write | Respond to issue mentions |
| Pull Requests | Read & Write | Comment on and modify PRs |

### The Workflow It Creates

```yaml
# .github/workflows/claude.yml
on:
  issue_comment:
    types: [created]
jobs:
  claude:
    if: contains(github.event.comment.body, '@claude')
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: anthropics/claude-code-action@v1
```

Add `contents: write` if Claude should push commits (not just comment).

### Lifecycle

- **Install:** `/install-github-app` (once)
- **Runs on:** GitHub's infrastructure, independent of your local machine
- **Remove:** Uninstall the app from GitHub repo settings + delete the workflow file
- **Secrets:** Encrypted at rest by GitHub, never logged, only available to workflows in that repo

---

## Hooks System

Hooks are shell commands or LLM prompts that run automatically in response to Claude Code events. They let you validate, block, transform, or extend Claude's behavior without modifying Claude itself.

### All Hook Events

| Event | Fires When | Can Block? | Matcher Filters On |
|-------|-----------|-----------|-------------------|
| `SessionStart` | Session begins or resumes | No | `startup`, `resume`, `clear`, `compact` |
| `SessionEnd` | Session terminates | No | `clear`, `logout`, `prompt_input_exit`, `other` |
| `UserPromptSubmit` | User submits a prompt | Yes | No matcher (always fires) |
| `PreToolUse` | Before tool executes | Yes | Tool name |
| `PostToolUse` | After tool succeeds | No | Tool name |
| `PostToolUseFailure` | After tool fails | No | Tool name |
| `PermissionRequest` | Permission dialog shown | Yes | Tool name |
| `Stop` | Claude finishes responding | Yes | No matcher (always fires) |
| `SubagentStart` | Subagent spawned | No | Agent type |
| `SubagentStop` | Subagent finishes | Yes | Agent type |
| `TaskCompleted` | Task marked complete | Yes | No matcher (always fires) |
| `TeammateIdle` | Team teammate going idle | Yes | No matcher (always fires) |
| `Notification` | Notification sent | No | `permission_prompt`, `idle_prompt`, `auth_success` |
| `PreCompact` | Before context compaction | No | `manual`, `auto` |

When multiple hooks match the same event, they all run **in parallel**.

### Configuration Format

Hooks live in settings.json files (user, project, or local):

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "ToolNameOrPattern",
        "hooks": [
          {
            "type": "command",
            "command": "your-script.sh",
            "timeout": 600
          }
        ]
      }
    ]
  }
}
```

### Three Hook Types

| Type | What It Does | Timeout Default | Tool Access |
|------|-------------|-----------------|-------------|
| `command` | Runs a shell command. Receives JSON on stdin, controls via exit code + stdout JSON | 600s (10 min) | None (shell only) |
| `prompt` | Single-turn LLM call. Returns `{"ok": true/false, "reason": "..."}` | 30s | None |
| `agent` | Multi-turn LLM subagent. Up to 50 tool-use turns. Same return format as prompt | 60s | Read, Grep, Glob, Bash |

### Matcher Syntax

Matchers are **regex patterns**, **case-sensitive**:

```json
"matcher": "Bash"              // exact match
"matcher": "Edit|Write"        // OR — matches Edit or Write
"matcher": "mcp__memory__.*"   // regex — all memory server tools
"matcher": ""                  // empty = match everything
// omit matcher entirely = match everything
```

MCP tools follow the naming convention: `mcp__<server>__<tool>`

### What Hooks Receive (stdin JSON)

Every hook gets session metadata plus event-specific data on stdin.

**PreToolUse example (Bash):**
```json
{
  "session_id": "abc-123",
  "hook_event_name": "PreToolUse",
  "tool_name": "Bash",
  "tool_use_id": "toolu_01ABC...",
  "tool_input": {
    "command": "npm test",
    "description": "Run tests",
    "timeout": 120000
  }
}
```

**PostToolUse example (Write):**
```json
{
  "session_id": "abc-123",
  "hook_event_name": "PostToolUse",
  "tool_name": "Write",
  "tool_input": { "file_path": "/src/app.ts", "content": "..." },
  "tool_response": { "filePath": "/src/app.ts", "success": true }
}
```

**UserPromptSubmit:**
```json
{
  "session_id": "abc-123",
  "hook_event_name": "UserPromptSubmit",
  "prompt": "Fix the login bug"
}
```

**Stop:**
```json
{
  "session_id": "abc-123",
  "hook_event_name": "Stop",
  "stop_hook_active": false
}
```

### Exit Codes

| Code | Meaning | Effect |
|------|---------|--------|
| `0` | Success | Action proceeds. Stdout JSON parsed for control fields |
| `2` | Block | Action blocked. Stderr fed to Claude as error context |
| Other | Non-blocking error | Stderr logged, execution continues |

**Exit code 2 by event:**

| Event | What Exit 2 Does |
|-------|-----------------|
| `PreToolUse` | Prevents tool execution |
| `UserPromptSubmit` | Blocks prompt, erases from context |
| `PermissionRequest` | Denies permission |
| `Stop` | Prevents Claude from stopping (careful: can loop!) |
| `TaskCompleted` | Task not marked complete |
| `PostToolUse` | Can't undo (already ran) — stderr shown to Claude |
| `Notification` | No effect — stderr shown to user only |

### Stdout JSON Control

Command hooks can return JSON on stdout (with exit code 0) for fine-grained control:

**Universal fields (all events):**
```json
{
  "continue": false,           // stop Claude entirely
  "stopReason": "Build failed", // shown to user when continue=false
  "suppressOutput": true,      // hide from verbose mode
  "systemMessage": "Warning..."// shown to user
}
```

**PreToolUse — allow, deny, or modify the tool call:**
```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "allow",
    "permissionDecisionReason": "Auto-approved git command",
    "updatedInput": { "command": "git log --oneline -10" },
    "additionalContext": "Extra info for Claude"
  }
}
```

`permissionDecision` options: `"allow"`, `"deny"`, `"ask"` (show normal permission prompt)

**PostToolUse — add context or block further action:**
```json
{
  "decision": "block",
  "reason": "Tests failed after this edit",
  "hookSpecificOutput": {
    "hookEventName": "PostToolUse",
    "additionalContext": "Test output: 3 failures..."
  }
}
```

**Stop — prevent Claude from stopping:**
```json
{
  "decision": "block",
  "reason": "Not all tasks are complete — keep working"
}
```

### Environment Variables

| Variable | Available In | Purpose |
|----------|-------------|---------|
| `$CLAUDE_PROJECT_DIR` | All hooks | Project root directory |
| `$CLAUDE_ENV_FILE` | SessionStart only | File path to persist env vars for the session |
| `$CLAUDE_PLUGIN_ROOT` | Plugin hooks | Plugin root directory |
| `$CLAUDE_CODE_REMOTE` | All hooks | `"true"` in remote web environments |

**Persisting env vars (SessionStart only):**
```bash
#!/bin/bash
echo 'export NODE_ENV=production' >> "$CLAUDE_ENV_FILE"
echo 'export DEBUG=true' >> "$CLAUDE_ENV_FILE"
```

All variables written to `$CLAUDE_ENV_FILE` are available in all subsequent Bash commands during the session.

### Async Hooks

Only for `type: "command"`. Claude doesn't wait — fires and continues immediately.

```json
{
  "type": "command",
  "command": "./run-tests.sh",
  "async": true,
  "timeout": 300
}
```

Cannot block or control behavior (action already completed). If the hook outputs `systemMessage` or `additionalContext`, it's delivered to Claude on the next turn.

Useful for: test suites, deployments, external API calls, long-running validations.

### Practical Examples

#### Block Dangerous Commands

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": ".claude/hooks/block-rm.sh"
      }]
    }]
  }
}
```

**block-rm.sh:**
```bash
#!/bin/bash
COMMAND=$(cat | jq -r '.tool_input.command')
if echo "$COMMAND" | grep -qE 'rm -rf|drop table|truncate'; then
  echo '{"hookSpecificOutput":{"hookEventName":"PreToolUse","permissionDecision":"deny","permissionDecisionReason":"Destructive command blocked"}}'
else
  exit 0
fi
```

#### Auto-Format After Edits

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write"
      }]
    }]
  }
}
```

#### Re-inject Context After Compaction

```json
{
  "hooks": {
    "SessionStart": [{
      "matcher": "compact",
      "hooks": [{
        "type": "command",
        "command": "echo 'Reminder: Use Bun, not npm. Always run tests before committing.'"
      }]
    }]
  }
}
```

Stdout from exit-0 SessionStart hooks is added to Claude's context — great for re-injecting rules that get lost during compaction.

#### macOS Notification on Permission Needed

```json
{
  "hooks": {
    "Notification": [{
      "matcher": "permission_prompt",
      "hooks": [{
        "type": "command",
        "command": "osascript -e 'display notification \"Claude needs your permission\" with title \"Claude Code\"'"
      }]
    }]
  }
}
```

#### Async Test Runner After File Writes

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write",
      "hooks": [{
        "type": "command",
        "command": ".claude/hooks/run-tests.sh",
        "async": true,
        "timeout": 120
      }]
    }]
  }
}
```

#### LLM-Based Stop Guard

```json
{
  "hooks": {
    "Stop": [{
      "hooks": [{
        "type": "agent",
        "prompt": "Check if all tasks in the task list are complete. If not, return {\"ok\": false, \"reason\": \"Tasks X, Y still pending\"}. If all done, return {\"ok\": true}.",
        "timeout": 60
      }]
    }]
  }
}
```

### Debugging Hooks

```bash
claude --debug "hooks"           # Hook-specific debug output
```

In-session: press `Ctrl+O` to toggle verbose mode and see hook execution in the transcript.

**Test a hook script manually:**
```bash
echo '{"tool_name":"Bash","tool_input":{"command":"ls"}}' | ./my-hook.sh
echo $?   # Check exit code
```

### Gotchas

| Gotcha | Detail |
|--------|--------|
| Matchers are case-sensitive | `bash` won't match `Bash` |
| Shell profile noise | If your `.bashrc` prints output, it corrupts JSON parsing. Guard with `if [[ $- == *i* ]]` |
| Stop hooks can loop | Always check `stop_hook_active` — if true, exit 0 immediately |
| Settings are snapshots | Editing settings mid-session doesn't apply. Use `/hooks` to reload |
| PostToolUse can't undo | The tool already ran. Exit 2 just feeds stderr to Claude as context |
| PermissionRequest skipped in headless | Use PreToolUse instead for `-p` mode |
| Env vars can be empty | Known issue — prefer reading stdin JSON over environment variables |
| Use absolute paths | `"command"` runs in a shell — relative paths may resolve wrong. Use `"$CLAUDE_PROJECT_DIR"` |

---

## MCP Servers

MCP (Model Context Protocol) is an open protocol that connects Claude Code to external tools, databases, APIs, and services. It lets Claude query your Postgres database, file GitHub issues, send Slack messages, or call any custom API — all from within a conversation.

> **Prerequisite:** Most MCP servers launch via `npx`, which requires **Node.js 18+**. Install it before setting up MCP servers (`brew install node` on macOS, `winget install OpenJS.NodeJS.LTS` on Windows, `sudo apt install nodejs npm` on Linux). Verify with `node --version`.

### What MCP Unlocks

```
"Implement the feature from JIRA ticket ENG-4521 and create a PR"
"Check Sentry for errors related to the checkout flow"
"Query our database for users affected by this bug"
"Post a summary of findings to #incidents in Slack"
```

MCP servers provide three capability types:
- **Tools** — functions Claude can invoke (with permission)
- **Resources** — data Claude can read (API responses, DB results)
- **Prompts** — pre-written templates for common tasks

### Setup Methods

#### 1. CLI Wizard (`claude mcp add`)

```bash
# HTTP server
claude mcp add --transport http github https://api.githubcopilot.com/mcp/ \
  --header "Authorization: Bearer ${GITHUB_TOKEN}"

# Stdio (local) server
claude mcp add --transport stdio postgres \
  --env DATABASE_URL="postgresql://user:pass@host/db" \
  -- npx -y @modelcontextprotocol/server-postgres

# SSE server (deprecated — prefer HTTP)
claude mcp add --transport sse asana https://mcp.asana.com/sse

# Windows: wrap npx with cmd /c
claude mcp add --transport stdio myserver -- cmd /c npx -y @some/package
```

**Flag ordering matters:** all options (`--transport`, `--env`, `--scope`, `--header`) must come BEFORE the server name. `--` separates the command.

#### 2. Direct Config File (`.mcp.json` at project root)

```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/",
      "headers": {
        "Authorization": "Bearer ${GITHUB_TOKEN}"
      }
    },
    "postgres": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    }
  }
}
```

#### 3. CLI Flags (one-off or CI/CD)

```bash
claude --mcp-config ./mcp.json                    # Load from file
claude --strict-mcp-config --mcp-config ./mcp.json # ONLY these servers
```

#### 4. Import from Claude Desktop

```bash
claude mcp add-from-claude-desktop     # Interactive import (macOS/WSL)
```

### `claude mcp` Subcommands

| Command | Purpose |
|---------|---------|
| `claude mcp add` | Add server (interactive wizard) |
| `claude mcp add-json <name> '<json>'` | Add server from JSON config |
| `claude mcp add-from-claude-desktop` | Import from Claude Desktop |
| `claude mcp list` | List all configured servers + status |
| `claude mcp get <name>` | Show server details and available tools |
| `claude mcp remove <name>` | Remove a server |
| `claude mcp reset-project-choices` | Re-approve previously denied project servers |
| `claude mcp serve` | Run Claude Code itself as an MCP server |

### Configuration Scopes

| Scope | File | Shared? | Use Case |
|-------|------|---------|----------|
| **Local** | `~/.claude.json` (project-specific) | No | Personal dev servers, sensitive credentials |
| **Project** | `.mcp.json` at project root | Yes (git) | Team-required integrations |
| **User** | `~/.claude.json` (global) | No | Personal tools across all projects |
| **Managed** | System directory (admin only) | Enforced | Enterprise control |

Set scope with `--scope`: `claude mcp add --scope project --transport http ...`

**Precedence:** Local > Project > User > Managed

**Managed config locations:**
- macOS: `/Library/Application Support/ClaudeCode/managed-mcp.json`
- Linux: `/etc/claude-code/managed-mcp.json`
- Windows: `C:\Program Files\ClaudeCode\managed-mcp.json`

### Transport Types

#### HTTP (Recommended for Remote)

```json
{
  "type": "http",
  "url": "https://api.example.com/mcp",
  "headers": {
    "Authorization": "Bearer ${API_TOKEN}"
  }
}
```

#### Stdio (Recommended for Local)

```json
{
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-postgres"],
  "env": {
    "DATABASE_URL": "${DATABASE_URL}"
  }
}
```

#### With OAuth

```json
{
  "type": "http",
  "url": "https://mcp.example.com/mcp",
  "oauth": {
    "clientId": "your-client-id",
    "callbackPort": 8080
  }
}
```

Client secret stored in system keychain (not in JSON). Set via `--client-secret` flag or `MCP_CLIENT_SECRET` env var.

### Environment Variables & Secrets

**Variable expansion** works in `url`, `args`, `env`, and `headers`:

```json
"url": "${API_BASE_URL:-https://api.example.com}/mcp"
"Authorization": "Bearer ${API_TOKEN}"
```

Syntax: `${VAR}` (required) or `${VAR:-default}` (with fallback)

**Servers do NOT inherit your shell environment.** You must explicitly pass variables:

```bash
# WRONG — server won't see MY_KEY
export MY_KEY="secret"
claude

# RIGHT — explicitly pass it
claude mcp add --transport stdio myserver --env MY_KEY="${MY_KEY}" -- npx server
```

**Best practices:**
- Never commit API keys — use `${VAR}` expansion
- Store OAuth secrets in system keychain (automatic via `--client-secret`)
- Use local scope for sensitive credentials, project scope for shared non-secret config

### How MCP Tools Appear to Claude

Tools follow the naming convention: `mcp__<server>__<tool>`

Example: GitHub server's `list_issues` tool → `mcp__github__list_issues`

**Viewing available tools:**
- In session: `/mcp` command shows connected servers, tools, and auth status
- CLI: `claude mcp get <name>` shows server details

**Permission for MCP tools:**

```json
{
  "allowedTools": [
    "mcp__github__*",
    "mcp__postgres__query"
  ],
  "disallowedTools": [
    "mcp__postgres__drop_table"
  ]
}
```

### Using MCP in Subagents

```markdown
---
description: Database query specialist
model: sonnet
mcpServers:
  - postgres
  - redis
permissions:
  allow:
    - mcp__postgres__query
  deny:
    - mcp__postgres__drop_table
---
```

Or inline server definitions in agents:

```markdown
---
mcpServers:
  custom-api:
    type: http
    url: https://internal.company.com/mcp
    headers:
      Authorization: Bearer ${TOKEN}
---
```

### Common MCP Servers

| Server | Transport | Setup |
|--------|-----------|-------|
| **GitHub** | HTTP | `https://api.githubcopilot.com/mcp/` |
| **Sentry** | HTTP | `https://mcp.sentry.dev/mcp` |
| **Notion** | HTTP | `https://mcp.notion.com/mcp` |
| **Stripe** | HTTP | `https://mcp.stripe.com` |
| **PostgreSQL** | Stdio | `npx -y @modelcontextprotocol/server-postgres` |
| **Filesystem** | Stdio | `npx -y @modelcontextprotocol/server-filesystem <path>` |
| **Git** | Stdio | `npx -y @modelcontextprotocol/server-git <repo>` |
| **Memory** | Stdio | `npx -y @modelcontextprotocol/server-memory` |
| **Playwright** | Stdio | `npx -y @playwright/mcp` |
| **Fetch** | Stdio | `npx -y @modelcontextprotocol/server-fetch` |

### Building Custom MCP Servers

**Python (fastest start):**

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("my-server")

@mcp.tool()
async def search_docs(query: str) -> str:
    """Search internal documentation."""
    results = await search_knowledge_base(query)
    return format_results(results)

if __name__ == "__main__":
    mcp.run(transport="stdio")
```

**TypeScript:**

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({ name: "my-server", version: "1.0.0" });

server.registerTool("search_docs", {
  description: "Search internal documentation",
  inputSchema: { query: z.string() }
}, async ({ query }) => ({
  content: [{ type: "text", text: await searchDocs(query) }]
}));

const transport = new StdioServerTransport();
await server.connect(transport);
```

**Critical rule for stdio servers:** Never write to stdout (it carries the JSON-RPC protocol). Use `stderr` or a logging library for debug output.

### Debugging MCP

```bash
claude --mcp-debug                    # Protocol-level logging
claude --verbose --mcp-debug          # Full diagnostics
MCP_TIMEOUT=120000 claude             # Increase startup timeout to 2 min
MAX_MCP_OUTPUT_TOKENS=50000 claude    # Increase max tool output
claude doctor                         # Health check includes MCP status
```

**In session:**
- `/mcp` — view server status, tools, and auth
- `claude mcp get <name>` — check individual server

### `--strict-mcp-config`

Only uses servers from `--mcp-config`, ignoring all other configurations:

```bash
claude --strict-mcp-config --mcp-config ./client-acme-tools.json
```

Use for: isolated client work, security-sensitive contexts, CI/CD pipelines.

### Gotchas

| Gotcha | Detail |
|--------|--------|
| Windows npx | Wrap with `cmd /c npx ...` or it fails with "Connection closed" |
| Env vars not inherited | Servers don't see your shell env — pass explicitly via `--env` or config |
| Stdout pollution | Stdio servers must never `print()` / `console.log()` — use stderr |
| Project servers need approval | First use prompts for approval; reset with `claude mcp reset-project-choices` |
| Absolute paths required | Stdio servers run with undefined CWD — always use absolute paths |
| Tool search needs Sonnet+ | Tool search fails with Haiku; use `ENABLE_TOOL_SEARCH=false` |
| OAuth token expiry | Re-authenticate via `/mcp` → Clear authentication → Authenticate |
| Secrets in `.mcp.json` | Use `${VAR}` expansion, never hardcode — file may be in git |

---

## Permissions & Auto-Approval

How to control when Claude asks for permission — from broad permanent rules to narrow temporary approvals.

### Method 1: Permanent Rules (`allowedTools`)

Add patterns to settings.json. These auto-approve across all sessions without prompting.

```json
// .claude/settings.json (project-shared) or ~/.claude/settings.json (personal)
{
  "allowedTools": [
    "Read",
    "Write(src/generated/*)",
    "Edit(src/generated/*)",
    "Bash(npm test *)",
    "Bash(npm run build *)",
    "Bash(git log *)",
    "Bash(git diff *)",
    "Bash(git status *)"
  ]
}
```

**Path patterns:** `Write(src/generated/*)` only auto-approves writes inside `src/generated/`. Writes to any other path still prompt.

**Also at launch:**
```bash
claude --allowedTools "Bash(git *)" "Read" "Write(src/generated/*)"
```

### Method 2: Session-Scoped ("Always Allow")

When Claude asks for permission, choose **"Always allow"** for that tool. Lasts until you exit the session — gone on next launch.

This is the most common way to temporarily approve things. Claude asks once, you allow, and it doesn't ask again for that tool pattern during the session.

### Method 3: Mode Toggle (`Shift+Tab`)

Cycle permission modes mid-session:

```
default → acceptEdits → plan → default
```

**`acceptEdits`** auto-approves all file writes (Write, Edit) but still asks for Bash commands. This is the closest thing to "approve until a task is finished" — flip to `acceptEdits` while Claude is working on a known task, flip back to `default` when it's done.

**`bypassPermissions`** approves everything. Only use in containers/VMs where there's nothing to damage.

### Method 4: Command-Level Approval

In custom slash command frontmatter, grant specific tools for that command's execution only:

```markdown
---
description: Generate API types from OpenAPI schema
allowed-tools: Write, Bash(npm run generate *)
---

Read the OpenAPI spec at $ARGUMENTS and generate TypeScript types.
Write them to src/generated/api-types.ts.
```

When you run `/generate-types`, Write and the generate script are auto-approved. Once the command finishes, permissions revert to normal.

### Method 5: Hook-Based Auto-Approval (Most Flexible)

A PreToolUse hook can auto-approve based on any logic — file path, command content, working directory, or anything you can script.

**Settings:**
```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": ".claude/hooks/auto-approve-paths.sh"
      }]
    }]
  }
}
```

**auto-approve-paths.sh:**
```bash
#!/bin/bash
INPUT=$(cat)
FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

# Auto-approve writes to generated files or test files
if [[ "$FILE" == */generated/* || "$FILE" == *_test.* || "$FILE" == *.test.* ]]; then
  echo '{"hookSpecificOutput":{"hookEventName":"PreToolUse","permissionDecision":"allow","permissionDecisionReason":"Auto-approved: generated/test file"}}'
else
  exit 0  # fall through to normal permission prompt
fi
```

You can also use a `prompt` or `agent` hook type to have an LLM decide whether to auto-approve:

```json
{
  "type": "prompt",
  "prompt": "Should this file write be auto-approved? Only approve writes to test files or generated code. Input: $ARGUMENTS"
}
```

### Method 6: `disallowedTools` (Permanent Deny)

The inverse — permanently block tools regardless of other settings:

```json
{
  "disallowedTools": [
    "Bash(rm -rf *)",
    "Bash(git push --force *)",
    "Read(./.env)"
  ]
}
```

These are removed from Claude's context entirely — it can't even attempt them. Works even in `bypassPermissions` mode.

### Comparison

| Method | Scope | Persists? | Granularity |
|--------|-------|-----------|-------------|
| `allowedTools` in settings | All sessions | Yes (permanent) | Tool + path/arg pattern |
| "Always allow" prompt | Current session | No (session only) | Single tool pattern |
| `Shift+Tab` mode toggle | Until toggled back | No (session only) | All edits or all actions |
| Command `allowed-tools` | Single command run | No (command only) | Named tools |
| PreToolUse hook | All sessions | Yes (permanent) | Any custom logic |
| `disallowedTools` | All sessions | Yes (permanent) | Tool + path/arg pattern |

### What Doesn't Exist (Yet)

- **Time-boxed approval** — "approve for the next 10 minutes"
- **Task-scoped approval** — "approve until this task completes, then revert"
- **Cumulative approval** — "approve up to N writes, then ask again"

The best workaround for task-scoped approval is `Shift+Tab` to `acceptEdits` while Claude works, then `Shift+Tab` back to `default` when it finishes.

---

## Subagents

Subagents are specialized AI assistants that run in **isolated context windows** with their own system prompts, tool restrictions, and permissions. They let Claude delegate work without polluting your main conversation with verbose output (test results, search results, log analysis).

### Why Use Subagents?

Your main conversation has a 200K token context window. Every file read, command output, and Claude response eats into it. Subagents run in their own window — they can explore extensively, then return a concise summary. The verbose intermediate work stays out of your context.

**Use subagents when:**
- Task produces verbose output you don't need (test suites, log analysis, data processing)
- Work is self-contained and can return a summary
- You want to enforce specific tool restrictions
- Running parallel investigations

**Use the main conversation when:**
- Task needs back-and-forth iteration
- Multiple phases share significant context
- Making a quick, targeted change

### Built-in Subagent Types

| Type | Model | Tools | Best For |
|------|-------|-------|----------|
| `Explore` | Haiku (fast) | Read, Grep, Glob, Bash (read-only) | File search, codebase analysis, pattern finding |
| `Plan` | Inherits | Read, Grep, Glob, Bash (read-only) | Architecture analysis, implementation planning |
| `Bash` | Inherits | Bash | Terminal operations in isolated context |
| `general-purpose` | Inherits | All tools | Complex multi-step tasks requiring read + write |

The Explore agent uses Haiku for speed — it's searching, not reasoning. For tasks that need deeper thinking, use `general-purpose` or a custom agent with Sonnet/Opus.

### Creating Custom Subagents

#### Via `.claude/agents/` (Recommended)

`.claude/agents/code-reviewer.md`:
```markdown
---
name: code-reviewer
description: Expert code reviewer. Use proactively after writing or modifying code.
tools: Read, Grep, Glob, Bash
model: sonnet
maxTurns: 10
---

You are a senior code reviewer. When invoked:
1. Run `git diff` to see recent changes
2. Focus on modified files
3. Review immediately

Check for:
- Security vulnerabilities (injection, exposed secrets)
- Performance issues (N+1 queries, unnecessary allocations)
- Code clarity (naming, structure, duplication)
- Error handling (missing catches, swallowed errors)
- Test coverage gaps

Report findings by priority: Critical > Warning > Suggestion.
Include specific fix examples.
```

#### Via `--agents` CLI Flag (Session-Only)

```bash
claude --agents '{
  "debugger": {
    "description": "Debugging specialist for errors and test failures.",
    "prompt": "You are an expert debugger. Use root cause analysis.",
    "tools": ["Read", "Edit", "Bash", "Grep", "Glob"],
    "model": "sonnet"
  }
}'
```

Not persisted — useful for one-off sessions or CI/CD.

### All Frontmatter Fields

| Field | Default | Description |
|-------|---------|-------------|
| `name` | Filename | Identifier (lowercase, hyphens) |
| `description` | — | **When Claude should delegate to this agent.** Include "Use proactively" for auto-delegation |
| `tools` | Inherit all | Comma-separated tool allowlist: `Read, Grep, Glob, Bash` |
| `disallowedTools` | None | Tools to deny (removed from inherited set) |
| `model` | `inherit` | `sonnet`, `opus`, `haiku`, or `inherit` |
| `permissionMode` | `default` | `default`, `acceptEdits`, `plan`, `dontAsk`, `bypassPermissions` |
| `maxTurns` | None | Max agentic turns before stopping |
| `skills` | None | Skills to preload into context (full content injected) |
| `mcpServers` | None | MCP servers available to this agent |
| `hooks` | None | Agent-scoped hooks (same format as global hooks) |
| `memory` | None | Persistent memory: `user`, `project`, or `local` |

### How Claude Decides to Use a Subagent

Claude reads each subagent's `description` and matches it to the current task. Write descriptions that clearly state **when** to delegate:

```yaml
# Good — Claude knows exactly when to use this
description: Expert code reviewer. Use proactively after writing or modifying code.

# Bad — too vague
description: Reviews code.
```

### Storage Locations (Priority Order)

| Location | Scope | Priority |
|----------|-------|----------|
| `--agents` CLI flag | Current session only | Highest |
| `.claude/agents/` | Project | High |
| `~/.claude/agents/` | User (all projects) | Medium |
| Plugin `agents/` | Where plugin enabled | Lowest |

When multiple agents share the same name, highest-priority wins.

### Foreground vs. Background Execution

**Foreground (default):** Blocks main conversation. Permission prompts passed through to you. Best for quick tasks.

**Background:** Runs concurrently while you keep working.

```
Run this in the background: Use the test-runner to run the full test suite
```

Or press `Ctrl+B` while a foreground task is running to push it to background.

**Background limitations:**
- Permissions must be pre-approved before launch (auto-denies anything not pre-approved)
- MCP tools NOT available in background
- Can't ask clarifying questions (tool call fails, agent continues)
- Check progress with `Ctrl+T` (task list) or ask Claude

### Resuming Subagents

Each invocation gets an agent ID. Resume to continue with full history preserved:

```
Continue the code review from earlier — also check the authorization module
```

Claude resumes the same subagent with its full conversation history intact. Subagent transcripts persist independently from the main conversation and survive session restarts.

### Parallel Execution

Claude can run multiple subagents concurrently (up to ~10):

```
Research the auth, database, and API modules in parallel using separate Explore agents
```

Each explores independently, results return to main conversation. Best when investigations don't depend on each other.

**Warning:** Many subagents returning detailed results can consume main context quickly. Keep return summaries concise.

### MCP Servers in Subagents

Reference existing servers or define inline:

```yaml
---
name: db-analyst
description: Query databases for analysis and reports
tools: Bash, Read
mcpServers:
  - postgres
---
```

Or inline:

```yaml
mcpServers:
  postgres:
    command: "npx"
    args: ["-y", "@modelcontextprotocol/server-postgres"]
    env:
      DATABASE_URL: "${DATABASE_URL}"
```

**Background subagents cannot use MCP tools** — foreground only.

### Persistent Memory

Enable cross-session learning so the agent builds institutional knowledge:

```yaml
---
name: code-reviewer
memory: user
---

Update your agent memory as you discover code patterns, library locations,
and architectural decisions. This knowledge persists across sessions.
```

Memory stored at:
- `user`: `~/.claude/agent-memory/<agent-name>/`
- `project`: `.claude/agent-memory/<agent-name>/` (version controlled)
- `local`: `.claude/agent-memory-local/<agent-name>/` (gitignored)

### Hooks in Subagents

Scope hooks to a specific agent's lifecycle:

```yaml
---
name: db-reader
tools: Bash
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-readonly-query.sh"
---
```

The hook validates every Bash command — exit code 2 blocks destructive queries.

From the main session, respond to subagent events:

```json
{
  "hooks": {
    "SubagentStop": [{
      "hooks": [{
        "type": "command",
        "command": "./scripts/post-review-cleanup.sh"
      }]
    }]
  }
}
```

### Practical Examples

#### Read-Only Security Auditor

```yaml
---
name: security-auditor
description: Security audit specialist. Use when reviewing code for vulnerabilities.
tools: Read, Grep, Glob
model: opus
---

Audit code for OWASP Top 10 vulnerabilities:
- Injection (SQL, command, XSS)
- Broken authentication
- Sensitive data exposure
- Security misconfiguration
- Exposed secrets or credentials

Rate each finding: Critical / High / Medium / Low.
Include file:line references and fix suggestions.
```

#### Test Runner with Reporting

```yaml
---
name: test-runner
description: Run tests and report results. Use after code changes.
tools: Bash, Read, Grep
model: haiku
---

Run the test suite and report:
1. Execute tests
2. Parse output for failures
3. For each failure: identify the test, the error, and likely cause
4. Summarize: total passed, failed, skipped
5. If failures exist, suggest which code changes likely caused them
```

#### Database Query Agent (Hook-Restricted)

```yaml
---
name: db-reader
description: Execute read-only database queries for analysis and reporting.
tools: Bash
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: ".claude/hooks/validate-readonly.sh"
---

You have read-only database access. Write SELECT queries to answer data questions.
You cannot INSERT, UPDATE, DELETE, or modify schema.
```

### Subagents vs. Agent Teams

| | Subagents | Agent Teams |
|--|-----------|-------------|
| Architecture | Within single session | Separate independent sessions |
| Communication | Report to main agent only | Peer-to-peer messaging between teammates |
| Context | Own window, no cross-talk | Each teammate has full independent context |
| Coordination | Main agent manages all | Shared task list, self-coordination |
| Cost | Lower (summarized results) | Higher (each teammate is full Claude instance) |
| Nesting | Cannot spawn sub-subagents | Cannot nest teams |
| Best for | Quick focused tasks | Complex work requiring discussion between specialists |
| Status | Production | Experimental |

### Gotchas

| Gotcha | Detail |
|--------|--------|
| No nesting | Subagents can't spawn other subagents |
| Context amnesia | Subagents don't see parent conversation — pass detailed context in the prompt |
| Name inference | Claude infers behavior from agent name; custom instructions may be overridden. Use non-obvious names if this happens |
| File conflicts | Parallel subagents editing the same file overwrite each other — assign different files |
| Background MCP | MCP tools not available in background subagents |
| Permission silence | Misconfigured permissions silently block operations — test explicitly |
| Resume after restart | `/resume` doesn't restore background subagents — spawn new ones |

### Managing Subagents

Use `/agents` in session to:
- View all subagents (built-in, user, project, plugin)
- Create new agents with guided setup
- Edit or delete existing configurations

---

## Tokenization & How LLMs Read Text

LLMs don't read text the way you do. They break everything into **tokens** first. Understanding tokens explains why context windows have limits, why costs are measured the way they are, and why some prompts are more expensive than others.

### What Is a Token?

A token is a chunk of text — usually a common word, part of a word, or a punctuation mark. The model converts your text into a sequence of tokens before processing it.

```
"Hello, how are you?"  →  ["Hello", ",", " how", " are", " you", "?"]
                           6 tokens
```

Common words are usually one token. Less common words get split:

```
"unhappiness"  →  ["un", "happiness"]           2 tokens
"ChatGPT"      →  ["Chat", "G", "PT"]           3 tokens
"authentication" → ["authentication"]            1 token (common in code)
"tsconfig"     →  ["ts", "config"]               2 tokens
```

### Rules of Thumb

| Text Type | Rough Ratio |
|-----------|-------------|
| English prose | ~1 token per 4 characters, or ~3/4 of a word |
| Code | Varies widely — keywords are 1 token, variable names may be several |
| Whitespace/indentation | Costs tokens too (spaces, tabs, newlines) |
| Numbers | Each digit or small group is a token (`123456` might be 2-3 tokens) |
| Non-English text | Typically more tokens per word than English |

**Quick estimates:**
- 1,000 tokens ≈ 750 words of English
- A typical source code file (200 lines) ≈ 1,000-3,000 tokens
- This entire deep dives document ≈ ~8,000 tokens

### Why Tokens Matter in Claude Code

**Context window** — Claude's memory for a conversation is measured in tokens, not characters or words. Claude currently has a 200K token context window. Everything in a conversation shares this space:

```
Context Window (200K tokens)
├── System prompt + CLAUDE.md files     (~2-5K tokens)
├── Tool definitions                     (~3-8K tokens)
├── Conversation history                 (grows with each turn)
│   ├── Your messages
│   ├── Claude's responses
│   └── Tool inputs and outputs          (file contents, command output)
└── Remaining space for Claude to think
```

When the window fills up, Claude Code **compacts** the conversation — summarizing older messages to free space. That's why `/context` exists (to see usage) and `/compact` exists (to compress early).

**Cost** — API usage is billed per token. Both input tokens (what you send) and output tokens (what Claude generates) count. This is why:
- Reading a large file into the conversation costs input tokens
- Long Claude responses cost output tokens
- Subagents each use their own tokens (agent teams multiply this)
- `/compact` reduces future input costs by shrinking history

**Prompt design** — shorter, clearer prompts use fewer tokens and leave more room for Claude to work. A 50-line file pasted into chat costs far fewer tokens than a 5,000-line file.

### Context Window Sizes

| Model | Standard | Extended |
|-------|----------|----------|
| Claude Sonnet 4.5 | 200K tokens | 1M tokens (`sonnet[1m]`) |
| Claude Opus 4.6 | 200K tokens | — |
| Claude Haiku 4.5 | 200K tokens | — |

The 1M extended context is available for Sonnet via the `[1m]` suffix, but larger contexts cost proportionally more and can be slower.

### Tokens vs. Characters vs. Words

```
"The quick brown fox"

Characters:  19  (including spaces)
Words:        4
Tokens:       4  ("The", " quick", " brown", " fox")
```

```
"src/components/AuthProvider.tsx"

Characters:  32
Words:        1  (it's a file path)
Tokens:      ~7  ("src", "/", "components", "/", "Auth", "Provider", ".tsx")
```

Tokens are the only unit the model actually works with. Characters and words are just human-friendly approximations.

### What Eats Tokens Fast

| Source | Impact | Mitigation |
|--------|--------|------------|
| Large file reads | A 1,000-line file can be 3-5K tokens | Read specific sections with `offset`/`limit` |
| Verbose command output | Build logs, test output | Pipe through `tail`, `head`, or filters |
| Long conversation history | Accumulates every turn | `/compact` or `/clear` + restart |
| Many tool calls | Each call's input + output stays in context | Use subagents (separate context window) |
| System prompts + CLAUDE.md | Loaded every turn | Keep CLAUDE.md lean |

### The Compaction Cycle

```
1. You chat with Claude, reading files and running commands
2. Context fills up (check with /context)
3. At ~75-92% full, auto-compaction triggers
4. Older messages get summarized — detail is lost
5. Cycle repeats
```

To stay ahead of this:
- Use `/context` to check before it's critical
- `/compact` manually at ~65-70% (before quality degrades)
- `/clear` for a clean slate if compaction has already lost important context
- Put critical instructions in CLAUDE.md (reloaded every turn, never compacted away)

### "What if I remove all the spaces from my prompt to save tokens?"

Don't. Tokenizers are trained on normal text with spaces. Removing spaces actually **increases** token count because the tokenizer can no longer recognize common words and falls back to splitting everything into small fragments.

```
"Fix the login bug"          →  5 tokens  (normal)
"Fixtheloginbug"             →  5+ tokens (no savings — may even cost more)
"pls fix auth err in login"  →  7 tokens  (abbreviations don't help much either)
```

The tokenizer already handles spaces efficiently — a space before a word is usually part of that word's token (`" the"` is one token, not two). Your best token-saving strategy isn't compressing your writing — it's being **specific and concise**. Say what you mean in fewer sentences, not fewer characters.

### "What about misspellings?"

---

*Last updated: February 2026*
