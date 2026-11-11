# Changelog

## 1.2.0

- Default model and effort for every tool (`DEFAULTS`). Without them Codex would use its own default from `~/.codex/config.toml`.
- Economy and stop modes based on the ChatGPT usage limits (`state/policy.json`).
- Kill switch `state/codex_off`.
- Journal `state/delegations.jsonl` with tokens, estimated credits and limits.
- Long answers are saved to `state/results/`; the reply keeps the beginning.
- Routing rules are read from `state/rules.md` (example in `examples/rules.md`).

## 1.1.0

- `gpt_search` runs as a normal model turn with web search, in a one-off thread.
- `gpt_apps` with `query` searches the official plugin catalog (`plugin/list`).
- The bridge identifies itself as `Claude Code`, the same as the official OpenAI plugin.
- Frequency limits for all bridge processes together, and a 15-minute pause after an OpenAI or Cloudflare refusal.
- The bridge is switched off inside Codex runs started by the bridge.

## 1.0.0

- First version: MCP server over `codex app-server`.
