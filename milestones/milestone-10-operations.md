# Milestone 10 — Reliability and Operations

*Phase 9 · ~4 weeks · Prerequisite: Milestone 9*

> **This is the milestone that distinguishes you for this specific job.** Six ITIL processes are written into the role definition, and almost no self-taught candidate can evidence any of them.

**Learning topics**
* **Incident management**: restore service first, understand later; severity classification; the incident commander role even when it is only you; communication cadence; workarounds as a legitimate outcome within an SLA
* **Problem management**: an incident is an event, a problem is its cause; known-error records; not closing a problem when the incident closes
* **Service level management**: SLA vs SLO vs SLI, measurement windows, what a breach costs
* **Availability management**: availability maths, single points of failure, MTBF/MTTR, and why MTTR is usually the cheaper lever
* Root cause analysis: causal chains, contributing factors, and why "human error" is never a root cause
* Blameless postmortems; toil, and automation as its cure; on-call handover

**Build**
`docs/incidents/` and `docs/problems/` with templates. A severity matrix (Sev1–Sev4) with response and resolution targets, written **before** the first drill — you cannot classify under pressure without a pre-agreed scale. `platform/scripts/inject-fault.sh` covering at least eight failure modes. `docs/runbooks/incident-response.md` with the escalation policy.

**Cloud resources**
None new. This milestone operates what exists.

**DevOps work**
A game-day harness: a script that injects one of five failures at random while you are not watching. Being genuinely surprised is the point.

**Operations exercise**
Work the full catalogue in [Step 9 — the operations playbook](../09-operations-playbook.md). Every scenario runs **Detect → Investigate → Diagnose → Mitigate → Fix → Verify → Document**, with the timeline written *during* the incident, not afterwards. Then take three related incidents and write one RCA explaining all three — that is problem management.

**Security considerations**
Include at least one security-flavoured incident in the catalogue this milestone: an unexpected authorisation failure, or an anomalous egress destination.

**Expected outcome**
At least ten incident records and four RCAs, all from drills you actually ran, plus one monthly operational review: incidents, MTTR, SLO status, error budget consumed, top toil, and what you automated.

**Evidence of mastery**
Explain incident vs problem management with an example from your own records. Describe your first five minutes after an alert. Explain why you mitigate before you diagnose. Present an RCA and defend its causal chain against questioning.
