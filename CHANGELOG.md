# Changelog

## 1.3.2

Fix: Claude sometimes received only a fraction of a Codex answer.

- The answer limit is removed completely. `gpt_ask`, `gpt_agent`, `gpt_computer_use` and `gpt_search` always return the whole text. Before, an answer longer than `max_chars` (6000) was cut to its first 2500 characters, so a 12000-character answer arrived as about a fifth.
- No tool truncates its output any more. The `max_chars` parameter is removed everywhere (including `codex_rpc`, `codex_cli` and `codex_connector`), and `state/results/` is no longer written.

## 1.3.1

Refinements to the exhausted-limit memory, from a Codex review of 1.3.0.

- A server refusal at an incomplete percentage (for example 96%): the wait time comes from the refusal text ("try again in 3 hours"). Without it, the nearest reset of a window that is at least 90% full is used. Without that, 30 minutes.
- Such a block is not lifted by an identical reading. It is lifted only when the window really reset (the reset time moved forward or the percentage dropped by 20 points) or credits appeared.
- When the server names the window (`rateLimitReachedType` is `primary` or `secondary`), the bridge waits for that window's reset. If the window is not named or the time is unknown, it pauses for 30 minutes and checks again.
- A window at 100% with no reset time pauses for 30 minutes instead of being treated as "limit available".
- First detection: if a fresh reading at the start of a turn shows exhaustion, the turn is not started.
- Review: a limit refusal at the end of `gpt_review` is remembered like in a normal turn, and the error text is returned in the result.
- Manual unblock: `python3 server.py --unblock`.
- The phrase "limit reached" without "usage limit" no longer counts as exhaustion (a context-length error looks like that).
- Self-test covers all of the cases above.

## 1.3.0

- **Memory of an exhausted limit.** When the ChatGPT usage limit runs out, the bridge remembers the reset time in `state/limits.json` (`exhausted_until`) and refuses model tools at once, without starting Codex. It turns itself back on after the reset.
- A limit counts as exhausted when a window (5 hours or week) reaches 100%, the server reports it (`rateLimitReachedType`, `spendControlReached`), or a turn fails with a usage-limit error. If the account has credits, the limit is not treated as exhausted.
- `gpt_status`, `gpt_jobs` and `gpt_threads` keep working while blocked. `gpt_status` shows the block and clears it if the window really reset.
- The routing rules in `examples/rules.md` and the skill `codex-route` tell Claude Code not to call Codex while the limit is exhausted.
- Claude Code skills `codex-route` and `codex-helpers` in `skills/`.
- Example SessionStart hook in `examples/session-start-hook.json`.

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
