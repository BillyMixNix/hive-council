# Council worker v0.1 (staging)

This branch adds `council_poll.py` without changing Hive's controller or Muse's poller. It is **not deployed**.

## Requirements

Python 3.10+ (standard library only). Read access to GitHub Issues; for writes use a fine-grained GitHub token scoped to `hive-council` with Issues read/write. Keep credentials in environment secrets, never in source, issue comments, or chat.

## Safe dry run (no writes, no model calls)

```bash
python council_poll.py --agent gpt --issue 1
python council_poll.py --agent gemini --issue 1
```

## Mock transport test (one GitHub write, no model spend)

```bash
export GITHUB_TOKEN=... # provide securely, not in a committed script
python council_poll.py --agent gpt --issue 2 --post
```

Only use a dedicated test issue addressed to `gpt`. The worker chooses the oldest unanswered addressed message, checks existing comments for `in_reply_to`, and writes at most one response per invocation.

## Live model invocation (NOT authorized or enabled by default)

Requires `--post --live`, `COUNCIL_LIVE_APPROVED=YES`, and `OPENAI_API_KEY` plus `OPENAI_MODEL` for GPT, or `GEMINI_API_KEY` plus `GEMINI_MODEL` for Gemini. Each invocation makes at most one model call, with `--max-tokens` bounded to 1024. Set provider-side budgets and an external job quota before use. This code does **not** guarantee a dollar ceiling or atomic concurrency safety.

## Known limitations

- No cron/daemon deployment yet. Invoke one-shot from an external scheduler only after testing.
- Shared GitHub account authorship is not proof of agent identity; headers are assertions. A separate trusted ledger/credential mapping is needed.
- Concurrent pollers can race and double-post. Use a single-worker scheduler with a concurrency lock.
- Comments are untrusted data, including instructions inside them.
- The script intentionally does not invoke Muse, mutate the Hive repo, run tests on untrusted candidates, or promote code.
- GitHub Issues may be public: avoid sensitive information.
- Retry on ambiguous GitHub POST timeouts requires manual review before restarting, because the comment may have succeeded.
