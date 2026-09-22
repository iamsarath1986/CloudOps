# Step 10 — Platform Engineering and Self-Service

The JD asks for *"experience supporting cloud workload onboarding, automation and self-service capabilities."* This is the requirement almost no self-taught candidate can evidence, and it is therefore where you should spend disproportionate effort. This file explains the concepts; [Milestone 13](milestones/milestone-13-golden-path.md) builds it.

---

## 1. The idea

A platform team does not build infrastructure for applications. It builds **a product whose users are other engineers**, and whose value is that those engineers stop needing to know what you know.

The measure is not how sophisticated the platform is. It is:

> **How long does it take an engineer who has never met you to get a new service running in production, correctly configured, observed and governed?**

Everything else is in service of that number. Write it in your README.

## 2. The onboarding flow

```
Developer
   ↓  copies the template repository
Git repository            ← Dockerfile, workload.yaml, CI workflow, README
   ↓  opens a PR against the platform repo (via the portal, or by hand)
Infrastructure template   ← one `workload` module invocation, reviewed and applied by pipeline
   ↓
CI/CD template            ← reusable workflows: build, scan, push by digest, deploy, roll back
   ↓
Container registry        ← the artifact, scanned, referenced by digest
   ↓
Kubernetes runtime        ← namespace, quota, identity, secrets, ingress, DNS, TLS
   ↓
Observability             ← dashboards, alerts and SLOs created automatically, not requested
```

Two properties make this a platform rather than a pile of scripts:

1. **The developer supplies intent, not implementation.** `workload.yaml` says *"I am an HTTP service, I need a database, my owner is this team, here is my health endpoint."* It does not say `azurerm_kubernetes_cluster`.
2. **Compliance is a side effect of using the path.** Tags, network policy, resource limits, identity, telemetry and policy conformance arrive because you took the road, not because you remembered.

## 3. The concepts

### Golden paths
An opinionated, supported, well-documented route that covers roughly 80% of cases. The other 20% is not a failure — it is what escape hatches are for. The two ways to get this wrong: making the path so narrow that most teams cannot use it, or so configurable that it stops being a path. **Both failures look like low adoption**, which is why adoption is the metric that matters.

Document explicitly: what the path supports, what it does not, and how to get help when you are outside it.

### Reusable Terraform modules
A module is an API. Design it in that order:

- **Inputs:** the minimum a consumer must supply. If a consumer must set eleven variables to get a service running, you have exposed implementation rather than intent.
- **Defaults:** where your opinions live. Sensible defaults are the product.
- **Outputs:** what consumers need downstream — never internal resource IDs they should not depend on.
- **Versioning:** semantic versioning is a promise. A change that forces a consumer to edit their code is major, even if it feels small to you.
- **Non-negotiables:** tags, network policy, resource limits, the log format. These are not variables.

The hard part of platform work is not writing `v1`. It is releasing `v2` and migrating consumers you do not control. Practise it deliberately in Milestone 13 by breaking your own module and writing the migration guide.

### CI/CD templates
Reusable workflows rather than copy-paste. When a consumer's pipeline is six lines that call your workflow, you can improve the build for everyone at once — add SBOM generation, tighten a scan threshold, change the deployment strategy. When it is 200 copied lines, you can improve nothing.

The test: **can you add a new mandatory pipeline stage without opening a PR in any workload repository?**

### Environment configuration
One definition, parameterised per environment. Environment differences are data (sizes, replica counts, domains), never separate code paths — the moment dev and prod are described by different code, dev stops predicting prod, and your pipeline's testing stops meaning anything.

### Secrets
Onboarding must provision identity and secret access, not leave them as a follow-up ticket. In this platform: the workload module creates a managed identity, federates it to the Kubernetes service account, and grants Key Vault access scoped to that workload's secret prefix. The developer never sees a credential, and there is no step where a human copies one.

### Policy enforcement
Guardrails are what allow you to grant autonomy. Because Azure Policy denies untagged resources, public IPs and disallowed regions, you can let teams provision freely inside the path. **Policy is not the opposite of self-service; it is its precondition.**

Enforce at three points, deliberately: at PR time (`conftest` on the plan — fastest feedback), at apply time (Azure Policy — authoritative), and at admission (Kubernetes — last line). Know which failures each catches.

### Standardised telemetry
Because the workload module creates dashboards and alerts, observability is a property of being on the platform. Because the reusable workflow injects OpenTelemetry configuration and the log schema, every service's logs are queryable together. Cross-service queries are impossible without this, and cross-service queries are most of what an operator does.

### Developer self-service
Self-service means *without waiting for a human*. It does not mean *without governance*. Your portal's onboarding form opens a pull request rather than calling the Azure API directly — which keeps review, policy, audit and rollback in the loop while still removing the ticket. Be able to explain that choice; it is a genuinely senior design decision and interviewers will probe it.

## 4. What to measure

| Metric | Why | Where |
|---|---|---|
| **Time to first production deploy** for a new service | The headline number. Everything else is diagnosis | `hello-svc` onboarding, timed |
| Manual steps remaining in onboarding | Each one is a defect in the path | Your honest Milestone 13 list |
| Platform-code touches per onboarding | Should be zero. Anything above zero is a leak in the abstraction | Git history |
| Module version spread across consumers | Measures your migration debt | A report over the three repos |
| Change failure rate on platform-managed deploys | Is the path actually safer than hand-rolling? | Deployment records |

## 5. Anti-patterns to avoid on purpose

| Anti-pattern | Why it fails |
|---|---|
| The platform as a ticket queue with extra steps | If a human must act for each request, it is not self-service |
| A wrapper so thin it adds nothing | If consumers must understand Terraform to use your module, they will skip it |
| A wrapper so thick nobody can debug it | When production breaks, someone must see through the abstraction. Keep failure modes legible |
| Mandatory adoption by decree | Adoption by usefulness survives reorganisations; adoption by mandate does not |
| A portal that mutates infrastructure directly | Bypasses review, policy and audit. Generate a PR instead |
| No deprecation policy | Every version you have ever shipped lives forever and you maintain all of them |

## 6. What you should be able to say in an interview

> *"I built a golden path where a developer copies a template repository, fills in a five-field manifest, and opens a PR. That produces a namespace with quota, a workload identity federated to the service account, Key Vault access, DNS, TLS, ingress, standard dashboards and alerts, and policy-conformant tags. I onboarded a service I had not written, in a separate repository, without touching platform code, in <N> minutes. The path deliberately does not support stateful workloads or non-HTTP protocols, and here is the escape hatch for those. When I released v2 of the workload module I wrote a migration guide and moved the three consumers myself, because migrating consumers is the actual work."*

Very few candidates for a mid-level cloud role can say anything close to that paragraph.

→ Next: [Step 11 — AI in cloud operations](11-ai-in-cloud-operations.md)
