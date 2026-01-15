# Oh-My-OpenCode Model Overrides Reference

> **Last Updated:** 2026-01-15 (Fixed agent config keys)  
> **Based on Commit:** `abd1ec109264611474120a8136ebc3ae59b8a20f`

This document provides a complete reference for configuring agent models in Oh-My-OpenCode. 

**Recommended Setup:** Use the **Antigravity plugin** to access Claude and Gemini models via Google OAuth, eliminating the need for separate Anthropic/OpenAI API keys.

---

## Recommended: Google Antigravity Setup

The [opencode-antigravity-auth](https://github.com/NoeFabris/opencode-antigravity-auth) plugin provides:

- **Claude models via Google quota** - Access Claude Opus/Sonnet without Anthropic API credits
- **Gemini 3 models** - Full access to Gemini 3 Pro and Flash
- **Multi-account rotation** - Automatic failover when rate-limited
- **Extended thinking support** - Configurable thinking budgets via variants

### Prerequisites

1. Install the plugin in `~/.config/opencode/opencode.json`:
   ```json
   {
     "plugin": ["opencode-antigravity-auth@beta"]
   }
   ```

2. Authenticate:
   ```bash
   opencode auth login
   ```

3. Add models to `~/.config/opencode/opencode.json` (see [Antigravity README](https://github.com/NoeFabris/opencode-antigravity-auth))

---

## Quick Fix: Complete Antigravity Config (Recommended)

Copy this to `~/.config/opencode/oh-my-opencode.json`:

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
    // CRITICAL: Agent keys must match EXACTLY as shown below!
    "Sisyphus": {
      "model": "google/antigravity-claude-opus-4-5-thinking-high",
      "temperature": 0.1
    },
    "sisyphus-junior": {
      "model": "google/antigravity-claude-sonnet-4-5-thinking",
      "temperature": 0.1
    },
    "Prometheus (Planner)": {
      "model": "google/antigravity-claude-sonnet-4-5-thinking-high",
      "temperature": 0.1
    },
    // NOTE: These MUST use the FULL names with parentheses!
    "Metis (Plan Consultant)": {
      "model": "google/antigravity-claude-sonnet-4-5-thinking-low",
      "temperature": 0.3
    },
    "Momus (Plan Reviewer)": {
      "model": "google/antigravity-claude-sonnet-4-5-thinking-low",
      "temperature": 0.1
    },
    "oracle": {
      "model": "google/antigravity-claude-opus-4-5-thinking-high",
      "temperature": 0.1
    },
    "librarian": {
      "model": "google/antigravity-gemini-3-flash",
      "temperature": 0.1
    },
    "explore": {
      "model": "google/antigravity-gemini-3-flash",
      "temperature": 0.1
    },
    "frontend-ui-ux-engineer": {
      "model": "google/antigravity-gemini-3-pro-high",
      "temperature": 0.1
    },
    "document-writer": {
      "model": "google/antigravity-gemini-3-flash",
      "temperature": 0.1
    },
    "multimodal-looker": {
      "model": "google/antigravity-gemini-3-flash",
      "temperature": 0.1
    }
  },
  "categories": {
    "visual": {
      "model": "google/antigravity-gemini-3-pro-high",
      "temperature": 0.7
    },
    "visual-engineering": {
      "model": "google/antigravity-gemini-3-pro-high",
      "temperature": 0.7
    },
    "business-logic": {
      "model": "google/antigravity-claude-opus-4-5-thinking-high",
      "temperature": 0.1
    },
    "ultrabrain": {
      "model": "google/antigravity-claude-opus-4-5-thinking-high",
      "temperature": 0.1
    },
    "quick": {
      "model": "google/antigravity-gemini-3-flash",
      "temperature": 0.3
    },
    "writing": {
      "model": "google/antigravity-gemini-3-flash",
      "temperature": 0.5
    },
    "most-capable": {
      "model": "google/antigravity-claude-opus-4-5-thinking-high",
      "temperature": 0.1
    }
  }
}
```

**Important:** Set `"google_auth": false` when using Antigravity (the plugin handles auth).

---

## Antigravity Model Reference

### Claude Models (via Antigravity)

| Model ID | Thinking | Variants | Best For |
|----------|----------|----------|----------|
| `google/antigravity-claude-opus-4-5-thinking` | Yes | `low`, `max`, `high` | Primary orchestrator (Sisyphus) |
| `google/antigravity-claude-sonnet-4-5-thinking` | Yes | `low`, `max`, `high` | Planning agents (Prometheus) |
| `google/antigravity-claude-sonnet-4-5` | No | - | Fast Claude tasks |

**Variant Suffixes:**
- `-low`: 8k thinking budget
- `-high`: 32k thinking budget (recommended for Sisyphus)
- `-max`: Maximum thinking budget

**Example:** `google/antigravity-claude-opus-4-5-thinking-high`

### Gemini 3 Models (via Antigravity)

| Model ID | Thinking Levels | Best For |
|----------|-----------------|----------|
| `google/antigravity-gemini-3-pro` | `low`, `high` | Frontend UI/UX, complex visual tasks |
| `google/antigravity-gemini-3-flash` | `minimal`, `low`, `medium`, `high` | Fast tasks, research, docs |

**Variant Suffixes:**
- `-minimal`: Minimal thinking (fastest)
- `-low`: Light thinking
- `-medium`: Balanced
- `-high`: Maximum thinking (best quality)

**Examples:**
- `google/antigravity-gemini-3-pro-high` - Best for frontend-ui-ux
- `google/antigravity-gemini-3-flash` - Good for librarian, explore, multimodal

---

## Complete Agent Model Reference

### With Antigravity (Recommended)

> **CRITICAL:** Agent keys MUST match exactly as shown below, including capitalization and parentheses!

| Agent | Config Key (EXACT) | Recommended Antigravity Model |
|-------|-------------------|-------------------------------|
| **Sisyphus** | `Sisyphus` | `google/antigravity-claude-opus-4-5-thinking-high` |
| **Sisyphus-Junior** | `sisyphus-junior` | `google/antigravity-claude-sonnet-4-5-thinking` |
| **Prometheus** | `Prometheus (Planner)` | `google/antigravity-claude-sonnet-4-5-thinking-high` |
| **Metis** | `Metis (Plan Consultant)` | `google/antigravity-claude-sonnet-4-5-thinking-low` |
| **Momus** | `Momus (Plan Reviewer)` | `google/antigravity-claude-sonnet-4-5-thinking-low` |
| **Oracle** | `oracle` | `google/antigravity-claude-opus-4-5-thinking-high` |
| **Librarian** | `librarian` | `google/antigravity-gemini-3-flash` |
| **Explore** | `explore` | `google/antigravity-gemini-3-flash` |
| **Frontend-UI-UX** | `frontend-ui-ux-engineer` | `google/antigravity-gemini-3-pro-high` |
| **Document-Writer** | `document-writer` | `google/antigravity-gemini-3-flash` |
| **Multimodal-Looker** | `multimodal-looker` | `google/antigravity-gemini-3-flash` |

### Without Antigravity (Direct API Keys)

| Agent | Config Key (EXACT) | Default Model | Provider |
|-------|-------------------|---------------|----------|
| **Sisyphus** | `Sisyphus` | `anthropic/claude-opus-4-5` | Anthropic |
| **Sisyphus-Junior** | `sisyphus-junior` | `anthropic/claude-sonnet-4-5` | Anthropic |
| **Prometheus** | `Prometheus (Planner)` | `anthropic/claude-opus-4-5` | Anthropic |
| **Metis** | `Metis (Plan Consultant)` | `anthropic/claude-sonnet-4-5` | Anthropic |
| **Momus** | `Momus (Plan Reviewer)` | `anthropic/claude-sonnet-4-5` | Anthropic |
| **Oracle** | `oracle` | `openai/gpt-5.2` | OpenAI |
| **Librarian** | `librarian` | `opencode/glm-4.7-free` | OpenCode |
| **Explore** | `explore` | `opencode/grok-code` | OpenCode |
| **Frontend-UI-UX** | `frontend-ui-ux-engineer` | `google/gemini-3-pro-preview` | Google |
| **Document-Writer** | `document-writer` | `google/gemini-3-pro-preview` | Google |
| **Multimodal-Looker** | `multimodal-looker` | `google/gemini-3-flash` | Google |

---

## Category Model Overrides

Categories spawn `Sisyphus-Junior` with optimized settings:

### With Antigravity

```jsonc
{
  "categories": {
    "visual": {
      "model": "google/antigravity-gemini-3-pro-high",
      "temperature": 0.7
    },
    "visual-engineering": {
      "model": "google/antigravity-gemini-3-pro-high",
      "temperature": 0.7
    },
    "ultrabrain": {
      "model": "google/antigravity-claude-opus-4-5-thinking-high",
      "temperature": 0.1
    },
    "quick": {
      "model": "google/antigravity-gemini-3-flash",
      "temperature": 0.3
    },
    "writing": {
      "model": "google/antigravity-gemini-3-flash",
      "temperature": 0.5
    },
    "most-capable": {
      "model": "google/antigravity-claude-opus-4-5-thinking-high",
      "temperature": 0.1
    }
  }
}
```

### Without Antigravity

| Category | Default Model | Temperature | Purpose |
|----------|---------------|-------------|---------|
| `visual-engineering` | `google/gemini-3-pro` | 0.7 | Frontend, UI/UX |
| `ultrabrain` | `openai/gpt-5.2` | 0.1 | Architecture, logic |
| `artistry` | `google/gemini-3-pro` | 0.9 | Creative work |
| `quick` | `anthropic/claude-haiku` | 0.3 | Simple tasks |
| `writing` | `google/gemini-3-flash` | 0.5 | Documentation |
| `most-capable` | `anthropic/claude-opus` | 0.1 | Hardest tasks |

---

## Provider Reference

### Antigravity Models (via Google OAuth)

| Prefix | Quota Source | Notes |
|--------|--------------|-------|
| `google/antigravity-claude-*` | Antigravity | Claude models via Google quota |
| `google/antigravity-gemini-3-*` | Antigravity | Gemini 3 models |
| `google/gemini-*-preview` | Gemini CLI | Alternative Gemini quota pool |

### Direct API Providers

| Provider | Prefix | Notes |
|----------|--------|-------|
| Anthropic | `anthropic/` | Requires ANTHROPIC_API_KEY |
| OpenAI | `openai/` | Requires OPENAI_API_KEY |
| Google | `google/` | Requires GOOGLE_API_KEY |
| OpenCode | `opencode/` | Free/included models |

### Available Antigravity Models

**Claude (Antigravity):**
- `google/antigravity-claude-opus-4-5-thinking` + variants (`-low`, `-max`, `-high`)
- `google/antigravity-claude-sonnet-4-5-thinking` + variants (`-low`, `-max`, `-high`)
- `google/antigravity-claude-sonnet-4-5` (no thinking)

**Gemini 3 (Antigravity):**
- `google/antigravity-gemini-3-pro` + variants (`-low`, `-high`)
- `google/antigravity-gemini-3-flash` + variants (`-minimal`, `-low`, `-medium`, `-high`)

**Gemini CLI (Alternative quota):**
- `google/gemini-2.5-flash`
- `google/gemini-2.5-pro`
- `google/gemini-3-flash-preview`
- `google/gemini-3-pro-preview`

**OpenCode (Free):**
- `opencode/glm-4.7-free` - Free research model
- `opencode/grok-code` - Free code exploration

---

## Disabling Agents

If you don't want an agent to be available:

```jsonc
{
  "agents": {
    // CRITICAL: Use EXACT agent keys with parentheses!
    "Prometheus (Planner)": { "disabled": true },
    "Metis (Plan Consultant)": { "disabled": true },
    "Momus (Plan Reviewer)": { "disabled": true }
  }
}
```

Or disable the entire planning system:

```jsonc
{
  "sisyphus_agent": {
    "planner_enabled": false
  }
}
```

---

## Troubleshooting

### "No API credits" Error

**With Antigravity:** Usually means rate limit. Add more Google accounts:
```bash
opencode auth login
```

**Without Antigravity:** Override to a provider you have credits for.

### "Model not found" for Antigravity

1. Ensure plugin is installed: `"plugin": ["opencode-antigravity-auth@beta"]`
2. Ensure models are defined in `~/.config/opencode/opencode.json`
3. Run `opencode auth login` to authenticate

### Rate Limiting with Parallel Agents

When spawning multiple subagents, enable PID offset in `~/.config/opencode/antigravity.json`:

```json
{
  "pid_offset_enabled": true,
  "account_selection_strategy": "round-robin"
}
```

Or add more Google accounts via `opencode auth login`.

### Config Not Applied

**Priority order (highest wins):**
1. `.opencode/oh-my-opencode.json` (project)
2. `~/.config/opencode/oh-my-opencode.json` (user)

---

## Summary Table

### Recommended (Antigravity)

> **CRITICAL:** Use EXACT config keys. `metis` ≠ `Metis (Plan Consultant)`!

| Agent | Config Key (EXACT) | Recommended Model |
|-------|-------------------|-------------------|
| Sisyphus | `Sisyphus` | `google/antigravity-claude-opus-4-5-thinking-high` |
| Sisyphus-Junior | `sisyphus-junior` | `google/antigravity-claude-sonnet-4-5-thinking` |
| Prometheus | `Prometheus (Planner)` | `google/antigravity-claude-sonnet-4-5-thinking-high` |
| Metis | `Metis (Plan Consultant)` | `google/antigravity-claude-sonnet-4-5-thinking-low` |
| Momus | `Momus (Plan Reviewer)` | `google/antigravity-claude-sonnet-4-5-thinking-low` |
| Oracle | `oracle` | `google/antigravity-claude-opus-4-5-thinking-high` |
| Librarian | `librarian` | `google/antigravity-gemini-3-flash` |
| Explore | `explore` | `google/antigravity-gemini-3-flash` |
| Frontend-UI-UX | `frontend-ui-ux-engineer` | `google/antigravity-gemini-3-pro-high` |
| Document-Writer | `document-writer` | `google/antigravity-gemini-3-flash` |
| Multimodal-Looker | `multimodal-looker` | `google/antigravity-gemini-3-flash` |

### Default (Without Antigravity)

| Agent | Config Key (EXACT) | Default | Provider |
|-------|-------------------|---------|----------|
| Sisyphus | `Sisyphus` | claude-opus-4-5 | Anthropic |
| Sisyphus-Junior | `sisyphus-junior` | claude-sonnet-4-5 | Anthropic |
| Prometheus | `Prometheus (Planner)` | claude-opus-4-5 | Anthropic |
| Metis | `Metis (Plan Consultant)` | claude-sonnet-4-5 | Anthropic |
| Momus | `Momus (Plan Reviewer)` | claude-sonnet-4-5 | Anthropic |
| Oracle | `oracle` | gpt-5.2 | OpenAI |
| Librarian | `librarian` | glm-4.7-free | OpenCode |
| Explore | `explore` | grok-code | OpenCode |
| Frontend-UI-UX | `frontend-ui-ux-engineer` | gemini-3-pro-preview | Google |
| Document-Writer | `document-writer` | gemini-3-pro-preview | Google |
| Multimodal-Looker | `multimodal-looker` | gemini-3-flash | Google |

---

*This file is maintained by the update-prompt.md process. When agents are added or removed in oh-my-opencode, this table should be updated accordingly.*
