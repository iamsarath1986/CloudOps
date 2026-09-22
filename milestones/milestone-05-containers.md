# Milestone 5 — Containers and the Registry

*Phase 4 · ~2 weeks · Prerequisite: Milestone 1 (Milestone 4 for the registry identity)*

**Learning topics**
* Namespaces, cgroups and a union filesystem — what a container actually is, and is not
* Image layers, build cache, and why instruction order determines build time
* Multi-stage builds; base image choice (distroless vs alpine vs slim) and the musl/glibc trap for Python wheels
* Non-root users; `ENTRYPOINT` vs `CMD`; PID 1 signal handling
* Health checks; image size as deployment latency
* Tags vs digests, and why you deploy digests
* Vulnerability scanning, SBOMs, and triaging a CVE list without drowning

**Build**
Hardened multi-stage Dockerfiles for `cargo-api` and `carrier-sim`: non-root, minimal base, no build tools in the final layer, a `.dockerignore` that excludes `.env`, `.git` and tests, a healthcheck, correct signal handling via `exec` form, and pinned base image digests.

**Cloud resources**
Azure Container Registry (Basic tier — cheap, stateful, always on).

**DevOps work**
Trivy scanning in the `Makefile` so it is a local habit before it is a pipeline gate. SBOM generation. Push to ACR and reference by digest from that point on.

**Operations exercise**
Reproduce, and record the numbers for, each of: naive vs multi-stage image size and pull time; rebuild time before and after fixing layer cache order; CVE count as root vs non-root. Then get OOMKilled by setting a memory limit below what the process needs — read exit code 137 and the kernel message, because you will meet it again in Kubernetes and should recognise it instantly.

**Security considerations**
Break `.dockerignore` and find your `.env` inside a pushed image. Ship an image that cannot start because the non-root user cannot write where it expects — the most common hardening regression. Pull by a mutable tag someone moved underneath you and understand why digests exist.

**Expected outcome**
Images in ACR, hardened, scanned, with a committed triage note for every CVE you accepted. `docs/adr/ADR-003-base-image-strategy.md`.

**Evidence of mastery**
Explain what happens between `docker run` and your process executing. Explain why an image can pass tests and fail in production because of a base image difference. State your image's size, user and CVE count, and what you did about the count.
