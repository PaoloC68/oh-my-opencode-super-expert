# Oh My OpenCode Super-Expert Reference

> **Version:** 3.0.0-beta.7  
> **Last Updated:** 2026-01-15  
> **Based on Commit:** `abd1ec109264611474120a8136ebc3ae59b8a20f`  
> **Repository:** https://github.com/code-yeongyu/oh-my-opencode

---

## Table of Contents

1. [What is Oh-My-OpenCode?](#1-what-is-oh-my-opencode)
2. [Architecture Overview](#2-architecture-overview)
3. [Agent System](#3-agent-system)
4. [Category System (sisyphus_task)](#4-category-system-sisyphus_task)
5. [Hook System](#5-hook-system)
6. [Tool Ecosystem](#6-tool-ecosystem)
7. [MCP Integrations](#7-mcp-integrations)
8. [Skills System](#8-skills-system)
9. [Configuration Reference](#9-configuration-reference)
10. [Best Practices & Workflows](#10-best-practices--workflows)
11. [Anti-Patterns (CRITICAL)](#11-anti-patterns-critical)
12. [Troubleshooting](#12-troubleshooting)
13. [Quick Reference Cards](#13-quick-reference-cards)

---

## 1. What is Oh-My-OpenCode?

**Oh-My-OpenCode (OMO)** is an endlessly extensible plugin for [OpenCode](https://github.com/sst/opencode), designed as a "batteries-included" agentic harness. Think of it as the **"oh-my-zsh" for AI coding assistants**.

### Core Value Proposition

1. **Multi-Model Orchestration**: Seamlessly orchestrates Claude, GPT-5.2, Gemini, and Grok models
2. **Specialized Agents**: Each task type gets the optimal AI model
3. **Claude Code Compatibility**: Full support for Claude Code's hooks, skills, agents, and MCPs
4. **IDE-Grade Tools**: 11 LSP tools, AST-Grep search, session management
5. **Curated MCPs**: websearch (Exa AI), context7 (docs), grep_app (GitHub search)

### Philosophy: Separation of Planning and Execution

Traditional AI agents mix planning and execution, leading to:
- Context pollution
- Goal drift
- "AI slop" (low-quality code)

OMO solves this with two distinct roles:
- **Prometheus (Planner)**: Pure strategist who never writes code
- **Sisyphus (Executor)**: Orchestrator who delegates to specialists

---

## 2. Architecture Overview

### 2.1 Directory Structure

```
oh-my-opencode/
├── src/
│   ├── agents/        # AI agents (11+): Sisyphus, Oracle, Librarian, etc.
│   ├── hooks/         # 22+ lifecycle hooks
│   ├── tools/         # LSP, AST-Grep, session management, etc.
│   ├── features/      # Claude Code compatibility layer, skills, background agents
│   ├── auth/          # Google Antigravity OAuth
│   ├── shared/        # Cross-cutting utilities
│   ├── cli/           # CLI installer, doctor
│   ├── mcp/           # MCP configurations
│   ├── config/        # Zod schema (12k lines)
│   └── index.ts       # Main plugin entry
├── docs/              # Guides and documentation
└── dist/              # Build output
```

### 2.2 Component Relationships

```
┌─────────────────────────────────────────────────────────────┐
│                        USER REQUEST                          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    HOOKS (22+ Lifecycle)                     │
│  PreToolUse │ PostToolUse │ UserPromptSubmit │ Stop │ etc.  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   SISYPHUS (Orchestrator)                    │
│           Claude Opus 4.5 with 32k Extended Thinking         │
└─────────────────────────────────────────────────────────────┘
           │                  │                    │
           ▼                  ▼                    ▼
    ┌──────────┐      ┌─────────────┐      ┌────────────┐
    │  Oracle  │      │  Librarian  │      │  Explore   │
    │ GPT-5.2  │      │ GLM-4.7     │      │ Grok Code  │
    │  Debug   │      │  Research   │      │   Grep     │
    └──────────┘      └─────────────┘      └────────────┘
           │                  │                    │
           ▼                  ▼                    ▼
┌─────────────────────────────────────────────────────────────┐
│                    TOOLS (LSP, AST, etc.)                    │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    MCPs (External Services)                  │
│           websearch │ context7 │ grep_app │ custom          │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Agent System

### 3.1 Agent Inventory

| Agent | Model | Temp | Purpose | When to Use |
|-------|-------|------|---------|-------------|
| **Sisyphus** | `anthropic/claude-opus-4-5` | 0.1 | Primary orchestrator with 32k extended thinking | Default for all complex tasks |
| **Sisyphus-Junior** | `anthropic/claude-sonnet-4-5` | 0.1 | Focused executor for delegated tasks | Auto-spawned via `sisyphus_task` |
| **Oracle** | `openai/gpt-5.2` | 0.1 | Strategic advisor, high-IQ debugging | Architecture review, 2+ failed fix attempts |
| **Librarian** | `opencode/glm-4.7-free` | 0.1 | Multi-repo analysis, docs research | External library questions, OSS examples |
| **Explore** | `opencode/grok-code` | 0.1 | Fast contextual codebase grep | Quick file/pattern searches |
| **Frontend-UI-UX** | `google/gemini-3-pro-preview` | 0.1 | Designer-developer for UI/UX | ALL visual/styling changes |
| **Document-Writer** | `google/gemini-3-pro-preview` | 0.1 | Technical writing specialist | READMEs, API docs, guides |
| **Multimodal-Looker** | `google/gemini-3-flash` | 0.1 | Visual analysis | PDFs, images, diagrams |
| **Prometheus** | `anthropic/claude-opus-4-5` | 0.1 | Strategic planner with interview mode | Complex multi-step planning via `@plan` |
| **Metis** | `anthropic/claude-sonnet-4-5` | 0.3 | Plan Consultant | Pre-planning risk analysis |
| **Momus** | `anthropic/claude-sonnet-4-5` | 0.1 | Plan Reviewer | Plan validation and quality enforcement |

### 3.2 Agent Selection Logic

```
Is it a VISUAL change (styling, layout, animation)?
  └─ YES → Delegate to frontend-ui-ux-engineer
  └─ NO  → Continue...

Is it a RESEARCH question (docs, OSS examples)?
  └─ YES → Fire librarian in background
  └─ NO  → Continue...

Is it CODEBASE EXPLORATION (find patterns, files)?
  └─ YES → Fire explore in background
  └─ NO  → Continue...

Is it ARCHITECTURE or DEBUGGING (2+ failures)?
  └─ YES → Consult Oracle
  └─ NO  → Continue...

Is it DOCUMENTATION writing?
  └─ YES → Delegate to document-writer
  └─ NO  → Handle directly with Sisyphus
```

### 3.3 Delegation Protocol (7-Section Structure)

When Sisyphus delegates tasks, prompts MUST include all 7 sections:

```markdown
1. TASK: Atomic, specific goal (one action per delegation)
2. EXPECTED OUTCOME: Concrete deliverables with success criteria
3. REQUIRED SKILLS: Which skill to invoke (e.g., git-master, playwright)
4. REQUIRED TOOLS: Explicit tool whitelist (prevents tool sprawl)
5. MUST DO: Exhaustive requirements - leave NOTHING implicit
6. MUST NOT DO: Forbidden actions - anticipate and block rogue behavior
7. CONTEXT: File paths, existing patterns, constraints
```

**Example Delegation:**
```
TASK: Fix mobile layout breaking issue in LoginButton.tsx
EXPECTED OUTCOME: Buttons align vertically on mobile, no desktop regression
REQUIRED SKILLS: frontend-ui-ux
REQUIRED TOOLS: Read, Edit, lsp_diagnostics
MUST DO: 
  - Change flex-direction at md: breakpoint
  - Test with lsp_diagnostics after edit
MUST NOT DO:
  - Modify existing desktop layout
  - Add new dependencies
CONTEXT: src/components/LoginButton.tsx, using Tailwind CSS
```

### 3.4 Background Agent Execution

Agents can run in parallel using background mode:

```typescript
// Fire multiple agents in parallel
sisyphus_task(agent="explore", prompt="Find auth implementations...", run_in_background=true)
sisyphus_task(agent="librarian", prompt="Find JWT best practices...", run_in_background=true)

// Continue working immediately...

// Collect results when needed
background_output(task_id="bg_xxxxx")

// Cancel all before final answer
background_cancel(all=true)
```

### 3.5 Resume Previous Agent Sessions

Use `resume` to continue previous agent sessions with full context:

```typescript
// Previous task returned session_id
sisyphus_task(resume="ses_abc123", prompt="The previous search missed X. Also look for Y.")
```

---

## 4. Category System (sisyphus_task)

Categories are **runtime presets** that spawn `Sisyphus-Junior` with optimized settings.

### 4.1 Built-in Categories

| Category | Model | Temp | Use Case |
|----------|-------|------|----------|
| `visual-engineering` | `gemini-3-pro` | 0.7 | Frontend, UI/UX, animations, styling |
| `ultrabrain` | `gpt-5.2` | 0.1 | Architecture, complex business logic, debugging |
| `artistry` | `gemini-3-pro` | 0.9 | Creative ideation, design concepts |
| `quick` | `claude-haiku` | 0.3 | Simple tasks, refactoring, scripts |
| `writing` | `gemini-3-flash` | 0.5 | Documentation, blogs, README |
| `most-capable` | `claude-opus` | 0.1 | Extremely difficult complex tasks |
| `general` | Inherits parent | 0.1 | Standard tasks |

### 4.2 Category Usage

```typescript
sisyphus_task(
  category="visual-engineering",
  prompt="Add a responsive chart component to the dashboard page"
)
```

### 4.3 Category + Skill Combos

| Combo Name | Category | Skills | Effect |
|------------|----------|--------|--------|
| **The Designer** | `visual-engineering` | `["frontend-ui-ux", "playwright"]` | Aesthetic UI + browser verification |
| **The Architect** | `ultrabrain` | `[]` | Pure GPT-5.2 reasoning for design review |
| **The Maintainer** | `quick` | `["git-master"]` | Quick fixes with clean commits |
| **The Writer** | `writing` | `[]` | Documentation generation |

### 4.4 Category Configuration

Override categories in `oh-my-opencode.json`:

```jsonc
{
  "categories": {
    // Define new custom category
    "korean-writer": {
      "model": "google/gemini-3-flash-preview",
      "temperature": 0.5,
      "prompt_append": "You are a Korean technical writer."
    },
    
    // Override existing category
    "visual-engineering": {
      "model": "openai/gpt-5.2",
      "temperature": 0.8
    },

    // Enable thinking mode
    "deep-reasoning": {
      "model": "anthropic/claude-opus-4-5",
      "thinking": {
        "type": "enabled",
        "budgetTokens": 32000
      }
    }
  }
}
```

---

## 5. Hook System

### 5.1 Hook Events

| Event | Timing | Can Block | Description |
|-------|--------|-----------|-------------|
| **PreToolUse** | Before tool | Yes | Validate/modify inputs |
| **PostToolUse** | After tool | No | Append context/warnings |
| **UserPromptSubmit** | On prompt | Yes | Filter/modify user input |
| **Stop** | Session idle | No | Auto-continue tasks |
| **onSummarize** | Compaction | No | State preservation |

### 5.2 Key Hooks

| Hook | Purpose |
|------|---------|
| `sisyphus-orchestrator` | Main orchestration and agent delegation (677 lines) |
| `anthropic-context-window-limit-recovery` | Auto-summarize at token limit (555 lines) |
| `ralph-loop` | Self-referential dev loop until completion |
| `directory-agents-injector` | Auto-injects local AGENTS.md context |
| `directory-readme-injector` | Auto-injects local README.md context |
| `comment-checker` | Prevents AI slop/excessive comments |
| `auto-slash-command` | Detects and executes /command patterns |
| `rules-injector` | Conditional rules from .claude/rules/ |
| `edit-error-recovery` | Recovers from tool execution failures |
| `todo-continuation-enforcer` | Forces completion of [ ] items |
| `preemptive-compaction` | Triggers summary at 85% context usage |
| `think-mode` | Dynamic thinking budget adjustment |
| `background-notification` | OS notification on task completion |

### 5.3 Disabling Hooks

```jsonc
{
  "disabled_hooks": [
    "comment-checker",      // Disable AI slop detection
    "background-notification"  // Disable OS notifications
  ]
}
```

---

## 6. Tool Ecosystem

### 6.1 LSP Tools (11 Tools)

| Tool | Purpose |
|------|---------|
| `lsp_goto_definition` | Jump to symbol definition |
| `lsp_find_references` | Find all usages across workspace |
| `lsp_symbols` | Get document or workspace symbols |
| `lsp_diagnostics` | Get errors, warnings before build |
| `lsp_rename` | Rename symbol across workspace |
| `lsp_prepare_rename` | Check if rename is valid |
| `lsp_servers` | List available LSP servers |

**LSP Specifics:**
- Lazy initialization on first call
- Auto-shutdown on idle
- Supports full LSP spec including `codeAction/resolve`

### 6.2 AST-Grep Tools

| Tool | Purpose |
|------|---------|
| `ast_grep_search` | Search code patterns with AST awareness (25 languages) |
| `ast_grep_replace` | Replace code patterns with AST rewriting |

**Pattern Syntax:**
- `$VAR` - Match single node
- `$$$` - Match multiple nodes
- Patterns must be complete AST nodes

**Example:**
```typescript
ast_grep_search(
  pattern="console.log($MSG)",
  lang="typescript"
)

ast_grep_replace(
  pattern="console.log($MSG)",
  rewrite="logger.info($MSG)",
  lang="typescript"
)
```

### 6.3 Search Tools

| Tool | Purpose |
|------|---------|
| `grep` | Content search with regex (timeout-safe) |
| `glob` | File pattern matching (timeout-safe) |

### 6.4 Session Management Tools

| Tool | Purpose |
|------|---------|
| `session_list` | List all OpenCode sessions |
| `session_read` | Read messages from a session |
| `session_search` | Search across session messages |
| `session_info` | Get session metadata |

### 6.5 Background Task Tools

| Tool | Purpose |
|------|---------|
| `sisyphus_task` | Spawn agent with category/skills |
| `call_omo_agent` | Spawn explore/librarian agents |
| `background_output` | Get result from background task |
| `background_cancel` | Cancel running background tasks |

### 6.6 Other Tools

| Tool | Purpose |
|------|---------|
| `look_at` | Analyze PDFs, images, diagrams |
| `interactive_bash` | Tmux session management |
| `slashcommand` | Execute slash commands |
| `skill` | Load and execute skills |
| `skill_mcp` | Invoke skill-embedded MCPs |

---

## 7. MCP Integrations

### 7.1 Built-in MCPs

| MCP | Purpose |
|-----|---------|
| `websearch` (Exa AI) | Real-time web search with content scraping |
| `context7` | Up-to-date documentation lookup |
| `grep_app` | Search code patterns across GitHub |

### 7.2 Claude Code MCP Loader

OMO automatically loads MCPs from `.mcp.json` files:

```jsonc
// .mcp.json
{
  "mcpServers": {
    "my-server": {
      "command": "npx",
      "args": ["-y", "my-mcp-server"],
      "env": {
        "API_KEY": "${MY_API_KEY}"  // Environment variable expansion
      }
    }
  }
}
```

**Loading Priority:**
1. `.claude/.mcp.json` (project)
2. `.mcp.json` (project root)
3. `~/.claude/.mcp.json` (user)

### 7.3 Skill-Embedded MCPs

Skills can embed MCP configurations in YAML frontmatter:

```markdown
---
name: my-skill
mcp:
  playwright:
    command: npx
    args: ["-y", "@playwright/mcp@latest"]
---

# My Skill Instructions...
```

---

## 8. Skills System

### 8.1 Built-in Skills

| Skill | Purpose | MCP |
|-------|---------|-----|
| `git-master` | Atomic commits, rebase, history search | None |
| `playwright` | Browser automation, testing, screenshots | `@playwright/mcp` |
| `frontend-ui-ux` | Designer mindset, color/typography/motion | None |

### 8.2 Skill Usage

```typescript
// Load skill directly
skill(name="git-master")

// Use with sisyphus_task
sisyphus_task(
  category="quick",
  skills=["git-master"],
  prompt="Commit current changes. Follow commit message style."
)
```

### 8.3 Git-Master Skill Details

**Core Principle: MULTIPLE COMMITS BY DEFAULT**

| Files Changed | Minimum Commits Required |
|---------------|-------------------------|
| 3+ files | 2+ commits |
| 5+ files | 3+ commits |
| 10+ files | 5+ commits |

**Split By:**
- Different directories/modules
- Different component types (model/service/view)
- Can be reverted independently
- Different concerns (UI/logic/config/test)

**Commit Style Detection:**
Analyzes last 30 commits to detect:
- `SEMANTIC`: `feat:`, `fix:`, `chore:`, etc.
- `PLAIN`: "Add login feature"
- `SHORT`: "format", "lint"
- Language: Korean vs English

### 8.4 Custom Skill Creation

Create skills in `.opencode/skills/` or `~/.claude/skills/`:

```markdown
---
name: my-skill
description: My custom skill
mcp:
  my-mcp:
    command: npx
    args: ["-y", "my-mcp-server"]
---

# My Skill Instructions

You are an expert in...
```

**Skill Loading Priority:**
1. `.opencode/skill/`
2. `~/.config/opencode/skill/`
3. `.claude/skills/`
4. `~/.claude/skills/`

---

## 9. Configuration Reference

### 9.1 Configuration File Locations

| Level | Path | Priority |
|-------|------|----------|
| User | `~/.config/opencode/oh-my-opencode.json` | Lower |
| Project | `.opencode/oh-my-opencode.json` | Higher |

**JSONC Support:** Comments (`//`, `/* */`) and trailing commas are allowed.

### 9.2 Full Configuration Schema

```jsonc
{
  // Agent configuration
  "agents": {
    "oracle": {
      "model": "openai/gpt-5.2",  // Override model
      "disabled": false           // Disable agent
    }
  },

  // Sisyphus orchestrator settings
  "sisyphus_agent": {
    "disabled": false,           // Enable/disable Sisyphus
    "planner_enabled": true,     // Enable Prometheus planner
    "replace_plan": true         // Replace default plan agent
  },

  // Category overrides
  "categories": {
    "visual-engineering": {
      "model": "google/gemini-3-pro",
      "temperature": 0.7
    }
  },

  // Claude Code compatibility
  "claude_code": {
    "mcp": true,        // Load .mcp.json files
    "commands": true,   // Load commands/*.md
    "skills": true,     // Load skills/*/SKILL.md
    "agents": true,     // Load agents/*.md
    "hooks": true       // Load settings.json hooks
  },

  // Disable specific hooks
  "disabled_hooks": [
    "comment-checker"
  ],

  // Disable specific skills
  "disabled_skills": [
    "playwright"
  ],

  // Git-Master configuration
  "git_master": {
    "commit_footer": true,           // Add footer to commits
    "include_co_authored_by": true   // Add co-author trailer
  },

  // MCP configuration
  "mcps": {
    "websearch": true,
    "context7": true,
    "grep_app": true
  }
}
```

### 9.3 Permission Settings

Configure tool permissions:

```jsonc
{
  "permissions": {
    "bash": "ask",           // Prompt before running
    "write": "allow",        // Auto-allow
    "dangerous_tool": "deny" // Block completely
  }
}
```

---

## 10. Best Practices & Workflows

### 10.1 Magic Keywords

| Keyword | Effect |
|---------|--------|
| `ultrawork` / `ulw` | Maximum performance autonomous mode |
| `@plan` | Invoke Prometheus for strategic planning |
| `/start-work` | Execute plans from `.sisyphus/plans/` |

### 10.2 Workflow Decision Tree

```
Is it a quick fix or simple task?
  └─ YES → Just prompt normally
  └─ NO  → Is explaining full context tedious?
             └─ YES → Type "ulw" and let agent figure it out
             └─ NO  → Need precise, verifiable execution?
                        └─ YES → Use @plan then /start-work
                        └─ NO  → Just use "ulw"
```

### 10.3 Optimal Delegation Patterns

**For Visual Work:**
```
User: "Make the button prettier"
→ Sisyphus delegates to frontend-ui-ux-engineer
→ Agent uses frontend-ui-ux skill
→ Results verified with playwright (optional)
```

**For Research:**
```
User: "How do I use Prisma's new feature?"
→ Sisyphus fires librarian in background
→ Continues with other work
→ Collects results via background_output
```

**For Complex Planning:**
```
User: "@plan Migrate auth to NextAuth"
→ Prometheus enters interview mode
→ Metis consults on risks
→ Plan saved to .sisyphus/plans/
→ User runs /start-work
→ Sisyphus executes with delegation
```

### 10.4 DO / DON'T Summary

| DO | DON'T |
|----|-------|
| Use `bun` for all package operations | Use `npm` or `yarn` |
| Delegate visual work to frontend-ui-ux | Edit CSS/styling directly |
| Fire explore/librarian in parallel | Wait for one before starting another |
| Verify subagent results independently | Trust self-reports blindly |
| Create atomic commits (1-4 files each) | Giant commits with 5+ unrelated files |
| Use `@plan` for complex multi-step work | Jump into implementation immediately |
| Run `lsp_diagnostics` after edits | Skip verification |
| Mark todos `completed` immediately | Batch completions |

---

## 11. Anti-Patterns (CRITICAL)

### 11.1 Code Quality Anti-Patterns

| Forbidden | Why | What To Do Instead |
|-----------|-----|-------------------|
| `as any` | Breaks type safety | Fix the actual type issue |
| `@ts-ignore` | Hides errors | Address the underlying problem |
| `@ts-expect-error` | Technical debt | Proper typing |
| Empty `catch(e) {}` | Silent failures | Log and handle errors |
| Deleting failing tests | False success | Fix the code, not the test |

### 11.2 Git Anti-Patterns

| Forbidden | Why | What To Do Instead |
|-----------|-----|-------------------|
| 1 commit for 3+ files | Not atomic | Split by module/concern |
| Separating test from impl | Should be together | Same commit always |
| Default to semantic commits | May not match repo style | Detect from git log first |
| Rewriting pushed history | Breaks others' work | Only with explicit permission |

### 11.3 Development Anti-Patterns

| Forbidden | Why | What To Do Instead |
|-----------|-----|-------------------|
| npm/yarn | Project uses bun | Use `bun` exclusively |
| @types/node | Wrong types | Use `bun-types` |
| Bash for file ops | Should use tools | Use Read/Write/Edit tools |
| Year 2024 in code | Outdated | Use current year |
| Trusting subagent reports | May be wrong | Always verify independently |

### 11.4 Agent Anti-Patterns

| Forbidden | Why | What To Do Instead |
|-----------|-----|-------------------|
| Temperature >0.3 for code | Inconsistent output | Keep at 0.1 for code agents |
| Sequential agent calls | Slow | Use `run_in_background` for parallelism |
| Direct frontend edits | Not specialized | Delegate to frontend-ui-ux |
| Skip TODO creation | Loses track | Always create TODOs for multi-step tasks |
| Batch TODO completions | Defeats tracking | Mark complete immediately |

---

## 12. Troubleshooting

### 12.1 Common Issues

**"LSP tools not working"**
1. Run `bunx oh-my-opencode doctor`
2. Check LSP server is installed
3. Verify config in `opencode.json`

**"MCP not loading"**
1. Check `.mcp.json` syntax
2. Verify environment variables are set
3. Check Claude Code compatibility settings

**"Agent not responding"**
1. Check if agent is disabled in config
2. Verify model API key is configured
3. Check OpenCode version >= 1.0.150

**"Hooks not firing"**
1. Check `disabled_hooks` in config
2. Verify hook event type matches
3. Check for errors in hook execution

### 12.2 Diagnostic Commands

```bash
# Run full health check
bunx oh-my-opencode doctor

# Check specific subsystem
bunx oh-my-opencode doctor --check lsp
bunx oh-my-opencode doctor --check auth
bunx oh-my-opencode doctor --check config

# Interactive installation
bunx oh-my-opencode install
```

---

## 13. Quick Reference Cards

### 13.1 Agent Quick Reference

```
VISUAL → frontend-ui-ux-engineer (Gemini 3 Pro)
LOGIC  → Oracle (GPT-5.2) or Sisyphus directly
DOCS   → Librarian (GLM-4.7) for research
GREP   → Explore (Grok Code) for codebase search
WRITE  → Document-Writer (Gemini 3 Pro)
PLAN   → Prometheus (Claude Opus 4.5)
```

### 13.2 Category Quick Reference

```
visual-engineering → UI/UX, styling, animations
ultrabrain        → Architecture, complex logic
quick             → Simple tasks, scripts
writing           → Documentation
most-capable      → Hardest tasks
```

### 13.3 Tool Quick Reference

```
LSP:
  lsp_goto_definition    → Find where defined
  lsp_find_references    → Find all usages
  lsp_diagnostics        → Get errors/warnings

AST:
  ast_grep_search        → Pattern search
  ast_grep_replace       → Pattern replace

Background:
  sisyphus_task          → Spawn agent
  background_output      → Get results
  background_cancel      → Cancel tasks
```

### 13.4 Configuration Quick Reference

```
User Config:    ~/.config/opencode/oh-my-opencode.json
Project Config: .opencode/oh-my-opencode.json

Disable hooks:  { "disabled_hooks": ["hook-name"] }
Disable skills: { "disabled_skills": ["skill-name"] }
Override agent: { "agents": { "oracle": { "model": "..." } } }
```

---

## Footer

**Document Version:** 1.0.0  
**Based on Commit:** `abd1ec109264611474120a8136ebc3ae59b8a20f`  
**Last Updated:** 2026-01-14

---

*This document was generated from the oh-my-opencode source code. For updates, use the update-prompt.md to regenerate after pulling new changes.*
