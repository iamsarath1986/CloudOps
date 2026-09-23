# Milestone M19 — AI Infrastructure and AI Workloads

*Phase 19 · ~4 weeks · Months ~20–21 · Prerequisites: [M18](M18-developer-platform.md) (AI quota as a self-service request), [M11](M11-sre-operations.md) (SLOs, incidents), [M10](M10-observability.md) (telemetry), [M16](M16-finops.md) (unit economics), [M13](M13-azure-core.md) and [M17](M17-golden-paths.md) (cross-cloud federation, golden path) · Unlocks: [M20](M20-ai-agents-mcp.md)*

## Why this phase exists

The JD lists AI as *advantageous*: *"Experience with the following would be advantageous: AI / Generative AI (GenAI), AI agents, Model Context Protocol (MCP) integrations."* For a platform engineer, AI competence means something specific. **Can you run a model-backed service in production like any other dependency, can you govern its access and cost across teams, and can you stop it doing something stupid?** You are operating AI here, not researching it.

AI arrives at month ~21, deliberately late. You can only operate what you already know how to run. By now you have SLOs, incident practice, cost allocation, cross-cloud identity and a golden path. This phase *applies* all of them to a new kind of dependency, and the phase is really about finding out which of those practices still hold and which need changing.

## Concepts to learn

**AI service architecture.** The layers are:
- caller
- AI gateway (authn, quota, cache, metering, routing)
- model endpoint, one of:
  - managed API: Azure OpenAI / AI Foundry, Amazon Bedrock
  - self-hosted server (vLLM, llama.cpp) on your nodes

Know where each responsibility lives:
- **The provider** owns model weights, serving capacity and content filters.
- **You** own identity, network path, quotas, retries, caching, cost attribution, validation and fallback.

**Managed AI services.**
- **Azure OpenAI:**
  - deployments (a model *version* pinned under a deployment name)
  - deployment types (standard, global, provisioned throughput)
  - regional model availability
  - quota in **tokens per minute (TPM)** and **requests per minute (RPM)**, allocated per deployment and per region
  - content filtering
  - Entra ID auth via managed identity, so there is no API key
  - private endpoints
- **Bedrock:**
  - model access per region
  - on-demand vs provisioned throughput
  - IAM-authorised invocation
  - cross-region inference profiles
  - guardrails
- Learn the *shape* of both, not every SKU. Model names and versions change constantly, so **version pinning and planned model retirement are EOL management** (the JD's EOL line applies to models too).

**The failure modes are different, so the operational habits must adapt.**

| Traditional service | AI dependency | What changes in practice |
|---|---|---|
| Latency roughly stable; p99 a few × p50 | Latency proportional to output tokens; highly variable; time-to-first-token (TTFT) ≠ total time | Separate SLIs for TTFT and total; percentile discipline matters more; timeouts sized per use case |
| Capacity = CPU/memory you control | Capacity = provider quota (TPM/RPM) you request and share | Capacity planning becomes quota planning; `429` is normal feedback, not an outage |
| Cost ∝ provisioned infra | Cost ∝ tokens per request | Traffic spike = bill spike; cost is an SLI; per-team budgets and rate limits are reliability controls |
| Deterministic output; tests pass/fail | Non-deterministic; "correct" is a distribution | Evaluation sets replace unit tests for behaviour; regressions are statistical |
| Config changes are low-risk | Prompt or model version change = behaviour change | Prompts and model versions are deployable artefacts: versioned, promoted, rolled back, recorded in telemetry and change records |
| Output is data you trust | Output is **untrusted input** | Schema validation before use, always; injection is a real vector |
| Errors are exceptions | "Wrong but plausible" is the dominant failure | Quality monitoring, human review of samples, fallback to non-AI paths |

**Reliability patterns, applied.**
- Timeouts per call.
- Retries with exponential backoff and **full jitter**, honouring `Retry-After`. Naive immediate retry turns a brief rate limit into a sustained outage that you caused yourself.
- A circuit breaker per backend.
- **Fallback chain:** primary model, then secondary provider, then a non-AI degraded answer (for exception summaries, a rule-based template).
- **Load shedding** and **queues.** Asynchronous processing absorbs variable latency and quota ceilings. Backpressure arrives as queue depth, not as user-facing errors.

**Caching.**
- Exact-match response caching keys on a hash of the normalised prompt, model version and parameters. It is cheap and safe for deterministic, low-temperature calls.
- Semantic caching trades correctness for hit rate. Understand it, but don't build it yet.
- Cache hit rate is both a cost lever and a latency lever.

**Rate limiting and quotas across teams.**
- Token-bucket limits per caller identity in Redis.
- Per-team TPM allocations carved out of the provider quota. This is the "noisy neighbour" problem again, now with tokens.
- Quota requests go through self-service (M18).

**Security and governance.**
- Managed identity rather than keys. Private endpoints.
- Decide and document what data may be sent: redact before the call, and decide what the provider retains.
- An approved-use register for models and purposes.
- OWASP Top 10 for LLM Applications, especially:
  - prompt injection
  - insecure output handling
  - sensitive information disclosure
  - model denial of service / unbounded consumption
  - supply chain
- Content filtering is the provider's guardrail. Your schema validation is yours.

**AI observability.** Everything from M10, plus:
- tokens in and out, and cost, per request
- TTFT and total latency
- `429` rate
- cache hit rate
- fallback rate
- validation-failure rate
- all broken down by `model_version`, `prompt_version`, caller team and backend

Alert on the **rate of change of cost**, not only the total.

**CI/CD for AI applications.**
- Prompts and model configuration live in Git.
- An eval set (20–30 cases with expected structured outputs and rubric checks) runs as a pipeline gate.
- Canary a new prompt or model version to a percentage of traffic through the gateway, and compare quality and cost before promoting.
- Rollback means reverting the version pin.

**Self-hosted inference, and when it is justified.**
- **Pros:** data never leaves you, predictable cost at high steady volume, and no provider quota.
- **Cons:** GPU capacity, drivers and node images, model-server tuning (batching, KV-cache memory), cold starts measured in minutes, and you now own model EOL.
- Understand GPU node pools (taints, the device plugin or GPU operator, cloud GPU quota) conceptually.

## Architecture decisions in this phase

**ADR-023 — AI gateway: build vs adopt; routing and fallback across Azure OpenAI and Bedrock**

| | |
|---|---|
| **Problem** | Several consumers (`insights-worker` now, `flightdeck` next, teams later) need model access. Without a shared layer, each would handle its own credentials, retries, caching, quotas, cost attribution and failover. That is duplicated risk with no central visibility. |
| **Options** | (a) Each service calls the provider SDK directly. (b) Adopt LiteLLM proxy. (c) Adopt Envoy AI Gateway. (d) Azure API Management's AI gateway policies. (e) Build a thin `ai-gateway` (Python/FastAPI) on AKS. |
| **Decision** | (e) for learning. The gateway provides Entra-authenticated callers, per-team token-bucket quotas in Redis, an exact-match cache, token and cost metering exported to AMP, `prompt_version`/`model_version` tags, per-backend timeouts and circuit breakers, and routing: **Azure OpenAI (primary; private endpoint; managed identity) → Bedrock (fallback; via AKS→AWS IAM OIDC federation, ADR-017) → non-AI degraded response.** |
| **Trade-offs** | You own code that mature products already do better. The ADR says so plainly: *at work I would evaluate (d) where APIM is already the enterprise gateway, or (b)/(c), before building.* Building it once is how you learn what to demand from a product. Cross-cloud fallback adds a second provider's quirks, a different model and different outputs, so the fallback model must pass the same eval set. |

**ADR-024 — Managed vs self-hosted inference**

| | |
|---|---|
| **Problem** | Should Runway run its own model? |
| **Options** | Managed only · self-hosted CPU small model · self-hosted GPU · hybrid. |
| **Decision** | Managed in production. A **timeboxed lab** runs a small open-weights model on CPU (llama.cpp/Ollama, or vLLM on CPU) behind the gateway as a third backend, used to *measure* latency, throughput and cost per 1k summaries against the managed backends. GPU is covered conceptually, with an optional one-day spot-GPU lab. |
| **Trade-offs** | Self-hosting only wins at high, steady volume, or when data-residency rules forbid managed endpoints. Runway has neither. The numbers from the lab are the argument. |

## Hands-on labs

1. **Keyless call.** Deploy an Azure OpenAI model deployment with Terraform (`azure-openai` module). Call it from a pod using Workload Identity. Prove that no key exists anywhere: search Key Vault, env vars and the Git history. *Proves: identity, not secrets.*
2. **Private path.** Add a private endpoint and a private DNS zone, then disable public network access. Verify from inside AKS, and verify that the public call fails. Repeat the S24 DNS trap on purpose. *Proves: data-path privacy.*
3. **Quota wall.** Lower the deployment's TPM. Drive k6 against the gateway. Observe `429` and `Retry-After`. Compare three retry strategies: none, naive immediate, and exponential with full jitter honouring `Retry-After`. Plot the success rate and total tokens attempted for each. *Proves: naive retries make rate limits worse.*
4. **Validate or fail closed.** Force malformed or partial JSON (a low `max_tokens`). Implement schema validation, one repair attempt, then a fail-closed fallback. *Proves: output is untrusted input.*
5. **Cross-cloud fallback.** Open the primary circuit breaker (block the private endpoint). Confirm traffic moves to Bedrock through federated identity and the fallback rate shows on the dashboard. Then break Bedrock too, and confirm the non-AI degraded response. *Proves: degradation, not outage.*
6. **Cache economics.** Replay a day of exception jobs twice, with the cache off and then on. Record hit rate, token savings and p95. *Proves: caching is a cost and latency lever.*
7. **Eval gate.** Build 25 eval cases from real (synthetic, redacted) shipment exceptions. Change the prompt so that two cases regress, and have the pipeline block the change. *Proves: prompts are code.*
8. **Self-hosted comparison (timeboxed).** Run a small open-weights model on a CPU node pool behind the gateway. Measure TTFT, total latency, throughput and the cost per 1k summaries including node cost. Feed the numbers into ADR-024. *Proves: you decide with data.*

## Project implementation

- `platform/modules/azure-openai`: account, model deployment(s) with pinned versions, private endpoint + DNS, diagnostic settings to Log Analytics, and Entra role assignments (Cognitive Services OpenAI User) for the gateway's identity only. Bedrock model access and an IAM role trusting the AKS OIDC issuer in `live/aws/...`.
- `workloads/ai-gateway/`: FastAPI on AKS, onboarded **through the golden path**. It provides:
  - OpenAPI endpoints `/v1/summarise`, `/v1/classify` and a generic `/v1/complete` for platform consumers
  - Entra token validation
  - per-team quotas from the AI-quota entries created by the M18 portal flow
  - a Redis token bucket and exact-match cache
  - backends as pluggable adapters
  - OTel spans with `gen_ai.*` attributes
  - metrics `ai_tokens_total{direction,team,model_version,backend}`, `ai_cost_usd_total`, `ai_ttft_seconds`, `ai_fallback_total`, `ai_validation_failures_total`, `ai_cache_hits_total`
- `workloads/insights-worker/` on EKS:
  1. `cargo-api` publishes shipment-exception events to SQS `shipment-exceptions` (with a DLQ).
  2. The worker calls the gateway **across clouds**, with the EKS pod getting an Entra token via the federated credential (ADR-017).
  3. It validates the structured result (`cause_category`, `summary`, `confidence`, `recommended_action`) and writes it to Postgres.
  4. On failure it applies the non-AI rule-based summary.
  5. KEDA or HPA on queue depth, with the scaling ceiling set by **quota**, not CPU.
- `prompts/` versioned in Git. Each version is referenced in telemetry and in the change record, and promoted through the gateway's canary percentage.
- `evals/` + an `ai-eval` reusable workflow run as a gate in `ai-gateway` and `insights-worker` pipelines.
- Dashboards (via the golden path plus an AI panel set):
  - tokens, cost per summary (unit economics, M16)
  - TTFT and total p95
  - `429` rate, fallback rate, validation failures, cache hit rate
  - queue depth and age
- SLOs:
  - insights freshness: 95% of exceptions summarised within 10 minutes
  - gateway availability: including degraded answers as "available", with a separate "AI-quality" ratio
- Alerts: `L2` on sustained fallback above 20%, cost derivative, and DLQ above 0; `L1` on queue age SLO burn, with a runbook. Per-team AI budgets in the M16 showback.
- `docs/security/ai-data-handling.md`: what may be sent, redaction rules, provider retention, the approved-use register, and the OWASP LLM Top 10 mapping to controls.
- Runbooks:
  - `docs/runbooks/ai-rate-limited.md`
  - `docs/runbooks/ai-provider-outage.md`
  - `docs/runbooks/ai-cost-spike.md`
  - `docs/runbooks/prompt-rollback.md`

## Engineering problems to hit deliberately

| Problem | How to cause it | What you should learn |
|---|---|---|
| Retry storm on 429 | Naive retry loop in the worker; halve TPM | Backoff + jitter + `Retry-After`; queue as shock absorber |
| Quota noisy neighbour | One team's batch job consumes shared TPM | Per-team quotas in the gateway; quota is capacity |
| Prompt bloat | Include the whole shipment history in the prompt | Tokens in drive cost and latency; measure input size per request |
| Silent quality regression | Upgrade model version without eval | Model version is a change; eval gate + canary + telemetry by version |
| Unvalidated output | Let malformed JSON reach Postgres | Validate before use; fail closed |
| Key reintroduced | "Temporarily" add an API key to test | Policy/scan should catch it (M15); keys are a regression |
| Model retirement | Pin a deployment version you then mark as retiring | Model EOL goes on the EOL register with a migration plan |

## Troubleshooting exercises

Full write-ups: **Detection → Investigation → Mitigation → Root Cause → Permanent Fix → Prevention**.

1. **S31 — "The AI feature is slow and erroring."** Queue age climbs and the fallback rate is 60%. The cause is TPM exhaustion, made worse by a retry change that was merged yesterday. The trap is to scale out the worker, which makes it worse. The mitigation is to shed or slow consumers and revert. The permanent fix is to make retry policy a gateway concern rather than something each client decides.
2. **S32 — "AI spend rate spiked overnight."** Your cost-derivative alert fires. Plant one of: the cache disabled by a config change, prompt bloat, or a worker redelivery loop (visibility timeout shorter than processing time, so SQS re-delivers and every message is summarised three times). Attribute the spike to a change and a decision.
3. **"Summaries got worse this week, nobody changed anything."** The provider rolled the default model version on an unpinned deployment, or the fallback provider is silently serving 30% of traffic. Find it from the `model_version` and `backend` dimensions in telemetry. Prevention: pin versions, and alert on backend mix.
4. **"Everything AI is down in one region."** Simulate a provider outage. Verify the fallback chain end to end and measure what users actually experienced (the degraded-answer ratio).

## Deliverables

- [ ] `docs/adr/ADR-023-ai-gateway.md`, `docs/adr/ADR-024-managed-vs-self-hosted-inference.md` (with measured numbers)
- [ ] `platform/modules/azure-openai`; Bedrock access + federated role in Terraform
- [ ] `workloads/ai-gateway/`, `workloads/insights-worker/` on the golden path, with tests
- [ ] `prompts/`, `evals/`, and the `ai-eval` gate in CI
- [ ] AI dashboards, SLOs and alerts with runbooks; per-team AI budgets in showback
- [ ] `docs/security/ai-data-handling.md` with OWASP LLM Top 10 mapping
- [ ] EOL register entries for pinned model versions
- [ ] Incident records for S31 and S32

## Completion criteria

- Zero API keys exist for any model backend (search evidence attached). Every call is made by a workload identity over a private or federated path.
- With the primary provider blocked, the system keeps its insights-freshness SLO using fallback. You have **measured** the degraded-answer ratio and the extra latency.
- Under a quota squeeze, the retry strategy with jitter holds success rate and token spend within the bounds you predicted beforehand, and you can show the naive strategy's graph next to it.
- You can state, from dashboards: cost per summary, cache hit rate, p95 TTFT and total latency, and fallback rate. You can also state which prompt and model version produced last Tuesday's output.
- A prompt change that regresses the eval set cannot reach production.
- You can explain, without notes, how SLOs, capacity planning, testing, change management and cost control each change for AI workloads, with one concrete Runway example for each.

## Mentor checkpoints

- **Before building:** bring the gateway's OpenAPI spec, the routing and fallback state diagram, and your per-team quota model. I will ask what happens when both providers are slow rather than down.
- **Mid-phase:** bring the Lab 3 retry graphs and the Lab 8 self-hosted numbers.
- **Review questions:** "Why is AI output untrusted input? Give a correctness reason and a security reason." "Is a degraded answer 'available' for your SLO? Defend it." "What is your model EOL process?"
- **The failure I will inject after you think you are done:** something that makes the AI path more expensive without making it slower or less reliable. The category is a change that only your cost telemetry can see.

## Resources

- **Azure OpenAI / AI Foundry docs:** "Quotas and limits", "Manage quota" (TPM/RPM), "Model deprecations and retirements", "How to configure Azure OpenAI with Microsoft Entra ID authentication" (managed identity) and "Configure private endpoints".
- **Amazon Bedrock docs:** "Quotas", "Cross-region inference", "Guardrails", and "Identity-based policy examples" for model invocation.
- **OWASP Top 10 for LLM Applications** (genai.owasp.org): read every item, and map each one to a Runway control or an accepted risk.
- ***AI Engineering* (Chip Huyen):** chapters on evaluation methodology and on inference optimisation (latency, batching, caching), read selectively for operators.
- ***Release It!*, 2nd ed. (Nygard):** stability patterns (timeouts, circuit breaker, bulkheads, back pressure, shed load). Re-read them with the model as the integration point.

## Estimate vs actual

| | Planned | Actual | Variance note |
|---|---|---|---|
| Hours | 40 | | |
| Cloud + model cost this phase | $70–110 (tokens capped by budget) | | |
