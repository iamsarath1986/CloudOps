# Milestone M06 — Docker, Images and the Supply Chain

*Phase 06 · ~3 weeks · Months ~6–6.8 · Prerequisites: M00 (can be pulled forward from here if cloud spend must pause), M05 for the ECR part · Unlocks: M07 (CI/CD), M08 (Kubernetes)*

## Why this phase exists

The JD names *"containerisation and orchestration technologies such as Docker, Kubernetes, Amazon EKS, Azure AKS"*. Every runtime from M07 onwards — ECS Fargate, EKS, AKS — runs an image you build here. The image is also where the **supply chain** begins: what you ship, what vulnerabilities it carries, and whether production can prove it is running what CI tested.

You already use Docker. The goal is to go from "it builds" to **production-grade images**: minimal, non-root, correct signal handling, reproducible, scanned, with an SBOM, referenced by digest — the difference between a developer's Dockerfile and a platform team's standard. It sits after M05 because the registry and its access are Terraform-managed, and after M01 because a container is Linux namespaces and cgroups, which you have already seen by hand.

## Concepts to learn

- **What a container is:** namespaces (pid, net, mnt, uts, ipc, user), cgroups (CPU, memory, pids), a union/overlay filesystem, and a process — not a VM. The runtime stack: Docker/BuildKit → containerd → runc; OCI image and runtime specs.
- **Images:** layers, content-addressable digests, manifests and multi-arch manifest lists (arm64 vs amd64 — relevant for Graviton and Apple Silicon), tags as mutable pointers vs digests as immutable identities.
- **Build mechanics:** the build cache and why instruction order decides rebuild time; multi-stage builds; BuildKit cache mounts; `.dockerignore`.
- **Base images:** distroless vs slim vs alpine; the musl/glibc trap for Python wheels; pinned by digest; patch cadence as an end-of-life concern.
- **Runtime hygiene:** non-root `USER`, read-only root filesystem, dropped capabilities, no shell in production images (and how you debug without one — ephemeral debug containers in M08).
- **PID 1 and signals:** exec form vs shell form, why a shell as PID 1 swallows `SIGTERM`, `tini`/`--init`, graceful shutdown timing vs orchestrator grace periods.
- **Health checks:** Docker `HEALTHCHECK` vs orchestrator probes (ECS and Kubernetes ignore the Dockerfile healthcheck in different ways).
- **Resource limits:** memory limit → OOM killer → exit code 137; CPU shares vs quotas.
- **Supply chain:** vulnerability scanning (Trivy) and CVE triage (reachability, fix availability, severity vs exploitability); SBOMs (Syft, SPDX/CycloneDX); **signing** with cosign (keyless via OIDC in CI in M07); provenance/attestations (SLSA levels as vocabulary).
- **Registries:** ECR repositories, lifecycle policies, tag immutability, scan-on-push, cross-account pull from `shared-services`, pull-through cache for public images.
- **Reproducibility:** pinned base digests, lock files, deterministic builds; why "rebuild the same tag" is not the same artefact.

## Architecture decisions in this phase

### ADR-006 — Base images and supply-chain strategy
- **Problem:** Images must be small, patchable, auditable and trustworthy, and the approach must become a platform standard every onboarded workload inherits (M17).
- **Options:** (a) `python:3.x` full images; (b) slim Debian-based images; (c) distroless; (d) Alpine.
- **Decision:** Build stage on `python:3.x-slim`; runtime on distroless Python (or slim with a non-root user where a shell is genuinely needed, recorded as an exception). Base images pinned by digest; ECR in `shared-services` with **tag immutability**, scan-on-push and lifecycle policies; every image gets a Syft SBOM and is signed with cosign in CI (M07); Kyverno will verify signatures at admission (M15).
- **Trade-offs:** distroless complicates debugging (answered by ephemeral debug containers); digest pinning requires an automated update process (Renovate/Dependabot — adopted in M07); Alpine rejected for the musl/wheel problems and different DNS resolver behaviour.

## Hands-on labs

1. **Naive vs production.** Build `cargo-api` naively, then multi-stage on slim, then distroless. Record image size, build time, CVE count and cold pull time for each.
2. **Cache order.** Reorder the Dockerfile so dependency installation is cached; measure rebuild time after a one-line code change before and after.
3. **PID 1.** Run the app as a child of `sh -c`; `docker stop` it and watch it get `SIGKILL`ed after the grace period. Fix with exec form; confirm graceful shutdown completes an in-flight request.
4. **OOM.** Run with `--memory` below the working set. Read exit code 137 and the kernel message. You will recognise it instantly in M08 (S09).
5. **Non-root regressions.** Switch to a non-root user and make the app fail because it writes to a directory it does not own; then run with `--read-only` and fix properly with a `tmpfs` or a declared writable path (S08).
6. **Namespaces by hand.** `docker inspect` a container's PID, then `nsenter` into its network namespace from the host and run `ss`. Read its cgroup memory limit from `/sys/fs/cgroup`.
7. **SBOM surprise.** Generate an SBOM with Syft; find one transitive dependency you did not know you shipped.
8. **Triage.** Scan with Trivy; triage the top five findings: fixed version available? reachable? base vs application layer? Write the triage notes.
9. **Mutable tags.** Push `cargo-api:dev`, overwrite the tag with a different build, and pull on another machine. Then enable tag immutability and try again.
10. **Multi-arch.** Build for `linux/amd64` and `linux/arm64` with buildx; inspect the manifest list. Note the cost argument for Graviton later.

## Project implementation

- Hardened multi-stage Dockerfiles for `cargo-api`, `eta-worker` and `carrier-sim`: non-root, distroless runtime, no build tools in the final stage, exec-form entrypoint, pinned base digest, `.dockerignore` excluding `.env`, `.git`, tests and caches, OCI labels (`org.opencontainers.image.source`, `revision`).
- `docker-compose.yaml` updated to run everything with `read_only: true`, dropped capabilities, and memory limits — so local dev matches production constraints.
- `platform/modules/aws-ecr` — repositories in `shared-services` with tag immutability, scan-on-push, lifecycle policy (keep last N untagged, expire old dev tags), repository policy allowing pull from `workloads-*` accounts only.
- `Makefile`: `make image`, `make scan` (Trivy, fails on HIGH/CRITICAL with a fix available), `make sbom`.
- `docs/security/image-standard.md` — the image rules every workload must meet (this becomes a golden-path requirement in M17).
- `docs/security/vuln-triage-policy.md` — severity SLAs and what "accepted risk" requires.
- `docs/lifecycle/eol-register.md` created — first entries: Python runtime version and its end-of-life date, base image, PostgreSQL major version, Terraform and AWS provider versions.
- `docs/adr/ADR-006-base-images-and-supply-chain.md`.

## Engineering problems to hit deliberately

| Problem | How to cause it | What you should learn |
|---|---|---|
| Container ignores stop | Shell-form entrypoint | PID 1 signal handling decides deploy safety |
| OOMKilled | Memory limit below working set | Exit 137 = killed by the kernel, not a crash |
| Crash only in prod | Non-root + read-only FS | Hardening regressions surface late unless dev matches prod |
| Secrets baked into image | Broken `.dockerignore` | Anything in the build context can end up in a layer |
| "Same tag", different code | Overwrite a mutable tag | Deploy by digest; immutable tags |
| Wheel build failure | Alpine base with musl | Base choice has compatibility cost |
| CVE flood | Scan a full base image | Triage by fix availability and reachability, not by count |

## Troubleshooting exercises

- **S08 — "The new image starts fine locally but crashes immediately in the environment."** The runtime enforces a read-only root filesystem and non-root user; local runs did not. Diagnose from the container's last log lines and exit code.
- **S09 — "Worker containers restart every few minutes, exit code 137."** Find the memory growth pattern, decide between a leak and an undersized limit, and fix the right one.
- **"The image is 1.2 GB and deploys take four minutes."** Find the layer responsible with `docker history` / `dive` and fix it.
- **"Security flagged a critical CVE in production images."** Determine which images, which layer, whether a fixed base exists, rebuild, and prove production now runs the new digest. Write it up as `docs/incidents/INC-004-base-image-cve.md`.

## Deliverables

- [ ] Hardened Dockerfiles for `cargo-api`, `eta-worker`, `carrier-sim`
- [ ] Images in ECR (`shared-services`) by digest, tag-immutable, scan-on-push, lifecycle policies — all via Terraform
- [ ] `make image`, `make scan`, `make sbom`
- [ ] Scan report + triage notes committed
- [ ] `docs/security/image-standard.md`, `docs/security/vuln-triage-policy.md`
- [ ] `docs/lifecycle/eol-register.md` (first entries)
- [ ] ADR-006
- [ ] `docs/incidents/INC-004-base-image-cve.md`

## Completion criteria

- `cargo-api` image is non-root, distroless (or a documented exception), under 150 MB (measure yours and justify it), with zero HIGH/CRITICAL CVEs that have an available fix.
- `docker stop` during an in-flight request completes the request (demonstrate before/after).
- Explain what happens between `docker run` and your process executing, naming namespaces, cgroups, overlayfs, containerd and runc.
- Given an exit code (0, 1, 137, 143), explain what happened.
- Show that the digest running in the environment matches the digest built and scanned, and explain why a tag cannot prove that.
- Rebuild all images after a base-image update and redeploy in under 30 minutes, following your own procedure.

## Mentor checkpoints

- **Review:** bring the `cargo-api` Dockerfile and the image standard. Expect: "What runs as PID 1?", "What can an attacker do inside this container?", "How do you debug this in production without a shell?"
- **Triage review:** bring the Trivy triage notes. The mentor will pick two findings you accepted and ask you to defend them.
- **The failure I'll inject after you think you're done:** an image that passes every local test and fails at runtime because of an environment constraint you did not reproduce locally.

## Resources

- Nigel Poulton, *Docker Deep Dive* — chapters on images, containers, and "Containerizing an app" (multi-stage builds).
- Docker docs — "Building best practices", "Multi-stage builds", "Build cache" (optimisation section).
- iximiuz Labs — "Container Fundamentals" (namespaces, cgroups, what a container really is).
- Trivy docs — "Vulnerability scanning" and "SBOM"; Sigstore cosign docs — "Signing containers" (read now, implement keyless signing in M07).
- Amazon ECR User Guide — "Image tag mutability", "Lifecycle policies", "Image scanning".

## Estimate vs actual

| | Hours | Cost |
|---|---|---|
| Planned | 30 | $5–10 |
| Actual | | |
| Variance note | | |
