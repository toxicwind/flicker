# Flicker

> Part of [**the ranch**](https://github.com/toxicwind/ranch) — the whole inference estate, one map.

> **Retired 2026-09-30.** Flicker is no longer supervised or running on yote.
> Port `:25148` is now served by `mbx-cache` (the mise remote task cache);
> nothing listens on `:25240`. The committed binaries in `bin/` and the state
> dir `/home/toxic/flicker/` remain as artifacts.

Flicker is the estate's local-only build daemon: a Woodpecker v3 Go fork stripped down to direct job execution with content-hash caching. It replaced the old Python `buildsrv` daemon (briefly and mistakenly renamed "brand" on 2026-09-30 — that name is gone) on the same port, as a drop-in replacement.

No containers, no remote forges, no auth — single-tenant daemon that runs jobs through bash login shells (so mise toolchains resolve) and returns `CACHED` when an identical job spec was already run.

## Quick start

```bash
# Health
curl -fsS http://127.0.0.1:25148/api/health

# Submit a job
curl -s -X POST http://127.0.0.1:25148/api/jobs \
  -H "Content-Type: application/json" \
  -d {'"name":"hello","command":"echo hello","workdir":"/tmp","timeout":300}'}

# Or use the CLI
flicker submit --name hello -- 'echo hello'
flicker status 1
flicker logs 1
```

## API

| Method | Path | What |
|---|---|---|
| GET | `/api/health` | Service health + cache stats |
| POST | `/api/jobs` | Submit a job (`name`, `command`/`script`, `workdir`, `env`, `timeout`, `cache_key`, `artifacts`) |
| GET | `/api/jobs` | List jobs |
| GET | `/api/jobs/:id` | Job status (`pending`/`running`/`success`/`failure`/`canceled`/`error`) |
| GET | `/api/jobs/:id/logs` | Plain-text job logs |

Identical job specs (command, workdir, sorted env, cache key, sorted artifacts) hash to the same content key — resubmits return the cached success immediately.

## Layout

- This repo root — the Woodpecker v3 fork (`bin/` holds committed binaries: `flicker-server`, `flicker-agent`, `flicker`)
- `/home/toxic/.local/bin/flicker` — the CLI (on yote)
- `/home/toxic/flicker/` — state dir on yote (legacy, from when flicker ran)
- `pitchfork.d/flicker.toml` — the former supervisor manifest (retired; kept for reference)

Flicker is retired (2026-09-30): nothing is supervised or running on yote,
port `:25148` is now served by `mbx-cache` (mise remote task cache), and
nothing listens on `:25240`.

## History

- `buildsrv`: original stdlib-Python daemon (`buildsrvd.py`), port 25148.
- 2026-09-30 00:05: renamed to `brand` (bad name, acknowledged).
- 2026-09-30 01:54: deleted from the tree in an unrelated commit.
- Same night: Flicker (this fork) cut over as the live replacement. All `brand`/`branding` folders removed 2026-10-02.
- 2026-09-30 (later the same night): retired — `:25148` handed to `mbx-cache`
  (mise remote task cache); flicker no longer supervised or running.
