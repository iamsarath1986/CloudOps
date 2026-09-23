# Appendix A1 — Cloud Architecture Concepts

**Learn the concept, then the product name.** Every idea below is explained as *why it works*: the mechanism and the failure it prevents. Only after that does it map onto AWS and Azure. This habit makes you portable, and it is how you avoid the most common failure of cloud learners: knowing where a button is without knowing what it does.

Use this appendix in three ways:

- **Before each phase**, read the concepts that phase depends on ([M03](../milestones/M03-cloud-architecture.md) covers most of it).
- **When learning Azure** ([M13](../milestones/M13-azure-core.md)), read §4 and §5 to learn Azure as *differences* rather than as a second curriculum.
- **Before interviews**, explain each concept out loud without using a product name. If you cannot, you know the product but not the concept.

---

## 1. Physical and failure-domain concepts

### Regions
A region is a set of datacentres in one metropolitan area, with its own independent control plane for most services. **Why it matters:** a region is the unit of *legal* placement (data residency: UAE regions exist on both clouds), of *latency* to users, and of *blast radius* for control-plane failures. Most services are regional, which means a regional outage takes out both your resources *and your ability to change them*. That is why DR runbooks must not depend on the failed region's APIs.

### Availability zones
An AZ is one or more datacentres inside a region with independent power, cooling and networking, connected to the other AZs by low-latency links (typically single-digit milliseconds). **Why it works:** failures that are correlated inside a building (power, cooling, a top-of-rack switch, a fire) are uncorrelated across AZs. Spreading replicas across three AZs turns "a datacentre failed" from an outage into a capacity reduction. **The cost:** cross-AZ data transfer is billed, and synchronous replication across AZs adds latency to writes.

### Fault domains and update domains
Inside an AZ, hardware still fails in groups (racks, hosts). Fault domains are the cloud's guarantee that replicas you ask to spread are placed on independent hardware. Update domains are the equivalent for planned maintenance. In Kubernetes you express the same idea with **topology spread constraints** and **PodDisruptionBudgets**. The concept is identical at every layer: *do not put all replicas behind one shared failure.*

### Shared responsibility
The provider secures the infrastructure *of* the cloud. You secure what you put *in* it. The line moves with the service model:

| Model | Provider owns | You own |
|---|---|---|
| VM | hardware, hypervisor, physical network | OS, patches, runtime, app, data, network rules, identity |
| Managed Kubernetes | control plane, its availability and patching | nodes (unless auto-managed), add-ons, workloads, RBAC, network policy, upgrades you trigger |
| Managed database | engine patching, storage, backups mechanism | schema, users, connection limits, backup *testing*, network exposure |
| Serverless function | everything below your code | code, dependencies, identity, configuration |

The trap: "managed" never includes *your configuration*. Most cloud breaches are misconfiguration on the customer's side of this line.

---

## 2. Networking

### IP addressing and CIDR
An IPv4 address is 32 bits. CIDR notation `/n` says how many leading bits are the network. The rest are hosts. `10.10.0.0/16` has 65,536 addresses. `/24` has 256. **Why it matters:** CIDR ranges cannot overlap if two networks are ever to route to each other (peering, VPN, hybrid). This is why Runway fixes its plan on day one (ADR-004): AWS dev `10.10.0.0/16`, AWS prod `10.20.0.0/16`, Azure `10.30.0.0/16`, datacentre `10.40.0.0/16`. Overlap discovered later is almost impossible to fix without rebuilding.

Clouds reserve some addresses per subnet (AWS: 5, Azure: 5), and Kubernetes can consume IPs at a rate that surprises people (see VPC CNI in M09). Size subnets for pods, not only for nodes.

### Subnets and routing
A subnet is a range of addresses associated with a route table. **Routing decides where a packet goes next; it never decides whether it is allowed.** That is filtering, which is a separate layer. The longest-prefix match wins: a `/32` route beats a `/16`, which beats `0.0.0.0/0`.

"Public" and "private" subnet are not properties of the subnet itself. They are properties of its **route table**: a subnet is public if its default route points at an internet gateway and its hosts have public addresses. It is private if the default route goes to NAT or nowhere.

### NAT
Network Address Translation rewrites source addresses so that many private hosts can share one public address for *outbound* connections. **Why it works:** replies come back to the NAT, which remembers the mapping. Unsolicited inbound connections have no mapping, so private hosts are unreachable from the internet by construction. **What breaks:** each outbound connection consumes a source port on the NAT, so thousands of connections to one destination exhaust ports (SNAT exhaustion: intermittent connection failures that look like the remote side is flaky). **What costs:** managed NAT gateways bill per hour *and per GB processed*. This is one of the classic surprise bills (S29), and why VPC endpoints for S3/ECR/STS exist.

### Stateful filtering: security groups, NSGs, firewalls
A stateful filter tracks connections: if outbound is allowed, the reply is allowed automatically. **The debugging consequence:** a dropped packet produces a *timeout* (nothing comes back), while a closed port on a reachable host produces a *refused* (a TCP RST). Learn that mapping once ([M01](../milestones/M01-linux-networking.md)) and you will locate network faults in minutes.

### DNS
DNS maps names to records through a hierarchy (root → TLD → authoritative), with caching at every resolver for the record's TTL. **Why it is the most common cause of "the app is down":** it is invisible when working, cached when broken, and resolves differently depending on *where you ask from*. **Split-horizon DNS** (the same name resolving to a private address inside a VNet and a public address outside) is how private endpoints work, and a broken private DNS zone link makes a healthy database unreachable while everything *looks* correct (S24).

### TLS
TLS provides server authentication, encryption and integrity. The server presents a certificate chain; the client verifies each signature up to a trusted root, checks the hostname against the certificate (SAN), and checks validity dates. **SNI** lets one IP host many certificates. The four failures you must recognise on sight: expired, hostname mismatch, untrusted issuer, missing intermediate (S03). TLS **termination** point matters: terminate at the load balancer and the hop behind it is plaintext unless you re-encrypt.

### Proxies
A forward proxy acts for clients (corporate egress control). A reverse proxy acts for servers (load balancers, gateways, ingress). Every proxy is a hop with its own timeouts, headers (`X-Forwarded-For`), connection pools and TLS settings. Mismatched timeouts between hops are a classic source of mysterious 502/504s.

### Load balancers: L4 vs L7
An **L4** load balancer forwards TCP/UDP connections by address and port. It is fast and protocol-agnostic, but it cannot see HTTP. An **L7** load balancer terminates HTTP(S), so it can route by host and path, add headers, run a WAF and retry. Both run **health checks** and only send traffic to healthy targets. Your readiness endpoint *is* your load-balancing policy. **Connection draining** (deregistration delay) is why graceful shutdown matters: a pod that exits before the load balancer stops sending it traffic drops requests during every deploy.

### API gateways
An API gateway is an L7 reverse proxy specialised for APIs: authentication, rate limiting, quotas, request transformation, versioning and usage metering. **When it earns its place:** many consumers with different entitlements. Runway's `ai-gateway` is exactly this pattern, applied to model access.

### Private connectivity to managed services
Managed services have public endpoints by default. **Private endpoints / PrivateLink** place an interface for the service *inside your network*, with a private IP and DNS pointing at it, so traffic never leaves the provider's backbone and the public endpoint can be disabled. **Gateway endpoints** (AWS S3/DynamoDB) instead add a route. Both reduce exposure and NAT cost.

---

## 3. Compute, state and scaling

### Compute models
| Model | You manage | Right when |
|---|---|---|
| VM | OS and everything above | Legacy, special kernels, full control |
| Container on managed runtime (ECS Fargate, Container Apps) | image and configuration | Few services, no need for Kubernetes extensibility |
| Kubernetes | workloads, add-ons, cluster lifecycle | Many services, platform extensibility, multi-cloud portability |
| Functions (Lambda) | code | Short, event-driven or scheduled, spiky |

Choosing is an ADR, not a preference (ADR-007, ADR-008).

### Storage
**Block** (a disk attached to one host: low latency, one writer). **File** (shared POSIX filesystem: many writers, locking semantics). **Object** (HTTP API over immutable objects: effectively infinite, cheap, the default for backups, logs and static assets). Redundancy tiers (single AZ, zone-redundant, geo-redundant) trade cost for durability and availability. Lifecycle policies move cold data to cheaper tiers.

### Databases
Relational databases give you transactions and constraints, and their scaling limit is the single writer. **Read replicas** scale reads with replication lag. **Multi-AZ** gives a synchronous standby for failover, and does *not* scale reads on most engines. **Connections are a finite resource**: every pod with a pool of 10 times 30 pods is 300 connections, which is how autoscaling the application takes down the database (S18).

### Caching
A cache trades freshness for latency and load reduction. The questions every cache design must answer: *what is the TTL, how is it invalidated, and what happens on a miss storm?* The **cache stampede** (many clients miss at once and all hit the database) is solved with request coalescing, jittered TTLs or stale-while-revalidate. A cache must be an optimisation, never the only copy.

### Queues
A queue decouples a producer from a consumer in time and rate. **Why it works:** a burst is absorbed as queue depth rather than as latency or errors. You must understand:
- **At-least-once delivery**, which means consumers must be idempotent
- **Visibility timeouts**: if processing outlasts the timeout, the message is delivered twice
- **Dead-letter queues** for poison messages
- **Backpressure**: queue depth is a capacity signal and a scaling metric

### Stateless architecture and horizontal scaling
A stateless process keeps nothing between requests that another replica would need: session, cache and files live in shared services. **That is the precondition for horizontal scaling** (add replicas) and for safe rolling deploys, spot instances and autoscaling. **Vertical scaling** (bigger instances) is simpler but hits a ceiling, usually needs a restart, and leaves you with a single failure domain.

### Autoscaling
Scaling reacts to a signal: CPU, requests in flight, queue depth. Every autoscaler has **lag** (metric window + decision + provisioning time) and needs **stabilisation** to avoid flapping. Two layers interact: pod autoscaling (HPA) adds replicas, and node autoscaling (Karpenter, cluster autoscaler) adds capacity for them. Wrong resource *requests* break both.

---

## 4. Identity, secrets and trust

### Authentication vs authorisation
Authentication proves *who* you are. Authorisation decides *what* you may do. Clouds separate the two: an identity provider issues credentials, and a policy engine evaluates each API call.

### Principals
Humans (users, via SSO), workloads (roles / managed identities), and external systems (federated principals). **The modern rule:** humans log in through one workforce IdP. Workloads never hold long-lived secrets; they exchange a platform-issued token for short-lived cloud credentials.

### Federation, OIDC and workload identity
OIDC federation lets a cloud trust tokens issued by another system (GitHub Actions, a Kubernetes cluster's OIDC issuer, another cloud's IdP) under **conditions on the token's claims**: issuer, subject (which repo, branch, service account), audience. **Why it works:** the token is short-lived and bound to a specific workload, so there is nothing to leak or rotate. **Why it breaks:** a single mismatched claim produces an opaque 401 (S25). Runway uses federation for CI, for pods, and across the cloud boundary in both directions (ADR-017).

### Least privilege
Start from nothing and add what fails. Scope as narrowly as the model allows (resource, condition). Prefer many small roles to one broad one. The blast radius of a compromised identity is exactly its permissions.

### Secrets
A secret has a lifecycle: creation, distribution, rotation, revocation, audit. Stores (Secrets Manager, Key Vault) centralise it. Delivery mechanisms (ESO, CSI) get it to workloads. **Rotating** replaces a secret. **Revoking** makes the old one useless. Only revocation contains a leak (S27). Better still, remove the secret entirely through federation.

---

## 5. Reliability concepts

### Availability and composite availability
Availability = uptime / total time. **Serial dependencies multiply:** a service that needs an LB (99.99%), a cluster (99.95%) and a database (99.95%) is at best ~99.89%. **Redundant components combine:** two independent 99% replicas give 1 − (0.01²) = 99.99%, *if* their failures really are independent. Most real availability is lost to correlated failures and bad changes, not to hardware.

### High availability vs fault tolerance vs resilience
- **High availability:** the service stays up despite component failures, possibly with a brief blip (failover). Built from redundancy with no single point of failure.
- **Fault tolerance:** no visible interruption at all, which usually needs active-active redundancy and costs more.
- **Resilience:** the broader property of absorbing *and recovering from* failure, including degrading gracefully (serve cached data, disable a feature, return a fast 503) instead of collapsing. Timeouts, retries with backoff and jitter, circuit breakers, bulkheads and load shedding are resilience patterns. **Retries without backoff turn a blip into a cascading failure** (S20).

### Distributed-systems realities
Networks drop, delay and reorder messages. Clocks drift. Partial failure is normal: one replica is slow while others are fine. Two consequences you design for: **every remote call needs a timeout**, and **every operation that may be retried must be idempotent**. The CAP trade-off in practice: during a partition, choose between refusing writes (consistency) and accepting divergent writes (availability).

### SLI, SLO, SLA, error budget
- **SLI:** a measured ratio of good events (for example, requests under 300 ms and not 5xx).
- **SLO:** the target for that ratio over a window (99.5% over 28 days).
- **SLA:** a contractual promise with consequences, and always looser than the SLO.
- **Error budget:** 1 − SLO. The amount of unreliability you may spend on change. Burn-rate alerts fire when you spend it too fast.

### RTO and RPO
**RTO** (recovery time objective) is how long the service may be down. **RPO** (recovery point objective) is how much data you may lose, measured in time. They are *business* decisions that constrain architecture and cost: RPO near zero needs synchronous replication, and RTO near zero needs standby capacity that is already running. **An untested recovery is a belief, not an RTO.** Runway measures both in drills ([M12](../milestones/M12-capacity-ha-dr.md)).

### Disaster recovery strategies
In order of cost and speed:
1. **Backup and restore:** hours, cheapest
2. **Pilot light:** data replicated, minimal core running, compute scaled up on failover; tens of minutes
3. **Warm standby:** a scaled-down full copy running; minutes
4. **Multi-site active-active:** seconds; most expensive and complex

**Replication is not a backup:** it faithfully replicates your deletion. Runway uses backup-restore plus pilot light (ADR-014).

---

## 6. Concept → AWS ↔ Azure mapping

Learn the **row**, not the cell.

| Concept | AWS | Azure |
|---|---|---|
| Billing / isolation boundary | Account | Subscription |
| Hierarchy for policy inheritance | Organization → OU → account | Management group → subscription → resource group |
| Lifecycle grouping of resources | Tags / CloudFormation stack | Resource group |
| Org-wide preventive guardrail | Service Control Policy (SCP), Resource Control Policy | Azure Policy (Deny), management-group assignment |
| Detective compliance | AWS Config rules, Security Hub | Azure Policy (Audit), Defender for Cloud |
| Landing zone accelerator | Control Tower, Account Factory (for Terraform) | Azure Landing Zones (CAF), subscription vending |
| Workforce SSO | IAM Identity Center | Entra ID |
| Authorisation model | IAM policies (JSON, identity- and resource-based) | Azure RBAC (role definitions + assignments at scope) |
| Workload identity (VM/service) | IAM role (instance profile, task role) | Managed identity (system/user-assigned) |
| Kubernetes pod identity | EKS Pod Identity / IRSA | AKS Workload Identity |
| CI federation | IAM OIDC provider + role trust policy | Entra app/UAMI federated identity credential |
| Private network | VPC | VNet |
| Subnet-level filter | NACL (stateless) | NSG on subnet (stateful) |
| Instance/NIC-level filter | Security group (stateful) | NSG on NIC / Application Security Group |
| Custom routes | Route tables | Route tables / UDR |
| Internet egress for private hosts | NAT Gateway | NAT Gateway |
| Network peering / hub | VPC peering, Transit Gateway | VNet peering, Virtual WAN / hub VNet |
| Private access to PaaS | VPC endpoints / PrivateLink | Private Endpoint / Private Link |
| Site-to-site VPN | Site-to-Site VPN (VGW/TGW) | VPN Gateway |
| Dedicated private link to on-prem | Direct Connect | ExpressRoute |
| Public DNS | Route 53 | Azure DNS |
| Private DNS | Route 53 private hosted zone | Private DNS zone (linked to VNet) |
| Hybrid DNS forwarding | Route 53 Resolver endpoints | Azure DNS Private Resolver |
| L7 load balancer | Application Load Balancer | Application Gateway / App Gateway for Containers |
| L4 load balancer | Network Load Balancer | Azure Load Balancer |
| CDN | CloudFront | Azure Front Door |
| WAF | AWS WAF | Azure WAF (on App Gateway / Front Door) |
| API gateway | API Gateway | API Management |
| VMs / scale sets | EC2 / Auto Scaling group | VM / VM Scale Set |
| Serverless containers | ECS on Fargate / App Runner | Container Apps / Container Instances |
| Functions | Lambda | Azure Functions |
| Managed Kubernetes | EKS | AKS |
| Container registry | ECR | ACR |
| Object storage | S3 | Blob Storage |
| Block storage | EBS | Managed Disks |
| File storage | EFS | Azure Files |
| Managed PostgreSQL | RDS / Aurora PostgreSQL | Azure Database for PostgreSQL Flexible Server |
| Managed Redis | ElastiCache | Azure Managed Redis |
| Queue | SQS | Storage Queues / Service Bus |
| Pub/sub, events | SNS / EventBridge | Service Bus topics / Event Grid |
| Secret store | Secrets Manager / SSM Parameter Store | Key Vault |
| Key management | KMS | Key Vault keys / Managed HSM |
| Metrics & logs platform | CloudWatch | Azure Monitor / Log Analytics |
| Managed Prometheus / Grafana | Amazon Managed Prometheus / Managed Grafana | Azure Monitor managed Prometheus / Managed Grafana |
| Audit log of API calls | CloudTrail | Activity Log |
| Cost tooling | Cost Explorer, Budgets, Cost Anomaly Detection, Data Exports (FOCUS) | Cost Management, Budgets, anomaly alerts, exports (FOCUS) |
| Backup service | AWS Backup | Azure Backup |
| Hybrid server management | Systems Manager (hybrid activations) | Azure Arc-enabled servers |
| Hybrid Kubernetes | EKS Anywhere / EKS Hybrid Nodes | Azure Arc-enabled Kubernetes |
| Managed LLMs | Amazon Bedrock | Azure OpenAI / Azure AI Foundry |

Service names change more often than concepts. Verify names against current documentation when you use them.

---

## 7. Where the two clouds genuinely differ

A mapping table implies equivalence, and in five places that implication is dangerous. These are the differences worth being able to explain in an interview, because they change how you design rather than only what things are called.

### 7.1 IAM policy documents vs RBAC role assignments
**AWS** evaluates JSON policy documents attached to identities *and* to resources, combined with SCPs, permission boundaries and session policies. The evaluation logic is: explicit deny wins → otherwise at least one allow must exist in every applicable layer. You write fine-grained actions, resources and conditions directly. **Azure** evaluates **role assignments**: a role definition (a set of actions) assigned to a principal *at a scope* (management group, subscription, resource group or resource), inherited downward. Conditions exist but are less central, and deny assignments are rare. **Design consequence:** on AWS, least privilege is mostly *policy authoring*; on Azure it is mostly *choosing the right scope and built-in role*. "Why am I denied?" is answered on AWS by walking the evaluation layers, and on Azure by listing effective role assignments at each scope.

### 7.2 Security groups vs NSGs
An **AWS security group** attaches to an ENI (instance, pod with security groups for pods, load balancer). It is allow-only, stateful, and can reference *other security groups* as sources ("allow from the ALB's SG"), which makes identity-like rules possible. **NACLs** add a stateless subnet layer. An **Azure NSG** attaches to a subnet and/or NIC, has priority-ordered allow *and deny* rules including default rules you must understand, and uses **service tags** and **application security groups** for grouping. **Design consequence:** AWS rules compose by reference, while Azure rules are evaluated in priority order at two possible attachment points. A packet can be allowed at the NIC and denied at the subnet.

### 7.3 Account vs subscription
An **AWS account** is a hard boundary for identity, resources, quotas and billing. IAM principals live *inside* accounts, and cross-account access is explicit (role assumption). An **Azure subscription** is a billing and quota boundary under a tenant, and identities live in **Entra ID at the tenant level**, above subscriptions. One user can hold roles in many subscriptions without any "assume". **Design consequence:** on AWS, isolation by account is the primary security boundary and multi-account is the default. On Azure, the tenant is shared, so isolation relies more on RBAC scope and policy.

### 7.4 Organizations + SCPs vs management groups + Azure Policy
**SCPs** cap the maximum permissions in member accounts. They never grant; they only restrict what IAM can allow, and they do not affect the management account. **Azure Policy** evaluates *resource properties* (location, SKU, tags, public network access) with effects `Audit`, `Deny`, `Modify`, `DeployIfNotExists`, assigned at management-group or subscription scope. It can also *remediate*. **Design consequence:** SCPs govern *who may call which API*, while Azure Policy governs *what a resource may look like*. AWS covers resource-shape rules with Config rules (detective) or, increasingly, declarative controls. Azure covers "who may call" with RBAC. Runway uses both, side by side (ADR-019).

### 7.5 Workload identity
**EKS Pod Identity** associates a Kubernetes service account with an IAM role through an agent on the node. It is simple, but AWS-only. **IRSA** uses the cluster's public OIDC issuer and `AssumeRoleWithWebIdentity`. It is more setup, but because it is standard OIDC, *other clouds can trust the same issuer*. **AKS Workload Identity** also uses the cluster's OIDC issuer, with an Entra federated identity credential on a managed identity. **Design consequence:** both clusters expose standard OIDC issuers, which is what makes Runway's cross-cloud, keyless calls possible: the EKS issuer is trusted by Entra, and the AKS issuer is trusted by AWS IAM (ADR-017). Learn OIDC once and both clouds follow.

### 7.6 Smaller differences that cause real incidents
- **Default egress:** Azure has been retiring implicit default outbound access for new subnets, and AWS private subnets never had it. Either way, *design egress explicitly*.
- **Private DNS:** Azure private endpoints depend on private DNS zones *linked to each VNet* that must resolve them. Forgetting the link is S24. AWS interface endpoints enable private DNS on the VPC more directly.
- **Kubernetes networking:** the EKS VPC CNI gives pods real VPC IPs (IP exhaustion risk; S13). AKS Azure CNI Overlay gives pods an overlay range (no VNet exhaustion, different routing visibility).
- **Resource grouping:** Azure resource groups are a lifecycle unit (delete the group, delete everything in it). AWS has no direct equivalent, so tags and Terraform state do that job.

← [Back to the index](../README.md)
