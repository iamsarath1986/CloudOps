# Milestone 13 — The Golden Path

*Phase 13 · ~4 weeks · Prerequisite: Milestones 7, 8, 9, 12*

> **The JD's differentiating requirement** — *"supporting cloud workload onboarding, automation and self-service capabilities"* — and the centrepiece of your portfolio. Almost no candidate can evidence this at all.

**Learning topics**
* The platform as a product: internal engineers are users, adoption is voluntary in practice, and a platform slower than doing it by hand gets routed around
* **Golden paths**: an opinionated, supported route covering 80% of cases, with documented escape hatches
* Paved road vs guardrail: defaults that are right, limits that are enforced
* Module design as API design; semantic versioning as a promise to consumers
* Template repositories, scaffolding, cross-repository reusable workflows
* Environment configuration and promotion; secret provisioning as part of onboarding rather than a follow-up ticket
* Policy enforcement as the thing that lets you *give* autonomy safely
* Standardised telemetry, so observability is a property of being on the platform rather than a task
* **Time to first deploy** as the platform's primary metric; documentation as interface; deprecation and migration as the hard part

**Build**
A **template repository** containing a Dockerfile, a `workload.yaml` manifest, a CI workflow calling your reusable workflows, and a README. `platform/modules/workload` provisioning, from one invocation: a namespace with quota, a managed identity with workload-identity federation, Key Vault access, a DNS record, an ingress rule, a database schema if requested, standard dashboards, standard alerts, and correct tags. `runway-portal` — React SPA + Node/TypeScript BFF — with an "Onboard a service" form that **opens a pull request**, never calling Azure directly, plus read-only views of the service catalogue, deployment status, SLO status and cost per service.

**Cloud resources**
No new resource *types* — the point is that one module invocation now produces a correct, governed, observed slice of everything you have built.

**DevOps work**
Split `hello-svc` into its own repository and onboard it **cross-repo**. Cross-repository reusable-workflow consumption is what proves the path is real rather than a monorepo convenience. Version the workload module, release a breaking `v2`, and write the migration guide.

**Operations exercise**
Start by writing down, honestly, every manual step you currently perform to onboard a workload — the list will be longer than you expect, and it is the specification. Then onboard `runway-portal` through a draft path and record every place you had to touch platform code; each one is a defect. Then **time the `hello-svc` onboarding**. Improve the path. Time it again.

**Security considerations**
The portal's BFF holds the Azure credential; the browser never does. It uses Entra ID SSO and holds `Reader` for its read views. Provisioning goes through PR review and the pipeline, so there is no button that mutates infrastructure directly — that boundary is the whole design.

**Expected outcome**
Someone who has never seen your platform can take the template repository and reach a running, observed, governed production service by following `docs/platform/onboarding-guide.md`.

**Evidence of mastery**
Onboard an unfamiliar workload through your own path, touching no platform code, and **state the elapsed time**. Explain what your path deliberately does not support and why. Explain module versioning and how you would deprecate `v1` across consumers you do not control.
