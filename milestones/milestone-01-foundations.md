# Milestone 1 — Foundations and the Workload

*Phase 0 · ~2 weeks · Prerequisite: none*

**Learning topics**
* Git's object model; rebase, reflog, bisect; trunk-based development and branch protection
* Architecture Decision Records: context, options, decision, consequences, and what would change your mind
* Twelve-factor principles, specifically config-in-environment and stateless processes
* Semantic versioning and what "breaking change" means to a consumer
* Makefiles as the universal "how do I run this" interface

**Build**
The `runway` repository with its full directory skeleton. `cargo-api`: a FastAPI service with three endpoints, typed Pydantic models, an OpenAPI contract, PostgreSQL via docker-compose, Alembic migrations and pytest. `carrier-sim`: a second tiny service with a `/fail` toggle and configurable latency — your future failable dependency. A `Makefile`. **Timebox: two weekends. The application must stay boring.**

**Cloud resources**
None. Deliberately. You do not create a subscription until Milestone 3, so that when you do, you already know what you are deploying.

**DevOps work**
Branch protection on `main`: required PR, required status check, no direct push. A GitHub Actions workflow that runs lint, type check and tests on every PR. `.gitignore` and `.dockerignore` written before they are needed.

**Operations exercise**
Break the repository on purpose in a scratch clone: commit to the wrong branch, amend a pushed commit, force-push over work. Recover each with `reflog`. Then `git bisect` a regression you plant across twenty commits.

**Security considerations**
Commit a fake secret, remove it from history properly — and then write down why "removed from history" still means the credential is compromised and must be rotated. This is the lesson, not the `filter-repo` command.

**Expected outcome**
`make dev` starts the stack. `make test` is green. A pull request cannot merge without passing checks. `ADR-001` records why Azure, with the rejected options argued at greater length than the decision.

**Evidence of mastery**
Explain what `rebase` does to commit hashes and why that matters on a shared branch. Explain why configuration belongs in the environment and not in the image. Explain who reads an ADR and when. Recover deleted work from `reflog` without looking anything up.
