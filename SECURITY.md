# Security policy

## Reporting a vulnerability

Please do not report security problems in public issues.

Use GitHub's private vulnerability reporting: open the **Security** tab of this repository and choose **Report a vulnerability**. Include:

- what the problem is and which file or tool it affects;
- steps to reproduce;
- what an attacker could do with it.

You should get a reply within a few days. Please give us reasonable time to fix the problem before you disclose it.

## Do not share secrets

Never paste any of the following into issues, pull requests or discussions:

- ChatGPT or OpenAI tokens, API keys, cookies or passwords;
- the contents of `state/`, `jobs/` or `images/`;
- chat logs, Codex threads or tool output that contain personal data.

If you shared a secret by mistake, revoke it right away, then edit or delete the post.

## Scope

gpt-bridge runs Codex with full access to your Mac in `gpt_agent` and `gpt_computer_use`. Reports about ways to bypass the bridge's own guards (`BLOCKED_RPC`, `ALLOWED_CLI`, `BLOCKED_TOOL_PREFIXES`, `BLOCKED_APPS`, the kill switch, rate limits) are in scope. Problems in Codex, ChatGPT.app or Claude Code themselves should be reported to their vendors.
