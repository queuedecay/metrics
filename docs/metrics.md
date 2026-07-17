# GitHub Copilot Metrics & Billing: 2026 Mid-Year Update

> **Audience:** Enterprise admins, billing managers, platform engineering, and DevEx leads on GitHub Enterprise Cloud  
> **Last updated:** 2026-07-17  
> **Scope of this update:** What changed in roughly the last month (late June to mid July), plus corrected API guidance.

---

## What changed recently (high-impact)

| Date | Update | Why it matters for measurement |
|---|---|---|
| 2026-06-19 | **`ai_credits_used` added to Copilot usage user-level reports** (`users-1-day`, `users-28-day`) | You can now correlate user adoption/productivity signals with per-user AI credit consumption inside the usage metrics dataset. |
| 2026-06-25 | **Cost centers support enterprise teams** | Cost attribution can follow enterprise team membership automatically (including SCIM-driven membership changes). |
| 2026-06-26 | **`total_pull_requests_merged` added by AI adoption phase** | Adoption-phase analytics now include absolute throughput, not only per-user averages. |
| 2026-06-30 | **Cost center user-level budgets introduced (API)** | One per-user budget can be applied to all members of a cost center; budget precedence became operationally important. |
| 2026-07-02 | **Copilot usage metrics accuracy/coverage improvements** | CLI LOC metrics, IDE attribution, and AI credit attribution are more complete/trustworthy. |
| 2026-07-02 | **Cost centers support AI credit pools** (included usage caps) | You can cap how much of the included shared pool a cost center can draw, separate from metered-spend budgets. |
| 2026-07-07 | **Adoption-phase review metrics added**: `avg_pull_requests_minutes_to_review`, `avg_pull_requests_review_cycles` | Enables downstream review-efficiency analysis by adoption phase in both 1-day and 28-day reports. |
| 2026-07-07 | **Per-user budgets in Billing UI for cost centers** | Controls that were API-only are now easier to run at scale from the billing UI. |
| 2026-07-07 | **Copilot Billing Preview app retirement announced (Aug 3, 2026)** | Teams should migrate to built-in Billing/AI usage pages + REST endpoints for reporting. |
| 2026-07-10 | **Per-user states endpoint for multi-user budgets** | You can page/filter/sort all users against a budget from one API endpoint instead of one call per user. |

---

## Dashboard reality check (usage vs billing)

| Dimension | Copilot Usage Metrics (Insights) | Billing / AI Usage |
|---|---|---|
| Core question | Are people adopting Copilot and how are they working? | Where is AI credit spend going and what gets billed? |
| Typical grain | Enterprise/org/user reports + adoption-phase breakdowns | Enterprise/org/user + model/product/cost center filters |
| History depth (API) | Up to ~1 year for usage-metrics reports | Up to 24 months for billing usage endpoints |
| Best for | Adoption, engagement, workflow health, throughput proxies | Cost controls, chargeback/showback, budget governance |
| Not a substitute for | Invoicing totals | Developer adoption and productivity context |

**Important distinction:** `ai_credits_used` in usage metrics is a **metrics signal**, not your invoicing source of truth. Billing endpoints/reports remain authoritative for billed totals.

---

## Corrected REST API map (Enterprise Cloud)

### Copilot usage metrics reports (NDJSON download links)

| Endpoint | Purpose |
|---|---|
| `GET /enterprises/{enterprise}/copilot/metrics/reports/enterprise-1-day` | Enterprise daily report (use `day=YYYY-MM-DD`) |
| `GET /enterprises/{enterprise}/copilot/metrics/reports/enterprise-28-day/latest` | Enterprise rolling 28-day report |
| `GET /enterprises/{enterprise}/copilot/metrics/reports/users-1-day` | Enterprise user daily report |
| `GET /enterprises/{enterprise}/copilot/metrics/reports/users-28-day/latest` | Enterprise user rolling 28-day report |
| `GET /enterprises/{enterprise}/copilot/metrics/reports/user-teams-1-day` | Enterprise user-team join report |
| `GET /orgs/{org}/copilot/metrics/reports/organization-1-day` | Org daily report |
| `GET /orgs/{org}/copilot/metrics/reports/organization-28-day/latest` | Org rolling 28-day report |
| `GET /orgs/{org}/copilot/metrics/reports/users-1-day` | Org user daily report |
| `GET /orgs/{org}/copilot/metrics/reports/users-28-day/latest` | Org user rolling 28-day report |
| `GET /orgs/{org}/copilot/metrics/reports/user-teams-1-day` | Org user-team join report |

### Billing usage / AI usage (JSON)

| Endpoint | Purpose |
|---|---|
| `GET /enterprises/{enterprise}/settings/billing/ai_credit/usage` | Enterprise AI credit usage (filter by org/user/model/product/cost center/time) |
| `GET /enterprises/{enterprise}/settings/billing/premium_request/usage` | Enterprise premium request usage |
| `GET /enterprises/{enterprise}/settings/billing/usage` | Enterprise billed usage report |
| `GET /enterprises/{enterprise}/settings/billing/usage/summary` | Enterprise usage summary |
| `GET /organizations/{org}/settings/billing/ai_credit/usage` | Org AI credit usage |
| `GET /organizations/{org}/settings/billing/usage` | Org billed usage report |
| `GET /organizations/{org}/settings/billing/usage/summary` | Org usage summary |

### Budgets & controls

| Endpoint | Purpose |
|---|---|
| `GET /enterprises/{enterprise}/settings/billing/budgets` | List enterprise budgets |
| `POST /enterprises/{enterprise}/settings/billing/budgets` | Create budget (enterprise, org, cost center, multi-user, user, etc.) |
| `GET /enterprises/{enterprise}/settings/billing/budgets/{budget_id}/user-states` | Per-user progress/state for multi-user budgets (added July) |

---

## Budget model updates you should account for

GitHub now clearly separates three control planes:

1. **User-level budgets (ULB):** always-on hard stops (pool + metered phases).
2. **Included usage controls for cost centers (AI credit pools):** cap draw from shared included pool.
3. **Metered budgets (cost center / org / enterprise):** cap paid usage after pool exhaustion.

### Precedence (most specific wins)

Individual user-level budget > cost center user-level budget > universal user-level budget.

Also, user-level budgets are independent from metered limits; whichever control has the lowest remaining headroom can effectively block first.

---

## Data export and reporting caveats

- Billing reports now include an **AI usage report** with model and username dimensions.
- The **detailed usage report** (all paid products) is available through the web flow (email link), and the REST `/usage` endpoints expose summarized usage views.
- For robust long-term trend analysis, keep a daily archive of:
  - Copilot usage metrics NDJSON reports
  - Billing AI usage JSON extracts

---

## Practical recommendations (post-update)

1. **Refresh your data model** to include new usage fields:
   - `ai_credits_used`
   - `total_pull_requests_merged`
   - `avg_pull_requests_minutes_to_review`
   - `avg_pull_requests_review_cycles`
2. **Replace deprecated reporting paths** (Billing Preview app) with:
   - Billing AI usage UI
   - `settings/billing/*` REST endpoints
3. **Implement budget observability** using multi-user budget user-states endpoint for proactive alerts.
4. **Separate governance metrics from invoicing metrics** in dashboards (usage signal vs billed truth).
5. **Adopt cost center team mapping** to reduce manual chargeback operations.

---

## Source links (selected)

### Changelog (June–July 2026)

- https://github.blog/changelog/2026-06-19-ai-credits-consumed-per-user-now-in-the-copilot-usage-metrics-api
- https://github.blog/changelog/2026-06-25-assign-enterprise-teams-to-cost-centers
- https://github.blog/changelog/2026-06-26-track-total-merges-by-adoption-phase-in-enterprise-and-organization-reports
- https://github.blog/changelog/2026-06-30-per-user-ai-credit-budgets-available-for-cost-centers
- https://github.blog/changelog/2026-07-02-improved-accuracy-and-coverage-in-copilot-usage-metrics-reports
- https://github.blog/changelog/2026-07-02-cost-centers-now-support-included-usage-caps
- https://github.blog/changelog/2026-07-07-add-review-cycles-and-time-to-adoption-phases-in-the-usage-api
- https://github.blog/changelog/2026-07-07-per-user-budgets-for-cost-centers-in-the-billing-ui
- https://github.blog/changelog/2026-07-07-copilot-billing-preview-app-will-be-retired-on-august-3
- https://github.blog/changelog/2026-07-10-per-user-states-for-multi-user-budgets-in-the-rest-api

### GitHub Docs / REST references

- https://docs.github.com/en/rest/copilot/copilot-usage-metrics?apiVersion=2026-03-10
- https://docs.github.com/en/enterprise-cloud@latest/rest/billing/usage?apiVersion=2026-03-10
- https://docs.github.com/en/enterprise-cloud@latest/rest/billing/budgets?apiVersion=2026-03-10
- https://docs.github.com/en/copilot/concepts/billing/budgets-for-usage-based-billing
- https://docs.github.com/en/billing/reference/billing-reports
- https://docs.github.com/en/billing/tutorials/automate-usage-reporting

---

If your internal dashboard currently uses older endpoint names (for example non-`/settings/billing/...` paths), prioritize a migration pass before your next monthly close.
