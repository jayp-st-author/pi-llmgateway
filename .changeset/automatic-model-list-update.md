---
"@mcowger/pi-llmgateway": minor
---

Refresh the model list from the live LLM Gateway `/v1/models` catalog on **every** `session_start` instead of only the first one in a process. This makes Pi's model list fully dynamic: any model the gateway (including DevPass) adds or removes is reflected on the next session, while the on-disk cache plus the static snapshot keep models available immediately as a stale-while-revalidate seed.

- Refresh the live catalog on every `session_start` (was once per process via a `modelsLoaded` guard)
- Wrap the refresh in try/catch so a transient fetch/build failure gracefully keeps the current seed instead of silently skipping the refresh
- Fix an abort-check bug: the post-fetch guard re-read a module-scoped `fetchAbort` that a newer `session_start` would have reassigned; it now checks a local `controller`
- Regenerate the static snapshot from the live API (182 → 193 chat models), adding 38 missing DevPass models (e.g. `claude-opus-5`, `claude-sonnet-5`, `claude-fable-5`, `kimi-k3`/`kimi-k3-fast`, `gpt-5.6-luna/sol/terra`, `grok-4-5`, `hermes-4-70b/405b`, `nemotron-3-super-120b`, `qwen3.8-max`, `cosmos3-super-reasoner`, `fugu-ultra`)