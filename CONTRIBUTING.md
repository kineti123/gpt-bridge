# Contributing

Thanks for your interest in gpt-bridge.

## Before you start

- Open an issue first for anything larger than a small fix, so we can agree on the approach.
- Keep the project dependency-free: Python standard library only.
- Keep behavior changes and refactors in separate pull requests.

## Making a change

1. Fork the repository and create a branch.
2. Make your change. Keep code comments, tool descriptions and messages in English.
3. Run the checks:
   ```bash
   python3 -m py_compile server.py
   python3 server.py --selftest
   ```
   The self-test needs ChatGPT.app installed and signed in. It does not spend model usage.
4. Update `README.md` and `CHANGELOG.md` if the change affects users.
5. Open a pull request and describe what changed and how you tested it.

## Privacy

Never commit or paste the contents of `state/`, `jobs/` or `images/`, chat logs, tokens, API keys or personal paths. Check your diff before you push.

## Code of conduct

Be respectful and constructive. Assume good intent.
