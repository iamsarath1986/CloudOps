# Milestone 2 — Linux and the Network

*Phase 1 · ~3 weeks · Prerequisite: Milestone 1*

> The most under-invested phase in most cloud roadmaps, and the one that pays back in every incident from Milestone 8 onward.

**Learning topics**
* systemd, journald, processes and signals; `SIGTERM` vs `SIGKILL` and why it decides whether deploys drop requests
* Permissions, filesystems, inodes; cgroups and namespaces as the actual definition of a container
* The diagnostic toolkit: `ss`, `lsof`, `dig`, `curl -v`, `tcpdump`, `strace`, `iostat`, `dmesg`
* CIDR and subnetting by hand; routing and default gateways
* TCP handshake and the four client-visible failure modes: refused, timeout, reset, hang
* DNS resolution order, record types, TTL, caching, split-horizon
* TLS: handshake, certificate chains, SNI, expiry, hostname mismatch
* L4 vs L7 load balancing; proxies, NAT and SNAT port exhaustion; stateful firewalls

**Build**
`/healthz` and `/readyz` on `cargo-api`, where readiness genuinely checks the database. Graceful shutdown with a drain period. Structured JSON logging with a request ID. `platform/scripts/net-triage.sh` — given a host and port, runs resolution → connectivity → TLS checks in order and prints where the chain broke.

**Cloud resources**
Still none. Everything runs on local VMs.

**DevOps work**
`net-triage.sh` added to the `Makefile` and to CI as a smoke check against the local stack.

**Operations exercise**
Produce all four TCP failure modes deliberately and record what each looks like from `curl -v`. **Memorise the mapping** — it is the fastest diagnostic shortcut in cloud networking. Then fill a disk until a service cannot write, and separately exhaust inodes; find each with `df`, `du` and `df -i`.

**Security considerations**
Generate a self-signed certificate and reproduce four distinct TLS errors: expired, wrong hostname, untrusted issuer, missing intermediate. Being able to name which one you are looking at, from the error alone, is a genuine skill.

**Expected outcome**
`docs/architecture/network-plan.md` contains your subnet design for `10.0.0.0/16` — AKS nodes, pods, private endpoints, Application Gateway, management, and growth room — with each size justified. You will build exactly this in Milestone 3.

**Evidence of mastery**
Given "the app cannot reach the database", state the six checks in order and what each result rules out. Explain connection refused vs timeout and what each says about where the packet died. Subnet a `/16` on a whiteboard. Explain what a container is using only the words namespace, cgroup and filesystem.
