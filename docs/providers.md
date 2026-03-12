# Custom & Local Model Providers

GSD runs on the [Pi SDK](https://github.com/badlogic/pi-mono), which supports 20+ built-in providers. For anything else — local models, self-hosted endpoints, or cloud providers not built in — you configure a `models.json` file.

This guide covers setup for **Ollama**, **LM Studio**, **vLLM**, **OpenRouter** (as a proxy), **ZAI/custom cloud**, and any **generic OpenAI-compatible** endpoint.

---

## Quick Start

The fastest path is the built-in wizard:

```
gsd config
```

1. Select **"Custom/Local provider"**
2. Pick a preset (Ollama, LM Studio, vLLM, or Generic)
3. The wizard writes a `models.json` template to `~/.gsd/agent/models.json` and opens it in your editor
4. If the preset needs an API key, you'll be prompted — the key is stored in `~/.gsd/agent/auth.json`, not in `models.json`
5. Edit the template if needed (change model IDs, add models, adjust settings)
6. **Restart GSD** for changes to take effect

> **Restart required.** GSD reads `models.json` at startup. Any changes require restarting the process.

---

## Manual Setup

If you prefer to skip the wizard, create the file directly:

```
~/.gsd/agent/models.json
```

The file uses this structure:

```json
{
  "providers": {
    "<provider-name>": {
      "baseUrl": "http://...",
      "api": "openai-completions",
      "apiKey": "ENV_VAR_NAME",
      "models": [
        { "id": "model-id" }
      ]
    }
  }
}
```

- `<provider-name>` — any string you choose (e.g. `"ollama"`, `"my-cloud"`)
- `baseUrl` — the provider's OpenAI-compatible API endpoint
- `api` — the API format: `"openai-completions"` for chat completions, `"openai-responses"` for the responses API
- `apiKey` — an environment variable name, a `!command`, or a placeholder string (see [API Key Resolution](#api-key-resolution))
- `models` — array of model definitions (at minimum, each needs an `id`)

After creating the file, restart GSD.

---

## Provider Examples

### Ollama

Local inference with [Ollama](https://ollama.com). No API key required.

```json
{
  "providers": {
    "ollama": {
      "baseUrl": "http://localhost:11434/v1",
      "api": "openai-completions",
      "apiKey": "ollama",
      "models": [
        { "id": "llama3.1:8b" }
      ]
    }
  }
}
```

Start Ollama with `ollama serve`, pull a model with `ollama pull llama3.1:8b`, then restart GSD. The `apiKey` field is set to the literal `"ollama"` — Ollama doesn't require authentication, but the field must be present.

### LM Studio

Local inference with [LM Studio](https://lmstudio.ai). No API key required.

```json
{
  "providers": {
    "lm-studio": {
      "baseUrl": "http://localhost:1234/v1",
      "api": "openai-completions",
      "apiKey": "lm-studio",
      "models": [
        { "id": "loaded-model" }
      ]
    }
  }
}
```

Load a model in LM Studio's UI, start the local server, then restart GSD. Replace `"loaded-model"` with the actual model identifier shown in LM Studio.

### vLLM

Self-hosted inference with [vLLM](https://docs.vllm.ai). Requires an API key if your deployment is secured.

```json
{
  "providers": {
    "vllm": {
      "baseUrl": "http://localhost:8000/v1",
      "api": "openai-completions",
      "apiKey": "VLLM_API_KEY",
      "models": [
        { "id": "meta-llama/Llama-3.1-8B" }
      ]
    }
  }
}
```

Set `VLLM_API_KEY` in your shell environment before starting GSD. If your vLLM instance has no auth, set `apiKey` to any non-empty string (e.g. `"none"`).

### OpenRouter (as proxy)

Use [OpenRouter](https://openrouter.ai) as a proxy to access hundreds of models. GSD has OpenRouter as a built-in provider — use this manual setup only if you need custom headers, routing preferences, or want to combine OpenRouter models with other custom providers.

```json
{
  "providers": {
    "openrouter": {
      "baseUrl": "https://openrouter.ai/api/v1",
      "api": "openai-completions",
      "apiKey": "OPENROUTER_API_KEY",
      "models": [
        { "id": "deepseek/deepseek-r1" },
        { "id": "meta-llama/llama-3.1-405b-instruct" }
      ]
    }
  }
}
```

Set `OPENROUTER_API_KEY` in your environment. For built-in OpenRouter support (no `models.json` needed), run `gsd config` and select OpenRouter directly.

### ZAI / Custom Cloud Provider

Any cloud provider that exposes an OpenAI-compatible endpoint.

```json
{
  "providers": {
    "zai": {
      "baseUrl": "https://api.zai.example.com/v1",
      "api": "openai-completions",
      "apiKey": "ZAI_API_KEY",
      "models": [
        { "id": "zai-large" },
        { "id": "zai-fast", "contextWindow": 32768, "maxTokens": 8192 }
      ]
    }
  }
}
```

Replace the `baseUrl` with your provider's actual endpoint and set the API key environment variable.

### Generic OpenAI-Compatible

Template for any endpoint that implements the OpenAI chat completions API.

```json
{
  "providers": {
    "custom": {
      "baseUrl": "https://api.example.com/v1",
      "api": "openai-completions",
      "apiKey": "YOUR_API_KEY",
      "models": [
        { "id": "model-name" }
      ]
    }
  }
}
```

Adapt `baseUrl`, `apiKey`, and model IDs to your endpoint. The `api` field should be `"openai-completions"` for chat completions or `"openai-responses"` for the responses API.

---

## Schema Reference

### Provider-Level Fields

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `baseUrl` | `string` | No | — | API endpoint URL (e.g. `http://localhost:11434/v1`) |
| `apiKey` | `string` | No | — | API key resolution string — env var name, `!command`, or literal (see [API Key Resolution](#api-key-resolution)) |
| `api` | `string` | No | — | API format: `"openai-completions"` or `"openai-responses"` |
| `headers` | `Record<string, string>` | No | — | Extra HTTP headers sent with every request. Values support the same resolution as `apiKey`. |
| `authHeader` | `boolean` | No | — | Whether to include the standard `Authorization: Bearer <key>` header |
| `models` | `ModelDefinition[]` | No | — | Array of model definitions for this provider |
| `modelOverrides` | `Record<string, ModelOverride>` | No | — | Override settings for built-in models (see [modelOverrides](#modeloverrides)) |

### Model-Level Fields (ModelDefinition)

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `id` | `string` | **Yes** | — | Model identifier (e.g. `"llama3.1:8b"`, `"gpt-4o"`) |
| `name` | `string` | No | — | Display name shown in the UI |
| `api` | `string` | No | Provider's `api` | Override the API format for this specific model |
| `baseUrl` | `string` | No | Provider's `baseUrl` | Override the endpoint for this specific model |
| `reasoning` | `boolean` | No | — | Whether this model supports reasoning/chain-of-thought |
| `input` | `("text" \| "image")[]` | No | — | Supported input modalities |
| `cost` | `object` | No | — | Token pricing: `{ input, output, cacheRead, cacheWrite }` (per-token costs) |
| `contextWindow` | `number` | No | — | Maximum context window size in tokens |
| `maxTokens` | `number` | No | — | Maximum output tokens per response |
| `headers` | `Record<string, string>` | No | — | Per-model HTTP headers (merged with provider headers) |
| `compat` | `object` | No | — | Compatibility flags for non-standard endpoints (see [Compatibility Flags](#advanced-compatibility-flags)) |

---

## API Key Resolution

The `apiKey` field (and header values) go through a resolution process that supports three formats:

### Environment Variable (recommended)

Set `apiKey` to the name of an environment variable:

```json
{
  "apiKey": "MY_API_KEY"
}
```

GSD checks if `MY_API_KEY` exists in the environment. If it does, that value is used. If not, the literal string `"MY_API_KEY"` is used as-is (which will likely fail authentication).

### Shell Command

Prefix with `!` to execute a shell command and use its stdout:

```json
{
  "apiKey": "!cat ~/.secrets/my-api-key"
}
```

```json
{
  "apiKey": "!security find-generic-password -s my-provider -w"
}
```

Command results are **cached for the process lifetime** — the command runs once at startup, not on every request. Commands time out after 10 seconds.

### Literal Value

Any string that doesn't match an environment variable is used as-is:

```json
{
  "apiKey": "ollama"
}
```

This is useful for local providers that don't need real authentication (Ollama, LM Studio).

### Dual Auth Path

GSD supports two ways to provide API keys:

1. **Wizard via `auth.json`** (preferred) — When you use `gsd config` and enter an API key, it's stored in `~/.gsd/agent/auth.json`. This keeps secrets out of `models.json`, which is safe to version-control or share.

2. **Manual via `models.json` `apiKey` field** — Set the `apiKey` field directly in `models.json`. Use an environment variable name or `!command` to avoid writing the literal secret into the file.

> **Security note:** Never put literal API keys in `models.json`. Use environment variable names or `!command` references instead. The wizard's `auth.json` approach handles this automatically.

---

## modelOverrides

Use `modelOverrides` to tweak settings for **built-in** models without fully redefining them. Overrides are merged with the built-in model definition.

Example: increase `maxTokens` for Claude Sonnet:

```json
{
  "providers": {
    "anthropic": {
      "modelOverrides": {
        "claude-sonnet-4-20250514": {
          "maxTokens": 16384
        }
      }
    }
  }
}
```

Example: add compatibility flags for a built-in model routed through a proxy:

```json
{
  "providers": {
    "my-proxy": {
      "baseUrl": "https://proxy.example.com/v1",
      "modelOverrides": {
        "gpt-4o": {
          "maxTokens": 4096,
          "compat": {
            "maxTokensField": "max_tokens"
          }
        }
      }
    }
  }
}
```

Override fields: `name`, `reasoning`, `input`, `cost` (partial — individual sub-fields are optional), `contextWindow`, `maxTokens`, `headers`, `compat`.

---

## Advanced: Compatibility Flags

The `compat` object handles differences between OpenAI-compatible endpoints. Most local providers work without any flags. Set these only if you encounter issues with a specific endpoint.

| Flag | Type | Default | Description |
|---|---|---|---|
| `supportsStore` | `boolean` | — | Endpoint supports the `store` parameter |
| `supportsDeveloperRole` | `boolean` | — | Endpoint supports the `developer` message role |
| `supportsReasoningEffort` | `boolean` | — | Endpoint supports `reasoning_effort` parameter |
| `supportsUsageInStreaming` | `boolean` | — | Endpoint returns usage stats in streaming responses |
| `maxTokensField` | `"max_completion_tokens"` \| `"max_tokens"` | — | Which field name the endpoint uses for max output tokens |
| `requiresToolResultName` | `boolean` | — | Tool result messages must include the `name` field |
| `requiresAssistantAfterToolResult` | `boolean` | — | An assistant message is required after tool result messages |
| `requiresThinkingAsText` | `boolean` | — | Thinking/reasoning output should be sent as text, not structured |
| `requiresMistralToolIds` | `boolean` | — | Use Mistral-style tool call IDs |
| `thinkingFormat` | `"openai"` \| `"zai"` \| `"qwen"` | — | How the endpoint formats thinking/reasoning output |
| `openRouterRouting` | `object` | — | OpenRouter-specific routing preferences: `{ only?: string[], order?: string[] }` |

Example for an endpoint that uses `max_tokens` and doesn't support the developer role:

```json
{
  "compat": {
    "maxTokensField": "max_tokens",
    "supportsDeveloperRole": false
  }
}
```

---

## Troubleshooting

### Startup Warnings

If `models.json` has errors, GSD prints a warning at startup and continues with built-in models only:

```
[gsd] Warning: models.json error — <detail>
[gsd] Custom provider models were not loaded. Built-in models are still available.
```

Common causes:

- **Invalid JSON** — syntax error in the file. Validate with `cat ~/.gsd/agent/models.json | python3 -m json.tool` or `jq . ~/.gsd/agent/models.json`.
- **Missing required fields** — every model in the `models` array must have an `id` field with a non-empty string.
- **Wrong API format** — `api` must be `"openai-completions"` or `"openai-responses"`. Typos or other values will cause validation errors.
- **Empty or malformed file** — the file must contain a valid JSON object with a `providers` key.

### Provider Not Showing Up

1. Verify `~/.gsd/agent/models.json` exists (not `~/.gsd/` root or `~/.pi/agent/`)
2. Check the JSON is valid
3. **Restart GSD** — changes are only picked up at startup
4. Check for startup warnings in the terminal output

### API Key Not Working

1. If using an env var: verify it's set in your shell (`echo $MY_API_KEY`)
2. If using a `!command`: run the command manually to verify it outputs the key
3. If using the wizard path: check `~/.gsd/agent/auth.json` has an entry for your provider name
4. Verify the key is valid with a direct curl to the endpoint

### Model Not Selectable

- The model `id` in `models.json` must match what the provider expects
- For Ollama: use the tag format (`llama3.1:8b`, not `meta-llama/Llama-3.1-8B`)
- For LM Studio: use the identifier shown in the LM Studio UI
- For vLLM: use the full HuggingFace model path (`meta-llama/Llama-3.1-8B`)

---

## Multiple Providers

You can define multiple providers in a single `models.json`:

```json
{
  "providers": {
    "ollama": {
      "baseUrl": "http://localhost:11434/v1",
      "api": "openai-completions",
      "apiKey": "ollama",
      "models": [
        { "id": "llama3.1:8b" },
        { "id": "codellama:13b" }
      ]
    },
    "cloud-backup": {
      "baseUrl": "https://api.example.com/v1",
      "api": "openai-completions",
      "apiKey": "CLOUD_API_KEY",
      "models": [
        { "id": "large-model", "contextWindow": 128000 }
      ]
    }
  }
}
```

All models from all providers appear in GSD's model selection. Use `/gsd prefs` to assign different models to different phases (research, planning, execution, completion).
