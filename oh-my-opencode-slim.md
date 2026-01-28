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

Based on your screenshots and the technical specifications of the Slim fork, here is the optimal configuration for your remaining agent.
1. Model Selection for Designer and Fixer
Using your available models from the left window (oh-my-opencode.json), you should map them based on their strengths:
| Slim Agent | Role | Recommended Model (from your list) | Why? |
|---|---|---|---|
| designer | UI/UX & Visuals | "provider/GLM-4.7-FPB" | This is your largest multimodal model (358B/A32B). Multimodal power is essential for the Designer's "visual excellence" and aesthetic reasoning. |
| fixer | Fast Code Implementation | "provider/qwen3-coder-30b" | This model is your "coding specialist." The Fixer needs speed and high-precision implementation for well-defined tasks without the overhead of deep reasoning. |
2. Recommended Temperatures and Max Tokens
Settings for agents in the Slim fork should prioritize the specific nature of the task. Focused implementation requires low randomness, while UI design benefits from a bit of creative "vibe."
 * Designer (UI/UX):
   * Temperature: 0.7. UI work requires a "design touch." Setting it lower (e.g., 0.1) can result in sterile, repetitive interfaces that lack modern polish.
   * Max Tokens: 16,384 (or higher if supported by your provider endpoint). UI components with CSS/Tailwind can be token-heavy.
 * Fixer (Implementation):
   * Temperature: 0.1. Coding implementation must be deterministic. Higher temperatures in the Fixer role often lead to logic loops or syntax hallucinations in complex tasks.
   * Max Tokens: 8,192. Most "fixes" are surgical; providing too many tokens can encourage the model to rewrite unnecessary parts of the file.
3. Prompt Configuration in Slim
In oh-my-opencode-slim.json, you don't typically need to paste a massive text prompt. The fork uses Internal Prompt References to keep the config file clean and reduce token overhead.
In your right window , simply ensure the agent names match the Slim "Pantheon." The plugin will automatically inject the corresponding optimized prompt:
 * Designer gets the designer.ts prompt: Focuses on Tailwind mastery, micro-animations, and "visual excellence over code perfection".
 * Fixer gets the fixer.ts prompt: Focuses on being the "last step between vision and reality," performing fast, parallel execution for well-defined tasks.
4. Final Adjusted JSON (Manual Migration)
Apply these changes to lines 38-53 in your right window (oh-my-opencode-slim.json):
```"designer": {
  "model": "provider/GLM-4.7-FPB",
  "variant": "medium",
  "temperature": 0.7,
  "skills":,
  "mcps": 
},
"fixer": {
  "model": "provider/qwen3-coder-30b",
  "variant": "low",
  "temperature": 0.1,
  "skills":,
  "mcps":
}
```
⚠️ Critical Proxy Note for the Image
In your screenshot of the slim config, line 41 shows "agent-browser" under the Designer's skills. Remove this string. Even in the Slim version, enabling agent-browser will cause the agent to attempt an external connection to launch a headless browser, which your corporate Squid proxy will likely block (ERR_ACCESS_DENIED). Keep both skills and mcps arrays empty (``) for maximum stability behind your proxy.

