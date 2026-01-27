# Oh-My-OpenCode Model Overrides Guide

This guide provides complete configuration examples for overriding agent and category models in Oh-My-OpenCode v3.1.x.

## Quick Start

### File Locations

| File | Location | Purpose |
|------|----------|---------|
| `opencode.json` | `~/.config/opencode/opencode.json` | Provider & model definitions |
| `oh-my-opencode.json` | `~/.config/opencode/oh-my-opencode.json` | Agent & category assignments |

### Configuration Priority

1. Project-level `.opencode/oh-my-opencode.json` (highest)
2. User-level `~/.config/opencode/oh-my-opencode.json`
3. Built-in defaults (lowest)

---

## Complete Antigravity Configuration

This is the recommended configuration for using Google Antigravity (no API keys required):

```jsonc
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/master/assets/oh-my-opencode.schema.json",
  "google_auth": false,
  "sisyphus_agent": {
    "disabled": false,
    "planner_enabled": true,
    "replace_plan": true
  },
  "disabled_hooks": [
    "anthropic-context-window-limit-recovery"
  ],
  "agents": {
    "sisyphus": {
      "model": "google/antigravity-claude-opus-4-5-thinking-high",
      "temperature": 0.1
    },
    "atlas": {
      "model": "google/antigravity-claude-opus-4-5-thinking-high",
      "temperature": 0.1
    },
    "sisyphus-junior": {
      "model": "google/antigravity-claude-sonnet-4-5-thinking",
      "temperature": 0.1
    },
    "prometheus": {
      "model": "google/antigravity-claude-sonnet-4-5-thinking-high",
      "temperature": 0.1
    },
    "metis": {
      "model": "google/antigravity-claude-sonnet-4-5-thinking-low",
      "temperature": 0.3
    },
    "momus": {
      "model": "google/antigravity-claude-sonnet-4-5-thinking-low",
      "temperature": 0.1
    },
    "oracle": {
      "model": "openrouter/openai/gpt-4o",
      "temperature": 0.1
    },
    "librarian": {
      "model": "google/gemini-2.5-flash",
      "temperature": 0.1
    },
    "explore": {
      "model": "google/gemini-2.5-flash",
      "temperature": 0.1
    },
    "multimodal-looker": {
      "model": "google/gemini-2.5-flash",
      "temperature": 0.1
    }
  },
  "categories": {
    "visual-engineering": {
      "model": "google/antigravity-gemini-3-pro-high",
      "temperature": 0.7
    },
    "ultrabrain": {
      "model": "google/antigravity-claude-opus-4-5-thinking-high",
      "temperature": 0.1
    },
    "artistry": {
      "model": "google/antigravity-gemini-3-pro-high",
      "temperature": 0.7
    },
    "quick": {
      "model": "google/gemini-2.5-flash",
      "temperature": 0.3
    },
    "unspecified-low": {
      "model": "google/antigravity-claude-sonnet-4-5-thinking",
      "temperature": 0.1
    },
    "unspecified-high": {
      "model": "google/antigravity-claude-opus-4-5-thinking-high",
      "temperature": 0.1
    },
    "writing": {
      "model": "google/gemini-2.5-flash",
      "temperature": 0.5
    }
  }
}
```

---

## Summary Table: All Agents

| Agent | Config Key | Default Model | Provider | Temp |
|-------|------------|---------------|----------|------|
| **Sisyphus** | `sisyphus` | anthropic/claude-opus-4-5 | Anthropic | 0.1 |
| **Atlas** | `atlas` | anthropic/claude-opus-4-5 | Anthropic | 0.1 |
| **Sisyphus-Junior** | `sisyphus-junior` | anthropic/claude-sonnet-4-5 | Anthropic | 0.1 |
| **Prometheus** | `prometheus` | anthropic/claude-opus-4-5 | Anthropic | 0.1 |
| **Metis** | `metis` | anthropic/claude-sonnet-4-5 | Anthropic | 0.3 |
| **Momus** | `momus` | anthropic/claude-sonnet-4-5 | Anthropic | 0.1 |
| **Oracle** | `oracle` | openai/gpt-5.2 | OpenAI | 0.1 |
| **Librarian** | `librarian` | opencode/big-pickle | OpenCode | 0.1 |
| **Explore** | `explore` | opencode/gpt-5-nano | OpenCode | 0.1 |
| **Multimodal-Looker** | `multimodal-looker` | google/gemini-3-flash | Google | 0.1 |

> **IMPORTANT:** All agent config keys are **lowercase**. Using `"Sisyphus"` instead of `"sisyphus"` will cause the override to be silently ignored!

### REMOVED Agents (v3.1.x)

The following agents were **REMOVED** and should be deleted from your config:

| Agent | Former Config Key | Replacement |
|-------|-------------------|-------------|
| ~~frontend-ui-ux-engineer~~ | `frontend-ui-ux-engineer` | Use `visual-engineering` category |
| ~~document-writer~~ | `document-writer` | Use `writing` category |

### RENAMED Agents (v3.1.x)

| Old Name | New Config Key |
|----------|----------------|
| ~~orchestrator-sisyphus~~ | `atlas` |

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

**Config Key:** `sisyphus`  
**Default Model:** `anthropic/claude-opus-4-5`  
**Temperature:** `0.1`

```jsonc
"agents": {
  "sisyphus": {
    "model": "google/antigravity-claude-opus-4-5-thinking-high",
    "temperature": 0.1
  }
}
```

**Role:** Primary implementation agent. Has HARD BLOCK for frontend visual changes (must delegate to visual-engineering category).

---

### Atlas (Master Orchestrator)

**Config Key:** `atlas`  
**Default Model:** `anthropic/claude-opus-4-5`  
**Temperature:** `0.1`

```jsonc
"agents": {
  "atlas": {
    "model": "google/antigravity-claude-opus-4-5-thinking-high",
    "temperature": 0.1
  }
}
```

**Role:** Master orchestrator that holds the TODO list and coordinates all specialist agents. Renamed from `orchestrator-sisyphus` in v3.1.x.

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

**Role:** Spawned by `delegate_task` tool to execute category-specific tasks. Can call `delegate_task` for sub-delegation.

---

### Prometheus (Planner)

**Config Key:** `prometheus`  
**Default Model:** `anthropic/claude-opus-4-5`  
**Temperature:** `0.1`

```jsonc
"agents": {
  "prometheus": {
    "model": "google/antigravity-claude-sonnet-4-5-thinking-high",
    "temperature": 0.1
  }
}
```

**Role:** Pure strategic planner. Never writes code. Creates detailed plans with dependency graphs and parallel execution analysis.

---

### Metis (Plan Consultant)

**Config Key:** `metis`  
**Default Model:** `anthropic/claude-sonnet-4-5`  
**Temperature:** `0.3`

```jsonc
"agents": {
  "metis": {
    "model": "google/antigravity-claude-sonnet-4-5-thinking-low",
    "temperature": 0.3
  }
}
```

**Role:** Pre-planning analysis and gap detection. Higher temperature (0.3) for creative problem identification.

---

### Momus (Plan Reviewer)

**Config Key:** `momus`  
**Default Model:** `anthropic/claude-sonnet-4-5`  
**Temperature:** `0.1`

```jsonc
"agents": {
  "momus": {
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
    "model": "openrouter/openai/gpt-4o",
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
    "model": "google/gemini-2.5-flash",
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
    "model": "google/gemini-2.5-flash",
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
    "model": "google/gemini-2.5-flash",
    "temperature": 0.1
  }
}
```

**Role:** PDF and image analysis. Restricted to read-only tool access.

---

## Air-Gapped / Local LLM Configuration

For environments without internet access, set this environment variable:

```bash
export OPENCODE_DISABLE_MODELS_FETCH=true
```

See `samples/opencode-air-gapped.json` and `samples/oh-my-opencode-air-gapped.json` for complete configuration examples.

---

## Migration Guide: v3.0.0-beta.7 → v3.1.x

### Step 1: Fix agent config keys (use lowercase)

**Before (wrong):**
```jsonc
"agents": {
  "Sisyphus": { ... },
  "Atlas": { ... },
  "Prometheus (Planner)": { ... },
  "Metis (Plan Consultant)": { ... },
  "Momus (Plan Reviewer)": { ... }
}
```

**After (correct):**
```jsonc
"agents": {
  "sisyphus": { ... },
  "atlas": { ... },
  "prometheus": { ... },
  "metis": { ... },
  "momus": { ... }
}
```

### Step 2: Remove deprecated agents

Delete these entries from your config:
- `frontend-ui-ux-engineer`
- `document-writer`

### Step 3: Rename orchestrator-sisyphus

If you have `orchestrator-sisyphus` in your config, rename it to `atlas`.

### Step 4: Update tool references

If you have custom prompts or scripts referencing `sisyphus_task`, update to `delegate_task`.

---

## Troubleshooting

### ⚠️ CRITICAL: Model Silently Fails to Load (v3.1.2+)

**Symptom:** An agent's model doesn't load, but there's no error message.

**Most Common Cause:** Invalid configuration parameters in the `agents` section.

OMO v3.1.2 introduced **strict schema validation**. Any unrecognized parameter will cause the model to **silently fail to load**.

**Example of BROKEN config:**
```jsonc
"agents": {
  "sisyphus": {
    "model": "google/antigravity-claude-opus-4-5-thinking-high",
    "temperature": 0.1,
    "thinking": {  // ❌ INVALID - causes silent failure!
      "type": "enabled",
      "budgetTokens": 10000
    }
  }
}
```

**ALLOWED parameters in agents section:**
- `model`
- `variant`
- `temperature`
- `top_p`
- `prompt`
- `prompt_append`
- `tools`
- `disable`
- `description`
- `mode`
- `color`
- `permission`

**That's it!** Any other key (like `thinking`, `maxTokens`, `thinkingLevel`, etc.) will break model loading.

**Where to put thinking config instead:**

Thinking/reasoning settings belong in `opencode.json` model definitions, NOT in `oh-my-opencode.json`:

```jsonc
// opencode.json - CORRECT place for thinking config
"provider": {
  "google": {
    "models": {
      "antigravity-claude-opus-4-5-thinking-high": {
        "options": {
          "thinkingLevel": "high"
        }
      }
    }
  }
}
```

**Quick validation command:**
```bash
# Check for invalid keys in your config
jq '.agents | to_entries[] | {agent: .key, invalid_keys: (.value | keys - ["model","variant","temperature","top_p","prompt","prompt_append","tools","disable","description","mode","color","permission"])}' ~/.config/opencode/oh-my-opencode.json
```

---

### "Model not found" errors

1. Verify config key is **lowercase** (case-sensitive!)
2. **Check for invalid parameters** (see above - most common issue!)
3. Check if using Antigravity models without the plugin installed
4. Run `bunx oh-my-opencode doctor` to check model resolution

### Agent override not working

1. Config keys must be **exactly** as shown in the table above
2. Check for typos: `sisyphus` not `Sisyphus` or `sysiphus`
3. Verify the file is valid JSON (use `jq . oh-my-opencode.json`)

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
- **RENAMED:** `orchestrator-sisyphus` → `atlas`
- **REMOVED:** `frontend-ui-ux-engineer` (use `visual-engineering` category)
- **REMOVED:** `document-writer` (use `writing` category)
- **RENAMED:** `sisyphus_task` → `delegate_task`
- **CLARIFIED:** All agent config keys are lowercase
- **BREAKING:** Strict config parameter filtering - invalid keys cause silent model loading failures
- Added air-gapped configuration support

### v3.0.0-beta.7 (2026-01-14)
- Initial documented version

---

> **Based on Commit:** `0d938059f9fc01e925392f1fe0eb47dbda3410f4`  
> **Documentation Version:** 3.1.2