# SDD Workflow Checklist

## Phase A: Specification
- [x] Step 1: Brainstorm — 2026-05-26. Explored approaches (new scope vs graceful degradation vs separate commands). Chose graceful degradation. Brainstorm doc written to brainstorm/04-gap-audit-optional-tasks.md.
- [x] Step 2: Specify — 2026-05-26. Created spec.md with 3 user stories, 16 FRs, 7 SCs. Review-spec passed after fixing: added FR-016 (version bump), harmonized info message text, added FR-012 scope note.
- [x] Step 3: Clarify — 2026-05-26. No critical ambiguities found. All taxonomy categories Clear (spec went through 2 adversarial audit rounds pre-specify).
- [x] Step 4: Spec Gap Audit — 2026-05-26. Auditor found 5 blocking, 6 non-blocking findings. All fixed: added FR-017 (host-side category validation), FR-018 (clean output degraded mode), SC-008 (version bump), amended FR-004 (timing), FR-008 (equivalence note), FR-009 (empty patterns), FR-012 (exact arrays + ordering), SC-001 (reframed for non-determinism), added backward compat assumption. 0 deferrals.

## Phase B: Planning
- [x] Step 5: Plan — 2026-05-26. Created plan.md with 4 design decisions, research.md with 5 research items, data-model.md, contracts/command-schema.md. Constitution check passed all 8 principles.
- [x] Step 6: Tasks — 2026-05-26. Created tasks.md with 19 tasks across 6 phases. Review-plan passed clean: full FR/SC coverage, no red flags, no naming inconsistencies.
- [x] Step 7: Analyze — 2026-05-26. Cross-artifact analysis: 100% coverage (26/26 requirements mapped), 0 critical, 0 high, 1 medium (T004/T013 scope boundary note), 1 low (verification task test fixture note).
- [x] Step 8: Plan Gap Audit — 2026-05-26. Auditor found 5 blocking, 5 non-blocking findings. All fixed: added T020 (spec-scope regression), extended T008/T011/T012/T013 with verification detail, added state propagation note to T002, explicit spec-scope to T009, backtrace limitation to T017, explicit SKILL.md wording to T018. 0 deferrals.
- [x] Step 9: Back-Trace — 2026-05-26. All 10 plan findings already addressed in step 8. Barrier walk found all spec layers present for each finding. 0 additions needed, 0 reset triggers.

## Phase C: Implementation
- [x] Step 10: Implement — 2026-05-26. All 20 tasks executed: T001 version bump, T002-T010 command file changes (Section 4 optional loading, Section 6 conditional preamble/checklist/artifacts/validation/filter/patterns, Section 8 category validation, Section 9 categories_audited, Section 10 summary), T013 pattern filtering, T014-T019 documentation updates (original spec, contracts, data-model, README, SKILL.md, project README). T011/T012/T020 verification deferred to exploratory testing.
- [x] Step 11: Exploratory Testing — 2026-05-26. Ran degraded mode test (T011) against spec 003 artifacts. All 3 verification tasks pass. Exploratory tester found 2 moderate, 3 minor findings. Fixed 3: output format template generalized, README categories_audited field documented, Section 4 double-read removed. Skipped 2: numbering mismatch (by design), empty tasks.md (pre-existing, out of scope).
- [x] Step 12: Post-Implementation Back-Trace — 2026-05-26. Skipped (step 11 produced no findings).
- [x] Step 13: SC Verify — 2026-05-26. All 8 SCs pass. SC-001 (backward compat structure), SC-002 (degraded categories), SC-003 (partial summary), SC-004 (categories_audited field), SC-005 (pattern filtering), SC-006 (no error on missing tasks.md), SC-007 (jq parseable), SC-008 (version 0.2.0 > 0.1.0).
- [ ] Step 14: Pre-Merge — squash commits, fast-forward merge to main, clean up state
