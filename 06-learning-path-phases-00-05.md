# Step 7 — The Learning Path (Phases 0–5)

## How the ordering differs from the one you proposed, and why

Your draft sequence was sound. Four changes improve the dependency structure:

| Change | Reason |
|---|---|
| **IAM/security moved from Phase 4 to Phase 3**, before IaC and containers | Every resource you create from here on needs an identity decision. Learning IAM after you have already built things means retrofitting security, which is exactly the habit the JD's "solutions meet security standards" line is trying to filter out. |
| **Docker moved from Phase 6 to Phase 4**, before Terraform | Infrastructure exists to run an artifact. Building the artifact first means Phase 5's infrastructure has a real thing to carry, and Phase 2's manual deployment has something to deploy. |
| **Capacity split out of "reliability" into its own Phase 10** | Capacity management is named separately in the JD's role definition, and load testing is a large enough skill to deserve its own block. |
| **DR moved earlier (Phase 11) and portability added at 17** | A DR drill is most valuable while you still have eight weeks left to act on what it reveals. Your proposed final phase becomes Phase 11; the final phase becomes the AWS/Azure-DevOps translation exercise, which is what makes your single-cloud depth defensible in any interview. |

**Every phase has the same seven sections.** The two that people skip are Failure exercises and Verification. Those two are the phase.

---

# Phase 0 — Engineering foundations
*~2 weeks. Prerequisite: none.*

### Concepts to learn
Trunk-based development and why long-lived branches cause integration pain. Git's object model: commit, tree, blob, ref — understanding this makes rebase, reflog and bisect obvious rather than scary. Semantic versioning and what a "breaking change" means to a consumer. The architecture decision record: context, options considered, decision, consequences, and what evidence would change your mind. Twelve-factor app principles, specifically config-in-environment and stateless processes, because every later phase depends on them. Makefiles as the universal "how do I run this" interface.

### Hands-on labs
1. Break a repository deliberately: commit to the wrong branch, amend a pushed commit, force-push over a colleague's work in a scratch clone. Recover each with `reflog`. You must experience recovery before you need it.
2. `git bisect` a regression you plant yourself across 20 commits.
3. Write `ADR-001: Azure as primary cloud` using the decision from [Step 4](03-cloud-choice.md). Make the "options rejected" section longer than the "decision" section.
4. Set up branch protection on your repository: required PR, required status check, no direct pushes to `main`. Then try to push to `main` and watch it fail.

### Project implementation
Create the `runway` repository with the layout from [Step 5 §6](04-the-project.md#6-repository-layout). Write `cargo-api` as a local FastAPI service: three endpoints, an OpenAPI contract, typed Pydantic models, PostgreSQL via docker-compose, Alembic migrations, and pytest. Write `carrier-sim` as a second tiny service with a deliberate `/fail` toggle and a configurable latency. Write the `Makefile`. **Timebox this to two weekends** — this is the boring application and it must stay boring.

### Production considerations
In an enterprise, this phase is CODEOWNERS files, signed commits, a shared branching standard, and a repository template. Application code arrives already written by someone else and your job is to make it deployable — which is exactly why the code here must not absorb your time.

### Failure exercises
Commit a secret to Git and then properly remove it from history — and then understand why "removed from history" is still a compromised credential that must be rotated. Introduce a migration that is not backward compatible and observe that the old code breaks against the new schema; this is the single most common real deployment outage and you should meet it on your laptop.

### Verification
Explain, without notes: what `git rebase` does to commit hashes and why that matters for a shared branch; why config belongs in the environment rather than in the image; what an ADR is for and who reads it.

### Deliverable
`runway/` on GitHub with branch protection, `docs/adr/ADR-001-cloud-choice.md`, a running `cargo-api` + `carrier-sim` under docker-compose, a green test run, and a README that states what exists and what does not.

---

# Phase 1 — Linux and networking
*~3 weeks. Prerequisite: Phase 0. This is the most under-invested phase in most cloud roadmaps and the one that pays off in every incident.*

### Concepts to learn
**Linux:** the boot and service model (systemd units, dependencies, `journalctl`), processes and signals (why `SIGTERM` vs `SIGKILL` decides whether a deploy drops requests), file permissions and ownership, filesystems and inodes (a disk that is "not full" but cannot write), cgroups and namespaces (which is all a container is), and the diagnostic toolkit: `ss`, `lsof`, `dig`, `curl -v`, `tcpdump`, `strace`, `top`/`htop`, `iostat`, `dmesg`.

**Networking:** the OSI/TCP-IP layers as a *debugging order* rather than trivia. IP addressing and CIDR — be able to subnet `10.0.0.0/16` into six usable subnets on paper. Routing and default gateways. The TCP handshake, and what each failure mode looks like from the client: connection refused vs timeout vs reset vs hang. DNS resolution order, record types, TTL, caching, and split-horizon. TLS: handshake, certificate chains, SNI, expiry, and what a hostname mismatch looks like. HTTP/1.1 vs HTTP/2 and keep-alive. L4 vs L7 load balancing. Proxies, NAT, and SNAT port exhaustion. Firewalls as stateful allow-lists.

### Hands-on labs
1. Subnet `10.0.0.0/16` by hand for: AKS nodes, AKS pods, private endpoints, Application Gateway, a management subnet, and room to grow. Justify each size. You will use this exact plan in Phase 2.
2. `dig` a domain through each step: authoritative server, TTL, CNAME chain. Then set a 60-second TTL and watch a change propagate; set 86400 and understand why DNS changes are never fast in an incident.
3. `openssl s_client -connect` a site. Read the chain. Then generate a self-signed certificate, serve it, and reproduce four distinct TLS errors: expired, wrong hostname, untrusted issuer, missing intermediate.
4. `tcpdump` a `curl` and identify the SYN, SYN-ACK, ACK, the TLS ClientHello, and the FIN. Seeing this once changes how you read every later network problem.
5. Produce all four connection failures deliberately on a local VM and record what each looks like from `curl -v`: port closed (refused), packet dropped by firewall (timeout), service killed mid-request (reset), and backend not responding (hang). **Memorise this mapping** — it is the fastest diagnostic shortcut in cloud networking.
6. Fill a disk with a log file until a service fails to write. Find it with `du`/`df`, and separately reproduce inode exhaustion with many empty files.
7. Send `SIGTERM` to your `cargo-api` process mid-request. Does the request complete? Implement graceful shutdown and repeat.

### Project implementation
Add a `/healthz` (liveness) and `/readyz` (readiness) endpoint to `cargo-api`, where readiness actually checks the database. Implement graceful shutdown with a drain period. Add structured JSON logging with a request ID. Write `platform/scripts/net-triage.sh` — a Bash script that, given a hostname and port, runs the resolution/connectivity/TLS checks in order and prints where the chain broke. You will use this script in real incidents later.

### Production considerations
Enterprises add: centrally-managed DNS with delegated zones, corporate root CAs in the trust store, egress through a proxy or firewall with an allow-list, and network changes that require a change request. The practical consequence is that "it works on my machine" and "it works in the datacentre" differ mostly in *egress*, which is why Phase 12's egress exercise exists.

### Failure exercises
Break DNS resolution for your service and diagnose it only via `dig` and `/etc/resolv.conf`. Block the database port with `iptables`/`ufw` and diagnose the resulting timeout without being told what you did — have the script from the lab tell you. Let a certificate expire and observe the failure from both the client and the server side.

### Verification
Given "the app cannot reach the database", state the six things you check and in what order, and say what each result would rule out. Explain the difference between connection refused and connection timeout, and what each implies about where the packet died. Subnet a `/16` on a whiteboard.

### Deliverable
`docs/architecture/network-plan.md` with your subnet design and its justification. `platform/scripts/net-triage.sh`. `docs/runbooks/network-triage.md` documenting the four-failure-mode mapping. `cargo-api` with working probes and graceful shutdown.

---

# Phase 2 — Cloud fundamentals, built by hand on purpose
*~3 weeks. Prerequisite: Phase 1.*

> **Why manual.** You will rebuild all of this in Terraform in Phase 5 and delete it. That is the point. Terraform written by someone who has never clicked the thing is Terraform they cannot debug. This phase buys you the mental model; Phase 5 buys you the discipline. Use the free trial credit here, while destroy-discipline has not yet formed.

### Concepts to learn
The Azure resource hierarchy: tenant → management group → subscription → resource group → resource, and which level each control attaches to. Regions, availability zones and paired regions; what "zone-redundant" actually costs and buys. Resource providers, quotas and limits — and that quota exhaustion is a top-three cause of failed deployments. The shared responsibility model, concretely: for a VM, for AKS, for a managed database, what is Microsoft's job and what is yours. Service SLAs and how composite SLAs multiply downward. Naming and tagging conventions, decided now rather than retrofitted. ARM as the API underneath the portal, the CLI and Terraform — all three call the same thing, which is why the portal is a legitimate learning tool and an illegitimate production tool.

### Hands-on labs
1. Create the subscription, set up MFA on your account, and **create the budget and alert before the second resource exists**. This ordering is the habit.
2. Do the same task three ways: portal, `az` CLI, and Azure PowerShell (`New-AzResourceGroup`). The PowerShell repetition is deliberate — the JD names it and you need reading fluency.
3. Deploy `cargo-api` to a single VM by hand: VM, NSG, public IP, DNS A record, Docker installed by hand, TLS via Let's Encrypt. Note how long it takes and everything you had to remember.
4. Break it on purpose: remove the NSG rule and diagnose the timeout using Phase 1's method.
5. Explore the pricing calculator: model your intended architecture *before* building it, then compare against the real bill at month end. The gap is the lesson.
6. Trigger a quota error deliberately (request more vCPUs than your subscription allows) and read the error properly.

### Project implementation
A hand-built environment matching your Phase 1 network plan: resource group, VNet with the planned subnets, NSGs, a VM running `cargo-api` in Docker, PostgreSQL Flexible Server reachable **only** from the app subnet, a public DNS record with TLS, and Blob Storage. Write down every manual step in `docs/architecture/manual-build-log.md` as you go — that file is the specification for Phase 5's Terraform, and its length is the argument for IaC.

### Production considerations
Real subscriptions are pre-shaped by a landing zone: management group hierarchy, policy inherited from above, a central hub VNet you peer into, delegated DNS, and cost centre tags you do not choose. You will rarely create a VNet from nothing; you will request a spoke. Learning the full stack once means you understand what was done for you.

### Failure exercises
Delete the DNS record and observe the failure mode. Stop the database and watch how `cargo-api` behaves — badly, at first: does it hang, retry forever, or fail fast? Fix it with a connection timeout. Remove your own RBAC assignment on a resource group and experience an authorisation failure from the inside.

### Verification
Draw the Azure resource hierarchy and say what attaches at each level. Explain what you are responsible for in AKS that you are not responsible for in Container Apps. State your monthly cost for this environment and which line item dominates — and why.

### Deliverable
`docs/architecture/manual-build-log.md` (every click, honestly recorded). A working public URL with valid TLS. A budget with alerts. `docs/cost/month-01.md` with the modelled estimate next to the actual bill and an explanation of the difference.

---

# Phase 3 — Identity, access and secrets
*~2.5 weeks. Prerequisite: Phase 2. Moved earlier than your draft because everything after this depends on it.*

### Concepts to learn
Authentication vs authorization, stated precisely enough that you never conflate them again. Entra ID objects: users, groups, service principals, app registrations, managed identities (system- vs user-assigned) — and when each is correct. Azure RBAC: role definitions, assignments, scope inheritance from management group down to resource, and how `Deny` assignments and policy interact. Least privilege as a practice: start from zero, add what fails. The secret problem: why a long-lived credential in CI is the highest-risk thing in most pipelines, and how **OIDC federated credentials** eliminate it. Workload identity on Kubernetes: how a pod's service account becomes an Azure identity. Key Vault: secrets vs keys vs certificates, access policies vs RBAC, soft delete and purge protection, rotation, and versioning. OAuth2/OIDC flows at the level you need to configure SSO: authorization code with PKCE for the portal SPA, client credentials for service-to-service.

### Hands-on labs
1. Create a custom role definition that grants exactly the permissions `cargo-api` needs and nothing more. Derive it empirically: assign nothing, run, read the authorization error, add one action, repeat. This is how least privilege is actually built.
2. Configure OIDC federation between GitHub Actions and Entra ID. Run a workflow that authenticates to Azure with **no secret in the repository**. Then break the subject claim condition and read the failure.
3. Give a VM a managed identity and read a Key Vault secret from it with no credentials on disk.
4. Reproduce the four classic authorization failures and learn to tell them apart from the error text alone: wrong scope, wrong role, missing role assignment propagation delay, and disabled principal.
5. Rotate a Key Vault secret while the application is running. Observe whether the app picks it up. It will not. Now make it.

### Project implementation
Entra groups for `platform-admins`, `workload-developers`, `readers`, with role assignments at the resource-group scope. `cargo-api`'s database credentials move into Key Vault; the VM reads them via managed identity. A GitHub Actions OIDC federated credential replaces any service principal secret. `docs/security/iam-model.md` records who can do what, where, and why.

### Production considerations
Enterprises add: PIM (just-in-time elevation with approval), access reviews, conditional access policies, break-glass accounts that are monitored, and separation of duties so that the person who writes the Terraform cannot approve its apply to production. Know these by name and what problem each solves.

### Failure exercises
**The one you must do:** delete a role assignment the pipeline depends on, then diagnose the failed deployment from the error alone. Set a Key Vault secret to expire during a session and watch the application fail — this is the "expired secret" scenario and it is common in real life. Deliberately over-assign (`Owner` at subscription scope), then write down the blast radius, then fix it.

### Verification
Explain managed identity vs service principal vs OIDC federated credential, and when you would use each. Given "the pipeline cannot deploy", list the five IAM causes in the order you would check them. Explain what an attacker could do with your pipeline's identity — and be satisfied with the answer.

### Deliverable
`docs/security/iam-model.md`. A pipeline that authenticates with zero stored secrets. All application secrets in Key Vault. `docs/incidents/INC-001-broken-role-assignment.md` — your first incident record, written during the drill, not after.

---

# Phase 4 — Containers
*~2 weeks. Prerequisite: Phase 0. Moved before IaC: build the artifact before you build what carries it.*

### Concepts to learn
What a container actually is: namespaces (pid, net, mnt, user), cgroups for limits, and a union filesystem — not a lightweight VM. Image layers, the build cache, and why instruction order determines build time. Multi-stage builds. Base image choice: distroless vs alpine vs slim, and the musl/glibc trap that bites Python wheels. Non-root users and why `USER` is not optional. `ENTRYPOINT` vs `CMD`, and PID 1 signal handling — the reason containers ignore `SIGTERM` and get `SIGKILL`ed nine seconds later. Health checks. Image size and its effect on pull time, which is deployment latency. Registries, tags vs digests, and why you deploy digests. Vulnerability scanning, SBOMs, and how to triage a CVE list without drowning. Build reproducibility.

### Hands-on labs
1. Build `cargo-api` naively, then with multi-stage and a slim base. Compare size, build time, and cold-start pull time. Record the numbers.
2. Reorder your Dockerfile so dependency installation is cached. Measure the rebuild time before and after a one-line code change.
3. Run as root, then as a non-root user. Scan both with Trivy and compare the CVE counts.
4. Reproduce PID 1 signal handling: run your app as a shell child, send `SIGTERM`, watch it be ignored. Fix it with `exec` form and confirm.
5. Set a memory limit below what the process needs and get OOMKilled. Read the exit code (137) and the kernel message. You will see this again in Kubernetes and should recognise it instantly.
6. Generate an SBOM and find one transitive dependency you did not know you shipped.

### Project implementation
Hardened multi-stage Dockerfiles for `cargo-api`, `carrier-sim` and (later) `runway-portal`: non-root, minimal base, no build tools in the final layer, `.dockerignore` that excludes `.env`, `.git` and tests, healthcheck, correct signal handling, and pinned base image digests. Push to ACR. Add a Trivy scan to the local `Makefile` so scanning happens before it is ever a pipeline concern.

### Production considerations
Enterprises mandate an approved base image catalogue, a registry that blocks images above a CVE severity, image signing and admission policies that reject unsigned images, and an automatic rebuild when a base image is patched. Know that "we rebuilt everything because of a base image CVE" is a routine Tuesday.

### Failure exercises
Ship an image that cannot start because the non-root user cannot write where it expects — this is the most common hardening regression. Break the `.dockerignore` and find your `.env` inside a pushed image. Pull by a mutable tag that someone has moved underneath you, and understand why digests exist.

### Verification
Explain what happens between `docker run` and your process executing. Explain why an image can pass tests and fail in production due to a base image difference. State your image's size, its user, its CVE count, and what you did about the count.

### Deliverable
Hardened Dockerfiles in the repository. Images in ACR referenced by digest. `docs/adr/ADR-003-base-image-strategy.md`. A scan report committed with your triage notes.

---

# Phase 5 — Infrastructure as Code
*~4 weeks. Prerequisite: Phases 2, 3, 4. The highest-leverage phase in the roadmap.*

### Concepts to learn
Declarative vs imperative, and why the declarative model requires **state**. What Terraform state is, what it contains (including secrets, which is why it must be encrypted and access-controlled), and why concurrent applies need locking. Providers and version pinning. Resource graph and implicit vs explicit dependencies. Variables, locals, outputs, and data sources. `count` vs `for_each` and why `for_each` is almost always correct. `lifecycle` blocks: `prevent_destroy`, `ignore_changes`, `create_before_destroy`. Reading a plan critically — specifically spotting a *replace* that will destroy data. `terraform import` and `moved` blocks for refactoring without destruction. Module design: inputs, outputs, sane defaults, and versioning as a contract with your consumers. Environment separation: directories with separate state files, not workspaces (and be able to say why). **Drift**: what causes it, how to detect it, and the organisational question of whether to reconcile or adopt.

### Hands-on labs
1. Rebuild Phase 2's hand-built environment entirely in Terraform. Then `terraform destroy` the hand-built one. Compare `manual-build-log.md` against the code — that diff is the argument for IaC in one page.
2. Configure remote state in Blob Storage with locking. Then open two terminals and apply simultaneously; watch the lock work. Then break the lock deliberately and learn `force-unlock`.
3. Change a resource in the portal, run `terraform plan`, and read the drift. Reconcile it. Then do the opposite: import an out-of-band resource into state.
4. Refactor a resource into a module without destroying it, using `moved`. This is the skill that separates people who can maintain Terraform from people who can only write it.
5. Corrupt your state file (in a copy), and recover from the versioned blob. Write the recovery procedure down while it is fresh.
6. Write a plan that would destroy your database, spot it in review, and add `prevent_destroy`.

### Project implementation
`platform/modules/` with `network`, `postgres`, `keyvault`, `storage` and a first `workload` module. `platform/environments/dev/` and `prod/` as thin root modules that only pass variables — **all logic lives in modules**. Remote state in Blob with locking, separate state per environment. A tagging module applied everywhere: `service`, `environment`, `owner`, `cost-centre`, `managed-by=terraform`. A `make up` / `make down` pair that brings the environment up for a session and tears it down after — your budget depends on this being one command.

### Production considerations
Enterprises run Terraform through a pipeline with plan-on-PR and apply-on-merge, a private module registry with versioned releases, policy checks on the plan (OPA/Sentinel/Conftest) before apply, and state that individual engineers cannot touch. Nobody runs `apply` from a laptop. Phase 6 moves you there.

### Failure exercises
Delete a resource in the portal and reconcile the drift. Upgrade the AzureRM provider by a major version and deal with the breaking changes. Apply with an expired credential and read the failure. Run a plan that shows `-/+ destroy and then create` on your database and stop yourself — then write the review checklist that would have caught it automatically.

### Verification
Explain what state is and three distinct ways it can be corrupted. Explain why `for_each` is safer than `count` when removing a middle element. Given a plan output, identify which changes are destructive before applying. Explain your environment separation strategy and defend it against workspaces.

### Deliverable
`platform/` with versioned modules and two environments. Remote state with locking. `make up` / `make down`. `docs/runbooks/terraform-state-recovery.md`. `docs/adr/ADR-004-environment-separation.md`. The hand-built resources are gone, and the environment can be destroyed and recreated from nothing.

→ Next: [Phases 6–11](07-learning-path-phases-06-11.md)
