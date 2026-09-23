# Section 17 — Security & Governance Learning Path

*The spine of "safe by default" across two clouds: identity first, secrets never stored, networks private by default, a signed supply chain, policy at four enforcement points, and landing zones that let you grant autonomy without losing control.*

---

## 1. What the JD actually asks for

> *"Strong knowledge of cloud concepts and best practices, including: Security · Governance · Resilience · FinOps"*
> *"...drive the adoption of ... governance, and FinOps practices while ensuring solutions meet required standards for availability, performance, security, reliability, and operational support."*
> *"Ensure that solutions to problems meet architectural and security standards within the technology area."*
> *"Experience operating large public cloud footprints and onboarding applications using governed and automated DevOps practices."*
> *"...ensure data accuracy, accessibility, and governance across systems."*

The word that recurs is **governed**. At enterprise scale, security is not a set of settings — it is a system of guardrails that lets hundreds of teams move quickly without each one having to be a security expert. A senior platform engineer is expected to *design* that system: identity model, account/subscription structure, policy hierarchy, exception process, evidence.

**Depth expected:** Production Proficiency on IAM, secrets, network security and supply chain; Advanced on policy-as-code and landing-zone governance; Working Knowledge on threat modelling and AI security (rising to Production Proficiency by M22).

## 2. Principles that organise the track

1. **Identity is the perimeter.** Every resource decision from M04 onward starts with "which identity, with what permission, at what scope?"
2. **No static credentials, including across clouds** — the recurring theme of Runway's architecture (ADR-017). OIDC federation for CI, Pod Identity/Workload Identity for pods, federated credentials for EKS→Entra and AKS→AWS.
3. **Private by default.** Stateful services have no public endpoint. Public exposure is an explicit, reviewed exception.
4. **Enforce at the earliest point that can see the change, verify at the last.** PR → cloud API → cluster admission → runtime.
5. **Audit-first, then deny.** A `Deny` without a compliance baseline breaks other people's deployments and your credibility (S28).
6. **Accepted risk is a legitimate, written outcome.** The honest entries in a threat model are the ones you chose not to fix.

## 3. The progression

| Stage | Theme | Milestones |
|:-:|---|---|
| 1 | Secrets hygiene and Git history | [M00](milestones/M00-foundations.md) |
| 2 | Linux and network fundamentals of security | [M01](milestones/M01-linux-networking.md) |
| 3 | AWS identity-first foundation | [M04](milestones/M04-aws-core.md) |
| 4 | Security in IaC | [M05](milestones/M05-terraform.md) |
| 5 | Image and supply-chain basics | [M06](milestones/M06-containers.md), [M07](milestones/M07-cicd.md) |
| 6 | Kubernetes security | [M08](milestones/M08-kubernetes-fundamentals.md), [M09](milestones/M09-eks.md) |
| 7 | Enterprise identity and Azure governance | [M13](milestones/M13-azure-core.md), [M14](milestones/M14-aks-multicluster.md) |
| 8 | Multi-cloud security and governance | [M15](milestones/M15-security-governance.md) |
| 9 | Governance as the precondition for self-service | [M17](milestones/M17-golden-paths.md), [M18](milestones/M18-developer-platform.md) |
| 10 | AI security | [M19](milestones/M19-ai-infrastructure.md), [M20](milestones/M20-ai-agents-mcp.md) |
| 11 | Hybrid control and final hardening | [M21](milestones/M21-hybrid-multicloud.md), [M22](milestones/M22-hardening-capstone.md) |

### Stage 1 — Secrets and history · M00
Commit a fake secret; remove it from history; understand why it is **still compromised** and must be rotated. gitleaks as a pre-commit and CI check.

### Stage 2 — Fundamentals · M01
File permissions and ownership; users and sudo; SSH keys; stateful firewalls as allow-lists; TLS, certificate chains and trust stores; why outbound (egress) is the direction that matters for exfiltration.

### Stage 3 — AWS identity-first · M04
- Root account lockdown, MFA, IAM Identity Center with permission sets; **no IAM users with access keys**.
- **Policy evaluation logic** — explicit deny > SCP > permission boundary > session policy > identity/resource policy. Learn to read an `AccessDenied` and name which layer denied it (S04).
- Roles and trust policies; the `sub`/`aud` conditions that make OIDC federation safe; least privilege derived empirically (start from zero, add what fails, then tighten with IAM Access Analyzer).
- Organizations and OUs (ADR-003); **SCPs as guardrails** (deny leaving the org, deny disabling CloudTrail, restrict regions); organisation CloudTrail to the `security` account.
- Security groups vs NACLs; private subnets; VPC endpoints so traffic to S3/ECR/STS never leaves AWS.

### Stage 4 — Security in IaC · M05
Terraform state contains secrets — encrypt it, restrict it, version it. Never put secret values in variables; reference them. Static analysis (trivy config / checkov) and `prevent_destroy` on data. Tagging including `data-classification` from day one.

### Stage 5 — Supply chain · M06–M07
Non-root, minimal images; pinned digests; SBOM (Syft); scanning (Trivy) with a written triage policy and CVE SLA by severity; cosign signing; OIDC-only CI identities scoped per repository and environment.

### Stage 6 — Kubernetes security · M08–M09
Kubernetes Secrets are base64, not encryption (enable envelope encryption with KMS on EKS). RBAC in the cluster; EKS access entries vs the old `aws-auth` ConfigMap. **Pod Identity** (and IRSA) for AWS access — no node-role over-privilege (S14). **External Secrets Operator** with Secrets Manager (ADR-011). Pod Security Standards (`restricted`); default-deny NetworkPolicy; read-only root filesystem; dropped capabilities; private API endpoint or restricted CIDRs.

### Stage 7 — Enterprise identity and Azure governance · M13–M14
- **Entra ID** as the workforce IdP: users, groups, app registrations, service principals, managed identities; Conditional Access and PIM at the level of "what problem each solves".
- **Entra → AWS IAM Identity Center federation** (SAML + SCIM) — one identity, two clouds (ADR-016). Break-glass accounts that are monitored.
- **Azure RBAC** scope inheritance (management group → subscription → resource group → resource) vs AWS's account boundary model — the most important conceptual difference between the clouds.
- Management groups and **Azure Policy** (effects: Audit, Deny, DeployIfNotExists, Modify); initiatives; exemptions with expiry.
- Private endpoints + private DNS zones (S24); Key Vault with RBAC, soft delete, purge protection; AKS Workload Identity; cross-cloud federation both directions (S25; ADR-017).

### Stage 8 — Multi-cloud security and governance · M15
The consolidation milestone. See §4 (enforcement points) and §5 (threat model). Also: the **leaked-credential drill** (S27) end to end — contain, rotate, assess exposure from CloudTrail/Activity Log, prevent; detection with GuardDuty and Defender for Cloud (free/foundational tiers, timeboxed) and what each would alert on; egress restriction with an allow-list and discovering the dependencies you did not know you had.

### Stage 9 — Governance enables self-service · M17–M18
Policy is not the opposite of self-service; it is its **precondition**. Because untagged resources, public IPs, unsigned images and disallowed regions are denied, you can let teams provision freely inside the golden path. Account/sandbox vending with guardrails pre-attached. The data catalogue (`catalog/data/*.yaml`) records owner, classification, PII, retention and backup for every dataset — the JD's "data catalogue ... governance across systems".

### Stage 10 — AI security · M19–M20
Model output is untrusted input: validate against a schema before it touches anything. Redact before the call, not after. Managed identity to Azure OpenAI; private endpoint; no keys. Prompt injection through tool results (log content is attacker-influenced). Three independent read-only mechanisms for flightdeck: identity, tool surface + CI test, audit (ADR-025). OWASP Top 10 for LLM Applications as the checklist.

### Stage 11 — Hybrid control and hardening · M21–M22
One policy set across cloud and "datacentre" via Arc and SSM; VPN pre-shared keys and their rotation; final STRIDE review, controls matrix, rotation schedule, and a defended list of accepted risks.

## 4. Policy at four enforcement points (ADR-019)

| # | Point | Tool | Catches | Speed of feedback | Can be bypassed by |
|:-:|---|---|---|---|---|
| 1 | **Pull request** | conftest (OPA/Rego) on `terraform show -json`, Kyverno CLI on manifests, trivy config | Most misconfigurations, before anything exists | Seconds, in the PR | Anyone applying outside the pipeline |
| 2 | **Cloud control plane** | AWS SCPs (+ AWS Config rules, timeboxed), Azure Policy | Anything that reaches the cloud API, from any tool | At apply time | Nothing, within scope — **authoritative** |
| 3 | **Cluster admission** | Kyverno (signed images, no `:latest`, required labels, restricted pod security, resource limits) | Workload-level violations, including those created by operators and humans with `kubectl` | At deploy time | Cluster-admin with an exception |
| 4 | **Runtime / detective** | GuardDuty, Defender for Cloud, CloudTrail/Activity Log queries, drift detection | What slipped through or changed afterwards | Minutes to hours | — (it only detects) |

Know which failure each catches that the others miss. That table *is* the senior interview answer to "how do you govern a large cloud estate?".

## 5. Threat modelling

STRIDE over Runway's architecture in M15, revisited in M22, organised by trust boundary: internet ↔ edge, edge ↔ cluster, cluster ↔ managed data, AWS ↔ Azure, CI ↔ cloud, platform ↔ tenant namespace, human ↔ AI agent, cloud ↔ datacentre. For each threat: control, evidence (a path in the repo), or **accepted risk with a reason**. Keep it in `docs/security/threat-model.md` with a controls matrix.

## 6. Checkpoints

| By | You can |
|---|---|
| M04 | Given an `AccessDenied`, name which policy layer denied it and fix it with the narrowest change |
| M09 | Show that no pod, pipeline or person holds a long-lived AWS credential |
| M13 | Explain AWS's account boundary model vs Azure's scope inheritance model with a concrete consequence of each |
| M15 | Show a policy that blocked a real deployment at each of the four points; run the leaked-credential drill end to end |
| M18 | Show a sandbox account vended with guardrails attached, via a PR |
| M20 | Name three independent mechanisms that stop flightdeck changing infrastructure |
| M22 | Walk through the threat model and defend two accepted risks |

## 7. Resources

1. **AWS IAM User Guide** — "Policy evaluation logic" (with the flowchart) and "IAM roles → Roles terms and concepts"; **AWS Organizations** — "Service control policies". Before M04.
2. **Microsoft Cloud Adoption Framework** — "Ready → Azure landing zone → Design areas" (identity and access, resource organisation, governance); **Azure Policy docs** — "Understand Azure Policy effects". Before M13/M15.
3. **Kubernetes docs** — "Security → Pod Security Standards", "RBAC Good Practices"; **Kyverno docs** — "Verify Images" and "Policy Reports". Before M09/M15.
4. ***Threat Modeling: Designing for Security*** (Shostack) — Part I (ch. 1–4) only; enough to run STRIDE honestly.
5. **OWASP Top 10 for LLM Applications** — read LLM01 (prompt injection), LLM02 (sensitive information disclosure), LLM05 (improper output handling), LLM06 (excessive agency) before M19.

→ Next: [Section 18 — Platform Engineering Learning Path](18-platform-engineering-learning-path.md)
