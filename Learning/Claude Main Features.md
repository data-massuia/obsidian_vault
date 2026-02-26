# Claude Code Extension Layer: Features Explained

> A detailed breakdown of Skills, MCP, Hooks, Subagents, Plugins, and CLAUDE.md — what they are, how they differ, and when to use each.

---

## The Extension Layer: How Claude Code Gets Customized

Think of it as a stack. At the core, Claude has built-in tools (file read/write, bash, web search). Everything below is the *extension layer* you bolt on top.

---

## 1. CLAUDE.md — Persistent Project Memory

**What it is:** A Markdown file you place in your project root. Claude reads it automatically at the start of every session.

**What it does:** Sets always-on context — coding conventions, folder structure, team preferences, banned patterns. It's not a tool; it's a briefing document.

**Analogy:** Your company's onboarding wiki that a new employee reads before touching a codebase.

**Key trait:** Always loaded. No triggering required. This means it costs context window on every session, so keep it focused.

---

## 2. Skills — On-Demand Knowledge & Workflows

**What it is:** A folder containing a `SKILL.md` file (and optional scripts). Introduced October 2025, now an open standard.

**What it does:** Encodes *procedural knowledge* — how to do something, not just what something is. Claude reads available skill descriptions and decides if a skill is relevant to the current task. If so, it loads the full skill instructions on demand (like Neo downloading kung fu in *The Matrix*).

**Two types:**
- **Reference skills** — inject knowledge throughout your session (e.g., "our API style guide")
- **Action skills** — trigger specific workflows (e.g., `/deploy`)

**Key trait:** Lazy loading. Unlike CLAUDE.md, skills only consume context when relevant. This makes them much more scalable for large teams with many workflows.

**Example:** A "meeting prep" skill that knows which Notion pages to check, how to format notes, and what your team's output standards are.

---

## 3. MCP (Model Context Protocol) — External Connectivity

**What it is:** An open standard (now under the Linux Foundation, co-founded by Anthropic, OpenAI, and Block) for connecting Claude to external tools and data sources via server/client architecture.

**What it does:** Gives Claude access to real-time data and actions in external systems — GitHub issues, Notion pages, databases, Slack, Salesforce, your internal APIs. MCP handles *connectivity*, not behavior.

**How it works:** You run an MCP server (often a Node or Python package), configure it in `.mcp.json`, and Claude gains new tools with names like `mcp__github__list_issues`.

**Key trait:** MCP is about *access*. It answers "can Claude reach this thing?" — not "does Claude know what to do with it?" That's where Skills come in.

**Important nuance:** Each MCP server eats context. Claude Code has a tool search feature that lazy-loads MCP tools when the total count would exceed 10% of your context window.

---

## 4. Hooks — Event-Driven Automation

**What it is:** Shell scripts configured in `.claude/settings.json` that fire automatically at specific lifecycle events.

**What it does:** Automates reactions to what Claude does — not Claude itself making decisions, but your system enforcing rules. Zero additional context cost (hooks run externally).

**Hook events include:**
- `PreToolUse` — before Claude runs a tool (e.g., validate a bash command before execution)
- `PostToolUse` — after Claude writes a file (e.g., auto-run a linter/formatter)
- `SessionStart` — when a session opens (e.g., log to a deployment audit trail)
- `PromptSubmit` — when you send a message

**Key trait:** Deterministic and automatic. Hooks don't ask Claude for permission — they just fire. This is powerful but was also a security concern: in mid-2025, researchers showed that repository-level hooks could execute arbitrary shell commands on collaborators' machines (now patched).

**Analogy:** Git hooks, but for your AI agent loop.

---

## 5. Subagents — Isolated Task Execution

**What it is:** Separate Claude instances spawned to handle a specific subtask, running their own full agentic loop in an isolated context.

**What it does:** Prevents "context pollution." When you need Claude to read 50 files, run extensive searches, or do parallel deep dives, a subagent does all that work and returns only a summary to your main session. Your main context stays clean.

**Key trait:** Context isolation. Subagents don't inherit your conversation history or already-loaded skills. They get a fresh context with just what they need.

**When to use:** When your context is getting full, when you need parallel execution, or when the intermediate work isn't relevant to the main thread.

---

## 6. Plugins — The Packaging Layer

**What it is:** A bundle that packages skills, hooks, subagents, and MCP servers into a single installable, distributable unit.

**What it does:** Makes it easy to share complete setups across teams or publish to a marketplace. Plugin skills are namespaced (e.g., `/my-plugin:deploy`) so multiple plugins coexist without conflicts.

**Key trait:** Distribution and reuse. It's not a new capability — it's a packaging format for all the capabilities above.

---

## Side-by-Side Comparison

| Feature | Purpose | When it loads | Context cost | Author expertise needed |
|---|---|---|---|---|
| **CLAUDE.md** | Always-on project context | Every session | Always | Low (just Markdown) |
| **Skills** | On-demand knowledge & workflows | When relevant | On demand | Low-Medium (Markdown + optional scripts) |
| **MCP** | External tool/data connectivity | On demand (tool search) | Per server | High (run servers, JSON config) |
| **Hooks** | Automated lifecycle reactions | On trigger event | Zero | Medium (shell scripting) |
| **Subagents** | Isolated parallel task execution | When spawned | Isolated | Medium |
| **Plugins** | Packaging & distribution | On install | Inherited from contents | Medium-High |

---

## The Core Mental Model

These features answer different questions:

- **"What should Claude always know?"** → CLAUDE.md
- **"How should Claude do X process?"** → Skills
- **"What external systems can Claude access?"** → MCP
- **"What should happen automatically when Claude does Y?"** → Hooks
- **"How do I offload heavy work without clogging my context?"** → Subagents
- **"How do I share all of the above with my team?"** → Plugins

The most powerful setups combine them: CLAUDE.md for conventions, a skill for your deployment workflow, MCP to connect to your database and GitHub, and a hook to run linting after every file edit.

---

## For AI Product Managers & Architects

Understanding this extension architecture signals technical depth — it's essentially the scaffolding layer that makes multi-agent systems composable and maintainable. The Skill + MCP separation (expertise vs. connectivity) maps directly to design patterns seen in agent orchestration frameworks like LangGraph or CrewAI.

Key architectural principles to internalize:
- **Separation of concerns:** MCP owns connectivity, Skills own expertise, Hooks own automation
- **Context is a resource:** Every feature has a different context cost profile — design accordingly
- **Composability:** A single skill can orchestrate multiple MCP servers; a single MCP server can power dozens of skills
- **Lazy loading beats always-on:** Skills and MCP tool search both use on-demand loading to preserve context

---

*Last updated: February 2026 | Sources: Anthropic docs, Claude Code documentation, claude.com/blog*
