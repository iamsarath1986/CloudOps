# Milestone M08 — Kubernetes Fundamentals (local, kind/k3d)

*Phase 08 · ~3 weeks · Months ~8.1–8.9 · Prerequisites: [M06](M06-containers.md) (hardened images), [M07](M07-cicd.md) (you have something worth deploying) · Unlocks: [M09](M09-eks.md)*

> **Learn on a laptop for free. Operate in the cloud later.** Nothing in this milestone costs money. Save the budget for what only a real cluster can teach: cloud load balancers, node autoscaling, cloud identity and upgrades. Those are in M09.

---

## Why this phase exists

The JD asks for an *"understanding of containerisation and orchestration technologies such as Docker, Kubernetes, Amazon EKS, Azure AKS"*, and it expects you to be *"troubleshooting across application and cloud infrastructure layers"*. Kubernetes sits between those two layers, and on most platforms it is where the majority of incidents turn up.

It comes after containers (M06), because Kubernetes schedules containers and knowing images is a precondition. It comes after CI/CD (M07), because you should already have an artefact promoted by digest. It comes before EKS (M09) for two reasons. EKS adds a whole cloud layer of failure (VPC CNI, IAM, load balancer controllers), and you cannot tell a cloud failure from a Kubernetes failure until you know what Kubernetes on its own looks like. And every minute spent learning pod basics on EKS is a minute of control plane billing.

## Concepts to learn

**Architecture and the core idea**
- Control plane: `kube-apiserver` (the only thing that talks to etcd), etcd (the state), the scheduler, and the controller manager. Node components: kubelet, kube-proxy (or an eBPF replacement), and the container runtime (containerd) behind the CRI.
- **The reconciliation loop.** You declare desired state and controllers keep converging towards it. Once this clicks, Kubernetes becomes predictable: a deleted pod "comes back" because the ReplicaSet controller noticed a gap, not because anything was "restarted".
- **What happens between `kubectl apply` and a running process.** API server authentication, authorisation and admission, then persistence to etcd. Next the Deployment controller creates a ReplicaSet and the ReplicaSet controller creates Pods. The scheduler filters and scores nodes and binds the pod. The kubelet sees the binding, pulls the image, creates the sandbox and asks the CNI for an IP, then starts containers and runs probes. Finally the EndpointSlice controller adds the pod IP once it is Ready. **This is interview question #1. Be able to draw it.**

**Workloads**
- Pod: the atomic unit, with shared network and volumes. Init containers and native sidecars (`restartPolicy: Always` on an init container).
- Deployment → ReplicaSet → Pod. StatefulSet: stable identity and per-pod PVCs. DaemonSet: one per node. Job and CronJob. Know when each is correct, and why "a database in a Deployment" is a bug.
- Rolling updates: `maxSurge` and `maxUnavailable`, `minReadySeconds`, `progressDeadlineSeconds`, revision history, `kubectl rollout undo`.

**Networking**
- The flat pod network model: every pod gets an IP and there is no NAT between pods. CNI is the plugin that makes that true.
- Services: ClusterIP, NodePort, LoadBalancer and headless. Selectors → EndpointSlices → kube-proxy rules. DNS: `svc.namespace.svc.cluster.local`, and `ndots:5` and why it multiplies lookups.
- Ingress, and why the **Gateway API** replaces it: role separation (GatewayClass/Gateway owned by the platform, HTTPRoute owned by the application team), richer matching, and portable traffic splitting. **ingress-nginx was retired** (retirement announced November 2025, maintenance ended March 2026). You will use it once here, deliberately, as the thing you migrate off.

**Configuration and state**
- ConfigMaps, and Secrets, which are **base64, not encryption**. Who can `get secrets` in a namespace can read them all. Envelope encryption at rest is a cluster setting, not a property of the object.
- Environment variables versus mounted files. Mounted files update in place, environment variables never do. That is why "I rotated the secret and nothing changed" is so common.
- Volumes: PersistentVolume, PersistentVolumeClaim, StorageClass, dynamic provisioning, access modes, `reclaimPolicy`, and `WaitForFirstConsumer` volume binding (zones matter in M09).

**Resources, health and scheduling**
- Requests (the scheduling guarantee) versus limits (enforcement). QoS classes: Guaranteed, Burstable, BestEffort, and eviction order. **CPU limits cause CFS throttling, and memory limits cause OOMKill (exit 137).** They are different failure shapes.
- Probes: startup, readiness and liveness, and what each is for. Readiness removes a pod from endpoints. Liveness restarts it. **A liveness probe that checks a dependency is an outage multiplier.** A startup probe protects slow starters.
- Scheduling: nodeSelector, node affinity, pod (anti-)affinity, taints and tolerations, topology spread constraints, and priority and preemption.
- Namespaces as the tenancy unit. ResourceQuota and LimitRange. The difference between a namespace boundary and a security boundary.

**Access, isolation and packaging**
- RBAC: Role/ClusterRole and RoleBinding/ClusterRoleBinding, ServiceAccounts, and `kubectl auth can-i --as`. Least privilege for humans and for workloads.
- NetworkPolicy: default-deny ingress and egress, label-based allow rules, and that **egress default-deny breaks DNS unless you allow it**. Policies need a CNI that enforces them. kind's default CNI does not, so install Calico or Cilium.
- Pod Security Standards (privileged/baseline/restricted) via Pod Security Admission labels.
- Helm (charts, values, releases, `helm template`, `helm diff`) versus Kustomize (overlays). **Decision for Runway:** Helm for third-party components, and a thin Helm chart (`charts/runway-service`) for your own workloads, which later becomes the golden-path chart in M17.
- HPA v2: resource and custom metrics, `behavior` (stabilisation windows), and why HPA needs correct *requests* to compute utilisation. metrics-server.

## Architecture decisions in this phase

No new ADR is numbered here. The ADRs for traffic entry and the runtime move (ADR-008, ADR-009) are written in M09 against a real cloud. Make one local design decision and record it in the chart README:

| Problem | Options | Decision | Trade-offs |
|---|---|---|---|
| How are Runway's own workloads packaged? | Raw YAML · Kustomize overlays · a shared Helm chart | One opinionated Helm chart, `runway-service`, with values per workload and environment | Helm templating is harder to read than YAML. In exchange you get one place to enforce probes, resources, security context and labels. That is the seed of the golden path. |

## Hands-on labs

Use `kind` (or `k3d`) with three nodes and Calico installed so NetworkPolicy is enforced.

1. **Trace the apply path.** Run `kubectl apply` on a Deployment with `-v=8`. Watch `kubectl get events -w` in a second terminal. Write down each controller's action in order. *Proves: you understand reconciliation.*
2. **Kill things.** Delete a pod, then a ReplicaSet, then the Deployment. Predict what comes back each time before you do it. *Proves: ownership and controllers.*
3. **Service plumbing.** Create a Service, run `kubectl get endpointslices`, then break the selector with a typo. The Service exists and has zero endpoints. Resolve names from a debug pod (`kubectl debug` / `nicolaka/netshoot`) and explain `ndots`. *Proves: selectors → endpoints → DNS.*
4. **Probe self-harm.** Set a liveness probe with a 1 s timeout on an endpoint that takes 2 s at start-up. Watch the restart loop. Fix it with a startup probe. Then make readiness depend on Postgres, stop Postgres, and watch the pod leave endpoints *without* restarting. *Proves: the three probes do different jobs.*
5. **Resource shapes.** Run a memory hog with a 64Mi limit: OOMKilled, exit code 137, `lastState.terminated.reason`. Run `cargo-api` with a 100m CPU limit under `hey`/k6: p99 explodes while `kubectl top` looks unremarkable. *Proves: throttling ≠ OOM.*
6. **Scheduling.** Taint a node, then use tolerations, anti-affinity (two replicas never on one node) and topology spread. Request 64 CPU and read the `FailedScheduling` event verbatim. *Proves: you can read the scheduler's reasons.*
7. **Rollout and rollback.** Deploy a broken tag with `maxUnavailable: 0`. The rollout stalls and the service stays up. Run `kubectl rollout undo` and time it. Repeat with `maxUnavailable: 100%` and see the difference. *Proves: rollout settings are availability settings.*
8. **Config and Secrets.** Mount a ConfigMap as a file and as an env var, then change it. Only the file changes, after a delay. Base64-decode a Secret as a user who can `get secrets`. *Proves: Secrets are access-controlled data, not encrypted data.*
9. **State.** Create a StatefulSet with a PVC per replica. Delete a pod: same name, same volume. Delete the StatefulSet: the PVCs survive. Understand why. *Proves: identity + storage lifecycle.*
10. **RBAC + NetworkPolicy.** Create a `dev-readonly` user and a ServiceAccount, and check both with `kubectl auth can-i --list --as`. Apply default-deny to the `cargo` namespace, watch everything break (including DNS), then allow exactly what is needed. *Proves: least privilege at both the API and the packet layer.*
11. **HPA.** Install metrics-server, create an HPA on CPU, run load and watch it scale. Remove the requests and watch the HPA report `<unknown>`. *Proves: HPA depends on requests.*
12. **The EOL exercise.** Expose `cargo-api` through ingress-nginx. Then migrate it to Gateway API (Envoy Gateway locally) with a written cut-over checklist. Keep the checklist, because it becomes the template for ADR-009. *Proves: you can replace a retired component methodically.*

## Project implementation

- `charts/runway-service/`: one Helm chart used by `cargo-api`, `carrier-sim` and `eta-worker`. It sets non-negotiable defaults: `runAsNonRoot`, `readOnlyRootFilesystem`, dropped capabilities, all three probes, requests (and memory limits; CPU limits off by default, with the reason written in the README), a PodDisruptionBudget, topology spread and standard labels (`app.kubernetes.io/*`, `runway.io/service`, `runway.io/owner`).
- `platform/k8s/local/`: a kind config (3 nodes), Calico, metrics-server and Envoy Gateway. `make kind-up` / `make kind-down`.
- Namespaces `cargo` and `platform` with ResourceQuota and LimitRange. Default-deny NetworkPolicy plus explicit allows (ingress from the gateway, egress to Postgres/Redis/DNS/carrier-sim).
- Postgres and Redis as StatefulSets **for the local lab only**. RDS replaces Postgres in M09. Redis stays in-cluster, as the spec says.
- `eta-worker` as a Deployment consuming a local SQS stand-in (ElasticMQ/LocalStack), because the real SQS arrives in M09.
- `docs/runbooks/pod-failure-triage.md` (first version): the six failure states, three commands each.

## Engineering problems to hit deliberately

| Problem | How to cause it | What you should learn |
|---|---|---|
| Service has no endpoints | Label/selector mismatch | "Service exists" ≠ "traffic flows". Always check EndpointSlices |
| Liveness restart loop | Liveness probe timeout < start-up time | Startup probes exist for this. Liveness must never check dependencies |
| Throttled but "idle" | CPU limit 100m under load | Throttling appears in `container_cpu_cfs_throttled_*`, not in CPU % |
| DNS dies under default-deny | Egress deny without a UDP/TCP 53 allow | NetworkPolicy is L3/L4. DNS is traffic too |
| Secret rotation "doesn't work" | Secret consumed as env var | Env vars are read once at start. Mount files or roll the pods |
| PVC stuck `Pending` | StorageClass missing or wrong access mode | Storage binding is a scheduling input |
| HPA shows `<unknown>` | No requests / no metrics-server | Autoscaling is built on requests |

## Troubleshooting exercises

Work each one from the symptom only. Have a script (or a friend) inject the fault so you do not know the cause.

- **S12, "Pods never become Ready".** Four variants, injected at random: a wrong image tag (ImagePullBackOff), a missing ConfigMap key (CreateContainerConfigError), an app crash on a bad env var (CrashLoopBackOff), and readiness pointed at a stopped dependency. **Target: identify the variant in under five minutes each using only `kubectl get/describe/logs --previous/events`.**
- **S09, "Containers restart, exit code 137"**, reproduced in-cluster. Distinguish a container OOMKill from node-pressure eviction (`kubectl describe node`, `Evicted` status).
- **"Half the requests fail after a deploy"**: a new ReplicaSet with a broken readiness path while `maxUnavailable: 50%`. Mitigate with `rollout undo` first, then explain.
- **"Everything in the namespace stopped talking"**: a colleague applied a default-deny policy. Find it from `kubectl get netpol -A` and a `netshoot` connectivity matrix.

Write S12 up in full (Detection → Investigation → Mitigation → Root Cause → Permanent Fix → Prevention) as `docs/incidents/INC-0xx-pods-not-ready.md`, even though it was local. The format is the habit.

## Deliverables

- [ ] `charts/runway-service/` with a README explaining every default and why
- [ ] `platform/k8s/local/` + `make kind-up` / `make kind-down`
- [ ] Runway's three workloads running locally with probes, requests, PDBs, spread, quotas, default-deny NetworkPolicy
- [ ] `docs/runbooks/pod-failure-triage.md`: six states, symptoms, three commands each, likely causes
- [ ] `docs/architecture/k8s-apply-path.md`: your diagram of `kubectl apply` → running pod
- [ ] Gateway API cut-over checklist (input to ADR-009)
- [ ] One incident record (S12)

## Completion criteria

- From a blank terminal, `make kind-up && make deploy-local` gives three Ready workloads in under 10 minutes.
- You diagnose all four S12 variants plus S09 **in under 5 minutes each**, from `kubectl` alone, with the fault injected blind.
- You draw the `kubectl apply` → running pod path from memory, naming each component and what it writes.
- You explain requests versus limits and how each affects scheduling, QoS, HPA, throttling and OOM, with the numbers from lab 5.
- `kubectl auth can-i --list --as=system:serviceaccount:cargo:cargo-api` shows only what the workload needs, and you can justify each verb.
- Removing any single NetworkPolicy allow rule breaks exactly the path you predict.

## Mentor checkpoints

- **Before you build:** bring the `runway-service` chart's values schema and your default choices (especially "no CPU limits"). Expect to defend each default.
- **During:** bring lab 5's latency numbers and lab 12's cut-over checklist.
- **Questions I will ask:** Why does a pod "come back"? What removes a pod from a Service? Why is a liveness probe on the database dangerous? What does `ndots:5` cost you?
- **The failure I will inject after you think you're done:** a fault that makes some pods healthy and some not, *without* any pod restarting. Expect a scheduling-and-selector class of problem.

## Resources

1. **Kubernetes docs:** *Concepts → Workloads*, *Services, Load Balancing and Networking* (Service, EndpointSlices, DNS, Gateway API, NetworkPolicy), *Configuration → Resource Management for Pods and Containers*, *Tasks → Monitoring, Logging, and Debugging → Troubleshoot Applications*.
2. ***Kubernetes Up & Running*, 3rd ed.** Chapters on Pods, Labels, Service Discovery, ReplicaSets, Deployments, ConfigMaps & Secrets, RBAC.
3. **Kubernetes the Hard Way (Hightower).** Do it once, end to end, on local VMs. You are not learning to run clusters this way. You are learning what EKS hides.
4. **Gateway API docs (gateway-api.sigs.k8s.io):** *Concepts → API Overview, Roles and Personas*, and the Kubernetes blog post announcing the ingress-nginx retirement.

## Estimate vs actual

| | Hours | Cost |
|---|---|---|
| Planned | 30 | $0 |
| Actual | | |
| Variance note | | |
