# Oh-My-OpenCode Model Overrides Reference

> **Last Updated:** 2026-01-27 (Updated for v3.1.2 - Atlas rename, removed agents)  
> **Based on Commit:** `0d938059f9fc01e925392f1fe0eb47dbda3410f4`

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
    "Atlas": {
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
    "multimodal-looker": {
      "model": "google/antigravity-gemini-3-flash",
      "temperature": 0.1
    }
  },
  "categories": {
    "visual-engineering": {
      "model": "google/antigravity-gemini-3-pro-high"
    },
    "ultrabrain": {
      "model": "google/antigravity-claude-opus-4-5-thinking-high",
      "variant": "xhigh"
    },
    "artistry": {
      "model": "google/antigravity-gemini-3-pro-high",
      "variant": "max"
    },
    "quick": {
      "model": "google/antigravity-claude-haiku-4-5"
    },
    "unspecified-low": {
      "model": "google/antigravity-claude-sonnet-4-5-thinking"
    },
    "unspecified-high": {
      "model": "google/antigravity-claude-opus-4-5-thinking-high",
      "variant": "max"
    },
    "writing": {
      "model": "google/antigravity-gemini-3-flash"
    }
  }
}
```

---

## Summary Table: All Agents

| Agent | Config Key | Default Model | Provider | Temp |
|-------|------------|---------------|----------|------|
| **Sisyphus** | `Sisyphus` | anthropic/claude-opus-4-5 | Anthropic | 0.1 |
| **Atlas** | `Atlas` | anthropic/claude-opus-4-5 | Anthropic | 0.1 |
| **Sisyphus-Junior** | `sisyphus-junior` | anthropic/claude-sonnet-4-5 | Anthropic | 0.1 |
| **Prometheus** | `Prometheus (Planner)` | anthropic/claude-opus-4-5 | Anthropic | 0.1 |
| **Metis** | `Metis (Plan Consultant)` | anthropic/claude-sonnet-4-5 | Anthropic | 0.3 |
| **Momus** | `Momus (Plan Reviewer)` | anthropic/claude-sonnet-4-5 | Anthropic | 0.1 |
| **Oracle** | `oracle` | openai/gpt-5.2 | OpenAI | 0.1 |
| **Librarian** | `librarian` | opencode/big-pickle | OpenCode | 0.1 |
| **Explore** | `explore` | opencode/gpt-5-nano | OpenCode | 0.1 |
| **Multimodal-Looker** | `multimodal-looker` | google/gemini-3-flash | Google | 0.1 |

### REMOVED Agents (v3.1.x)

The following agents were **REMOVED** and should be deleted from your config:

| Agent | Former Config Key | Replacement |
|-------|-------------------|-------------|
| ~~frontend-ui-ux-engineer~~ | `frontend-ui-ux-engineer` | Use `visual-engineering` category |
| ~~document-writer~~ | `document-writer` | Use `writing` category |

### RENAMED Agents (v3.1.x)

| Old Name | New Config Key |
|----------|----------------|
| ~~orchestrator-sisyphus~~ | `Atlas` |

---

## Summary Table: All Categories

| Category | Config Key | Default Model | Variant |
|----------|------------|---------------|---------|
| **Visual Engineering** | `visual-engineering` | google/gemini-3-pro | - |
| **Ultrabrain** | `ultrabrain` | openai/gpt-5.2-codex | xhigh |
| **Artistry** | `artistry` | google/gemini-3-pro | max |
| **Quick** | `quick` | anthropic/claude-haiku-4-5 | - |
| **Unspecified Low** | `unspecified-low` | anthropic/claude-sonnet-4-5 | - |
| **Unspecified High** | `unspecified-high` | anthropic/claude-opus-4-5 | max |
| **Writing** | `writing` | google/gemini-3-flash | - |

---

## Individual Agent Details

### Sisyphus (Primary Executor)

**Config Key:** `Sisyphus`  
**Default Model:** `anthropic/claude-opus-4-5`  
**Temperature:** `0.1`

```jsonc
"agents": {
  "Sisyphus": {
    "model": "google/antigravity-claude-opus-4-5-thinking-high",
    "temperature": 0.1
  }
}
```

**Role:** Primary implementation agent. Has HARD BLOCK for frontend visual changes (must delegate to visual-engineering category).

---

### Atlas (Master Orchestrator)

**Config Key:** `Atlas`  
**Default Model:** `anthropic/claude-opus-4-5`  
**Temperature:** `0.1`

```jsonc
"agents": {
  "Atlas": {
    "model": "google/antigravity-claude-opus-4-5-thinking-high",
    "temperature": 0.1
  }
}
```

**Role:** Master orchestrator that holds the TODO list and coordinates all specialist agents. Renamed from `orchestrator-sisyphus` in v3.1.x.

**Note:** If you have `orchestrator-sisyphus` in your config, rename it to `Atlas`.

---

### Sisyphus-Junior (Category Executor)

**Config Key:** `sisyphus-junior`  
**Default Model:** `anthropic/claude-sonnet-4-5`  
**Temperature:** `0.1`

```jsonc
"agents": {
  "sisyphus-junior": {
    "model": "google/antigravity-claude-sonnet-4-5-thinking",
    "temperature": 0.1
  }
}
```

**Role:** Spawned by `delegate_task` tool to execute category-specific tasks. Now can call `delegate_task` for sub-delegation.

---

### Prometheus (Planner)

**Config Key:** `Prometheus (Planner)`  
**Default Model:** `anthropic/claude-opus-4-5`  
**Temperature:** `0.1`

```jsonc
"agents": {
  "Prometheus (Planner)": {
    "model": "google/antigravity-claude-sonnet-4-5-thinking-high",
    "temperature": 0.1
  }
}
```

**Role:** Pure strategic planner. Never writes code. Creates detailed plans with dependency graphs and parallel execution analysis.

---

### Metis (Plan Consultant)

**Config Key:** `Metis (Plan Consultant)`  
**Default Model:** `anthropic/claude-sonnet-4-5`  
**Temperature:** `0.3`

```jsonc
"agents": {
  "Metis (Plan Consultant)": {
    "model": "google/antigravity-claude-sonnet-4-5-thinking-low",
    "temperature": 0.3
  }
}
```

**Role:** Pre-planning analysis and gap detection. Higher temperature (0.3) for creative problem identification.

---

### Momus (Plan Reviewer)

**Config Key:** `Momus (Plan Reviewer)`  
**Default Model:** `anthropic/claude-sonnet-4-5`  
**Temperature:** `0.1`

```jsonc
"agents": {
  "Momus (Plan Reviewer)": {
    "model": "google/antigravity-claude-sonnet-4-5-thinking-low",
    "temperature": 0.1
  }
}
```

**Role:** Ruthless plan validator. Constrained to evaluate documentation, not design direction.

---

### Oracle (Debugger)

**Config Key:** `oracle`  
**Default Model:** `openai/gpt-5.2`  
**Temperature:** `0.1`

```jsonc
"agents": {
  "oracle": {
    "model": "google/antigravity-claude-opus-4-5-thinking-high",
    "temperature": 0.1
  }
}
```

**Role:** Consultation and debugging specialist. Tool-restricted: no write, edit, task, or delegate_task.

---

### Librarian (Research)

**Config Key:** `librarian`  
**Default Model:** `opencode/big-pickle`  
**Temperature:** `0.1`

```jsonc
"agents": {
  "librarian": {
    "model": "google/antigravity-gemini-3-flash",
    "temperature": 0.1
  }
}
```

**Role:** Multi-repo research via GitHub CLI and Context7. Uses dynamic year instead of hardcoded dates.

---

### Explore (Search)

**Config Key:** `explore`  
**Default Model:** `opencode/gpt-5-nano`  
**Temperature:** `0.1`

```jsonc
"agents": {
  "explore": {
    "model": "google/antigravity-gemini-3-flash",
    "temperature": 0.1
  }
}
```

**Role:** Fast contextual grep for code exploration.

---

### Multimodal-Looker (Vision)

**Config Key:** `multimodal-looker`  
**Default Model:** `google/gemini-3-flash`  
**Temperature:** `0.1`

```jsonc
"agents": {
  "multimodal-looker": {
    "model": "google/antigravity-gemini-3-flash",
    "temperature": 0.1
  }
}
```

**Role:** PDF and image analysis. Restricted to read-only tool access.

---

## Migration Guide: v3.0.0-beta.7 → v3.1.x

### Step 1: Rename orchestrator-sisyphus to Atlas

**Before:**
```jsonc
"agents": {
  "orchestrator-sisyphus": { ... }
}
```

**After:**
```jsonc
"agents": {
  "Atlas": { ... }
}
```

### Step 2: Remove deprecated agents

Delete these entries from your config:
- `frontend-ui-ux-engineer`
- `document-writer`

### Step 3: Update tool references

If you have custom prompts or scripts referencing `sisyphus_task`, update to `delegate_task`.

---

## Troubleshooting

### "Model not found" errors

1. Verify config key matches exactly (case-sensitive)
2. Check if using Antigravity models without the plugin installed
3. Run `bunx oh-my-opencode doctor` to check model resolution

### Agent not responding

1. Check model availability with your provider
2. Verify API key or Antigravity auth
3. Check `~/.config/opencode/opencode.json` for conflicting model definitions

### Category using wrong model

Category default model takes precedence over parent model. Override in `categories` section:

```jsonc
"categories": {
  "quick": {
    "model": "your-preferred-model"
  }
}
```

---

## Changelog

### v3.1.2 (2026-01-27)
- **RENAMED:** `orchestrator-sisyphus` → `Atlas`
- **REMOVED:** `frontend-ui-ux-engineer` (use `visual-engineering` category)
- **REMOVED:** `document-writer` (use `writing` category)
- **RENAMED:** `sisyphus_task` → `delegate_task`
- Updated default category models
- Added stale session detection for background agents

### v3.0.0-beta.7 (2026-01-14)
- Initial documented version

---

> **Based on Commit:** `0d938059f9fc01e925392f1fe0eb47dbda3410f4`  
> **Documentation Version:** 3.1.2
