# Top MCP Servers for Claude Code

A curated list of MCP (Model Context Protocol) servers worth adding to your Claude Code setup.

For full MCP documentation (scopes, transports, custom servers, debugging), see the [deep dives](CLAUDE_CODE_DEEP_DIVES.md#mcp-servers).

---

## How to Install MCP Servers

### Prerequisites

- **Node.js 18+** — required for most servers (they launch via `npx`). Install: `winget install OpenJS.NodeJS.LTS` (Windows), `brew install node` (macOS), `sudo apt install nodejs npm` (Linux). Verify with `node --version`.
- **Docker** — required for the GitHub MCP server specifically.

### Method 1: CLI Wizard (Recommended)

The fastest way to add a server — Claude Code prompts you for the details:

```bash
claude mcp add
```

To add a specific server directly:

```bash
# Stdio transport (most common — runs a local process)
claude mcp add context7 -- npx -y @upstash/context7-mcp

# HTTP transport (connects to a remote URL)
claude mcp add --transport http github https://api.githubcopilot.com/mcp/
```

### Method 2: Project Config File (`.mcp.json`)

Create `.mcp.json` at your project root. Good for sharing servers across a team:

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    }
  }
}
```

### Method 3: Settings File (`settings.json`)

Add servers directly to `~/.claude/settings.json` for personal, global access:

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    }
  }
}
```

### Scopes

| Scope | Where it lives | Shared via git? | Best for |
|-------|---------------|-----------------|----------|
| **Project** | `.mcp.json` at project root | Yes | Team-required integrations |
| **User** | `~/.claude/settings.json` | No | Personal tools |

Set scope with the CLI: `claude mcp add --scope project ...` or `claude mcp add --scope user ...`

### Verify Installation

```bash
claude mcp list          # List all configured servers + status
claude mcp get <name>    # Show server details and available tools
```

Inside a session, type `/mcp` to check connected servers and their tools.

---

## The List

### GitHub MCP

| | |
|---|---|
| **What it does** | Connects Claude to GitHub — read repos, manage issues and PRs, analyze code, trigger Actions workflows. |
| **Transport** | HTTP |
| **Requires** | Docker, GitHub Personal Access Token (`repo`, `read:packages`, `read:org` scopes) |
| **Repo** | [github/github-mcp-server](https://github.com/github/github-mcp-server) |

```bash
claude mcp add --transport http github https://api.githubcopilot.com/mcp/
```

<details>
<summary>Manual config (Docker-based stdio)</summary>

```json
{
  "github": {
    "command": "docker",
    "args": ["run", "-i", "--rm", "-e", "GITHUB_PERSONAL_ACCESS_TOKEN", "ghcr.io/github/github-mcp-server"],
    "env": {
      "GITHUB_PERSONAL_ACCESS_TOKEN": "<YOUR_TOKEN>"
    }
  }
}
```
</details>

---

### Context7 MCP

| | |
|---|---|
| **What it does** | Fetches up-to-date, version-specific library docs and code examples at prompt time. Prevents hallucinated APIs and outdated code. |
| **Transport** | Stdio or HTTP |
| **Requires** | Node.js (npx handles the rest) |
| **Repo** | [upstash/context7](https://github.com/upstash/context7) |

```bash
claude mcp add context7 -- npx -y @upstash/context7-mcp
```

<details>
<summary>Remote HTTP alternative (no local process)</summary>

```json
{
  "context7": {
    "url": "https://mcp.context7.com/mcp"
  }
}
```
</details>

Optional: Get a free API key from [context7.com/dashboard](https://context7.com/dashboard) for higher rate limits.

---

### Sequential Thinking MCP

| | |
|---|---|
| **What it does** | Structured step-by-step reasoning tool. Lets the model break down complex problems, revise previous steps, and branch into alternative paths. |
| **Transport** | Stdio |
| **Requires** | Node.js (npx handles the rest) |
| **Repo** | [modelcontextprotocol/servers — sequential thinking](https://github.com/modelcontextprotocol/servers/tree/main/src/sequentialthinking) |

```bash
claude mcp add sequential-thinking -- npx -y @modelcontextprotocol/server-sequential-thinking
```

---

### CodeGraphContext MCP

| | |
|---|---|
| **What it does** | Indexes your codebase into a Neo4j graph database. Enables analysis of code relationships (call graphs, class hierarchies, dependencies), complexity metrics, and dead code detection across 12 languages. |
| **Transport** | Stdio |
| **Requires** | Python (`pip install codegraphcontext`), running Neo4j instance |
| **Repo** | [CodeGraphContext/CodeGraphContext](https://github.com/CodeGraphContext/CodeGraphContext) |

```bash
pip install codegraphcontext
claude mcp add CodeGraphContext -- cgc mcp start
```

Set Neo4j connection via env vars: `NEO4J_URI`, `NEO4J_USERNAME`, `NEO4J_PASSWORD`.

---

### Bright Data MCP

| | |
|---|---|
| **What it does** | Real-time web access — search, scrape pages to clean markdown, and (in Pro Mode) 60+ tools for browser automation, e-commerce data, and social media scraping. |
| **Transport** | Stdio |
| **Requires** | Node.js, Bright Data account + API token (free tier: 5,000 requests/month) |
| **Repo** | [brightdata/brightdata-mcp](https://github.com/brightdata/brightdata-mcp) |

```bash
claude mcp add brightdata --env API_TOKEN=<your-token> -- npx @brightdata/mcp
```

Optional env vars: `PRO_MODE=true` (enables all 60+ tools), `GROUPS=browser,ecommerce` (specific tool bundles).

---

## More Servers

The official MCP server directory has dozens more:

**[modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers/tree/main)** — community-maintained collection of MCP servers.

Some notable ones already built into the [deep dives](CLAUDE_CODE_DEEP_DIVES.md#common-mcp-servers) common servers table:
Sentry, Notion, Stripe, Postgres, Playwright, and more.
