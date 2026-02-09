# Starting a Project with Claude Code

> A practical guide to setting up a new project (or onboarding Claude to an existing one) so you get the most out of every session from day one.

---

## Table of Contents

- [Before You Start](#before-you-start)
- [First Steps: Start Your Project](#first-steps-start-your-project)
- [Step 1: Launch from the Right Place](#step-1-launch-from-the-right-place)
- [Step 2: Initialize with /init](#step-2-initialize-with-init)
- [Step 3: Tune Your CLAUDE.md](#step-3-tune-your-claudemd)
- [Step 4: Set Up Permissions](#step-4-set-up-permissions)
- [Step 5: Create Your First Commands](#step-5-create-your-first-commands)
- [Step 6: Add Hooks (Optional)](#step-6-add-hooks-optional)
- [Step 7: Connect MCP Servers (Optional)](#step-7-connect-mcp-servers-optional)
- [Step 8: Start Building](#step-8-start-building)
- [Step 9: Create Subagents (As Needed)](#step-9-create-subagents-as-needed)
- [Tips That Save Hours](#tips-that-save-hours)
- [Common Mistakes to Avoid](#common-mistakes-to-avoid)
- [Project Templates](#project-templates)

---

## Before You Start

### Install Claude Code

**Requirements:** macOS 13+, Windows 10 1809+, or Ubuntu 20.04+. 4 GB RAM minimum. Internet connection required.

**Which terminal to use:**

| Platform | App | How to Open |
|----------|-----|-------------|
| **macOS** | Terminal | Applications → Utilities → Terminal (or Spotlight: "Terminal") |
| **Windows** | PowerShell | Start menu → search "PowerShell" → open **Windows PowerShell** |
| **Linux** | Terminal | Ctrl+Alt+T (most distros) |

**Install** (pick your platform):

macOS — uses Homebrew, the standard macOS package manager. If you don't have it yet, install it first:

```bash
# Install Homebrew (skip if you already have it — check with: brew --version)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Then install Claude Code:

```bash
brew install --cask claude-code
```

Windows — run this in PowerShell:

```powershell
irm https://claude.ai/install.ps1 | iex
```

Linux / WSL:

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

> **Auto-updates:** The native installers (Windows PowerShell, Linux curl) auto-update in the background. Homebrew does not — run `brew upgrade claude-code` periodically to stay current.

**Authenticate** — on first launch, Claude Code prompts you to log in:

```bash
claude    # opens login flow on first run
```

You need one of:
- **Claude Pro or Max subscription** (recommended) — log in with your claude.ai account at claude.ai. Pro is $20/mo, Max is $100-200/mo.

> **For organizations:** Claude Console (console.anthropic.com) offers pay-per-token API billing for developers. Claude Team and Enterprise plans provide centralized billing and team management. These are typically set up by your company — if you're on one, your admin will provide login details.

**Verify** it's working:

```bash
claude --version   # check version
```
```bash
claude doctor      # diagnose any issues
```

> **Note:** Claude Code auto-updates with the native installer. If you installed via Homebrew or WinGet, update manually (`brew upgrade claude-code` / `winget upgrade Anthropic.ClaudeCode`).

### Install Git

```bash
# Check if git is already installed
git --version
```

If not installed:

| Platform | Command | Notes |
|----------|---------|-------|
| **macOS** | `brew install git` | Uses Homebrew (installed in previous step). macOS also ships with git via Xcode Command Line Tools — run `xcode-select --install` as an alternative. |
| **Windows** | `winget install Git.Git` | Run in PowerShell. Installs Git for Windows, which includes Git Bash. Alternatively, download from https://git-scm.com. |
| **Linux** | `sudo apt install git` | Ubuntu/Debian. For Fedora: `sudo dnf install git`. |

After installing, configure your identity (required for commits):

```bash
git config --global user.name "Your Name"
```
```bash
git config --global user.email "your.email@example.com"
```

### Install Node.js (Optional)

Not required for Claude Code itself, but needed if you plan to use MCP servers (most launch via `npx`), or if your project uses npm/yarn.

| Platform | Command | Notes |
|----------|---------|-------|
| **macOS** | `brew install node` | Uses Homebrew. |
| **Windows** | `winget install OpenJS.NodeJS.LTS` | Run in PowerShell. Alternatively, download from https://nodejs.org. |
| **Linux** | `sudo apt install nodejs npm` | Ubuntu/Debian. For other distros, see https://nodejs.org. |

```bash
node --version    # should be 18+
```
```bash
npm --version
```

### Choose Your IDE

Most IDEs have a built-in terminal. Once Claude Code is installed, just open your IDE's terminal and type `claude` — that's it. No plugin required.

If you don't have an IDE yet, **VS Code** is a great free option: https://code.visualstudio.com

**Optional: IDE plugins** — Both VS Code and JetBrains (Rider, IntelliJ, PyCharm, etc.) offer Claude Code plugins that add inline diffs, visual plan review, and code selection sharing. Search **"Claude Code"** in your IDE's plugin/extension marketplace to install.

**Terminal tips for beginners:**

| Tip | What It Does |
|-----|-------------|
| `↑` / `↓` arrow keys | Scroll through previous commands — no retyping |
| `Tab` | Auto-complete file names and folder paths |
| `cd foldername` | Move into a folder |
| `cd ..` | Go back up one folder |
| `ls` (Mac/Linux) or `dir` (Windows) | List files in the current folder |
| `pwd` (Mac/Linux) or `cd` alone (Windows) | Show where you are right now |
| `Ctrl+C` | Cancel a running command |
| `clear` | Clear the screen (doesn't delete anything) |
| Right-click (Windows) or `Cmd+V` (Mac) | Paste into the terminal |

Once inside a Claude Code session, look at the **status bar at the bottom** of the terminal. It shows your current token usage and cost for the session — helpful for keeping track of how much context you're using.

> **Don't worry about memorizing these.** Once Claude Code is running, you can ask Claude to do anything — navigate folders, run commands, find files. The terminal is just how you launch it.

### Pick Your Model

For project setup and exploration, `opus` or `opusplan` gives the best results. Switch to `sonnet` for daily coding once you're set up.

```bash
claude --model opus    # Setup sessions
```
```bash
claude --model sonnet  # Daily work
```

---

## First Steps: Start Your Project

### Plan Your Project with Claude (Plan Mode)

Before writing any code, use **plan mode** — a special mode where Claude explores, researches, and designs your project without changing any files. It thinks first, then presents a plan for your approval.

Launch Claude Code in plan mode:

```bash
claude --permission-mode plan
```

Then tell Claude what you want to build:

```
> I want to build a simple interactive portfolio website. Help me plan it out.
```

In plan mode, Claude will:
- **Research options** — compare frameworks, hosting platforms, and tools
- **Recommend a tech stack** — based on your needs (beginner-friendly, easy to deploy, etc.)
- **Design the structure** — folder layout, pages, components
- **Create a step-by-step plan** — what to build first, second, third

Claude will present the plan and ask for your approval before doing anything. You can ask questions, request changes, or ask it to explore alternatives:

```
> What if I want animations? Would that change the tech stack?
```

```
> Keep it simple — I want something I can deploy for free on GitHub Pages
```

Once you approve the plan, Claude saves it and switches to building mode. You can also ask it to save the plan for reference:

```
> Write this plan to a PLAN.md file so we can reference it later
```

You don't need a repository yet for this step — just ideas. Claude works as a thinking partner before it becomes a coding partner.

### Clone This Guide (Practice Run)

If you're reading this online, start by cloning this repository to your computer. It's a great way to practice using git — and you'll have all these guides available locally for reference.

```
claude
> Clone https://github.com/PLACEHOLDER/claude-basics.git into my projects folder
```

<!-- TODO: Replace PLACEHOLDER with the actual repo URL once published -->

Claude will clone the repo and confirm when it's done. You now have a local copy of this guide, the cheat sheet, and the deep dives — all searchable and updatable.

---

### Create Your Own Repository

Now let's create a repository for your actual project.

**Option A: Let Claude do it (easiest)**

You need a GitHub account (sign up at github.com/join if you don't have one). Then let Claude handle the rest:

```
> Create a new GitHub repository called "my-portfolio" and clone it into my projects folder
```

Claude will use the GitHub CLI (`gh`) to create the repo, clone it locally, and set everything up. It may prompt you to authenticate with GitHub on the first run — just follow the login flow.

**Option A2: Create on GitHub website first**

If you prefer to set things up on the website:

1. Go to github.com → click **+** (top-right) → **New repository**
2. Name it, add a description, check **Add a README file**, click **Create repository**
3. Then ask Claude to clone it:

```
> Clone my repository https://github.com/yourname/my-portfolio.git into my projects folder
```

**Option B: Create locally first (if you already have project files)**

Open your terminal in your project folder and launch Claude Code:

```
cd ~/projects/my-portfolio
claude
> Initialize a git repository here and make the first commit
```

Claude will run `git init`, stage your files, and create the initial commit.

Then to connect it to GitHub:

1. Create an **empty** repository on github.com (don't add a README)
2. Tell Claude to link and push:

```
> Connect this repository to https://github.com/yourname/my-portfolio.git and push
```

### Install the GitHub App for Claude (Optional)

```
> /install-github-app
```

Run this inside Claude Code. It's a **one-time permanent setup** — a wizard walks you through authorizing the Claude GitHub App on your account or organization. Once installed, Claude can:
- Create and review pull requests
- Read and comment on issues
- Run as a GitHub Actions agent (triggered by `@claude` mentions in PRs/issues)

You only do this once. It persists across all sessions and projects for the authorized account.

---

## Step 1: Launch from the Right Place

Always launch Claude Code from your **project's main folder** — the one that contains the `.git/` folder:

```bash
cd ~/projects/my-portfolio
```
```bash
claude
```

Not a parent directory, not a subfolder. The top level of your project. This ensures Claude finds your `.claude/` settings, CLAUDE.md files, and all your project files correctly.

> **For larger projects** with multiple components (frontend, backend, etc.), you still launch from the top level. You can add separate CLAUDE.md files inside subfolders for component-specific instructions — Claude picks them up automatically when working in those directories.

---

## Step 2: Initialize with /init

Now that you're in a Claude session inside your project, run the `/init` command. Claude will scan your project and generate a CLAUDE.md file — a short reference document that helps Claude understand your project on every future session.

```
> /init
```

Claude will look at your files and create a CLAUDE.md containing:
- What your project is and what tech it uses
- How to build and run it
- Code style patterns it detected

For our portfolio, it might generate something like:

```markdown
# my-portfolio

Static portfolio site built with HTML, CSS, and JavaScript.

## Dev Server
- `npx live-server`

## Structure
- `index.html` — main page
- `css/` — stylesheets
- `js/` — scripts
- `assets/` — images and media
```

This is your starting point. You'll refine it in the next step.

---

## Step 3: Tune Your CLAUDE.md

`/init` generates a reasonable draft, but it may be too verbose or missing details you care about. The golden rule: **every line in CLAUDE.md costs tokens on every turn**, because Claude reads it at the start of every interaction. Keep it lean.

**Target: ~20-30 lines.** Include only what Claude needs on every interaction. Ask Claude to help:

```
> Read the CLAUDE.md you generated. Trim it to under 30 lines — keep only the project description, build commands, code style rules, and folder structure. Remove anything I can just ask you about later.
```

**What belongs in CLAUDE.md:**
- What the project is (one line)
- How to build/run/test it
- Code style rules (naming, formatting, conventions)
- Folder structure overview

**What to leave out:**
- Detailed documentation (Claude can read the files when needed)
- Step-by-step workflows (put these in commands — see Step 5)
- "You MUST always..." instructions (Claude doesn't reliably follow these)

---

## Step 4: Set Up Permissions

Every time Claude wants to run a command or edit a file, it asks your permission. This is a safety feature — but it gets repetitive for things you always approve (like reading files or running tests).

You can pre-approve safe operations so Claude doesn't ask every time. Ask Claude to do it:

```
> Create a .claude/settings.json file that auto-approves reading files, git log, git diff, git status, and running npm test
```

Claude will create something like:

```json
{
  "allowedTools": [
    "Read",
    "Bash(git log *)",
    "Bash(git diff *)",
    "Bash(git status *)",
    "Bash(npm test *)"
  ]
}
```

Claude will still ask permission for anything not on this list — like writing files, deleting things, or running unfamiliar commands. You stay in control.

> **Tip:** You can also approve things on the fly during a session. When Claude asks permission, you'll see options to allow it once, always for this session, or always for this project.

---

## Step 5: Create Your First Commands

Commands are reusable prompts you can run with a slash — like `/status` or `/catchup`. They save you from typing the same instructions every session.

Two commands are worth creating right away. Ask Claude:

```
> Create a /status command in .claude/commands/status.md. It should read CLAUDE.md, run git log and git status, then summarize what the project is, what changed recently, and what to work on next.
```

```
> Create a /catchup command in .claude/commands/catchup.md. It should re-read CLAUDE.md and recent git history, then summarize where we left off. I'll use this after clearing the conversation.
```

Now you can start any session with:

```
> /status
```

And Claude gives you a full briefing — what the project is, recent commits, current branch, uncommitted changes, and what to do next.

Other commands worth creating as you go:

| Command | Purpose | When |
|---------|---------|------|
| `/review` | Code review with your standards | After writing code |
| `/test` | Run tests and report results | After changes |
| `/commit` | Stage, commit with a good message | When ready to commit |

Don't try to create every command upfront — add them as you notice patterns in your workflow.

---

## Step 6: Add Hooks (Optional)

Hooks run automatically when Claude does things — like formatting your code every time Claude edits a file, or reminding you of project rules after the conversation gets compressed.

**You don't need hooks to get started.** They're a power-user feature. Come back to this when you find yourself doing something repetitive that you want automated.

A simple example — auto-format with Prettier after every edit:

```
> Add a PostToolUse hook to .claude/settings.json that runs Prettier on any file Claude writes or edits
```

See the [deep dives](CLAUDE_CODE_DEEP_DIVES.md#hooks-system) for more examples and the full list of hook events.

---

## Step 7: Connect MCP Servers (Optional)

MCP servers connect Claude to external tools — databases, APIs, services. For a portfolio site, you probably don't need any yet.

**Come back to this when you need to:**
- Query a database during development
- Work with GitHub issues and PRs from inside Claude
- Connect to Slack, Jira, or other team tools

Adding a server is one command:

```bash
claude mcp add --transport stdio memory -- npx -y @modelcontextprotocol/server-memory
```

Don't add servers "just in case" — each one adds tokens to your context. Add them when you have a real need.

See the [deep dives](CLAUDE_CODE_DEEP_DIVES.md#mcp-servers) for setup details and common servers.

---

## Step 8: Start Building

You've set up your tools, planned your project, and configured Claude Code. Time to build. Here's a workflow for your first coding session, using the portfolio example:

### Review Your Plan

If you created a plan in "First Steps," start by pointing Claude to it:

```
> Read PLAN.md and summarize what we're building and what to do first
```

If you didn't write a formal plan, that's fine — just tell Claude what you want:

```
> I'm building a portfolio website. I want a homepage with a hero section, a projects grid, and an about page. Let's start with the homepage.
```

Either way, make sure Claude understands the big picture before you start coding.

### Scaffold the Project

Scaffolding means creating the initial structure of your project — the folders, starter files, and dependencies that everything else builds on. Instead of doing this manually, ask Claude:

```
> Set up the project based on our plan. Create the folder structure, install dependencies, and build a basic homepage.
```

Claude will create the files, install packages, and give you a working starting point. Review what it creates — approve or ask for changes.

> **Power move:** For larger plans with multiple steps, Claude can use **subagents** — background workers that handle different parts of the plan in parallel. For example, one subagent sets up the CSS framework while another creates the page templates. You don't need to manage this yourself — Claude will use subagents automatically when it makes sense, or you can ask:
>
> ```
> > Execute the plan. Use subagents to work on the homepage and the projects page in parallel.
> ```

### Build Feature by Feature

Work on one thing at a time. Be specific about what you want:

```
> Add a projects section to the homepage that displays a grid of project cards
```

```
> Add a navigation bar with links to Home, Projects, and About sections
```

```
> Make the site responsive so it looks good on mobile
```

### Review and Commit as You Go

After each feature, review what changed and commit:

```
> Show me what you changed
```

```
> Commit this with a descriptive message
```

Small, frequent commits give you checkpoints to rewind to if something goes wrong.

### Test Along the Way

```
> Open this in the browser and check if everything looks right
```

```
> Run the tests
```

```
> Fix any errors you see
```

### Keep the Conversation Focused

When you finish a feature and want to start the next one, check your context:

```
> /context
```

If it's getting heavy (above 60%), clear and reload:

```
> /clear
```
```
> /catchup
```

Then start fresh on the next feature.

### Example: A Full Session

Here's what a typical first session might look like:

```
claude
> I want to build my portfolio site. Let's start with the homepage.
  [Claude scaffolds the project]
> Add a hero section with my name and a short bio
  [Claude creates the component]
> Show me what that looks like — open it in the browser
  [Claude runs the dev server]
> The font is too small on mobile. Fix that.
  [Claude adjusts the CSS]
> Looks good. Commit this as "Add responsive hero section"
  [Claude commits]
> Now add a projects grid below the hero
  [continues...]
```

The pattern is simple: **describe → review → adjust → commit → next feature**.

---

## Step 9: Create Subagents (As Needed)

You don't need custom subagents on day one. The built-in ones (Explore, Plan, Bash, general-purpose) handle most tasks. Create custom agents when you find yourself repeating the same delegation pattern during development.

**Signs you need a custom subagent:**
- You keep saying "review this code for security issues" → make a security-reviewer agent
- You run the same test analysis every time → make a test-analyzer agent
- You have project-specific review criteria → encode them in an agent

`.claude/agents/reviewer.md`:
```markdown
---
name: reviewer
description: Code reviewer. Use proactively after writing or modifying code.
tools: Read, Grep, Glob, Bash
model: sonnet
---

Review code changes for:
1. Bugs and logic errors
2. Security issues (injection, exposed secrets)
3. Performance problems
4. Missing error handling
5. Test coverage gaps

Run `git diff` to see what changed. Report findings by severity.
```

---

## Tips That Save Hours

### 1. Start Every Session with Context

Don't just start typing. Give Claude context first:

```
> /status
```

Or if continuing from yesterday:

```
> /catchup
```

Or be explicit:

```
> I'm working on the user authentication feature. We're adding OAuth2 with Google.
> The PR is at #142. Pick up where the last session left off.
```

### 2. Use /clear Instead of /compact

`/compact` is lossy — it summarizes and loses detail. When context gets heavy:

1. `/clear` to wipe the slate
2. `/catchup` to reload what matters
3. Continue working

This gives you a clean 200K window with exactly the context you need. No leftover noise from earlier exploration.

### 3. Keep CLAUDE.md in Git

Your `.claude/CLAUDE.md` and `.claude/commands/` should be committed. They're as much a part of your project as your linter config. Team members get the same Claude experience.

**Don't commit:** `.claude/settings.local.json` (personal preferences, auto-gitignored).

### 4. Let Claude Explore Before You Direct

For new codebases or unfamiliar areas, let Claude figure things out:

```
> Explore the authentication module. How is it structured?
> What patterns does it use? Where are the key files?
```

This is cheaper than explaining everything yourself and often reveals things you didn't know.

### 5. Use Subagents for Verbose Work

Anything that produces lots of output — test suites, log analysis, codebase exploration — should run in a subagent so it doesn't eat your main context:

```
> Use an Explore agent to map the entire API surface of this project
```

The subagent explores in its own context window. You get a concise summary.

### 6. Check Context Before It's Too Late

```
> /context
```

Do this periodically. If you're above 60%, consider `/compact` or `/clear`. Don't wait for auto-compaction — by then Claude's quality is already degrading.

### 7. One Thing at a Time

Claude works best with clear, focused tasks. Instead of:

```
> Refactor the auth module, add tests, update the API docs, and fix the login bug
```

Do:

```
> Fix the login bug in src/auth/login.ts
```

Then after that's done:

```
> Now add tests for the fix
```

### 8. Commit Often

Ask Claude to commit after each meaningful change. Small, focused commits:
- Are easier to rewind if something goes wrong
- Give you clear checkpoints
- Make `/status` and `/catchup` more useful (git log tells the story)

### 9. Trust Plan Mode for Big Changes

For anything touching multiple files or requiring architectural decisions:

```bash
claude --permission-mode plan --model opusplan
```

Claude explores and plans in read-only mode, presents the approach, and only starts writing code after you approve. Prevents wasted work on the wrong approach.

### 10. Build Your Commands Over Time

Don't try to create every command upfront. As you find yourself repeating a workflow:

1. Notice the pattern
2. Create a command for it
3. Refine it over a few uses

Your `.claude/commands/` directory should grow organically with your actual needs.

---

## Common Mistakes to Avoid

| Mistake | Why It Hurts | Do This Instead |
|---------|-------------|-----------------|
| Launch from wrong directory | `.claude/` in wrong place, paths break | Always `cd` to your project's main folder first |
| 200-line CLAUDE.md | Wastes ~600 tokens every turn | Keep root under 30 lines |
| "You MUST always..." in CLAUDE.md | Claude doesn't reliably follow imperatives | Use hooks or commands |
| Never using `/context` | Context fills silently, quality degrades | Check periodically, compact at 60-65% |
| Huge file reads into conversation | Eats context fast | Use `offset`/`limit`, or let subagents explore |
| Not committing .claude/ to git | Team doesn't get your commands/settings | Commit everything except settings.local.json |
| Adding every MCP server | Each adds tokens to context | Only add what you actively use |
| Skipping plan mode for big changes | Wrong approach = wasted work | Use `--permission-mode plan` for multi-file changes |
| One mega-session for everything | Context gets polluted, quality drops | `/clear` + `/catchup` for fresh starts |
| Never creating commands | Repeating the same workflows manually | Automate patterns you use more than twice |

---

## Project Templates

### Minimal Setup (Any Project)

```
my-project/
├── .claude/
│   ├── settings.json          # Permission rules
│   └── commands/
│       ├── status.md          # /status
│       └── catchup.md         # /catchup
└── CLAUDE.md                  # 20-30 lines: stack, build, style, structure
```

### Standard Setup (Active Development)

```
my-project/
├── .claude/
│   ├── settings.json          # Permissions + hooks
│   ├── settings.local.json    # Personal overrides (gitignored)
│   ├── commands/
│   │   ├── status.md
│   │   ├── catchup.md
│   │   ├── review.md
│   │   └── commit.md
│   └── agents/
│       └── reviewer.md        # Custom code reviewer
├── CLAUDE.md
└── CLAUDE.local.md            # Personal notes (gitignored)
```

### Full Setup (Team Project)

```
my-project/
├── .claude/
│   ├── settings.json          # Shared permissions, hooks
│   ├── settings.local.json    # Personal (gitignored)
│   ├── commands/
│   │   ├── status.md
│   │   ├── catchup.md
│   │   ├── review.md
│   │   ├── commit.md
│   │   └── deploy.md
│   ├── agents/
│   │   ├── reviewer.md
│   │   └── security-auditor.md
│   ├── skills/
│   │   └── architecture-explorer/
│   │       └── SKILL.md
│   └── rules/
│       ├── style.md
│       └── security.md
├── CLAUDE.md                  # Lean root (~20 lines)
├── CLAUDE.local.md            # Personal (gitignored)
├── frontend/
│   └── CLAUDE.md              # Frontend-specific
└── backend/
    └── CLAUDE.md              # Backend-specific
```

---

## Quick Start Checklist

```
[ ] Launch from your project's main folder
[ ] Run /init to generate CLAUDE.md
[ ] Trim CLAUDE.md to ~20-30 lines
[ ] Create .claude/settings.json with safe allowedTools
[ ] Create /status and /catchup commands
[ ] Commit .claude/ to git
[ ] Start building — add commands, hooks, agents as patterns emerge
```

---

*For feature details, see `CLAUDE_CODE_CHEATSHEET.md` (quick reference) and `CLAUDE_CODE_DEEP_DIVES.md` (in-depth explanations).*
