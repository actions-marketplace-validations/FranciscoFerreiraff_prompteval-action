# PromptEval GitHub Action

Lint your prompts in CI. Fails the build when a prompt scores below your bar, contains conflicting instructions, or **regresses** versus the version you have in production.

It's ESLint for prompts: a quality gate that runs on every PR that touches a prompt — no eval suite to write.

## Quick start

```yaml
# .github/workflows/prompt-check.yml
name: Prompt check
on:
  pull_request:
    paths: ['prompts/**']        # only run when a prompt changes

jobs:
  prompteval:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: FranciscoFerreiraff/prompteval-action@v1
        with:
          api_key: ${{ secrets.PROMPTEVAL_API_KEY }}
          prompt_file: prompts/support-agent.txt
          min_score: 80
          fail_on_conflict: true
```

Get an API key at **prompt-eval.com → dashboard → API**. Store it as the `PROMPTEVAL_API_KEY` repository secret.

## Inputs

| Input | Required | Default | Description |
|-------|:--:|--------|-------------|
| `api_key` | ✅ | — | PromptEval API key (`pe_live_...`). |
| `prompt` | * | — | Prompt text. Provide this **or** `prompt_file`. |
| `prompt_file` | * | — | Path to a file with the prompt. |
| `min_score` | | account threshold | Minimum passing score (0–100). Omit to use your account's production threshold. |
| `fail_on_conflict` | | `false` | Fail if conflicting instructions are detected. |
| `baseline_slug` | | — | A PromptEval prompt **slug**. Fails if the candidate scores lower than that prompt's current **production** version (regression gate). The recommended way to reference a baseline. |
| `baseline_version_id` | | — | Advanced: a specific version id. Prefer `baseline_slug`. |
| `max_drop` | | `0` | Allowed score drop vs the baseline before failing. `0` = any drop fails. |
| `mode` | | `lint` | `lint` (score + conflicts) or `full` (adds graph + recommendations + improved prompt). |
| `provider_key` | | — | Anthropic key (BYOK). Runs on your key: no quota, and unlocks `full` on any plan. |
| `api_url` | | prod | Override the API endpoint. |

\* exactly one of `prompt` / `prompt_file` is required.

## Regression gate

Point `baseline_slug` at your prompt's slug. The Action gates against whatever version is currently **production** for that slug — the version your business team promoted in the library. So a "small tweak" in a PR can't silently degrade the prompt you ship today, and the baseline stays correct even as production moves forward (no id to update).

```yaml
      - uses: FranciscoFerreiraff/prompteval-action@v1
        with:
          api_key: ${{ secrets.PROMPTEVAL_API_KEY }}
          prompt_file: prompts/support-agent.txt
          baseline_slug: support-agent
          max_drop: 2          # tolerate up to 2 points of noise
```

## BYOK (bring your own key)

Pass `provider_key` with your Anthropic key to run the eval on your own key: it doesn't count against your managed quota and unlocks `full` mode on any plan. Your CI already has the key — wire it as a secret.

```yaml
        with:
          api_key: ${{ secrets.PROMPTEVAL_API_KEY }}
          provider_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt_file: prompts/support-agent.txt
          mode: full
```

## Requirements

Runs on a Linux runner (`ubuntu-latest`) with `curl` and `jq` — both preinstalled on GitHub-hosted runners.

---

> Distribution note: to publish on the GitHub Marketplace and enable `uses: FranciscoFerreiraff/prompteval-action@v1`, move this folder to its own repository named `prompteval-action` and tag a `v1` release. Until then, reference it locally with `uses: ./prompteval/prompteval-action`.
