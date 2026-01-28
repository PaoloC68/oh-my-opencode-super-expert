You should **not** directly copy your `oh-my-opencode.json` file over to `oh-my-opencode-slim.json`. The two projects have diverged significantly in their configuration schemas to support different philosophies: the main project is moving toward a dynamic "Category & Skill" system, while the **slim** fork uses a simplified "Pantheon & Presets" model to minimize token usage and dependencies.

### **Configuration Differences**

The `slim` fork replaces the complex orchestration logic of the main branch with a cleaner structure. If you force your old configuration into the new file, the plugin will either crash or ignore your settings because the keys do not match.

| Feature | `oh-my-opencode` (Main) | `oh-my-opencode-slim` |
| --- | --- | --- |
| **Primary Keys** | `agents`, `categories`, `hooks` | `presets`, `agents`, `skills` |
| **Agent Names** | `sisyphus`, `atlas`, `prometheus` 

 | `orchestrator`, `oracle`, `librarian`, `explorer`, `designer`, `fixer` |
| **Model Mapping** | Provider fallback chains 

 | Hardcoded presets (`openai`, `antigravity`, `zen-free`) |
| **Hooks** | 38+ behavioral hooks (e.g., `todo-continuation`) 

 | Core hooks only; most "bloat" removed 

 |

### **How to Migrate Surgically**

Instead of a full copy, follow these steps to preserve your specific model preferences while adopting the `slim` architecture:

1. **Run the Installer First**:
Execute `bunx oh-my-opencode-slim@latest install`. This generates a valid `oh-my-opencode-slim.json` with the correct schema for your environment.
2. **Map Agents Individually**:
Open your new `oh-my-opencode-slim.json`. Under the `agents` or `presets` section, manually point the new "Pantheon" names to your preferred models.
* **Main Orchestrator**: Map your old `sisyphus` or `atlas` model to the new `orchestrator` key.
* **Specialists**: Map `oracle`, `librarian`, and `explore` to their respective keys in the Pantheon.


3. **Handle Your Custom Provider**:
Since you use a custom OpenAI-compatible provider, ensure it is defined in your main `opencode.json`. In `oh-my-opencode-slim.json`, you can override the agent models like this:
```json
{
  "agents": {
    "orchestrator": { "model": "your-provider/model-name" },
    "oracle": { "model": "your-provider/model-name" }
  }
}

```


4. **Review the Hook System**:
The `slim` version has already removed the aggressive telemetry hooks and the SSE sync dashboard that caused your proxy crashes. You do **not** need to copy your `disabled_hooks` list, as those features are likely absent from the fork entirely.



### **Why this protects your long-running tasks**

The main branch's instability in tmux often comes from the `todo-continuation-enforcer` and the persistent SSE loop. The `slim` version removes these "helicopter parent" behaviors, meaning the system will no longer hang while attempting to report back to the `opencode.ai` dashboard via your blocked proxy.
