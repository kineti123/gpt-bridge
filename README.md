# gpt-bridge

English | [Русский](README.ru.md)

An MCP server that lets Claude Code hand tasks to Codex (OpenAI GPT models) through your ChatGPT subscription on macOS.

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Platform: macOS](https://img.shields.io/badge/platform-macOS-lightgrey.svg)](#requirements)
[![Python 3](https://img.shields.io/badge/python-3-3776AB.svg)](#requirements)

> **Disclaimer.** This project is not affiliated with, endorsed by, or supported by OpenAI or Anthropic. It starts the Codex CLI that ships inside ChatGPT.app and talks to it through the `codex app-server` protocol. All usage counts against your own ChatGPT plan. Read OpenAI's terms of use before you use it. You are responsible for how you use it.

## Why

Claude Code and Codex are good at different things. If you pay for ChatGPT, you already have Codex usage included in your plan. gpt-bridge lets Claude Code use that usage as a helper: it can ask Codex a question, give it a long agent task, have it read images, generate images, or review code. No OpenAI API key is needed.

## How it works

```
Claude Code ──MCP (stdio)──> server.py ──JSON-RPC (stdio)──> codex app-server ──> your ChatGPT account
```

- `server.py` is a stdio MCP server. It uses only the Python standard library.
- For each tool call it starts one `codex app-server` process and stops it when the call ends. There is no daemon and no background polling.
- Login, tokens and network traffic to OpenAI are handled by the Codex CLI itself. The bridge never reads or stores your credentials.
- Background jobs (`background: true`) run as a separate `server.py --job <id>` process. It exits when the job ends and shows a macOS notification.

## Requirements

- macOS. The code uses `fcntl`, `osascript` and the ChatGPT.app path.
- ChatGPT.app, signed in, with the bundled Codex CLI at `/Applications/ChatGPT.app/Contents/Resources/codex-cli/bin/codex`. You can point to another `codex` binary with the `GPT_BRIDGE_CODEX` environment variable, or put `codex` on your `PATH`.
- Python 3.
- Claude Code.

## Quick start

1. Clone the repository:
   ```bash
   git clone https://github.com/<owner>/gpt-bridge.git /path/to/gpt-bridge
   ```
2. Run the self-test. It checks the code offline, then starts `codex app-server` once to read your plan. It does not spend model usage.
   ```bash
   python3 /path/to/gpt-bridge/server.py --selftest
   ```
3. Check your plan, usage and available models:
   ```bash
   python3 /path/to/gpt-bridge/server.py --call gpt_status '{}'
   ```
4. Register the server in Claude Code:
   ```bash
   claude mcp add gpt-bridge -s user -- python3 /path/to/gpt-bridge/server.py
   ```
5. Optional: add routing rules, so Claude Code knows when to use Codex:
   ```bash
   mkdir -p /path/to/gpt-bridge/state && cp /path/to/gpt-bridge/examples/rules.md /path/to/gpt-bridge/state/rules.md
   ```

Then ask Claude Code in plain words, for example "ask Codex to review this diff".

The model names in `DEFAULTS` (`gpt-6.1-terra`, `gpt-6.1-sol`) are the ones the author's account offers. Run `gpt_status` and change them in `server.py` if your account lists different models.

## Claude Code skills

The MCP server gives Claude Code the tools. The skills in `skills/` teach it how to use them well. Claude Code loads a skill only when the task matches its description.

| Skill | What it covers |
|---|---|
| `codex-route` | When to hand work to Codex and when to do it yourself, which tool and model fit each task, the brief format, how to accept the result, what to do when Codex is paused. |
| `codex-helpers` | Bringing Codex in on Claude's own initiative: fresh web results, images, second opinions, sites Claude cannot open, and splitting big research jobs between Claude agents and Codex. |

Install them for your user:

```bash
mkdir -p ~/.claude/skills && cp -R /path/to/gpt-bridge/skills/* ~/.claude/skills/
```

Optional: load the routing rules at the start of every session, not only when the skill triggers. Merge `examples/session-start-hook.json` into `~/.claude/settings.json`. The hook prints the body of `codex-route/SKILL.md` into the session context.

The skills, `examples/rules.md` and the hook overlap on purpose. `rules.md` is short and always loaded as server instructions. The skills hold the details. Edit all of them to match your own workflow and the model names your account offers.

## Tools

| Tool | What it does | Uses model quota |
|---|---|---|
| `gpt_ask` | Light question to Codex: no plugins, read-only. Accepts local images. Optional web search. | yes |
| `gpt_agent` | Full Codex agent in a task folder (`cwd` is required): files, commands, plugins, sub-agents. Full access by default. | yes |
| `gpt_computer_use` | Codex operates apps on this Mac (OpenAI Computer Use). The screen must be unlocked. | yes |
| `gpt_search` | Web search or open one page. Returns sources with URLs and short excerpts. | yes |
| `gpt_image` | Generate or edit images. Files are copied to `out_dir` (default `images/`). | yes |
| `gpt_review` | Code review of uncommitted changes, or of the diff against a `base` branch. | yes |
| `gpt_connector_list` | List connector and MCP tools on the ChatGPT side. | no |
| `gpt_connector_call` | Call one connector tool directly, without a model. | no |
| `gpt_apps` | Connected ChatGPT apps, or search the plugin catalog with `query`. | no |
| `gpt_plugin_install` | Install a Codex plugin (`codex plugin add`). | no |
| `gpt_threads` | List, search or read Codex threads. | no |
| `gpt_jobs` | List background jobs, read a result, or cancel a job. | no |
| `codex_rpc` | Call any `app-server` method that is not blocked (see below). | no, except `thread/compact/start` |
| `codex_cli` | Run an allowed Codex CLI subcommand: `apply`, `archive`, `cloud`, `debug`, `doctor`, `features`, `mcp`, `plugin`, `queue`, `unarchive`. | depends on the command |
| `gpt_status` | Plan, usage, reset times, mode, defaults, today's journal, models. | no |

Common parameters:

- Tools that use model quota accept `model`, `effort` and `strict_model`.
- `gpt_ask`, `gpt_agent`, `gpt_computer_use`, `gpt_image` and `gpt_review` accept `background: true`. The call returns a `job_id` at once. Read the result with `gpt_jobs`.
- `thread_id` continues an existing thread, including one started in the ChatGPT app.
- Answers longer than `max_chars` (default 6000) return only the beginning. The full text is saved to `state/results/`.

Default model and effort per tool (`DEFAULTS` in `server.py`):

| Tool | Model | Effort |
|---|---|---|
| `gpt_ask`, `gpt_search`, `gpt_image` | `gpt-6.1-terra` | medium |
| `gpt_agent`, `gpt_computer_use` | `gpt-6.1-terra` | high |
| `gpt_review` | `gpt-6.1-sol` | high |

## Routing and limits

Claude Code decides what to send to Codex. The bridge adds guard rails:

- **Normal mode.** The requested or default model and effort are used.
- **Economy mode.** From 70% usage in the 5-hour window or 60% in the weekly window: Sol and Astra models become Terra, and effort drops one step (`ultra`, `max`, `xhigh` to `high`; `high` to `medium`).
- **Stop mode.** From 90% (5 hours) or 85% (week): every model except Luna becomes Terra, and effort becomes `medium` unless it is `low`.
- **`strict_model: true`** skips economy and stop adjustments for that call.
- **Kill switch.** While the file `state/codex_off` exists, every tool that uses model quota refuses to run. Delete the file to turn it back on.
- **Journal.** Each model call adds one line to `state/delegations.jsonl`: time, tool, model, effort, mode, thread ID, status, duration, token counts, estimated credits, usage before and after, and answer length. Prompts and answers are not written to the journal.
- **Rate limits** (`LIMITS`, `MAX_PARALLEL`), shared by all bridge processes: at most 12 Codex starts per minute, 4 Codex processes at once, and 60 direct connector calls per hour.
- **Exhausted limit.** When a usage window reaches 100% (or the server says the limit is reached, or a turn fails with a usage-limit error), the bridge remembers the reset time and refuses model tools at once, without starting Codex. It turns itself back on after the reset. `gpt_status`, `gpt_jobs` and `gpt_threads` keep working. If the window is not named or the reset time is unknown, the bridge pauses for 30 minutes and checks again. Credits on the account mean the limit is not exhausted. Clear the block by hand with `python3 server.py --unblock`.
- **Cooldown.** If an error mentions Cloudflare, HTTP 429, "too many requests" or "unusual activity", the bridge pauses for 15 minutes. Delete `state/limits.json` to clear the pause early.
- Inside Codex runs started by the bridge, the bridge's own MCP server is disabled, so Codex cannot call Codex in a loop.

The thresholds are the bridge's own numbers, not OpenAI's. Credit estimates use a fixed price table in `CREDITS` and may not match your actual billing.

## Configuration files in `state/`

The bridge creates `state/` next to `server.py` as needed. It is listed in `.gitignore` and must never be committed.

| File | Purpose |
|---|---|
| `state/rules.md` | Optional. Routing rules sent to Claude Code as MCP server instructions. Example: `examples/rules.md`. |
| `state/policy.json` | Optional. Overrides thresholds, for example `{"economy_5h": 70, "economy_week": 60, "stop_5h": 90, "stop_week": 85}`. |
| `state/codex_off` | Optional. Kill switch: create an empty file to stop model calls. |
| `state/delegations.jsonl` | Journal of model calls. |
| `state/results/` | Full text of long answers. |
| `state/limits.json`, `*.lock` | Shared rate-limit state, the remembered exhausted-limit time (`exhausted_until`) and file locks. |

Two more folders are created next to `server.py` and are also ignored by git: `jobs/` (background job files, including their arguments and results) and `images/` (default output folder for `gpt_image`).

## Security and privacy

What runs locally:

- `server.py` and `codex app-server` run on your Mac. The bridge sends data only to the local Codex process. Codex then talks to OpenAI as it normally does.

What the bridge does not do:

- It does not read, copy or store ChatGPT tokens, passwords or API keys.
- It blocks `app-server` methods that change login, spend money, publish data or delete threads (`BLOCKED_RPC`): `account/login`, `account/logout`, `account/gatewayOAuth`, `account/rateLimitResetCredit`, `account/sendAddCreditsNudgeEmail`, `app/list`, `feedback/upload`, `plugin/share`, `thread/delete`. Turns are started only through the dedicated tools.
- It blocks connector tools under `safety_settings.` (parental controls).
- `codex_cli` allows only the subcommands listed above. `exec` and `login`/`logout` are not allowed.
- It turns off Codex memory generation for bridge runs, and disables its own MCP server (`gpt-bridge`) inside them. Add entries to `BASE_CONFIG` in `server.py` to keep other MCP servers out of bridge runs.

What you should know:

- **`gpt_agent` and `gpt_computer_use` run with full access** (`danger-full-access` sandbox) and approve connector requests automatically. Codex can read and change any file your user can, run commands, and operate apps. Use `sandbox: "read-only"` or `"workspace-write"` with `gpt_agent` when you do not need full access.
- `BLOCKED_APPS` is empty by default, so Computer Use may operate any app. Add app names to block them, for example `("keychain", "1password")`.
- The model is told not to follow instructions found in images, web pages or files. This lowers the risk of prompt injection but does not remove it.
- `state/`, `jobs/` and `images/` can contain your prompts, answers and generated files. Keep them out of git, and do not paste them into issues.

## Known limitations

- macOS only.
- The `codex app-server` protocol is experimental and may change with a ChatGPT update. To see the current schema:
  ```bash
  codex app-server generate-json-schema --out /path/to/codex-schema
  ```
- `gpt_image` needs a paid plan. On the Free plan it returns without calling the model (use `force: true` to try anyway).
- `gpt_computer_use` needs an unlocked screen and the permissions macOS asks for.
- Codex may ask a follow-up question. The bridge cannot answer it in the same call: reply with a new call that uses the same `thread_id`.
- Credit numbers in the journal and in `gpt_status` are estimates.

## Troubleshooting

| Message or symptom | What to do |
|---|---|
| `Codex not found` | Install ChatGPT.app in `/Applications`, put `codex` on `PATH`, or set `GPT_BRIDGE_CODEX`. |
| Bridge is paused until a time | OpenAI or Cloudflare refused a request. Wait, or delete `state/limits.json`. |
| `ChatGPT usage limit is used up until ...` | Your plan's usage window is exhausted. The bridge turns itself back on after the reset. To clear it early, run `gpt_status` (it clears the block if the window really reset) or `python3 server.py --unblock`. |
| Bridge rate limit reached | More than 12 Codex starts per minute or 60 connector calls per hour. Wait and retry. |
| 4 Codex processes already running | Wait, or cancel a background job with `gpt_jobs` and `cancel: true`. |
| Codex is switched off | Delete `state/codex_off`. |
| `gpt_agent` asks for `cwd` | Pass an existing folder as `cwd`. |
| A turn timed out | Continue with the same `thread_id`. Raise `timeout_sec` for long tasks. |
| `gpt_search` stopped working | Check that `thread/start` still accepts `ephemeral` and that `web_search="live"` still enables search in your Codex version. |

## FAQ

**Do I need an OpenAI API key?**
No. The bridge uses the Codex CLI from ChatGPT.app, which uses your ChatGPT sign-in.

**Does it cost extra money?**
No extra payment is made by the bridge. Model calls use your ChatGPT plan's usage limits. Paid `app-server` methods are blocked.

**Does it work on Linux or Windows?**
Not as is. It depends on `fcntl`, `osascript` and the ChatGPT.app path.

**Can I use it without Claude Code?**
Yes, for single calls: `python3 server.py --call <tool> '<json>'`. It is also a standard stdio MCP server, so other MCP clients may work, but only Claude Code was tested.

**Where are my conversations stored?**
Codex threads are stored by Codex and your ChatGPT account, as usual. The bridge keeps only the local files listed in the `state/` section.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). To report a security problem, see [SECURITY.md](SECURITY.md).

## License

MIT. See [LICENSE](LICENSE).
