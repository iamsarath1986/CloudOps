# Milestone M15 — Cloud Security and Governance (multi-cloud)

*Phase 15 · ~4 weeks · Months ~16.2–17 · Prerequisites: [M14](M14-aks-multicluster.md), [M11](M11-sre-operations.md) · Unlocks: [M16](M16-finops.md), [M17](M17-golden-paths.md), [M21](M21-hybrid-multicloud.md)*

---

## Why this phase exists

The JD asks for *"strong knowledge of cloud concepts and best practices, including Security, Governance, Resilience, FinOps"*, and for solutions that *"meet architectural and security standards"* and are *"conforming to corporate standards"*, delivered with *"governed and automated DevOps practices"*. In a platform team, governance is not a document. It is **the set of controls that makes self-service safe**. Policy is the precondition for M17's golden paths, not the opposite of them.

It sits here because you now have both clouds, both clusters, a delivery pipeline and an incident process. Security controls applied earlier would have been retrofitted to an architecture that was still changing. Applied now, they are enforced on a stable estate, and the incident machinery (M11) handles the security drills.

## Concepts to learn

**Threat modelling and defence in depth**
- **STRIDE** per trust boundary: internet → edge, edge → cluster, pod → pod, pod → cloud API, CI → cloud, cloud → cloud (federation), human → consoles, and the AI tier later. A data flow diagram with trust boundaries is the input.
- Layers: identity, network, workload, data, supply chain, detection. Prevention versus detection, and why you need both.

**Identity and access (hardening what exists)**
- Least privilege at scale: AWS IAM Access Analyzer (unused access, policy generation from CloudTrail), permission boundaries, and **SCPs** (and the newer resource control policies, at concept level) as organisational guardrails. Azure: custom roles, PIM, and deny assignments.
- Break-glass accounts: sealed, monitored, tested quarterly.
- Separation of duties: the author of a Terraform change cannot approve its prod apply (GitHub environment rules + CODEOWNERS).

**Network security**
- Private-by-default: no public IPs on nodes, private API endpoints, private endpoints/VPC endpoints for PaaS. **Egress control**: exfiltration goes outbound. AWS: security group egress, NAT + DNS firewall (Route 53 Resolver DNS Firewall, concept + timeboxed lab). Azure: NSG + UDR to a firewall (concept, because of cost). Kubernetes: default-deny egress NetworkPolicy with FQDN policies (Cilium on AKS).
- WAF: managed rule groups, detection versus prevention mode, false positives, rate-based rules.

**Workload and supply-chain security**
- Pod Security Standards `restricted`, read-only root FS, dropped capabilities, seccomp `RuntimeDefault`, no host namespaces, no privileged containers.
- **Supply chain:** SBOM (Syft), vulnerability scanning (Trivy) with a **triage policy** and SLA per severity, image signing (**cosign**, keyless via GitHub OIDC or KMS-backed), provenance (SLSA levels at concept depth), **admission verification with Kyverno** (only signed images from your registries, digest-pinned).
- Secrets: rotation versus revocation, detection (secret scanning in Git, GitHub push protection), and short-lived credentials everywhere you can.

**Governance as code**
- **Enforcement points** and what each catches:
  - PR time: **conftest/OPA** on the Terraform plan JSON, and checkov/trivy config scan. Fastest feedback.
  - Cloud control plane: **SCPs** (AWS: deny leaving the org, deny disabling CloudTrail/GuardDuty, region allow-list, deny root) and **Azure Policy** (Deny, Audit, Modify, DeployIfNotExists; initiatives at management group scope). Authoritative.
  - Kubernetes admission: **Kyverno** (validate, mutate, verifyImages, generate). Last line in the cluster.
  - Continuous: AWS Config rules / Security Hub standards, Azure Policy compliance, Defender for Cloud (free tier CSPM). Detection of what slipped through.
- **Audit → deny rollout**, exemptions with expiry and an owner, and the exception process as a governance artefact.
- Audit logging: CloudTrail organisation trail to the `security` account (immutable, S3 Object Lock), Azure Activity Log, Kubernetes audit logs. Who did what, when, from where.
- Detection: GuardDuty (including EKS protection) and Defender for Cloud / Defender for Containers at concept level + free-tier use. Routing findings into your EMS tiers (M11).
- Compliance frameworks at awareness level (CIS benchmarks for AWS/Azure/Kubernetes, ISO 27001 controls mapping). Evidence collection as a side effect of automation.

## Architecture decisions in this phase

**ADR-019 Policy enforcement points: SCP, Azure Policy, conftest, Kyverno.**

| | |
|---|---|
| **Problem** | Guardrails must stop non-compliant changes early, authoritatively, and in both clouds and both clusters, without blocking legitimate work or duplicating logic chaotically |
| **Options** | (a) Cloud-native only (SCP + Azure Policy). (b) Pipeline-only (conftest), which is bypassed by any out-of-band change. (c) **Layered: conftest at PR, SCP/Azure Policy at control plane, Kyverno at admission, Config/Defender for detection**, with a single control catalogue mapping each control to its enforcement point(s) |
| **Decision** | (c). `docs/security/controls-catalogue.md` is the source of truth: control → threat → enforcement point(s) → evidence → owner. Every new policy starts in Audit/`validationFailureAction: Audit` for ≥ 1 week with a compliance report before Deny/Enforce |
| **Trade-offs** | The same intent is expressed in 3 languages (Rego, SCP JSON / Azure Policy JSON, Kyverno YAML). You accept the duplication for defence in depth and fast feedback, and keep it manageable through the catalogue and tests for every policy. Policies can break deploys, which is why S28 exists |

## Hands-on labs

1. **STRIDE the whole system.** Draw the DFD with trust boundaries and enumerate threats per boundary. For each threat: the control that addresses it and its evidence, or **"accepted risk" with the reason**. The accepted-risk entries are the honest ones.
2. **SCPs.** Apply a region allow-list and a "deny disabling CloudTrail/GuardDuty" SCP to the workloads OU. From a workloads account admin role, try both. Then find a legitimate global service your region deny broke (IAM, Route 53, CloudFront, which are us-east-1 based) and fix it with the proper exemption pattern.
3. **Azure Policy audit → deny.** Author custom definitions (required tags, deny public IP in landing zones, require private endpoints for Key Vault/Storage, AKS must disable local accounts) as an initiative at the `landing-zones` MG. Run Audit, read the compliance report, fix the findings, then flip to Deny and try to violate.
4. **conftest on plans.** Rego policies over `terraform show -json`: no `0.0.0.0/0` ingress except on ALB listeners, mandatory tags, no unencrypted storage, no public RDS, and no IAM `*:*`. Unit-test the policies (`opa test`). Violations fail the PR with an actionable message.
5. **Sign and verify.** cosign-sign images in the `build-sign-push` workflow (keyless via GitHub OIDC) and generate/attach the SBOM. A Kyverno `verifyImages` policy on both clusters: only signed images from ECR/ACR, digest-pinned. Try deploying an unsigned image, then a signed image referenced by tag.
6. **Vulnerability triage.** Deploy an image with a known critical CVE, get blocked by the pipeline, and apply your triage policy: fix, accept with expiry, or not applicable, recorded in `docs/security/vuln-exceptions.md`.
7. **Egress lock-down.** Default-deny egress on the `cargo` and `portal` namespaces. Allow only what is needed (FQDN rules on AKS via Cilium, CIDR/SG plus VPC endpoints on EKS). Discover every outbound dependency you did not know you had.
8. **WAF tuning (timeboxed).** AWS WAF managed rules in Count mode on the ALB during a k6 run with realistic payloads. Find the false positives, tune, then switch to Block. Add a rate-based rule.
9. **Audit trail.** Answer, from logs alone: "who changed the RDS security group last Tuesday, from which IP, via which role, and was it through the pipeline?" Then do the same for an Azure NSG and for a Kubernetes RoleBinding.

## Project implementation

- `platform/policy/`: `aws-scp/`, `azure-policy/` (definitions, initiatives, assignments, exemptions with expiry), `conftest/` (Rego + tests), `kyverno/` (cluster policies, delivered by Argo CD as a fleet add-on), and **`controls-catalogue.md`**.
- Organisation CloudTrail → `security` account S3 with Object Lock. GuardDuty (incl. EKS runtime/audit monitoring as budget allows) and Security Hub foundational standard, delegated to `security`. Defender for Cloud CSPM (free) on Azure subscriptions. Findings → EventBridge / Azure Monitor → **ems-bridge** with `tier: L2`, `category: security`.
- Pipeline: gitleaks/GitHub secret scanning + push protection; Trivy (image + IaC); Syft SBOM; cosign signing; conftest; checkov. All results attached to the change record (M11).
- Clusters: PSA `restricted` everywhere, Kyverno policies (signed images, no `latest`, required labels/owner, disallow privileged/host mounts, require requests/probes), default-deny ingress + egress with explicit allows.
- IAM hardening pass: Access Analyzer unused-access findings resolved, permission boundaries on roles the golden path will create (M17), break-glass accounts in both clouds (sealed, alarmed on use).
- `docs/security/`: `threat-model.md` (DFD + STRIDE + accepted risks), `controls-catalogue.md`, `vuln-policy.md`, `secret-rotation.md` (every secret, owner, rotation method, last rotated), `incident-response-security.md`.

## Engineering problems to hit deliberately

| Problem | How to cause it | What you should learn |
|---|---|---|
| Your own SCP breaks global services | Naive region deny | Global services and exemption patterns |
| Deny without baseline breaks every team | Flip Azure Policy straight to Deny | Audit-first rollout; the exception process |
| Kyverno blocks the platform's own add-ons | Signature policy with no exclusion for third-party charts | Policy scope and trusted publishers |
| Egress default-deny breaks token exchange | Blocked STS/Entra endpoints | Identity *is* egress. Map it |
| Signed image, still rejected | Referenced by tag, policy requires digest | Why digests matter for verification |
| Scanner noise paralyses delivery | Fail on any CVE | A triage policy with severities, fixability and SLAs |

## Troubleshooting exercises

Full write-up (Detection → Investigation → Mitigation → Root Cause → Permanent Fix → Prevention) for each:

- **S27, "Credential found in a public place".** Planned drill. Assume a long-lived credential (plant a *deactivated-by-design* test access key or Entra client secret in a public gist-like location, or simulate one) is exposed. **Contain** (disable), **rotate/revoke**, **assess exposure** from CloudTrail/Entra sign-in logs (what could it reach, what did it do), **prevent** (why did a long-lived credential exist at all?). The trap is stopping at rotation. Run it as a Sev1 security incident with comms.
- **S28, "All deploys suddenly blocked".** Someone assigned a Deny policy (Azure) or a Kyverno Enforce policy without an audit period on a Friday. Mitigate (exemption with expiry, or revert via Git), then fix the process. Write a PIR.
- **"GuardDuty/Defender finding: unusual API calls from a pod."** A simulated finding (use the GuardDuty sample findings or a harmless test pattern). Triage it through EMS as L2, and determine blast radius from the pod's identity permissions. The finding might be benign, and proving that is the job.
- **"Users get 403 from the edge, the app never saw the request."** A WAF managed rule false positive after tuning. Diagnose from WAF logs.
- **Problem record:** "Security controls are validated only at one layer." Map incidents to gaps in the controls catalogue and close the gaps.

## Deliverables

- [ ] `docs/security/threat-model.md` with DFD, STRIDE per boundary, and accepted risks with reasons
- [ ] `docs/security/controls-catalogue.md` mapping every control → threat → enforcement point → evidence
- [ ] `platform/policy/` (SCPs, Azure Policy initiative, conftest + tests, Kyverno) all as code with ≥ 1 week of audit data before enforce
- [ ] Signed images + SBOMs + Kyverno verification on both clusters
- [ ] Org CloudTrail (immutable), GuardDuty/Security Hub, Defender CSPM, findings routed to EMS
- [ ] Egress allow-lists; WAF tuned (window evidence)
- [ ] ADR-019; incident records S27, S28; one PIR; one PRB

## Completion criteria

- Your threat model lists ≥ 20 threats, each with a control and evidence or an explicit accepted risk, and you can defend any accepted risk under questioning.
- An unsigned image, an untagged resource, a public IP in a landing zone and a `0.0.0.0/0` SG are each blocked **at the earliest enforcement point you designed**, and you show where.
- The S27 drill completes containment in **under 30 minutes** from detection, with an exposure assessment from logs, not guesses.
- You answer the "who changed X" question for AWS, Azure and Kubernetes from audit logs in under 10 minutes each.
- Every secret in `secret-rotation.md` has been rotated at least once, with the procedure verified.
- You explain what an attacker who compromises one pod in `cargo` could and could not reach, citing the specific controls.

## Mentor checkpoints

- **Before you build:** the DFD with trust boundaries. I will point at a boundary you did not draw.
- **During:** the controls catalogue. Expect me to pick a control and ask you to *demonstrate* it being enforced.
- **Questions I will ask:** Why audit before deny? Rotation versus revocation? What do SCPs not protect against? How do you know an image in prod is the one CI built? What did you accept, and why?
- **The failure I will inject:** a change that is compliant at every enforcement point individually and still insecure in combination.

## Resources

1. ***Threat Modeling: Designing for Security* (Shostack):** Part I (four-question framework, STRIDE). Or ***Security Engineering* (Anderson)**, selectively on access control.
2. **AWS docs:** *Organizations → SCPs (examples, evaluation)*, *IAM Access Analyzer*, *GuardDuty EKS protection*; **AWS Security Reference Architecture** (Prescriptive Guidance).
3. **Azure docs:** *Azure Policy → Definition structure, effects, exemptions*; Cloud Adoption Framework *Govern* and the Azure landing zone policy initiatives (read, don't copy blindly).
4. **Kyverno docs:** *Writing policies → Validate, verifyImages*; **Sigstore/cosign docs:** *Keyless signing with GitHub Actions*; **SLSA** (slsa.dev) *Levels*.
5. **CIS Benchmarks** for EKS and AKS: skim to map your controls and find gaps. Do not treat them as a checklist to chase 100%.

## Estimate vs actual

| | Hours | Cost |
|---|---|---|
| Planned | 40 | ~$50–90 (GuardDuty/Security Hub trials, WAF window, Config items, both clusters in windows) |
| Actual | | |
| Variance note | | |
