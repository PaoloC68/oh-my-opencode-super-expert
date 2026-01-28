Variants in the OpenCode ecosystem are pre-defined or custom configurations that allow you to toggle specific model capabilities—such as "reasoning effort" or "thinking levels"—without needing to create duplicate provider entries. They are essentially profiles for the same model that adjust parameters like `reasoningEffort`, `textVerbosity`, or `budgetTokens`.

### **How to Use Variants**

To apply a variant to your `orchestrator` (the agent we configured previously), add the `variant` key to the agent's block in your `oh-my-opencode-slim.json`:

```json
{
  "agents": {
    "orchestrator": {
      "model": "your-provider/model-name",
      "variant": "high",
      "prompt_append": "\n\nYou have access to a persistent memory system..."
    }
  }
}

```

### **Common and Custom Variants**

* **Built-in Variants:** Most providers support standard variants like `high`, `low`, `max`, or `thinking`. For example, the Zen provider uses these to switch between "fast" responses and "thinking" modes.


* **Defining Custom Variants:** If your OpenAI-compatible provider supports specific parameters, you can define your own variants in your main **`opencode.json`** under the provider section:

```json
"provider": {
  "my-compatible-provider": {
    "models": {
      "my-model-id": {
        "variants": {
          "deep-reason": {
            "options": {
              "reasoningEffort": "high",
              "textVerbosity": "low",
              "reasoningSummary": "auto"
            }
          }
        }
      }
    }
  }
}

```

Once defined in your global config, you can reference the variant name (`deep-reason`) inside your `oh-my-opencode-slim.json` for any agent using that model.

### **Why use them with Long-Running Tasks?**

Using a "high" or "thinking" variant for your `orchestrator` is highly recommended for complex, long-running tasks. While the specialized specialists (like `librarian` or `explorer`) can use "low" variants to save tokens and speed up codebase reconnaissance, the `orchestrator` benefits from the extra reasoning power to maintain context and manage the "State-First Architecture" across your tmux sessions.
