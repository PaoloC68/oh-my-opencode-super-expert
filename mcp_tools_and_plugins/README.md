# MCP Tools and Plugins for Oh-My-OpenCode

This directory contains documentation and configuration guides for MCP (Model Context Protocol) servers that enhance Oh-My-OpenCode capabilities.

## Available Plugins

### 
| Plugin | Description | Air-Gapped |
|--------|-------------|------------|
| [Memento](./memento-local-memory.md) | Local SQLite + BGE-M3 semantic memory | Yes | 

### Memory Protocol

- [MEMORY_PROTOCOL.md](./MEMORY_PROTOCOL.md) - Instructions for teaching LLM agents how to effectively use persistent memory

## Quick Setup - Memento

```bash
# 1. Create storage directory
mkdir -p ~/.local/share/memento

# 2. Add to ~/.config/opencode/mcp.json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": ["@iachilles/memento@latest"],
      "env": {
        "MEMORY_DB_PATH": "~/.local/share/memento/memory.db"
      }
    }
  }
}
```

## Adding New Plugins

When documenting new MCP plugins:

1. Create a `<plugin-name>.md` file with installation and configuratio1. Create a `<plugin-name>.md` file with es
3. Document all available tools
4. Add OMO agent integration examples
5. Update this README

## See Also

- [OMO Configuration Guide](../samples/)
- [OMO Configurdes](..-models_overrides.md)
- [Main Documentation](../oh-my-opencode-super-expert.md)
