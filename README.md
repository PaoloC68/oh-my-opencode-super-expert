# Oh My OpenCode Super-Expert

A Claude Desktop Project for becoming an expert on [Oh-My-OpenCode](https://github.com/code-yeongyu/oh-my-opencode) - the "oh-my-zsh" for AI coding assistants.

## What is this?

This project provides comprehensive reference documentation for Claude Desktop to answer questions about Oh-My-OpenCode (OMO). Upload these files to a Claude Desktop project to get expert-level assistance with:

- Configuring agents and model overrides
- Understanding the agent delegation system
- Troubleshooting API credit issues
- Using skills, hooks, and MCPs
- Following best practices and avoiding anti-patterns

## Files

| File | Purpose |
|------|---------|
| `project-instructions.md` | **Upload to Claude Project settings** - Concise instructions for Claude |
| `oh-my-opencode-super-expert.md` | Comprehensive 800+ line reference guide |
| `models_overrides.md` | Complete model override configurations (especially for Antigravity) |
| `update-prompt.md` | Instructions for updating docs when OMO changes |

## Setup

### 1. Create a Claude Desktop Project

1. Open [Claude Desktop](https://claude.ai)
2. Create a new Project called "Oh My OpenCode Expert"
3. Go to Project Settings

### 2. Add Project Instructions

Copy the contents of `project-instructions.md` into the **Custom Instructions** field.

### 3. Add Knowledge Files

Upload these files to the project's **Knowledge** section:
- `oh-my-opencode-super-expert.md`
- `models_overrides.md`

### 4. Start Asking Questions

Now you can ask Claude about:
- "How do I configure all agents to use Antigravity models?"
- "What's the correct config key for Metis?"
- "How do I delegate frontend work properly?"
- "What anti-patterns should I avoid?"

## Key Features Covered

### Agent System
11 specialized agents including Sisyphus (orchestrator), Oracle (debugger), Librarian (researcher), and more.

### Model Overrides
Complete configurations for:
- **Google Antigravity** - Use Claude/Gemini via Google quota (no API keys!)
- Direct API providers (Anthropic, OpenAI, Google)
- Custom model assignments per agent

### Critical Knowledge

**Agent Config Keys (EXACT format required):**
| Agent | Config Key |
|-------|------------|
| Sisyphus | `Sisyphus` |
| Prometheus | `Prometheus (Planner)` |
| Metis | `Metis (Plan Consultant)` |
| Momus | `Momus (Plan Reviewer)` |
| Oracle | `oracle` |

> **Warning**: Using wrong keys like `metis` instead of `Metis (Plan Consultant)` will cause overrides to silently fail!

## Keeping Documentation Updated

When Oh-My-OpenCode releases new versions:

1. Pull latest changes to your local OMO repo
2. Use the prompt in `update-prompt.md` to regenerate documentation
3. Re-upload updated files to your Claude project

## Related Links

- [Oh-My-OpenCode Repository](https://github.com/code-yeongyu/oh-my-opencode)
- [OpenCode](https://github.com/sst/opencode)
- [Antigravity Auth Plugin](https://github.com/NoeFabris/opencode-antigravity-auth)

## License

MIT

---

*Based on Oh-My-OpenCode commit: `abd1ec109264611474120a8136ebc3ae59b8a20f`*
