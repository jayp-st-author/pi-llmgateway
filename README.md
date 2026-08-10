# Pi LLM Gateway Extension

A Pi extension that adds [LLM Gateway](https://llmgateway.io) as a model provider, giving you access to 160+ chat models from OpenAI, Anthropic, Google, xAI, Mistral, DeepSeek, and many others through a single OpenAI-compatible API.

> [!NOTE]
> This fork updates the extension for LLM Gateway DevPass subscribers. It refreshes the live model catalog whenever a Pi session starts, so models added to or removed from a DevPass subscription appear automatically without reinstalling the extension. It also tracks the `@earendil-works` Pi packages and carries the compatibility updates for Pi 0.84.1. The original extension is maintained at [mcowger/pi-llmgateway](https://github.com/mcowger/pi-llmgateway).

## Installation

### Get API Key

Sign up at [llmgateway.io](https://llmgateway.io) to get an API key.

### Configure Credentials

Add your API key to `~/.pi/agent/auth.json` (recommended):

```json
{
  "llmgateway": { "type": "api_key", "key": "your-api-key-here" }
}
```

Or set an environment variable:

```bash
export LLMGATEWAY_API_KEY="your-api-key-here"
```

### Install This Fork

```bash
# From git
pi install git:github.com/jayp-st-author/pi-llmgateway

# Local development
pi -e ./src/extensions/provider/index.ts
```

To use the upstream npm release instead:

```bash
pi install npm:@mcowger/pi-llmgateway
```

## Usage

Select `llmgateway` as your provider and choose from available models:

```
/model llmgateway gpt-5
/model llmgateway claude-opus-4-8
/model llmgateway gemini-2.5-pro
/model llmgateway auto
```

The special `auto` model lets the gateway pick the best provider and model for each request based on the configured routing strategy.

## Automatic DevPass Model Updates

With a DevPass API key configured, the extension refreshes LLM Gateway's live `/v1/models` catalog on every `session_start`. New DevPass models become available in Pi on the next session, and models removed from the subscription disappear from the list automatically.

The most recently fetched catalog is cached locally and used immediately at startup. If the live refresh fails, the extension keeps the cached catalog—or its bundled static snapshot—so a temporary network or gateway failure does not empty the model picker.

## Settings

Configure with `/llmgateway:settings`:

| Setting | Values | Default | Description |
|---|---|---|---|
| `routing` | `auto`, `price`, `throughput`, `latency` | `auto` | Provider routing strategy. Note: coding (dev) plans only support `auto` and `price`. |
| `webSearch` | `enabled`, `disabled` | `disabled` | Enable native web search for models that support it. |
| `baseUrl` | URL string | `https://api.llmgateway.io/v1` | API base URL for self-hosted instances. |
| `includeDeactivated` | `include`, `ignore` | `ignore` | Show deactivated models (may stop working at any time). |

Settings can also be changed with `pi config`.

## Compat values

The extension sends all requests in standard OpenAI Chat Completions format. The LLM Gateway normalises everything server-side:

- `max_tokens` is used for all models (the gateway translates to `max_completion_tokens` for GPT-5/o-series internally)
- `developer` role is not used (`system` role is sent for all models)
- Reasoning is controlled via top-level `reasoning_effort` (`none|minimal|low|medium|high|xhigh|max`)
- `reasoning_content` on assistant messages is not required (the gateway reconstructs it for upstreams that need it)
- Prompt caching is managed by the gateway

## Self-hosted instances

Change the base URL in settings to point to your own LLM Gateway deployment:

```
/llmgateway:settings → Connection → Base URL → https://your-gateway.example.com/v1
```

Restart Pi after changing the URL.

## Updating the Model Snapshot

Models are seeded from a hardcoded snapshot (`src/extensions/provider/models/static.ts`) and refreshed from the live `/v1/models` API on each session start. To update the snapshot (e.g. after significant model changes):

1. Run `bun run test` — it fetches the live API and reports drift
2. Regenerate the snapshot by running the generator script
3. Re-run `bun run test` to confirm no drift

## Development

```bash
git clone https://github.com/jayp-st-author/pi-llmgateway.git
cd pi-llmgateway
bun install
```

### Commands

```bash
bun run typecheck   # TypeScript type check
bun run lint        # Biome lint
bun run format      # Biome format (auto-fix)
bun run test        # Vitest (includes live API drift check)
bun run gen:schema  # Regenerate schema.json from config.ts
```

## Requirements

- `@earendil-works/pi-coding-agent` 0.84.x (tested with 0.84.1)
- LLM Gateway API key (configured in `~/.pi/agent/auth.json` or via `LLMGATEWAY_API_KEY`)

## Links

- [LLM Gateway](https://llmgateway.io)
- [LLM Gateway Docs](https://docs.llmgateway.io)
- [LLM Gateway GitHub](https://github.com/theopenco/llmgateway)
- [Pi Documentation](https://buildwithpi.ai/)
- [Upstream pi-llmgateway](https://github.com/mcowger/pi-llmgateway)
