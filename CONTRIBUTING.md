# Contributing to time-anchor

Tiny project, low-ceremony.

## Filing an issue

If the anchor emits something wrong or unexpected, please open an issue with:

1. Exact invocation (`bash time-anchor.sh text` or the piped JSON).
2. Observed output.
3. Expected output.
4. `python3 --version`.
5. Shell + OS.
6. Your system timezone if relevant.

## Filing a PR

PRs welcome for:

- bugs (timezone edge cases, DST, leap seconds if you're feeling adventurous)
- portability fixes (BSD/GNU date differences, Windows WSL quirks)
- new output modes that other hook ecosystems would want
- documentation improvements

Before opening:

- Add a new smoke test to `.github/workflows/test.yml` covering the case.
- Keep the script self-contained — no extra dependencies beyond bash + python3.

## Out of scope

- Calling external time APIs. The whole point is the local system clock.
- Becoming a full timezone library. If you need that, use `pytz` / `zoneinfo` in your own code.
- Bundling unrelated hooks. See [no-vibes](https://github.com/waitdeadai/no-vibes) for the sister hook on closeout verification.
