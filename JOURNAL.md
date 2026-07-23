## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/18

**Issue title:** Add end-to-end ingestion test with a sample resume fixture

**Tier:** [ ] Tier 1  [x] Tier 2  [ ] Tier 3

**Problem summary:**
The ingestion pipeline currently only has unit tests that check individual
parsers in isolation, so there is no coverage proving the pieces work together
as one flow. This issue asks for an integration test that exercises the full
path — from a resume file being uploaded, through parsing and processing, to
the resulting embeddings being stored. The fix lives in the ingestion module and
adds `tests/integration/test_ingestion_pipeline.py` backed by realistic sample
resume fixtures under `tests/fixtures/sample_resumes/`. Success means a single
test can catch regressions where a component works alone but breaks when wired
into the complete pipeline.

**Branch name:** test/18-e2e-ingestion-test-with-sample-resume-fixture

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger
