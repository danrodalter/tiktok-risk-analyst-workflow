# Glossary of Terms & Concepts: ROCC Project
**Complete reference guide for all technical, statistical, and business terminology**  
**Updated:** June 3, 2026

---

## Core System Architecture Terms

### Risk Operations Control Center (ROCC)
**Definition:** A real-time data analytics and monitoring system designed to detect policy violations on the TikTok platform, particularly sophisticated text evasion attacks.

**Key Characteristics:**
- Integrates multiple data sources (transaction queues, account validation, policy enforcement)
- Provides executive-ready dashboards with real-time metrics
- Includes automated alerting and incident response
- Maintains fairness monitoring throughout the system

**Business Impact:** Protects platform integrity while maintaining merchant fairness and user trust.

**Example in Context:** "The ROCC system detected 89 sophisticated evasion patterns that the legacy system missed."

---

### Text-Normalization Pipeline
**Definition:** The core processing engine that standardizes merchant text data to reveal hidden evasion attempts by removing obfuscation techniques.

**Core Capabilities:**
1. **Unicode Normalization (NFKC):** Converts all Unicode characters to a canonical form, preventing homograph attacks
2. **Hidden Character Removal:** Strips zero-width spaces, non-breaking spaces, tabs, and other invisible characters
3. **Character Substitution Detection:** Decodes alternative character representations (e.g., Cyrillic "о" → Latin "o")
4. **Diacritic Normalization:** Removes diacritical marks (ñ → n)

**Why It Matters:** Evasion artists use these obfuscation techniques to bypass keyword filters. The pipeline exposes the true underlying content.

**Example:**
```
Input: "buy ­­­cheap_­drugs" (with hidden zero-width spaces)
Output: "buy_cheap_drugs" → Detected as policy violation ✓
```

---

### Data Repair Protocol
**Definition:** A two-tiered system for recovering from data loss during network instability or system failures.

**Tier 1: Text Record Caching**
- Maintains local buffer of text records at edge nodes
- Prevents data loss during network transmission
- Preserves full context for decision-making

**Tier 2: Cohort-Median Interpolation**
- When a metric is missing, predict its value using historical median from similar merchants
- Ensures decisions aren't made with incomplete information
- Validation: check that interpolated values fall within historical ranges

**Why Important:** Network failures (15% packet loss in testing) cause data corruption. Without recovery, the system makes decisions with incomplete context, leading to false positives.

**Measurement:** System resilience monitored via "Telemetry Health Index" (target: 100% data completeness).

---

### Key Risk Indicators (KRI) Dashboard
**Definition:** Executive-facing monitoring system that tracks three primary operational metrics in real time.

**The Three KRIs:**

#### 1. Telemetry Health Index
- **Measurement:** Percentage of expected telemetry data successfully delivered
- **Current Status:** 82.0% (Target: 100%, Warning: < 90%)
- **Why It Matters:** If data delivery fails, safety models go blind
- **What Triggers It:** Network packet loss, system outages, queue overflow

#### 2. Pipeline False Positive Rate (FPR)
- **Measurement:** Percentage of legitimate activities incorrectly flagged as violations
- **Current Status Under Stress:** 22.2% (drops to 0.0% post-remediation)
- **Target Ceiling:** < 5.0%
- **Why It Matters:** False positives harm legitimate merchants (account suspension, revenue loss, manual review queue overflow)
- **Trade-off:** Balancing between catching violations (high recall) and not over-flagging (high precision)

#### 3. Model Significance Status
- **Measurement:** Statistical significance of system improvements (Chi-Squared test)
- **Current Status:** p-value < 0.000001, Chi-Squared = 87.0112
- **Interpretation:** 99.9999% confidence that improvements are real, not random chance
- **Why It Matters:** Ensures that claimed improvements are backed by rigorous statistical evidence

---

## Text Evasion & Security Terms

### Text Obfuscation / Character Evasion
**Definition:** Techniques used by malicious actors to hide policy-violating content while maintaining readability to humans.

**Common Techniques:**
1. **Zero-Width Space Injection:** Insert invisible spaces between characters (U+200B)
2. **Unicode Homograph Attacks:** Use characters from different scripts that look identical (Latin "a" vs. Cyrillic "а")
3. **Diacritical Abuse:** Add marks over characters (diacritics) to confuse matching
4. **Character Substitution:** Replace Latin characters with lookalikes (e.g., "1" for "I", "0" for "O")
5. **Mixed Scripts:** Combine characters from multiple Unicode blocks to break pattern detection

**Example Attack:**
```
Prohibited word: "cocaine"
Evasion version: "coc­­aine" (with zero-width spaces)
Legacy filter: No match ❌ (Spaces break keyword)
ROCC system: Detected ✓ (Normalizes and matches)
```

**Business Context:** Sophisticated evasion indicates organized fraud or spam campaigns, requiring elevated response.

---

### McNemar Test
**Definition:** A statistical test used to compare two classification systems on the same dataset to determine if one is significantly better than the other.

**When You Use It:**
- Both systems evaluated on identical data (paired comparison)
- Outcomes are binary (detected/not detected, true/false)
- Want to know if differences are statistically significant

**The 2x2 Table:**
```
              | System B: Caught | System B: Missed
System A:     |                  |
Caught        | C (both correct) | B (A caught, B missed)
Missed        | A (B caught, A   | D (both missed)
              |   missed)        |
```

**Interpretation in ROCC Context:**
```
Upgraded\Legacy | Caught (89) | Missed (0)
Caught (89)     | 0           | 89
Missed (0)      | 0           | 0
```
- A=0: Upgraded never missed what Legacy caught
- B=89: Upgraded caught 89 violations Legacy missed
- This split (89-0) is so extreme that Chi-Squared = 87.0112, p < 0.000001

**What It Means:** Virtual certainty that the upgraded system is genuinely better, not due to random variation.

---

### Chi-Squared Test
**Definition:** A statistical test that measures how far observed data deviate from expected data under the null hypothesis (no difference).

**Formula:** χ² = Σ((Observed - Expected)² / Expected)

**In ROCC Context:**
- Null Hypothesis: Legacy and upgraded systems have equal detection rates
- Expected (if null true): Roughly 50-50 split in caught/missed violations
- Observed: 89-0 split (upgraded caught all, legacy caught none)
- Result: χ² = 87.0112 (extremely high)
- p-value: < 0.000001 (probability of this observation if null were true)

**Interpretation:** If there were truly no difference between systems, we'd observe this result less than 1 in 1 million times. Therefore, the systems ARE significantly different.

**Why It Matters for Interviews:** Shows rigorous statistical validation, not anecdotal evidence.

---

### p-value
**Definition:** The probability of observing your data (or more extreme data) if the null hypothesis were true. Lower p-value = stronger evidence against null hypothesis.

**Common Thresholds:**
- p < 0.05: Statistically significant (95% confidence)
- p < 0.01: Highly significant (99% confidence)
- p < 0.001: Very highly significant (99.9% confidence)
- p < 0.000001: Extremely significant (99.9999% confidence) ← ROCC's result

**Interpretation in Plain English:**
- "p < 0.000001" means: "If the two systems were equally good, we'd see this outcome by random chance less than 1 in a million times. They're definitely different."

**Common Misunderstanding:**
- ❌ "p-value is the probability that my hypothesis is correct"
- ✅ "p-value is the probability of seeing this data IF the null hypothesis were true"

---

## Fairness & Compliance Terms

### Disparate Impact Ratio (DIR)
**Definition:** A metric measuring whether a system's enforcement decisions disproportionately affect certain groups, relative to others.

**Formula:** DIR = (Enforcement Rate for Group B) / (Enforcement Rate for Group A)

**Regulatory Standard:** DIR ≥ 0.80 (80% rule)
- If DIR < 0.80, the system may have disparate impact and violate fairness regulations
- ROCC's DIR: 0.9375 (well above threshold, demonstrating fairness)

**Example:**
```
Hypothetical Scenario:
- Merchant Group A (US-based): 10% flagged for violations
- Merchant Group B (International): 15% flagged for violations
- DIR = 0.15 / 0.10 = 1.5 (no disparate impact—actually favors Group B)

ROCC Scenario:
- Merchant Group A: 8% flagged
- Merchant Group B: 8.5% flagged
- DIR = 0.085 / 0.08 = 1.0625 (minimal difference, no disparate impact)
```

**Why It Matters:** Ensures the system doesn't accidentally (or intentionally) discriminate based on protected characteristics (nationality, ethnicity, gender, etc.).

**ROCC Context:** "The system achieved a Disparate Impact Ratio of 0.9375, clearing the regulatory floor of 0.80 and demonstrating fair treatment across merchant segments."

---

### Algorithmic Fairness
**Definition:** Ensuring that machine learning or decision-making systems treat different groups equitably and don't encode or amplify biases.

**Key Dimensions:**
1. **Disparate Impact:** System doesn't affect groups differently (measured by DIR)
2. **Calibration:** Confidence scores are accurate within groups (80% confidence = 80% actual positive rate)
3. **Transparency:** Decisions are explainable to users and regulators
4. **Appealability:** Users can challenge decisions and provide feedback

**ROCC Implementation:**
- Monitors DIR in real-time on dashboard
- Stratifies validation by business cohort (TT4B vs. E-Commerce) to ensure fairness across categories
- Maintains audit trails of all decisions for post-hoc fairness audits
- Plans to implement explainability and appeals process in Phase 2

---

### False Positive Rate (FPR)
**Definition:** The percentage of legitimate activities incorrectly flagged as violations.

**Formula:** FPR = (False Positives) / (All Legitimate Activities) × 100%

**ROCC Metrics:**
- Under Network Stress: 22.2%
- Post-Remediation: 0.0%
- Target Ceiling: < 5.0%

**Why It Matters:**
- **Merchant Impact:** Legitimate merchants incorrectly suspended lose revenue and may churn to competitors
- **Review Queue Impact:** False positives overwhelm human review teams, creating backlogs and delayed decisions
- **User Impact:** Customers may face service interruptions due to false account flags

**Trade-off:** Improving false positive rate sometimes means accepting higher false negative rates (missing some real violations). ROCC balanced this through sophisticated text-normalization, not just lowering the threshold.

---

### False Negative Rate (FNR)
**Definition:** The percentage of actual violations that the system fails to catch.

**Formula:** FNR = (False Negatives) / (All Violations) × 100%

**ROCC Metrics:**
- Pre-upgrade: 100% (caught 0 of 89 sophisticated violations)
- Post-upgrade: 0% (caught 89 of 89 violations)

**Why It Matters:**
- Undetected violations continue harming the platform
- Regulatory risk: if the system fails to catch violations within known categories, it's negligent
- Business risk: allows fraud/spam to proliferate, harming user experience and platform reputation

**Trade-off:** Achieving FNR of 0% is often impossible (requires flagging everything). ROCC achieved it on the test set through targeted improvements, not brute-force thresholds.

---

## Cohort Analysis Terms

### Cohort
**Definition:** A group of merchants/users with shared characteristics, analyzed separately to ensure fairness and tailor strategies.

**ROCC Cohorts:**
1. **TT4B Lead Generation Cohort:**
   - Business model: B2B lead generation services
   - Validation accuracy: 81.37% (maintained stable under stress through cohort-median interpolation)
   - Risk profile: Medium (coordinated lead quality issues)

2. **TikTok Shop E-Commerce Cohort:**
   - Business model: Direct-to-consumer product sales
   - Validation accuracy: 95.92% (high variability pre-remediation due to complex product listings)
   - Risk profile: High (product authenticity, counterfeiting)

**Why Analyze by Cohort?**
- Different business models have different risk profiles
- One-size-fits-all policies may be unfair (e.g., lead gen violates differently than e-commerce)
- Enables targeted improvements (e.g., optimize for each cohort's specific challenges)

**ROCC Practice:** Validated system performance and fairness separately for each cohort to ensure no group is disproportionately affected.

---

### Cohort-Median Interpolation
**Definition:** Predicting missing values in one merchant's data using the statistical median value from similar merchants in the same cohort.

**When It's Used:** When network packet loss corrupts incoming telemetry for a specific merchant.

**Example:**
```
Normal Data:
- Merchant X: FPR = 8% (calculated from today's activity)

With Data Loss:
- Merchant X: FPR = ??? (data corrupted during transmission)
- Solution: Use median FPR from other TT4B Lead Gen merchants = 7.5%
- Result: Use 7.5% as predicted value for Merchant X

Validation: Check that 7.5% falls within historical range (5%-12%) ✓
```

**Why Better Than Alternatives:**
- ❌ Dropping the record: Leaves decision-making blind
- ❌ Using zero: Biases the system (assumes no risk)
- ✅ Using cohort-median: Preserves context, fair approximation

**Caveat:** Interpolated data is flagged as "uncertain" in the system, and true data replaces it when available.

---

## Statistical & Mathematical Terms

### Null Hypothesis
**Definition:** The default assumption in statistical testing—that there is NO effect, difference, or relationship between variables.

**ROCC Context:** "Null hypothesis: The legacy and upgraded pipelines have equal detection rates."

**Statistical Testing Goal:** Gather evidence to either:
- Reject the null (conclude there IS a difference)
- Fail to reject the null (insufficient evidence of a difference)

**Result:** We rejected the null with p < 0.000001 (extremely strong evidence of a difference).

---

### Confidence Level / Significance Level
**Definition:** The probability that a statistical finding is real (not due to random chance).

**Common Thresholds:**
- 95% confidence (p < 0.05): Standard in many fields
- 99% confidence (p < 0.01): High stakes (medical, regulatory)
- 99.9999% confidence (p < 0.000001): Extremely strong evidence

**ROCC Achievement:** 99.9999% confidence that the upgraded system is better than legacy.

**Plain English:** "We're 99.9999% certain the improvement is real. The 0.0001% chance it's random is negligible."

---

### Type I Error (False Positive in Statistics)
**Definition:** Incorrectly rejecting the null hypothesis—concluding there IS a difference when there actually isn't.

**Probability:** Denoted as α (alpha), typically set to 0.05 (5% risk)

**In ROCC Context:** "If we claim the upgraded system is better but it's actually no better, that's a Type I error."

**Safeguard:** With p < 0.000001, Type I error risk is < 0.0001%. Extremely safe.

---

### Type II Error (False Negative in Statistics)
**Definition:** Failing to reject the null hypothesis when you should have—missing a real difference.

**Probability:** Denoted as β (beta), related to statistical power (1 - β)

**In ROCC Context:** "If the upgraded system IS better but our test fails to detect it, that's a Type II error."

**ROCC Safeguard:** The massive effect size (89-0 split) makes Type II error virtually impossible. We'd have to be very unlucky to miss a signal this strong.

---

### Standard Deviation
**Definition:** A measure of how spread out data points are from the average. High std = high variability, low std = consistent.

**ROCC Context:** "Cohort-median interpolation works best when cohort members have low standard deviation—i.e., merchants in the same category behave similarly."

**Example:**
```
TT4B Lead Gen FPR across merchants:
- Merchant A: 7%, Merchant B: 8%, Merchant C: 7.5%
- Mean: 7.5%, Std Dev: 0.5% (very consistent)
- Interpolating missing Merchant D's FPR with 7.5% is reliable ✓

Hypothetical high variability:
- Merchant A: 2%, Merchant B: 15%, Merchant C: 30%
- Mean: 15.7%, Std Dev: 12.5% (inconsistent)
- Interpolating would be unreliable ✗
```

---

## Testing & Validation Terms

### Regression Testing
**Definition:** Re-running tests on existing functionality to ensure new changes didn't break anything.

**ROCC Context:** "Before deploying the upgraded pipeline, we ran regression tests comparing outputs on historical data. The new system improved detection without sacrificing accuracy on previously-handled cases."

**Why Important:** Major changes can have unintended side effects. Regression testing catches them before production.

---

### A/B Testing / Controlled Experiment
**Definition:** Randomly splitting users/merchants into two groups (A and B) and comparing outcomes under different treatments to isolate the effect of the change.

**ROCC Application:**
- Group A: Merchants evaluated with upgraded pipeline
- Group B: Merchants evaluated with legacy pipeline (control)
- Comparison: Detect rates, false positive rates, fairness metrics
- Result: Upgraded system significantly outperformed

**Why Better Than Before/After:** Eliminates confounding factors (e.g., "detection rates improved because we had fewer spammers this month").

---

### Edge Cases
**Definition:** Unusual, extreme, or boundary conditions that most systems handle poorly.

**ROCC Edge Cases:**
1. **Unicode-Heavy Merchant Names:** Names with many non-Latin characters (Cyrillic, Arabic, CJK)
2. **Very New Merchants:** Lack of historical data for interpolation
3. **High-Frequency Merchants:** So much activity that statistical methods break down
4. **Contradictory Signals:** Multiple risk signals pointing in different directions

**ROCC Approach:** Identify edge cases early through red-team testing, then decide whether to:
- Handle gracefully (system degrades safely)
- Flag for human review (high-uncertainty cases)
- Accept limitation (edge case is too rare to warrant investment)

---

### Validation (of system/model)
**Definition:** Confirming that a system works correctly on data it hasn't seen before, to ensure it generalizes beyond training data.

**ROCC Validation Approach:**
1. **Hold-out test set:** Separate data not used during development
2. **Stratified sampling:** Ensure test set represents all cohorts
3. **Multiple time windows:** Test on different time periods to catch temporal drift
4. **Fairness audits:** Verify no disparate impact on any group

**Result:** System validated across TT4B and E-Commerce cohorts, both time periods, with Disparate Impact Ratio of 0.9375.

---

### Overfitting
**Definition:** A model learns the training data perfectly but fails to generalize to new data. Often indicated by perfect training accuracy but poor test accuracy.

**Risk in ROCC:** "If we optimize the text-normalization pipeline too aggressively on test data, it might fail on real production traffic."

**Safeguards:**
- Multiple independent metrics (FPR, fairness, significance) must all improve
- Cross-cohort validation (test on TT4B AND E-Commerce)
- Long-term monitoring post-deployment
- If performance drifts, trigger retraining

---

## Monitoring & Operations Terms

### Key Performance Indicator (KPI)
**Definition:** Quantifiable metric that tracks progress toward business goals.

**ROCC KPIs:**
- Detection rate (violations caught)
- False positive rate (< 5%)
- Fairness (Disparate Impact Ratio ≥ 0.80)
- System uptime (target: 99.9%)
- Alert latency (< 5 seconds)

**Distinction from KRI:** KPI measures success toward goals; KRI measures risk/health status.

---

### Service Level Objective (SLO)
**Definition:** A target level of service performance that the system commits to meeting.

**ROCC SLOs:**
- Alert delivery: < 5 seconds (need fast response to violations)
- Dashboard freshness: < 30 seconds (stakeholders need near-real-time data)
- Data ingestion latency: < 10 seconds (don't accumulate backlog)

**Measurement:** If SLO is breached, it triggers investigation and incident response.

---

### Real-Time vs. Batch Processing
**Definition:**
- **Real-Time:** Process data immediately as it arrives (millisecond latency)
- **Batch:** Collect data, then process in bulk at scheduled intervals (minute to hour latency)

**ROCC Choice:** Real-time for decision-making (alerts must fire fast), batch for long-running analysis (fairness audits, trend analysis).

**Trade-off:** Real-time is more complex and expensive but enables faster incident response.

---

### Observability
**Definition:** The ability to understand system behavior from external outputs (logs, metrics, traces), especially for complex distributed systems.

**ROCC Observability Tools:**
- **Metrics:** KRI/KPI dashboards tracking health in real time
- **Logging:** Detailed records of decisions made (audit trail)
- **Tracing:** Following a single decision through the entire pipeline to identify bottlenecks
- **Alerting:** Automated notification when metrics cross safety boundaries

**Why Important:** Without observability, you can't diagnose failures or identify performance regressions.

---

## Business & Strategy Terms

### Root Cause Analysis (RCA)
**Definition:** Systematic investigation to identify the underlying reason a problem occurred, not just the symptom.

**ROCC RCA Example:**
- **Symptom:** False positive rate jumped to 22.2%
- **Initial Cause:** Data loss due to network packet drops
- **Root Cause:** Network infrastructure hadn't been upgraded to handle peak traffic load
- **Solution:** (1) Implement data repair protocols, (2) upgrade network infrastructure

**Why Important:** Treating symptoms (tuning thresholds) might temporarily reduce FPR but won't prevent recurrence if the root cause remains.

---

### Trade-off Analysis
**Definition:** Acknowledging that optimizing one dimension often worsens another; choosing a balanced solution.

**ROCC Trade-offs:**
1. **Detection vs. False Positives:** Higher sensitivity catches more violations but creates more false positives. Solution: Use text-normalization to improve precision without sacrificing recall.

2. **Latency vs. Sophistication:** More sophisticated models = longer compute time = higher latency. Solution: Tiered processing (fast text-norm for alerts, deep analysis in background).

3. **Cost vs. Coverage:** More sophisticated system (ROCC) costs more than legacy. Solution: ROI analysis showing that prevented losses exceed infrastructure costs.

---

### Regulatory Compliance
**Definition:** Meeting legal and regulatory requirements, such as fairness standards and data privacy laws.

**ROCC Compliance:**
- **Fairness:** Disparate Impact Ratio ≥ 0.80 (regulatory requirement)
- **Transparency:** Audit trail of all decisions for regulatory investigation
- **Accountability:** Clear ownership and escalation paths
- **Appeals:** Users can challenge decisions

**Business Impact:** Demonstrating compliance protects against regulatory fines and reputational damage.

---

### Cross-Functional Alignment
**Definition:** Ensuring that different teams (engineering, product, legal, finance) share understanding and work toward common goals.

**ROCC Example:**
- **Engineering:** "We can build a sophisticated text-normalization system"
- **Product:** "Merchants need this to trust the platform"
- **Legal:** "Fairness monitoring protects against regulatory risk"
- **Finance:** "Prevented losses justify infrastructure investment"
- **Result:** All teams aligned on ROCC deployment

**Why Important:** Complex projects fail when teams optimize locally without understanding cross-functional impact.

---

## Quick Reference: Critical Numbers

| Metric | Value | Significance |
|--------|-------|--------------|
| Telemetry Health Index | 82.0% | Current data completeness (target: 100%) |
| Network Packet Loss (in test) | 15% | Stress test condition |
| FPR Under Stress | 22.2% | Performance before remediation |
| FPR Post-Remediation | 0.0% | Performance after fix |
| Target FPR Ceiling | < 5.0% | Operational requirement |
| Sophisticated Violations Caught | 89 | Legacy caught 0, upgraded caught 89 |
| Chi-Squared Statistic | 87.0112 | Extreme value indicating significance |
| p-value | < 0.000001 | 99.9999% confidence |
| Disparate Impact Ratio | 0.9375 | Fairness metric (floor: 0.80) |
| TT4B Cohort Accuracy | 81.37% | Business category 1 performance |
| E-Commerce Cohort Accuracy | 95.92% | Business category 2 performance |
| Testing Phases | 6 | Comprehensive evaluation periods |
| Merchant Tier 1 (Clean) | 72 accounts | Automated approval |
| Merchant Tier 2 (Warning) | 124 accounts | Shadow monitoring |
| Merchant Tier 3 (Critical) | 4 accounts | Immediate isolation |

---

## How to Use This Glossary During Interviews

1. **Before the interview:** Read all terms, focus on the ones marked with "ROCC Context"
2. **During the interview:** If asked about a technical term, explain it in simple terms FIRST, then add the technical depth
3. **In your answers:** Use terminology confidently but don't over-use jargon. Balance accessibility with precision.

**Example Good Answer:**
"We used a statistical test called McNemar to compare the two systems. Basically, we ran both pipelines on the same test data and found that the upgraded system caught all 89 violations while the legacy system caught zero. The test showed this difference is statistically significant—p < 0.000001 means we're 99.9999% confident this is real, not random luck."

**Example Bad Answer:**
"We ran a McNemar matrix with Chi-Squared analysis and got p < 0.000001." ← Too jargony, no business context

---

**Print or bookmark this glossary during interview prep!** 📚
