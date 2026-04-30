# Improvement Research: Backtrace Extension

Research into potential improvements for the backtrace skill, drawing from
computer science, safety engineering, quality management, argumentation
theory, and LLM agent design. Conducted 2026-04-30.

## Methodology

Three research agents investigated techniques from different domains in
parallel. An adversarial auditor then challenged every recommendation
against the project's constraints: markdown-based skill, specs under 2000
lines, one user at v0.1.0, constitution that prioritizes correctness over
recall. A separate reference auditor validated all citations.

## Accepted Improvements

### I-001: Barrier Enumeration Step

**Source discipline**: Root cause analysis, nuclear safety (DOE barrier
analysis, defense-in-depth).

**Technique**: Before freeform tracing, walk the spec's defense-in-depth
stack top-down for each finding: Is there a relevant FR? Does it have
adequate ACs? Is there a corresponding SC? Is there a task? Identify the
first missing layer.

**Problem it solves**: The current Section 6 describes what to trace to
(FR, AC, SC) but not how to decide which level is the right target. The
LLM may latch onto the first plausible AC and propose an amendment when
the real gap is a missing FR upstream. An AC cannot exist without a FR to
hang from.

**Implementation**: 5-10 lines added to Section 6, integrated into the
existing category logic as a structured checklist. Not a separate
preliminary step.

**Constitution alignment**: Directly supports Principle I (correctness
over coverage). Misplacing an addition produces an incorrect trace even if
the content is reasonable.

**Auditor verdict**: Accepted. Called "the strongest recommendation in the
set."

**References**:
- Barrier Analysis as an RCA technique: [R-17]
- Defence-in-depth concept: [R-18]

---

### I-002: Categorical Trace Certainty

**Source discipline**: LLM confidence calibration research, adapted to
avoid unreliable numeric scores.

**Technique**: Add `trace_certainty: direct | plausible | uncertain` to
ProposedAdditions.

- `direct`: finding explicitly names the spec item
- `plausible`: finding relates to the spec item's domain
- `uncertain`: inferential trace, could map to multiple items

**Problem it solves**: All proposed additions are currently presented with
equal confidence. Some traces are obvious, others are inferential. The
auditor and developer have no signal for where to focus scrutiny.

**Implementation**: One field added to ProposedAddition, ~5 lines of
instruction in Section 6, one sentence in auditor prompt telling it to
weight uncertain traces with more skepticism.

**Constitution alignment**: Directly implements Principle VII ("Flag
Uncertainty"). Distinguishes certain from uncertain claims.

**Auditor verdict**: Accepted. Low cost, directly supports constitution.

**Why not numeric scores**: LLM self-assessed numeric confidence is
unreliable. Expected Calibration Errors range from 0.24 to 0.47 in
controlled studies [R-33]. Categorical markers avoid this problem while
still surfacing ambiguity.

**References**:
- LLM confidence unreliability: [R-33]
- Self-consistency vs. verbal confidence: general finding across multiple
  calibration studies (no single authoritative source identified; see
  notes on [R-34])

---

### I-003: Cluster Mention in Output

**Source discipline**: Pareto analysis (quality management, Six Sigma).

**Technique**: Add one sentence to Section 11: "If three or more applied
additions target the same spec item, note this as a potential structural
issue worth manual review."

**Problem it solves**: When multiple findings trace to the same FR, that
FR likely needs structural rework rather than incremental amendments. The
current sequential output does not surface this pattern.

**Implementation**: One sentence in Section 11. No schema changes, no
threshold logic, no new output format.

**Why simplified from full Pareto**: At typical finding counts (5-15),
clustering is trivially visible to the developer. Full Pareto analysis
with grouping, counting, and threshold logic is over-engineered for the
volume. The auditor recommended the simplified version.

**Auditor verdict**: Accepted in simplified form.

**References**:
- Pareto Analysis (80/20 rule): [R-23]
- 5 Whys for deeper tracing: [R-25]

## Explored but Not Adopted

### STPA-Derived Tracing Categories

**Source discipline**: STAMP/STPA (Systems-Theoretic Accident Model and
Processes), created by Nancy Leveson at MIT for safety-critical systems.

**Technique**: Replace the current 4 tracing categories with STPA-inspired
categories based on Unsafe Control Action types:

1. **Constraint not specified** (UCA type: not provided): The spec never
   stated a requirement that would prevent this behavior. Maps to
   proposing new FRs or ACs.

2. **Constraint specified but insufficient** (UCA type: provided, causes
   hazard): An existing FR/AC exists but its wording permits the bad
   behavior. Maps to amendments.

3. **Constraint specified for wrong phase** (UCA type: too early/too
   late): The requirement exists but is placed in the wrong artifact or
   lifecycle stage. Maps to moving or splitting requirements.

4. **Constraint scope too narrow or too broad** (UCA type: stopped too
   soon/applied too long): An AC covers some boundary conditions but stops
   short, or an FR applies too broadly. Maps to scoping amendments.

**Why not adopted**: The auditor identified that these categories are
orthogonal to, not replacements for, the existing categories. The current
categories describe what kind of gap was found (missing test, implicit
assumption). STPA categories describe what kind of spec deficiency caused
the gap. These are two different classification axes. A "missing test
coverage" finding could be "constraint not specified" or "constraint
specified but insufficient."

Additionally, replacing the current categories would break conceptual
continuity with gap-audit's own classification scheme.

**Path forward**: Consider as a second classification dimension (gap-type
+ constraint-status) rather than a replacement. Revisit when current
categories prove insufficient through observed misclassifications.

**References**:
- STPA method and four UCA types: [R-15], [R-16]

---

### Toulmin-Structured Rationales

**Source discipline**: Argumentation theory (Stephen Toulmin, 1958). Used
in legal reasoning and rhetoric.

**Technique**: Replace free-text `rationale` field in ProposedAddition
with structured argument: claim, grounds, warrant, qualifier, rebuttal.

**Why not adopted**: The LLM generating the rationale is the same LLM that
would structure it as Toulmin. Structure does not create reasoning quality;
it reflects it. Five subfields per addition also significantly increases
output volume sent to the auditor.

**Path forward**: Lighter alternative: require rationale to include one
sentence explaining why no existing spec item covers this (the "rebuttal"
equivalent). Captures most of Toulmin's value with minimal complexity.

**References**:
- Toulmin's model of argumentation: [R-26]

---

### Latent Condition Scan (Swiss Cheese Model)

**Source discipline**: Human factors, cognitive science (James Reason's
Swiss Cheese Model, HFACS).

**Technique**: Check for absent spec sections (no error handling section,
no concurrency model) that create latent conditions for gaps. Propose new
sections for structural absences rather than individual ACs.

**Why not adopted**: The auditor identified this as gap-audit's job, not
backtrace's. Adding gap-finding capability to backtrace would violate
Constitution Principle IV (One Code Path Per Operation) and create role
confusion between the two skills.

**Path forward**: Propose as a new checklist category in gap-audit.

**References**:
- Swiss Cheese Model (active vs latent failures): [R-21]
- HFACS four-level taxonomy: [R-22]

---

### Lateral Dependency Tracing

**Source discipline**: Requirements traceability (ontology-based lateral
tracing, bidirectional traceability matrices).

**Technique**: Build a dependency graph between spec items during artifact
loading. When a finding traces to FR-003, check if FR-007 depends on
FR-003 and might need amendment too.

**Why not adopted**: "Building a dependency graph" is software engineering
advice for a codebase, not for a markdown prompt. There is no data
structure to build. The LLM would need to construct an explicit graph in
prose, which is expensive and unreliable. The auditor's existing conflict
check (Verification Checklist item 2) partially covers cascade effects.

**References**:
- Bidirectional traceability: [R-01]
- Ontology-based lateral traceability: [R-02]
- OntoTraceV2.0 automated trace link discovery: [R-03]

---

### Bounded Iterative Refinement (CEGAR-Inspired)

**Source discipline**: Formal verification (Counterexample-Guided
Abstraction Refinement), LLM self-improvement research.

**Technique**: Allow 1-2 refinement passes for "revise" verdicts. After
auditor returns revisions, re-trace those specific additions with feedback
as context.

**Why not adopted**: Directly contradicts FR-010 ("One pass only"). The
"revise" verdict already incorporates one round of refinement. A second
pass is unlikely to help: if the first trace was wrong, the same context
produces the same error. Token cost doubles or triples per refinement
pass. Constitution Principle I warns against trading precision for recall.

**References**:
- CEGAR: [R-09]
- SELF-REFINE (~20% improvement, steep diminishing returns): [R-36]
- RISE recursive introspection: [R-37]
- NL explanation verification with theorem provers: [R-10]

---

### Dual Specialized Auditors

**Source discipline**: Multi-agent debate and verification in LLM systems.

**Technique**: Replace single adversarial auditor with two parallel
auditors: "completeness" (does this close the gap?) and "consistency"
(does this conflict with existing content?).

**Why not adopted**: Doubles token cost and latency. The current single
auditor's verification checklist already covers both dimensions. The ICLR
2025 scaling study found that homogeneous agents add noise, not signal.
The claimed 4-6% accuracy gain is from a different task domain (factual
accuracy in text generation) and likely does not transfer to structured
spec auditing.

**References**:
- A-HMAD heterogeneous multi-agent debate: [R-27]
- MAD scaling challenges (homogeneous agents underperform): [R-28]
- Agent diversity over agent count: [R-29]

---

### Focused Input Structuring (F-CoT)

**Source discipline**: Prompt engineering research.

**Technique**: Extract topically relevant spec sections before tracing
each finding, rather than presenting the full spec.

**Why not adopted**: Specs fit in context (under 2000 lines). Selective
extraction is itself a reasoning task that could miss cross-cutting
concerns. Premature optimization that trades precision for speed. Would
become relevant if specs routinely exceeded 10,000+ lines.

**References**:
- Focused Chain-of-Thought: [R-30]
- Wharton report on CoT diminishing value for reasoning models: [R-31]
- LCoT2Tree on optimal reasoning path complexity: [R-32]

---

### Spec Backward Slicing

**Source discipline**: Static analysis (program slicing).

**Technique**: Compute reachability from finding's target through a
dependency graph, scope tracing to the reachable set.

**Why not adopted**: Same fundamental problem as lateral dependency
tracing. There is no graph to slice through in a markdown prompt. Solves a
scaling problem that does not exist at current spec sizes.

**References**:
- Backward program slicing: [R-04]
- Path-sensitive backward slicing: [R-05]

---

### FTA Decomposition for Compound Findings

**Source discipline**: Fault Tree Analysis, FMEA.

**Technique**: Build fault trees with AND/OR gates for findings that span
multiple artifacts.

**Why not adopted**: Compound findings should be split by gap-audit
upstream (gap-audit Filter 4 requires splitting multi-root-cause findings).
The current skill already allows multiple additions per finding. LLMs
struggle to maintain consistent AND/OR gate semantics in multi-gate
structures.

**References**:
- Fault Tree Analysis: [R-06]
- FMEA vs FTA comparison: [R-07]
- Software FMEA cause categories: [R-08]

---

### RAG Over Spec

**Source discipline**: Retrieval-augmented generation.

**Why not adopted**: Requires infrastructure (embeddings, vector DB) that
does not exist in the Claude Code skill execution model. Specs fit in
context. No benefit over full-text inclusion at current document sizes.

---

### Numeric Confidence Scores

**Source discipline**: LLM calibration research.

**Why not adopted**: LLM self-assessed confidence is persistently
unreliable. Categorical trace certainty (I-002) was adopted instead.

**References**:
- LLM confidence calibration: [R-33]

---

### Bradford Hill Validation Criteria

**Source discipline**: Epidemiology, causal inference.

**Technique**: Apply specificity and temporality checks to each trace
claim before sending to auditor.

**Why not adopted**: Specificity check risks producing false negatives on
findings that legitimately trace to multiple spec items. Temporality
requires reasoning about artifact ordering that the LLM may get wrong.
Interesting as heuristics but risky as mandatory validation gates.

**References**:
- Bradford Hill viewpoints: [R-20]
- DAGs in causal inference: [R-19]

## Reference Table

All references were validated by an auditor agent on 2026-04-30. Each was
checked for URL accessibility, claim accuracy, and correct attribution.

### Verified (claim matches source)

| ID | Source | URL |
|----|--------|-----|
| R-01 | Traceability Matrix (Wikipedia) | https://en.wikipedia.org/wiki/Traceability_matrix |
| R-02 | Ontology-Based Improved Software Requirement Traceability Matrix (IEEE 2009) | https://ieeexplore.ieee.org/document/5362244/ |
| R-03 | OntoTraceV2.0 (Springer, 2025) | https://link.springer.com/article/10.1007/s00766-025-00447-4 |
| R-04 | Program Slicing (Wikipedia) | https://en.wikipedia.org/wiki/Program_slicing |
| R-05 | Path-Sensitive Backward Slicing (Jaffar et al., SAS 2012) | https://jorgenavas.github.io/papers/BackwardSlicing-SAS12.pdf |
| R-06 | Fault Tree Analysis (Wikipedia) | https://en.wikipedia.org/wiki/Fault_tree_analysis |
| R-07 | FMEA vs FTA (TWI) | https://www.twi-global.com/technical-knowledge/faqs/fmea-vs-fta |
| R-08 | Software FMEA (Accendo Reliability) | https://accendoreliability.com/understanding-software-fmea/ |
| R-09 | CEGAR (Wikipedia) | https://en.wikipedia.org/wiki/Counterexample-guided_abstraction_refinement |
| R-10 | Verification and Refinement of NL Explanations (Quan et al., EMNLP 2024) | https://aclanthology.org/2024.emnlp-main.172.pdf |
| R-12 | Fine-Grained TLR (Hey, KIT 2022) | https://publikationen.bibliothek.kit.edu/1000140404/134392428 |
| R-14 | LSA Tutorial (McCormick) | https://mccormickml.com/2016/03/25/lsa-for-text-classification-tutorial/ |
| R-20 | Bradford Hill Viewpoints (PMC) | https://pmc.ncbi.nlm.nih.gov/articles/PMC8206235/ |
| R-21 | Swiss Cheese Model, Active vs Latent Failures (PMC) | https://pmc.ncbi.nlm.nih.gov/articles/PMC8514562/ |
| R-23 | Pareto Analysis (Lean Six Sigma Hub) | https://lean6sigmahub.com/pareto-analysis-in-the-analyse-phase-a-complete-guide-to-problem-prioritisation-using-the-80-20-rule/ |
| R-25 | 5 Whys (Six Sigma Study Guide) | https://sixsigmastudyguide.com/5-whys/ |
| R-26 | Toulmin's Model of Argumentation (Purdue OWL) | https://owl.purdue.edu/owl/general_writing/academic_writing/historical_perspectives_on_argumentation/toulmin_argument.html |
| R-27 | A-HMAD: Adaptive Heterogeneous Multi-Agent Debate (Springer, 2025) | https://link.springer.com/article/10.1007/s44443-025-00353-3 |
| R-28 | Multi-LLM-Agents Debate Scaling Challenges (ICLR 2025) | https://d2jud02ci9yv69.cloudfront.net/2025-04-28-mad-159/blog/mad/ |
| R-29 | Can LLM Agents Really Debate? Controlled Study (arXiv) | https://arxiv.org/abs/2511.07784 |
| R-30 | Focused Chain-of-Thought (arXiv, Nov 2025) | https://arxiv.org/abs/2511.22176 |
| R-31 | Decreasing Value of CoT for Reasoning Models (Wharton, 2025) | https://gail.wharton.upenn.edu/research-and-insights/tech-report-chain-of-thought/ |
| R-32 | LCoT2Tree: Optimal Reasoning Path Complexity (EMNLP 2025) | https://aclanthology.org/2025.emnlp-main.329.pdf |
| R-37 | RISE: Recursive Introspection (NeurIPS 2024) | https://proceedings.neurips.cc/paper_files/paper/2024/file/639d992f819c2b40387d4d5170b8ffd7-Paper-Conference.pdf |

### Partially Verified (directionally correct, caveats noted)

| ID | Source | URL | Caveat |
|----|--------|-----|--------|
| R-13 | Fine-Grained TLR (Hey, KIT Ernst Denert Prize 2023) | https://fb-swt.gi.de/fileadmin/FB/SWT/Softwaretechnik-Trends/Verzeichnis/Band_44_Heft_2/Denert2023_3_Hey.pdf | Dissertation summary, not a "survey" as originally described |
| R-16 | STPA PDF (SAS Sofia, 2024) | https://sassofia.com/wp-content/uploads/2024/10/Systems-Theoretic-Process-Analysis-STPA.pdf | "Provably complete" claim unverifiable from this PDF; canonical source is the STPA Handbook (Leveson/Thomas) |
| R-17 | Barrier Analysis (Blue Dragon RCA) | https://bluedragonrootcause.com/barrier-analysis/ | Three-question framing is interpretive summary; nuclear safety origin (DOE-NE-STD-1004-92) is correct but not explicitly attributed by this source |
| R-18 | Defence in Depth (Risk Engineering) | https://risk-engineering.org/concept/defence-in-depth | "Treats every gap as a multi-layer defense failure" is interpretive; page emphasizes barrier independence |
| R-22 | Swiss Cheese Model / HFACS (Wikipedia) | https://en.wikipedia.org/wiki/Swiss_cheese_model | Wikipedia article references HFACS but does not elaborate the four-level taxonomy in detail |
| R-24 | DMAIC Methodology (CSU Ohio) | https://pressbooks.ulib.csuohio.edu/applyingleansixsigmaoe/chapter/chapter-6-the-dmaic-methodology/ | Places Pareto charts in Measure phase appendix, not specifically Analyze phase as claimed |
| R-33 | LLM Confidence Calibration in Biomedical NLP (PMC) | https://pmc.ncbi.nlm.nih.gov/articles/PMC12249208/ | Uses Flex-ECE with range 0.239-0.466, not standard ECE 0.108-0.427 as originally cited. ECE range 0.108-0.427 comes from [R-35] |
| R-35 | LLM Classifier Confidence Scores (Aejaspan blog) | https://aejaspan.github.io/posts/2025-09-01-LLM-Clasifier-Confidence-Scores | Reports ECE 0.108-0.427 across formats. The "0.32 to 0.03 reduction" figure does not appear in this post |
| R-36 | SELF-REFINE: Iterative Refinement with Self-Feedback (Madaan et al., NeurIPS 2023) | https://arxiv.org/abs/2303.17651 | ~20% absolute improvement is accurate. "Diminishing returns are steep" is reasonable interpretation (capped at 4 iterations) but not an explicit paper statement. Originally miscited as ICLR; actual venue is NeurIPS 2023 |
| R-38 | Prompt Injection in Multi-Agent Systems / MASpi (OpenReview, 2025) | https://openreview.net/forum?id=1khmNRuIf9 | Core claim verified. Listed as partially verified due to specificity of transfer-failure claims |
| R-40 | Prompt Injection on Agentic Coding Assistants (arXiv, 2026) | https://arxiv.org/html/2601.17548v1 | Reports 90%+ bypass rate confirmed. Paper analyzes 18 defense mechanisms, not 12 as originally cited |

### Unverifiable

| ID | Source | URL | Issue |
|----|--------|-----|-------|
| R-15 | STPA Method Overview (Octo) | https://octo.org.uk/posts/stpa_method/ | SSL certificate expired. Content confirmed via web search to describe four UCA categories accurately |

### Removed (incorrect attribution)

| Original ID | Source | Issue |
|-------------|--------|-------|
| ~~R-11~~ | SELF-REFINE (duplicate of R-36) | Duplicate citation with wrong venue (ICLR). Consolidated into R-36 with correct venue (NeurIPS 2023). The "13% improvement" figure does not appear in the paper. |
| ~~R-34~~ | Stepwise Confidence Estimation (arXiv 2511.07364) | Paper compares holistic vs. stepwise scoring, not self-consistency vs. verbal confidence. Claim mischaracterizes the paper's contribution. |
| ~~R-39~~ | Multi-Agent LLM Defense Pipeline (arXiv 2509.14285) | Paper does not describe a "PALADIN framework." PALADIN is a different paper (arXiv 2509.25238) about tool-failure recovery. Framework name was fabricated or confused with the wrong paper. |

### Additional Canonical Sources (not URL-validated)

These are foundational works referenced by name but not fetched:

- Leveson, N. & Thomas, J. "STPA Handbook." MIT. Canonical source for
  STPA methodology and UCA completeness arguments.
- DOE-NE-STD-1004-92. US Department of Energy standard for barrier
  analysis in nuclear safety.
- Reason, J. "Human Error" (1990). Original Swiss Cheese Model.
- Wiegmann, D. & Shappell, S. "A Human Error Approach to Aviation
  Accident Analysis" (2003). HFACS taxonomy.
- Toulmin, S. "The Uses of Argument" (1958). Original Toulmin model.
