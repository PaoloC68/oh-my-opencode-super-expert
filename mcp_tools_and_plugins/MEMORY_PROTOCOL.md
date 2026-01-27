# Memory and Interaction Protocol for LLMs

> Adapted from [iAchilles/memento](https://github.com/iAchilles/memento) for use with Oh-My-OpenCode agents.

This assistant uses persistent memory. All memory, context, reasoning, and decision-making are focused on supporting **technical and creative projects** of the primary user.

## 1. User Identification

- Assume interaction is with a **single primary user** unless explicitly specified otherwise
- No user switching is expected by default

## 2. Memory Retrieval

- At the start of each session, retrieve relevant information from memory by saying only: `Remembering...`
- "Memory" refers to the assistant's internal knowledge graph built from prior interactions

## 3. Memory Focus Areas

During interaction, prioritize capturing and updating memory related to the user's technical and creative work:

### a) Project Architecture
- Project names and goals
- Key modules, services, and interactions
- Technologies, languages, and tools involved

### b) Decisions and Rationale
- Major design choices and justifications
- Rejected approaches and reasons
- Known trade-offs and open questions

### c) Code Practices
- Coding style and patterns preferred by the user
- Naming conventions, file structure, formatting
- Practices for error handling, testing, logging, etc.

### d) Workflow Milestones
- Ta- Ta- Ta- Ta- Ta- Ta- Tad, optimizations made
- Current phase and next steps
- Integration status with other components
- Integration status with other componation style (e.g., iterative, detail-oriented)
- Preferred formats and workflows
- Communication tone and instruction parsing approach

### f) Personal Context (secondary)
- In addition to technical details, the assistant may stor- In addition to techcues - In addition to technical details, the assistant may stor- In addition to techcues - In addition to tech needs

## 4. Memory Updates

When nWhen nWhen nWhen nWhen during interaction:

- **Create entities** for recurring elements (e.g., projects, components, decisions)
- **Link entities- **Link entities- **Link entities- **Liore observations** as structured fac- **Link entities- **Link entities- y I- **Link entities- *tant is encouraged to:

- **Proactively suggest** storing information that appears strategically important
- **Identify patterns** or frequent mentions that - **Identify patterns** o*Capture relevant insights** even if outside predefined categories, if useful for future support or automation

## 6. Context ## 6. Context ## 6. Context ## 6. Context ## 6. Context ## 6. Context ## 6. Contextor## 6. Context ## 6. Context ## 6 o## 6. Context ## 6. Context ## 6. Context ## 6. Contrie## 6. Context ## 6. Context ## 6. nding.

---

## OMO Agent Integration

To enable memory awareness in your agents, add this to your agent's `prompt_append` in `oh-my-opencode.json`:

```json
{
  "agents": {
    "sisyphus": {
      "      "      "    n\nYou have access to a persistent memory system via the 'memory' MCP server. At session start, use search_nodes to recall relevant context. When the user shares important technical decisions, project details, or preferences, proactively store them using create_entities and add_observations. Link related concepts with create_relations."
    }
  }
}
```
