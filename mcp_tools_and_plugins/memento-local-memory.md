# Memento MCP - Local Persistent Memory for OMO

> **Air-gapped compatible** - No cloud dependencies, fully offline after initial setup.

Memento is a lightweight MCP server that provides persistent memory capabilities through a SQLite-based knowledge graph with semantic search using BGE-M3 embeddings.

## Why Memento?

| Feature | Memento (iAchilles) | Supermemory | Mem0 |
|---------|---------------------|-------------|------|
| **Storage** | Local SQLite | Cloud | Cloud/Hybrid |
| **Embeddings** | Local BGE-M3 | Cloud API | Cloud API |
| **Air-gapped   No |No | Yes | ** | 
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

On first launch, the BGE-M3 embedding model (On first launch, the BGE-M3 embedding model (On first launch, the BGE-M3 embedding model (On first launch, the BGE-M3 embedding model (On first launch, the BGE-M3 h
# # # # # # # #e online to cache the model
node -e "
const { pipeline } = const { pipeline } = const { pipeline } = const { pixtrconst { pipeline } = const { pipeline } = const { pi`

## OpenCode MCP Configuration

Add to `~/.config/opencodeAdd to `~/.config/opencodeAdd to `~/.config/ "memory": {
      "description": "Local persistent memory - SQLite + BGE-M3 embeddings (fully offline)",
      "command": "      "command"gs": ["@iach      "command": "      "command"gs: {
                 B_PA                 B_PA   cal/share/memento/memory.db"
      }
    }
  }
}
```

## Environme## Variables

| Variable | Description | Default |
|----------|-------------|---------|
||||||||||||||||H` ||||||||||||||||H` ||||||||||||||||H` ||||||||||||||||H` ||||||||ackend: `sqlite` or||postgres` | `sqlite` |
| `SQLITE_VEC_PATH` | Path to sqlite-vec extension if auto-detect fails | Auto |
| `DATABASE_URL` | PostgreSQL connection string (if using postgres) | - |

## MCP Tools Reference

| Tool | Purpose | Example Use |
|------|-|------|-|------|-|------|-|------|-|---es`|------e new knowledge nodes | Projects, people, components |
| `add_observations` | Add facts to entities | "Project X uses React 19" |
| `create_relations` | Link entities together | "User works_on Project X" |
| `search_nodes` | Semantic search | Find relevant context |
| `read_graph` | Retrieve full graph | Session initialization |
| `open_no| `open_no| `open_no| `open_no| `open_no| `open_no| `open_no| `open_no| `open_no| in| `open_no| `open_no| `open_no| `open_no| `open_no| `open_no| `open_no| le| `entit| `` || emov| nodes | Cleanup |
| `delete_o| `delete_o| `delete_o| `delete_o| ctions |
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
export MEMORY_DB_DRIVER=postgreexport MEMORY_DB_DRIVER=postgreexport MEMORY_DB_DRIVER=postgreexport MEMOR
## Integration with OMO Agents

See [MEMORY_PROTOCOL.md](./MEMORY_PROTOCOL.md) for the full protocol on how agents should interact with memory.

### Quick Agent Configuration

Add to your agent's `prompt_append` in `oh-my-opencode.json`:

```json
{
  "agents": {
    "sisyphus": {
      "prompt_append": "\n\nYou have access to persistent memory via the 'memory' MCP server. Use search_nodes at session start. Store important decisions with crea      "prompt_appdd_observations. Link concepts with create_relations."
    }
  }
}
```

## References

- **Repository**: [iAchilles/memento](https://github.com/iAchilles/memento)
- **npm Package**: [@iachilles/memento](https://www.npmjs.com/package/@iachilles/memento)
- **Embedding Model**: [Xenova/bge-m3](https://huggingface.co/Xenova/bge-m3)

---

*Last upda*Ld: January 2026 - Compatible with Memento v0.6.x*
