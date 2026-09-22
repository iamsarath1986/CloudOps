# Step 12 — Certification Mapping

## 1. What the JD actually says

> *"Cloud certification or relevant technical certification is desirable, such as Associate-level cloud, DevOps or developer certification."*

**Desirable. Singular-ish. Associate level.** This is a tiebreaker and a screening convenience, not a requirement. Read it as: *one associate-level certification makes you easier to shortlist.* It does not say what you can do, and the interview will not be about it.

The correct posture: **certifications are a by-product of the roadmap, not a parallel track.** If studying for an exam is competing with building, you have inverted the priority. Every exam below is taken *after* the corresponding milestones, when most of the material is already muscle memory and preparation is two or three weeks rather than three months.

> Exam codes and names change. Verify the current code and objective domain on the vendor's site before booking — the mapping below is by *content*, not by code.

## 2. The mapping

| Certification | Relevance to this JD | Ready after | Prep on top | Worth it? |
|---|---|:--:|:--:|---|
| **AZ-900** Azure Fundamentals | Low on its own. It is the vocabulary, and you will have it by Milestone 3 anyway | M3 | 1 week | **Only if you want an early confidence win.** Skippable. Never list it once you hold AZ-104 |
| **AZ-104** Azure Administrator Associate | **Highest.** This *is* the "Associate-level cloud certification" the JD names, and its domains — identity, storage, compute, networking, monitoring — are Milestones 3, 4, 6, 8, 9 | M9 | 3 weeks | **Yes. Take this one first.** If you take only one certification, this is it |
| **HashiCorp Terraform Associate** | **High.** Terraform is named explicitly in the JD and it is the highest-leverage skill in the roadmap. Cheap, short, and directly evidences a named requirement | M6 | 1–2 weeks | **Yes.** Best effort-to-signal ratio of anything on this list |
| **CKA** Certified Kubernetes Administrator | **High.** Kubernetes is named, and CKA is genuinely hands-on — it is a lab exam, so it evidences skill rather than recall | M11 | 3–4 weeks | **Yes, if Kubernetes becomes one of your depth areas.** More respected by engineers than any multiple-choice exam |
| **AZ-400** DevOps Engineer Expert | **Medium-high.** Covers the JD's "DevOps practices and CI/CD pipelines" directly, but it is an expert-level exam and assumes AZ-104 or AZ-204 | M13 | 4 weeks | **Later.** Strong for the *next* role. Do not rush it |
| **AZ-204** Azure Developer Associate | **Medium.** The JD does name a developer certification, and you have the coding background for it. But it overlaps AZ-104 less usefully for a platform role | — | 4 weeks | **Skip unless** you find yourself drifting toward application-platform work. AZ-104 covers the JD's intent better |
| **AZ-500** Azure Security Engineer | **Medium.** Security is a requirement but not the role's centre | M15 | 4 weeks | **Optional.** Only if security becomes your specialisation |
| **AI-102** Azure AI Engineer | **Low-medium.** The JD wants *exposure* to AI; a certification overshoots and risks repositioning you as an AI candidate | — | 3 weeks | **Skip.** Milestone 14's artifacts evidence the desirable far better |
| **CKAD** / **CKS** | Low / medium | — | — | **Skip.** CKAD is developer-facing; CKS only matters if you go deep on cluster security |
| **FinOps Certified Practitioner** | Low-medium. Cheap and short; "cost awareness" is in the JD | M12 | 1 week | **Optional.** Your cost-anomaly write-up is stronger evidence, but this is inexpensive if you enjoy it |
| **ITIL 4 Foundation** | **Medium, and underrated for this specific JD.** Six ITIL processes are in the role definition. It is a one-week, low-cost exam that lets you speak the employer's operational language precisely | M10 | 1 week | **Worth considering** — unusually well-aimed at this employer, and almost no infrastructure candidate has it |

## 3. The recommended sequence

| When | Certification | Why then |
|---|---|---|
| After **Milestone 6** | HashiCorp Terraform Associate | You have just written modules, managed state, and handled drift. Two weeks of revision, and it evidences a named JD requirement early |
| After **Milestone 9** | **AZ-104** | Its domains are exactly what you have just built and operated. This is the JD's "Associate-level cloud certification" |
| After **Milestone 10** *(optional)* | ITIL 4 Foundation | One week. Lets you speak precisely about the six processes the role definition names |
| After **Milestone 11** | CKA | You will have operated, upgraded and debugged a real cluster. The lab format rewards exactly that |
| After **Milestone 13** *(optional)* | AZ-400 | Positions you for the next role up, once everything it covers is something you have shipped |

**Two or three certifications is the right number for this JD.** Terraform Associate + AZ-104 satisfies the requirement comfortably. Adding CKA makes you clearly credible on the containerisation requirement. Beyond that you are collecting badges instead of building judgement, and experienced interviewers read a long certification list on a thin project history as a warning sign.

## 4. How to study without derailing the roadmap

- **Never study for an exam before building the thing.** The material becomes revision instead of learning, and takes a third of the time.
- **Use the exam's objective domain as a gap-finder.** Read it, tick off what you have built, and *only* study the remainder. For AZ-104 after Milestone 9, that will typically be Azure Files, Azure Backup for VMs, VNet peering details, and Entra ID features you have not needed.
- **Take exams during revision weeks** (see [Step 13](13-timeline-and-weekly-pattern.md)), not during build weeks. Never let an exam date push a milestone.
- **Put the practical evidence next to the certification on your CV.** "AZ-104" is a line. "AZ-104, plus a production AKS platform I upgraded without downtime and recovered from a timed DR drill" is a conversation.

## 5. What to say when asked about certifications

> *"I hold AZ-104 and the Terraform Associate, and I took both after I had built and operated the equivalent systems rather than before. The certifications confirmed coverage; the platform is where the judgement came from. I would rather discuss the DR drill where my runbook turned out to be wrong the first time."*

That answer positions the certification correctly and redirects to the evidence, which is where you want the interview to go.

→ Next: [Step 13 — Timeline](13-timeline-and-weekly-pattern.md)
