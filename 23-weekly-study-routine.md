# Section 23 — Weekly Study Routine

The timeline in [Section 22](22-timeline.md) is made of weeks. This section is what one week looks like: roughly 10 hours, alongside a full-time job, repeated about 90 times. A two-year plan is decided by the routine, not by motivation.

---

## 1. The weekly pattern

| Day | Focus | Hours | The rule |
|---|---|:-:|---|
| **Mon** | Concepts | 1–1.5 | Read, draw, take notes. **No terminal.** Finish with three questions you can't answer yet |
| **Tue** | Hands-on lab | 1–1.5 | The milestone's small experiments, done in a scratch environment or locally. The work is meant to be thrown away |
| **Wed** | Project implementation | 1–1.5 | Build the real Runway increment. Small commits. A PR even when you are the reviewer |
| **Thu** | Implement + break it | 1–1.5 | Finish the increment, then run **one** failure exercise or troubleshooting scenario. **Never skip the failure** |
| **Fri** | Review + documentation | 1 | ADRs, runbooks, incident records, README, estimate-vs-actual. Answer Monday's three questions in writing. Check the week's cloud spend |
| **Sat or Sun** | Milestone block | 3–5 | One long session for work that needs unbroken time: `terraform apply` of a new layer, a cluster build, a migration, a DR drill, a game day |
| **Other weekend day** | Off | 0 | Not optional. See §6 |

Why the week is shaped this way:

- **Monday without a terminal.** Understanding before typing is the point of the roadmap. It's the difference between knowing why and knowing which button to press, and the "why" is what transfers between AWS and Azure.
- **Tuesday's labs are separate from Wednesday's project.** That way the project repository only gets work you already understand. Messy experiments stay out of the portfolio.
- **Thursday is for failure** because it forces the increment to be finished by Thursday. It also spreads the breakage across the week instead of piling it into the weekend.
- **Friday is documentation** because notes written the same day are honest and notes written a month later are fiction. The JD asks you to *"publish and maintain high-quality documentation… support procedures, operations manuals"*, and this is the habit that produces them.

---

## 2. How mentor sessions fit the week

You asked for mentoring, not instructions. Every major task runs through the same 14-step loop. The loop maps onto the week like this:

| Step | What happens | When | Who leads |
|:-:|---|---|---|
| 1 | Explain the concept | Mon | Mentor |
| 2 | Explain why we need it (which JD line, which failure it prevents) | Mon | Mentor |
| 3 | Explain how it works internally | Mon | Mentor |
| 4 | Show the architecture | Mon | Mentor, while you draw it back |
| 5 | Explain the alternatives | Mon | Mentor |
| 6 | Explain the trade-offs | Mon → your ADR draft on Fri | Both |
| 7 | Give you the implementation task | end of Mon / Tue | Mentor |
| 8 | You attempt it | Tue–Wed | **You** — no full solutions unless you ask for one explicitly |
| 9 | Review your implementation | Wed evening / Thu | Mentor |
| 10 | Identify problems | Thu | Mentor asks questions first; answers come later |
| 11 | Help you improve it | Thu | Both |
| 12 | Introduce a failure scenario | Thu or weekend | Mentor (you're told the symptom, never the cause) |
| 13 | You troubleshoot it | Thu / weekend | **You**, writing the timeline as you go |
| 14 | How a production engineer would handle it | Fri | Mentor, compared against your incident record |

**How to bring work for review.** Bring the smallest reviewable unit: a PR link, a plan output, a diagram, or an incident record. Include your own written answer to "what could go wrong with this?". A review of work you haven't questioned yourself is half as useful.

**When you are stuck:** after 45 minutes without progress, stop and write down three things: what you expected, what happened, and the last hypothesis you ruled out. Then ask. You'll get a hint or a question, not the answer. Ask explicitly for the full solution only when the learning goal isn't in that detail (for example, a provider quirk you only need to see once).

**Milestone checkpoints.** Every milestone file ends with *Mentor checkpoints*. Bring the design *before* you build it, and the evidence *after* you think you're done. Expect a failure to be injected at that point. That's normal and intended.

---

## 3. The weekend session template

A 3–5 hour block. The structure matters more than the length.

1. **`make up`** (10–25 min). If it fails, that's today's first incident. Open a record and work it before doing anything else.
2. **Re-read Friday's notes** (10 min), including the open questions and the estimate-vs-actual line.
3. **The main build work** (2–3 h): the step that needs uninterrupted focus.
4. **One timed failure exercise** (45 min), with the record written *during* the exercise. Detection → Investigation → Mitigation → Root Cause → Permanent Fix → Prevention.
5. **Commit, update the README numbers if any changed, then `make down`** (20 min). Check that the destroy really finished: look at the console or run a list command, not just the exit code. Orphaned load balancers, NAT gateways and disks are the classic surprise bills.

**Soak windows.** M11, M12, M14 and M19 each include one or two 48–72 hour windows where the environment stays up with synthetic traffic, so alerts, SLO burn and cost behave the way they do over time. Put them on the calendar in advance, set a budget alert for the window, and write a short operations log for each day.

---

## 4. The cloud-spend rhythm

| When | Where the work runs | Spend |
|---|---|---|
| Weekdays | Local: `kind`/`k3d`, docker-compose, `terraform plan`/`validate`/`test`, unit and contract tests | ~$0 |
| Weekend block | Real cloud, `make up` → `make down` | Most of the month's spend |
| Soak windows | Real cloud, left running, synthetic load | Planned spikes |
| Timeboxed windows | WAF, Multi-AZ, second region, VPN, resolver endpoints, GPU lab | Same-day destroy |
| Every Friday | Look at Cost Explorer and Azure Cost Management for the week, one line in `docs/cost/` | This habit *is* FinOps practice |

Rules that don't bend: budget alerts in both clouds exist before the second resource does; the $120 hard alarm is treated as a Sev-2 incident; and you never leave "I'll destroy it tomorrow" resources running over a work week.

---

## 5. The revision week protocol

Revision weeks (R1–R9) replace the normal pattern:

| Day | Revision activity |
|---|---|
| Mon | Re-read the last 2–3 milestones' concept notes. Write the one-page summary you would give a new teammate |
| Tue | Reproduce three earlier failure exercises **from memory**, timed. Anything you can't reproduce goes back on the list |
| Wed | Follow one of your runbooks literally, as a stranger would. Fix every stumble |
| Thu | Answer the milestones' completion-criteria questions out loud. If you hedge, you're not finished |
| Fri | Score the [competency matrix](24-competency-matrix.md) honestly. Update the README numbers. Update the plan if you have slipped |
| Weekend | Exam, if one is due. Otherwise rest. A revision weekend with no build is allowed |

---

## 6. Guarding against burnout

A 24-month self-funded plan alongside a full-time job usually fails through exhaustion around month five or month thirteen, not through difficulty.

- **One day a week with zero roadmap work.** Every week, including exam weeks.
- **A work crunch is a planned pause, not a failure.** The six slack weeks a year exist for this. Log the pause, `make down`, and restart with a revision day before new material.
- **Lower the floor instead of skipping the week.** When energy is low, a week can be Monday reading plus Friday documentation (2 hours). The streak matters more than the volume.
- **Signs to stop and re-plan:** three weekends in a row where the build didn't happen, a growing pile of un-written incident records, or dreading Monday reading. Re-plan with [Section 22 §8](22-timeline.md#8-when-you-fall-behind), don't push through.
- **Celebrate checkpoints A–D.** Write down what you can now do that you couldn't at the previous checkpoint. Progress over two years is invisible week to week and obvious at each checkpoint.
- **Protect sleep before hours.** Troubleshooting skill falls off sharply with fatigue. A tired 2-hour session that ends in a mess costs more than it gains.

→ Next: [Section 24 — Competency Matrix](24-competency-matrix.md)
