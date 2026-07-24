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

---

### Reproduction of the gap

Because this is a missing-coverage issue (not a runtime bug), "reproducing" it
means confirming the test does not exist and pinning down exactly where it
should live. I verified three things locally on this branch.

**1. The integration suite is empty — it collects zero tests:**

```
$ .venv/bin/pytest tests/integration -v
collected 0 items
============================ no tests ran in 1.25s =============================
```

`tests/integration/` contains only `__init__.py`; there is no
`test_ingestion_pipeline.py`.

**2. The pipeline orchestrator has no test that exercises it end-to-end:**

```
$ grep -rl "ingest_resume\|IngestionPipeline" tests/
(no matches)
```

The orchestrator lives in
[`ingestion/pipeline.py`](ingestion/pipeline.py) — `IngestionPipeline.ingest_resume()`
chains the full flow: `ResumeParser.parse()` → `StrategySelector.chunk()` →
`BatchEmbeddingProcessor.process()` (embed + store in the vector DB) →
`_record_ingested_source()`. Every unit test under `tests/unit/` covers one of
these components in isolation (e.g. `test_resume_parser.py`,
`test_semantic_chunker.py`, `test_batch_processor.py`), but nothing wires them
together, so a break at a seam between components would go undetected.

**3. The required fixtures directory does not exist:**

```
$ ls tests/fixtures/sample_resumes
ls: tests/fixtures/sample_resumes: No such file or directory
```

There is no `tests/fixtures/` directory at all. The only resume sample today is
an inline string fixture, `sample_resume_text`, in
[`tests/conftest.py`](tests/conftest.py) — not a file-based fixture that
exercises the upload path.

**Conclusion / where the fix lives:**
- Add sample resume file(s) under `tests/fixtures/sample_resumes/`.
- Add `tests/integration/test_ingestion_pipeline.py` that drives
  `IngestionPipeline.ingest_resume()` from a fixture file through to stored
  embeddings, asserting each stage's output and the final `IngestResult`
  (`chunk_count`, `skipped`, `source_id`), with the embedding provider / vector
  DB stubbed so the test runs offline.

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/emilypmendez/pathreview/commit/ab805f9

**Reproduction summary:**
Ran `pytest tests/integration -v` and `grep -rl "ingest_resume\|IngestionPipeline" tests/`,
observing that the integration suite collects 0 tests, no test touches the
pipeline orchestrator, and `tests/fixtures/sample_resumes/` does not exist —
confirming the coverage gap is real and isolating exactly where the fix belongs.

**PLAN.md link:** https://github.com/emilypmendez/pathreview/blob/test/18-e2e-ingestion-test-with-sample-resume-fixture/PLAN.md

**Walkthrough video (recommended):** N/A

**Blockers or open questions:**
None blocking. Verified via a throwaway script that the full pipeline runs
end-to-end with the existing `MockEmbeddingProvider` and a fake vector-DB spy,
so no production code changes are needed — the fix is test-only. One decision
settled during research: use a fake vector DB rather than a live ChromaDB
client, because chunk metadata includes a list (`detected_sections`) that
ChromaDB's scalar-only metadata constraint would reject.
