# Milestone M18 — Internal Developer Platform and Self-Service

*Phase 18 · ~4 weeks · Months ~19–20 · Prerequisites: [M17](M17-golden-paths.md) (golden path and contract), [M13](M13-azure-core.md) (Entra ID), [M16](M16-finops.md) (cost data) · Unlocks: [M19](M19-ai-infrastructure.md) (AI quota becomes a self-service request)*

## Why this phase exists

The JD asks for experience building *"internal developer tools, self-service portals"*. It also asks for *"strong full-stack development experience, ideally using technologies such as Python, TypeScript/JavaScript, APIs, React, Node.js"*, and for designing *"REST APIs, backend services, integrations, web-based applications"*. It wants you to *"publish and maintain high-quality documentation and a data catalogue for the technology area"*.

This is the one place in the roadmap where full-stack web development is genuinely justified. You are building internal platform tooling, which is the context the JD implies. It comes after M17 because a portal is a front door onto golden paths. A portal with no paths behind it is only a form that files tickets.

The portal is the safest scope reduction in the roadmap if you fall behind (see [§22](../22-timeline.md)). The **catalogue** and **PR-based provisioning** must stay. The UI can shrink.

## Concepts to learn

**What an internal developer platform (IDP) is.** An IDP is the sum of:
- the golden paths
- the service catalogue
- the platform APIs
- documentation
- the front door, which can be a portal, a CLI or both

The portal is one interface onto the IDP. It is not the IDP itself.

**Self-service, precisely.** Self-service means *without waiting for a human*. It does not mean *without governance*. Three designs, in increasing order of risk:

1. **A request becomes a PR.** Review, policy checks, plan, audit and rollback all stay in the loop. This is what you build.
2. **A request becomes a platform API call.** The API validates the request and writes to Git itself, so the effect is the same as option 1 with better UX.
3. **A request calls the cloud directly.** This bypasses review, policy-at-plan and audit trails. It is an anti-pattern, and you must be able to explain why. Interviewers probe this.

**Service catalogue.**
- Entities (Component, API, System, Resource, Group) with an owner, lifecycle, tier, dependencies, links to runbooks, dashboards and ADRs, and on-call information.
- It uses the Backstage `catalog-info.yaml` format so the data is portable.
- A catalogue is only useful if it is **correct**, so it must be generated from, or validated against, reality. That means CI checks that every deployed namespace has a catalogue entry and every entry has an owner.

**Data catalogue.** This is a JD requirement: *"data catalogue... to ensure data accuracy, accessibility, and governance"*. Each dataset entry records:
- where the data lives
- the owner
- the classification (public, internal, confidential or restricted), and whether it contains PII
- retention, backup and RPO
- lineage (which services read and write it)
- who can access it

Governance uses it for access reviews, DR tiering and deletion requests.

**Platform APIs and backend-for-frontend (BFF).**
- The browser never holds cloud credentials. The BFF holds them.
- **Authorisation code + PKCE** is the flow for the SPA's user sign-in.
- The BFF calls the GitHub API and the Argo CD, Prometheus and cost endpoints using **its own workload identity**, with read-only roles.
- The token belongs to the user when it acts as the user, and to the BFF only when it reads.
- Access control applies at the API layer as well as the UI layer. The same OpenAPI discipline as `cargo-api` applies.

**Full-stack for platform tools.**
- **React:** routing, data fetching, forms with JSON-Schema validation (reusing the `workload.yaml` schema from M17 so the UI and CI validate identically), and accessibility basics.
- **Node/TypeScript:** typed API layer, error handling, timeouts on every outbound call (the same lesson as M11), structured logs and OTel.
- **Testing:** unit tests, contract tests against the OpenAPI spec, Playwright end-to-end tests of the onboarding flow.

**Developer experience as a measured thing.** Track:
- time to first deploy through the portal compared with through the CLI
- abandonment rate on the request form
- time a request spends waiting for review
- support questions per onboarding

**Static delivery.** The SPA is built once and served from S3 behind CloudFront with correct cache headers:
- hashed assets are immutable
- `index.html` has a short TTL
- invalidation happens on release

The CDN is justified here, and you must be able to say why the BFF is *not* behind it.

## Architecture decisions in this phase

**ADR-022 — Developer portal: custom thin portal vs Backstage; PR-based provisioning**

| | |
|---|---|
| **Problem** | Engineers need one place to discover services, data, owners, SLOs, cost and change status, and to request new services, environments, sandbox accounts and AI quota, without learning the platform's internals or waiting on the platform team. |
| **Options** | (a) No portal: CLI plus docs. (b) Backstage with the software catalogue, scaffolder templates and plugins. (c) A custom thin portal (React SPA + Node/TS BFF) over a Backstage-compatible catalogue. (d) Buy a commercial IDP portal. |
| **Decision** | (c) for this roadmap. The catalogue stays in Backstage format, so moving to Backstage later is cheap. Every mutating request opens a PR. The portal never calls cloud mutation APIs. |
| **Trade-offs** | Backstage is what many enterprises would choose, and it brings plugins, community and scaffolder. It also carries a real operational and upgrade burden, and it teaches you plugin wiring rather than the concepts. The custom portal teaches the concepts and exercises the JD's full-stack requirement, but it would not scale to hundreds of teams without rebuilding much of what Backstage already provides. The ADR says honestly: *at work, I would evaluate Backstage first.* |

## Hands-on labs

1. **Catalogue from reality.** Write a script that lists every namespace, Argo CD Application and Terraform root across both clouds, then diffs that against `catalog/`. Fix the gaps. *Proves: a catalogue is a governed dataset, not a wiki.*
2. **Data catalogue audit.** For each datastore (RDS, S3 buckets, Blob containers, Redis, Loki, Tempo, AMP), write an entry. You will find at least one whose retention or backup nobody decided. *Proves: data governance begins with an inventory.*
3. **PKCE by hand.** Walk through an authorisation-code-with-PKCE sign-in against Entra ID with curl and a browser, decode the ID token and the access token, and explain every claim. *Proves: you understand SSO rather than configuring it by rote.*
4. **Direct-API anti-pattern.** In a scratch branch, add a button that creates an S3 bucket directly through the SDK. List everything you lost compared with the PR path: review, conftest, Infracost, plan preview, audit trail, rollback, drift baseline. Delete the branch. *Proves: you can argue the design.*
5. **Cache headers.** Deploy the SPA with wrong caching (`index.html` cached for one day), ship a release, and watch users get stale code. Fix the headers and invalidation. *Proves: CDNs have failure modes too.*
6. **Backstage spike (timeboxed to 4 hours).** Run Backstage locally against your `catalog/`. Record what worked without changes and what you would have to build. Feed the results into ADR-022. *Proves: your decision rests on evidence.*

## Project implementation

**`workloads/runway-portal/`**: SPA on S3 + CloudFront; BFF on AKS, onboarded through **your own golden path**, with `target: azure`. Entra ID SSO. Entra groups map to portal roles: `platform-admins`, `service-owners`, `readers`.

**Read views:**
- **Service catalogue:** owners, tier, dependencies, and links to runbooks, dashboards and ADRs.
- **Data catalogue:** classification, retention, RPO, and readers and writers.
- **Deployments:** Argo CD sync and health status for both clusters.
- **SLO status and error budget:** read from AMP.
- **Cost per service and team this month:** read from `finops-collector` (showback, M16).
- **Change calendar:** change records and freeze windows (M11).
- **EOL register:** `docs/lifecycle/eol-register.md`, with dates approaching highlighted.

**Self-service requests.** Each request is a form validated against the relevant JSON Schema, and each opens a PR:
- **New service:** renders `workload.yaml` plus a repo from `runway-service-template`.
- **New environment** for an existing service.
- **Sandbox AWS account:** the account-vending module `aws-account` in the `sandbox` OU, with a budget, a TTL tag and auto-expiry.
- **AI quota:** creates the entry that ai-gateway consumes in M19. The stub is added now.
- **Request status:** the portal tracks each PR's checks, plan and review state back to the requester.

**BFF identity and security:**
- Uses its workload identity with read-only roles.
- Uses a GitHub App with contents and pull-requests write access on the `runway` repo **only**.
- Has rate limits and audit logs of every request.
- Is protected by an OpenAPI contract, contract tests and Playwright end-to-end tests in CI.

**Catalogue governance.** A CI check fails if:
- a deployed workload has no `catalog-info.yaml`
- a catalogue entry has no owner
- a datastore has no `catalog/data/` entry

**Documentation:**
- `docs/platform/portal.md`
- `docs/platform/self-service-model.md` (what can be requested, who approves what, and SLAs for review)
- `docs/platform/data-catalogue.md`

## Engineering problems to hit deliberately

| Problem | How to cause it | What you should learn |
|---|---|---|
| Catalogue drift | Deploy a service by hand without catalogue entry | Catalogues rot unless validated against reality in CI |
| Token confusion | Have the BFF call GitHub with the user's token for reads and its own for writes, inconsistently | Be deliberate about acting-as-user vs acting-as-service; audit shows which |
| Approval bottleneck | Require platform-team review on every request; queue five requests | Self-service with a human gate is a ticket queue; auto-approve low-risk changes by policy (standard changes, ADR-013) |
| Sandbox sprawl | Vend three sandbox accounts, forget them | Every self-service resource needs an owner, budget and expiry |
| Stale SPA | Wrong `Cache-Control` on `index.html` | Hashed assets immutable, entry point short TTL, invalidate on deploy |
| Slow dependency in the BFF | Make the Argo CD API slow | Timeouts, partial rendering and degraded views, same as any service |

## Troubleshooting exercises

1. **"The portal says my service is healthy, but it isn't."** The portal shows Argo CD `Healthy` while users get 5xx. Investigate: Argo CD health only reflects Kubernetes resource status. The SLO view is what shows user pain. Fix the portal so SLO status leads, and write down what "healthy" means on your platform.
2. **"My onboarding request has been stuck for two days."** The PR is waiting on a check that never ran because the GitHub App lost a permission after a secret rotation. Diagnose from the portal's request status and the audit log. Permanent fix: an alert for "requests pending more than 4h", plus a runbook.
3. **"A sandbox account is costing $40 a day."** An expired sandbox wasn't reclaimed. Tie this to M16's `idle-detector`, add a TTL enforcement job, and treat it as S29 with a known cause.
4. **"Portal is down."** The BFF can't reach Key Vault after a network change (S24 pattern). The on-call engineer should find it from the portal's own dashboards, which are generated by the golden path. If that isn't true, the portal isn't really on the golden path.

## Deliverables

- [ ] `docs/adr/ADR-022-developer-portal.md` including the Backstage spike findings
- [ ] `workloads/runway-portal/` (SPA + BFF, OpenAPI, unit/contract/Playwright tests), onboarded via the golden path
- [ ] `catalog/` complete for every component; `catalog/data/` complete for every datastore; CI catalogue checks
- [ ] Self-service flows: new service, new environment, sandbox account (with budget + TTL), AI quota stub
- [ ] `platform/modules/aws-account` (account vending) used by the sandbox flow
- [ ] `docs/platform/portal.md`, `docs/platform/self-service-model.md`, `docs/platform/data-catalogue.md`
- [ ] `docs/platform/metrics.md` updated: portal vs CLI time-to-first-deploy, request lead time
- [ ] One incident record from the troubleshooting set

## Completion criteria

- Someone unfamiliar with Runway requests a new service **through the portal** and reaches production without messaging you. Record their elapsed time next to the CLI number from M17.
- **100%** of deployed workloads have catalogue entries with owners, and **100%** of datastores have data catalogue entries with classification, retention and RPO. CI fails otherwise.
- No path from the portal mutates cloud resources except through a merged PR. You can show this with the BFF's IAM, Entra and GitHub App permissions.
- A vended sandbox account has a budget, an owner and a TTL, and it is removed automatically on expiry. You have tested the expiry.
- You can explain in under three minutes: PKCE, why the browser never holds cloud credentials, and why PR-based provisioning beats direct API calls, while also naming what it costs in latency.

## Mentor checkpoints

- **Before building:** bring wireframes of three screens plus the BFF's OpenAPI spec, and a table of which identity each call uses and why.
- **Mid-phase:** bring the catalogue diff (Lab 1) and the data catalogue gaps you found. I will ask who owns each gap.
- **Review questions:** "If the platform team disappears for a week, what still works?" "Which requests should auto-approve, and on what evidence?" "Why not Backstage?" Answer with your spike data, not preference.
- **The failure I will inject after you think you are done:** something that makes the portal show something reassuring and wrong. The category is a view that trusts a proxy signal.

## Resources

- **Backstage docs:** "Software Catalog → Descriptor Format" (entity kinds, `spec.owner`, relations) and the "Software Templates" overview. Read these even though you are not adopting Backstage.
- **Microsoft identity platform docs:** "OAuth 2.0 authorization code flow" (with PKCE) and "Microsoft identity platform and the On-Behalf-Of flow". Read the second to understand what you chose not to do.
- **GitHub docs:** "About GitHub Apps" and "Choosing permissions for a GitHub App". Least privilege for automation that writes to Git.
- **CNCF Platform Engineering Maturity Model:** the "Interfaces" and "Measurement" aspects. Score your portal against them.
- ***Platform Engineering* (Fournier & Nowland):** chapters on user research for internal platforms and on measuring platform success.

## Estimate vs actual

| | Planned | Actual | Variance note |
|---|---|---|---|
| Hours | 40 | | |
| Cloud cost this phase | $70–100 | | |
