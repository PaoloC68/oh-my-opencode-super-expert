# Oh My OpenCode Super-Expert Reference

> **Version:** 3.1.2  
> **Last Updated:** 2026-01-27  
> **Based on Commit:** `0d938059f9fc01e925392f1fe0eb47dbda3410f4`  
> **Repository:** https://github.com/code-yeongyu/oh-my-opencode

---

## Table of Contents

1. [What is Oh-My-OpenCode?](#1-what-is-oh-my-opencode)
2. [Architecture Overview](#2-architecture-overview)
3. [Agent System](#3-agent-system)
4. [Category System (delegate_task)](#4-category-system-delegate_task)
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
- **Atlas (Orchestrator)**: Master orchestrator who manages the todo list and delegates to specialists
- **Sisyphus (Executor)**: Primary implementation agent

---

## 2. Architecture Overview

### 2.1 Directory Structure

```
oh-my-opencode/
├── src/
│   ├── agents/        # AI agents (10): Sisyphus, Atlas, Oracle, Librarian, etc.
│   ├── hooks/         # 25+ lifecycle hooks
│   ├── tools/         # LSP, AST-Grep, session management, delegate-task
│   ├── features/      # Claude Code compatibility layer, skills, background agents
│   ├── auth/          # Google Antigravity OAuth
│   ├── shared/        # Cross-cutting utilities
│   ├── cli/           # CLI installer, doctor
│   ├── mcp/           # MCP configurations
│   ├── config/        # Zod schema
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
│                    HOOKS (25+ Lifecycle)                     │
│  PreToolUse │ PostToolUse │ UserPromptSubmit │ Stop │ etc.  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   ATLAS (Master Orchestrator)                │
│           Claude Opus 4.5 with 32k Extended Thinking         │
│              Holds TODO list, coordinates specialists        │
└─────────────────────────────────────────────────────────────┘
           │                  │                    │
           ▼                  ▼                    ▼
    ┌──────────┐      ┌─────────────┐      ┌────────────┐
    │  Oracle  │      │  Librarian  │      │  Explore   │
    │ GPT-5.2  │      │ big-pickle  │      │ gpt-5-nano │
    │  Debug   │      │  Research   │      │   Grep     │
    └──────────┘      └─────────────┘      └────────────┘
           │                  │                    │
           ▼                  ▼                    ▼
┌─────────────────────────────────────────────────────────────┐
│                    delegate_task Tool                        │
│            Categories → Sisyphus-Junior Agents               │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Agent System

### 3.1 Agent Overview Table

| Agent | Config Key | Default Model | Temp | Purpose |
|-------|------------|---------------|------|---------|
| **Sisyphus** | `Sisyphus` | anthropic/claude-opus-4-5 | 0.1 | Primary executor |
| **Atlas** | `Atlas` | anthropic/claude-opus-4-5 | 0.1 | Master orchestrator (holds TODO) |
| **Prometheus** | `Prometheus (Planner)` | anthropic/claude-opus-4-5 | 0.1 | Strategic planning |
| **Metis** | `Metis (Plan Consultant)` | anthropic/claude-sonnet-4-5 | 0.3 | Pre-planning analysis |
| **Momus** | `Momus (Plan Reviewer)` | anthropic/claude-sonnet-4-5 | 0.1 | Plan validation |
| **Oracle** | `oracle` | openai/gpt-5.2 | 0.1 | Consultation, debugging |
| **Librarian** | `librarian` | opencode/big-pickle | 0.1 | Docs, GitHub search |
| **Explore** | `explore` | opencode/gpt-5-nano | 0.1 | Fast contextual grep |
| **Multimodal-Looker** | `multimodal-looker` | google/gemini-3-flash | 0.1 | PDF/image analysis |
| **Sisyphus-Junior** | `sisyphus-junior` | anthropic/claude-sonnet-4-5 | 0.1 | Category-spawned executor |

### 3.2 Tool Restrictions by Agent

| Agent | Denied Tools |
|-------|-------------|
| oracle | write, edit, task, delegate_task |
| librarian | write, edit, task, delegate_task, call_omo_agent |
| explore | write, edit, task, delegate_task, call_omo_agent |
| multimodal-looker | Allowlist: read only |
| Sisyphus | call_omo_agent |
| Sisyphus-Junior | task, delegate_task |

### 3.3 Agent Descriptions

#### Atlas (Master Orchestrator)
**File:** `src/agents/atlas.ts`

Atlas is the renamed orchestrator (previously `orchestrator-sisyphus`). It:
- Holds and manages the master TODO list
- Coordinates all specialist agents
- Enables parallel delegation by default
- Uses 32k extended thinking budget

#### Sisyphus (Primary Executor)
**File:** `src/agents/sisyphus.ts`

The main implementation agent with:
- SF Bay Area engineer persona
- HARD BLOCK for frontend visual changes (must delegate to visual-engineering)
- Direct code writing capabilities

#### Prometheus (Planner)
**File:** `src/agents/prometheus-prompt.ts`

Pure strategist that:
- Never writes code
- Uses Interview/Consultant mode
- Auto-generates plan workflow with self-review
- Strengthened plan-mode constraints with constraint-first architecture

#### Sisyphus-Junior (Category Executor)
**File:** `src/agents/sisyphus-junior.ts`

Spawned by `delegate_task` tool:
- Receives category-specific context
- Can now call `sisyphus_task`/`delegate_task`
- Uses model from category config

---

## 4. Category System (delegate_task)

> **Note:** Renamed from `sisyphus_task` to `delegate_task` in v3.1.x

### 4.1 Default Categories

| Category | Default Model | Variant | Purpose |
|----------|---------------|---------|---------|
| `visual-engineering` | google/gemini-3-pro | - | Frontend, UI/UX, design, styling, animation |
| `ultrabrain` | openai/gpt-5.2-codex | xhigh | Deep logical reasoning, complex architecture |
| `artistry` | google/gemini-3-pro | max | Highly creative/artistic tasks |
| `quick` | anthropic/claude-haiku-4-5 | - | Trivial tasks, single file changes |
| `unspecified-low` | anthropic/claude-sonnet-4-5 | - | Moderate effort, doesn't fit other categories |
| `unspecified-high` | anthropic/claude-opus-4-5 | max | High effort, doesn't fit other categories |
| `writing` | google/gemini-3-flash | - | Documentation, prose, technical writing |

### 4.2 Category Prompts

Each category injects specialized context:

- **visual-engineering**: Design-first mindset, bold aesthetics, distinctive typography
- **ultrabrain**: Strategic advisor mindset, bias toward simplicity
- **artistry**: Artistic genius mindset, push beyond conventional boundaries
- **quick**: Efficient execution, minimal overhead (with CALLER_WARNING about limited model)
- **unspecified-low**: Selection gate to verify task doesn't fit other categories
- **unspecified-high**: For genuinely unclassifiable high-effort work
- **writing**: Wordsmith mindset, clear flowing prose

### 4.3 delegate_task Tool Usage

```typescript
delegate_task({
  prompt: "Implement user authentication",
  category: "unspecified-high",
  skills: ["typescript-programmer", "git-master"],
  run_in_background: true  // Parallel execution
})
```

---

## 5. Hook System

### 5.1 Hook Overview (25+ Hooks)

| Hook | File | Lifecycle | Purpose |
|------|------|-----------|---------|
| `todoContinuationEnforcer` | todo-continuation-enforcer.ts | Stop | Ensures TODO items are completed |
| `contextWindowMonitor` | context-window-monitor.ts | Transform | Monitors context window usage |
| `sessionNotification` | session-notification.ts | Stop | Session completion alerts |
| `sessionRecovery` | session-recovery.ts | Stop | Recovers crashed sessions |
| `commentChecker` | comment-checker.ts | PostToolUse | Validates comments |
| `toolOutputTruncator` | tool-output-truncator.ts | PostToolUse | Truncates large outputs |
| `directoryAgentsInjector` | directory-agents-injector.ts | Transform | Injects AGENTS.md context |
| `directoryReadmeInjector` | directory-readme-injector.ts | Transform | Injects README context |
| `emptyTaskResponseDetector` | empty-task-response-detector.ts | Stop | Detects empty responses |
| `anthropicContextWindowLimitRecovery` | anthropic-context-window-limit-recovery.ts | Stop | Context limit recovery |
| `compactionContextInjector` | compaction-context-injector.ts | Transform | Compaction context |
| `thinkMode` | think-mode/index.ts | Transform | Extended thinking |
| `claudeCodeHooks` | claude-code-hooks.ts | Multiple | Claude Code compatibility |
| `rulesInjector` | rules-injector.ts | Transform | Project rules injection |
| `backgroundNotification` | background-notification.ts | Stop | Background task alerts |
| `autoUpdateChecker` | auto-update-checker.ts | Stop | Version updates |
| `agentUsageReminder` | agent-usage-reminder.ts | Transform | Agent reminder |
| `keywordDetector` | keyword-detector.ts | UserPromptSubmit | Detects ultrawork keywords |
| `nonInteractiveEnv` | non-interactive-env.ts | Transform | Non-interactive mode |
| `interactiveBashSession` | interactive-bash-session.ts | Multiple | Bash session |
| `thinkingBlockValidator` | thinking-block-validator.ts | PostToolUse | Validates thinking |
| `categorySkillReminder` | category-skill-reminder.ts | Transform | Reminds about categories/skills |
| `ralphLoop` | ralph-loop.ts | Stop | Loop detection |
| `autoSlashCommand` | auto-slash-command.ts | UserPromptSubmit | Auto commands |
| `editErrorRecovery` | edit-error-recovery.ts | PostToolUse | Edit error recovery |
| `prometheusMdOnly` | prometheus-md-only.ts | Transform | Prometheus markdown mode |
| `sisyphusJuniorNotepad` | sisyphus-junior-notepad.ts | Transform | Junior notepad |
| `taskResumeInfo` | task-resume-info.ts | Transform | Task resume info |
| `startWork` | start-work/index.ts | UserPromptSubmit | Start work trigger |
| **`atlas`** | atlas.ts | Multiple | **NEW: Atlas orchestrator hook** |
| **`delegateTaskRetry`** | delegate-task-retry.ts | PostToolUse | **NEW: Retry failed delegations** |
| **`questionLabelTruncator`** | question-label-truncator.ts | PostToolUse | Truncates question labels |
| **`subagentQuestionBlocker`** | subagent-question-blocker.ts | PreToolUse | **NEW: Blocks subagent questions** |

---

## 6. Tool Ecosystem

### 6.1 Built-in Tools

| Tool | Category | Purpose |
|------|----------|---------|
| `lsp_goto_definition` | LSP | Jump to definition |
| `lsp_find_references` | LSP | Find all references |
| `lsp_symbols` | LSP | List symbols |
| `lsp_diagnostics` | LSP | Get diagnostics/errors |
| `lsp_prepare_rename` | LSP | Prepare rename operation |
| `lsp_rename` | LSP | Execute rename |
| `ast_grep_search` | AST | Pattern-based code search |
| `ast_grep_replace` | AST | Pattern-based code replacement |
| `grep` | Search | Text search |
| `glob` | Search | File pattern matching |
| `session_list` | Session | List sessions |
| `session_read` | Session | Read session content |
| `session_search` | Session | Search sessions |
| `session_info` | Session | Session metadata |
| `interactive_bash` | Shell | Interactive bash |
| `call_omo_agent` | Agent | Call specialized agent |
| `look_at` | Vision | Analyze images/PDFs |
| **`delegate_task`** | Task | Delegate to category agent |
| `background_output` | Background | Get background task output |
| `background_cancel` | Background | Cancel background task |

---

## 7. MCP Integrations

### 7.1 Built-in MCPs

| MCP | File | Purpose |
|-----|------|---------|
| `websearch` | websearch.ts | Exa AI web search |
| `context7` | context7.ts | Documentation lookup |
| `grep_app` | grep-app.ts | GitHub code search |

---

## 8. Skills System

### 8.1 Built-in Skills

| Skill | Directory | Purpose |
|-------|-----------|---------|
| `agent-browser` | agent-browser/ | Browser automation, web testing |
| `dev-browser` | dev-browser/ | Persistent browser state automation |
| `frontend-ui-ux` | frontend-ui-ux/ | Stunning UI/UX design |
| `git-master` | git-master/ | Atomic commits, git operations |

### 8.2 External Skills (via skill tool)

| Skill | Domain |
|-------|--------|
| `typescript-programmer` | Production TypeScript code |
| `python-programmer` | Production Python code |
| `svelte-programmer` | Svelte components |
| `golang-tui-programmer` | Go TUI with Charmbracelet |
| `python-debugger` | Interactive Python debugging |
| `data-scientist` | DuckDB/Polars data processing |
| `prompt-engineer` | AI prompt optimization |

---

## 9. Configuration Reference

### 9.1 Main Config File

`~/.config/opencode/oh-my-opencode.json`

```jsonc
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/master/assets/oh-my-opencode.schema.json",
  "google_auth": false,
  "sisyphus_agent": {
    "disabled": false,
    "planner_enabled": true,
    "replace_plan": true
  },
  "agents": {
    // See models_overrides.md for full agent config
  },
  "categories": {
    // Override category models here
  }
}
```

### 9.2 Agent Config Keys

> **CRITICAL:** Agent keys must match EXACTLY as shown below!

| Agent | Config Key (EXACT) |
|-------|-------------------|
| Sisyphus | `Sisyphus` |
| Atlas | `Atlas` |
| Prometheus | `Prometheus (Planner)` |
| Metis | `Metis (Plan Consultant)` |
| Momus | `Momus (Plan Reviewer)` |
| Oracle | `oracle` |
| Librarian | `librarian` |
| Explore | `explore` |
| Multimodal-Looker | `multimodal-looker` |
| Sisyphus-Junior | `sisyphus-junior` |

---

## 10. Best Practices & Workflows

### 10.1 Ultrawork Mode

Trigger with keywords: `ultrawork`, `ulw`, or `/ulw`

This activates:
1. Prometheus planning mode
2. Metis consultation
3. Momus review
4. Atlas orchestration

### 10.2 Parallel Execution

Use `run_in_background: true` with `delegate_task` for parallel work:

```typescript
// Launch multiple agents in parallel
delegate_task({ prompt: "Task 1", category: "quick", run_in_background: true })
delegate_task({ prompt: "Task 2", category: "quick", run_in_background: true })
```

### 10.3 Category Selection Guide

| Task Type | Recommended Category |
|-----------|---------------------|
| UI/UX changes | `visual-engineering` |
| Complex algorithms | `ultrabrain` |
| Creative writing | `artistry` |
| Typo fixes | `quick` |
| Documentation | `writing` |
| General moderate work | `unspecified-low` |
| General complex work | `unspecified-high` |

---

## 11. Anti-Patterns (CRITICAL)

### 11.1 Trust Reports
**NEVER** trust "I'm done" - always verify outputs

### 11.2 High Temperature
Don't use >0.3 for code agents

### 11.3 Sequential Exploration
Use `delegate_task` with `run_in_background` for exploration instead of sequential calls

### 11.4 Prometheus Writing Code
Planner only - never implements

### 11.5 Wrong Category Selection
Using `quick` for complex tasks wastes less-capable model

### 11.6 Missing Skills
Always include relevant skills when delegating

---

## 12. Troubleshooting

### 12.1 Common Issues

| Issue | Solution |
|-------|----------|
| Agent not responding | Check model availability with `opencode doctor` |
| Wrong model used | Verify config key matches exactly (case-sensitive) |
| Category not found | Check spelling, use kebab-case |
| Background task stuck | Use `background_cancel` tool |
| Context overflow | Enable compaction hooks |

### 12.2 Doctor Command

```bash
bunx oh-my-opencode doctor
```

Checks:
- Model resolution
- Plugin configuration
- LSP servers
- Authentication

---

## 13. Quick Reference Cards

### 13.1 Agent Quick Reference

```
Primary: Sisyphus (claude-opus-4-5)
Orchestrator: Atlas (claude-opus-4-5)
Planner: Prometheus (claude-opus-4-5)
Debugger: Oracle (gpt-5.2)
Research: Librarian (big-pickle)
Search: Explore (gpt-5-nano)
Vision: Multimodal-Looker (gemini-3-flash)
```

### 13.2 Category Quick Reference

```
visual-engineering → gemini-3-pro (UI/UX)
ultrabrain → gpt-5.2-codex (Complex logic)
artistry → gemini-3-pro max (Creative)
quick → claude-haiku-4-5 (Trivial)
writing → gemini-3-flash (Docs)
unspecified-low → claude-sonnet-4-5 (Moderate)
unspecified-high → claude-opus-4-5 max (Complex)
```

### 13.3 Key Commands

```
/ultrawork, /ulw - Enter ultrawork mode
/start-work - Start work session
/plan - Create plan with Prometheus
```

---

## Changelog from v3.0.0-beta.7

### Agent Changes
- **RENAMED:** `orchestrator-sisyphus` → `atlas`
- **REMOVED:** `frontend-ui-ux-engineer` (use `visual-engineering` category)
- **REMOVED:** `document-writer` (use `writing` category)

### Tool Changes
- **RENAMED:** `sisyphus_task` → `delegate_task`

### New Hooks
- `createAtlasHook` - Atlas orchestrator hook
- `createDelegateTaskRetryHook` - Retry failed delegations
- `createSubagentQuestionBlockerHook` - Block subagent questions

### Improvements
- Parallel delegation enabled by default for Atlas
- Sisyphus-Junior can now call delegate_task
- Category default model takes precedence over parent model
- Stale session detection and auto-interrupt for background agents
- Per-key queue processor for background agents
- Windows/PowerShell support for non-interactive-env

---

> **Based on Commit:** `0d938059f9fc01e925392f1fe0eb47dbda3410f4`  
> **Documentation Version:** 3.1.2
