# Data dictionary — Startup Success Prediction

**Source:** Kaggle, `manishkc06/startup-success-prediction`, derived from Crunchbase.  
**Observation unit:** one US startup.  
**Original file:** `startup_data_original.csv`, 923 rows × 49 columns.  
**Analytical file:** `startup_data_analytical.csv`, 922 rows × 46 columns (one duplicate company removed, debris columns dropped, engineered columns and `split` added).

## Outcome

| Variable | Type | Meaning | Use |
|---|---|---|---|
| `status` | category | Final outcome: `acquired` (597) or `closed` (326) | Source of target |
| `closed` | 0/1 | Engineered target: 1 = closed (positive class) | Target |
| `labels` | 0/1 | 1 = acquired; exact copy of `status` | Excluded (leakage) |

## Identifiers and location

| Variable | Type | Meaning | Use |
|---|---|---|---|
| `id`, `object_id` | text | Crunchbase company id (`c:NNNN`); identical columns | Identifier only |
| `name` | text | Company name | Identifier only |
| `state_code`, `state_code.1` | text | US state; second column is a copy | Excluded; `state_code.1` dropped |
| `city`, `zip_code`, `latitude`, `longitude` | text / numeric | Location | Excluded (covered by state dummies) |
| `Unnamed: 0` | integer | Row index from an earlier export | Dropped |
| `Unnamed: 6` | text | Free-text address, 53% missing | Dropped |
| `is_CA`, `is_NY`, `is_MA`, `is_TX`, `is_otherstate` | 0/1 | State dummies; `is_otherstate` = reference | Features (except reference) |

## Dates and ages

| Variable | Type | Meaning | Use |
|---|---|---|---|
| `founded_at` | date | Founding date | Temporal split only |
| `closed_at` | date | Closure date; filled for all closed firms and 9 acquired | Excluded (leakage) |
| `first_funding_at`, `last_funding_at` | date | First and last funding round | Excluded (last = leakage) |
| `age_first_funding_year` | years | Firm age at first funding; 46 negatives set to missing | Feature |
| `age_last_funding_year` | years | Firm age at last funding | Excluded (cut short by closure) |
| `age_first_milestone_year` | years | Firm age at first milestone; missing = no milestone | Feature (+ missing indicator) |
| `age_last_milestone_year` | years | Firm age at last milestone | Excluded (cut short by closure) |
| `neg_first_funding` | 0/1 | Engineered flag: original first-funding age was negative | Audit only |

## Funding and network history (cumulative to data snapshot)

| Variable | Type | Meaning | Use |
|---|---|---|---|
| `funding_total_usd` | USD | Total funding raised; $11K to $5.7B | Via `log_funding_total` |
| `log_funding_total` | numeric | log(1 + funding_total_usd) | Feature |
| `funding_rounds` | count | Number of funding rounds (1–10) | Feature |
| `relationships` | count | Linked people: founders, executives, board members, advisors (0–63) | Via `log_relationships` |
| `log_relationships` | numeric | log(1 + relationships) | Feature |
| `milestones` | count | Recorded milestones such as launches or press events (0–8) | Feature |
| `avg_participants` | numeric | Average number of investors per round (1–16) | Feature |
| `has_VC`, `has_angel` | 0/1 | Has venture capital / angel funding | Features |
| `has_roundA` … `has_roundD` | 0/1 | Reached Series A, B, C, D | Features |
| `is_top500` | 0/1 | Undocumented "top 500" flag; date and source unknown | Excluded |

## Category

| Variable | Type | Meaning | Use |
|---|---|---|---|
| `category_code` | category | Crunchbase category, 35 levels | Error analysis only |
| `is_software`, `is_web`, `is_mobile`, `is_enterprise`, `is_advertising`, `is_gamesvideo`, `is_ecommerce`, `is_biotech` | 0/1 | Main category dummies | Features |
| `is_consulting` | 0/1 | Consulting (3 firms) | Excluded (too rare) |
| `is_othercategory` | 0/1 | All other categories | Reference level |

## Added by the analysis

| Variable | Meaning |
|---|---|
| `split` | `train` (590, founded 1984–Apr 2007), `validation` (147, Apr 2007–2008), `test` (185, 2009–2013) |
