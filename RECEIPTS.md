# Receipts

## Reproducible local tests

Setup:

```bash
git clone https://github.com/waitdeadai/time-anchor
cd time-anchor
```

### Test 1 — `text` mode produces a human-readable anchor

```bash
bash time-anchor.sh text
```

Expected output (your local time will differ):

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
…
```

### Test 2 — `json` mode produces a structured payload

```bash
bash time-anchor.sh json
```

Expected (sample):

```json
{
  "source": "local system clock via time-anchor.sh",
  "local_time": "2026-05-11T02:29:08-03:00",
  "utc_time": "2026-05-11T05:29:08+00:00",
  "epoch_seconds": 1747284548,
  "timezone_name": "-03",
  "timezone_offset": "-0300",
  "local_date": "2026-05-11",
  "local_hour": "02:29:08",
  "weekday": "Monday",
  "project_dir": "/path/to/cwd",
  "hook_event_name": "SessionStart"
}
```

### Test 3 — `hook` mode returns Claude-Code-shaped JSON

```bash
echo '{"hook_event_name":"SessionStart"}' | bash time-anchor.sh hook
```

Expected (single-line JSON, formatted here for readability):

```json
{
  "hookSpecificOutput": {
    "hookEventName": "SessionStart",
    "additionalContext": "temporal anchor from the local system clock.\n\nlocal_time: …"
  }
}
```

### Test 4 — UserPromptSubmit event passes through

```bash
echo '{"hook_event_name":"UserPromptSubmit"}' | bash time-anchor.sh hook
```

Expected: `hookEventName` in the output is `UserPromptSubmit`.

### Test 5 — invalid command returns non-zero exit

```bash
bash time-anchor.sh nonsense; echo "exit=$?"
```

Expected: `Usage:` line on stderr, `exit=2`.

## Real receipt

Every prompt in this very repository's documentation session showed the anchor injected at `UserPromptSubmit`. The model used the injected `local_date: 2026-05-11` to resolve "tonight," "tomorrow morning," and "May 18 eligibility" correctly, instead of guessing from training cutoff.
