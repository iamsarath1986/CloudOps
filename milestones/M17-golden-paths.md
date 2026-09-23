# Milestone M17 — Platform Engineering: Standard Patterns, Reusable Modules, Golden Paths

*Phase 17 · ~4 weeks · Months ~18–19 · Prerequisites: [M14](M14-aks-multicluster.md) (two clusters with GitOps), [M15](M15-security-governance.md) (policy), [M16](M16-finops.md) (cost allocation), [M11](M11-sre-operations.md) (operational standards) · Unlocks: [M18](M18-developer-platform.md), [M19](M19-ai-infrastructure.md) · **Never cut this milestone.***

## Why this phase exists

This is the milestone the JD is built around. The team is *"our Platform Engineering team"*. The role is to support *"cloud automation, workload onboarding, platform engineering, and the development of internal self-service capabilities"*. The requirements ask for experience *"operating large public cloud footprints and onboarding applications using governed and automated DevOps practices"* and building *"cloud-native platforms, internal developer tools, self-service portals, automation solutions"*.

It sits here because a golden path is a **packaging of everything you already know how to do**:
- IaC (M05)
- delivery (M07, M14)
- runtime (M09, M14)
- observability (M10)
- operations standards (M11)
- security and policy (M15)
- cost allocation (M16)

Packaging things you cannot yet do yourself produces a template, not a platform.

## Concepts to learn

**The progression you have lived through, named.** Platform engineering is the last step of a ladder you have been climbing since M04. Write out each rung with your own Runway example:

| Rung | What it is | Runway example | What it could not do |
|---|---|---|---|
| Manual operations | Humans click and remember | M04: `cargo-api` on EC2 by hand, `manual-build-log.md` | Repeat, audit, scale |
| Automation | Scripts do the steps | M02 tools, `net-triage.sh` | Hold desired state; scripts drift from reality |
| Infrastructure as Code | Declared desired state, planned and reviewed | M05 Terraform | Get itself applied safely and consistently |
| CI/CD | Every change flows through a pipeline | M07 reusable workflows, M14 Argo CD | Stop each team reinventing the same pipeline |
| Standardised cloud patterns | Approved reference designs ("an HTTP service looks like this") | ADRs, the SDD template, the M15 baseline | Be *consumed*; a pattern on a wiki is advice |
| Reusable Terraform modules | Patterns as versioned code with an interface | `aws-network`, `aws-eks`, `k8s-baseline` | Compose into "a whole service" in one step |
| **Golden paths** | An opinionated, end-to-end supported route: repo template + contract + module + pipeline + GitOps + telemetry + policy | **This milestone** | Be discovered and requested without reading a README |
| Internal developer platform | The paths, catalogue, and APIs behind one front door | [M18](M18-developer-platform.md) | — |
| Self-service infrastructure | Requesting without waiting for a human, still governed | [M18](M18-developer-platform.md) | — |

**The platform as a product.** Internal engineers are your users. In practice, adoption is voluntary: a platform that is slower than doing it by hand gets bypassed. Developer experience (DX) is the product's usability, and it is measured by:
- time to first deploy
- the number of steps and concepts a user must learn
- how legible the platform's failures are
- how quickly users get help

*Team Topologies* supplies the vocabulary: the platform team exists to **reduce the cognitive load** of stream-aligned teams. Its interaction mode is "X-as-a-Service", with occasional facilitation.

**Golden paths.** A golden path is an opinionated, supported, well-documented route covering roughly 80% of cases. It needs documented escape hatches for the other 20%. There are two ways to get it wrong: too narrow, so most teams cannot use it, or too configurable, so it stops being a path. Both show up as low adoption.

**Paved road vs guardrail.**
- A paved road is a default that is right, so most users never think about it.
- A guardrail is a limit that is enforced, so no one can exceed it: policy as code from M15, working at three points (conftest on the plan, cloud policy at apply time, Kyverno at admission).

Guardrails are what let you *grant* autonomy. **Policy is not the opposite of self-service; it is its precondition.**

**The platform API: contract over implementation.**
- The developer states *intent* in `workload.yaml`: what kind of service it is, the owner, the tier, the target cloud, whether it needs a database, a queue or an AI quota.
- The platform owns *implementation*: which Terraform resources, which Helm values, which alerts.
- The contract is the API. It has a schema, validation, versioning and a deprecation policy.
- Module design is API design:
  - minimal required inputs
  - opinionated defaults
  - stable outputs
  - non-negotiables that are **not** variables (tags, NetworkPolicy, resource limits, log schema, image signing)
- Semantic versioning is a promise. Anything that forces a consumer to edit their code is a major version.

**Templates and scaffolding.**
- A template repository gives a new service the right shape on day one: Dockerfile, `workload.yaml`, thin CI caller, `catalog-info.yaml`, a README skeleton and a runbook skeleton.
- Templates rot as soon as they are copied, so everything that must stay current belongs in *called* things (reusable workflows, modules, Helm charts) rather than *copied* things.

**Standardised CI/CD.**
- Consumers call reusable workflows pinned by tag.
- The test of this: **can you add a mandatory pipeline stage for everyone without opening a PR in any workload repo?**

**Standardised observability.**
- Being on the platform gives you dashboards, SLOs, alerts routed to L1 and L2, OTel configuration and the log schema.
- Observability becomes a property of the platform, not a task each team remembers to do.

**Environment configuration.**
- Define a service once and parameterise it per environment.
- Differences between environments are data (sizes, replica counts, domains), never separate code paths.

**Platform metrics.**
- time to first production deploy
- manual steps remaining
- platform-code touches per onboarding (target: zero)
- module version spread across consumers (your migration debt)
- change failure rate for platform-managed deploys compared with hand-rolled ones
- adoption

**Deprecation and migration are the real work.**
- Shipping v1 is easy.
- Releasing v2 and migrating consumers you do not control is the job.

## Architecture decisions in this phase

**ADR-021 — Golden path contract (`workload.yaml`) and module versioning policy**

| | |
|---|---|
| **Problem** | Onboarding a workload currently means editing Terraform in `live/`, copying workflow YAML, hand-writing Argo CD Applications, dashboards and alerts, and remembering tags and policies. It is slow, error-prone and depends on one person (you). |
| **Options** | (a) Documented checklist + copy-paste templates. (b) A `workload` Terraform module that teams call directly in HCL. (c) A small declarative contract (`workload.yaml`, JSON-Schema validated) rendered by the platform into one `workload` module invocation, an Argo CD Application and standard telemetry. (d) Crossplane/Kubernetes-native claims. |
| **Decision** | (c). The contract has ~8 required fields (`name, owner, tier, target: aws\|azure, runtime: http\|worker, port, health`) and optional `needs: {postgres, queue, cache, ai_quota}`. It is schema-validated in CI. `platform/modules/workload` is released by Git tag under SemVer. Deprecation policy: N-1 supported, 90-day notice, a migration guide mandatory for each major version. Escape hatches: `overrides:` limited to an allow-listed set, or "graduate" to raw modules with an ADR. |
| **Trade-offs** | (a) doesn't scale or enforce. (b) leaks implementation: consumers must learn Terraform and the internals of your provider. (d) is elegant but adds a control plane you would operate for four services, so it is revisited if the platform ever has many more consumers. (c) puts a renderer and a schema you must maintain in the middle. A thin abstraction must stay *debuggable*: `runway render` shows exactly what it generated. |

## Hands-on labs

1. **Onboarding inventory.** Onboard `carrier-sim` from scratch using today's process, and time it. Write down every manual step, file edit, credential and concept involved. This list is your specification. *Proves: you measure before you abstract.*
2. **Design the contract on paper first.** Write `workload.yaml`'s JSON Schema and its README before writing any code. Get it reviewed by the mentor. *Proves: interface first.*
3. **Module interface kata.** Write two versions of the `workload` module interface: one with 25 variables and one with 8 plus defaults. Get a friend or the mentor to "use" each. *Proves: exposing implementation vs intent.*
4. **Mandatory stage test.** Add an SBOM attestation step to the reusable `build-sign-push` workflow and roll it out to every consumer **without touching their repos**. If you cannot, fix the workflow structure. *Proves: called beats copied.*
5. **Breaking change.** Rename a contract field and release `workload` module `v2.0.0`. Write the migration guide. Migrate your three consumers, and deliberately leave one on v1 for a week to watch version spread. *Proves: migration is the job.*
6. **Leaky abstraction.** Try to onboard a service that needs a persistent volume, or a non-HTTP TCP port. Decide between extending the path and documenting an escape hatch, and justify the decision in writing. *Proves: senior judgement on scope.*

## Project implementation

- `templates/golden-path/`, published as its own GitHub template repo `runway-service-template`. It contains:
  - hardened Dockerfile
  - `workload.yaml`
  - `.github/workflows/ci.yml` (about 10 lines, calling `build-sign-push`, `deploy`/Argo promotion and `change-record` at pinned tags)
  - `catalog-info.yaml`
  - `docs/runbook.md` skeleton
  - an OpenAPI stub
  - a standard OTel bootstrap
  - a resilient HTTP-client wrapper with timeouts, retries with jitter and a circuit breaker. This is the permanent fix for the "missing timeouts" problem found in M11. **Operations feeding the platform.**
- `platform/modules/workload`, released as `v1.x`. From one invocation it provisions:
  - **AWS target:** EKS namespace + ResourceQuota/LimitRange; Pod Identity association + least-privilege IAM role; Secrets Manager path + ExternalSecret; optional RDS database/schema + user; optional SQS queue + DLQ; Route 53 record + ALB routing rule; Kyverno policy labels.
  - **Azure target:** AKS namespace; Workload Identity + federated credential; Key Vault access scoped to a prefix; Gateway API HTTPRoute; DNS.
  - **Both targets:** the Argo CD Application (app-of-apps entry), standard dashboards, SLOs (availability + p95 latency by tier), L1/L2 alert rules with runbook links, and mandatory tags.
- `platform/scripts/runway` (Python CLI): `validate`, `render` (shows the generated HCL and manifests for debugging), `diff`, `scaffold`.
- A reusable `onboard` workflow. It validates `workload.yaml` against the schema, renders it, opens a PR against `platform/live/` with the Infracost comment and conftest result, and runs `tf-plan-apply` on merge.
- **`hello-svc` in its own repository**, onboarded **cross-repo** to **both** targets (`target: aws`, then `target: azure`) by changing one field. This is the proof that the path is cloud-agnostic.
- `docs/platform/onboarding-guide.md`, written for a stranger. `docs/platform/golden-path-design.md` states what the path supports, what it deliberately does not (stateful workloads, non-HTTP ingress, GPU) and the escape hatches.
- Optional, if time allows: Argo Rollouts canary as an opt-in `release: canary` contract field, with analysis driven by the SLO metrics from M10.

## Engineering problems to hit deliberately

| Problem | How to cause it | What you should learn |
|---|---|---|
| Template rot | Change a reusable workflow input; the copied template in an old repo breaks | Copy only the thin caller; version what is called |
| Abstraction too thin | Make consumers pass subnet IDs | If users need your internals, you've shipped a wrapper, not a product |
| Abstraction too thick | Hide a failing Terraform error behind a generic "onboarding failed" | Failure modes must stay legible — `runway render` and error passthrough |
| Contract drift | Change the schema without bumping the module version | SemVer is a promise; CI must check contract ↔ module compatibility |
| Policy vs path conflict | Golden path emits something Kyverno denies | Path and policy must be tested together — conformance test in the module's CI |
| Cross-cloud asymmetry | Feature works on `target: aws` but not `azure` | A cloud-agnostic contract needs a conformance suite per target |

## Troubleshooting exercises

1. **S30 — "A new team cannot onboard through the golden path."** Hand `hello-svc` and the onboarding guide to someone else, or to yourself after two weeks away. They report "it doesn't work". Diagnose using only what the platform exposes: PR checks, `runway render`, Argo CD status, Kyverno events. Every question they had to ask you is a defect. Record the time-to-first-deploy and every defect you find. Write it up as Detection → Investigation → Mitigation → Root Cause → Permanent Fix → Prevention.
2. **"v2 of the workload module broke team B."** A consumer upgraded and lost their DNS record because a resource address changed. Recover it (`moved` blocks, or roll back the module pin). Then work out why your module CI didn't catch it and add a plan-diff test against a fixture consumer.
3. **"The golden path made prod slower."** After a platform release, every onboarded service's p95 rose. The cause is in the template's client wrapper defaults. This is a platform change with a blast radius equal to your whole user base. Run it as a change failure: emergency change plus PIR (S35 pattern).

## Deliverables

- [ ] `docs/adr/ADR-021-golden-path-contract.md`
- [ ] `runway-service-template` repository; `templates/golden-path/` source
- [ ] `platform/modules/workload` with README interface docs, JSON Schema, tagged releases `v1.0.0`, `v1.1.0`, `v2.0.0`, and `docs/platform/migrations/v1-to-v2.md`
- [ ] `platform/scripts/runway` CLI with tests
- [ ] Reusable `onboard` workflow; the mandatory SBOM stage rolled out with zero consumer PRs
- [ ] `hello-svc` repo running on **both** EKS and AKS via the path
- [ ] `docs/platform/onboarding-guide.md`, `docs/platform/golden-path-design.md`
- [ ] `docs/platform/metrics.md` — time to first deploy (before/after), manual steps, platform-code touches, version spread
- [ ] `docs/incidents/INC-0xx-onboarding-failure.md` (S30)

## Completion criteria

- An unfamiliar workload in a separate repo reaches production **touching zero platform code**, by changing only `workload.yaml`. The time-to-first-deploy is measured and written down, and the target is **under 60 minutes** of human time, excluding pipeline wall-clock.
- The same service moves between `target: aws` and `target: azure` with a one-field change, and both have dashboards, SLOs, L1/L2 alerts, tags and policy compliance **without anyone asking for them**.
- A new mandatory pipeline stage reached every consumer with **no consumer PRs**.
- A breaking module release was shipped with a migration guide, and all consumers were migrated. Version spread is back to one major version.
- You can defend, without notes: what the path deliberately doesn't support, why the contract has the fields it has, why guardrails enable autonomy, and when you would adopt Crossplane or Backstage templates instead.

## Mentor checkpoints

- **Design review, before any code:** bring the `workload.yaml` schema, the module's README (inputs, defaults, outputs, non-negotiables) and your onboarding inventory. Expect me to delete half your fields.
- **Mid-phase:** bring a `runway render` output and explain every generated resource. If you can't explain one, it shouldn't be generated.
- **Review questions:** "What happens when a team needs something the path doesn't support?" "How do you know people use this?" "What's your deprecation policy, and have you followed it once?"
- **The failure I'll inject after you think you're done:** a consumer whose needs are slightly outside your contract, arriving with a deadline. It tests your judgement on scope, not your Terraform.

## Resources

- ***Platform Engineering* (Fournier & Nowland, 2024):** chapters on platform as product, building the right abstractions, and migrations and sunsetting.
- ***Team Topologies* (Skelton & Pais):** chapters on cognitive load, the four team types (the platform team) and interaction modes.
- **CNCF Platforms White Paper** and the **Platform Engineering Maturity Model** (tag-app-delivery.cncf.io): read both whole; they're short. Use the maturity model to score Runway honestly.
- **Terraform docs:** "Module creation — recommended pattern", "Module composition", "Refactoring (`moved`)". **GitHub Actions docs:** "Reusing workflows" (versioning and `secrets: inherit` caveats).
- **Argo CD docs:** "Cluster bootstrapping: app of apps" and "ApplicationSet" (generating Applications from the contract).

## Estimate vs actual

| | Planned | Actual | Variance note |
|---|---|---|---|
| Hours | 40 | | |
| Cloud cost this phase | $70–100 | | |
