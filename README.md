# Is AI Helping My Business? (Practical Metrics Guide)

Yes—**if you measure outcomes, not just activity**.

This project is a practical guide for teams using GitHub Copilot who need a clear answer to: **“Is AI actually improving delivery, quality, and ROI?”** It helps you move from noisy usage stats to business-impact evidence.

## What this project contains

- `docs/metrics.md`: what GitHub Copilot Usage Metrics and Billing dashboards can (and cannot) tell you.
- `docs/dashboard-plan.md`: a concrete plan for joining Copilot usage with engineering outcomes (DORA, PR flow, quality) in a unified dashboard.

## The short answer framework

If you’re frustrated, use this sequence:

1. **Adoption** – Are licensed users active? (DAU/WAU, acceptance rate, feature usage)
2. **Efficiency** – Are teams shipping faster? (PR throughput, cycle time, lead time)
3. **Quality** – Is output still healthy? (test pass rate, review churn, defect rate)
4. **Cost** – Is spend justified? (credits consumed, overages, cost by model/feature)
5. **ROI** – Do gains outweigh AI spend? (time saved/value created vs. AI cost)

If adoption rises but efficiency/quality does not, AI is likely being used without business impact.

## What to measure (minimum viable scorecard)

| Area | Core metrics |
|---|---|
| Usage | Weekly active users, suggestion acceptance rate, chat/agent adoption |
| Delivery | PRs merged per engineer, cycle time, lead time to production |
| Quality | Test failure rate, rework/review rounds, post-release defects |
| Cost | Credits consumed, cost per user/team, overage risk |
| Business impact | Cost per productive PR, estimated hours saved, ROI trend |

## How to answer leadership in one sentence

“AI is helping when teams with sustained Copilot adoption show **faster delivery** and **stable or better quality** at a **cost that produces positive ROI**.”

## Suggested starting path

1. Use `docs/metrics.md` to establish baseline adoption and spend.
2. Use `docs/dashboard-plan.md` to design data joins with PR, CI/CD, and quality signals.
3. Publish a weekly trend view (not one-off snapshots) and compare high-adoption vs. low-adoption cohorts.

---

This repository is intentionally focused on **measurement clarity**: turning “people are using AI” into “AI is improving business outcomes.”
