# Memento MCP - Local Persistent Memory for OMO

> **Air-gapped compatible** - No cloud dependencies, fully offline after initial setup.

Memento is a lightweight MCP server that provides persistent memory capabilities through a SQLite-based knowledge graph with semantic search using BGE-M3 embeddings.

## Why Memento?

| Feature | Memento (iAchilles) | Supermemory | Mem0 |
|---------|---------------------|-------------|------|
| **Storage** | Local SQLite | Cloud | Cloud/Hybrid |
| **Embeddings** | Local BGE-M3 | Cloud API | Cloud API |
| **Air-gapped** | Yes | No | No |
| **Privacy** | 100% local | Data in cloud | Data in cloud |
| **Cost** | Free | Freemium | Freemium |

## Installation

### Prerequisites

```bash
# Verify SQLite version (3.38+ required)
sqlite3 --version

# If missing or outdated:
# macOS: brew install sqlite
# Ubuntu: sudo apt install sqlite3
```

### Quick Start

```bash
# Create memory directory
mkdir -p ~/.local/share/memento

# Test standalone
MEMORY_DB_PATH="$HOME/.local/share/memento/memory.db" npx @iachilles/memento@latest
```

### First Run - Model Download

On first run, Memento downloads the BGE-M3 embedding model (~700MB). This happens once:

```javascript
// Downloads to: ~/.cache/onnxruntime-web/bge-m3/
const { pipeline } = await import('@xenova/transformers');
```

## OpenCode MCP Configuration

Add to `~/.config/opencode/opencode.json` or `.opencode/opencode.json`:

```jsonc
{
  "mcp": {
    "memory": {
      "type": "local",
      "command": ["npx", "@iachilles/memento@latest"],
      "environment": {
        "MEMORY_DB_PATH": "/Users/YOUR_USERNAME/.local/share/memento/memory.db"
      },
      "enabled": true
    }
  }
}
```

> **⚠️ Important**: Use absolute paths (not `~`) for `MEMORY_DB_PATH` to avoid expansion issues.

### Alternative: Using bun (faster startup)

```jsonc
{
  "mcp": {
    "memory": {
      "type": "local",
      "command": ["bunx", "@iachilles/memento@latest"],
      "environment": {
        "MEMORY_DB_PATH": "/Users/YOUR_USERNAME/.local/share/memento/memory.db"
      },
      "enabled": true
    }
  }
}
```

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `MEMORY_DB_PATH` | Full path to SQLite database file | `./memory.db` |
| `MEMORY_DB_DRIVER` | Database backend: `sqlite` or `postgres` | `sqlite` |
| `SQLITE_VEC_PATH` | Path to sqlite-vec extension if auto-detect fails | Auto |
| `DATABASE_URL` | PostgreSQL connection string (if using postgres) | - |

## MCP Tools Reference

| Tool | Purpose | Example Use |
|------|---------|-------------|
| `create_entities` | Create new knowledge nodes | Projects, people, components |
| `add_observations` | Add facts to entities | "Project X uses React 19" |
| `create_relations` | Link entities together | "User works_on Project X" |
| `search_nodes` | Semantic search | Find relevant context |
| `read_graph` | Retrieve full graph | Session initialization |
| `open_nodes` | Get specific nodes by name | Detailed entity info |
| `delete_entities` | Remove nodes | Cleanup |
| `delete_observations` | Remove facts | Corrections |
| `delete_relations` | Remove links | Reorganization |

## Troubleshooting

### sqlite-vec Extension Issues

Memento loads sqlite-vec programmatically through Node.js, NOT via the sqlite3 CLI.

```bash
# Find the extension if auto-detect fails
find node_modules -name "vec0.so"     # Linux
find node_modules -name "vec0.dylib"  # macOS

# Use explicitly
SQLITE_VEC_PATH="/path/to/vec0.so" npx @iachilles/memento@latest
```

### PostgreSQL Alternative

For multi-user or larger deployments:

```bash
# Requires pgvector extension
export MEMORY_DB_DRIVER=postgres
export DATABASE_URL="postgresql://user:pass@localhost:5432/memento"
npx @iachilles/memento@latest
```

## Integration with OMO Agents

See [MEMORY_PROTOCOL.md](./MEMORY_PROTOCOL.md) for the full protocol on how agents should interact with memory.

### Quick Agent Configuration

Add to your agent's `prompt_append` in `oh-my-opencode.json`:

```jsonc
{
  "agents": {
    "sisyphus": {
      "prompt_append": "\n\nYou have access to persistent memory via the 'memory' MCP server. Use search_nodes at session start. Store important decisions with create_entities and add_observations. Link concepts with create_relations."
    }
  }
}
```

### Full Memory Protocol Integration

For comprehensive memory usage, add the MEMORY_PROTOCOL.md content to your agent's system prompt via `prompt` or `prompt_append`:

```jsonc
{
  "agents": {
    "sisyphus": {
      "prompt_append": "\n\n## Memory Protocol\n\nAt session start, call `search_nodes` with relevant project keywords.\nStore new entities for: projects, components, decisions, people.\nLink entities with meaningful relations: works_on, depends_on, decided_by.\nUpdate observations when information changes."
    }
  }
}
```

## References

- **Repository**: [iAchilles/memento](https://github.com/iAchilles/memento)
- **npm Package**: [@iachilles/memento](https://www.npmjs.com/package/@iachilles/memento)
- **Embedding Model**: [Xenova/bge-m3](https://huggingface.co/Xenova/bge-m3)

---

*Last updated: January 2026 - Compatible with Memento v0.6.x*