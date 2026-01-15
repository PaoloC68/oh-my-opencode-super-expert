# Oh-My-OpenCode Documentation Update Prompt

Use this prompt whenever you want to update the documentation files after pulling new changes from the repository.

**Files to maintain:**
- `oh-my-opencode-super-expert.md` - Comprehensive reference
- `models_overrides.md` - Agent model override reference

---

## Current State

**Last documented commit:** `abd1ec109264611474120a8136ebc3ae59b8a20f`

---

## Instructions for Claude

Copy the following prompt and paste it into a new Claude conversation with the Oh-My-OpenCode Super-Expert project:

---

### UPDATE PROMPT

```
I need you to update the oh-my-opencode-super-expert.md documentation based on recent changes to the repository.

## Step 1: Detect Changes

First, run these commands to understand what changed:

```bash
cd /Users/paolo/Documents/Projects/oh-my-opencode

# Get current commit
git rev-parse HEAD

# See what changed since last documented commit
git diff abd1ec109264611474120a8136ebc3ae59b8a20f..HEAD --stat

# See commit messages for context
git log abd1ec109264611474120a8136ebc3ae59b8a20f..HEAD --oneline
```

## Step 2: Analyze Changed Areas

Based on the diff, identify which sections need updates:

| Changed Path | Update Section |
|--------------|----------------|
| `src/agents/*.ts` | Section 3: Agent System + `models_overrides.md` |
| `src/agents/index.ts` | **models_overrides.md** (agent added/removed) |
| `src/hooks/*` | Section 5: Hook System |
| `src/tools/*` | Section 6: Tool Ecosystem |
| `src/features/*` | Section 7: MCPs, Section 8: Skills |
| `src/config/schema.ts` | Section 9: Configuration |
| `README.md` | Section 1: Overview |
| `docs/*` | Section 10: Best Practices |
| `package.json` | Version in header |

## Step 3: Update Documentation

For each changed area:

1. Read the relevant AGENTS.md file in that directory
2. Read the changed source files if needed
3. Update the corresponding section in oh-my-opencode-super-expert.md

Specifically update:
- Agent table if new agents added
- Hook table if new hooks added
- Tool table if new tools added
- Anti-patterns if new ones discovered
- Configuration schema if changed

## Step 3b: Update models_overrides.md

If agents changed (`src/agents/index.ts` or individual agent files):

1. Check for NEW agents - add entry to Summary Table
2. Check for REMOVED agents - remove entry from Summary Table
3. Check for MODEL CHANGES - update default model in table
4. Update the "Quick Fix" config examples if needed

For each agent, verify:
- Config key (how it's referenced in oh-my-opencode.json)
- Default model (from agent file or index.ts)
- Provider (Anthropic/OpenAI/Google/OpenCode)

## Step 4: Update Metadata

At the end:
1. Update the version in the header (from package.json)
2. Update the commit hash in header AND footer of both files:
   - `oh-my-opencode-super-expert.md`
   - `models_overrides.md`
3. Update "Last Updated" date in both files

## Step 5: Update This File

After updating the main documentation:
1. Update the "Last documented commit" in this update-prompt.md file

## Expected Output

Provide:
1. Summary of what changed
2. Updated oh-my-opencode-super-expert.md content (or just the changed sections)
3. Updated models_overrides.md if agents changed
4. Confirmation of metadata updates
```

---

## Quick Update Checklist

When reviewing changes, pay special attention to:

- [ ] New agents in `src/agents/index.ts` → Update both docs + models_overrides.md
- [ ] Removed agents → Remove from models_overrides.md
- [ ] Changed default models → Update models_overrides.md
- [ ] New hooks in `src/hooks/index.ts`
- [ ] New tools in `src/tools/index.ts`
- [ ] New skills in `src/features/builtin-skills/`
- [ ] New MCPs in `src/mcp/`
- [ ] Changed categories in `src/tools/sisyphus-task/`
- [ ] New anti-patterns mentioned in AGENTS.md files
- [ ] Version bump in `package.json`

---

## Alternative: Quick Version Check

If you just want to check if an update is needed:

```bash
cd /Users/paolo/Documents/Projects/oh-my-opencode

# Compare current to documented
CURRENT=$(git rev-parse HEAD)
DOCUMENTED="abd1ec109264611474120a8136ebc3ae59b8a20f"

if [ "$CURRENT" = "$DOCUMENTED" ]; then
  echo "Documentation is up to date!"
else
  echo "Update needed. Changes:"
  git log $DOCUMENTED..$CURRENT --oneline | wc -l
  echo "commits since last documentation."
fi
```

---

## History of Updates

| Date | Commit | Version | Notes |
|------|--------|---------|-------|
| 2026-01-14 | abd1ec10 | 3.0.0-beta.7 | Initial documentation |

---

*Keep this file updated after each documentation refresh.*
