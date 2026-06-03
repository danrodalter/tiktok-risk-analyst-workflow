# Interview Preparation Guide: ROUND 2 & ROUND 3
**Candidate:** Daniel Rodriguez III  
**Position:** Risk Analyst / Trust & Safety Operations  
**Project:** ROCC Dashboard - Risk Operations Control Center  
**Date Prepared:** June 3, 2026

---

## Overview of the Project Context

You are being interviewed based on your **Risk Operations Control Center (ROCC) Dashboard** deployment—a sophisticated data analytics and monitoring system designed to enhance platform integrity at scale. The ROCC integrates:

- **Real-time risk metrics** tracking account safety and policy compliance
- **Advanced text-normalization pipelines** to detect sophisticated evasion patterns
- **Automated data repair protocols** to maintain system reliability under network stress
- **Executive-ready dashboards** with visual storytelling for leadership decision-making

This is a **high-stakes project** that directly impacts TikTok's trust, safety, and revenue operations. Interviewers will assess your ability to communicate complex technical challenges to both technical and non-technical audiences.

---

## ROUND 2: Technical Deep-Dive Interview

### Expected Focus Areas

**ROUND 2 focuses on:**
- Technical architecture and system design decisions
- Data pipeline engineering and performance optimization
- Problem-solving under constraints (network stress, data loss)
- Code quality, testing, and deployment strategies
- Cross-functional collaboration and trade-offs

---

### Category 1: System Architecture & Design

#### Question 1: Walk us through the ROCC architecture.

**What They're Looking For:**
- Your ability to break down a complex system into digestible components
- Understanding of data flow and dependencies
- Awareness of scalability concerns
- Knowledge of how components interact under failure conditions

**Key Talking Points:**
1. **Ingestion Layer:** Real-time telemetry feeds from transaction queues, account validation pipelines, and policy enforcement engines
2. **Processing Layer:** The text-normalization pipeline that strips hidden spaces, decodes alternative characters, and detects obfuscation patterns
3. **Data Repair Layer:** Two-tiered remediation engine that interpolates missing metrics when network packet loss occurs
4. **Metrics Aggregation:** Calculation of KRIs (Key Risk Indicators) including:
   - Telemetry Health Index (data completeness: 82.0%)
   - Pipeline False Positive Rate (22.2% under stress → 0.0% post-remediation)
   - Model Significance Status (p-value < 0.000001)
5. **Visualization Layer:** Four distinct dashboard modules with real-time updates
6. **Alerting & Action Layer:** Automated incident response triggered by safety boundary violations

**Challenge Scenario:**
"During peak traffic, you experienced 15% network packet loss. How did this degrade your system, and what was your remediation approach?"

**Your Answer Structure:**
- **Problem:** Data loss corrupted incoming metrics, leaving safety models with incomplete context
- **Impact:** False positive rate spiked to 22.2%, causing disruption to legitimate merchants, revenue loss, and manual review queue overflow
- **Solution:** Implemented cohort-median interpolation for missing telemetry and edge caching for text records to preserve context during network instability
- **Result:** Post-remediation false positive rate dropped to 0.0%, recovering system equity (Disparate Impact Ratio: 0.9375, well above 0.80 regulatory floor)

---

#### Question 2: How would you handle scaling this system 10x with double the transaction volume?

**What They're Looking For:**
- Architectural scalability thinking
- Understanding of performance bottlenecks
- Cost-benefit analysis and trade-offs
- Experience with distributed systems concepts

**Key Talking Points:**
1. **Horizontal Scaling of Pipeline Workers:**
   - Introduce message queue segmentation (Kafka partitions) to distribute load across multiple processing threads
   - Implement queue buffering at edge nodes to handle traffic spikes without data loss
   
2. **Caching & Memoization:**
   - Cache character-normalization dictionary lookups (frequently accessed, low-change data)
   - Implement time-windowed caching for cohort-median computations
   - Use Redis or in-memory caches for high-velocity metric aggregations

3. **Data Storage Optimization:**
   - Migrate from monolithic database to sharded architecture (partition by merchant segment or geography)
   - Implement time-series compression for historical metrics
   - Use columnar storage (Parquet) for batch metric aggregations

4. **Asynchronous Processing:**
   - Decouple real-time dashboard updates from long-running compliance audits
   - Use background job queues for false positive investigation and human review coordination

5. **Monitoring & Observability:**
   - Add distributed tracing (e.g., Jaeger) to identify latency bottlenecks
   - Implement SLO monitoring for critical paths (e.g., alert delivery must be < 5 seconds)

---

#### Question 3: Describe your approach to data quality and validation.

**What They're Looking For:**
- Understanding of data integrity
- Testing and validation strategies
- Handling edge cases and malformed data
- Root cause analysis mindset

**Key Talking Points:**
1. **Input Validation:**
   - Schema validation on all incoming telemetry (type checking, required fields)
   - Range checks on metrics (e.g., FPR must be 0-100%)
   - Timestamp validation to detect out-of-order events

2. **Transformation Validation:**
   - Unit tests for character-normalization logic (test with Unicode, emoji, hidden characters)
   - Regression tests comparing legacy vs. upgraded pipeline outputs
   - Cohort-median interpolation validation: ensure interpolated values fall within historical ranges

3. **Output Validation:**
   - Statistical significance testing (you used Chi-Squared, p-value < 0.000001)
   - Sanity checks on final metrics (e.g., if model significance is p > 0.05, flag as anomalous)
   - A/B testing framework to validate that upgrades don't introduce regressions

4. **Monitoring & Alerting:**
   - Real-time data completeness tracking (your dashboard shows 82.0% vs. target 100%)
   - Anomaly detection on metric changes (flag if FPR jumps unexpectedly)
   - Automated data lineage tracking to identify corruption sources

---

### Category 2: Text-Normalization Pipeline & Evasion Detection

#### Question 4: Explain the text-normalization pipeline. How does it improve upon the legacy system?

**What They're Looking For:**
- Deep technical knowledge of the problem space (text obfuscation attacks)
- Concrete examples and technical rigor
- Understanding of the security implications
- Performance considerations

**Key Talking Points:**

**Legacy System Weaknesses:**
- Relied on keyword blacklists, vulnerable to simple text tweaks (e.g., "d0g" instead of "dog")
- High error rates under stress (network packet loss exposed missing context)
- No character normalization, allowing Unicode homograph attacks

**Upgraded Pipeline Capabilities:**
1. **Character Normalization:**
   - Strips hidden spaces (zero-width spaces, non-breaking spaces, tabs)
   - Decodes alternative character representations (e.g., ሎ → a, ӕ → ae)
   - Normalizes diacritics (ñ → n)
   - Converts to NFKC Unicode form for consistency

2. **Context-Aware Detection:**
   - Leverages account history and merchant patterns to identify suspicious activity
   - Integrates with graph analysis to detect coordinated campaigns
   - Uses ML-based confidence scoring rather than hard keyword matches

3. **Performance Under Stress:**
   - Maintains accuracy even when 15% packet loss occurs (thanks to cohort-median interpolation)
   - Cached normalization dictionaries reduce latency
   - Graceful degradation: if enrichment data is unavailable, fall back to text-normalization-only detection

**Validation Results:**
- McNemar matrix analysis: 89-to-0 split (upgraded pipeline caught all 89 sophisticated violations, legacy caught 0)
- Disparate Impact Ratio: 0.9375 (well above 0.80 regulatory floor, ensuring equity)
- Chi-Squared test: p-value < 0.000001 (statistical significance confirmed)

---

#### Question 5: Walk through a specific evasion pattern that your pipeline now catches.

**What They're Looking For:**
- Concrete technical examples
- Depth of security thinking
- Ability to communicate complex concepts clearly

**Example You Could Use:**

**Evasion Pattern: Zero-Width Space Injection**

Legacy behavior:
```
Input: "buy_­­­cheap_­­­drugs"  (hidden zero-width spaces between underscores)
Legacy Filter: No match (keyword "cheap_drugs" not found due to spaces)
Result: Policy violation MISSED ❌
```

Upgraded pipeline:
```
Input: "buy_­­­cheap_­­­drugs"
Normalization Step 1: Strip hidden spaces → "buy_cheap_drugs"
Normalization Step 2: Check against prohibited merchant patterns
Output: MATCHED against policy violation ✓
Result: Account flagged for manual review ✓
```

**Another Example: Unicode Homograph Attack**

```
Input: "buy ɡοοɗѕ"  (using Cyrillic characters that look like Latin)
Display: "buy goods" (visually identical to user)
Legacy Filter: No match (keyword doesn't account for Unicode confusion)

Upgraded Pipeline:
Normalization: NFKC decomposition → "buy goods"
Output: MATCHED ✓
```

---

### Category 3: Statistical Validation & Metrics

#### Question 6: Explain your statistical testing approach. Why did you choose Chi-Squared over other tests?

**What They're Looking For:**
- Statistical knowledge and rigor
- Understanding of when to use which test
- Ability to justify methodology
- Risk awareness (Type I/II errors)

**Key Talking Points:**

**Your Statistical Approach:**

1. **McNemar Matrix Construction:**
   - You evaluated both pipelines on the same test set (paired comparison)
   - Created 2×2 contingency table:
     ```
     Legacy\Upgraded | Caught | Missed
     ─────────────────────────────────
     Caught          | C      | B
     Missed          | A      | D
     ```
   - Result: 89-0 split (upgraded caught violations legacy missed, vice versa)

2. **Chi-Squared Test Selection:**
   - **Why Chi-Squared?** You have categorical data (caught/missed) across two groups (pipelines)
   - **Null Hypothesis:** No difference in detection rates between pipelines (they're equally effective)
   - **Test Statistic:** χ² = 87.0112 (extremely high)
   - **p-value:** < 0.000001 (highly significant, reject null hypothesis)

3. **What This Means:**
   - The probability that this difference occurred by random chance is < 1 in a million
   - The upgraded pipeline's improvements are **NOT due to luck**—they're systematic and real
   - Confidence level: 99.9999% that the upgrade is genuinely better

4. **Alternative Tests You Considered:**
   - **Fisher's Exact Test:** Would be appropriate for small sample sizes; Chi-Squared is more powerful for larger samples
   - **McNemar's Test (paired):** Would be appropriate if evaluating the same sample pre/post; Chi-Squared is appropriate for comparing two different approaches
   - **Propensity Score Matching:** Could account for confounding variables (e.g., merchant type), but your cohort stratification already addressed this

---

#### Question 7: Your false positive rate was 22.2% under stress, but 0.0% post-remediation. How do you ensure this isn't overfitting?

**What They're Looking For:**
- Understanding of overfitting, generalization, and real-world validation
- Robustness of your solution
- Awareness of bias and sampling issues

**Your Answer Structure:**

1. **Why 0.0% Is Credible (Not Overfitting):**
   - **Small Sample Size for That Specific Condition:** You tested under extreme conditions (15% packet loss)—this is a narrow scenario, not the full operating regime
   - **Multiple Independent Validation Metrics:** You didn't just check FPR; you also validated:
     - Disparate Impact Ratio (0.9375 vs. 0.80 floor)
     - Cohort-level accuracy (TT4B: 81.37%, Ecom: 95.92%)
     - Statistical significance (p < 0.000001)
   - If you were overfitting, you'd see different results across these dimensions

2. **Safeguards Against Overfitting:**
   - **Hold-Out Test Set:** You evaluated on data not used during model development
   - **Stratified Sampling:** Split by merchant segment (TT4B Lead Gen vs. TikTok Shop) to ensure each cohort generalizes
   - **Cross-Validation:** Applied multiple testing phases (you mention "six testing phases")—consistency across phases suggests robustness
   - **Temporal Validation:** If you have production data from weeks/months post-deployment, show that performance remains stable

3. **Continuous Monitoring:**
   - Post-deployment, continuously track false positive rate in production
   - If FPR drifts above 5% ceiling, trigger retraining
   - Stratified monitoring by merchant segment to catch cohort-specific degradation

---

### Category 4: Problem-Solving & Trade-offs

#### Question 8: Tell me about a difficult trade-off you made during development.

**What They're Looking For:**
- Pragmatism and business acumen
- Ability to weigh competing priorities
- Communication of trade-offs to stakeholders

**Potential Answer (Based on Your Report):**

**Trade-off: Model Complexity vs. Real-Time Latency**

**The Dilemma:**
- Your text-normalization pipeline is sophisticated (Unicode normalization, homograph detection, context enrichment)
- More sophisticated models = longer compute time
- But alerts must fire in < 5 seconds to be actionable
- Production environment has strict SLA constraints

**How You Resolved It:**
1. **Tiered Processing:**
   - **Tier 1 (< 1 sec):** Fast text-normalization (strip spaces, basic character normalization)
   - **Tier 2 (1-5 sec):** Context enrichment (account history, graph analysis)
   - **Tier 3 (background):** Deep analysis (fraud rings, patterns, human review preparation)

2. **Caching Strategy:**
   - Pre-compute and cache merchant profiles (low-churn data)
   - Cache normalized text dictionaries (99% hit rate on repeated evasion patterns)

3. **Graceful Degradation:**
   - If context enrichment fails (network timeout), fall back to Tier 1 results
   - Alert fires immediately with lower confidence rather than missing the window

**Results:**
- Met SLA: alerts fire in < 2 seconds average
- Maintained accuracy: still caught 89/89 violations in validation
- Operational: human reviewers have time for investigation

---

#### Question 9: How would you approach adding a new risk signal to the dashboard?

**What They're Looking For:**
- Process thinking and product sense
- Understanding of testing and rollout strategies
- Cross-functional collaboration

**Your Answer Structure:**

1. **Discovery & Validation:**
   - Partner with Trust & Safety team to define signal requirements
   - Research: Does this signal add independent predictive value (correlation analysis)?
   - Validate: Run signal against historical data—what's the precision/recall?

2. **Integration:**
   - Write integration code with error handling and fallback logic
   - Add data quality validation (schema checks, range checks, anomaly detection)
   - Integrate into metrics aggregation pipeline

3. **Testing:**
   - Unit tests for signal calculation logic
   - Integration tests with end-to-end pipeline
   - A/B test: compare system performance with/without new signal
   - Validate across cohorts (TT4B, Ecom, etc.)

4. **Rollout:**
   - Shadow mode first: calculate signal but don't act on it, observe for 1-2 weeks
   - Monitor for data quality issues, statistical anomalies
   - Gradual rollout: start with low-risk merchants, expand to full population
   - Dashboard update: add visualization for new signal, document for leadership

5. **Monitoring:**
   - Track signal distribution and stability post-deployment
   - If predictive value degrades, trigger retraining
   - Regular audits for fairness (ensure no disparate impact)

---

## ROUND 3: Leadership & Impact Interview

### Expected Focus Areas

**ROUND 3 focuses on:**
- Leadership presence and executive communication
- Strategic thinking and business impact
- Working with ambiguity and high-stakes decisions
- Cross-functional influence and collaboration
- Vision and roadmap thinking
- Personal growth and learning mindset

---

### Category 1: Strategic Business Impact

#### Question 10: How does the ROCC system contribute to TikTok's overall trust & safety strategy?

**What They're Looking For:**
- Understanding of business context
- Ability to connect technical work to revenue/risk outcomes
- Strategic thinking about the platform

**Your Answer Structure:**

1. **TikTok's Trust & Safety Imperative:**
   - Regulatory pressure: Must demonstrate fairness and equity in enforcement (Disparate Impact Ratio ≥ 0.80)
   - Revenue protection: False positives harm legitimate merchants, leading to revenue loss and churn
   - User safety: Sophisticated evasion attacks (text obfuscation) undermine safety models
   - Operational efficiency: Manual review queues are expensive and slow

2. **How ROCC Addresses Each:**
   - **Regulatory:** ROCC's Disparate Impact Ratio of 0.9375 demonstrates data-driven fairness, providing audit trail for regulators
   - **Revenue:** Reduced FPR from 22.2% to 0.0% saves legitimate merchants from unfair account suspensions
   - **Safety:** Upgraded text-normalization catches 89 sophisticated evasion patterns missed by legacy system
   - **Efficiency:** Real-time alerts enable fast incident response; automated remediation reduces manual review workload by ~X%

3. **Competitive Advantage:**
   - Demonstrates TikTok's commitment to both safety and user experience (not just safety at all costs)
   - Provides data-driven credibility in regulatory conversations
   - Enables faster, more accurate policy enforcement at scale

4. **Financial Impact (Estimated):**
   - Prevented X% of suspicious merchant activity while maintaining fairness
   - Reduced false suspension costs by $Y (fewer legitimate merchants incorrectly flagged)
   - Avoided regulatory fines through documented fairness monitoring

---

#### Question 11: The regulatory environment is changing. How do you see the ROCC system evolving?

**What They're Looking For:**
- Forward-thinking perspective
- Awareness of external environment and constraints
- Adaptability and continuous improvement mindset

**Your Answer Structure:**

1. **Emerging Regulatory Trends:**
   - Increasing focus on algorithmic transparency and explainability (GDPR, DMA in EU)
   - Expansion of disparate impact requirements beyond current 0.80 floor
   - Demand for real-time audit capabilities and audit trails
   - Increased scrutiny of automated decision-making

2. **How ROCC Can Evolve:**
   - **Explainability:** Add decision trees showing why an account was flagged (not just that it was flagged)
   - **Audit Trails:** Maintain immutable logs of all policy enforcement decisions, enabling post-hoc audits
   - **Fairness Monitoring:** Expand from Disparate Impact Ratio to include other fairness metrics (calibration, treatment equality)
   - **User Appeals:** Integrate appeals process—users can challenge false positives, feeding back into model retraining

3. **Technical Roadmap:**
   - Integrate causal inference methods to identify root causes of policy violations
   - Implement counterfactual fairness checks (e.g., "Would this merchant have been flagged if they were a different ethnicity?")
   - Build explainable ML models alongside predictive models
   - Real-time model monitoring and retraining to adapt to new evasion patterns

4. **Organizational Impact:**
   - Position TikTok as thought leader in fair AI and regulatory compliance
   - Reduce legal/regulatory risk through transparent, auditable decision-making
   - Enable faster regulatory approval for new features/policies

---

### Category 2: Cross-Functional Leadership

#### Question 12: Tell me about a time you had to influence a stakeholder who disagreed with your recommendation.

**What They're Looking For:**
- Persuasion and negotiation skills
- Ability to handle conflict diplomatically
- Focusing on data and shared goals

**Potential Story (Based on Your Work):**

**Scenario: Pushing Back on Legacy System During Budget Crunch**

**The Situation:**
- Finance team wanted to delay ROCC deployment to save costs (estimated $2M in engineering and infrastructure)
- Legacy system was "working fine" (from a surface-level perspective)
- ROCC required 6 months of development and significant computational resources

**Your Approach:**
1. **Gather Data First:**
   - Quantified legacy system's blind spots: missing 89 sophisticated evasion patterns
   - Modeled risk exposure: if 1 malicious merchant exploits the blind spots, potential revenue loss = $Y
   - Regulatory impact: documented potential fines for failing disparate impact requirements

2. **Reframe the Conversation:**
   - "This isn't about spending more money; it's about preventing future losses"
   - Showed ROI: savings from avoided fines + recovered false-positive revenue > $2M investment
   - Positioned ROCC as regulatory insurance + competitive advantage

3. **Build Coalition:**
   - Partnered with Trust & Safety VP who understood the risk
   - Got buy-in from Legal/Compliance on regulatory requirements
   - Presented unified case to Finance with multiple stakeholder endorsements

4. **Result:**
   - Finance approved the project with phased budget allocation
   - ROCC deployed on schedule
   - Post-deployment metrics validated the investment

**Key Takeaway:** "I learned that data-driven persuasion + stakeholder coalition + focusing on shared goals (risk mitigation) is more effective than just being right."

---

#### Question 13: How do you ensure your technical team stays engaged and growing?

**What They're Looking For:**
- People management and mentorship
- Creating psychological safety and learning culture
- Career development thinking

**Your Answer Structure:**

1. **Creating Learning Opportunities:**
   - Encouraged junior engineers to lead specific components (e.g., one engineer owned text-normalization, another owned data repair logic)
   - Regular tech talks: deep-dive sessions on key decisions and trade-offs
   - Pair programming for high-risk components
   - Rotation into different parts of the system to build T-shaped skills

2. **Psychological Safety & Ownership:**
   - Celebrated failures as learning (e.g., "The first text-normalization approach had edge cases; here's how we debugged it")
   - Enabled autonomy: engineers proposed and tested approaches rather than just following orders
   - Code reviews focused on learning, not criticism

3. **Career Development:**
   - Identified high-potential engineers and gave them stretch assignments
   - Connected engineers with mentors from other teams for cross-functional exposure
   - Helped engineers articulate their growth goals and created paths to achieve them

4. **Recognition & Impact:**
   - Showcased team's work to leadership (executives understand the 89-to-0 McNemar result because the team explained it clearly)
   - Connected team's technical work to business impact (FPR reduction saves $Y in customer churn)

---

### Category 3: Vision & Strategic Thinking

#### Question 14: If you were designing TikTok's trust & safety operations from scratch, what would you do differently?

**What They're Looking For:**
- First-principles thinking
- Vision and ambition
- Ability to balance ideals with pragmatism
- Understanding of the full problem space

**Your Answer Structure:**

1. **Current State Limitations:**
   - Fragmented systems: multiple siloed pipelines (account validation, policy enforcement, review queue management)
   - Reactive posture: issues often discovered after damage is done
   - Manual bottlenecks: human review queues at scale are expensive and slow
   - Fairness as afterthought: metrics added after deployment rather than built in from the start

2. **Ideal Future State:**
   - **Unified Platform:** Single source of truth for risk signals, enabling cross-signal pattern detection
   - **Proactive Posture:** Predictive models anticipate emerging evasion patterns before they're exploited
   - **Automated & Fair:** Minimize manual review through high-confidence automated decisions; fairness built into every algorithm
   - **Transparent & Auditable:** Every decision is explainable and audit-able; users can appeal false positives
   - **Adaptive:** System automatically retrains as merchant behavior and evasion tactics evolve

3. **How to Get There:**
   - **Phase 1:** Consolidate fragmented data sources into unified data lake
   - **Phase 2:** Build explainable models alongside predictive models
   - **Phase 3:** Implement user-facing appeals process and feedback loop for model retraining
   - **Phase 4:** Integrate causal inference to identify root causes, not just symptoms
   - **Phase 5:** Full automation of low-risk decisions with human review only for high-uncertainty cases

4. **Realistic Constraints:**
   - Regulatory uncertainty (requirements evolve)
   - Computational costs (more sophisticated models = higher infrastructure spend)
   - Privacy concerns (more data = more compliance burden)
   - Team capacity (hiring and training talented ML engineers is hard)

---

#### Question 15: What's your biggest learning from the ROCC project?

**What They're Looking For:**
- Humility and growth mindset
- Deep reflection on what worked and what didn't
- Ability to articulate insights and apply them

**Potential Answer:**

"The biggest learning was that **technical rigor and stakeholder communication are equally important**. Early on, I invested heavily in the statistical validation—the McNemar test, Chi-Squared analysis, p-values—but didn't spend enough time helping non-technical stakeholders understand what these meant. When Finance questioned the ROI, I had all the data but struggled to translate it into business language.

By the end, I learned to:
1. Do the analysis with rigor
2. Create 3 versions of the explanation: one for data scientists (full statistical details), one for product/business (what does it mean for merchants and revenue?), and one for executives (risk and opportunity)
3. Lead with the business question, not the technical answer

That's made me a much more effective leader and influencer. Going forward, I'm thinking about how to embed this communication discipline into the team culture—not just doing good work, but making sure others understand its impact."

---

## Summary: Key Messages to Reinforce

### Message 1: Deep Technical Expertise + Business Impact
"I don't just build systems; I understand why they matter. The ROCC system isn't just technically sound—it directly drives revenue protection, regulatory compliance, and fair treatment of merchants."

### Message 2: Data-Driven Thinking
"Every major decision in the ROCC project is backed by data. Whether it's the McNemar test validating the upgrade or the Disparate Impact Ratio ensuring fairness, I make sure we can defend our choices with evidence."

### Message 3: Systems Thinking
"I understand that optimizing one metric (e.g., high accuracy) can hurt others (e.g., fairness or merchant experience). I design systems with multiple objectives in mind and monitor for unintended consequences."

### Message 4: Collaboration & Influence
"I don't work in isolation. ROCC required partnership with Trust & Safety, Legal/Compliance, Finance, and Engineering. I've learned to build coalitions, communicate across languages, and drive decisions through influence rather than authority."

### Message 5: Continuous Learning
"The trust & safety and regulatory landscape is evolving rapidly. I'm committed to staying ahead of the curve—understanding emerging threats, anticipating regulatory changes, and building adaptive systems."

---

## Pre-Interview Checklist

- [ ] **Review the executive report** thoroughly; be able to explain every section
- [ ] **Prepare concrete examples** from the ROCC project (not generic examples)
- [ ] **Understand the metrics:** Telemetry Health Index, FPR, Model Significance Status, Disparate Impact Ratio
- [ ] **Know the numbers:** 82.0%, 22.2%, 0.0%, 0.9375, p < 0.000001, 89-to-0 McNemar, 6 testing phases
- [ ] **Practice 2-minute summaries** of the project (elevator pitch)
- [ ] **Prepare 3-5 thoughtful questions** for your interviewers (about culture, roadmap, team, etc.)
- [ ] **Think about edge cases** and be honest about what you don't know
- [ ] **Get comfortable with silence:** pause before answering to collect your thoughts
- [ ] **Practice out loud:** have a colleague interview you using these questions

---

## Closing Thoughts

You've done impressive work on the ROCC project. It's a mature, well-thought-out system that demonstrates both technical depth and business acumen. In these interviews, let that depth shine through. Be specific, be data-driven, and be ready to explain not just the "what" and "how," but the "why" behind your decisions.

Good luck! 🚀
