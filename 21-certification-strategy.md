# Section 21 — Certification Strategy

## 1. What the JD actually says

> *"Cloud certification in **AWS and/or Azure**, such as: AWS Certified Solutions Architect – Professional · AWS Certified DevOps Engineer – Professional · Microsoft Certified: Azure Solutions Architect Expert · Or equivalent certifications"*

It helps to read this closely:

- **All three named certifications are Professional or Expert level.** None of them is an Associate. The panel expects someone who has already been through the associate tier and gone further.
- **"and/or"**: one of them meets the stated bar. Holding one per cloud is what makes "proficiency in managing and operating AWS and Azure" believable on paper.
- **Certifications aren't listed under "must meet the following requirements".** That list is only the degree and the years of experience. So certifications are a strong screening signal, not a gate. The interview will still be about what you have built and run.

The approach doesn't change: **certifications come out of the roadmap, not alongside it.** Every exam below is taken *after* the milestones that cover its content. By then most of the material is muscle memory, and preparation takes 2–4 weeks of gap-filling instead of 3 months of learning from scratch. If an exam is competing with the build, you have the priority backwards.

> Exam codes, names, prerequisites and prices change. Check the current exam guide and objective domain on the vendor's site before you book. The mapping below is by **content**, not by exam code.

---

## 2. The mapping

| Certification | Level | Relevance to this JD | Ready after | Prep on top | Worth it? |
|---|---|---|:-:|:-:|---|
| **HashiCorp Terraform Associate** | Associate | **High.** Terraform is the IaC tool the JD names. The exam is cheap and short, and it is the earliest credible signal you can earn | [M05](milestones/M05-terraform.md) + [M07](milestones/M07-cicd.md) | 1–2 wks | **Yes.** Best effort-to-signal ratio on the list. Take it at **R2 (~month 8)** |
| **AWS Solutions Architect – Associate** | Associate | **High, as a stepping stone.** Not named, but it covers the ground the SA Professional assumes: VPC, IAM, compute choices, storage, HA patterns | [M09](milestones/M09-eks.md) | 2–3 wks | **Yes.** Take it at **R3 (~month 10)**. It checks the AWS half before you add a second cloud |
| **CKA** Certified Kubernetes Administrator | Professional-grade, hands-on | **High.** EKS and AKS are named. CKA is a practical lab exam, so it proves skill rather than recall, and engineers respect it more than any multiple-choice exam | [M09](milestones/M09-eks.md) + [M12](milestones/M12-capacity-ha-dr.md) | 3–4 wks (killer.sh) | **Yes.** Take it at **R4 (~month 14)**, after you have upgraded, broken and repaired a real cluster |
| **AZ-104** Azure Administrator Associate | Associate | **High, and required**: it is the prerequisite for the Azure Solutions Architect Expert certification | [M13](milestones/M13-azure-core.md) + [M14](milestones/M14-aks-multicluster.md) | 3 wks | **Yes.** Take it at **R5 (~month 16)** |
| **ITIL 4 Foundation** | Foundation | **Medium, and underrated here.** The role's own text names incidents within service levels, the Problem Management Process, and owning changes. A one-week exam lets you speak the employer's operations vocabulary precisely | [M11](milestones/M11-sre-operations.md) | 1 wk | **Optional, well-aimed.** Almost no infrastructure candidate has it |
| **AZ-305** → *Azure Solutions Architect Expert* | **Expert (JD-named)** | **Highest for Azure.** It tests design judgement: identity, governance, data, BCDR, infrastructure. M13–M18 are that content | [M18](milestones/M18-developer-platform.md) | 3–4 wks | **Yes.** Open the prep window at R7 (~month 20). Sit it at R8 or at the end (~month 22–24) |
| **AWS Solutions Architect – Professional** | **Professional (JD-named)** | **Highest for AWS.** Multi-account strategy, hybrid connectivity, migration, cost-aware design, and DR at scale. That is M04, M09, M12, M16 and M21 | [M16](milestones/M16-finops.md) + [M21](milestones/M21-hybrid-multicloud.md) ideally | 4–6 wks | **Yes.** Target **R8 (~month 22)**. If M21 isn't done yet, sit it at the end |
| **AWS DevOps Engineer – Professional** | **Professional (JD-named)** | **High.** CI/CD, IaC, monitoring, incident response and automation on AWS. M05, M07, M10 and M11 cover most of it | [M22](milestones/M22-hardening-capstone.md) | 4 wks | **Yes, but after the roadmap (~month 24–28).** Two Professional exams inside the 24 months already takes enough revision time |
| AWS Certified Security – Specialty / **AZ-500** | Specialty / Associate | Medium. Security matters but is not the centre of this role | M15 | 4 wks | **Only if** you pick security as your specialisation |
| **CKS** Certified Kubernetes Security Specialist | Specialty | Low–medium | M15 | 3 wks | **Only if** Kubernetes becomes your deep specialism. Requires an active CKA |
| **FinOps Certified Practitioner** | Foundation | Low–medium. FinOps is named, but your cost-anomaly write-ups are stronger evidence | M16 | 1 wk | Optional and cheap |
| AI-102 / AWS AI Practitioner / ML Specialty | — | Low. The JD wants AI *exposure* on a platform. An AI certificate risks presenting you as an AI candidate | — | — | **Skip.** The M19–M20 artefacts show the desirable skill better |
| AZ-900 / AWS Cloud Practitioner | Fundamentals | Very low at senior level | M03 | 1 wk | **Skip.** Don't list one once you hold an associate |

---

## 3. The recommended sequence

| When | Revision week | Certification | Why then |
|---|:-:|---|---|
| ~Month 8 | R2 | **Terraform Associate** | You have just written modules, handled state and drift, and put Terraform into a pipeline. Two weeks of revision |
| ~Month 10 | R3 | **AWS Solutions Architect – Associate** | Checks the AWS core (M04–M09) before Azure starts |
| ~Month 12.5 *(optional)* | after M11 | ITIL 4 Foundation | One week. The operations phase is still fresh |
| ~Month 14 | R4 | **CKA** | You have operated and upgraded EKS and diagnosed every pod failure state. The lab format rewards that |
| ~Month 16 | R5 | **AZ-104** | Azure core and AKS are fresh. It is also the prerequisite for the Expert certification |
| ~Month 20–24 | R7 → R8 or end | **AZ-305 → Azure Solutions Architect Expert** | Governance, identity and platform design (M13–M18) are the syllabus |
| ~Month 22–24 | R8 or end | **AWS Solutions Architect – Professional** | Multi-account, migration, hybrid and DR are all things you have done yourself |
| ~Month 24–28 | after the roadmap | **AWS DevOps Engineer – Professional** | Completes all three JD-named certifications |

**Honest framing.** Holding all three JD-named certifications by about month 28 is **ambitious but realistic**, because the project covers most of their content. If you have to choose, a **minimum credible set** for this JD is **Terraform Associate + CKA + one Professional/Expert per cloud** (AWS SA Pro and AZ-305). Past that point you're collecting badges, not building judgement, and experienced interviewers read a long certification list on a thin project history as a warning sign.

---

## 4. How to study without derailing the roadmap

1. **Never study for an exam before you have built the thing.** The material then becomes revision, not new learning, and takes a third of the time.
2. **Use the exam guide to find gaps.** Print the objective domain, tick what you have built and operated, and study only what is left. For SA Pro after M16, that will typically be organisation-scale networking (Transit Gateway, Direct Connect), migration tooling, and data services you haven't needed.
3. **One exam per revision week, at most.** A revision week has other jobs too (see [Section 23](23-weekly-study-routine.md)). Never let an exam date push a milestone.
4. **Prep happens in the Monday/Friday slots of the four weeks before the exam.** Build days stay build days.
5. **Practice exams show you gaps; they don't teach.** For every question you get wrong, find where it touches Runway and write one line in your notes about it. If it doesn't touch Runway, learn it briefly and move on.
6. **The Professional/Expert exams are scenario-driven.** Your ADRs are the best preparation you have: each one is a worked "given these constraints, choose" question.
7. **Watch the spend.** Exam fees add up. Book only when the build evidence already exists, and use retake-discount or bundle offers when they're available.

---

## 5. Putting certifications next to evidence

A certification is one line on a CV. The same line with evidence next to it starts a conversation:

| Instead of… | Write… |
|---|---|
| "CKA" | "CKA, plus an EKS cluster I upgraded with zero failed requests and six pod failure states I can diagnose in under five minutes" |
| "AWS Solutions Architect – Professional" | "AWS SA Pro, plus a multi-account Organizations landing zone in Terraform and a DR drill with measured RTO 22 min / RPO 5 min" |
| "Azure Solutions Architect Expert" | "Azure SA Expert, plus Entra ID federated as the workforce IdP for both clouds and Azure Policy governing cloud and on-prem resources through Arc" |

(Your own numbers go there, measured, never these examples.)

## 6. What to say when asked

> *"I took each exam after I'd built and operated what it covers. Terraform after my modules and state recovery, CKA after a live EKS upgrade, SA Pro after the multi-account and DR work. The certifications confirmed I had the coverage. The judgement came from the platform, and I'd rather walk you through the DR drill where my runbook turned out to be wrong the first time."*

That puts the certification in its place and steers the conversation to the evidence, which is where you want the interview to go.

→ Next: [Section 22 — The 12–24 Month Timeline](22-timeline.md)
