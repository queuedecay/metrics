# GitHub Copilot: Usage Metrics vs. Billing Dashboard Comparison

> **Audience:** Enterprise admins, billing managers, and platform engineering leads on GitHub Enterprise Cloud (Copilot Business & Enterprise plans)  
> **Last updated:** June 2026

---

## Table of Contents

1. [Dashboard Comparison](#dashboard-comparison)
2. [2-Minute Walkthrough Scripts](#2-minute-walkthrough-scripts)
3. [Relevant GitHub REST API Endpoints](#relevant-github-rest-api-endpoints)
4. [APIs That Improve AI Usage Measurement](#apis-that-improve-a-companys-ability-to-measure-and-improve-internal-ai-usage)
5. [Remaining Gaps for Companies](#remaining-gaps-for-normal-companies)
6. [Using the Detailed Usage Report Export to Fill Gaps](#using-the-detailed-usage-report-export-to-fill-gaps)

---

## Dashboard Comparison

| Dimension | Copilot Usage Metrics Dashboard<br>*(Enterprise > Insights > Copilot Usage)* | Billing & Licensing AI Usage Dashboard<br>*(Enterprise > Billing and Licensing > Usage > AI usage)* |
|---|---|---|
| **Primary purpose** | Measure **adoption, engagement, and productivity impact** of Copilot | Track **cost, credit consumption, and budget governance** for AI features |
| **Key question answered** | "Are developers actually using Copilot, and is it helping?" | "How much are we spending, and where is the money going?" |
| **Metric focus** | Active users (daily/weekly), acceptance rates, lines of code generated, feature adoption, language/IDE breakdown | AI credits consumed, credit pool balance, cost per org/user/model, overage tracking |
| **Granularity** | Enterprise > Organization > Team > User | Enterprise > Organization > User > Model > Feature |
| **Time window** | Rolling 28-day window (daily drill-down); historical up to 1 year via API | Current billing cycle; historical up to 24 months |
| **Key KPIs shown** | Daily/weekly active users, Agent mode adoption rate, Lines added/deleted by Copilot, Suggestions accepted/rejected, IDE & language breakdown, Model usage trends | Total credits consumed (current cycle), Credits remaining in pool, Per-org/user consumption, Cost by model (GPT-4o, Claude, etc.), Overage charges, Budget utilization % |
| **What it tells you about features** | Which features are used (completions, chat, agent mode, CLI) and how often | Which features cost the most (agent mode sessions burn more credits than completions) |
| **What it does NOT show** | Cost, pricing, budget caps, or financial data | Whether developers find Copilot useful, adoption curves, or acceptance rates |
| **Who typically uses it** | Engineering leadership, DevEx teams, enablement leads | Finance, procurement, billing managers, CFO office |
| **Data freshness** | Up to 3 days lag (depends on IDE telemetry) | Near real-time (within hours) |
| **Export options** | NDJSON via API; CSV activity report download | CSV via Billing UI; JSON via Billing API |
| **Access required** | Enterprise owner, org owner, or custom role with "View Copilot Metrics" | Enterprise owner or billing manager |

### Key Insight

The **Usage Metrics dashboard** answers *"Is Copilot working for our developers?"* while the **Billing dashboard** answers *"Is Copilot within our budget?"* Together, they let you correlate adoption with spend--e.g., "Team X has the highest credit burn but also the highest acceptance rate and productivity gains."

---

## 2-Minute Walkthrough Scripts

### Script 1: Copilot Usage Metrics Dashboard (2 minutes)

---

**[0:00-0:15] Opening & Navigation**

> "Let's look at the Copilot Usage Metrics dashboard. From your Enterprise account, go to **Insights**, then **Copilot Usage**. This dashboard shows you whether Copilot is being adopted and whether it's actually helping your developers."

**[0:15-0:40] Top-Level Adoption Metrics**

> "At the top, you'll see the headline numbers: total assigned seats, daily active users, and weekly active users. These tell you at a glance what percentage of licensed developers are actually engaging with Copilot. If you're paying for 500 seats but only 200 are active weekly, that's your adoption gap."

**[0:40-1:05] Feature & Engagement Breakdown**

> "Scrolling down, you can see usage broken out by feature--code completions, Copilot Chat, agent mode, and CLI. This helps you understand which capabilities are gaining traction. You'll also see acceptance rates: how often suggestions were accepted versus dismissed. A rising acceptance rate means developers are finding the suggestions increasingly relevant."

**[1:05-1:30] Language, IDE, and Model Trends**

> "The next section breaks usage down by programming language and IDE. You can see if, for example, your Python teams have higher adoption than Java teams--which might signal a training opportunity. You'll also see which AI models are being used, useful now that Copilot supports multiple model backends."

**[1:30-1:50] Drill-Down by Org and Team**

> "Use the filters at the top to drill into a specific organization or team. This is where enablement leads get the most value--you can identify which teams need extra support, run targeted training, and track progress over time. You can also switch to user-level views to spot individual outliers."

**[1:50-2:00] Export & API**

> "Finally, you can export this data via the REST API in NDJSON format for custom dashboards in Power BI, Grafana, or your internal analytics platform. The API supports daily and 28-day reports at enterprise, org, and team levels. That's the usage side--next, let's look at billing."

---

### Script 2: Billing & Licensing AI Usage Dashboard (2 minutes)

---

**[0:00-0:15] Opening & Navigation**

> "Now let's look at the financial side. Navigate to **Enterprise > Billing and Licensing > Usage**, then select the **AI usage** tab. This is where you track how much Copilot is actually costing your organization under the new usage-based billing model."

**[0:15-0:40] Credit Pool & Consumption Overview**

> "At the top you'll see your enterprise's total AI credit pool--this is calculated based on your licensed seats. Copilot Business gives you 1,900 credits per user per month, Enterprise gives 3,900. Below that is your current consumption: how many credits have been used this billing cycle, the percentage of your pool consumed, and your projected end-of-cycle usage."

**[0:40-1:05] Breakdown by Organization and User**

> "The table below breaks consumption down by organization and individual user. This is critical for chargebacks--if your enterprise has multiple business units, you can see exactly which org is consuming what. Sort by highest consumption to find your power users and determine if any single team is driving disproportionate spend."

**[1:05-1:30] Cost by Feature and Model**

> "You can also filter by product feature--chat, agent mode, completions--and by model. Agent mode sessions are the biggest credit consumers because they involve multiple turns of reasoning. If you see agent mode costs spiking, it might be time to set per-user or per-org budget caps. Note that standard code completions remain unlimited and don't consume credits."

**[1:30-1:50] Budget Controls & Overage Policies**

> "On the right panel, you'll see your budget policies. You can set hard caps that block usage when credits run out, or allow overages at $0.01 per credit. This is also where you configure budget alerts--set a threshold like 80% to get notified before you hit your limit. These controls prevent bill shock from agentic workloads."

**[1:50-2:00] Export & Reconciliation**

> "For finance teams, you can export this data as CSV or pull it via the Billing REST API for integration with your internal cost management systems. Cross-reference this with the usage metrics dashboard to calculate your cost-per-productive-suggestion and build an ROI model. That's the billing side."

---

## Relevant GitHub REST API Endpoints

### Copilot Usage Metrics APIs

| Endpoint | Scope | Description |
|---|---|---|
| `GET /enterprises/{enterprise}/copilot/metrics/reports/enterprise-1-day?day=YYYY-MM-DD` | Enterprise | Daily usage metrics report (NDJSON download) |
| `GET /enterprises/{enterprise}/copilot/metrics/reports/enterprise-28-day/latest` | Enterprise | Rolling 28-day usage metrics report |
| `GET /orgs/{org}/copilot/metrics/reports/organization-1-day?day=YYYY-MM-DD` | Organization | Daily org-level usage report |
| `GET /orgs/{org}/copilot/metrics/reports/organization-28-day/latest` | Organization | Rolling 28-day org-level report |
| `GET /orgs/{org}/copilot/metrics/reports/users-1-day?day=YYYY-MM-DD` | User-level | Per-user daily metrics |
| `GET /orgs/{org}/copilot/metrics/reports/users-28-day/latest` | User-level | Per-user 28-day metrics |
| `GET /enterprises/{enterprise}/copilot/metrics/reports/user-teams-1-day` | Team mapping | Maps users to teams for team-level aggregation |
| `GET /orgs/{org}/copilot/metrics/reports/user-teams-1-day` | Team mapping | Org-level user-to-team mapping |

### Billing & Seat Management APIs

| Endpoint | Scope | Description |
|---|---|---|
| `GET /enterprises/{enterprise}/billing/usage` | Enterprise | AI credit/billing usage (filterable by org, user, model, product) |
| `GET /enterprises/{enterprise}/billing/ai/usage` | Enterprise | AI-specific billing usage |
| `GET /enterprises/{enterprise}/copilot/billing/seats` | Enterprise | All Copilot seat assignments and billing status |
| `GET /orgs/{org}/copilot/billing/seats` | Organization | Org-level seat assignments |
| `GET /orgs/{org}/copilot/billing` | Organization | Copilot billing summary for org |

### Required Permissions

- **Fine-grained PAT:** "View Copilot Metrics" (enterprise/org level) for usage APIs
- **Classic PAT scopes:** `manage_billing:copilot`, `read:enterprise`, or `read:org`
- **API version header:** `X-GitHub-Api-Version: 2026-03-10` (or latest)

---

## APIs That Improve a Company's Ability to Measure and Improve Internal AI Usage

### Tier 1: Essential for Measurement

| API | Why It Helps |
|---|---|
| **Enterprise 1-day metrics** (`enterprise-1-day`) | Enables daily trend tracking of active users, feature adoption, and acceptance rates across the entire enterprise. Build time-series dashboards showing adoption curves. |
| **Per-user metrics** (`users-1-day`) | Identifies individual adoption gaps. Find users who have seats but never engage, target them for enablement. Calculate per-user productivity metrics. |
| **User-teams mapping** (`user-teams-1-day`) | Enables team-level roll-ups without manually maintaining team rosters. Critical for identifying which teams benefit most and which need support. |
| **Billing usage** (`billing/usage`) | Tracks actual cost by org, user, model, and feature. Essential for ROI calculations (cost vs. productivity gains). |
| **Seat management** (`copilot/billing/seats`) | Identifies unused seats for reallocation. Ensures you're not paying for inactive users. |

### Tier 2: Strategic Improvement

| API | Why It Helps |
|---|---|
| **28-day rolling reports** | Smooths out daily noise for executive reporting. Shows sustained adoption rather than one-off spikes. |
| **Org-level metrics** | Enables org-to-org benchmarking. "Org A has 85% weekly active; Org B has 40%--what's different?" |
| **Billing filtered by model** | Understand cost efficiency across models. If Claude is cheaper per accepted suggestion than GPT-4o, you can guide model selection policies. |

### Tier 3: Automation & Governance

| API | Why It Helps |
|---|---|
| **Seat management + usage cross-reference** | Automate seat reclamation: if a user has zero activity for 30+ days, automatically remove their seat and reallocate. |
| **Billing + budget APIs** | Programmatically enforce budget policies, trigger alerts, and auto-pause orgs approaching limits. |
| **Daily exports to data warehouse** | Build longitudinal datasets spanning months/years for trend analysis, seasonal patterns, and correlating Copilot usage with engineering output metrics (PRs merged, cycle time, etc.). |

---

## Remaining Gaps for Normal Companies

Even with both dashboards and all available APIs, typical companies face these measurement gaps:

| Gap | Description | Impact |
|---|---|---|
| **1. No direct productivity/outcome measurement** | GitHub shows *usage* (suggestions accepted) but not *outcomes* (did the code ship faster? fewer bugs?). There's no link between Copilot activity and DORA metrics, cycle time, or defect rates. | Can't prove ROI beyond "developers used it." |
| **2. No code quality signal** | Acceptance rate does not equal quality. A developer might accept a suggestion that later causes a bug. No connection between Copilot-generated code and code review outcomes, test failures, or production incidents. | Risk of optimizing for volume over quality. |
| **3. No developer sentiment/satisfaction data** | Dashboards show behavioral data (clicks, acceptances) but not attitudinal data (does the developer feel more productive? less frustrated?). | Adoption might be high but satisfaction low (e.g., noisy suggestions in certain languages). |
| **4. No repository/project-level attribution** | Usage is tied to users and teams, not to specific repositories or projects. You can't answer "How much did Copilot contribute to Project X?" | Can't allocate AI value to business initiatives. |
| **5. No time-savings estimation** | No built-in way to estimate how much time Copilot saved. Lines of code generated does not equal hours saved (a 1-line fix might save hours of debugging). | Core ROI calculation requires external estimation models. |
| **6. No correlation with non-GitHub tools** | Copilot metrics live in GitHub's silo. No native connection to Jira tickets, CI/CD pipeline data, on-call rotations, or sprint velocity. | Holistic engineering intelligence requires manual data joining. |
| **7. No onboarding/ramp-up tracking** | Can't natively see how new developers' Copilot usage evolves over their first 30/60/90 days. | Hard to measure enablement program effectiveness. |
| **8. No prompt/context quality insights** | You know a suggestion was rejected, but not why. Was the prompt bad? Was the context insufficient? Was the model wrong? | Can't systematically improve developer-AI interaction quality. |
| **9. Limited historical depth for trends** | Usage metrics API only provides 1 year of history; billing provides 24 months. No native way to track multi-year adoption curves. | Long-term strategic reporting requires external archival. |
| **10. No A/B testing or experiment framework** | Can't natively compare outcomes between Copilot-enabled vs. disabled groups, or between different model configurations. | Evidence-based policy decisions require custom experimentation. |

---

## Using the Detailed Usage Report Export to Fill Gaps

The NDJSON detailed usage report (via API or CSV activity report via UI) provides per-user, per-day, per-feature granular records. Here's how to leverage it to address the gaps above:

### Gap 1: Productivity/Outcome Measurement

**How the export helps:**
- Export daily per-user Copilot activity (lines generated, features used, acceptance rates)
- Join with your DORA metrics pipeline (deploy frequency, lead time, MTTR) by user and date
- Build regression models: "Do weeks with higher Copilot usage correlate with shorter cycle times?"
- Calculate: `Copilot-active PRs merged / total PRs merged` to measure Copilot's share of output

**Example pipeline:**
```
NDJSON export -> Data warehouse -> JOIN with PR merge data (GitHub API) -> 
JOIN with deploy data (CI/CD) -> Correlation dashboard
```

### Gap 2: Code Quality Signal

**How the export helps:**
- Cross-reference per-user Copilot activity dates with code review data (comments, requested changes, rejections)
- Join with test failure data: "Do PRs created on high-Copilot-activity days have more/fewer test failures?"
- Track post-merge incident rates correlated with Copilot-heavy development periods

**Example analysis:**
```sql
SELECT user, copilot_lines_generated, pr_review_comments, test_failures
FROM copilot_usage 
JOIN pr_metrics ON user AND date
WHERE copilot_lines_generated > 0
GROUP BY user
```

### Gap 3: Developer Sentiment

**How the export helps:**
- Use per-user acceptance rate trends as a proxy for satisfaction (rising = finding it useful)
- Identify users whose acceptance rates are declining -> target for survey or 1:1
- Segment by language/IDE to find where Copilot underperforms (low acceptance = poor fit)
- Combine with periodic developer surveys, tagging respondents with their usage tier

### Gap 4: Repository/Project Attribution

**How the export helps:**
- The per-user daily export shows which languages were used
- Cross-reference with `git log --author` data to identify which repos a user committed to on each day
- Build approximate attribution: "User X generated 200 lines via Copilot on June 5; they committed to repos A and B that day"
- Not perfect, but gives directional project-level insight

### Gap 5: Time-Savings Estimation

**How the export helps:**
- Use lines of code generated + acceptance rates to model conservative time savings
- Apply industry benchmarks (e.g., "1 accepted Copilot suggestion saves ~30 seconds on average")
- Build user-segment models: `senior devs x 15 sec/suggestion` vs. `junior devs x 45 sec/suggestion`
- Export enables building these models at scale across thousands of users

**Example formula:**
```
Monthly time saved = SUM(accepted_suggestions x avg_time_per_suggestion x user_segment_multiplier)
```

### Gap 6: Cross-Tool Correlation

**How the export helps:**
- NDJSON export is warehouse-friendly (load into Snowflake, BigQuery, Databricks)
- Join with:
  - Jira: story points completed per sprint vs. Copilot activity
  - CI/CD: build success rates on Copilot-active days
  - PagerDuty: incident frequency vs. Copilot-generated code volume
- The export's per-user/per-day granularity is the join key that enables all cross-tool analysis

### Gap 7: Onboarding/Ramp-Up Tracking

**How the export helps:**
- Tag users by hire date (from HRIS system)
- Plot each user's Copilot usage trajectory from day 0 of seat assignment
- Build cohort curves: "Users onboarded in Q1 reached 80% weekly usage by day 45"
- Identify if enablement programs (training, docs) accelerate the ramp

**Example cohort analysis:**
```
Day 0-7:   avg 2 suggestions/day accepted
Day 8-30:  avg 15 suggestions/day accepted  
Day 31-60: avg 35 suggestions/day accepted (plateau)
```

### Gap 8: Prompt/Context Quality Insights

**How the export helps (partial):**
- Analyze acceptance rates by feature type (completions vs. chat vs. agent)
- Low acceptance in specific languages -> context/training data is weaker there
- Declining acceptance for a user -> possibly changed projects or languages
- Combined with qualitative surveys, identifies where GitHub could improve models

### Gap 9: Long-Term Historical Trends

**How the export helps:**
- Set up a daily cron job pulling the 1-day report into your data warehouse
- Build your own unlimited historical archive (GitHub only retains 1 year)
- Enables year-over-year comparisons, seasonal analysis, and long-term adoption curves

**Recommended architecture:**
```
Daily cron -> GET /enterprises/{ent}/copilot/metrics/reports/enterprise-1-day
           -> Parse NDJSON -> Load to data warehouse -> Retain indefinitely
```

### Gap 10: A/B Testing & Experimentation

**How the export helps:**
- Use seat management API to create control groups (assign/remove seats for specific users)
- Compare exported metrics between treatment (Copilot-enabled) and control groups
- Measure: acceptance rates, lines generated, and then cross-reference with PR velocity, bug rates
- Not a true randomized experiment, but enables quasi-experimental designs

---

## Summary Recommendation

For maximum insight, companies should:

1. **Enable both dashboards** -- Usage Metrics for engineering leadership, Billing for finance
2. **Set up daily API exports** -- Archive NDJSON data to a warehouse for longitudinal analysis
3. **Build cross-system joins** -- Connect Copilot data with DORA metrics, HRIS, and project management tools
4. **Establish a measurement framework** -- Define success metrics *before* analyzing (avoid cherry-picking)
5. **Combine quantitative + qualitative** -- Supplement usage data with developer surveys quarterly
6. **Automate governance** -- Use seat and billing APIs to auto-reclaim unused seats and enforce budget caps

---

*Document generated June 2026. API endpoints and dashboard features subject to change--refer to [GitHub Docs](https://docs.github.com/en/copilot) for the latest.*
