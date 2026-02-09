# claude-basics Status

## Documents

| File | Lines | Last Updated | Status |
|------|-------|-------------|--------|
| `CLAUDE_CODE_CHEATSHEET.md` | ~984 | 2026-02-09 | Current |
| `CLAUDE_CODE_DEEP_DIVES.md` | ~2020 | 2026-02-09 | Current |
| `START_A_PROJECT.md` | ~830 | 2026-02-09 | Current |

## Deep Dive Topics

| Topic | Status | Notes |
|-------|--------|-------|
| Agent Teams | Complete | Experimental feature, setup, examples, limitations |
| CLAUDE.md Best Practices | Complete | Golden rule, file types, lean guidelines, examples, common mistakes |
| Custom Commands & Skills | Complete | Commands vs skills, frontmatter, arguments, dynamic context, subagents, examples |
| /install-github-app | Complete | One-time setup wizard, permissions, lifecycle |
| Hooks System | Complete | All 14 events, stdin/stdout schemas, examples, gotchas |
| MCP Servers | Complete | Setup methods, scopes, transports, subagents, custom servers, debugging |
| Permissions & Auto-Approval | Complete | 6 methods, comparison table, what doesn't exist yet |
| Subagents | Complete | Built-in types, custom agents, foreground/background, parallel, MCP, hooks, memory |
| Tokenization & How LLMs Read Text | Complete | What tokens are, why they matter, context windows, cost, compaction |

## Cheat Sheet Sections

| # | Section | Status |
|---|---------|--------|
| 1 | CLI Flags & Invocation | Complete |
| 2 | Slash Commands | Complete |
| 3 | Keyboard Shortcuts | Complete |
| 4 | Permission Modes & Rules | Complete |
| 5 | Context Management | Complete |
| 6 | Model Selection | Complete |
| 7 | Plan Mode & Extended Thinking | Complete |
| 8 | Custom Commands & Skills | Complete |
| 9 | Hooks System | Complete |
| 10 | MCP Servers | Complete |
| 11 | Settings & Configuration | Complete |
| 12 | Subagents & Agent Teams | Complete |
| 13 | Git & GitHub Integration | Complete |
| 14 | IDE Integrations | Complete |
| 15 | Headless / SDK Mode | Complete |
| 16 | Checkpointing & Rewind | Complete |
| 17 | Debugging & Troubleshooting | Complete |
| 18 | Power-User Workflows | Complete |

## Next Session

**Continue work on `START_A_PROJECT.md`:**
- Full read-through for flow and consistency — the file grew iteratively, check for repetition or rough transitions
- The "Tips That Save Hours" and "Common Mistakes" sections at the bottom are still generic (from the original write) — consider updating them to match the portfolio demo tone used in the rest of the guide
- The "Project Templates" section at the bottom is also generic — may want to tie it to what the reader just built or simplify
- Quick Start Checklist at the bottom may need updating to reflect the new step order and plan mode workflow
- Verify all cross-references to deep dives still work
- Consider: should the guide walk through deploying the portfolio site as a final step?
- TODO: Replace PLACEHOLDER repo URL in "Clone This Guide" section once published

---

## Candidates for Future Deep Dives

- Checkpointing & Rewind (detailed workflow, limitations, fork sessions)
- Context Management (strategies, compaction vs /clear, CLAUDE.md organization)
- Headless / SDK Mode (CI/CD pipelines, structured output patterns, Agent SDK)
- IDE Integrations (VS Code and JetBrains setup, features, workflows)
- Model Selection & Effort Levels (when to use which model, opusplan, cost optimization)
- Settings & Configuration (full hierarchy walkthrough, enterprise managed settings)
