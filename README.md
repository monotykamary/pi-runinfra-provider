<div align="center">

# 🏃 pi-runinfra-provider

**DeepSeek V4, Nemotron 3.5 Lightning & Qwen3.8 through [RunInfra](https://runinfra.ai/)**

_A [pi](https://github.com/earendil-works/pi-coding-agent) provider extension for RunInfra's OpenAI-compatible inference gateway._

[![pi extension](https://img.shields.io/badge/pi-extension-blueviolet)](https://github.com/earendil-works/pi-coding-agent)
[![license](https://img.shields.io/badge/license-MIT-blue)](./LICENSE)

</div>

---

## Features

- **10 AI Models** covering the full RunInfra lineup: DeepSeek V4 Flash, DeepSeek V4 Pro, Nemotron 3.5 Lightning 30B, Ornith 1.5 35B, Qwen3.8 2.4T A95B, Qwen3.8 27B, Parakeet TDT 0.6B v3, and the Qwen3 embedding/reranker models
- **OpenAI-Compatible API** — Just change the base URL and API key
- **Cost Tracking** — Per-model pricing for budget management
- **Reasoning Models** — Extended thinking via `reasoning_content` field
- **Reasoning Effort** — Control reasoning depth on DeepSeek V4 (including `max`) and other reasoning models
- **Request Tracing** — Automatic `X-Client-Request-Id` UUID header on every request for server-side observability

## Installation

### Option 1: Using `pi install` (Recommended)

Install directly from GitHub:

```bash
pi install https://github.com/monotykamary/pi-runinfra-provider
```

Then set your API key and run pi:
```bash
# Recommended: add to auth.json
# See Authentication section below

# Or set as environment variable
export RUNINFRA_GATEWAY_KEY=your-api-key-here

pi
```

### Option 2: Manual Clone

1. Clone this repository:
   ```bash
   git clone https://github.com/monotykamary/pi-runinfra-provider.git
   cd pi-runinfra-provider
   ```

2. Set your RunInfra gateway key:
   ```bash
   # Recommended: add to auth.json
   # See Authentication section below

   # Or set as environment variable
   export RUNINFRA_GATEWAY_KEY=your-api-key-here
   ```

3. Run pi with the extension:
   ```bash
   pi -e /path/to/pi-runinfra-provider
   ```

## Available Models

| Model | Context | Vision | Reasoning | Input $/M | Cache Read $/M | Output $/M |
|-------|---------|--------|-----------|-----------|-----------------|------------|
| DeepSeek V4 Flash | 1.0M | ❌ | ✅ | $0.13 | $0.01 | $0.27 |
| DeepSeek V4 Pro | 1.0M | ❌ | ✅ | $0.60 | $0.03 | $1.90 |
| GLM 5.3 Flash | 1.0M | ✅ | ✅ | $0.10 | $0.01 | $0.40 |
| Nemotron 3.5 Lightning 30B | 262K | ❌ | ✅ | $0.05 | $0.01 | $0.15 |
| Ornith 1.5 35B | 262K | ✅ | ✅ | $0.10 | $0.01 | $0.40 |
| Parakeet TDT 0.6B v3 | 131K | ❌ | ❌ | $10.00 | — | — |
| Qwen3 8 Flash Next | 1.0M | ❌ | ❌ | $0.12 | $0.01 | $0.40 |
| Qwen3 Embedding 0.6B | 33K | ❌ | ❌ | $0.01 | — | — |
| Qwen3 Embedding 8B | 33K | ❌ | ❌ | $0.05 | — | — |
| Qwen3 Reranker 8B | 16K | ❌ | ❌ | $0.05 | — | — |
| Qwen3.8 2.4T A95B | 1.0M | ❌ | ✅ | $2.00 | $0.20 | $6.00 |
| Qwen3.8 27B | 262K | ❌ | ✅ | $0.10 | $0.01 | $0.40 |

*Costs are per million tokens. Prices subject to change — check RunInfra for current pricing.*

## Usage

After loading the extension, use the `/model` command in pi to select your preferred model:

```
/model runinfra deepseek-v4-flash
```

Or start pi directly with a RunInfra model:

```bash
pi --provider runinfra --model deepseek-v4-flash
```

## Authentication

The RunInfra gateway key can be configured in multiple ways (resolved in this order):

1. **`auth.json`** (recommended) — Add to `~/.pi/agent/auth.json`:
   ```json
   { "runinfra": { "type": "api_key", "key": "your-api-key" } }
   ```
   The `key` field supports literal values, env var names, and shell commands (prefix with `!`). See [pi's auth file docs](https://github.com/badlogic/pi-mono) for details.
2. **Runtime override** — Use the `--api-key` CLI flag
3. **Environment variable** — Set `RUNINFRA_GATEWAY_KEY`

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `RUNINFRA_GATEWAY_KEY` | No | Your RunInfra gateway key (fallback if not in auth.json) |

## Configuration

Add to your pi configuration for automatic loading:

```json
{
  "extensions": [
    "/path/to/pi-runinfra-provider"
  ]
}
```

### Compat Settings

RunInfra's API follows the OpenAI Chat Completions API:

- **`supportsDeveloperRole: false`** — All models. RunInfra uses standard OpenAI roles.
- **`maxTokensField: "max_tokens"`** — All models. RunInfra accepts `max_tokens`.
- **`thinkingFormat: "deepseek"`** — DeepSeek V4 Flash / V4 Pro. Reasoning is toggled via a `thinking` parameter and effort is sent as `reasoning_effort` (pi maps Shift+Tab levels to `low`/`high`/`max`).
- **`thinkingFormat: "openai"`** — Nemotron 3.5 Lightning, Ornith 1.5 35B, and Qwen3.8. Returns thinking in the `reasoning_content` field.
- **`supportsReasoningEffort: true`** — All reasoning models. Supports the `reasoning_effort` parameter.
- **`supportsStore: false`** — All models. RunInfra doesn't support the `store` parameter.

### Request Tracing

Every request to RunInfra automatically includes a fresh `X-Client-Request-Id` header containing a UUID. This matches RunInfra's recommended tracing header and makes it easy to correlate client sessions with server-side logs.

### Patch Overrides

The `patch.json` file contains overrides that are applied on top of `models.json` data. This is useful for:
- Marking models as reasoning-capable when the API doesn't report it
- Filling in pricing for models where the API returns empty values
- Adding compat settings (thinking format, reasoning effort, level maps)
- Setting `thinkingFormat: "deepseek"` and `thinkingLevelMap` for the DeepSeek V4 models

### Custom Models

The `custom-models.json` file contains full model definitions for models that need manual curation beyond what the API provides, or models not yet available from the API. These are merged in after patch application, taking precedence for matching IDs.

Merge order: `[live|cache|embedded] → patch.json → custom-models.json`

## Updating Models

Run the update script to fetch the latest models from RunInfra's API:

```bash
export RUNINFRA_GATEWAY_KEY=your-api-key
node scripts/update-models.js
```

This will:
1. Fetch models from `https://api.runinfra.ai/v1/models`
2. Preserve existing model data (pricing, compat) for known models
3. Apply overrides from `patch.json`
4. Update `models.json` and the README model table

To regenerate just the README model table from local data — no API key needed,
useful for offline curation:

```bash
node scripts/update-models.js --readme-only
```

## License

MIT
