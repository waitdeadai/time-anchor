# time-anchor

[![tests](https://github.com/waitdeadai/time-anchor/actions/workflows/test.yml/badge.svg)](https://github.com/waitdeadai/time-anchor/actions/workflows/test.yml)
[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-hook-orange)](https://code.claude.com/docs/en/hooks)

> A Claude Code hook that injects the local system clock into every session and prompt, so the model stops giving "as of my training cutoff" answers and stops getting the year wrong.

`time-anchor` is one bash file (~127 lines, depends only on `python3`) wired into Claude Code's `SessionStart` and `UserPromptSubmit` events. On every session start and every user prompt, it injects an `additionalContext` block containing the local time, UTC time, timezone, date, hour, weekday, and a short set of temporal-reasoning rules.

The result: when you ask "what's the latest version of X?" or "is this product still available?" or anything time-sensitive, the model has the actual current date in its context, knows training-cutoff answers are not the right answer, and is steered toward live source verification.

## What it injects

```
temporal anchor from the local system clock.

local_time: 2026-05-11T02:29:08-03:00
utc_time: 2026-05-11T05:29:08+00:00
timezone: -03 (-0300)
local_date: 2026-05-11
local_hour: 02:29:08
weekday: Monday
source: local system clock via time-anchor.sh

Temporal reasoning rules:
1. Treat local_time as the current present date and hour for this Claude Code prompt.
2. Resolve "today", "yesterday", "tomorrow", "this week", "current", "latest", "recent", and year-specific claims relative to this anchor.
3. For SOTA 2026, current model/provider behavior, pricing, laws, docs, benchmarks, APIs, schedules, or news, do live source verification before making factual claims.
4. Cite source publish/update dates and the access date when research is time-sensitive.
5. If live verification is unavailable or incomplete, say the claim is stale, unverified, or insufficient_data instead of relying on pretrained memory.
```

You see this block as `<system-reminder>` context in the assistant's input on every prompt. The model treats it as ground truth.

## Why this matters

LLM training cutoffs make every time-sensitive answer wrong by default — often by months, sometimes by years. The model knows it has a cutoff but has no idea what *today* is unless something tells it. Without an anchor, you get answers like:

- "As of my last training cutoff in early 2025…"
- "The latest version of X is Y" (where Y was current 8 months ago)
- "This product was announced in 2024 and is expected to launch in early 2025" (where the launch happened months ago)

With an anchor, the model knows the real date, knows its memory is potentially stale, and is rule-pushed to verify via live sources when the claim is time-sensitive. This is the same fix Anthropic applies to Claude.ai's web UI, ported to Claude Code where it isn't shipped by default.

## Install (30 seconds)

```bash
mkdir -p .claude/hooks
curl -fsSL https://raw.githubusercontent.com/waitdeadai/time-anchor/main/time-anchor.sh \
  -o .claude/hooks/time-anchor.sh
chmod +x .claude/hooks/time-anchor.sh
```

Then merge the hook entries from [`settings.example.json`](settings.example.json) into your `.claude/settings.json`.

Requires `python3` (preinstalled on macOS, Ubuntu, and most Linux distros).

## What the script does

The script is invoked by Claude Code at `SessionStart` and `UserPromptSubmit`. It returns JSON shaped as:

```json
{
  "hookSpecificOutput": {
    "hookEventName": "SessionStart",
    "additionalContext": "temporal anchor from the local system clock.\n\nlocal_time: 2026-05-11T02:29:08-03:00\n…"
  }
}
```

Claude Code injects `additionalContext` into the assistant's input. The model sees the current date for the rest of the session.

You can also call the script directly for testing:

```bash
bash time-anchor.sh text     # human-readable output
bash time-anchor.sh json     # JSON payload
bash time-anchor.sh hook     # full hook response (default)
```

## What it does NOT do

- It does not call any external time API. The clock comes from `date` / Python's `datetime.now()` on your local machine. If your machine clock is wrong, the anchor is wrong.
- It does not push the model to verify *every* claim — only ones the temporal-reasoning rules flag as time-sensitive (current state, latest version, recent news, "as of" claims).
- It does not enforce verification. For that, see the sister tool [no-vibes](https://github.com/waitdeadai/no-vibes), which blocks closeouts that claim verification without evidence.

## Sister tools

Part of a small series of single-purpose Claude Code hooks extracted from the [minmaxing](https://github.com/waitdeadai/minmaxing) governance harness.

- [no-vibes](https://github.com/waitdeadai/no-vibes) — blocks Claude from claiming work is finished without verification evidence.
- [no-curfew](https://github.com/waitdeadai/no-curfew) — suppresses unsolicited rest/sleep/wellness paternalism.
- [no-sycophancy](https://github.com/waitdeadai/no-sycophancy) — blocks praise-spam at turn open.
- [no-cliffhanger](https://github.com/waitdeadai/no-cliffhanger) — blocks dangling permission-loop endings.
- [llm-dark-patterns](https://github.com/waitdeadai/llm-dark-patterns) — umbrella catalog of the suite.
- [minmaxing](https://github.com/waitdeadai/minmaxing) — the parent governance harness.

## License

Apache-2.0. See [LICENSE](LICENSE).
