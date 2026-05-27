# Business Data Document
## Project: Operation Biscuit Dunk Analytics (BDA-001)
**Version:** 2.0 | **Owner:** S. McVitie (Project Manager) | **Last Updated:** 2026-05-27
**lastUpdated:** 2026-05-27

---

## 1. Purpose

This document defines the data structures, KPI logic, reporting conventions, and Claude behaviour rules used across all BDA-001 dashboards. It is the single source of truth for analysts building or interpreting HTML reports within this project.

**Document responsibilities:**
- This document covers: data definitions, KPI formulas, filters, business rules, and Claude behaviour instructions
- `BDA001_report_config.json` covers: all visual styling — colours, typography, spacing, chart defaults, and component definitions

Any dashboard Claude generates must reference both files. In the event of any conflict, the JSON config takes precedence on visual matters and this document takes precedence on data and business logic matters.

---

## 2. Claude Behaviour Instructions

These instructions govern how Claude should communicate and present information across all BDA-001 dashboards and responses. They are non-negotiable and apply at all times.

### 2.1 Tone & Language

- **Meme literacy is mandatory.** Claude should pepper responses and dashboard copy with appropriate meme references. If a Hobnob scores highly, it is "no cap the GOAT." If a Rich Tea dissolves, it "caught an L." Use sparingly enough to land, not so frequently it becomes unhinged.
- **Gen-Z slang where appropriate.** Data insights should be delivered with energy. A high Dunkability Score is "absolutely slay." A DISSOLVE outcome is "lowkey giving soggy vibes." An ABORT is "not it, bestie."
- **Seriousness is sus.** Any response that reads like a corporate memo should be revised. If Claude produces something that sounds like a press release, something has gone wrong.
- **Dashboard tooltips and labels** may include brief slang annotations in brackets, e.g. `Structural Integrity Rate (no crumble, no cap)`.

### 2.2 Insight Delivery

- Lead with the most dramatic finding first. "Bestie the Digestive is eating — 94.2 Dunkability Score, that's unreal fr."
- Use rhetorical questions to frame data stories. "Wait, the Wafer dissolved at 71°C?? We don't stan that."
- When data is insufficient, Claude should say "not enough data to spill the tea on this one."
- Anomalies and data quality flags should be flagged as "something's giving off here 👀" before the technical explanation.

### 2.3 Boundaries

- Financial or compliance-related content (if ever introduced to this project) should revert to normal professional language. The memes stop where the auditors start.
- Avoid slang that is region-specific to the point of being unclear. The goal is vibes, not confusion.
- Do not use slang to obscure a metric definition. KPI formulas must still be precise. You can say "the slay score (Dunkability Score)" but the formula beneath it must be correct.

---

## 3. Data Sources

| Source Name | Description | Refresh Frequency | Format |
|---|---|---|---|
| `biscuit_trials.csv` | One row per biscuit trial — variety, dunk duration, outcome | Daily | CSV |
| `variety_master.csv` | Lookup table of all biscuit varieties and their attributes | Static | CSV |
| `tea_log.csv` | Temperature and brew strength readings per session | Per session | CSV |
| `tester_registry.csv` | Analyst IDs, names, and assigned biscuit categories | Static | CSV |

---

## 4. Column Definitions

### 4.1 biscuit_trials.csv

| Column | Data Type | Description | Example Value |
|---|---|---|---|
| `trial_id` | String | Unique identifier for each dunk trial | `TRL-00142` |
| `trial_date` | Date (YYYY-MM-DD) | Date the trial was conducted | `2026-04-15` |
| `analyst_id` | String | ID of the analyst who conducted the trial | `ANA-002` |
| `variety_code` | String | Foreign key to `variety_master` | `BRB-01` |
| `dunk_duration_sec` | Float | Duration of dunk in seconds | `4.2` |
| `pre_weight_g` | Float | Biscuit weight before dunking (grams) | `11.4` |
| `post_weight_g` | Float | Biscuit weight after dunking (grams) | `13.1` |
| `outcome` | String (enum) | Result of the dunk — see Outcome Codes | `CLEAN` |
| `tea_temp_c` | Float | Tea temperature at time of dunk (°C) | `82.5` |
| `brew_strength` | String (enum) | Brew strength — `WEAK`, `STANDARD`, `STRONG` | `STANDARD` |
| `notes` | String | Free text analyst notes | `Minor crumble at 3.8s` |

### 4.2 variety_master.csv

| Column | Data Type | Description | Example Value |
|---|---|---|---|
| `variety_code` | String | Primary key | `BRB-01` |
| `variety_name` | String | Common name of the biscuit | `Bourbon` |
| `category` | String (enum) | See Category Codes below | `CHOCOLATE` |
| `avg_thickness_mm` | Float | Average biscuit thickness in millimetres | `8.2` |
| `filling` | Boolean | Whether biscuit has a filling/cream layer | `TRUE` |
| `manufacturer` | String | Brand/manufacturer | `McVitie's` |
| `recommended_max_dunk_sec` | Float | Manufacturer's suggested maximum dunk duration | `5.0` |

---

## 5. Enumerated Values (Lookup Codes)

### 5.1 Outcome Codes

| Code | Label | Description |
|---|---|---|
| `CLEAN` | Clean dunk | Biscuit retrieved intact with no breakage or crumble |
| `CRUMBLE` | Minor crumble | Partial surface break — biscuit retrieved but degraded |
| `SPLIT` | Split | Biscuit broke into two or more pieces during retrieval |
| `DISSOLVE` | Dissolved | Biscuit disintegrated fully; lost in the cup |
| `ABORT` | Aborted | Trial abandoned (tester error, distraction, cold tea, etc.) |

### 5.2 Category Codes

| Code | Label |
|---|---|
| `CHOCOLATE` | Chocolate-coated or chocolate-flavoured biscuits |
| `PLAIN` | Plain, unfilled biscuits (e.g. Rich Tea, Digestive) |
| `CREAM` | Cream-filled biscuits (e.g. Custard Cream, Bourbon) |
| `OATY` | Oat-based biscuits (e.g. Hobnob, Flapjack style) |
| `SHORTBREAD` | Shortbread varieties |
| `WAFER` | Wafer-based biscuits |

---

## 6. KPI Definitions

All KPIs must be calculated exactly as defined below. Do not use alternative formulas without updating this document.

### 6.1 Dunkability Score (primary KPI)

> The headline metric. A composite score out of 100 reflecting overall dunk performance.

**Formula:**
```
Dunkability Score = (Clean Rate × 0.5) + (Avg Dunk Duration Ratio × 0.3) + (Absorption Ratio × 0.2)
```

Where:
- **Clean Rate** = `COUNT(outcome = 'CLEAN') / COUNT(all trials)` for that variety × 100
- **Avg Dunk Duration Ratio** = `MIN(avg_dunk_duration / recommended_max_dunk_sec, 1.0)` × 100
- **Absorption Ratio** = `AVG((post_weight_g - pre_weight_g) / pre_weight_g)` × 100, capped at 100

**Display:** Round to 1 decimal place. Show as a score out of 100.

---

### 6.2 Structural Integrity Rate

> Percentage of trials where the biscuit was retrieved without full collapse (i.e. outcome was not `SPLIT` or `DISSOLVE`).

**Formula:**
```
Structural Integrity Rate = COUNT(outcome NOT IN ['SPLIT','DISSOLVE']) / COUNT(all trials) × 100
```

**Display:** Round to 1 decimal place. Show as a percentage (%).

---

### 6.3 Average Dunk Duration

> Mean dunk duration in seconds across all non-aborted trials for the selected scope.

**Formula:**
```
Avg Dunk Duration = AVG(dunk_duration_sec) WHERE outcome != 'ABORT'
```

**Display:** Round to 2 decimal places. Show in seconds (s).

---

### 6.4 Tea Temperature Sensitivity Index (TTSI)

> Measures how much a biscuit's clean rate varies across different tea temperatures. Higher = more sensitive to temperature.

**Formula:**
```
TTSI = STDEV(clean_rate_per_temp_band) across temp bands: <75°C, 75–84°C, 85°C+
```

**Display:** Round to 2 decimal places. No unit. Lower is better (more consistent across temperatures).

---

### 6.5 Trial Volume

> Total count of completed trials (excludes `ABORT`).

**Formula:**
```
Trial Volume = COUNT(outcome != 'ABORT')
```

**Display:** Integer. No rounding.

---

## 7. Filters & Slicers

All dashboards should support the following standard filters unless otherwise specified:

| Filter | Field | Type | Default |
|---|---|---|---|
| Date range | `trial_date` | Date picker / range | Last 30 days |
| Biscuit category | `category` | Multi-select | All |
| Brew strength | `brew_strength` | Multi-select | All |
| Analyst | `analyst_id` | Multi-select | All |
| Outcome | `outcome` | Multi-select | Exclude ABORT |

---

## 8. Reporting Periods

| Period Label | Definition |
|---|---|
| MTD | Month to date — 1st of current month to today |
| QTD | Quarter to date — 1st of current quarter to today |
| YTD | Year to date — 1st January to today |
| Rolling 30 | Last 30 calendar days including today |
| Rolling 7 | Last 7 calendar days including today |

---

## 9. Business Rules & Edge Cases

- **Minimum trial threshold:** A variety must have at least **10 completed trials** before its Dunkability Score is displayed. Fewer than 10 = show "Insufficient data."
- **Duplicate trials:** If two trials share the same `trial_id`, retain only the most recent by `trial_date`. Flag duplicates in a data quality note.
- **Negative absorption:** If `post_weight_g < pre_weight_g`, set absorption to 0 for that trial and flag as anomalous.
- **Temperature outliers:** Exclude trials where `tea_temp_c < 50°C` or `tea_temp_c > 100°C` — these indicate sensor error.
- **Analyst attribution:** Trials with no `analyst_id` should be labelled "Unassigned" in any analyst breakdowns.

---

## 10. Glossary

| Term | Definition |
|---|---|
| Dunkability | The overall quality of a biscuit's performance when submerged in tea |
| Clean Rate | Proportion of trials with a CLEAN outcome |
| Structural Integrity | Whether the biscuit survives dunking without full collapse |
| Absorption Ratio | The relative weight gain from liquid absorption during dunking |
| Trial | A single controlled dunk of one biscuit in one cup of tea |
| TTSI | Tea Temperature Sensitivity Index — see Section 6.4 |

---

*End of document. For questions or amendments contact S. McVitie (Project Manager).*
