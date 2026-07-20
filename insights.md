# HR Attrition Analysis — Business Insights & Recommendations

**Dataset:** IBM HR Analytics Employee Attrition (1,470 employees, 35 original variables)
**Overall attrition rate:** 16.1% (237 of 1,470 employees)
**Analysis date:** July 2026

---

## Executive Summary

Attrition at this organisation is not evenly distributed — it is **heavily concentrated in identifiable, addressable segments**. Three findings drive almost the entire problem:

1. **First-year employees leave at 34.9%** — 2.2× the company baseline. This single cohort accounts for 32% of all departures while representing only 15% of headcount.
2. **Employees working overtime leave at 30.5% vs 10.4%** — roughly 3× higher. Overtime affects 28.3% of the workforce and is the strongest independent predictor in the multivariate model.
3. **Risk concentrates in junior, frontline roles** — Job Level 1 (26.3%), Sales Representatives (39.8%), and Laboratory Technicians (23.9%).

Critically, **every major driver is something the organisation controls** — workload, onboarding, compensation structure, and career progression — rather than a fixed employee characteristic. This makes the problem actionable.

A separate salary equity audit found **no evidence of gender-based pay inequity** once job level is controlled for (gaps ≤2.5% and non-directional).

---

## Methodology

**Data preparation.** Removed three constant columns (`EmployeeCount`, `Over18`, `StandardHours`) carrying zero information. No missing values or duplicates were present. Ordinal 1–4 satisfaction scales were mapped to readable labels for reporting while retaining numeric versions for modelling. Continuous variables were banded (tenure, age, income, distance) for segment reporting.

**Diagnostic analysis.** Attrition rates were computed per segment and indexed against the 16.1% company baseline, so every finding is expressed as a multiple of the norm (e.g. "2.2× baseline"). Headcount was reported alongside every rate to guard against over-interpreting small samples.

**Predictive modelling.** A logistic regression with `class_weight='balanced'` was fitted on a stratified 75/25 train-test split. Because the target is imbalanced (16.1%), the model was evaluated on **recall and ROC-AUC rather than accuracy** — a naive "everyone stays" model achieves 84% accuracy while detecting zero leavers.

**Model performance:** ROC-AUC 0.819 · Recall (leavers) 0.678 · Precision (leavers) 0.421. The model identifies roughly two-thirds of leavers in advance. Lower precision is an accepted trade-off: the cost of a false alarm (offering support to a stable employee) is far lower than the cost of a missed departure.

---

## Key Findings

| Risk factor | Attrition rate | vs baseline (16.1%) | Headcount | Organisation can act? |
|---|---|---|---|---|
| Sales Representative | 39.8% | 2.47× | 83 | Yes — role design, compensation |
| Tenure 0–1 year | 34.9% | 2.16× | 215 | Yes — onboarding |
| Work-life balance "Bad" | 31.2% | 1.94× | 80 | Yes — workload |
| Overtime = Yes | 30.5% | 1.89× | 416 | Yes — workload |
| Job Level 1 (entry) | 26.3% | 1.63× | 543 | Yes — career pathway |
| Frequent business travel | 24.9% | 1.55× | 277 | Partially — travel policy |
| No stock options (Level 0) | 24.4% | 1.51× | 631 | Yes — compensation |
| Laboratory Technician | 23.9% | 1.48× | 259 | Yes — role design |
| Low job satisfaction | 22.8% | 1.42× | 289 | Yes — engagement |
| **Gender pay gap (within level)** | **≤2.5%** | **no gap** | 1,470 | Clean finding |

### What the multivariate model added

Univariate analysis alone would have been misleading. Two examples:

**Time since last promotion.** Raw comparison suggested leavers had *fewer* years since promotion (1.9 vs 2.2) — counterintuitive. This was **confounding**: leavers are disproportionately junior and simply have not accumulated time. After controlling for age and tenure, the model reverses the sign — longer time since promotion becomes a **significant risk factor** (OR 1.74–1.86, p < 0.001), matching intuition.

**Job level.** Univariate showed Level 1 with the highest attrition, but in the model `JobLevel` carries a positive coefficient because `MonthlyIncome` (strongly protective, OR 0.66 per SD) absorbs the protective effect. These two variables are collinear and must be interpreted jointly, not in isolation.

**Odds ratios for the strongest, statistically significant drivers** (p < 0.05): frequent business travel, overtime, Sales Representative and Laboratory Technician roles, time since last promotion, number of prior employers, and commute distance all increase odds of leaving. Higher income, job satisfaction, environment satisfaction, job involvement, work-life balance, tenure with current manager, and age are protective.

*Note on magnitude:* the unregularised model produces very large odds ratios for small-cell job roles (e.g. Sales Representative OR 17.7). The direction is robust; the exact magnitude is not. The conservative balanced model estimate (OR ~3.4) should be quoted instead.

---

## Recommendations

### 1. Structured First-Year Onboarding Programme

**Finding.** Employees in their first year leave at 34.9% — 2.2× the 16.1% baseline. This cohort accounts for 75 of 237 total departures (32%) while representing only 15% of headcount. Attrition then falls sharply: 18.1% (2–4 yrs), 11.1% (5–9 yrs), 10.4% (10+ yrs).

**Business impact.** Industry benchmarks place replacement cost at 50–100% of annual salary once recruitment, onboarding and lost productivity are included. Against a median monthly income of 4,919, each first-year departure represents an estimated 30,000–60,000 in replacement cost. Cutting first-year attrition by 10 percentage points would retain roughly 22 employees per year.

**Recommendation.**
- Assign a peer mentor from day one, distinct from the line manager.
- Formal structured check-ins at 30, 60, 90 and 180 days, each producing a documented action plan rather than an informal chat.
- Clarify career progression expectations within the first 90 days — Job Level 1 shows the highest level-based attrition (26.3%), suggesting unclear advancement pathways.
- Introduce mandatory exit interviews for all first-year leavers to capture root causes not observable in HRIS data.

**Success metric.** Reduce 0–1 year attrition from 34.9% to below 25% within 12 months. Track cohort retention at the 90-day and 12-month marks monthly.

---

### 2. Overtime and Workload Governance

**Finding.** Employees working overtime leave at 30.5% versus 10.4% for those who do not — approximately 3× higher. Overtime affects 28.3% of the workforce (416 employees). The multivariate model confirms overtime as one of the two strongest independent predictors (p < 0.001), meaning the effect persists after controlling for role, level, income and tenure. Employees reporting "Bad" work-life balance leave at 31.2% (1.94× baseline), a consistent signal.

**Business impact.** 416 employees are currently exposed to a risk factor that roughly triples departure probability. This is the largest addressable risk population in the organisation, and unlike role or tenure, workload can be adjusted immediately.

**Recommendation.**
- Audit overtime by team and manager to distinguish **chronic structural overtime** (a staffing problem) from **occasional peak-period overtime** (acceptable).
- Where overtime is chronic and sustained beyond one quarter, treat it as a headcount gap and open a requisition rather than absorbing it through existing staff.
- Set a monitored overtime threshold with automatic escalation to HR when an individual exceeds it for consecutive months.
- Include workload and work-life balance questions in quarterly pulse surveys, reported at team level.

**Success metric.** Reduce the share of workforce on regular overtime from 28.3% to below 20% within 12 months, and narrow the attrition gap between overtime and non-overtime groups from 20 points to under 12 points.

---

### 3. Sales Representative Role Review

**Finding.** Sales Representatives show the highest role-level attrition at 39.8% — 2.47× baseline. The Sales department overall runs at 20.6% versus 13.8% in Research & Development. Sales Representatives are also concentrated in Job Level 1 and lower income bands, both independently associated with higher risk.

**Caveat.** This finding rests on 83 employees, of whom 33 left. The direction is clear and statistically significant, but the precise rate is less stable than findings drawn from larger groups. This should be validated against a further period of data before major structural investment.

**Business impact.** Losing roughly two in five Sales Representatives annually creates continuous disruption to customer relationships and pipeline continuity, and imposes a permanent recruitment and ramp-up burden on the Sales function.

**Recommendation.**
- Review the compensation structure specifically for this role, including base-to-commission ratio and quota attainability — low base pay combined with variable earnings is a known driver of early-tenure sales attrition.
- Define a visible progression path from Sales Representative to Sales Executive with explicit criteria and timelines. Sales Executives show materially lower attrition (17.5%), suggesting progression itself is retentive.
- Conduct structured exit interviews with departing Sales Representatives over the next two quarters to identify whether the driver is compensation, targets, management, or role clarity — this dataset cannot distinguish between them.

**Success metric.** Reduce Sales Representative attrition from 39.8% to below 28% within 12 months; track internal promotion rate from Representative to Executive.

---

### 4. Compensation and Equity Participation Review

**Finding.** Employees with no stock option grant (Level 0) leave at 24.4% (1.51× baseline), while those holding options at Levels 1–2 leave at only 9.4% and 7.6% respectively. 631 employees — 43% of the workforce — currently hold no equity. Separately, monthly income is strongly protective in the model (OR 0.66 per standard deviation), and leavers earn 30% less on average than those who stay (4,787 vs 6,833).

**Note on causality.** Equity participation is likely correlated with tenure and seniority, so part of this effect reflects who receives options rather than the options themselves. However, the effect remains substantial and equity is a lever the organisation directly controls.

**Business impact.** The single largest at-risk population by headcount (631 employees) is defined by a compensation design decision rather than by performance or capability.

**Recommendation.**
- Extend stock option eligibility further down the organisation, prioritising Job Level 1 and 2 employees in high-attrition roles, with a standard vesting schedule to create genuine retention incentive.
- Conduct a compensation benchmarking review for Job Level 1 positions against market rates, since this level combines the highest attrition with the lowest pay.
- Note that percentage salary increase showed **no meaningful relationship** with attrition (15.2% vs 15.1% between stayers and leavers). Raising the size of annual increments is therefore unlikely to be effective — **absolute pay level, not increment size, is what correlates with retention.**

**Success metric.** Increase equity participation from 57% to 75% of workforce within 18 months; reduce Job Level 1 attrition from 26.3% to below 20%.

---

## Ethical Considerations

**Protected attributes were analysed but deliberately excluded from recommendations.** Gender, marital status and age were included in the diagnostic model to observe their statistical relationships. Marital status ("Single") did emerge as statistically significant (OR ~2.5). However, **no recommendation in this report is based on a protected characteristic.** Targeting employees for differential treatment on the basis of marital status, gender or age would be discriminatory and, in most jurisdictions, unlawful. Any risk-scoring model deployed operationally must have these fields removed.

**Salary equity finding.** Raw median pay is marginally higher for women (5,081 vs 4,837). After controlling for job level, gaps fall within ±2.5% with no consistent direction, indicating **no evidence of systematic gender-based pay inequity**. The raw difference is explained by role and level composition, not by differential pay for equivalent work.

**Individual risk scores must support, not penalise.** The model assigns each employee an attrition probability. These scores are intended to trigger **supportive intervention** — a career conversation, workload review, or development opportunity. They must never be used to withhold promotion, exclude employees from projects, or justify pre-emptive termination. Access should be restricted to HR personnel with a defined legitimate purpose.

**Correlation is not causation.** Every finding here is observational. Overtime is strongly associated with attrition, but this analysis cannot prove that reducing overtime *causes* retention to improve. Recommendations should be implemented as monitored interventions with before/after measurement, ideally piloted in selected teams first.

---

## Limitations

**Simulated dataset.** This is IBM's synthetic HR dataset. It contains no missing values or duplicates — unrealistically clean compared with real HRIS extracts, which typically require substantial reconciliation of duplicate records, inconsistent date formats and incomplete compensation history.

**No temporal dimension.** The dataset has no hire dates, termination dates or snapshot periods. Consequently no trend analysis, seasonality, or cohort-over-time comparison is possible, and Excel Timeline controls are not applicable. All figures represent a single point in time.

**Compressed performance ratings.** `PerformanceRating` contains only values 3 and 4 — no employee is rated below "Excellent". This rating inflation makes performance useless as a risk signal here, though in real organisations performance data is often informative.

**In-sample risk scores.** Risk probabilities were generated using a model trained on the same records, making them optimistic. A production implementation should use out-of-fold or fully held-out scoring.

**Multicollinearity.** Age, total working years, tenure, income and job level are strongly intercorrelated. Individual odds ratios for these variables should be interpreted jointly rather than independently.

**No qualitative data.** Exit interview content, engagement survey verbatims, and manager quality data are absent. These typically explain a substantial share of attrition variance and should be integrated before major structural investment.

---

## Appendix — Metric Definitions

**Attrition rate** = employees with `Attrition = "Yes"` ÷ total employees in segment. Computed as the mean of a 0/1 indicator.

**vs baseline** = segment attrition rate ÷ overall company attrition rate (16.1%). A value of 2.0 indicates twice the company-wide rate.

**Odds ratio** = exp(logistic regression coefficient). For standardised numeric predictors, interpreted per one standard deviation increase. For binary indicators, interpreted relative to the reference category. Values above 1 increase the odds of leaving; values below 1 are protective.

**Recall (leavers)** = of all employees who actually left, the proportion the model correctly identified in advance.

**Risk tier** = predicted attrition probability banded as Low (<0.30), Medium (0.30–0.60), High (>0.60).
