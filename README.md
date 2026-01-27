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

## Sample Configurations

The `samples/` directory contains tested configuration examples:

| File | Purpose |
|------|---------|
| `opencode.json` | Working config with Antigravity + OpenRouter |
| `oh-my-opencode.json` | Working OMO config for cloud providers |
| `opencode-air-gapped.json` | Config for air-gapped/local LLM environments |
| `oh-my-opencode-air-gapped.json` | OMO config for air-gapped/local LLM environments |

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

## Air-Gapped / Local LLM Setup

For environments without internet access or when using local LLMs (e.g., via LM Studio, Ollama, vLLM):

### Required Environment Variable

**You must set this environment variable before running OpenCode:**

```bash
export OPENCODE_DISABLE_MODELS_FETCH=true
```

This prevents OpenCode from attempting to fetch model definitions from the internet.

Add to your shell profile (`~/.bashrc`, `~/.zshrc`, etc.):

```bash
# For air-gapped OpenCode environments
export OPENCODE_DISABLE_MODELS_FETCH=true
```

### Configuration Files

1. Copy `samples/opencode-air-gapped.json` to `~/.config/opencode/opencode.json`
2. Copy `samples/oh-my-opencode-air-gapped.json` to `~/.config/opencode/oh-my-opencode.json`
3. Adjust the `baseURL` in `opencode.json` to match your local LLM server:
   - LM Studio: `http://127.0.0.1:1234/v1`
   - Ollama: `http://127.0.0.1:11434/v1`
   - vLLM: `http://127.0.0.1:8000/v1`

4. Update model names to match what your local server provides

### Air-Gapped Config Structure

The air-gapped configs follow the same structure as cloud configs:

- **opencode.json**: Defines the provider and available models
- **oh-my-opencode.json**: Assigns models to agents and categories

```
~/.config/opencode/
├── opencode.json           # Provider & model definitions
└── oh-my-opencode.json     # Agent & category assignments
```

## Key Features Covered

### Agent System
10 specialized agents including Sisyphus (executor), Atlas (orchestrator), Oracle (debugger), Librarian (researcher), and more.

### Model Overrides
Complete configurations for:
- **Google Antigravity** - Use Claude/Gemini via Google quota (no API keys!)
- Direct API providers (Anthropic, OpenAI, Google)
- **Local LLMs** - Air-gapped environments with local model servers
- Custom model assignments per agent

### Critical Knowledge

**Agent Config Keys (lowercase in oh-my-opencode.json):**
| Agent | Config Key |
|-------|------------|
| Sisyphus | `sisyphus` |
| Atlas | `atlas` |
| Prometheus | `prometheus` |
| Metis | `metis` |
| Momus | `momus` |
| Oracle | `oracle` |
| Librarian | `librarian` |
| Explore | `explore` |
| Multimodal-Looker | `multimodal-looker` |
| Sisyphus-Junior | `sisyphus-junior` |

> **Note**: v3.1.x removed `frontend-ui-ux-engineer` and `document-writer` agents. Use the `visual-engineering` and `writing` categories instead.

## Keeping Documentation Updated

When Oh-My-OpenCode releases new versions:

### Prerequisites

You need **both repositories** checked out locally:

```bash
# Oh-My-OpenCode source (for reading latest changes)
git clone https://github.com/code-yeongyu/oh-my-opencode.git
# or: ~/Documents/Projects/oh-my-opencode

# This documentation project (for updating docs)
git clone https://github.com/PaoloC68/oh-my-opencode-super-expert.git
# or: ~/Documents/Projects/oh-my-opencode-super-expert
```

### Update Workflow

1. Pull latest changes in your local OMO repo
2. Open `update-prompt.md` and follow the instructions
3. The prompt will diff against the last documented commit and update relevant sections
4. Re-upload updated files to your Claude project

## MCP Tools and Plugins

The [`mcp_tools_and_plugins/`](./mcp_tools_and_plugins/) directory contains documentation for MCP servers that enhance OMO:

| Plugin | Description | Air-Gapped |
|--------|-------------|------------|
| [Memento](./mcp_tools_and_plugins/memento-local-memory.md) | Local SQLite + BGE-M3 semantic memory | Yes | 

See also:
- [Memory Protocol](./mcp_tools_and_plugins/MEMORY_PROTOCOL.md) - How to teach agents to use persistent memory

## Related Links

- [Oh-My-OpenCode Repository](https://github.com/code-yeongyu/oh-my-opencode)
- [OpenCode](https://github.com/sst/opencode)
- [Antigravity Auth Plugin](https://github.com/NoeFabris/opencode-antigravity-auth)

## License

MIT

---

*Based on Oh-My-OpenCode v3.1.2 - commit: `0d938059f9fc01e925392f1fe0eb47dbda3410f4`*