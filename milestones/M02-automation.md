# Milestone M02 — Python, Bash and PowerShell Automation

*Phase 02 · ~2 weeks · Months ~2–2.6 · Prerequisites: M00, M01 · Unlocks: M03, and every tool you will write for the platform*

## Why this phase exists

The JD asks for *"automation/scripting experience using Python, Bash, PowerShell"* and *"automation solutions"*. You already write Python services well; this phase is about a different craft — **tools that other engineers and pipelines depend on**, that must fail loudly and safely at 3 a.m. It is short because you have a head start. It sits here because from M04 onwards you will script the cloud, and a script that silently half-succeeds against a cloud API is how incidents start.

PowerShell is named in the JD and signals a Microsoft-heavy enterprise. The target is **read, modify and write small tools fluently** — not mastery.

## Concepts to learn

- **Scripts vs tools.** When a script needs tests, packaging, a `--help`, versioning and a changelog. Rule of thumb: the second person who runs it, or the first pipeline.
- **Bash safety:** `set -euo pipefail` and its sharp edges (pipelines, `|| true`, subshells), quoting, `trap` for cleanup, exit codes as an API, `shellcheck`.
- **Idempotency:** running twice produces the same result. Check-then-act vs desired-state. Why this is Terraform's core idea too.
- **Python tooling:** `argparse`/`typer` CLIs, `logging` with structured output, `pyproject.toml` packaging, `uv` or `pipx` install, type hints, pytest with fakes for external calls.
- **Calling cloud-style APIs correctly:** pagination, throttling (HTTP 429 / `ThrottlingException`), retries with **exponential backoff and full jitter**, timeouts on every call, partial failure reporting. You will meet AWS's `botocore` retry modes in M04.
- **Dry-run and confirmation patterns** for destructive tools. `--dry-run` default for anything that deletes.
- **PowerShell model:** objects not text in the pipeline, `Get-Member`, `Where-Object`/`Select-Object`, modules, `-WhatIf`/`-Confirm`, error handling with `try/catch` and `$ErrorActionPreference`. PowerShell 7 runs on Linux.
- **Secrets in scripts:** never in arguments (visible in `ps`), never in logs; read from environment or a secret store.

## Architecture decisions in this phase

Minor, recorded in the tooling README rather than as a numbered ADR:
- **Problem:** Which language for which tool?
- **Options:** everything in Bash; everything in Python; split by purpose.
- **Decision:** Bash for thin glue and diagnostic scripts that must run anywhere (`net-triage.sh`, `inject-fault.sh`); Python for anything with API calls, pagination, retries or data; PowerShell for one Windows/Azure-flavoured admin tool in M13 and read-fluency now.
- **Trade-offs:** two-and-a-half languages to maintain; justified by the JD and by using each where it is strongest.

## Hands-on labs

1. **Break `set -e`.** Write a Bash script where a failing command inside a pipeline or `if` is silently ignored. Fix it. *Proves: you know the edges of the safety net.*
2. **Retry storm in miniature.** Point a Python client at `carrier-sim` with `fail?rate=0.5`. Retry immediately with no backoff from 20 concurrent workers and watch load multiply. Add exponential backoff with jitter and a retry budget. *Proves: naive retries amplify failures (you will meet this again as S20 and S31).*
3. **Pagination bug.** Write a client for `GET /shipments?page=` that stops early on an empty page that is not the last. Write the test that catches it.
4. **Idempotent tool.** A script that creates a directory structure, user and systemd unit — safe to run ten times. Prove by running it ten times.
5. **PowerShell objects.** Rewrite a small Bash log-parsing pipeline in PowerShell 7 using objects rather than `awk`. Note what became easier and what became harder.
6. **Shellcheck and ruff in CI.** Add both to the existing CI workflow; fix every warning or justify each suppression in a comment.

## Project implementation

- `platform/scripts/runway-tools/` — a Python package (`pyproject.toml`, `typer` CLI, tests) with:
  - `runway-tools seed --count N` — seeds `cargo-api` through its API, idempotently (uses idempotency keys).
  - `runway-tools smoke --base-url` — a smoke test that exits non-zero with a clear message; the pipeline in M07 will call it.
  - A shared `http` helper with timeouts, backoff with jitter and a retry budget — every later tool uses it.
- `platform/scripts/inject-fault.sh` — the first version of the fault injector: flips `carrier-sim` toggles, fills a disk on the VM, blocks a port. It grows all the way to M20.
- `platform/scripts/pwsh/Runway.Tools/` — a PowerShell module skeleton with one function (`Get-RunwayHealth` calling `/readyz` across endpoints, returning objects).
- CI: `shellcheck`, `ruff`, `mypy` and tests for the tools package.

## Engineering problems to hit deliberately

| Problem | How to cause it | What you should learn |
|---|---|---|
| Silent partial failure | Loop over items, ignore one error | Tools must report partial success explicitly |
| Retry amplification | Immediate retries at scale | Backoff, jitter, retry budgets |
| Secret leaked via `ps` | Pass token as CLI argument | Environment or stdin, never argv |
| Destructive script run twice | Non-idempotent create/delete | Desired-state thinking |
| Hung script in CI | No timeout on an HTTP call | Every network call needs a deadline |

## Troubleshooting exercises

- **"The nightly seeding job reports success but half the data is missing."** Cause: an exception swallowed inside a loop plus an early pagination exit. Find it from logs alone, then fix the tool so this failure is impossible to miss.
- **"`carrier-sim` is overloaded every time it has a blip."** Cause: your own client's retry behaviour. Measure request rate before and after the fix and put the numbers in the incident note.

## Deliverables

- [ ] `platform/scripts/runway-tools/` package with tests, installable with one command
- [ ] Shared HTTP helper with timeout, backoff + jitter, retry budget, and tests for each
- [ ] `platform/scripts/inject-fault.sh` v1 with `--list` and `--dry-run`
- [ ] `platform/scripts/pwsh/Runway.Tools/` with one working, `-WhatIf`-aware function
- [ ] CI running shellcheck, ruff, mypy, pytest

## Completion criteria

- Your retry helper, under 50% injected failure, keeps upstream request rate within 2× baseline (measured), where naive retry exceeded 5×.
- Every tool exits non-zero on any partial failure and prints which items failed.
- Explain `set -euo pipefail` and two situations where it does *not* stop the script.
- Read an unfamiliar 50-line PowerShell script and explain what it does, including the pipeline object types.
- `inject-fault.sh --dry-run` shows exactly what it would do; without it, it does it.

## Mentor checkpoints

- **Review:** bring the HTTP helper. Expect: "What is the maximum time a caller can wait in the worst case?" and "What happens if the server is down for ten minutes?"
- **The failure I'll inject after you think you're done:** an API that returns HTTP 200 with an error in the body, and one that paginates with a cursor that sometimes repeats. Your tool must not report success.

## Resources

- AWS Architecture Blog, "Exponential Backoff and Jitter" (Marc Brooker) — the whole post; you will cite it in M19.
- Google Shell Style Guide + ShellCheck wiki — read the wiki pages for SC2086, SC2155, SC2181.
- Python Packaging User Guide — "Packaging Python Projects"; Typer docs — "Tutorial: First Steps", "CLI Arguments", "Testing".
- *Learn PowerShell in a Month of Lunches* — ch. 6–10 (pipeline, objects, formatting, filtering) only.

## Estimate vs actual

| | Hours | Cost |
|---|---|---|
| Planned | 20 | $0 |
| Actual | | |
| Variance note | | |
