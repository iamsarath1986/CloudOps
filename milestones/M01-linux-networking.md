# Milestone M01 — Linux and Networking

*Phase 01 · ~5 weeks · Months ~1–2 · Prerequisites: M00 · Unlocks: M02 (automation), M03 (cloud architecture), and every incident you will ever work*

## Why this phase exists

The JD never says "Linux" and barely says "networking" — but it asks for *"troubleshooting across application and cloud infrastructure layers"* and describes a team that *"unif[ies] control across datacentres, networks, public and private cloud environments"*. Every one of those layers is Linux and TCP/IP underneath. EKS nodes, AKS nodes, containers, load balancers and VPNs are all Linux processes and network paths.

This is the most under-invested phase in most cloud roadmaps and the one that pays back in every incident. It comes before any cloud because cloud networking is ordinary networking with a console on top: a security group is a stateful firewall, a VPC route table is a routing table, a private DNS zone is split-horizon DNS. Learn the ordinary version first and the cloud version becomes obvious.

## Concepts to learn

**Linux**
- The boot and service model: systemd units, dependencies, restart policies, `journalctl`. You will run `cargo-api` as a systemd service on EC2 in M04 before containers ever exist.
- Processes and signals: `SIGTERM` vs `SIGKILL`, exit codes, zombies, PID 1. Why graceful shutdown decides whether a deploy drops requests.
- Users, groups, permissions, ownership, `umask`, capabilities. Why "run as non-root" matters.
- Filesystems: mounts, inodes, a disk that is "not full" but cannot write, `/proc` and `/sys` as the kernel's API.
- **cgroups and namespaces** — the whole of what a container is. Read `/sys/fs/cgroup` for a process.
- Resource diagnosis: load average vs CPU utilisation, memory (RSS, page cache, OOM killer), I/O wait. Tools: `top`/`htop`, `vmstat`, `iostat`, `free`, `dmesg`, `ss`, `lsof`, `strace`, `tcpdump`.
- Package management and patching as a lifecycle concern (the start of the JD's end-of-life theme).

**Networking — as a debugging order, not trivia**
- The TCP/IP layers and the question each answers: can I resolve? route? connect? handshake TLS? get a valid HTTP response?
- IP addressing and **CIDR**: subnet `10.10.0.0/16` into six subnets across three availability zones on paper. Private ranges (RFC 1918), why overlapping CIDRs block future peering and VPNs.
- Routing: default gateway, longest-prefix match, `ip route`, asymmetric routing.
- **NAT**: SNAT/DNAT, why private hosts need NAT to reach the internet, SNAT port exhaustion.
- TCP: three-way handshake, what **refused vs timeout vs reset vs hang** each look like from the client and what each says about where the packet died.
- **DNS**: resolution order (`/etc/nsswitch.conf`, `/etc/resolv.conf`), recursive vs authoritative, record types (A, AAAA, CNAME, ALIAS, TXT, MX), TTL and caching, negative caching, split-horizon.
- **TLS**: handshake, certificate chains, SNI, expiry, hostname verification, mutual TLS as a concept.
- HTTP/1.1 vs HTTP/2, keep-alive and connection reuse, `curl -v` literacy.
- L4 vs L7 load balancing; proxies (forward vs reverse); health checks; stateful vs stateless firewalls.

## Architecture decisions in this phase

No numbered ADR yet — but you make the decision that ADR-004 (M04) will formalise: **the CIDR plan**.
- **Problem:** AWS, Azure and a simulated datacentre must eventually be routable to each other without overlap.
- **Options:** default VPC ranges everywhere; ad hoc ranges per environment; a planned supernet per location.
- **Decision (draft):** AWS dev `10.10.0.0/16`, AWS prod `10.20.0.0/16`, Azure `10.30.0.0/16`, datacentre `10.40.0.0/16`, with per-AZ subnet tiers (public, private-app, data) and room for Kubernetes pod IPs.
- **Trade-offs:** planning now costs an evening; renumbering later costs weeks. `/16`s are generous — justify that EKS's VPC CNI gives every pod a VPC IP, so pod density drives subnet size.

## Hands-on labs

Use a local Linux VM (multipass, Vagrant or UTM) — free, destroyable.

1. **Subnet on paper.** `10.10.0.0/16` → public, private-app and data subnets in three AZs, plus reserve. Justify each size given pods consume VPC IPs. Keep this; it is ADR-004's appendix.
2. **DNS walk.** `dig +trace` a domain; read TTLs and the CNAME chain. Change a record with a 60 s TTL and watch propagation; then understand why an 86400 s TTL makes incidents slow to fix.
3. **TLS failures.** Create a local CA, issue certificates and reproduce four errors: expired, wrong hostname, untrusted issuer, missing intermediate. Read each with `openssl s_client -connect host:443 -servername host`.
4. **Packet capture.** `tcpdump` a `curl`; identify SYN, SYN-ACK, ACK, ClientHello, FIN.
5. **The four failure modes.** Produce refused (port closed), timeout (firewall drop with `nft`/`iptables`), reset (process killed mid-request), hang (backend accepts but never answers). Record what each looks like in `curl -v` and in `tcpdump`. **Memorise the mapping.**
6. **Disk and inodes.** Fill a disk with a log file until `cargo-api` cannot write; find it with `df`/`du`. Separately exhaust inodes with many empty files and see `df -i`.
7. **Signals.** Run `cargo-api` under systemd; send `SIGTERM` mid-request. Does the request complete? Implement graceful shutdown with a drain period and repeat.
8. **NAT.** Put a second VM behind the first acting as a NAT router; make the inner VM reach the internet only through it. You have just built what an AWS NAT gateway is.
9. **cgroups by hand.** Start a process in a cgroup with a memory limit and watch the OOM killer take it (`dmesg`). You will see exit code 137 again in M06 and M08.

## Project implementation

- `cargo-api` gains `/healthz` (liveness: the process is alive) and `/readyz` (readiness: it can reach the database and would serve traffic). Know why they differ.
- Graceful shutdown with a configurable drain period; structured JSON logs with `request_id`.
- `platform/scripts/net-triage.sh` — given host and port, checks in order: resolution (which resolver, which answer, TTL), route, TCP connect (distinguishing refused/timeout), TLS (chain, expiry, hostname), HTTP status. Prints *where* the chain broke. You will run it inside pods in M09 and M13.
- `platform/systemd/cargo-api.service` — a hardened unit (dedicated user, `Restart=on-failure`, `ProtectSystem=strict`, resource limits). It is the deployment unit for M04.
- `docs/architecture/network-plan.md` — the CIDR plan with justification.
- `docs/runbooks/network-triage.md` — the four-failure-mode mapping and the order of checks.

## Engineering problems to hit deliberately

| Problem | How to cause it | What you should learn |
|---|---|---|
| Name does not resolve | Point `/etc/resolv.conf` at a dead resolver | Resolution is step one of every network diagnosis |
| Stale DNS after change | High TTL, change the record | TTL is a commitment you made in the past |
| Timeout vs refused | Drop vs reject in the firewall | The client error tells you where the packet died |
| Certificate chain incomplete | Serve leaf without intermediate | Works in some clients, fails in others — classic |
| Deploy drops requests | `SIGKILL` instead of `SIGTERM` + drain | Graceful shutdown is an availability feature |
| Disk full / inode exhaustion | Log flood / tiny files | "Disk not full" can still mean "cannot write" |

## Troubleshooting exercises

Have a friend, or a script you write and then forget about for a day (`inject-local.sh` choosing randomly), break the VM:

- **S01 — "The service is unreachable by name."** Diagnose using only `dig`, `/etc/resolv.conf` and `net-triage.sh`.
- **S02 — "Requests to the database hang, then time out."** A firewall rule now drops port 5432. Prove where the packet dies with `tcpdump` on both ends.
- **S03 — "Clients reject HTTPS."** One of the four certificate errors, chosen at random. Name which from the client error alone before opening the certificate.
- **"The API is slow, but only sometimes."** Hidden cause: disk I/O saturation from a log flood. Use `iostat`/`vmstat`, not guesswork.

Write S02 as your first full incident record (`docs/incidents/INC-001-db-timeout.md`) using Detection → Investigation → Mitigation → Root Cause → Permanent Fix → Prevention. It is practice for the format, so write the timeline *during* the drill.

## Deliverables

- [ ] `docs/architecture/network-plan.md` (CIDR plan, justified)
- [ ] `platform/scripts/net-triage.sh` with a `--help` and exit codes per failure stage
- [ ] `platform/systemd/cargo-api.service`
- [ ] `docs/runbooks/network-triage.md`
- [ ] `cargo-api` with liveness/readiness separation and graceful shutdown
- [ ] `docs/incidents/INC-001-db-timeout.md`

## Completion criteria

- Subnet a `/16` into nine subnets across three AZs on a whiteboard in under five minutes, and explain why pod density matters on EKS.
- Given "the app cannot reach the database", list the six checks in order and what each result rules out.
- Identify refused / timeout / reset / hang from `curl -v` output alone, 4/4, without hesitation.
- Identify which of the four TLS errors from the client message alone.
- Show a `SIGTERM` during a request completing successfully after your change, and failing before it.
- Diagnose S01–S03 from a cold start in under 15 minutes each, with notes written during.

## Mentor checkpoints

- **Design review:** bring the CIDR plan before building anything. Expect: "Where do Kubernetes pod IPs come from on EKS?", "What happens when Azure wants to peer?"
- **Script review:** bring `net-triage.sh`. The mentor checks `set -euo pipefail`, timeouts on every probe, and whether output tells an on-call engineer *what to do next*.
- **The failure I'll inject after you think you're done:** a problem that looks like the network but is not. You will need the four-failure-mode mapping to rule the network out quickly.

## Resources

- Brian Ward, *How Linux Works* (3rd ed.) — ch. 3–4 (devices, disks, filesystems), ch. 6 (how user space starts: systemd), ch. 8 (processes and resource utilisation), ch. 9–10 (network and applications).
- Julia Evans zines — *How DNS Works*, *Networking! ACK!*, *HTTP: Learn Your Browser's Language*, *Bite Size Linux*.
- Kurose & Ross, *Computer Networking: A Top-Down Approach* — ch. 2.4 (DNS), ch. 3.5 (TCP), ch. 4.3 (IP addressing, NAT). Selectively.
- iximiuz Labs — "Computer Networking Fundamentals" and "Linux networking namespaces" tracks.

## Estimate vs actual

| | Hours | Cost |
|---|---|---|
| Planned | 50 | $0 |
| Actual | | |
| Variance note | | |
