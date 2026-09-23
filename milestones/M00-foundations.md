# Milestone M00 — Engineering Foundations and the Boring Workload

*Phase 00 · ~3 weeks · Months ~0–1 · Prerequisites: none · Unlocks: M01 (Linux and networking), M06 (containers can be pulled forward from here)*

## Why this phase exists

The JD asks you to *"ensure process and enhancement specifications are translated into effective programs with error-free code using enterprise development tools, methods, and languages"* and to *"publish and maintain high-quality documentation … within standard templates and documentation guidelines"*. Before any cloud resource exists, you need three things every later phase stands on:

1. **A repository with enforced discipline** — branch protection, required checks, a decision record. Every later artefact (Terraform, pipelines, runbooks, incident records) lives here, and the JD's "Git … automated testing" requirement starts here.
2. **A workload worth operating** — `cargo-api` and `carrier-sim`. They must have every operationally interesting property (database, dependency, cache, background work) and **no interesting business logic**. You already write Python/FastAPI well; the goal is a service you can break in twenty different ways, not a product.
3. **The documentation habit** — ADRs and a README written from day one, because documentation written later is fiction.

It is first because nothing depends on the cloud yet, and because a clean foundation is far cheaper than a retrofit.

## Concepts to learn

- **Git's object model**: blob, tree, commit, ref. Once you see that a branch is a movable pointer and a commit is an immutable snapshot with a parent hash, `rebase`, `reflog`, `cherry-pick` and `bisect` stop being scary. Understand *why* rebasing a shared branch rewrites hashes and breaks collaborators.
- **Trunk-based development** vs long-lived branches: integration pain grows with branch age. Short-lived branches, small PRs, required checks.
- **Branch protection, CODEOWNERS, required status checks, signed commits** — the enterprise controls around a repository, and what each one prevents.
- **Semantic versioning** and what "breaking change" means to a consumer. You will version Terraform modules and APIs with it later.
- **Architecture Decision Records**: context, options considered, decision, consequences, and *what evidence would change your mind*. Who reads them (future you, reviewers, auditors) and when.
- **Twelve-factor principles**, specifically config in the environment, stateless processes, disposability (fast start, graceful stop), dev/prod parity, logs as event streams. Every later runtime (EC2, Fargate, Kubernetes) assumes them.
- **API design basics you will be held to later**: resource modelling, status codes, idempotency keys for `POST`, pagination, versioning, and an OpenAPI contract as the source of truth.
- **The testing pyramid**: unit → contract → integration → smoke. Why a fast suite that runs on every PR beats a slow thorough one that people skip.
- **Database migrations** and the *expand–contract* pattern (preview — M07 makes you live it).
- **Makefiles as the interface**: `make dev`, `make test`, `make lint`. Anyone should run the project without reading your mind.

## Architecture decisions in this phase

### ADR-001 — Repository strategy
- **Problem:** Runway will contain platform code, several workloads, docs, policy and pipelines. How should it be split?
- **Options:** (a) polyrepo from day one; (b) monorepo forever; (c) monorepo now, splitting out `hello-svc` and the golden-path template later.
- **Decision:** (c). A monorepo is far easier to review, refactor and show; `hello-svc` and the template move to their own repositories in M17 because *cross-repository* consumption of reusable workflows and modules is exactly what you must prove.
- **Trade-offs:** monorepo CI needs path filters or it becomes slow; ownership boundaries must be expressed with CODEOWNERS rather than repository permissions; the split in M17 is a small migration you must plan.

## Hands-on labs

1. **Break and recover Git.** In a scratch clone: commit to the wrong branch, amend a pushed commit, force-push over "a colleague's" work, delete a branch with unmerged commits. Recover each with `reflog`. *Proves: you can recover before you need to.*
2. **`git bisect`** a regression you plant across 20 commits, then automate it with `git bisect run make test`. *Proves: you can find which change broke something without reading every diff.*
3. **Branch protection.** Require a PR, one passing check and linear history on `main`. Try to push directly and watch it fail. *Proves: controls are enforced, not promised.*
4. **Idempotency.** Send the same `POST /shipments` twice with and without an `Idempotency-Key` header. *Proves: you understand why retries in later phases need idempotent APIs.*
5. **Migration pain on a laptop.** Rename a column in one migration while the old code still runs against the database. Watch it break. *Proves: the most common real deployment outage, met safely.*
6. **Secret in history.** Commit a fake secret, remove it with `git filter-repo`, then write down why the credential is still compromised and must be rotated. *Proves: removal ≠ revocation.*

## Project implementation

Create the `runway/` repository with the layout from [Section 5](../05-production-project-proposal.md) (empty directories get a `README.md` saying what will live there and in which milestone).

- `workloads/cargo-api/`: FastAPI, typed Pydantic models, endpoints `POST /shipments`, `GET /shipments/{id}`, `GET /shipments?status=&page=`, `GET /shipments/{id}/eta`. PostgreSQL via docker-compose, Alembic migrations, pytest with a fast unit suite and a small integration suite using testcontainers. Idempotency key on create. An OpenAPI document committed as `openapi.yaml` and checked in CI against the generated one.
- `workloads/carrier-sim/`: a tiny service representing an external carrier API with runtime toggles: `/admin/latency?ms=`, `/admin/fail?rate=`, `/admin/hang`. You will use it to cause incidents for eighteen months.
- `Makefile` with `dev`, `test`, `lint`, `fmt`, `migrate`.
- `.github/workflows/ci.yml`: lint (ruff), type check (mypy), unit tests on every PR; required on `main`.
- `docs/adr/ADR-001-repository-strategy.md`, `docs/adr/README.md` (index), `docs/adr/template.md`.
- `catalog/cargo-api/catalog-info.yaml` and `catalog/carrier-sim/catalog-info.yaml` (Backstage format, even though nothing reads them until M18 — the habit starts now).
- `CODEOWNERS`, PR template with "what / why / how tested / rollback plan" sections (the seed of change records in M07/M11).

**Timebox the application to two weekends.** If you find yourself adding business features, stop — you have drifted.

## Engineering problems to hit deliberately

| Problem | How to cause it | What you should learn |
|---|---|---|
| Lost work | Hard-reset away three commits | `reflog` is your safety net; nothing committed is really lost locally |
| Duplicate records from retries | Retry a `POST` without idempotency | Why retries require idempotent operations |
| Schema/code mismatch | Rename a column while old code runs | Backward compatibility is a deployment property, not a code property |
| Secret committed | Commit `.env` | Rotation is the fix; history rewrite is hygiene |
| Flaky test | Test that depends on wall-clock time | Flaky tests destroy trust in CI faster than missing tests |

## Troubleshooting exercises

- **"CI is red on main but it passed on my branch."** Cause something that passes in isolation and fails after merge (two PRs that each pass but conflict semantically). Find it with `bisect`. Write the fix to your branching rules.
- **"The API returns 500 on the list endpoint for some pages only."** Plant an off-by-one in pagination that only fails at a boundary. Find it from logs and a failing test, not by reading code first.

No scenario from the catalogue yet — there is no system to fail. The discipline here is recording what you did.

## Deliverables

- [ ] `runway/` on GitHub with branch protection, CODEOWNERS and a PR template
- [ ] `cargo-api` + `carrier-sim` running with `make dev`; `make test` green
- [ ] `openapi.yaml` committed and verified in CI
- [ ] `docs/adr/ADR-001-repository-strategy.md`, `docs/adr/template.md`, ADR index
- [ ] `catalog/*/catalog-info.yaml` for both components
- [ ] `README.md` stating what exists and — explicitly — what does not yet
- [ ] `docs/estimates/M00.md` (see Estimate vs actual)

## Completion criteria

- A PR cannot merge without lint, types and tests passing; you have demonstrated it by opening a failing PR.
- From a fresh clone, `make dev && make test` succeeds in under 10 minutes on a clean machine (time it).
- You recover a deleted branch from `reflog` without looking anything up.
- `git bisect run` finds a planted regression automatically.
- Explain without notes: what `rebase` does to hashes and why that matters on shared branches; why config lives in the environment; who reads an ADR and when; why `POST` needs idempotency when clients retry.

## Mentor checkpoints

- **Before building:** bring the resource model for `cargo-api` (endpoints, status codes, error format) and the ADR-001 draft. Expect: "What would make you regret the monorepo?" and "Which of these endpoints is not idempotent and what happens on retry?"
- **Code review:** bring the PR that adds the list endpoint. The mentor will look for pagination limits, input validation, error shape consistency and test quality — not features.
- **The failure I'll inject after you think you're done:** a change that is green in CI but breaks a consumer of your API contract. You will find it from the evidence.

## Resources

- *Pro Git* (free at git-scm.com) — ch. 3 (Branching), ch. 7.6 (Rewriting History), ch. 10 (Git Internals: objects and refs).
- The Twelve-Factor App (12factor.net) — III Config, VI Processes, IX Disposability, X Dev/prod parity, XI Logs.
- Michael Nygard, "Documenting Architecture Decisions" (the original ADR post) — and the adr.github.io template list.
- FastAPI docs — "Bigger Applications", "Testing", "Settings and Environment Variables"; Alembic tutorial.

## Estimate vs actual

| | Hours | Cost |
|---|---|---|
| Planned | 30 | $0 |
| Actual | | |
| Variance note | | |
