# Milestone M21 — Hybrid and Multi-Cloud Architecture

*Phase 21 · ~3 weeks · Months ~22–23 · Prerequisites: [M13](M13-azure-core.md) (Azure, Entra, cross-cloud identity), [M15](M15-security-governance.md) (policy across clouds), [M04](M04-aws-core.md) (network plan) · Unlocks: [M22](M22-hardening-capstone.md) · Can run before M19 if needed; one of the safest reductions if you fall behind.*

## Why this phase exists

The team's mission is to build *"next-generation cloud platforms that unify control across datacentres, networks, public and private cloud environments"*. The same posting describes it as a team that *"drive[s] the evolution of hybrid cloud capabilities"*, and the role includes *"Manage end-of-life replacement of equipment and/or software within the technology area"*.

Runway has run on two clouds since M13, but so far they have only touched through identity and HTTPS. This phase adds the missing third piece, a **datacentre**. You will connect it to both clouds and bring it under the same controls. Then you will step back and describe the whole estate as the architecture it now is.

It comes this late because hybrid work combines everything before it: networking from M01 and M04, identity from M13, policy from M15, observability from M10 and the EOL process from M11. Much of the expensive connectivity runs only in timeboxed windows.

## Concepts to learn

**What "unify control" means in practice.** You want one way of governing resources wherever they run:
- the same identity
- the same policy
- the same inventory
- the same telemetry
- the same patching and change process

This does not mean running the same runtime everywhere. There are two control-plane projection tools, and you should learn what each one projects and what it does not:
- **Azure Arc**, for servers and for Kubernetes: inventory, Azure Policy and guest configuration, tags, RBAC, extensions, GitOps (Flux) on Arc-enabled clusters, and Defender and Monitor integration.
- **AWS Systems Manager hybrid activations**, which make on-prem servers SSM managed nodes: inventory, patching, Run Command and Session Manager.

**Hybrid connectivity.**
- **Site-to-site IPsec VPN:** tunnels, IKE phases, BGP versus static routes, redundancy (two tunnels), and throughput limits.
- **Private circuits:** AWS Direct Connect and Azure ExpressRoute. Know what problem they solve (predictable latency and bandwidth, a private path, and data-transfer pricing) and why you won't buy one.
- **Transit patterns:** AWS Transit Gateway compared with VPC peering, Azure Virtual WAN compared with hub-spoke peering, and why transitive routing matters once you pass three networks.
- **Overlapping CIDRs:** why they are fatal to connectivity. This is why ADR-004 fixed non-overlapping ranges back in month 4.

**Hybrid DNS.** On-prem names must resolve from the clouds and cloud private names must resolve from on-prem. The pieces:
- conditional forwarding
- Route 53 Resolver inbound and outbound endpoints and forwarding rules
- Azure DNS Private Resolver
- a split-horizon view of the private endpoint zones

Most hybrid outages are DNS outages.

**Multi-cloud architecture, argued rather than decorated.**
- **Legitimate reasons:** best-of-breed services (Entra and Azure OpenAI; the AWS primitives), acquisitions, regulatory requirements, and negotiating leverage.
- **Illegitimate reasons:** "avoiding lock-in" by using the lowest common denominator, and running active-active across clouds for resilience you haven't measured.
- **Multi-cloud costs:** two IAM models, two policy engines, egress between clouds, double the skills, and more ways to fail.

Runway's multi-cloud shape (ADR-015) is by *placement*, not duplication. Be ready to defend it.

**Portability layers and where they stop.**
- Kubernetes, Terraform, OpenTelemetry, OCI images, GitOps and the `workload.yaml` contract travel between clouds.
- Identity, networking, data services and policy engines do not.

**Lifecycle and EOL in hybrid estates.** Datacentre kit has hardware EOL, OS EOL, and k3s or Kubernetes version skew relative to the cloud clusters. The migration choices are rehost, replatform, refactor, retire and retain, which form the "6 Rs" together with repurchase. For each workload, decide which one applies and write down why.

## Architecture decisions in this phase

**ADR-026 — Hybrid connectivity and control: VPN, Arc, SSM, hybrid DNS**

| | |
|---|---|
| **Problem** | A "datacentre" (a local Linux VM and k3s) must be governed like cloud resources and reachable privately from both clouds, and cloud private services must be reachable from it, on a lab budget. |
| **Options** | Connectivity: (a) public endpoints with mTLS; (b) site-to-site VPN to each cloud plus AWS↔Azure VPN; (c) a transit hub (TGW / Virtual WAN); (d) Direct Connect / ExpressRoute. Control: (e) Arc only; (f) SSM only; (g) both, each for its strength. DNS: (h) hosts files; (i) resolver endpoints with conditional forwarding. |
| **Decision** | **(b)** in timeboxed windows: AWS↔Azure via AWS Site-to-Site VPN and Azure VPN Gateway with BGP, and datacentre→cloud via a strongSwan or WireGuard router VM. **(g)**: Arc for policy, tags, inventory and GitOps on k3s; SSM for patching and session access on the VM. **(i)**: Route 53 Resolver endpoints plus Azure DNS Private Resolver with conditional forwarding, also timeboxed. Daily operation stays on the identity-federated HTTPS paths from ADR-017. |
| **Trade-offs** | VPN gateways and resolver endpoints bill hourly (roughly tens of dollars a month each if left on), so they exist only in windows, and you document what an always-on estate would cost. (c) and (d) are right at enterprise scale and unaffordable here, so they are explained, not built. Running two control tools overlaps, and the ADR records which tool is authoritative for what. |

Also finalise **ADR-015** (workload placement) with a section of real evidence gathered since M13: cross-cloud latency, egress cost and failure incidents.

## Hands-on labs

1. **Arc the datacentre.** Onboard the local VM (Arc-enabled server) and k3s (Arc-enabled Kubernetes). Assign the same tag and baseline Azure Policy initiative used for the cloud. Read the compliance report, fix the findings, then disconnect the agent and watch how staleness is reported. *Proves: one policy plane.*
2. **SSM hybrid activation.** Register the VM as an SSM managed node. Patch it with a Patch Manager baseline and open a Session Manager shell with no SSH port open. *Proves: governed access without inbound ports.*
3. **Build the VPN (timeboxed day).** Set up AWS↔Azure site-to-site VPN with BGP. Verify the routes you learn in both clouds. Then add the datacentre router. Measure latency and throughput between each pair of sites. *Proves: you understand routing, not just the console wizard.*
4. **Break routing on purpose.** Advertise an overlapping prefix from the datacentre and watch what BGP and the route tables do. Then withdraw it. *Proves: why ADR-004's CIDR plan mattered.*
5. **Hybrid DNS.** Resolve an RDS private endpoint name and an Azure private-link name *from the datacentre*, and a `.dc.runway.internal` name from EKS and AKS pods. Trace every forwarding hop with `dig +trace` and the resolver query logs. *Proves: you can debug the most common hybrid failure.*
6. **GitOps to the edge.** Deploy `hello-svc` to k3s using Flux through Arc's GitOps extension, from the same Git source. Compare this with Argo CD's model in two paragraphs. *Proves: portability layers are real, and so are their limits.*
7. **Egress bill check.** Push 5 GB between the clouds over the VPN, then over public HTTPS. Find both in the FOCUS data (M16). *Proves: inter-cloud traffic is a cost decision.*

## Project implementation

- `platform/modules/azure-arc-onboarding`, `platform/modules/aws-ssm-hybrid`, and the datacentre inventory in `catalog/` (the VM and k3s appear in the service catalogue with owners, and in the data catalogue if they hold data).
- `platform/modules/aws-vpn`, `platform/modules/azure-vpn-gateway` and `platform/modules/hybrid-dns`, all behind a `make hybrid-up` / `make hybrid-down` pair with a hard TTL tag. The idle-detector from M16 alerts if any of them outlive the window.
- The Azure Policy initiative now covers Arc resources. Kyverno baseline policies are applied to k3s through Flux. SSM patch compliance is exported to the ops dashboard.
- Observability: the OTel Collector on k3s and the VM ships metrics to AMP and logs to Loki over the VPN (window) or public TLS (default). A `site` label (`aws | azure | dc`) is added to the standard dashboards.
- `docs/architecture/hybrid.md`: topology diagram (Mermaid), routing tables, DNS forwarding map, and which control tool is authoritative for what.
- `docs/lifecycle/eol-register.md` gains the datacentre entries (OS, k3s version skew, hardware) plus a **migration assessment** that applies the 6 Rs to each datacentre workload, with a decision and a timeline.
- `docs/architecture/multi-cloud-rationale.md`: why Runway is multi-cloud by placement, what it costs (measured egress and duplicated skills), and what would make you consolidate.
- Runbooks: `docs/runbooks/vpn-tunnel-down.md`, `docs/runbooks/hybrid-dns.md`, `docs/runbooks/arc-agent-disconnected.md`.

## Engineering problems to hit deliberately

| Problem | How to cause it | What you should learn |
|---|---|---|
| Asymmetric routing | Advertise a route on one side only | Tunnels "up" ≠ traffic flows; check both route tables |
| MTU/MSS black hole | Leave default MTU over IPsec; large TLS responses hang | The classic "small requests work, large ones hang" signature |
| Stale Arc agent | Stop the VM for a week | Compliance of a disconnected resource is unknown, not compliant |
| DNS forwarding loop | Forward the same zone both ways | Resolver query logs; one authority per zone |
| Version skew | Run k3s two minors behind EKS | Fleet version policy belongs on the EOL register |
| Window left open | Forget `make hybrid-down` | The bill is the detector; TTL tags and idle-detector are the prevention |

## Troubleshooting exercises

1. **S34, "Datacentre workloads unreachable from the cloud."** EKS pods can't reach a service on the VM. Both tunnels show `UP`. Diagnose layer by layer with `net-triage.sh`, route tables, BGP learned routes and a packet capture on the router VM. Plant one of three causes: a removed propagated route, an MTU black hole, or a broken conditional forwarder. Write it up in full: Detection → Investigation → Mitigation → Root Cause → Permanent Fix → Prevention.
2. **"Arc says compliant, but the VM is unpatched."** The Arc guest-configuration policy and SSM patch compliance disagree. Find out which is authoritative, why they differ, and fix the gap in ADR-026's authority table.
3. **"AWS↔Azure calls got slow and expensive."** A change moved `insights-worker` traffic from the federated HTTPS path onto the VPN during a window. Latency and egress both changed. Treat this as a change that was never assessed (ADR-013 process) and write the PIR.

## Deliverables

- [ ] `docs/adr/ADR-026-hybrid-connectivity-and-control.md`; ADR-015 finalised with evidence
- [ ] Arc (server + k3s) and SSM hybrid modules; datacentre in the service catalogue and under the same policy
- [ ] VPN + hybrid DNS modules with `make hybrid-up/down` and TTL enforcement
- [ ] `docs/architecture/hybrid.md`, `docs/architecture/multi-cloud-rationale.md`
- [ ] EOL register datacentre entries + 6 Rs migration assessment
- [ ] Three hybrid runbooks; incident record for S34; measured latency/throughput/egress table

## Completion criteria

- One Azure Policy initiative reports compliance for cloud and datacentre resources together, and you have fixed at least one finding on the datacentre side.
- From the datacentre, an RDS private endpoint and an Azure private-link name both resolve and connect. From EKS and AKS, a datacentre name resolves. You can draw every forwarding hop from memory.
- You diagnosed a planted S34 cause in **under 30 minutes** without being told which of the three it was.
- A measured table exists for latency, throughput and cost per GB for each site pair and path (VPN compared with public TLS).
- Given "the business wants active-active across AWS and Azure for cargo-api", you can explain in five minutes why you would or would not, using your own measured numbers and your DR decisions (ADR-014).

## Mentor checkpoints

- **Before building:** bring the updated CIDR and routing diagram and the DNS forwarding map. I will ask where each name resolves from each site, and what happens to each one when a tunnel drops.
- **Mid-phase:** bring BGP learned-route output from both clouds and your packet capture from Lab 4.
- **Review questions:**
  - "What does Arc actually give you that a CMDB spreadsheet doesn't?"
  - "Why is your multi-cloud design not a lock-in hedge?"
  - "What would an enterprise do here with Transit Gateway or Virtual WAN, and when is that worth it?"
- **The failure I'll inject after you think you're done:** a connectivity fault where every status page says healthy. The category is data-plane failure with a green control plane.

## Resources

- **Azure Arc docs:** "Azure Arc-enabled servers overview", "Azure Arc-enabled Kubernetes overview", and "GitOps with Flux v2" on Arc-enabled clusters.
- **AWS docs:** "AWS Site-to-Site VPN — How it works" (tunnels, BGP), the "Route 53 Resolver" developer guide (inbound and outbound endpoints, forwarding rules), and "Setting up Systems Manager for hybrid and multicloud environments".
- **Azure docs:** "VPN Gateway: Create a site-to-site connection" and "About BGP with VPN Gateway", plus "Azure DNS Private Resolver overview".
- **Azure Cloud Adoption Framework:** "Hybrid and multicloud" scenario guidance, including unified operations. Read it for the governance model, not the product list.
- ***Computer Networking: A Top-Down Approach*** (Kurose & Ross): the routing chapter on BGP. Also revisit the Julia Evans networking zine for MTU and fragmentation.

## Estimate vs actual

| | Planned | Actual | Variance note |
|---|---|---|---|
| Hours | 30 | | |
| Cloud cost this phase | $80–140 (VPN/resolver windows) | | |
