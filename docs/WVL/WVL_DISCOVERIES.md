# AIMI Discoveries (Knowledge Base)

## Document purpose
This document contains discoveries made over time from analyzing the code and docs.

Principles:
- preserve original detail level,
- preserve historical observations,
- preserve hypotheses and confidence levels,
- add new findings INLINE,
- keep code/runtime behaviour authoritative over manuals.

## Fully merged consolidated master version
## Consolidated through 2026-05-26
## Version 2 — DynamicISF / MealRisk / Temperature reinterpretation integrated

## Commit-style KB Version Metadata
- KB version date: 2026-05-26
- KB version label: v2_dynamicisf_integrated
- Repository baseline commit: `2fd4d5f`
- Repository branch focus: `dev_OAPSAIMI`
- KB update source window: chat analyses through 2026-05-26
- Runtime/code state: mixed historical + current runtime interpretation
- Commit traceability status:
  - baseline commit confirmed
  - newer runtime/code observations partially commit-linked
  - not all modern runtime behaviours yet mapped to exact commits

---

# KB GOVERNANCE PRINCIPLES
## Purpose of this KB
This document is a persistent engineering/runtime research knowledge base. It is intended to preserve:
- original detail and historical observations,
- hypotheses and confidence evolution,
- repo evolution and runtime interpretation,
- deprecated or superseded conclusions with explicit labels.

This KB is not meant to be a rolling summary, short troubleshooting note, or rewritten digest.

---

## Update principles
When updating this KB:
- start from the latest complete master version,
- add new findings inline,
- do not replace or shorten the existing KB as the final state,
- do not remove prior content silently,
- mark outdated items explicitly as deprecated, superseded, or invalidated.

These principles are intended to protect against automatic summarization that loses original detail.

---

## Future chat behavior
For future updates:
- treat the latest complete master KB as the primary persistent context,
- merge new findings into that KB,
- use standalone draft notes only temporarily, never as the final KB state.

---

## Settings
This section is for user-applied settings changes over time, so they can be tracked separately from code/runtime discoveries.

### PK/PD settings
- 2026-07-05 13:46 — User switched "DynISF trajectory tuning" from On to Off in the app settings.
- 2026-07-05 13:46 — This change is recorded as a user-applied settings adjustment and should be treated as relevant runtime context for later interpretation.
- 2026-07-03 — User noted that "Correction aggressiveness" and "Late insulin action (SMB)" were already at the left / more cautious end of their sliders.
- 2026-07-03 — No further move was applied for those two PK/PD sliders because they were already set to the most cautious position.

### Other settings
- Add future setting changes here with a timestamp and a short description.
- 2026-08-02 — Legacy setting `openaps_smb_min_5m_carbimpact` was historically used in older OpenAPS/AMA-style SMB logic as a minimum floor for inferred carb impact, preventing very small meal-related BG effects from being treated as negligible.
- 2026-08-02 — Current AIMI implementation does not use this setting in the main calculation path; the AIMI plugin passes `min_5m_carbimpact = 0.0` and marks it as "not used" in the code, so the value in user settings is effectively ignored by AIMI today.

#### SMB tail-damping floor 
- Lowering the SMB tail-damping floor makes the guard stronger.
- Raising the floor makes the guard weaker.

- 0.70 = strongest damping (up to −30% SMB at tail)
- 0.85 = default damping (up to −15% SMB at tail)
- 0.92 = lightest damping (up to −8% SMB at tail)
- 1.0 = guard effectively disabled

Practical effect
Lower floor → more SMB reduction when insulin tail/IOB is high → smoother, more conservative, better hypo protection
Higher floor → less SMB reduction on the tail → more aggressive SMB delivery, better at chasing long rises

### Summary of recent changes and rationale
- 2026-08-02 — Working summary of the recent tuning changes and the reasoning behind them.
  My question:
  I experience overcorrections again. Perhaps because my days in the weekend go different ???
And still I see that when the rise of my BG is already flattening down that aimi decides to give pretty large SMB.
During the daytime its hard to correct this with holding back the basal because then its pretty low for me (0,25 at 16:00)

  Answer MTR:
  You have an option to make it more gentle on the queue in autodrive

  Actions:
- AutoDrive max basal: 6 -> 4.5 U/h. Rationale: reduce sustained basal-side exposure and prolonged high temp basal pressure without broadly breaking meal control.
-  turned off HTR hyper trajectory SMB release. Rationale: This makes the system less willing to act with full authority on a belief-driven decision.
It is a good “less aggressive” lever if you want the loop to be more hesitant.

- earlier
- DynamicISF Adjustment Factor: 200 -> 100. Rationale: reduce amplified adaptive correction aggressiveness, reduce delayed braking, and limit late steep drops caused by over-interpretation of rising or flattening curves.
- CombinedDelta: 1 -> 2. Rationale: require more confirmed rise before AutoDrive escalates, which should reduce false-positive acceleration handling on Libre2 lag/noise.

- DynISF trajectory tuning: switched from On to Off in app settings. Rationale: keep the system more conservative while the current architecture still appears too willing to escalate on changing day structure and flattening curves.
- Why these changes were grouped together: the observed issue pattern was not only “too much SMB,” but rather cumulative insulin exposure from AutoDrive/adaptive basal persistence plus delayed braking. The changes therefore target the escalation and persistence layers first, not only the final SMB size.

### AutoDrive: HTR + RBT
- HTR (Hyper Trajectory SMB Release) and RBT (Recursive Belief Tree) are both part of the AutoDrive decision stack.
- HTR is the trajectory-based release layer: when AutoDrive V3 is active, it can lift V3 SMB from a credible predicted hyper-rise trajectory rather than waiting for a fixed 200 mg/dL threshold.
- RBT is the belief-gating layer: it unfolds nested belief leaves each loop tick, evaluates credibility and authority, and can cap or suppress SMB/TBR when the signal is weak or conflicting.
- Together, they make AutoDrive more conditional: HTR can raise the SMB floor when the trajectory is credible, while RBT can still hold back or constrain the action if the underlying belief is not robust enough.
- The aggressive HTR setting raises HTR SMB floors by about 15% when the scenario confirms a hyper rise.
- 2026-08-02 — User-reported runtime pattern: overcorrections recur when the day structure changes, especially on weekend-style days, and the problem is particularly visible when BG is already flattening or peaking yet AIMI still decides to deliver a relatively large SMB.
- This supports the interpretation that the issue is not only raw SMB size, but also the fact that the AutoDrive/HTR layer can still treat a flattening curve as credible enough for a late correction when the daily routine changes.
- The user also noted that during daytime it is hard to compensate by holding basal back because the basal is already very low (for example around 0.25 U/h at 16:00), which reinforces the hypothesis that the main problem is cumulative exposure from AutoDrive/adaptive basal persistence rather than a simple need for more basal reduction.

---

## Core metadata guidance
Every update should include reasonable metadata for traceability.

### Recommended metadata
- KB version label
- KB version date
- author/update timestamp
- source chat window or update context
- repository branch and baseline commit
- analyzed commit range or repo state when known
- runtime/code state classification when known

If exact commits are unknown:
- uncertainty should be stated explicitly,
- commit traceability must not be fabricated.

---

## MANDATORY HISTORICAL PRESERVATION
Historical findings are valuable even when partially outdated.

Therefore:
- preserve old interpretations,
- preserve old runtime observations,
- preserve old hypotheses,
- preserve old practical findings,
- preserve old settings experiments.

Then annotate:
- whether they still appear valid,
- partially valid,
- deprecated,
- or architecture-dependent.

The KB must function as:
> an evolving research history.

NOT as:
> a cleaned-up final summary.

---

## MANDATORY INLINE INTEGRATION RULE
New findings must ALWAYS be integrated into:
- existing relevant sections,
- confidence classifications,
- runtime interpretation sections,
- recommendations,
- and repo evolution sections.

Do NOT:
- place all new findings only at the end,
- create isolated appendices as final state,
- or create disconnected mini-updates.

---

## MANDATORY CONFIDENCE CLASSIFICATION
Every important conclusion should ideally be classified as:

### Confirmed code behaviour
- directly verified in source code.

### Runtime/log-confirmed behaviour
- repeatedly observed in logs/runtime.

### Behaviour-inferred but plausible
- strong behavioural interpretation but not yet code-confirmed.

### Repo-evolution interpretation
- inferred from commit direction/refactors.

### Requires future validation
- important but still uncertain.

---

## MANDATORY USER PRIORITY
The user explicitly prioritizes:
1. historical preservation,
2. cumulative engineering knowledge,
3. commit-aware reasoning,
4. runtime truth over documentation,
5. long-term continuity across chats.

Important operating note:
- The user does not want code changes for this work; when a tuning or behavior change is discussed, prefer only settings that are already exposed in the app's settings pages.

Future KB maintenance MUST prioritize these goals even if:
- the KB becomes very large,
- repetitive,
- or historically layered.

Compression/summarization is NOT the primary objective.

---

# KB MAINTENANCE RULES
## Mandatory consolidation workflow
For every future KB update:
- ALWAYS use the most recent complete master KB as the base.
- NEVER create partial addendums as final KB state.
- ALWAYS preserve all historical findings unless explicitly deprecated.
- ALWAYS integrate new findings INLINE into existing relevant sections.
- ALWAYS create a complete SUPERSET master KB.
- Runtime/code behaviour has priority over older manuals.
- Clearly distinguish:
  - confirmed code behaviour,
  - runtime/log-confirmed behaviour,
  - hypotheses,
  - repo interpretations,
  - recommendations.

---

## Commit/version guidance
Every update should include:
- KB version date
- KB version label
- repository branch
- baseline commit
- known runtime/code window
- provenance or source note when useful

If exact commit mapping is unknown:
- explicitly state uncertainty
- never fabricate commit traceability

New findings should be integrated inline rather than collected in a dedicated "Changes Since Previous Version" section.

---

## Historical preservation rules
Do NOT:
- overwrite old findings silently,
- remove older interpretations without explanation,
- collapse confidence distinctions,
- summarize away historical runtime observations.

Instead:
- preserve historical context,
- add reinterpretations INLINE,
- mark outdated interpretations clearly.

---

## Newly integrated findings
### DynamicISF Adjustment Factor
- Direct code inspection confirmed DynamicISF Adjustment Factor directly multiplies dynISF-related TDD behaviour.
- Walter discovered his value remained at 200 while modern defaults appear closer to 100-120.
- This is now considered one of the strongest setting-level suspects for excessive adaptive aggressiveness.

### New active experiment
- DynamicISF Adjustment Factor changed:
  - 200 -> 100
- Goal:
  - reduce delayed braking,
  - reduce persistent correction pressure,
  - reduce late rapid drops.

### Temp Target reinterpretation
- Runtime observations increasingly suggest Temp Targets behave more like soft modulation than dominant braking.
- Modern AIMI architecture appears increasingly dominated by:
  - trajectory systems,
  - AutoDrive state,
  - adaptive basal scaling,
  - physiology modulation.

### Alcohol + warmth physiology event
- Important late-drop event occurred:
  - without walking,
  - during alcohol intake,
  - during warm weather.
- This strongly reinforced the hypothesis that delayed adaptation to changing physiology is central.

### Meal Risk High runtime behaviour
- AIMI Context -> Meal Risk High triggered immediate ~2U insulin delivery during fondue/bread/alcohol situation.
- Meal Risk High now suspected to act as:
  - escalation permission,
  - aggressive meal-state confirmation,
  - or prebolus-like activation.

### Calm breakfast observation
- Yoghurt + nuts breakfast behaves relatively well autonomously.
- This reinforced the interpretation that baseline settings are likely not fundamentally broken.
- Main issue appears concentrated in:
  - escalation states,
  - delayed de-escalation,
  - adaptive amplification.

---

# Status
Living knowledge base for:
- AIMI / dev_OAPSAIMI
- AutoDrive V3
- DynISF / PKPD
- Adaptive Basal
- Libre2 + Lyumjev behaviour
- trajectory systems
- activity/sport systems
- physiology modulation
- real-world log analysis
- dynamicISF amplification behaviour

Purpose:
- preserve practical and technical findings from iterative analysis,
- bridge the gap between manuals and runtime/code behaviour,
- maintain a reusable base for future chats and future repo updates.

Important:
- code/runtime behaviour has priority over older manuals,
- many conclusions remain behaviour-inferred and should be revalidated against future commits.

---

# Docs
are inside folder /docs

# Settings
Actual decrypted settings are in the folder docs/WVL: 2026-05-20_151959_full.json.decrypted.sanitized.pretty.json

# Repository Baseline
## Repo
- `MTR93600/OpenApsAIMI`
- branch focus: `dev_OAPSAIMI`

## Baseline
- baseline commit: `2fd4d5f`

## Observed evolution after baseline
Recent AIMI evolution strongly suggests movement toward:
- trajectory-aware insulin logic,
- physiological context engines,
- adaptive smoothing,
- UKF/Kalman concepts,
- stronger exercise protection,
- sustained adaptive basal systems,
- predictive braking,
- confirmed-rise filtering,
- physiology-aware modulation,
- AutoDrive throttling/deduplication,
- AI Auditor integration,
- T3C trajectory braking,
- forward-looking hyporisk prevention.

---

Recent AIMI evolution strongly suggests movement toward:
- trajectory-aware insulin logic,
- physiological context engines,
- adaptive smoothing,
- UKF/Kalman concepts,
- stronger exercise protection,
- sustained adaptive basal systems,
- predictive braking.

---

# Executive Summary
## Most important current conclusion
For this specific setup:
- Libre2
- Lyumjev
- AutoDrive V3
- strong manual prebolus
- short post-meal walks

The main issue appears NOT to be:
- pure SMB aggressiveness.

Most likely mechanism:
> sustained insulin exposure from AutoDrive V3 + Adaptive Basal / BASAL_UNIFIED_SCALING + delayed braking while Libre2 lags behind real physiology.

---

## NEW major refinement
DynamicISF amplification now appears to be a potentially CENTRAL amplification layer.

Direct code inspection confirmed:

```kotlin
val dynISFadjust = sp.getString(key_DynISFAdjust, "120") / 100.0
val dynISFadjusthyper = sp.getString(key_DynISFAdjusthyper, "150") / 100.0

bg > 180 -> tdd * dynISFadjusthyper
else -> tdd * dynISFadjust
```

Interpretation:
- DynamicISF Adjustment Factor directly amplifies dynISF-related TDD behaviour.
- Values above 100 increase adaptive correction aggressiveness.
- This is confirmed code behaviour.

Walter was still using:

```text
DynamicISF Adjustment Factor = 200
```

while modern defaults appear near:
- 100-120.

Current interpretation:
> adaptive sensitivity modulation may effectively have been heavily amplified.

This may strongly contribute to:
- persistent correction pressure,
- delayed braking,
- weak apparent Temp Target influence,
- strong late drops,
- exaggerated response to:
  - alcohol,
  - warmth,
  - post-meal walking,
  - trajectory acceleration.

---

## Strongest confirmed practical finding
Changing:
- `CombinedDelta: 1 -> 2`

significantly improved behaviour.

Interpretation:
> AutoDrive was likely triggering too easily on short-term CGM movement.

---

## Current best combined hypothesis
1. rise detected
2. DynamicISF amplification increases adaptive sensitivity interpretation
3. AutoDrive V3 escalates
4. Adaptive Basal scaling increases
5. correction pressure persists too long
6. Libre2 still lags behind real physiology
7. Lyumjev begins acting strongly
8. insulin exposure already accumulated
9. late braking occurs
10. steep decline / near-hypo follows

---

# User Setup Context
## Insulin
- Lyumjev

## Sensor
- Freestyle Libre 2

## Activity pattern
Typical routine:
- Monday / Tuesday / Thursday / Friday:
  - ~20 min walk shortly after breakfast
  - ~20 min walk shortly after lunch

- Monday / Tuesday / Thursday:
  - ~45 min activity shortly after dinner

Important implication:
> glucose trajectory may reverse physiologically earlier than Libre2 reports.

---

# Calm Breakfast Observation
## Important real-world finding
Breakfast consisting of:
- yoghurt,
- nuts,
- relatively low glycemic load,

works relatively well when AIMI handles it autonomously.

Interpretation:
- baseline profile is probably NOT fundamentally broken.
- main problem appears concentrated in:
  - dynamic escalation states,
  - aggressive adaptive interpretation,
  - delayed de-escalation.

This strongly supports:
- adaptive amplification,
- AutoDrive persistence,
- delayed braking,

rather than globally incorrect basal/ISF settings.

---

# AutoDrive V3
15-07-2026 MTR: 
no more V1 and V3, only one version, you have now a fusion autodrive

## Updated interpretation
Older interpretation:
> aggressive SMB/prebolus engine.

Current interpretation:
> dynamic metabolic-response state engine.

It appears to influence:
- trigger sensitivity
- SMB permissions
- sustained TBR behaviour
- adaptive basal scaling
- trajectory interpretation
- braking behaviour
- PKPD context
- physiological modulation

---

## Observed behaviour in logs
Frequent markers:
- `Autodrive: ✔V3`
- `AdaptiveBasal`
- `BASAL_UNIFIED_SCALING`
- `Basal×2.15`

Observed pattern:
- repeated prolonged high temp basals,
- often reaching 5.5 U/h ceiling,
- while SMBs are already partially reduced by PKPD Guard.

Interpretation:
> basal-side exposure is probably more important than raw SMB size.

---

# AutoDrive: Recursive Belief Tree (RBT) + Hyper Trajectory Release (HTR)
## Description
Recursive Belief Tree (RBT) is AIMI’s nested decision architecture that unfolds belief leaves every loop tick and uses credibility, authority gating, and pattern context to influence SMB/TBR decisions.

It is not just another dose calculator. RBT is the engine that lets AIMI combine physiological patterns, recursive belief authority, and HTR/RBT merge rules so that complex states like exercise, sleep debt, or counter-regulatory risk can cap or suppress insulin delivery more conservatively.

HTR complements this layer by acting as the trajectory-based release path: it can raise the SMB floor when a credible hyper-rise trajectory is detected, while RBT decides whether that signal is trustworthy enough to act on.

In practice, RBT can run in:
- **shadow mode** for export and validation,
- **authority mode** for real pump action when the tree is trusted.

This means the combined AutoDrive layer behind phrases like `AutodriveV3+RBT` and `AutodriveV3+HTR` is able to say “this belief is not credible enough yet” before allowing aggressive delivery.

---

# Adaptive Basal / BASAL_UNIFIED_SCALING
## Current major suspect
Strong evidence suggests:
> sustained adaptive basal exposure is central to the observed near-hypo pattern.

---

## Important observed values
From logs:
- `Basal×2.15`
- repeated 5.5 U/h TBR

From learner state:
- shortTermMultiplier = 2.5
- mediumTermMultiplier = 2.5

Interpretation:
- AIMI appears to calculate a combined adaptive basal scaling.
- `Basal×2.15` is likely NOT a single setting.
- It is probably the resulting effective multiplier after multiple adaptive systems.

Likely contributors:
- Adaptive Basal
- Basal Learner
- trajectory state
- meal state
- PKPD
- unified scaling
- amplified dynISF interpretation

---

# SMB vs Basal Exposure
## Important finding
Logs suggest:
- SMBs are often moderate,
- and frequently reduced by PKPD Guard.

Meanwhile:
- temp basal frequently remains high for prolonged periods.

Interpretation:
> SMB safety may already be functioning reasonably,
while basal/TBR persistence remains aggressive.

---

# DynamicISF Adjustment Factor
## NEW confirmed code behaviour
Direct code inspection confirmed:

```kotlin
val dynISFadjust = sp.getString(key_DynISFAdjust, "120") / 100.0
```

and:

```kotlin
tdd * dynISFadjust
```

Interpretation:
- DynamicISF Adjustment Factor is NOT cosmetic.
- It directly affects dynISF adaptive behaviour.
- Increasing above 100 increases adaptive correction aggressiveness.

---

## Important practical implication
Walter previously used:

```text
DynamicISF Adjustment Factor = 200
```

Interpretation:
- adaptive sensitivity influence may effectively have been strongly amplified.

This now becomes one of the strongest concrete setting-level suspects.

---

## New active test
Current active experiment:

```text
DynamicISF Adjustment Factor:
200 -> 100
```

while keeping other settings stable.

Purpose:
- isolate adaptive amplification behaviour.

Monitor for:
- fewer rapid late drops,
- weaker prolonged TBR,
- earlier braking,
- fewer Safety Halt/LGS events,
- preserved meal control.

---

# CombinedDelta
## Current interpretation
CombinedDelta controls:
> how easily AIMI believes a rise deserves aggressive escalation.

---

## Behaviour
### Low value (1)
- extremely sensitive
- vulnerable to Libre2 noise/lag
- easier AutoDrive escalation
- more false-positive acceleration detection
- higher overshoot risk

### Moderate value (2)
- more confirmation required
- smoother behaviour
- reduced false escalation

---

## Real-world observation
Changing:
- 1 -> 2

produced clear practical improvement.

This remains one of the strongest confirmed observations.

---

# Libre2 + Lyumjev Behaviour
## Important conclusion
Libre2 + Lyumjev behaves differently from:
- slower insulins,
- lower-lag CGM systems.

---

## Observed risk pattern
1. glucose starts rising
2. AutoDrive escalates
3. Lyumjev begins acting rapidly
4. Libre2 still reports delayed rise
5. AIMI continues correction
6. real physiology already turning downward
7. overshoot / near-hypo occurs

---

## Important implications
This setup benefits from:
- less trigger nervousness,
- smoother adaptation,
- less persistent TBR,
- stronger braking,
- earlier trajectory skepticism.

---

# Alcohol + Warmth Physiology Observation
## Important real-world event
Rapid late drop occurred during:
- beer,
- sangria,
- warm weather (~29°C),
- relaxed reading,
- no walking,
- no continuing meal.

Interpretation:
- delayed drops are NOT limited to post-meal walking.
- changing physiology itself appears central.

Likely contributors:
- increased insulin absorption from warmth,
- reduced hepatic glucose output from alcohol,
- Libre2 lag,
- amplified adaptive dynISF behaviour.

This strongly reinforces:
> delayed adaptation to changing physiology.

---

# Temp Target Reinterpretation
## New interpretation
Observed behaviour strongly suggests:
- Temp Targets are no longer dominant control layers in modern AIMI.

Likely hierarchy now:
1. trajectory / relevance systems
2. AutoDrive state
3. adaptive basal scaling
4. PKPD interpretation
5. physiology modulation
6. Temp Target bias

Interpretation:
> Temp Targets may now function more as soft modulation than hard braking.

This remains behaviour-inferred but strongly supported by runtime observations.

---

# Meal Risk High Context
## Runtime observation
Using:
- AIMI Context → Meal Risk High

produced:
- immediate ~2U insulin delivery.

Interpretation:
- Meal Risk High appears substantially more aggressive than passive contextual bias.
- It may function closer to:
  - escalation permission,
  - meal-state confirmation,
  - or prebolus-like activation.

However:
- exact internal architecture remains unconfirmed.

---

# Fondue Event Interpretation
## Context
Meal Risk High test occurred during:
- cheese fondue,
- bread,
- beer,
- sangria,
- warm weather,
- relaxed inactivity.

Interpretation:
- from AIMI perspective this genuinely resembles a prolonged high-risk meal trajectory.

Therefore:
- the initial 2U itself may not necessarily have been unreasonable.

However:
- the larger concern remains:
  - persistence,
  - cumulative exposure,
  - delayed braking.

---

# Sport / Activity Logic
## Confirmed runtime evolution
Current runtime/code behaviour appears materially different from older manuals.

Observed behaviour includes:
- explicit SMB lockout during activity,
- possible basal stop under BG<=220,
- exercise insulin lockout,
- strong hypo-first protection.

Practical implication for Walter:
> sport/activity context should probably NOT be routinely used for short post-meal walks.

Reason:
- current activity lockout appears too aggressive for this specific use case.

---

# Current Working Recommendations
## Keep
- CombinedDelta = 2
- Adaptive smoothing ON
- trajectory tuning shadow-only
- AutoDrive max basal = 4.5 U/h

## Current major active test
- DynamicISF Adjustment Factor:
  - 200 -> 100

## Avoid for now
- walking/activity context for short post-meal walks
- live trajectory tuning
- aggressive PKPD tweaking
- drastic SMB reduction
- drastic MaxIOB reduction
- increasing correction aggressiveness further

---

# Current Confidence Classification
## Strongly supported by logs/settings/code
- sustained TBR importance
- Basal×2.15 behaviour
- CombinedDelta improvement
- PKPD Guard SMB braking
- frequent 5.5 U/h temp basal
- AutoDrive V3 persistence
- explicit SMB lockout during activity
- weak practical Temp Target dominance
- DynamicISF Adjustment Factor directly amplifies adaptive behaviour
- DynamicISF factor 200 likely materially increased aggressiveness

## Behaviour-inferred but plausible
- trajectory tunnel relationship
- adaptive basal emergent scaling
- Libre2 lag contribution magnitude
- timing mismatch between physiology and AutoDrive
- Temp Target deprioritization in modern AIMI hierarchy

## Needs future validation
- exact interaction between:
  - DynamicISF amplification
  - AutoDrive V3
  - adaptive basal
  - PKPD
  - PhysioModulation
  - trajectory relevance
  - AI Auditor

---

# Final Consolidated Interpretation
Current strongest combined interpretation:

> Walter's setup likely suffers from amplified adaptive insulin interpretation during rapidly changing physiology, leading to excessive cumulative insulin exposure before braking fully activates.

Most important contributors currently suspected:
- DynamicISF Adjustment Factor = 200,
- AutoDrive persistence,
- Adaptive Basal scaling,
- Libre2 lag,
- Lyumjev fast action,
- delayed physiology recognition.

Most important current experiment:

```text
DynamicISF Adjustment Factor:
200 -> 100
```

with all other major settings kept stable.

## Consolidated through 2026-05-26
This document is intended to become the new complete master/source-of-truth KB.

Merged sources:
- original AIMI knowledge base,
- 2026-05-22 upgrade-analysis additions,
- ALL major findings from the 2026-05-26 discussion.

Principles:
- preserve original detail level,
- preserve historical observations,
- preserve hypotheses and confidence levels,
- add new findings INLINE,
- keep code/runtime behaviour authoritative over manuals.

---

## Current best hypothesis
Problem pattern:
1. meal rise begins
2. AutoDrive V3 escalates
3. Adaptive basal scaling increases
4. temp basal often reaches 5.5 U/h
5. Lyumjev + walking increase insulin effectiveness
6. Libre2 still shows delayed/rising curve
7. insulin exposure already accumulated
8. later steep fall / near-hypo occurs
9. Safety Halt / LGS triggers only after decline is already underway

---

## Most important current recommendations
### Keep
- CombinedDelta = 2
- Adaptive smoothing = ON
- DynISF trajectory tuning = shadow only

### Recently changed
- AutoDrive max basal:
  - 5.5 -> 4.5 U/h

### Do NOT currently do
- enable live trajectory tuning
- aggressively lower MaxIOB
- drastically reduce SMB
- globally reduce profile basal
- rely on sport mode for short post-meal walks

---

# Core Behaviour Findings
# 1. AutoDrive V3
## Updated interpretation
Older interpretation:
> aggressive SMB/prebolus engine.

Current interpretation:
> dynamic metabolic-response state engine.

It appears to influence:
- trigger sensitivity
- SMB permissions
- sustained TBR behaviour
- adaptive basal scaling
- trajectory interpretation
- braking behaviour
- PKPD context
- physiological modulation

---

## Observed behaviour in logs
Frequent markers:
- `Autodrive: ✔V3`
- `AdaptiveBasal`
- `BASAL_UNIFIED_SCALING`
- `Basal×2.15`

Observed pattern:
- repeated prolonged high temp basals,
- often reaching 5.5 U/h ceiling,
- while SMBs are already partially reduced by PKPD Guard.

Interpretation:
> basal-side exposure is probably more important than raw SMB size.

---

## New repo evolution relevance
Recent repo evolution additionally introduced:
- confirmed-rise logic,
- AutoDrive throttling,
- periodic deduplication,
- explicit decision handling,
- trajectory relevance filtering.

Interpretation:
> newer AutoDrive generations appear increasingly focused on avoiding false escalation and delayed braking.

---

# 2. Adaptive Basal / BASAL_UNIFIED_SCALING
## Current major suspect
Strong evidence suggests:
> sustained adaptive basal exposure is central to the observed near-hypo pattern.

---

## Important observed values
From logs:
- `Basal×2.15`
- repeated 5.5 U/h TBR

From learner state:
- shortTermMultiplier = 2.5
- mediumTermMultiplier = 2.5

Interpretation:
- AIMI appears to calculate a combined adaptive basal scaling.
- `Basal×2.15` is likely NOT a single setting.
- It is probably the resulting effective multiplier after multiple adaptive systems.

Likely contributors:
- Adaptive Basal
- Basal Learner
- trajectory state
- meal state
- PKPD
- unified scaling

---

## Current tuning direction
Most reasonable first intervention:
- reduce AutoDrive max basal

Recently changed:
- 5.5 -> 4.5 U/h

Purpose:
- reduce sustained insulin exposure,
- without destroying meal control.

---

## Important post-upgrade note
Recent commits indicate adaptive basal scaling logic changed materially.

Therefore:
> old conclusions remain important but require fresh validation after upgrade.

---

# 3. SMB vs Basal Exposure
## Important finding
Logs suggest:
- SMBs are often moderate,
- and frequently reduced by PKPD Guard.

Meanwhile:
- temp basal frequently remains high for prolonged periods.

Interpretation:
> SMB safety may already be functioning reasonably,
while basal/TBR persistence remains aggressive.

---

## PKPD Guard
Observed behaviour:
- SMB reduction occurs frequently.

Examples:
- 0.58 -> 0.29 U
- 0.77 -> 0.46 U
- 0.78 -> 0.47 U

Interpretation:
> PKPD Guard brakes SMB,
but does not necessarily prevent excessive basal-side insulin exposure.

---

# 4. CombinedDelta
## Current interpretation
CombinedDelta controls:
> how easily AIMI believes a rise deserves aggressive escalation.

## 3.1 CombinedDelta
CombinedDelta is a sensitivity parameter in AIMI’s AutoDrive tuning. In the code it maps to the `OApsAIMIcombinedDelta` preference, and it is applied to a blended glucose velocity value:
- `combinedDelta = (delta + predictedDelta) / 2.0`

That means it is not a direct insulin dose or a fixed rate. It is a trigger threshold: the system requires both current CGM delta and short-term predicted delta to be strong enough before AutoDrive escalates aggressively.

In practice:
- a lower value like `1` makes AIMI more eager to react to small rises, which can be too sensitive on Libre2 and Lyumjev because of sensor lag and short-term noise.
- a moderate value like `2` requires more confirmed momentum and tends to smooth the rise-response, reducing false-positive aggressive escalation.
- in the current user setup, changing `CombinedDelta` from `1` to `2` produced a clear improvement in behaviour.

So the main effect of CombinedDelta is to tune how easily AutoDrive interprets a short-term upward trend as a valid reason to increase SMB/TBR, not to directly change the amount of insulin delivered.

---

## Behaviour
### Low value (1)
- extremely sensitive
- vulnerable to Libre2 noise/lag
- easier AutoDrive escalation
- more false-positive acceleration detection
- higher overshoot risk

### Moderate value (2)
- more confirmation required
- smoother behaviour
- reduced false escalation

---

## Real-world observation
Changing:
- 1 -> 2

produced clear practical improvement.

This remains one of the strongest confirmed observations.

---

# 5. AIMI behavior families and Control Center

AIMI uses five high-level behavior families to translate product intent into legacy AIMI settings. These families are the main product-facing controls in the AIMI Control Center.

The AIMI Control Center is the primary product entry point. It is designed to let users adjust broad behavior intentions, not raw internal knobs. The Control Center:

- displays the current state of the five families using existing preferences,
- lets the user move a slider relative to that current state,
- projects the resulting behavior change before writing anything,
- then writes back only the legacy keys that belong to the selected family profile.

This means the Control Center is not a hidden preset system. It is a behavior-level cockpit that preserves existing user settings and avoids sudden, unexplained rewrites.

- **Protection**
  - Controls how conservative or aggressive AIMI is when reacting to low-glucose risk and rising BG.
  - Influences correction strength, max SMB, max IOB, and safety thresholds.
  - A more protective profile increases hypoglycemia prevention and reduces aggressive correction.

- **Meal capture**
  - Controls how strongly AIMI interprets a rise as an undeclared meal or rapid glucose event.
  - Influences automatic meal detection, recursive belief authority, hyper-trajectory release, and prebolus behavior.
  - A more assertive meal-capture profile makes AIMI respond faster to unexplained rises, while a more cautious profile avoids false meal interpretations from stress, dawn, or hormones.

- **Stability**
  - Controls how much AIMI smooths glucose trends and resists oscillation.
  - Influences damping, smoothing, and how much the system waits before changing insulin when glucose is already moving.
  - A stability-focused profile favors gentler corrections and fewer abrupt changes, helping reduce yoyo and instability.

- **Physio**
  - Controls the importance of physiological context such as exercise, hormones, and other body-state signals.
  - Influences whether AIMI uses physiological input aggressively or stays closer to standard trajectory-based control.
  - A stronger physio profile lets AIMI trust body-state context more, while a weaker profile keeps the system more conservative and model-driven.

- **Autonomy**
  - Controls how much AIMI can act without user intervention.
  - Influences auto-drive activation, safety guard authority, and how freely the system can change basal/SMB settings.
  - Higher autonomy means AIMI takes larger, more automatic actions; lower autonomy means it stays more restrained and leaves more decisions to the user.

These families are not separate algorithms themselves. They are product-level lenses that project into the real AIMI configuration layer.

The Control Center reads current preference values, maps them into these family levels, lets the user move a family slider relative to the current state, and only then writes back the corresponding legacy keys.

The AIMI Advisor also works with the same family taxonomy: it recommends family-level changes instead of raw preference keys, making Control Center guidance more understandable.

---

# 6. Meal Advisor and meal modes

AIMI includes a Meal Advisor that can calculate and deliver insulin for meals.
It is designed to do three things:
1. compute the correct bolus,
2. send the bolus automatically,
3. activate a TBR with override safety limits when needed.

From the docs:
- the meal bolus formula is:
  `netBolus = (Carbs / IC) - IOB - (TBR_rate × 0.5h)`
- the system can send the bolus through `finalizeAndCapSMB(...)`.
- meal mode TBR can be set with `overrideSafetyLimits = true`, which allows TBR up to `max_basal`.

Meal modes are explicit user actions such as:
- breakfast, lunch, dinner, highcarb, snack, sport, sleep, stop.

## Sport mode
- `sport` is a dedicated exercise mode code in AIMI.
- It is treated as declared activity / exercise context, not a regular meal.
- When sport mode is active, AIMI enables `exerciseInsulinLockout`.
- This typically means:
  - **SMB = 0**
  - **basal is reduced**
  - basal can sometimes go to **0 U/h** if BG is low enough (≤ 220 mg/dL)
- The goal is to avoid insulin stacking and protect against exercise-induced hypoglycemia.
- In some high-BG or strong physiological stress situations, AIMI may still adjust delivery under a safe override rule, but it remains conservative.

The Meal Advisor docs show that meal handling is a deliberate process rather than an accidental injection path.

# 7. PKPD and absorption behavior

PKPD is a core part of AIMI safety.
The docs identify a specific problem: after a fix to early return logic, SMB and basal decisions could overlap and cause overcorrection.

AIMI’s PKPD work is focused on:
- identifying whether insulin action is still rising,
- using DIA/peak/tail and IOB activity to adjust SMB timing,
- applying soft modulation instead of hard blocking,
- preventing rapid repeat SMBs during the insulin absorption window.

The audit document defines a physiological safe window:
- `PRE_ONSET`, `ONSET`, `PEAK`, and `TAIL` phases,
- each phase receives a softer SMB factor and a delay before the next SMB.

This is a key safety concept: insulin should not be delivered too aggressively while earlier doses are still absorbing.

---

## PKPD Pragmatic Relief

### What it is
PKPD pragmatic relief prevents the PKPD absorption guard from over-damping SMB during explicit meal contexts or confirmed high rises.

The PKPD guard normally reduces SMB to prevent overdosing while previous insulin is still absorbing. Pragmatic relief adds a **floor** to this reduction, ensuring SMB retains meaningful intent in priority situations.

### How it works
When pragmatic relief is **enabled** AND you are in an **aggressive priority context**, the PKPD guard cannot reduce SMB below the **relief minimum factor** (default **0.75**).

Example:
- PKPD guard calculates: SMB reduction to 0.30×
- Relief floor is set to: 0.75×
- **Result: SMB is actually reduced to only 0.75×** (not 0.30×)

This preserves meal bolus intent while still maintaining PKPD safety.

### When it activates (priority contexts)
Relief applies in these situations:
1. **Explicit meal advisor** — you manually triggered meal bolus calculator
2. **Active meal mode** — breakfast, lunch, dinner, snack, or high-carb mode is on
3. **Confirmed high rise** — AIMI detects confirmed aggressive glucose rise

### User-facing settings
| Setting | Default | Range | Effect |
|---------|---------|-------|--------|
| **Enable PKPD pragmatic relief** | ON | Toggle | Master on/off for the feature |
| **PKPD relief minimum factor** | 0.75 | 0.50–1.00 | Floor multiplier for SMB during priority contexts |

### Tuning guidance
- **Increase to 0.80–0.90** if SMB is still being over-dampened during explicit meals or high rises → preserves more meal control
- **Decrease to 0.65–0.70** if you observe post-meal drops that are too fast or recurrent hypos → allows more aggressive PKPD braking
- **Disable entirely** only if you observe recurrent over-corrections despite hard caps

### Relevance for Walter's setup
Currently enabled by default (No, it is disabled (09-07-26). This mechanism specifically addresses the problem of PKPD guard being too conservative during meal contexts—which aligns with your KB hypothesis that the issue is not pure SMB aggressiveness but rather delayed braking and cumulative insulin exposure. Relief helps meals get proper insulin intent while still preventing abuse of the absorption window.

# 8. AI Decision Auditor

AIMI also has a second-brain auditor layer.
This is described as a separate AI Decision Auditor that:
- analyzes AIMI decisions in context,
- generates a verdict and confidence,
- can optionally modulate decisions softly.

The auditor supports three modes:
- **AUDIT_ONLY**: analyze decisions but do not change them.
- **SOFT_MODULATION**: modulate decisions if confidence is high enough.
- **HIGH_RISK_ONLY**: modulate only when explicit risk flags are present.

The auditor never issues direct pump commands or free insulin doses.
It only adjusts the existing AIMI decision using bounded factors such as SMB multiplier and interval extension.

# 9. AIMI advisor / profile recommendations

AIMI contains a profile advisor system that reviews settings and recommends preference changes.
This advisor is more like a coach than an automatic tuner.
It collects metrics over days and suggests changes to PKPD settings, ISF fusion, max SMB, and other AIMI preferences.

Important points from the advisor docs:
- the advisor currently covers only a subset of AIMI prefs.
- it has stronger coverage for PKPD settings than for trajectory or meal mode settings.
- many recent features such as trajectory guard, adaptive basal governance, and WCycle are not yet fully covered by advisor recommendations.
- advisory recommendations may be visible, applyable, or guidance-only.

# 10. Implementation and quality status

The AIMI implementation is actively maintained and audited.
Key development topics in the docs include:
- a refactor checklist for `DetermineBasalAIMI2`,
- performance and caching improvements,
- safety and prediction test coverage,
- removal of obsolete `runBlocking` calls,
- logging of final decisions and audit tags.

The refactor checklist shows that:
- many safety and prediction components are complete,
- some performance and refactor tasks remain,
- the adaptive basal governance plan is on-device only and does not use cloud.

---

# 11. Adaptive Smoothing / UKF
## Important insight
Adaptive smoothing is NOT just visual smoothing.

It likely:
- smooths noisy CGM movement,
- integrates IOB as context,
- compensates compression lows,
- improves trajectory confidence,
- stabilizes AutoDrive triggers,
- evaluates sensor quality.

---

## MTR guidance
MTR strongly suggested:
> non-G6 users should keep Adaptive Smoothing enabled.

Especially relevant for:
- Libre2
- noisy sensors
- compression lows
- rapid glucose changes

---

## UKF interpretation
UKF = Unscented Kalman Filter.

AIMI already used UKF/Kalman-like ideas internally before newer standalone UKF plugins.

---

UKF = Unscented Kalman Filter.

AIMI already used UKF/Kalman-like ideas internally before newer standalone UKF plugins.

MTR warning:
- replacing AIMI adaptive smoothing with standalone UKF smoothing may lose:
  - adaptive behaviour,
  - sensor quality logic,
  - IOB-context integration,
  - compression-low handling.

---

## R variable
Likely Kalman/UKF measurement-noise confidence.

Poor R estimation can:
- trust noisy Libre2 movement too much,
- destabilize trajectory interpretation,
- worsen overshoot behaviour.

---

# 6. DynISF / PKPD / Trajectory Systems
## Important new architecture evolution
Recent AIMI evolution strongly suggests movement away from:
- heavy manual PKPD tuning,
- isolated DIA/peak tweaking,
- low-level aggressiveness micromanagement.

And toward:
- trajectory-aware braking,
- physiology-aware modulation,
- predictive hyporisk prevention,
- confidence/relevance filtering,
- AI Auditor supervision,
- adaptive learning.

Interpretation:
> newer AIMI generations appear increasingly like metabolic-state engines rather than simple SMB calculators.

---

## DynISF trajectory tuning (CGM geometry)
This feature likely uses:
- short-horizon CGM geometry,
- acceleration,
- curve shape,
- parabolic projection,
- trajectory behaviour.

Goal:
> allow DynISF to react not only to glucose level but also to curve behaviour.

---

## Current user configuration
- trajectory tuning = enabled
- shadow only = ON
- max ISF change per tick = 0.060

---

## Shadow-only interpretation
Current behaviour:
- logs what the system WOULD do,
- without affecting actual insulin delivery.

Current recommendation:
> keep shadow-only enabled for now.

Reason:
- system already feels too aggressive,
- geometry tuning may tighten ISF further,
- trajectory systems are still evolving rapidly.

---

# 7. Libre2 + Lyumjev Behaviour
## Important conclusion
Libre2 + Lyumjev behaves differently from:
- slower insulins,
- lower-lag CGM systems.

---

## Observed risk pattern
1. glucose starts rising
2. AutoDrive escalates
3. Lyumjev begins acting rapidly
4. Libre2 still reports delayed rise
5. AIMI continues correction
6. real physiology already turning downward
7. overshoot / near-hypo occurs

---

## Important implications
This setup benefits from:
- less trigger nervousness,
- smoother adaptation,
- less persistent TBR,
- stronger braking,
- earlier trajectory skepticism.

---

# 8. Sport / Exercise Logic
## Older behaviour
Older manuals describe:
- reduced basal during sport,
- reduced SMB,
- increased intervals,
- softer activity modulation.

Examples:
- light cardio: −15% to −25%
- moderate: −30% to −40%
- intense: −50% to −60%

This also matched older real-world behaviour.

---

Older manuals describe:
- reduced basal during sport.

Examples:
- light cardio: −15% to −25%
- moderate: −30% to −40%
- intense: −50% to −60%

---

## NEW HARD-CONFIRMED ACTIVITY LOCKOUT BEHAVIOUR
Current runtime/code behaviour is materially different.

Recent repo evolution introduced:
- exercise insulin lockout,
- stronger hypo-first safety,
- explicit SMB suppression during activity,
- basal blocking under activity conditions.

---

## Activity contexts collapse into same pipeline
Current architecture appears to route:
- walking
- cardio
- yoga
- strength
- sport

into the same underlying Activity pipeline.

Differences appear mainly:
- intensity,
- duration,
- metadata.

NOT fundamentally different insulin logic.

Practical implication:
> walking low is NOT a uniquely mild reduction mode.

---

## Explicit SMB lockout
Current runtime/code behaviour explicitly sets:

```kotlin
maxSMB = 0.0
maxSMBHB = 0.0
```

when:
- sport/activity context
- exercise lockout

is active.

Interpretation:
> SMB suppression during activity is explicit runtime behaviour.

---

## Explicit basal + SMB stop under BG<=220
Current runtime/code behaviour includes:

```kotlin
if (exerciseInsulinLockoutActive && bg <= 220)
```

Observed runtime log:

```text
Sport / contexte AIMI activité : basale & SMB arrêtés (BG<=220)
```

Associated behaviour:

```kotlin
setTempBasal(0.0, ...)
```

Meaning:
- basal = 0
- SMB = 0

while activity lockout is active and BG<=220.

---

## Important nuance
The older ContextInfluenceEngine reduction layer still exists:
- SMB ×0.92
- SMB ×0.85
- SMB ×0.75
- SMB ×0.60
- preferBasal=true

However:
> later exercise lockout overrides practical runtime behaviour.

This explains why:
- older manuals,
- older runtime behaviour,
- and current runtime behaviour

now appear contradictory.

---

## Practical finding for Walter
Using sport/walking context during short post-meal walks:
- caused excessive rises,
- because meal control was effectively interrupted.

Current recommendation:
> do NOT routinely use sport/activity context for short post-meal walks.

Reason:
- current activity lockout appears too aggressive for this use case.

---

# 9. Physio Context
Observed current state:
- state = OPTIMAL
- reduceBasal = false
- reduceSMB = false

Interpretation:
- AIMI currently believes physiology is normal/safe.
- no automatic damping is active.

Possible limitation:
> post-meal walking may not be recognized early enough.

---

# 10. Confirmed AAPS Log Findings
## Time window analysed
- 2026-05-19 to 2026-05-20

## Observed
- repeated AutoDrive V3 activity
- repeated temp basal at 5.5 U/h
- repeated Basal×2.15 scaling
- SMB often reduced by PKPD Guard
- later LGS/Safety Halt after strong declines

---

- 156 AIMI APSResult entries
- repeated AutoDrive V3 activity
- repeated temp basal at 5.5 U/h
- repeated Basal×2.15 scaling
- SMB often reduced by PKPD Guard
- later LGS/Safety Halt after strong declines

---

## Important interpretation
AIMI DOES brake eventually.

The issue appears to be:
> braking occurs after significant insulin exposure already accumulated.

---

# 11. Upgrade Interpretation
## Why moving to newer AIMI version appears logical
Recent repo evolution introduced:
- confirmed-rise filtering,
- trajectory relevance,
- AutoDrive throttling,
- periodic deduplication,
- adaptive smoothing improvements,
- T3C trajectory braking,
- MPC hyporisk prevention,
- physiological gating,
- AI Auditor evolution.

Interpretation:
> the newer AIMI direction aligns closely with the suspected Libre2 + Lyumjev overshoot failure mode.

---

## Important nuance
The newer version should NOT automatically be assumed universally better.

However:
- the architecture direction appears specifically relevant for this setup.

---

# 12. Current Working Recommendations
## Keep
- CombinedDelta = 2
- Adaptive smoothing ON
- trajectory tuning shadow-only
- AutoDrive max basal = 4.5 U/h

## Avoid for now
- walking/activity context for short post-meal walks
- live trajectory tuning
- aggressive PKPD tweaking
- drastic SMB reduction
- drastic MaxIOB reduction

---

# 13. Confidence Classification
## Strongly supported by logs/settings/code
- sustained TBR importance
- Basal×2.15 behaviour
- CombinedDelta improvement
- PKPD Guard SMB braking
- frequent 5.5 U/h temp basal
- AutoDrive V3 persistence
- explicit SMB lockout during activity
- basal stop under BG<=220 during activity lockout
- activity-context collapse into same pipeline
- mismatch between older manuals and current runtime behaviour

## Behaviour-inferred but plausible
- trajectory tunnel relationship
- adaptive basal emergent scaling
- Libre2 lag contribution magnitude
- timing mismatch between activity and AutoDrive

## Needs future validation
- exact order of all safety layers
- exact interaction between:
  - ActivityManager
  - Auditor
  - PhysioModulation
  - exercise insulin lockout
  - adaptive basal systems
  - trajectory braking

---

# 14. Future Update Strategy
## Recommended workflow
### Chat phase
Use chat for:
- hypotheses
- experiments
- settings changes
- log interpretation
- repo changes

### KB phase
Only periodically:
- integrate stable findings,
- preserve historical conclusions,
- update confidence classifications,
- merge new repo-direction interpretations.

---

# 15. Final Consolidated Interpretation
The newer AIMI generation appears increasingly optimized around:
- trajectory confidence,
- physiology-aware braking,
- hyporisk prediction,
- false-rise filtering,
- adaptive safety.

This direction appears highly relevant for:
- Libre2 lag,
- Lyumjev rapid action,
- post-meal walking overshoot,
- delayed braking patterns.

However:
- fresh logs after upgrade must become the new source of truth.

Older assumptions about:
- AutoDrive,
- adaptive basal scaling,
- trajectory behaviour,
- exercise handling,
- PKPD dynamics

may no longer fully apply unchanged.

# END
- curve shape,
- parabolic projection,
- trajectory behaviour.

Goal:
> allow DynISF to react not only to glucose level,
but also to curve behaviour.

---

## Current user configuration
- trajectory tuning = enabled
- shadow only = ON
- max ISF change per tick = 0.060

---

## max ISF change per tick
Likely meaning:
> maximum trajectory-driven DynISF adaptation per loop.

Interpretation:
- approximately ±6% adjustment per loop.

This likely corresponds closely to what MTR called:
- “trajectory tunnel”.

---

## Shadow-only interpretation
Current behaviour:
- logs what the system WOULD do,
- without affecting actual insulin delivery.

Current recommendation:
> keep shadow-only enabled for now.

Reason:
- system already feels too aggressive,
- geometry tuning may tighten ISF further,
- trajectory systems are still evolving rapidly.

---

## Newer behaviour
Recent AIMI evolution appears to include:
- SMB suppression during sport,
- stronger insulin lockout,
- basal restriction,
- possible blocking while BG <= 220 mg/dL.

Interpretation:
> newer philosophy prefers hypo prevention over aggressive meal control.

---

## Practical finding for this setup
Using sport mode/context during short post-meal walks:
- caused excessive rises.

Interpretation:
- current sport lockout appears too strong for light post-meal walking.

Current recommendation:
> do NOT use sport mode routinely for short post-meal walks.

Reason:
- the real problem seems to be late insulin persistence,
not lack of early meal control.

---

# 11. Current Working Recommendations
## Keep
- CombinedDelta = 2
- Adaptive smoothing ON
- trajectory tuning shadow-only

## Current active test
- AutoDrive max basal:
  - 5.5 -> 4.5

Observe for:
- fewer near-hypo events
- fewer Safety Halt events
- less steep negative deltas
- less need to eat preventively
- acceptable meal peaks

---

## Do NOT currently change
- MaxIOB
- profile basal globally
- ISF profile
- DIA
- Unified Reactivity
- live trajectory tuning

---

# 12. Confidence Classification
## Strongly supported by logs/settings
- sustained TBR importance
- Basal×2.15 behaviour
- CombinedDelta improvement
- PKPD Guard SMB braking
- frequent 5.5 U/h temp basal
- AutoDrive V3 persistence

## Behaviour-inferred but plausible
- trajectory tunnel relationship
- adaptive basal emergent scaling
- Libre2 lag contribution magnitude
- timing mismatch between activity and AutoDrive

## Needs future code verification
- exact internal composition of Basal×2.15
- exact trajectory geometry weighting
- exact UKF adaptive implementation
- full exercise lockout conditions

---

# 13. Future Update Strategy
## Recommended workflow
### Chat phase
Use chat for:
- hypotheses
- experiments
- settings changes
- log interpretation
- repo changes

### Knowledge base updates
Only periodically:
- integrate stable findings,
- summarize insights,
- remove repetition,
- update commit-aware conclusions.

---

# 14. Intended Use In Future Chats
This document is intended to serve as:
- reusable AIMI context,
- troubleshooting baseline,
- historical knowledge map,
- settings behaviour reference.

Important:
- future chats can use this document as starting context,
- but new repo commits and new logs may supersede older conclusions.

---

This document is intended to serve as:
- reusable AIMI context,
- troubleshooting baseline,
- historical knowledge map,
- settings behaviour reference.

Important:
- future chats can use this document as starting context,
- but new repo commits and new logs may supersede older conclusions.

---


---

# 15. Latest Repo/Upgrade Decision Update
## Date added
- 2026-05-22

## User decision
Walter decided to move to the newest available AIMI build after discussion of recent repo changes.

## Reason for considering the newest version
The current working problem is still:
> recurrent hypos / near-hypos, probably related to delayed braking and cumulative insulin exposure rather than pure SMB size.

The newer repo direction appears relevant because recent commits show work on:
- confirmed-rise logic,
- PKPD absorption guard behaviour,
- MPC 60/120/180 min hyporisk prevention,
- trajectory relevance scoring,
- physiological gating / PhysioModulation,
- AI Auditor decision handling,
- AutoDrive state / decision refactoring,
- adaptive basal scaling changes,
- expanded physiological data collection,
- T3C / brittle-mode neural learning.

Interpretation:
> the codebase is moving further toward trajectory-aware braking, physiological filtering, and explicit hyporisk prevention, which matches the suspected failure mode in Libre2 + Lyumjev + AutoDrive.

## Important repo observations from 2026-03 commits
### Confirmed-rise / PKPD guard / MPC hyporisk
A commit on 2026-03-18 introduced `isConfirmedRise` to modify reactivity learner adjustments and PKPD absorption guard behaviour, plus MPC 60/120/180 minute hyporisk prevention.

Practical interpretation:
- AIMI may now require stronger confirmation before treating a rise as actionable.
- PKPD guard behaviour may be more context-aware.
- Hypo prevention may look further ahead than older logic.

Relevance for Walter:
> potentially helpful for Libre2 lag + Lyumjev overshoot patterns.

### Trajectory relevance and PhysioModulation
Commits on 2026-03-13 added trajectory relevance scoring, dashboard display of trajectory insights, and renamed/refactored trajectory modulation into PhysioModulation with physiological gating.

Practical interpretation:
- trajectory modulation should be filtered by relevance/confidence,
- noisy or weakly relevant trajectory signals may have less effect,
- physiology context is becoming more explicit in decision modulation.

Relevance:
> potentially helpful for Libre2 noise/lag and post-meal walking situations.

### Auditor / AutoDrive decision refactoring
Commits introduced explicit decision result handling, Auditor UI state validation, and refactored AutoDrive to use newer decision models/state management.

Practical interpretation:
- decisions may now be represented more explicitly as approved/reduced/rejected/skipped/cancelled states,
- this may make safety behaviour and logging clearer,
- AutoDrive internals may behave differently from the older observed AutoDrive V3 behaviour.

Relevance:
> important to re-check old assumptions after upgrade.

### Adaptive basal scaling
A commit on 2026-03-17 applied adaptive basal scaling universally after boost calculation.

Practical interpretation:
- adaptive basal behaviour may have changed materially.
- The earlier finding that sustained basal exposure was central remains important, but the exact mechanism may no longer be identical.

Relevance:
> after upgrade, old conclusions about `Basal×2.15` and sustained TBR must be revalidated from fresh logs.

### T3C / brittle neural learning
Several commits added or expanded T3C neural learner and brittle-mode training/aggressiveness.

Practical interpretation:
- newer AIMI may learn more from physiological/performance data,
- additional learner state may influence aggressiveness over time.

Relevance:
> avoid blindly carrying old learner state if behaviour is unstable after upgrade.

### T3C / T3C Brittle – korte samenvatting

**Wat is het?**

Type 3c diabetes (also known as **pancreatogenic diabetes**) is a form of diabetes caused by structural damage or disease of the pancreas, rather than autoimmune destruction (Type 1) or insulin resistance (Type 2).

### Key Characteristics

* **Underlying Causes:** Typically triggered by conditions that damage the pancreas, such as chronic pancreatitis, pancreatic cancer, pancreatic surgery, cystic fibrosis, or hemochromatosis.
* **Dual Failure:** Because the damaged pancreas cannot function properly, patients usually suffer from a lack of both **insulin** (leading to high blood sugar) and **digestive enzymes** (leading to malabsorption and nutritional deficiencies).
* **"Brittle" and Unstable:** T3c diabetes is notoriously difficult to manage. It is frequently referred to as "brittle diabetes" **because pients experience extreme, unpredictable, and rapid day-to-day or hour-to-hour fluctuations between severe high and dangerous low blood sugar levels.**
* **Lack of Counter-Regulation:** Unlike other forms of diabetes, patients often lack *glucagon* (the hormone that raises blood sugar when it drops), making sudden hypoglycemic episodes harder for the body to correct on its own. 

* Een extra **veiligheidsrem** voor mensen die zeer gevoelig zijn voor insuline of snel hypo's krijgen.
* AIMI wordt daardoor **voorzichtiger** met insuline.

**Effect volgens handleidingen/KB**

* Lagere effectieve IOB-limieten.
* Tragere verhoging van basal/TBR.
* Minder agressieve correcties bij stijgende glucose.
* Minder kans op overshoot → minder hypo's.

**Nadeel**

* Hoge glucosewaarden kunnen langer blijven hangen.
* AIMI kan "te braaf" worden.

**Voor jouw setup (Libre 2 + Lyumjev + AutoDrive V3)**

* In je KB staat expliciet: **"Désactivé t3c"** (uitgezet).
* In de huidige runtime-context staat T3C echter **aan**.
* De huidige werkhypothese in je KB is dat je probleem eerder kwam door:

  * AutoDrive V3,
  * langdurige hoge TBR's,
  * Libre2-lag,
  * Lyumjev + wandelen,

  en **niet** door te agressieve SMB's alleen.

In de huidige runtime-interpretatie lijkt T3C vooral een extra veiligheidsrem te zijn die AIMI meer conservatief maakt wanneer de beslisboom naar basaal-first, TBR-only of strengere hypo-preventie neigt. De relevante code- en log-signalen suggereren dat T3C in deze build eerder een richtinggevende beschermlaag is dan de primaire oorzaak van het probleem. Voor Walter's setup betekent dat: T3C is niet de eerste knob om te draaien als het doel is om overcorrecties en late dalingen te verminderen; eerst verdienen de meer directe stuurparameters zoals CombinedDelta, AutoDrive max basal en Adaptive Smoothing prioriteit. T3C zou pas als vervolgstap worden meegenomen als de basisinstellingen nog steeds leiden tot structurele overcorrectie of herhaalde late hypo's.

### Mijn huidige advies

**T3C uit laten.**

Eerst optimaliseren via:

* CombinedDelta,
* AutoDrive max basal,
* Adaptive Smoothing,
* eventuele toekomstige PKPD/Trajectory-tuning.

T3C zou ik pas testen als je ondanks die aanpassingen nog steeds structureel hypo's krijgt door overcorrecties. Dat maakt AIMI veiliger, maar waarschijnlijk ook merkbaar minder effectief tegen hoge waarden.

---

# 16. Upgrade Safety Plan
## Before upgrade
Recommended:
- export full preferences,
- save screenshots or notes of:
  - AutoDrive settings,
  - PKPD settings,
  - CombinedDelta,
  - Adaptive smoothing,
  - trajectory tuning / shadow mode,
  - max SMB,
  - max IOB,
  - AutoDrive max basal,
  - meal mode factors,
  - reactivity / unified reactivity,
  - basal caps.

## Settings to keep initially after upgrade
- CombinedDelta = 2
- Adaptive smoothing = ON
- DynISF trajectory tuning = shadow only
- AutoDrive max basal = 4.5 U/h for now
- current conservative PKPD test:
  - PKPD initial peak changed from 40 to 50 min

## What not to change simultaneously
Do not also change:
- profile basal,
- ISF profile,
- MaxIOB,
- global reactivity,
- meal mode factors,
- live trajectory tuning,
- sport/activity strategy.

Reason:
> the upgrade itself is already a major variable.

## Learner / state caution
Because recent repo changes affect AutoDrive, PKPD guard, trajectory, physiology, adaptive basal, and neural learning, old learned states may not transfer cleanly.

Potentially consider resetting or closely monitoring:
- PKPD learned state,
- AutoDrive/adaptive learner state,
- trajectory learner state,
- old CSV learner files.

Conservative approach:
> carry preferences, but be cautious with old adaptive/ML state if first 24-72h behaviour is strange.

---

# 17. Post-Upgrade Monitoring Checklist
For the first 2-3 days after upgrade, specifically check logs for:

## Hypo prevention / braking
- Safety Halt / LGS frequency
- predicted hypo / hyporisk messages
- MPC 60/120/180 min references if logged
- whether braking begins earlier than before

## AutoDrive
- AutoDrive state names / V3 markers
- confirmed rise markers
- skipped/cancelled/reduced decisions
- whether repeated glucose-driven and periodic decisions are deduplicated or throttled

## PKPD
- learned DIA
- learned peak
- PKPD absorption guard reductions
- whether peak remains around 70-80 min or shifts after initial peak = 50

## Adaptive basal
- maximum TBR reached
- duration of high TBR
- any `Basal×...` / unified scaling markers
- whether 4.5 U/h cap is respected
- whether sustained basal exposure is reduced

## Activity / physiology
- PhysioModulation or physiological gate markers
- whether post-meal walks are recognized
- whether SMB/basal are reduced too much during light walking

## Practical outcome metrics
- fewer late steep drops
- fewer preventive carbs
- fewer LGS/Safety Halt events
- acceptable meal peaks
- no prolonged high rebounds due to excessive braking

---

# 18. Current Interpretation After Upgrade Decision
The previous core hypothesis still stands until new logs prove otherwise:

> In Walter's setup, the main problem is likely cumulative insulin exposure and delayed braking, amplified by Libre2 lag and Lyumjev/walking dynamics.

However, after moving to the newest version:
- AutoDrive internals may differ,
- adaptive basal scaling may differ,
- trajectory relevance may filter decisions differently,
- PKPD guard may behave differently,
- hyporisk prediction may be more forward-looking.

Therefore:
> old log interpretations remain useful background, but fresh logs from the new build should become the new source of truth.

---

# 19. Next Recommended Analysis
After running the new version, collect one or more log windows containing:
- one breakfast/lunch with post-meal walk,
- one dinner/activity window,
- one hypo or near-hypo event if it occurs,
- one stable period without meal/activity.

Key question for next analysis:
> Does the new version reduce sustained basal exposure and brake earlier, or do hypos still occur despite improved trajectory/PKPD safety?

If hypos persist, next likely tuning target remains:
- AutoDrive/adaptive basal persistence,
not first-line global SMB reduction.

---

# 20. Hormonitor Study Framework

## What is Hormonitor?
Hormonitor is a clinical study and data export framework within AIMI (OpenAPS-AAPS) that provides a structured, human-readable narrative of the body's physiological state and therapeutic decisions.

**Schema evolution:** `1.1.0` → `1.2.0` → `1.3.0` (additive)
**Export file:** `Documents/AAPS/AIMI_HORMONITOR_event_stream_v1.jsonl`

## Core Architecture
Hormonitor represents a paradigm shift from "dose-first" to "body-state-first" loop:

- **Recursive Belief Tree (RBT)** — one nested tree unfolded every tick; existing engines become sensors (leaves), not silent competing deciders
- **Physiological state before action** — phase, meal absorption, UAM hypotheses, latent physio probabilities, and patient mode are evaluated **before** SMB/TBR channels are resolved
- **Patient story in Hormonitor** — human-readable `patient_story` plus live wearable context, so study replay can follow *why* AIMI chose a posture, not only *how much* insulin was delivered

## Key Data Structures

### PatientStateSnapshot
Contains physiological state including:
- Physiological phase (dawn, fast, post-hypo, etc.)
- Meal absorption phase and belief
- Transient resistance probability
- Sleep debt score
- UAM (Unannounced Meal) hypothesis state
- Thermal inflammation index and recovery burden

### PhysioLiveDigest
Live wearable/phone physio signals including:
- Steps (15m/60m), heart rate, HRV
- Activity state, sleep debt
- Thermal hypothesis and narrative

## Patient Story (schema 1.2.0)
The `patient_story` block includes:
- `patient_mode` — Dominant clinical reading (e.g., DAWN_ENDOGENOUS, STRESS_RESISTANCE)
- `patient_strategy_hint` — Therapeutic posture (BASAL_BRIDGE, SMB_PRIORITY, etc.)
- `patient_narrative` — Plain-language summary
- `patient_reason_codes` — Traceable codes linking to latent/UAM/pattern/context logic
- `physio_live` — Wearable snapshot at export time
- `thermal_belief` — Structured thermal hypothesis (schema 1.3.0)

## Thermal Belief Layer (schema 1.3.0)
- Uses temperature evolution vs personal baseline (not absolute fever)
- Sources: Garmin/Oura skin temperature → Health Connect → Oura API
- Noise floor: 0.03°C delta, 0.01°C/h slope
- Feeds into latent resistance/recovery, patient mode, UI, and Hormonitor — upstream of RBT and dose channels

## Study Hypotheses Enabled
1. Reduced false meal aggression at dawn
2. Post-hypo protection coherence
3. Activity without dose trigger
4. Tree tension vs outcome correlation
5. Thermal vs false illness detection
6. Cycle BBT (body basal temperature) analysis

## Relationship to Existing KB
Hormonitor provides the **observability and audit layer** that makes AIMI's decision-making process transparent and analyzable for clinical research purposes. It complements the existing knowledge base by:
- Exporting the "why" behind decisions, not just the "what"
- Enabling correlation between physiological state and insulin outcomes
- Providing structured data for validating the hypotheses documented in this KB

**Source:** `docs/AIMI_HORMONITOR_STUDY_NOTES_2026-06-06.md`
**Code:** `plugins/aps/src/main/kotlin/app/aaps/plugins/aps/openAPSAIMI/patient/PatientStateSnapshot.kt`

---

# 21. AIMI Context System - Code Analysis

## Context Types (from ContextIntent.kt)

The AIMI Context system supports the following context types that affect insulin dosing:

| Context Type | Icon | Purpose | Effect on Insulin |
|--------------|------|---------|-------------------|
| **Activity** | 🏃 | Exercise (cardio, strength, yoga, sports, walking) | ⬇️ -30% to -60% SMB (hypo risk) |
| **Illness** | 🤒 | Sickness, infection, fever | ⬆️ +20% to +50% (resistance) |
| **Stress** | 😰 | Emotional/work/exam stress | ⬆️ +10% to +30% (cortisol) |
| **Alcohol** | 🍷 | Alcohol consumption | ⬇️⬆️ Complex (initial drop, then delayed rise) |
| **UnannouncedMealRisk** | 🍕 | Risk of surprise meals/snacking | Stay reactive with safety margins |
| **SlowCarbMeal** | 🧀 | Fat/protein-rich meals (pizza, cheese, etc.) | Capped early SMB, damped late coverage |
| **HypoRecovery** | 🍬 | Declared hypo/recovery window | Hard SMB-off, prefer basal |
| **MenstrualCycle** | 🔄 | Menstrual cycle phase | Variable (luteal = +10-20% resistance) |
| **Travel** | ✈️ | Timezone changes, travel stress | Conservative mode |
| **Custom** | 📝 | Generic/unrecognized contexts | User-defined |

## Pump Site Change - NOT an AIMI Context

**Finding:** "Pump Site Change" is **NOT** an AIMI context type.

- The string `careportal_pump_site_change` in `core/ui/src/main/res/values/strings.xml` (line 364) is a **Careportal entry type** for logging infusion site changes.
- This is a **record-keeping feature**, not an insulin-dosing context.
- The string exists in the UI strings but there is **no corresponding ContextIntent** in the codebase.

## Activity Context BG Thresholds (from ContextInfluenceEngine.kt)

The current code shows:
- Activity context reduces SMB by 8-40% depending on intensity
- Sets `preferBasal = true` for MEDIUM/HIGH/EXTREME intensity
- Extra caution when `currentBG < 90` or `currentBG < 110 && iob > 2.0`

**Note:** The WVL_DISCOVERIES.md previously mentioned "possible basal stop under BG<=220" and "explicit SMB lockout during activity" as **observed runtime behavior**. This may be:
1. Older code that has since been refactored
2. A different module (Autodrive, Physio, or Safety)
3. Runtime behavior that emerges from the combination of multiple systems

## Confidence Classification

- **Confirmed code behaviour**: ContextIntent.kt, ContextInfluenceEngine.kt contain the actual context types
- **Runtime/log-confirmed behaviour**: The BG 220 threshold for activity may be from older versions or combined system effects
- **Requires future validation**: Exact interaction between Activity context and basal/TBR during exercise

---

# 21. AIMI Context System - Code Analysis

## Context Types (from ContextIntent.kt)

The AIMI Context system supports the following context types that affect insulin dosing:

| Context Type | Icon | Purpose | Effect on Insulin |
|--------------|------|---------|-------------------|
| **Activity** | 🏃 | Exercise (cardio, strength, yoga, sports, walking) | ⬇️ -30% to -60% SMB (hypo risk) |
| **Illness** | 🤒 | Sickness, infection, fever | ⬆️ +20% to +50% (resistance) |
| **Stress** | 😰 | Emotional/work/exam stress | ⬆️ +10% to +30% (cortisol) |
| **Alcohol** | 🍷 | Alcohol consumption | ⬇️⬆️ Complex (initial drop, then delayed rise) |
| **UnannouncedMealRisk** | 🍕 | Risk of surprise meals/snacking | Stay reactive with safety margins |
| **SlowCarbMeal** | 🧀 | Fat/protein-rich meals (pizza, cheese, etc.) | Capped early SMB, damped late coverage |
| **HypoRecovery** | 🍬 | Declared hypo/recovery window | Hard SMB-off, prefer basal |
| **MenstrualCycle** | 🔄 | Menstrual cycle phase | Variable (luteal = +10-20% resistance) |
| **Travel** | ✈️ | Timezone changes, travel stress | Conservative mode |
| **Custom** | 📝 | Generic/unrecognized contexts | User-defined |

## Pump Site Change - NOT an AIMI Context

**Finding:** "Pump Site Change" is **NOT** an AIMI context type.

- The string `careportal_pump_site_change` in `core/ui/src/main/res/values/strings.xml` (line 364) is a **Careportal entry type** for logging infusion site changes.
- This is a **record-keeping feature**, not an insulin-dosing context.
- The string exists in the UI strings but there is **no corresponding ContextIntent** in the codebase.

## Activity Context BG Thresholds (from ContextInfluenceEngine.kt)

The current code shows:
- Activity context reduces SMB by 8-40% depending on intensity
- Sets `preferBasal = true` for MEDIUM/HIGH/EXTREME intensity
- Extra caution when `currentBG < 90` or `currentBG < 110 && iob > 2.0`

**Note:** The WVL_DISCOVERIES.md previously mentioned "possible basal stop under BG<=220" and "explicit SMB lockout during activity" as **observed runtime behavior**. This may be:
1. Older code that has since been refactored
2. A different module (Autodrive, Physio, or Safety)
3. Runtime behavior that emerges from the combination of multiple systems

## Confidence Classification

- **Confirmed code behaviour**: ContextIntent.kt, ContextInfluenceEngine.kt contain the actual context types
- **Runtime/log-confirmed behaviour**: The BG 220 threshold for activity may be from older versions or combined system effects
- **Requires future validation**: Exact interaction between Activity context and basal/TBR during exercise
