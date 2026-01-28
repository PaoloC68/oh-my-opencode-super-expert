# MCP Tools and Plugins for Oh-My-OpenCode

This directory contains guides for integrating MCP (Model Context Protocol) servers with Oh-My-OpenCode.

## Available Guides

### Memory Systems

- [memento-local-memory.md](./memento-local-memory.md) - Local SQLite-based persistent memory with BGE-M3 embeddings (air-gapped compatible)

### Memory Protocol

- [MEMORY_PROTOCOL.md](./MEMORY_PROTOCOL.md) - Instructions for teaching LLM agents how to effectively use persistent memory

## Quick Setup - Memento

```bash
# 1. Create storage directory
mkdir -p ~/.local/share/memento

# 2. Add to opencode.json (NOT a separate mcp.json file!)
```

Add the following to `~/.config/opencode/opencode.json` or `.opencode/opencode.json`:

```jsonc
{
  "mcp": {
    "memory": {
      "type": "local",
      "command": ["npx", "@iachilles/memento@latest"],
      "environment": {
        "MEMORY_DB_PATH": "/absolute/path/to/.local/share/memento/memory.db"
      },
      "enabled": true
    }
  }
}
```

> **⚠️ Note**: Use `environment` (not `env`) and absolute paths (not `~`).

## Adding MCP Integration to OMO Agents

To make an agent memory-aware, add to `oh-my-opencode.json`:

```jsonc
{
  "agents": {
    "sisyphus": {
      "prompt_append": "\n\nYou have access to a persistent memory system via the 'memory' MCP server. At session start, use search_nodes to recall relevant context. When the user shares important technical decisions, project details, or preferences, proactively store them using create_entities and add_observations. Link related concepts with create_relations."
    }
  }
}
```
or
```jsonc
{
  "agents": {
    "sisyphus": {
      "prompt_append": "\n\n## Knowledge Systems\n\n**memento** → Cross-project personal knowledge\n- User preferences, decisions, learnings\n- Use when: 'remember that I prefer...', 'what did we decide about...'\n- Tools: create_entities, add_observations, search_nodes\n\n**serena memories** → Project-specific code context\n- Session continuity, codebase understanding\n- Use when: 'continue where we left off', 'what were we working on'\n- Tools: list_memories, prepare_for_new_conversation"
    }
  }
}
```

## Adding New Plugins

When documenting new MCP plugins:

1. Create a `<plugin-name>.md` file with installation and configuration
2. Include OpenCode MCP config format (with `type`, `command`, `environment`)
3. Document all available tools
4. Add OMO agent integration examples (using `prompt_append`)
5. Update this README

## Configuration Reference

### OpenCode MCP Format

```jsonc
{
  "mcp": {
    "server-name": {
      "type": "local",           // or "remote"
      "command": ["cmd", "args"],
      "environment": {           // NOT "env"!
        "VAR": "value"
      },
      "enabled": true,
      "timeout": 5000            // optional, in ms
    }
  }
}
```

### OMO Agent Config (allowed keys only)

In `oh-my-opencode.json` agents section, only these keys are valid:
- `model`, `variant`, `temperature`, `top_p`
- `prompt`, `prompt_append`
- `tools`, `disable`, `description`
- `mode`, `color`, `permission`

**Invalid keys** like `thinking`, `maxTokens` will cause **silent model loading failure**.

## See Also

- [OMO Configuration Guide](../configurations.md)
- [Model Overrides](../models_overrides.md)
- [Main Documentation](../oh-my-opencode-super-expert.md)
