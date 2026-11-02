---
name: freetoken-bots
description: >-
  Discover, verify, and maintain zero-priced OpenRouter models for Pi
  child-agent workflows (Muse, Grokbot, Dot). Use when the user mentions
  OpenRouter, Pi, free tokens, or free models (免费 token / 免费模型): Pi install,
  free-model discovery, quota-safe fallback, model recommendations, daily health
  checks.
---
# FreeToken-Bots

Source: https://github.com/limin112/min-skill/tree/main/skills/FreeToken-Bots (v0.2.0). Script and tests live in this skill's folder: `/home/box/agent-data/workflows/freetoken-bots/scripts/radar.py` (Python stdlib only).

Pi is the execution layer; Muse, Grokbot and Dot may invoke it **only if they can run shell commands or access a separately deployed Pi service**. Do not claim to replace those products' native models.

## Step 0 — Install Pi first (required)

If `command -v pi` fails, install it before anything else (explain and get the user's ok first):

```bash
curl -fsSL https://pi.dev/install.sh | sh
pi --version
```

Alternative (Node.js >= 22.19): `npm install -g @earendil-works/pi-coding-agent`.

- Pi config lives in `~/.pi/agent/` (`settings.json`, `models.json`). Re-run this step whenever `pi` is missing from PATH.
- After install, run `pi --list-models` once to confirm.

## Principles

- Free means OpenRouter catalog `pricing.prompt == 0` and `pricing.completion == 0`, with any request/cache prices also zero. Free eligibility is dynamic.
- Only classify **verified** total-parameter counts >= 100B as flagship; never guess from a name. Unknown counts go to `needs_parameter_verification`.
- Require `tools` in `supported_parameters`; independently probe tool-calling before production use.
- Never route to a paid model automatically. A 402, quota exhaustion, or rate limit is not a reason to remove existing Pi provider config.
- `probe --model` resolves IDs against the latest free scan: missing `:free` suffix is auto-corrected with a warning; unknown IDs are refused.
- Never put API keys in Git or print them. Use `OPENROUTER_API_KEY` from the environment; ask for it via a secret input if unset.
- Don't rewrite Pi settings without preserving user providers/defaults.
- All unattended `pi -p` calls must use `</dev/null`.

## Usage

```bash
cd /home/box/agent-data/workflows/freetoken-bots
python3 scripts/radar.py scan
python3 scripts/radar.py status
python3 scripts/radar.py probe --model 'exact/openrouter-id:free'   # needs OPENROUTER_API_KEY
python3 scripts/radar.py sync --dry-run
```

`scan` writes `~/.pi/agent/freetoken-bots.json`. `sync` only emits a proposal; it does not modify `models.json`.

## Pi integration workflow

1. Pi on PATH and `pi --version` working.
2. Check `~/.pi/agent/settings.json`, `models.json`, `pi --list-models` without disclosing keys.
3. Scan; move only independently verified >=100B entries into the approved pool.
4. Probe a plain completion and a separate real `tools` call; distinguish 429 from hard daily-quota exhaustion.
5. Generate a candidate diff; don't overwrite custom providers or `enabledModels`.
6. Require explicit user approval of the diff. Back up, write atomically, validate with `python3 -m json.tool`, `pi --list-models`, `pi -p 'say pong' --no-extensions </dev/null`; revert on failure.
7. Never expose Pi's shell/API publicly without auth. Ask which model to use before major tasks; auto-fallback only after opt-in.

## Scheduling (required)

The skill does not scan by itself. A daily 08:00 local scan is required, or the free list goes stale. Tell the user on install: "这个 skill 需要每天早上 8 点跑一次扫描定时任务，不配的话免费名单会过期，它自己不会动。" Each run compares to the previous report and surfaces newly free, delisted, or repriced models and changes to the `eligible` pool.

## Model selection

Prefer verified free >=100B tool-capable models; report a smaller tool-capable free model separately when operationally better. If none passes, report an empty pool rather than falling back to paid tokens.
