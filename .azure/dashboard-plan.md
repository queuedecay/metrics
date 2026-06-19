# Copilot Productivity Outcome Dashboard Plan
## Bridging Gap #1: Usage → DORA Metrics Correlation

**Objective:** Build a unified dashboard showing the correlation between Copilot usage (acceptance rates, lines generated) and measurable productivity outcomes (DORA metrics, cycle time, PR velocity).

**Target Platform:** Microsoft Fabric + Azure

---

## 1. Executive Summary

### Problem Statement
GitHub's Copilot Usage Metrics dashboard answers *"Are developers using Copilot?"* but not *"Is it making them more productive?"* To answer the latter, we must correlate:
- **Copilot activity** (per-user daily suggestions, acceptances, lines generated)
- **Engineering outcomes** (PR merge rate, deployment frequency, lead time, test pass rates)
- **Team metrics** (velocity, cycle time, code review cycle)

### Success Metrics
- Establish statistical correlation between weekly Copilot usage and weekly PR merged count
- Quantify time savings: "X accepted suggestions = Y hours saved" (by user segment)
- Show DORA metric improvement in high-adoption vs. low-adoption cohorts
- Enable org-level ROI calculation: cost per Copilot credit vs. productivity gains

---

## 2. Data Architecture

### Data Sources

| Source | Data | Frequency | API/Method |
|--------|------|-----------|-----------|
| **GitHub (Copilot Metrics)** | Daily per-user Copilot activity (lines, suggestions, acceptances, features) | Daily (3-day lag) | `GET /enterprises/{enterprise}/copilot/metrics/reports/enterprise-1-day` (NDJSON) |
| **GitHub (PR & Commit)** | PR metadata (author, merge date, files changed), commits by user/date | Real-time | GitHub REST API: `GET /repos/{owner}/{repo}/pulls`, Git log export |
| **CI/CD System** (Jenkins, GitHub Actions, Azure Pipelines) | Build success/failure, deployment timestamps, test results, pipeline duration | Real-time | Native API (GitHub Actions: `GET /repos/{owner}/{repo}/actions/runs`) |
| **Code Review System** | PR review cycles, review duration, requested changes, approval count | Real-time | GitHub REST API: `GET /repos/{owner}/{repo}/pulls/{pull_number}/reviews` |
| **Optional: HRIS** | Hire date, team assignment, seniority level (for user segmentation) | Weekly | CSV export or custom API |

### Data Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         DATA INGESTION LAYER                             │
├──────────────┬──────────────┬────────────────┬──────────────────────────┤
│   GitHub     │   GitHub     │   CI/CD        │    Code Review           │
│  Copilot     │   PR Data    │   Pipelines    │    (GitHub)              │
│  Metrics API │   (REST)     │   (Actions)    │                          │
└──────┬───────┴──────┬───────┴────────┬───────┴──────────────────────────┘
       │              │                │
       └──────────────┼────────────────┘
                      ▼
     ┌─────────────────────────────────────────────────┐
     │  Azure Data Factory (ADF)                       │
     │  - Orchestrate daily/hourly ingestion           │
     │  - Transform raw JSON/CSV → star schema         │
     │  - Run incremental copies                       │
     │  - Error handling & retry logic                 │
     └──────────────────┬──────────────────────────────┘
                        ▼
     ┌─────────────────────────────────────────────────┐
     │  Azure Data Lake Gen2 (Bronze/Silver/Gold)      │
     │  - Bronze: Raw ingested data (immutable)        │
     │  - Silver: Cleaned, deduplicated, typed         │
     │  - Gold: Analytics-ready (star schema)          │
     └──────────────────┬──────────────────────────────┘
                        ▼
     ┌─────────────────────────────────────────────────┐
     │  Microsoft Fabric (Lakehouse + Warehouse)       │
     │  - Synapse SQL: Fact & dimension tables         │
     │  - Semantic model: Measures & relationships     │
     │  - DirectQuery to Power BI                      │
     └──────────────────┬──────────────────────────────┘
                        ▼
     ┌─────────────────────────────────────────────────┐
     │  Power BI Dashboard                             │
     │  - Executive KPI cards                          │
     │  - Correlation scatter plots                    │
     │  - Cohort comparison (high vs. low adoption)    │
     │  - Time-series trend analysis                   │
     └─────────────────────────────────────────────────┘
```

---

## 3. Data Model (Star Schema)

### Fact Table: `fact_daily_developer_activity`

```sql
CREATE TABLE fact_daily_developer_activity (
  activity_key INT PRIMARY KEY,
  user_key INT FOREIGN KEY,
  team_key INT FOREIGN KEY,
  org_key INT FOREIGN KEY,
  date_key INT FOREIGN KEY,
  
  -- Copilot Metrics
  copilot_suggestions_generated INT,
  copilot_suggestions_accepted INT,
  copilot_lines_added INT,
  copilot_lines_deleted INT,
  copilot_chat_turns INT,
  copilot_agent_sessions INT,
  copilot_feature_type VARCHAR (50), -- completions, chat, agent, cli
  
  -- Developer Output
  commits_count INT,
  pr_created_count INT,
  pr_merged_count INT,
  lines_added_via_pr INT,
  files_changed_count INT,
  
  -- Code Review
  review_cycles_started INT,
  review_requests_received INT,
  review_approval_time_hours DECIMAL(6,2),
  
  -- Team Context
  is_copilot_active BIT, -- 1 if user has active seat, else 0
  days_since_copilot_assigned INT,
  
  CONSTRAINT fk_user FOREIGN KEY (user_key) REFERENCES dim_user(user_key),
  CONSTRAINT fk_date FOREIGN KEY (date_key) REFERENCES dim_date(date_key)
);
```

### Fact Table: `fact_weekly_dora_metrics`

```sql
CREATE TABLE fact_weekly_dora_metrics (
  dora_key INT PRIMARY KEY,
  user_key INT FOREIGN KEY,
  team_key INT FOREIGN KEY,
  week_starting_key INT FOREIGN KEY, -- date_key of Monday
  
  -- DORA Metrics (per user, aggregated weekly)
  deployment_frequency DECIMAL(5,2), -- deploys per week
  lead_time_hours DECIMAL(8,2), -- avg hours from commit to production
  mttr_hours DECIMAL(8,2), -- mean time to restore (hours)
  change_failure_rate DECIMAL(5,2), -- % of deploys causing failure
  
  -- Copilot Attribution
  prs_with_copilot_count INT, -- PRs containing >=50% Copilot-generated code
  prs_without_copilot_count INT,
  
  -- Test Quality
  test_pass_rate DECIMAL(5,2), -- % passing tests in PRs this week
  defect_count INT, -- bugs filed against code from this week
  
  CONSTRAINT fk_user FOREIGN KEY (user_key) REFERENCES dim_user(user_key),
  CONSTRAINT fk_week FOREIGN KEY (week_starting_key) REFERENCES dim_date(date_key)
);
```

### Dimension Tables

**`dim_user`**
```sql
CREATE TABLE dim_user (
  user_key INT PRIMARY KEY,
  github_login VARCHAR(255) UNIQUE,
  github_id INT,
  team_key INT,
  org_key INT,
  email VARCHAR(255),
  hire_date DATE,
  seniority_level VARCHAR(50), -- junior, mid, senior, staff (from HRIS)
  primary_language VARCHAR(50), -- most-used language in last 90 days
  is_active BIT,
  created_at DATETIME,
  updated_at DATETIME
);
```

**`dim_date`**
```sql
CREATE TABLE dim_date (
  date_key INT PRIMARY KEY,
  full_date DATE UNIQUE,
  year_month VARCHAR(7), -- YYYY-MM
  week_of_year INT,
  day_of_week VARCHAR(10),
  is_business_day BIT
);
```

**`dim_team`** & **`dim_org`** -- Standard dimensions with team/org names, hierarchies, budget allocations, etc.

---

## 4. Semantic Model (Fabric)

### Key Measures

```dax
// Copilot Adoption
Copilot Acceptance Rate = 
  DIVIDE(
    SUM(fact_daily_developer_activity[copilot_suggestions_accepted]),
    SUM(fact_daily_developer_activity[copilot_suggestions_generated])
  )

Copilot Lines Generated Per Day = 
  AVERAGE(fact_daily_developer_activity[copilot_lines_added])

Weekly Active Copilot Users = 
  DISTINCTCOUNT(fact_daily_developer_activity[user_key])

// Productivity
Avg PR Merge Time Hours = 
  AVERAGE(fact_daily_developer_activity[review_approval_time_hours])

Deployment Frequency (per user, per week) = 
  AVERAGE(fact_weekly_dora_metrics[deployment_frequency])

Lead Time (days) = 
  AVERAGE(fact_weekly_dora_metrics[lead_time_hours]) / 24

// Attribution
Pct Copilot-Influenced PRs = 
  DIVIDE(
    SUM(fact_weekly_dora_metrics[prs_with_copilot_count]),
    SUM(fact_weekly_dora_metrics[prs_with_copilot_count]) + 
      SUM(fact_weekly_dora_metrics[prs_without_copilot_count])
  )

Test Pass Rate = 
  AVERAGE(fact_weekly_dora_metrics[test_pass_rate])

// Correlation Indicators
Copilot Adoption Cohort = 
  IF(
    [Copilot Acceptance Rate] > 0.6,
    "High Adoption",
    IF([Copilot Acceptance Rate] > 0.3, "Medium", "Low")
  )
```

---

## 5. Dashboard Design (Power BI)

### Page 1: Executive Summary

**Layout:**
- **Top KPI Cards (4x):**
  - `Avg Weekly PR Merge Rate (High Adoption Cohort)`
  - `Avg Weekly PR Merge Rate (Low Adoption Cohort)` → % difference = ROI proxy
  - `Avg Lead Time (High vs. Low)`
  - `Test Pass Rate (High vs. Low)`

- **Key Correlation Scatter Plot:**
  - X-axis: Weekly Copilot Acceptance Rate (binned by quartile)
  - Y-axis: Weekly PR Merged Count (per user)
  - Color: Team
  - Size: Lines Generated
  - Trend line with R² value displayed

- **Time Series Overlay (2-Y axis):**
  - Left Y: Weekly Avg PR Merge Rate
  - Right Y: Weekly Copilot Acceptance Rate
  - Highlight correlation dips/spikes

### Page 2: Cohort Analysis

**Layout:**
- **Cohort Comparison Table:**
  - Rows: User cohorts (High Adoption, Medium, Low)
  - Columns: 
    - Avg Weekly PRs Merged
    - Avg Lead Time (hours)
    - Deployment Frequency
    - MTTR
    - Test Pass Rate
    - Defect Rate
  - Conditional formatting: Green/Red for better/worse performance

- **Onboarding Ramp Curve:**
  - X-axis: Days since Copilot seat assignment
  - Y-axis: Avg Acceptance Rate (by day cohort: 0-7, 8-30, 31-60, 61+)
  - Dual series: High-seniority vs. junior devs
  - Goal: Show junior devs' productivity gains accelerate with Copilot

### Page 3: Team & Org Drill-Down

**Layout:**
- **Org Selector Slicer** (dropdown)
- **Team Card Grid:**
  - Each card shows: Team name, Copilot adoption %, Avg weekly PRs merged (YoY Δ), Lead time (days)
  - Click to drill into team details

- **Team Detail View:**
  - Sankey diagram: Copilot-active users → PR volume → deployment frequency → MTTR
  - Per-user scatter: Copilot acceptance rate vs. PR merge count (bubble size = lines generated)

### Page 4: Predictive Insights (Advanced)

**Layout:**
- **Regression Model Card:**
  - Display R² and p-value for correlation between Copilot adoption and lead time reduction
  - Show confidence interval: "95% confidence: 1% increase in Copilot adoption → X hour reduction in lead time"

- **Time Savings Estimation:**
  - Formula: `SUM(copilot_suggestions_accepted) × 30 seconds per suggestion × seniority_multiplier`
  - Breakdown by team, org, user segment
  - Compare estimated hours saved vs. Copilot credit cost

- **Anomaly Detection Card:**
  - Highlight teams/users with unexpected patterns (e.g., high adoption but declining PR velocity)

---

## 6. Implementation Roadmap

### Phase 1: Foundation (Weeks 1-2)

**Deliverables:**
- Azure Data Lake Gen2 storage (Bronze/Silver/Gold containers)
- Azure Data Factory pipeline for GitHub Copilot metrics ingestion
- Star schema tables in Fabric Lakehouse/Warehouse (fact & dimension tables)

**Tasks:**
1. Provision Azure resources (ADL Gen2, ADF, Fabric Lakehouse)
2. Create GitHub PAT with "View Copilot Metrics" permission
3. Implement ADF pipeline:
   - Daily scheduled copy of enterprise 1-day Copilot metrics NDJSON
   - Transform NDJSON → normalized CSV (Bronze layer)
   - Load to Lakehouse (Silver)
4. Create `dim_user`, `dim_date`, `dim_team`, `dim_org` tables

**Owner:** Data Engineer

---

### Phase 2: Data Integration (Weeks 3-4)

**Deliverables:**
- ADF pipelines for GitHub PR, commit, and CI/CD data
- Fact tables populated with 30+ days of historical data
- Data quality checks in place

**Tasks:**
1. Ingest GitHub PR metadata (author, merge date, file count, review cycle time)
   - Use GitHub REST API or GraphQL bulk export
   - Schedule: daily incremental copy
2. Ingest CI/CD metrics (GitHub Actions: deployment timestamps, build status, duration)
   - Parse action run logs for deployment success/failure, duration
3. Implement T-SQL transformation logic in Fabric:
   - Join PR data + Copilot activity by (user, date) to derive per-PR Copilot attribution
   - Aggregate to fact_daily_developer_activity, fact_weekly_dora_metrics
4. Add data quality monitoring:
   - Alert if daily Copilot report missing
   - Check for data completeness (% of users reporting activity)

**Owner:** Data Engineer + Analytics Engineer

---

### Phase 3: Semantic Model & Dashboard (Weeks 5-6)

**Deliverables:**
- Fabric semantic model with DAX measures
- Multi-page Power BI dashboard
- Documentation & user guide

**Tasks:**
1. Create Fabric semantic model:
   - Import fact/dimension tables
   - Define relationships (fact → dim)
   - Author DAX measures (acceptance rate, PR merge rate, lead time, correlation indicators)
   - Test measure accuracy vs. raw data
2. Build Power BI dashboard:
   - Page 1: Executive summary (KPI cards + scatter plot + time series)
   - Page 2: Cohort analysis (table + ramp curves)
   - Page 3: Team/org drill-down (slicer + card grid + Sankey)
   - Page 4: Predictive insights (regression results + time savings)
3. Add interactivity:
   - Org/team/date slicers
   - Drill-through from cohort table to user details
   - Bookmarks for "High Adoption Cohort" vs. "Low Adoption Cohort"
4. Create executive summary document (1-pager) explaining dashboard + key findings

**Owner:** BI Developer + Analytics Translator

---

### Phase 4: Validation & Refinement (Week 7)

**Deliverables:**
- Validated dashboard accuracy
- Documented correlation findings
- User feedback incorporated

**Tasks:**
1. Reconciliation checks:
   - Compare GitHub Copilot dashboard metrics vs. Fabric totals (should match)
   - Validate PR merge counts, lead times against GitHub
2. Gather stakeholder feedback:
   - Engineering leadership: Do cohorts match your intuition about adoption?
   - Finance: Does ROI calculation look reasonable?
   - Data: Any data quality issues or gaps?
3. Refine dashboard:
   - Adjust time windows if needed (weekly → bi-weekly if data is noisy)
   - Add additional slicers based on feedback (e.g., by language, IDE)
   - Fix data issues (missing PR data, incorrect timestamps, etc.)
4. Document:
   - Data dictionary (fact/dimension table schema)
   - Dashboard user guide (how to interpret cohort analysis, regression metrics)
   - Known limitations (e.g., PR attribution is approximate; DORA metrics may include non-Copilot changes)

**Owner:** Analytics Team + Stakeholders

---

### Phase 5: Operationalization & Insights (Week 8+)

**Deliverables:**
- Automated monthly reporting
- Quarterly insights memo
- Continuous refinement backlog

**Tasks:**
1. Finalize ADF scheduling:
   - Daily Copilot metrics ingestion (3am UTC, after GitHub lag)
   - Daily PR/CI pipeline ingestion (5am UTC)
   - Weekly aggregation to fact_weekly_dora_metrics (Monday 6am)
2. Set up monitoring:
   - Alert if pipeline fails
   - Alert if data freshness > 24 hours
3. Establish reporting cadence:
   - Weekly: Dashboard auto-refresh, shared with engineering leadership
   - Monthly: Deep dive analysis email (key findings, anomalies, recommendations)
   - Quarterly: Strategic insights memo (ROI calculation, enablement recommendations, cost optimization)
4. Capture learnings:
   - Document unexpected findings (e.g., specific teams with strong correlation)
   - Identify improvement areas (e.g., languages with low adoption + weak correlation)
   - Recommend policy changes (e.g., invest in Copilot training for low-adoption teams)

**Owner:** Analytics Team (ongoing)

---

## 7. Key Metrics & Correlation Analysis

### Primary Outcome Metric
**Weekly PR Merge Rate by User** = `COUNT(PRs merged) / 7 days`

### Primary Predictor Metrics
- **Copilot Acceptance Rate** = `Suggestions Accepted / Suggestions Generated`
- **Copilot Lines Generated** = `Lines of code generated by Copilot per day (weekly avg)`
- **Time on Copilot** = `Days with any Copilot activity / 7` (proxy for engagement)

### Secondary Outcome Metrics (DORA)
- **Lead Time for Changes** = Avg hours from commit to production deployment
- **Deployment Frequency** = Deploys per week (or per day)
- **Mean Time to Restore (MTTR)** = Avg hours to resolve a production incident
- **Change Failure Rate** = % of deployments causing incident/rollback

### Correlation Hypothesis
> **H1:** Users with high Copilot acceptance rates (>60%) will show 15-25% higher weekly PR merge rates and 20-30% shorter lead times compared to low-adoption users (<30%).

> **H2:** Senior developers will show faster onboarding to Copilot (higher acceptance by day 30) and stronger correlation with productivity gains.

### Success Criteria for Gap #1
- ✅ Establish statistical significance (p < 0.05) for Copilot usage → PR velocity correlation
- ✅ Quantify ROI: "1 Copilot credit consumed = X hours saved" (by user segment)
- ✅ Identify which teams/languages show strongest correlation (where to invest enablement)
- ✅ Enable monthly reporting on Copilot's contribution to engineering productivity

---

## 8. Tools & Services Summary

| Component | Service | Notes |
|-----------|---------|-------|
| **Data Ingestion** | Azure Data Factory (ADF) | Orchestrates daily GitHub API calls, transforms JSON → SQL |
| **Data Lake** | Azure Data Lake Gen2 (ADLS) | Bronze/Silver/Gold storage structure; also backs Fabric Lakehouse |
| **Analytics Database** | Fabric Warehouse (Synapse SQL) | Hosts fact & dimension tables; supports DirectQuery from PBI |
| **Semantic Model** | Fabric Semantic Model | DAX calculations, relationships, measures |
| **Visualization** | Power BI Premium | Multi-page dashboard with drill-through, slicers, custom visuals |
| **Monitoring** | Azure Monitor / Data Factory alerts | Pipeline health, data freshness, failures |
| **Access Control** | Entra ID + Fabric RBAC | Role-based dashboard access (executive, manager, analyst) |

---

## 9. Estimated Costs (Monthly, Azure East US Pricing)

| Service | Usage | Cost |
|---------|-------|------|
| **ADF** | 10 pipeline runs/day, 5 min avg | ~$10 |
| **ADLS** | 100 GB stored | ~$4 |
| **Fabric Lakehouse** | 10 GB, 1 TB queries/month | ~$50-100 (depends on compute) |
| **Power BI Premium** | 1 capacity (10 cores) | ~$400 |
| **Azure Monitor** | 1 GB ingestion/day | ~$25 |
| **Total** | | **~$490-540/month** |

*Note: Costs scale with data volume and query concurrency. Capacity provisioning can be adjusted based on usage patterns.*

---

## 10. Risk Mitigation

| Risk | Mitigation |
|------|-----------|
| **GitHub API rate limits** | Use batch endpoints, cache responses, stagger requests |
| **PR attribution to Copilot uncertain** | Use conservative heuristic (>50% lines from Copilot), document assumptions |
| **Incomplete CI/CD pipeline data** | Start with GitHub Actions; extend to other systems incrementally |
| **Time lag between activity and outcome** | Use rolling correlation windows (e.g., weekly Copilot activity → following 2-week PR output) |
| **Confounding factors** | Control for team size, project size, deadline pressure—use statistical models |
| **Data quality issues** | Implement reconciliation checks, monitoring alerts, manual review process |

---

## 11. Success Metrics (Project Level)

- **On-time delivery:** All phases complete within 8 weeks
- **Data accuracy:** Copilot metrics ≥95% match GitHub dashboard totals
- **Adoption:** ≥80% of engineering leadership using dashboard weekly
- **Actionability:** Identify ≥2 teams for targeted Copilot enablement by week 8
- **ROI clarity:** Publish initial ROI estimate (cost per credit vs. hours saved) by week 8

---

## Next Steps

1. **Week 0 (Prep):**
   - Obtain GitHub Enterprise PAT (requires GitHub admin)
   - Identify sponsor org(s) for pilot
   - Reserve Azure resources (ADF, ADLS, Fabric capacity)

2. **Week 1 (Kickoff):**
   - Provision Azure resources
   - Begin Phase 1 implementation
   - Set up weekly sync with stakeholders

3. **Ongoing:**
   - Daily ADF pipeline monitoring
   - Weekly data quality review
   - Bi-weekly sync with engineering leadership to discuss emerging insights
