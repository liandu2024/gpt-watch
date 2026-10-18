# gpt-watch

**Quickly detect when ChatGPT / Codex is being silently degraded.** When a requested model is quietly replaced by a smaller, older or weaker one, the UI may show nothing. gpt-watch uses local logs and active probes to catch it and keep evidence. Pure Python stdlib; data stays on your machine.

[中文](README.md) | **English**

> A derivative of [ysh1112/codex-model-watch](https://github.com/ysh1112/codex-model-watch). Thanks to the original author **ysh1112**.

![Probe Testing](screenshots/panel-probe.png)

## Why it matters

Under capacity pressure or policy changes, ChatGPT / Codex may quietly route a request to a weaker model — sometimes even faking the model name in the response. Quality drops, but it is hard to tell whether the model got dumber or something else happened. gpt-watch turns "degradation" into verifiable data:

- **Degradation detection**: a quick probe catches direct swaps; the deep canary uses identity fingerprints, knowledge cutoff / contemporaneity, reasoning depth and an answer check to catch swaps that also fake the self-report.
- **Evidence & history**: every probe records requested → served and a unified Pass / Fail / Uncertain result; search by model, type, result, time or verdict text, and hover a result for the reason.
- **Usage & quota**: turns, tokens and duration per model / project, plus 5h / 7d quota levels and capacity rejections, as supporting evidence.

![Dashboard](screenshots/panel-overview.png)

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

### Reference signals and evidence

The quick probe also records the `x-codex-turn-state` length, `x-codex-safety-buffering-faster-model`, `request-id` and `cf-ray`. Community reports correlate turn-state length with degraded routing (e.g. roughly 292 healthy / 312 degraded on personal plans), but this is a **heuristic** and may change with the server; it **does not affect the Pass / Fail / Uncertain verdict**. The authoritative signal remains requested model == served model plus canary behavior.

Use **Export evidence** at the top right of the probe history to download a JSON file with requested / served models, latency, verdict, request-id, cf-ray and turn-state length — ready to attach to a support ticket.

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

The UI follows the system theme (light / dark); on phones the top bar uses two rows and tables scroll horizontally. Probe progress and results use a single-line toast at the top without shifting the page.

## Privacy

- Parsing, aggregation and storage happen on your machine; the dashboard binds to `127.0.0.1` by default.
- The only outbound request is a probe you trigger manually (to your Codex backend or a relay you configure).
- No credentials, telemetry or phone-home in the repository.

## Credits

This project is a derivative of [ysh1112/codex-model-watch](https://github.com/ysh1112/codex-model-watch). Thanks to the original author **ysh1112** for the open-source work. Probe and canary methodology also draws on public ideas from BazaarLink / LLMprobe-engine, and on [codex_state_monitor](https://github.com/YueDongSun/codex_state_monitor)'s public analysis of the `x-codex-turn-state` / safety-buffering headers.

## License

[MIT](LICENSE)
