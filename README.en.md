# gpt-watch

A local-first dashboard for Codex: model usage, 5h/7d quota, capacity rejections, and active probes that catch silent model swaps. Pure Python stdlib; data stays on your machine.

[中文](README.md) | **English**

![Dashboard](screenshots/panel-overview.png)

## Features

- **Usage & quota**: turns, tokens and duration per model / project, plus 5h / 7d quota levels and reset times.
- **Anomalies**: capacity rejections, rate limits and network errors; the original message appears when you hover the type badge.
- **Quick probe**: sends one minimal request and compares the requested and served model names.
- **Deep canary**: identity fingerprint + knowledge cutoff / contemporaneity + reasoning depth + answer check, for swaps that also fake the self-reported model name.
- **Probe history**: unified Pass / Fail / Uncertain results, searchable by model, type, result, time and verdict text; hover a result for the reason.
- **Global toast**: probe progress/results, model-list errors and demo mode appear as a single line at the top, without shifting the page, and dismiss automatically.

![Probe Testing](screenshots/panel-probe.png)

## Quick start

Requires Python 3.8+ (standard library only).

```bash
git clone https://github.com/liandu2024/gpt-watch.git
cd gpt-watch

python3 codex_model_watch.py          # real Codex logs
python3 codex_model_watch.py --demo   # built-in demo data
```

Open <http://127.0.0.1:8787>. On startup it incrementally scans `~/.codex/sessions/**/*.jsonl`; the database lives at `~/.codex-model-watch/state.db`.

## Two probes

| | Quick probe | Deep canary |
|---|---|---|
| What it does | Sends `hi` and reads the served model name | Identity, knowledge and reasoning checks |
| Use it for | Verifying the link and direct swaps | Suspected self-report faking or an old-model shell |
| Cost | Seconds, tiny quota | Minutes, more quota |
| Result | Pass / Fail / Uncertain | Pass / Fail / Uncertain |

> Since Oct 2026, a soft swap can also fake the model name in `response.created`. A quick probe proves the link is alive, not that the model is genuine — use the canary for cross-checks.

Hover a result to read the reason. A verdict describes that moment only; swapping can change with capacity.

## Data and limits

- **Log side**: parses `turn_context`, `token_usage_record`, `task_complete` and `token_count` for the effective model, tokens, duration, errors and 5h/7d quota.
- **Historical swaps cannot be reconstructed**: Codex writes the post-swap model consistently into local logs, so both requested and effective fields show the replacement. Only an active probe can verify the current moment.
- **Probes are evidence, not proof**: self-reported identity can be corrupted by instructions or training residue; combine multiple canary signals.
- The rollout format is internal to Codex and may change (verified on Codex CLI 0.153–0.155).

## CLI options

| Option | Description |
|---|---|
| `--host 0.0.0.0` | Allow LAN / phone access (default: `127.0.0.1` only) |
| `--port 8787` | Dashboard port |
| `--max-age-days 30` | Parse only the last N days; `0` = everything |
| `--codex-home PATH` | Codex home (default `~/.codex`) |
| `--demo` | Built-in demo data, real logs untouched |
| `--scan-only` | Parse once, print a model summary, exit |
| `--no-open` | Do not auto-open the browser |
| `--api-base` / `--api-key` | Send the canary to a relay / pool via `/v1/chat/completions` |

LAN example:

```bash
python3 codex_model_watch.py --host 0.0.0.0 --no-open
# From the same Wi-Fi: http://YOUR_LAN_IP:8787
```

The UI follows the system theme (light / dark); on phones the top bar uses two rows and tables scroll horizontally.

## Privacy

- Parsing, aggregation and storage happen on your machine; the dashboard binds to `127.0.0.1` by default.
- The only outbound request is a probe you trigger manually (to your Codex backend or a relay you configure).
- No credentials, telemetry or phone-home in the repository.

## License

[MIT](LICENSE)
