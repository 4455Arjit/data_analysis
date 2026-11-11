# Power BI & Full-Stack Analytics Portfolio: Projects 1, 2 & 3

Enterprise business intelligence portfolio developed by transitioning from full-stack software engineering into data analytics and analytics engineering. These projects demonstrate end-to-end data pipeline construction, REST API ingestion, multi-fact and single-fact Star Schema relational modeling, Query Folding on PostgreSQL, grain-safe DAX architecture, and executive dashboard suites.

---

## Project 1: Live Open-Source AI Telemetry Tracker

### Executive Overview
An automated executive monitoring dashboard that tracks real-time repository health, developer adoption, and engagement across top open-source TypeScript and AI repositories. Data is queried dynamically from GitHub's REST API, parsed through Power Query, and presented across core executive KPIs, category distributions, and community issue health.

### Architecture & Data Pipeline
* **Data Source**: Live [GitHub REST API v3](https://docs.github.com/en/rest) endpoints (`/search/repositories`).
* **Ingestion Engine**: Power Query M script utilizing `Web.Contents` and `Json.Document` with anonymous API handshakes.
* **Data Transformations**:
  * Flattened nested JSON structures (`owner`, `license`) into tabular dimensions.
  * Extracted and formatted ISO-8601 timestamps into standard date schemas.
  * Programmed custom M logic to compute `RepoAgeDays` from repository inception dates.
  * Built a categorical `PopularityTier` conditional column segmenting repos into `Emerging (<5k)`, `High Growth (5k–20k)`, and `Mega Repo (20k+)`.

### Dashboard Visualizations & Metrics
* **Executive Metric Cards**: Tracks **3M+ Total Stars**, **456K+ Forks**, **40K+ Open Community Issues**, and **100 Monitored Repositories**.
* **Top 10 AI Repositories by Stars**: Clustered horizontal bar chart featuring leading frameworks including OpenClaw, n8n, Firecrawl, Supabase, and Gemini CLI with explicit data labels.
* **Forks vs. Stars Correlation Matrix**: Bubble scatter visual evaluating repo popularity versus development forks, scaled by total open issue load to identify projects with high maintenance debt.
* **Interactive Slicers**: Multi-select filtering across `PopularityTier` cohorts and SPDX `LicenseType` classifications (MIT, Apache-2.0, AGPL-3.0, GPL-3.0).

---

<img width="1918" height="954" alt="image" src="https://github.com/user-attachments/assets/6e528aa3-3c23-479d-a4fc-8ba469c2acc2" />



## Project 2: Supabase B2B Pipeline & Star Schema Command Center

### Executive Overview
A multi-tier B2B CRM sales intelligence platform modeled after enterprise HubSpot/Salesforce pipeline structures. Hosted on a cloud PostgreSQL instance via Supabase, this solution tracks deal progression, sales velocity, regional conversion bottlenecks, and rep-level revenue contributions across $1.51M+ in closed revenue.

### Architecture & Data Modeling
* **Database & Hosting**: Cloud PostgreSQL instance on [Supabase](https://supabase.com/) configured with session pooling and custom SSL certificate chain verification.
* **Query Folding & SQL Layer**: Built a dedicated database view (`vw_fact_sales_pipeline`) in PostgreSQL to compute sales velocity (`days_to_close = close_date - engage_date`) and chronological pipeline ordering directly on the database engine prior to Power BI ingestion.
* **Star Schema Implementation**:
  * **Fact Table**: `fact_sales_pipeline` (500 opportunity records with monetary close values and transaction lifecycle timestamps).
  * **Dimension Tables**: 
    * `dim_accounts` (Enterprise clients, industry sectors, company revenue, employee headcount, headquarters).
    * `dim_products` (Hardware series catalog including GTX, MG, and enterprise GTK 500 tiers).
    * `dim_sales_teams` (Sales agents, sales managers, and regional offices: East, West, North).
  * **Relational Integrity**: Enforced strict `1:*` single-direction relationships from dimensions to fact table, with foreign keys hidden to guarantee clean report consumer navigation.
  * **Custom Chronological Sorting**: Overrode Power BI default alphabetical sorting on `deal_stage` by binding it to `stage_sort_order` (`Prospecting` → `Engaging` → `Won` → `Lost`).

### Analytical Findings & Visualizations
* **Executive Summary Cards**: Highlights **$1.51M Total Closed Revenue**, **500 Total Opportunities**, and a **37.84-Day Average Sales Velocity**.
* **Pipeline Conversion Funnel**: Visualizes deal progression volume across stages (50 in Prospecting, 100 in Engaging, 250 Won, 100 Lost).
* **Regional & Manager Performance Matrix**: Cross-tabulates Regional Office and Manager rows against deal stages, revealing:
  * **Regional Revenue Skew**: East office generated **$1,063,620 (70.3%)** of total company revenue, outperforming West ($272,928 / 18.0%) and North ($176,973 / 11.7%).
* **Root-Cause Decomposition Tree**: Dissects revenue flows from gross company performance down to individual reps and sectors:
  * **Rep Concentration Risk**: Sales rep Zoya Khan closed **$899,393** (59.4% of total company revenue), diversified across Retail, Finance, Healthcare, and Manufacturing.
* **Sales Velocity by Product Series**: Proves consistent sales cycle length across hardware tiers (~36.5 days for high-ticket GTK series vs. ~38 days for entry-level GTX/MG series), validating that premium pricing does not lengthen sales friction.

---
<img width="1920" height="976" alt="image" src="https://github.com/user-attachments/assets/6bad4091-8ac6-4c4a-b691-9851009cc297" />
<img width="1916" height="937" alt="image" src="https://github.com/user-attachments/assets/2f7838e0-4917-4749-bc71-52888868bfbc" />

## Project 3: Hospital Revenue Cycle & Clinical Utilization Intelligence

### Executive Overview
An enterprise clinical operations and revenue cycle analytics suite developed on a multi-table Electronic Health Record (EHR) dataset for **Massachusetts General Hospital** ([Maven Analytics Hospital Patient Records](https://mavenanalytics.io/data-playground/hospital-patient-records)). Spanning **27,891 encounters**, **47,701 clinical procedures**, **974 patients**, and **10 insurance payers** from **2011 to 2022**, this solution resolves complex multi-grain relationship traps to audit $101.5M in claims and deliver actionable clinical cost containment insights across a 3-page interactive suite.

### Architecture & Data Engineering (Power Query & Modeling)
* **Grain Trap Resolution (Header vs. Line-Item)**:
  * `encounters.csv` (27,891 rows) tracks visit-level claim totals, while `procedures.csv` (47,701 rows) records individual clinical line items performed during only 14,670 of those encounters.
  * Isolated financial metrics from procedure line-item counts via strict DAX context separation to prevent duplicate claim costs.
* **Power Query (M) ETL & Data Hygiene**:
  * Stripped synthetic numeric artifacts from patient identities using `Text.Remove`.
  * Standardized ISO-8601 UTC timestamps into localized date/time dimensions and calculated visit `Duration Hours`.
  * Segmented 10 individual payers into 3 macro categories: `Government` (Medicare, Medicaid), `Commercial` (Humana, Anthem, Aetna, UHC, Cigna), and `Self-Pay / Uninsured`.
  * Cleaned 70%+ unpopulated diagnosis codes by defaulting blank fields to `"Unspecified Visit"`.
* **Multi-Fact Star Schema Architecture**:
  * Generated a dedicated DAX `DimDate` calendar table spanning 2011–2022.
  * Wired single-direction `1-to-Many` relationships from `DimDate`, `patients`, and `payers` into `encounters`.
  * Connected `encounters[Id]` (1) to `procedures[ENCOUNTER]` (*), while hiding redundant foreign keys (`procedures[PATIENT]`) to eliminate ambiguous cross-filter paths.
  * Engineered patient cohorts: `Patient Age`, `Age Group` bins, and `Dead_alive` status (`Active / Living` vs. `Deceased`).

### Grain-Safe DAX Measure Library
* **Encounter-Grain Financials**: `[Total Claim Cost]`, `[Payer Coverage Amount]`, `[Uncovered Out-of-Pocket]`, `[Payer Coverage %]`, `[Avg Claim per Encounter]`, `[Median Claim per Encounter]`.
* **Procedure-Grain Clinicals**: `[Total Procedures]`, `[Total Procedure Cost]`, `[Procedure Attachment Rate %]`, `[Procedures per Active Encounter]`.
* **Patient-Grain Demographics & Pareto**: `[Unique Patients]`, `[Encounters per Patient]`, `[Mortality Rate %]`, `[Top 10% Patient Spend Share %]`.

### Core Analytical Findings & Strategic Impact

| Dashboard Page | Core Analytical Finding | Quantified Impact | Strategic Value & Decision Support |
| :--- | :--- | :--- | :--- |
| **Page 1: Financial & Payer Audit** | **Commercial Non-Payment & Underinsurance** | **$101.51M** gross claims vs. **$31.10M** covered (**30.63% coverage rate**), leaving **$70.42M uncovered**. | Uncovered that `NO_INSURANCE` drives **$49.26M** in bad debt, while 5 commercial payers paid only **$8.64K across $14.10M in claims (0.06%)**, shifting reimbursement reliance heavily onto **Medicaid (94.01%)** and **Medicare (77.96%)**. |
| **Page 1: Encounter Class Skew** | **Mean vs. Median Cost Distortion** | Urgent Care averages **$6,369** per encounter, but its median is only **$142.58** (vs. Inpatient median of **$2,355.59**). | Prevented executive misinterpretation of Urgent Care and Emergency (**$4,630 mean / $146.18 median**) as universally expensive by proving extreme outlier cases and low reimbursement rates (13.10%) distort unweighted averages. |
| **Page 2: Clinical & Procedures** | **Procedure Spend Dominance** | **47,701** procedures total **$105.52M**, occurring in **52.60%** of visits (**3.25** per active encounter). | Isolated **Electrical cardioversion** as the #1 cost driver at **$35.82M** (**33.95%** of all procedure spend across 1,383 procedures at **$25,903** avg), separating high-cost cardiac care from high-volume, flat-rate ($431) screenings. |
| **Page 2: Diagnosis Drivers** | **Volume vs. Acuity Divergence** | **Normal pregnancy** leads known diagnosis spend at **$20.92M** (1,341 visits), followed by **COVID-19** (**$1.23M**) and **Breast cancer** (**$1.16M**). | Helped resource planners distinguish between predictable high-volume maternity care and acute per-visit cost spikes like **Sepsis** (**$161,113** avg/visit) and **COVID-19** (**$47,170** avg/visit). |
| **Page 3: Patient Demographics** | **Pareto Super-Utilizers & Chronic Aging** | **974** patients average **28.64** encounters (**15.81%** mortality); the **top 10% (97 patients) drive 70.25% ($71.32M)** of all claims. | Proved the **80+ Living cohort** alone drives **12,282 encounters** and **$42.31M** in claims (**$31.78M uncovered**), providing clear ROI justification for targeted chronic disease care management. |

---

## Technical Skills Demonstrated Across Projects
* **Languages & DBs**: SQL (PostgreSQL / Supabase DDL, DML & Views), Power Query M Scripting, DAX.
* **Data Modeling**: Star Schema Architecture, Multi-Fact Modeling, Granularity Alignment, Cross-Filter Direction Control, Role-Playing Calendar Dimensions.
* **DAX & Optimization**: Grain-Isolated Measures, Statistical Skew Evaluation (Mean vs. Median), Pareto Curve Modeling, Query Folding Verification.
* **BI & UX**: Executive KPI Design, Multi-Page Clinical Drill-Throughs, Dynamic Tooltips, Slicer Isolation, Context-Aware Formatting.

<img width="1913" height="944" alt="image" src="https://github.com/user-attachments/assets/590d1e53-c2d0-420b-b05b-8aa973d006ad" />
<img width="1714" height="888" alt="image" src="https://github.com/user-attachments/assets/6afce1e1-8b0e-4f19-a594-3ea33987e495" />
<img width="1548" height="906" alt="image" src="https://github.com/user-attachments/assets/feedc861-7ed6-4a88-b14f-f89d299ad4a7" />

