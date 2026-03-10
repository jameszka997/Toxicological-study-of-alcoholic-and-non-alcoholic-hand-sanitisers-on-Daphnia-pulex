# 🔬 Deep Dive: Analysis, Problems & Solutions
## Toxicological Effects of Hand Sanitisers on *Daphnia pulex*

> A detailed walkthrough of the analytical methodology, experimental challenges encountered, and the solutions developed throughout this BSc thesis project.

---

## Table of Contents

- [Study Design Overview](#study-design-overview)
- [Experiment 1: Acute Toxicity Tests](#experiment-1-acute-toxicity-tests-48h)
- [Experiment 2: Chronic Toxicity Tests](#experiment-2-chronic-toxicity-tests-14-day)
- [Experiment 3: Heart Rate Analysis](#experiment-3-heart-rate-analysis)
- [Statistical Analysis Deep Dive](#statistical-analysis-deep-dive)
- [Problems Encountered & How They Were Solved](#problems-encountered--how-they-were-solved)
- [Key Findings Summary](#key-findings-summary)
- [Lessons for Future Studies](#lessons-for-future-studies)

---

## Study Design Overview

Two commercially available hand sanitisers were tested:

| Property | ABHS (Alcohol-Based) | NABHS (Non-Alcohol-Based) |
|----------|----------------------|---------------------------|
| **Product** | Oceanfree+ Alcoholic Hand Sanitiser | Dettol Anti-Bacterial Surface Cleanser |
| **Active Ingredient** | Ethanol (80 v/v%) | Benzalkonium Chloride / BAC (0.96 g/L) |
| **Concentration Units** | v/v% | μg/L |
| **Key Chemical Property** | Highly volatile | Persistent, accumulates in sediment |

*Daphnia pulex* was used as the test organism — a standard freshwater ecotoxicology model species chosen for its sensitivity to chemical changes, transparent body (allowing physiological observation), and ecological importance in freshwater food webs.

Test concentrations for both sanitisers across all three experiments, diluted using a dilution factor of 2:

| Group | ABHS Acute | NABHS Acute | ABHS Chronic | NABHS Chronic | ABHS Heart Rate | NABHS Heart Rate |
|-------|-----------|------------|-------------|--------------|----------------|-----------------|
| C1 | 0.08% | 384 μg/L | 0.0016% | 9.6 μg/L | 0.04% | 480 μg/L |
| C2 | 0.04% | 192 μg/L | 0.0008% | 4.8 μg/L | 0.02% | 240 μg/L |
| C3 | 0.02% | 96 μg/L | 0.0004% | 2.4 μg/L | 0.01% | 120 μg/L |
| C4 | 0.01% | 48 μg/L | 0.0002% | 1.2 μg/L | 0.005% | 60 μg/L |
| C5 | 0.005% | 24 μg/L | 0.0001% | 0.6 μg/L | 0.0025% | 30 μg/L |

---

## Experiment 1: Acute Toxicity Tests (48h)

### Setup
- Followed OECD (2004) Guideline 202 for *Daphnia* acute immobilisation tests
- 5 concentration groups + 1 control per sanitiser
- 5 replicates per concentration group, 5 *D. pulex* specimens per replicate (10 mL test solution in glass vials)
- Mortality assessed at 24h and 48h — defined as no movement for 15 seconds after gentle agitation
- Temperature and pH recorded at start and end of test

### Analysis Pipeline

```
Raw mortality counts (per replicate)
        ↓
Proportional mortality calculation per concentration group
        ↓
Probit regression (ecotox::LC_probit)
        ↓
LC50 + 95% Confidence Intervals
        ↓
MATC = LC50 × 0.1  (Sprague equation)
        ↓
Ratio Test (ecotox) → Compare ABHS vs NABHS LC50 values
```

### Results

**NABHS (BAC)** — Clean, interpretable results:
- Mortality increased consistently with concentration (8% → 12% → 4% → 52% → 96%)
- LC₅₀ = **164 μg/L** (95% CI: 114–261 μg/L)
- MATC = **16.4 μg/L**
- Concentration-response curve was sigmoid and well-fitted

**ABHS (Ethanol)** — Problematic results:
- Very low and irregular mortality (0%, 4%, 0%, 0%, 20% across increasing concentrations)
- LC₅₀ calculated at **0.406 v/v%**, but 95% CI was anomalous (0.108 to 2.39e-8 v/v%)
- Concentration-response curve was non-sigmoid — a shallow, irregular curve rather than the expected S-shape
- MATC = **0.0406 v/v%**

Ratio Test comparing both sanitisers found a statistically significant difference (se=1.18, t=2.21, p=0.027) — though reliability of this is limited due to the abnormal ABHS curve.

> **The non-sigmoid ABHS curve was the first major signal that something was wrong with the experimental methodology for the alcohol-based tests.**

---

## Experiment 2: Chronic Toxicity Tests (14-day)

### Setup
- Followed US-EPA (1996) guidelines
- 5 concentration groups + 1 control per sanitiser
- 10 replicates per group, 1 specimen per replicate (10 mL test solution)
- Static renewal test — test solutions replaced 3× per week
- Mortality, offspring count, and any physical changes recorded at each renewal
- Chlorella sp. extract (1 mL/L) added to EPA medium as food source

### Analysis Pipeline

```
Survival/mortality data per specimen per day
        ↓
Probit regression → LC10 & LC50 (ecotox::LC_probit)
        ↓
MATC = LC50 × 0.1
        ↓
Ratio Test → Compare ABHS vs NABHS LC50
        ↓
Kaplan–Meier survival curves (survival package)
→ Visualise survival probability over 14 days per concentration group
```

### Results

**LC values from chronic tests:**

| LC Value | NABHS | ABHS |
|----------|-------|------|
| LC₅₀ | 7.61 μg/L (95% CI: —) | 0.0454 v/v% (95% CI: —) |
| LC₁₀ | 1.25e7 μg/L (95% CI: 6.52–0.624 μg/L) | 4.32e-16 v/v% (95% CI: 0.00917–0.0000333 v/v%) |
| MATC | 0.761 μg/L | 0.00454 v/v% |

Both LC₅₀ values lacked calculable 95% confidence intervals — a clear indicator of unreliable data distributions.

**ABHS Survival Curves** — Showed near-zero dose-response: the highest (C1) and lowest (C5) concentration groups produced almost identical survival curves, with a flat concentration-response. This indicated the tested concentration range was too low to distinguish between groups.

**NABHS Survival Curves** — Highest survival rates were unexpectedly found in mid-range concentrations (C3 and C4), not the lowest, indicating a highly irregular distribution.

**A notable drop in survival was observed around Day 5 across both sanitisers** — attributed to age variability in the specimens (see Problems section).

Ratio Test found **no significant difference** between chronic LC₅₀ values (se=6.41, t=0.147, p=0.883).

---

## Experiment 3: Heart Rate Analysis

### Setup
- Based on Qin Xiang (2015) methodology
- 5 concentration groups + 1 control per sanitiser
- 10 replicates per group, 1 specimen per replicate
- Baseline heart rate of *D. pulex* established: **343 ± 15 BPM** (n=100)

### Procedure
1. Specimen transferred to cavity slide; left 2 minutes to normalise heart rate
2. Excess liquid removed with tissue to reduce specimen movement
3. **15-second pre-exposure video** recorded through 10X microscope eyepiece (60 fps, Huawei P20)
4. Drop of test substance applied; specimen left 2 minutes to react
5. **15-second post-exposure video** recorded
6. Footage compiled and slowed down using DaVinci Resolve 17
7. Heart beats manually counted from slowed footage → BPM calculated

### Analysis Pipeline

```
Raw BPM (before & after) per specimen
        ↓
Mean BPM per concentration group (before & after)
        ↓
Heart Rate Difference (%) = ((after - before) / before) × 100
        ↓
Paired t-test → Significant change before vs after for each sanitiser?
        ↓
Kruskal-Wallis test (ABHS) → Significant difference between concentration groups?
One-way ANOVA (NABHS) → Significant difference between concentration groups?
        ↓
ANOVA → Significant difference in % heart rate change between ABHS and NABHS?
```

### Why Different Tests for ABHS vs NABHS?
A **Kruskal-Wallis** (non-parametric) test was used for ABHS concentration group comparisons, and a **parametric ANOVA** for NABHS. This reflects the assumption that NABHS data better met the assumptions of normality required for parametric testing, whereas the ABHS data — already shown to be irregular — warranted a non-parametric approach.

### Results

| Test | Result |
|------|--------|
| Paired t-test (ABHS before vs after) | Not significant (t=−0.588, df=89.8, p=0.558) |
| Paired t-test (NABHS before vs after) | Not significant (t=−1.321, df=96.17, p=0.190) |
| Kruskal-Wallis (ABHS concentration groups) | Not significant (p=0.160) |
| One-way ANOVA (NABHS concentration groups) | Not significant (F=1.632, df=4, p=0.183) |
| ANOVA (ABHS vs NABHS % difference) | Not significant (p=0.689) |

Despite no statistically significant results, **biologically relevant trends** were observed:
- Both sanitisers showed a tendency toward **decreasing heart rates** at higher concentrations
- Variance in heart rate response **increased with concentration** in both groups
- No concentration caused more than a **20% mean heart rate reduction**

---

## Statistical Analysis Deep Dive

### Tools & Packages

| Task | Package/Function | Notes |
|------|-----------------|-------|
| LC₁₀ / LC₅₀ calculation | `ecotox::LC_probit` | Probit regression on mortality data |
| Comparing LC₅₀ values | `ecotox` Ratio Test | Tests whether two LC₅₀ values differ significantly |
| Survival curves | `survival` (Kaplan-Meier) | Plots survival probability over 14 days |
| Paired before/after comparisons | `stats::t.test` (paired) | Heart rate pre/post exposure |
| Between-group comparisons (normal) | `stats::aov` (ANOVA) | NABHS concentration groups |
| Between-group comparisons (non-normal) | `stats::kruskal.test` | ABHS concentration groups |
| MATC calculation | Manual (Sprague equation) | MATC = LC₅₀ × 0.1 |

### Why LC₁₀ Instead of NOEC/LOEC?
The study chose to calculate **LC₁₀** values rather than the more traditional NOEC (No Observed Effect Concentration) and LOEC (Lowest Observed Effect Concentration). This decision was based on Warne & Dam (2008), who argued that NOEC/LOEC values make inefficient use of observed data and provide no information about the shape of the concentration-response relationship. LC₁₀ values derived from probit modelling utilise the full dataset and are anchored to the concentration-response curve, providing a more meaningful and reproducible toxicological benchmark.

### Diagnosing Bad Data: The Non-Sigmoid Problem

A well-functioning acute or chronic toxicity test should produce a **sigmoid (S-shaped) concentration-response curve** — low mortality at low concentrations gradually rising to near-total mortality at high concentrations. This is the expected shape from probit regression.

When this shape does not emerge, it signals one or more of:
- Concentration range too narrow or misaligned with actual toxicity
- Unstable exposure concentrations (e.g., evaporation)
- High variability in organism susceptibility (e.g., age differences)
- An artefact of the test substance or methodology

In this study, the ABHS data produced a **flat, non-sigmoid curve** — the clearest statistical signal that the ethanol concentrations were not behaving as expected. This observation directly prompted the investigation that identified alcohol evaporation as the root cause.

---

## Problems Encountered & How They Were Solved

### Problem 1: Alcohol Evaporation Destabilising ABHS Concentrations

**What happened:**
The ABHS acute toxicity test was repeated **four times** (one range-finder + three full repeats). Across tests, the mortality response at identical concentrations varied wildly and non-monotonically — sometimes higher concentrations caused *less* mortality than lower ones. This made LC₅₀ calculation unreliable and the concentration-response curve non-sigmoid.

**Root Cause:**
Ethanol is highly volatile. O'Hare et al. (1993) established that ethanol in binary water-ethanol mixtures at concentrations below 4 v/v% can lose approximately **~1 v/v% of ethanol in just a few hours** of air exposure. Critically, the evaporation rate increases as concentration decreases — meaning the lowest concentration groups were losing proportionally more ethanol than higher ones, inverting the expected dose-response relationship.

Before this was identified, test beakers and replicate vials were left **open and uncovered for 1–3 hours** during set-up, causing substantial and inconsistent ethanol loss. Because this issue was not documented in standard OECD toxicity guidelines (which were designed for stable, non-volatile test substances), it was not anticipated during the study design phase.

**How it was identified:**
The non-sigmoid concentration-response curves were the primary signal. After multiple repeated tests with shifting and inconsistent results — and after extensive discussion with laboratory technical staff and the study supervisor — ethanol evaporation was identified as the main driver of variability.

**Solution implemented:**
- Test solution beakers and replicate vials were **covered with plastic boards or inverted wider beakers** to minimise air exposure at all times
- Vials were **re-covered immediately** after each check-up inspection
- This methodological change produced improved results: at 843 mg/L, mortality improved from 4% (uncovered) to **20%** (covered), demonstrating the effectiveness of the intervention

**Alternative solution considered but not used:**
Szabo & Mandrekar (2008) suggest placing an open beaker containing a higher-concentration alcohol solution (at least 2× the highest tested concentration) at the bottom of the incubator to saturate the chamber atmosphere with ethanol vapour, maintaining stable concentrations in the test solutions. This could not be implemented because the same incubator housed both the NABHS replicates and the *D. pulex* mass culture, which would have been harmed by alcohol vapour exposure.

---

### Problem 2: Non-Monotonic NABHS Chronic Survival Curves

**What happened:**
In the chronic toxicity test, NABHS survival was paradoxically **highest at mid-range concentrations** (C3: 2.4 μg/L and C4: 1.2 μg/L) rather than at the lowest concentrations as expected. This produced a highly irregular survival pattern that undermined the dose-response logic.

**Root Cause:**
The primary driver was **specimen age variability**. OECD (2004) guidelines recommend using specimens less than 24 hours old for toxicity tests to standardise sensitivity. Due to strict COVID-19 time constraints that extended set-up procedures, specimens were selected from mass cultures **without regard to age**. Older specimens would naturally be more prone to early-study mortality (particularly in the first 5 days), and this variation in background mortality — not driven by the toxicant — introduced noise into the dose-response relationship. This explains the pronounced **Day 5 mortality drop** visible across all concentration groups in both sanitisers.

Once established, the methodology could not be changed mid-study for consistency across all tests.

**Partial mitigation:**
No full fix was applied, as changing the selection criteria mid-study would have invalidated comparisons across test runs. The problem was acknowledged, documented, and accounted for in the interpretation of results.

---

### Problem 3: Dissolved Oxygen Could Not Be Measured

**What happened:**
Dissolved oxygen (DO) levels could not be recorded in any of the toxicity tests across the study.

**Root Cause:**
The only available DO probe in the laboratory was physically too large to fit into the **10 mL glass vials** used as replicate containers. This was a hardware limitation with no available workaround given the constraints of the study.

**Why this matters:**
Ethanol is known to cause **aggressive oxygen depletion** in freshwater when present in significant concentrations through microbial degradation (NEIWPCC, 2001). *Daphnia magna* has been shown to have reduced body weight and environmental responsiveness at dissolved oxygen levels at or below 1.8 mg/L (Homer & Waller, 1983). Without DO measurements, it is impossible to determine whether some of the mortality observed — especially in the ABHS groups — was due to direct ethanol toxicity or **indirect hypoxia** caused by ethanol-driven oxygen depletion.

---

### Problem 4: Anomalous Carapace Discolouration

**What happened:**
During acute and chronic toxicity tests, multiple specimens were observed to develop a **brown discolouration of the carapace**, losing their characteristic transparency. The discolouration was not permanent — it reversed after moulting events — and did not appear to physically impair specimen movement or behaviour.

**Root Cause (Hypothesised):**
Because the discolouration appeared in specimens from both ABHS and NABHS test groups, the control group, *and* the untested mass cultures, it could not be attributed to either test substance. The working hypothesis is a **calcium-related carapace thickening**, as calcium is a critical resource in daphnid exoskeleton formation and carapace development (Goodberry, 2013). Artificial EPA medium may have a different calcium profile compared to natural pond water, potentially triggering an exoskeletal response. No literature was found to directly confirm or refute this hypothesis.

This observation was documented but not analysed statistically, as no study existed to provide an analytical framework for this phenomenon.

---

### Problem 5: BAC Alkyl Chain Length Unknown

**What happened:**
The Dettol product used as the NABHS source did not disclose the specific **alkyl chain length** of its benzalkonium chloride (BAC) content — whether C12, C14, or C16. This information is routinely disclosed in published toxicity studies (Kim et al., 2020; Lavorgna et al., 2016) because chain length affects BAC sorption rates to materials in solution, which in turn alters the effective bioavailable concentration experienced by test organisms.

**Impact:**
The acute LC₅₀ for NABHS calculated in this study (164 μg/L) was notably higher than values reported by Lavorgna et al. (2016) at 38 μg/L and Kim et al. (2020) at 41.1 μg/L. The unknown alkyl chain length — alongside specimen age variability and differences in dilution solvent (EPA here vs. dimethyl sulfoxide in Kim et al.) — is one of the likely contributing factors to this discrepancy.

---

## Key Findings Summary

| Finding | ABHS (Ethanol) | NABHS (BAC) |
|---------|----------------|-------------|
| **Acute LC₅₀** | 0.406 v/v% *(unreliable — non-sigmoid)* | 164 μg/L *(reliable)* |
| **Chronic LC₅₀** | 0.0454 v/v% *(no CI calculable)* | 7.61 μg/L *(no CI calculable)* |
| **MATC (Acute)** | 0.0406 v/v% | 16.4 μg/L |
| **MATC (Chronic)** | 0.00454 v/v% | 0.761 μg/L |
| **Heart Rate Effect** | Non-significant (p=0.558) | Non-significant (p=0.190) |
| **Dose-response quality** | Poor — non-sigmoid, evaporation artefacts | Moderate — irregular chronic, clean acute |
| **Environmental hazard** | Lower — biodegrades rapidly; indirect O₂ depletion risk | Higher — persistent, bioaccumulates, broader ecotoxicity |

**Compared to literature BAC values in natural environments:**
- Typical BAC in surface waters: ~0.1–1 μg/L
- Maximum measured in civilian freshwaters: ~99.6 μg/L (Poland)
- Hospital effluents: up to 2,800 μg/L (Austria)

The NABHS MATC values of 0.761 μg/L (chronic) overlap with concentrations measurable in real freshwater systems, suggesting genuine environmental risk at elevated exposure scenarios such as near hospital effluent discharge.

---

## Lessons for Future Studies

Based on the problems identified throughout this project, the following methodological improvements are recommended for future ecotoxicological studies involving volatile substances like ethanol:

1. **Cover all test vessels at all times** when working with volatile alcohols — from preparation through to every check-up
2. **Consider incubator atmosphere saturation** (Szabo & Mandrekar, 2008) where feasible — place an open vessel of concentrated alcohol at the incubator base to maintain vapour equilibrium
3. **Use age-standardised specimens** (<24 hours old per OECD guidelines) — even if this extends set-up time
4. **Measure dissolved oxygen** at regular intervals — use appropriately sized probes or adapt replicate vessel size accordingly
5. **Verify BAC alkyl chain length** from manufacturer or via analytical chemistry before commencing testing
6. **Run more concentration groups** for ABHS chronic tests to capture the full dose-response range, including 0% and 100% mortality anchors

---

*Study conducted January–April 2021 at Edinburgh Napier University.*
*Supervisor: Sonja Rueckert | Course: Marine and Freshwater Biology*
