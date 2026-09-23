# Milestone M04 — AWS Core Infrastructure, Identity-First, Built by Hand

*Phase 04 · ~5 weeks · Months ~3.4–4.7 · Prerequisites: M01, M02, M03 · Unlocks: M05 (Terraform), and through it everything on AWS*

## Why this phase exists

The JD requires *"proficiency in managing and operating AWS and Azure cloud platforms"* and *"experience operating large public cloud footprints"*. This is where you first own a real, paid-for cloud environment — and you build it **by hand, on purpose**.

> **Why manual.** You will rebuild all of this in Terraform in M05 and delete the hand-built version. Terraform written by someone who has never built the thing is Terraform they cannot debug. This phase gives you the mental model; M05 gives you the discipline. Every manual step goes into a build log, and the length of that log is the argument for infrastructure as code.

It is **identity-first**: the account structure, the guardrails, the budget and your own access exist before the first network or instance. Retrofitting identity is the habit the JD's *"meet architectural and security standards"* line is designed to filter out.

## Concepts to learn

**Organisation and accounts**
- AWS Organizations: management account, organisational units (OUs), member accounts, consolidated billing. Why the **account is the hard isolation boundary** in AWS (blast radius, quotas, billing, IAM).
- Service Control Policies (SCPs) as guardrails that cap what even an administrator can do. SCPs grant nothing; they only limit.
- Control Tower and landing zones — what they automate (account factory, guardrails, log archive) and why you will build the equivalent explicitly (ADR-003).
- Regions, availability zones, AZ IDs vs names, local zones; service quotas and how to request increases.

**Identity (deep — this is the foundation of everything)**
- IAM principals: users (avoid), roles (use), IAM Identity Center users and permission sets for humans, service-linked roles.
- **Policy evaluation logic**: explicit deny → SCP → resource-based policy → identity policy → permissions boundary → session policy. Implicit deny by default. Be able to walk it.
- Identity policies vs resource policies vs trust policies. `sts:AssumeRole`, trust relationships, the `Principal` element, conditions (`aws:SourceIp`, `aws:PrincipalOrgID`, `sts:ExternalId`).
- Temporary credentials: STS, session duration, why long-lived access keys are the highest-risk object in most AWS accounts.
- IAM Identity Center with its built-in directory for now (Entra ID replaces it in M13 — ADR-016).
- Root user protection, MFA, break-glass access.
- CloudTrail: who did what, when, from where. Your first audit trail.

**Networking in AWS**
- VPC, subnets (AZ-scoped), route tables, internet gateway, **NAT gateway** (and its cost per hour + per GB — a classic surprise bill), egress-only gateways.
- Security groups (stateful, allow-only, reference other SGs) vs NACLs (stateless, ordered). When to use each.
- VPC endpoints: gateway (S3) vs interface (ECR, STS, Secrets Manager) — private access to AWS APIs and a NAT cost reducer.
- VPC Flow Logs — how to see a dropped packet.
- Route 53: public hosted zones, alias records, health checks, TTLs. ACM for certificates and DNS validation.
- Application Load Balancer: listeners, target groups, health checks, deregistration delay (connection draining), cross-zone.

**Compute, storage, data**
- EC2: instance families, AMIs, user data, instance profiles, IMDSv2 (and why v1 is a credential-theft vector), EBS volumes and snapshots, Auto Scaling groups and launch templates.
- S3: buckets, policies, Block Public Access, versioning, lifecycle, encryption (SSE-S3 vs SSE-KMS), storage classes.
- RDS PostgreSQL: subnet groups, parameter groups, automated backups, PITR, Multi-AZ vs read replica, maintenance windows, connection limits.
- SQS: standard vs FIFO, visibility timeout, DLQ and redrive.
- Secrets Manager vs SSM Parameter Store; KMS keys and key policies.
- Systems Manager Session Manager — shell access with no SSH keys and no open port 22.

**Money**
- Pricing model per service class; AWS Budgets, Cost Explorer, Cost Anomaly Detection; tagging and cost allocation tags. Budget before the second resource.

## Architecture decisions in this phase

### ADR-003 — AWS account structure: Organizations + OUs via Terraform, not Control Tower
- **Problem:** The JD talks about *"large public cloud footprints"* and governed onboarding. You need a multi-account structure that behaves like an enterprise landing zone at lab scale.
- **Options:** (a) one account with env tags; (b) Control Tower landing zone; (c) Organizations + OUs + accounts managed explicitly (by hand now, Terraform in M05).
- **Decision:** (c). OUs `security`, `infrastructure`, `workloads`, `sandbox`; accounts `management`, `security` (log archive + audit), `shared-services` (ECR, Terraform state, observability later), `workloads-dev`, `workloads-prod`, `sandbox` (the future account-vending target in M18).
- **Trade-offs:** you build what Control Tower would give you, which is more work but teaches every piece; Control Tower is the likely choice at work and you must be able to explain what it adds (Account Factory, managed guardrails, drift detection of the landing zone). Option (a) was rejected because it hides the account-as-boundary concept, which is the most important AWS idea.

### ADR-004 — Network and CIDR plan across AWS, Azure and the datacentre
- **Problem:** Three locations must eventually route to each other (M21) without overlap, and EKS pods will consume VPC IPs (M09).
- **Options:** default ranges; per-environment ad hoc; a planned supernet.
- **Decision:** AWS dev `10.10.0.0/16`, AWS prod `10.20.0.0/16`, Azure `10.30.0.0/16`, datacentre `10.40.0.0/16`. Each AWS VPC: 3 AZs × (public `/24`, private-app `/19`, data `/24`) with reserve. Single NAT gateway in dev (documented cost-vs-availability trade-off), one per AZ in prod windows. Gateway endpoint for S3; interface endpoints for ECR, STS, Secrets Manager.
- **Trade-offs:** large private-app subnets reserve pod IP space that is unused for months; single NAT in dev is an accepted single point of failure. Both are written as accepted risks.

## Hands-on labs

1. **Account structure first.** Create the organisation, OUs and accounts; enable IAM Identity Center; create permission sets `PlatformAdmin`, `Developer`, `ReadOnly`. MFA on root, then lock root away. **Create AWS Budgets and a Cost Anomaly Detection monitor before the second resource exists.**
2. **SCP guardrail.** Attach an SCP denying actions outside your two allowed regions and denying `cloudtrail:StopLogging`. Try to launch in a third region and read the error. *Proves: SCPs cap even admins.*
3. **Least privilege, empirically.** Create a role for `cargo-api` with nothing. Run the app, read each `AccessDenied`, add one action, repeat. Then use IAM Access Analyzer policy generation from CloudTrail and compare with what you derived.
4. **Policy evaluation walk.** Build a case where an identity policy allows, a resource policy is silent and an SCP denies. Predict the outcome; confirm with the IAM policy simulator.
5. **Three ways, one API.** Create a bucket with the console, the AWS CLI and PowerShell (`AWS.Tools`). All three call the same API — which is why the console is a learning tool and not a production tool.
6. **Build the network by hand** from your CIDR plan: VPC, 9 subnets, IGW, NAT, route tables, endpoints. Enable VPC Flow Logs. Then remove the private route to NAT and watch an instance lose internet (see S15 later).
7. **Deploy `cargo-api` on EC2** using the M01 systemd unit, via user data, in private subnets behind an ALB with an ACM certificate and a Route 53 alias record. RDS PostgreSQL in the data subnets reachable only from the app security group. Secrets in Secrets Manager read with the instance profile. Shell access only via Session Manager — no SSH, no port 22.
8. **SQS for the ETA job.** Replace the local Redis list with SQS + DLQ. Make a poison message land in the DLQ.
9. **Quota error.** Request more vCPUs than your quota allows via an ASG and read the error properly. Request a quota increase and note how long it takes.
10. **Estimate vs bill.** Use the AWS Pricing Calculator to model this environment before building it; compare with the bill after a week.

## Project implementation

A hand-built environment in `workloads-dev`:
- VPC per ADR-004, Flow Logs to CloudWatch Logs; S3 gateway endpoint; interface endpoints for STS and Secrets Manager.
- ALB (HTTPS only, HTTP → HTTPS redirect) → Auto Scaling group of 2 EC2 instances across 2 AZs running `cargo-api` and `eta-worker` under systemd; IMDSv2 required.
- RDS PostgreSQL (single-AZ, burstable, automated backups, deletion protection on) in data subnets.
- SQS `eta-recompute` + DLQ.
- Route 53 public zone for your domain; ACM certificate; `api.dev.<domain>` alias record.
- `carrier-sim` on a small instance or as a second process — it must be reachable only from the app SG.
- CloudTrail organisation trail to the `security` account's log bucket.
- Tags on everything: `service`, `environment`, `owner`, `cost-centre`, `managed-by=manual` (it will change to `terraform`).

Documentation, written *as you build*:
- `docs/architecture/manual-build-log.md` — every click and command, honestly recorded. It is the specification for M05.
- `docs/security/iam-model.md` — who (humans, workloads) can do what, where, and why; the permission sets; the SCPs.
- `docs/adr/ADR-003-aws-account-structure.md`, `docs/adr/ADR-004-network-cidr-plan.md`.
- `docs/cost/month-01.md` — estimate vs actual and what surprised you.

## Engineering problems to hit deliberately

| Problem | How to cause it | What you should learn |
|---|---|---|
| AccessDenied on deploy | Remove one action from the deploy role | Read the error: which principal, which action, which resource, which policy type denied |
| Trust policy mismatch | Wrong principal in a role's trust policy | AssumeRole failures are trust problems, not permission problems |
| Timeout to database | Remove the app SG from the DB SG rule | Timeout = drop; SGs drop silently |
| Private subnet has no internet | Remove route `0.0.0.0/0 → NAT` | Routing is per subnet; endpoints bypass NAT |
| ALB 502/503 | Health check path wrong | Target health is the LB's view of readiness |
| NAT bill surprise | Pull a large image through NAT repeatedly | NAT charges per GB processed; endpoints fix it |
| Quota exhaustion | Scale beyond vCPU quota | Quotas are capacity ceilings you must plan |

## Troubleshooting exercises

- **S04 — "The deployment script fails with AccessDenied."** Somebody changed the deploy role. Diagnose from the error and CloudTrail alone; name the policy type that denied. Write it as `docs/incidents/INC-002-access-denied.md` in the full Detection → Investigation → Mitigation → Root Cause → Permanent Fix → Prevention format.
- **S02 — "Requests to the API hang for 30 seconds then fail."** The DB security group rule is gone. Use Flow Logs (`REJECT` records) to prove where the packet died.
- **S01 — "api.dev stopped resolving for some users."** A Route 53 record was changed with a long TTL. Explain why some users recovered and others did not.
- **"All targets unhealthy after an AMI refresh."** The new AMI is missing a package the unit needs. Diagnose via Session Manager and `journalctl` — no SSH.

## Deliverables

- [ ] Organization with OUs, six accounts, Identity Center permission sets, SCPs
- [ ] Budgets + Cost Anomaly Detection created before any workload resource
- [ ] Working `https://api.dev.<domain>` with a valid certificate, served by `cargo-api` on EC2 behind an ALB
- [ ] RDS reachable only from the app tier; secrets in Secrets Manager; no SSH anywhere
- [ ] `docs/architecture/manual-build-log.md`
- [ ] `docs/security/iam-model.md`
- [ ] ADR-003, ADR-004
- [ ] `docs/incidents/INC-002-access-denied.md`
- [ ] `docs/cost/month-01.md`

## Completion criteria

- Walk the IAM policy evaluation order from memory and apply it to a case with an SCP deny and a resource-policy allow.
- Given an `AccessDenied` message, identify principal, action, resource and the policy type responsible in under 10 minutes using CloudTrail.
- Explain why the account is the primary isolation boundary in AWS, and what an SCP can and cannot do.
- Draw the VPC from memory: subnets, route tables, IGW, NAT, endpoints, and the path of a packet from an instance to S3 and to the internet.
- State this environment's monthly cost and its dominant line item, and explain the difference from your estimate.
- Zero long-lived IAM user access keys exist in any account (show the credential report).

## Mentor checkpoints

- **Design review:** bring ADR-003 and ADR-004 before building. Expect: "Why is `shared-services` separate from `workloads-prod`?", "What does your SCP break that you have not thought of?"
- **IAM review:** bring `iam-model.md`. The mentor will ask you to justify each permission in the `cargo-api` role and to explain what an attacker holding it could do.
- **The failure I'll inject after you think you're done:** an identity problem that presents as a network problem. Your first hypothesis will probably be wrong; record it.

## Resources

- AWS IAM User Guide — "Policy evaluation logic" (whole section), "IAM roles" → "Terms and concepts", "Security best practices in IAM".
- Amazon VPC User Guide — "How Amazon VPC works", "Internetwork traffic privacy" (security groups vs NACLs), "NAT gateways" (including pricing notes), "Gateway endpoints".
- AWS Organizations User Guide — "Service control policies" (evaluation and examples); AWS Security Reference Architecture — the account structure diagrams.
- AWS Skill Builder — free "AWS Cloud Quest: Cloud Practitioner" labs for the console muscle memory, if you want it; skip anything you have already built.

## Estimate vs actual

| | Hours | Cost |
|---|---|---|
| Planned | 50 | $15–30 (new-account credits may cover this — verify current terms) |
| Actual | | |
| Variance note | | |
