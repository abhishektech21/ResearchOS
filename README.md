# Agentic AI Research Assistant

**Course 26ECAC401 — Agentic AI, Labs 1–14 (capstone).**

A working multi-agent research studio: FastAPI + React/TypeScript + LangGraph + a real MCP
tool server + live OpenAlex/Crossref/arXiv retrieval + Ollama, with seven agents, project
isolation, human approval gates, and an audit trail you can verify.

It is built to be **honest before it is impressive**: no invented papers, no demo corpus,
no fake progress bars, and when the LLM is unavailable it says so instead of pretending.

---

## Quick start (2 terminals, ~3 minutes)

```bash
# 1) backend
cd backend
python -m venv .venv && source .venv/bin/activate      # Windows: .\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
cp .env.example .env                                   # then set SCHOLARLY_CONTACT_EMAIL
python -m uvicorn app.main:app --host 0.0.0.0 --port 8000

# 2) frontend → built into backend/static/ui, served by the same process
cd frontend
npm install
npm run build
# open http://localhost:8000/
```

Optional LLM (prose synthesis, plus real embeddings):

```bash
ollama serve
ollama pull qwen2.5:3b
ollama pull nomic-embed-text
```

Without Ollama the app still runs end-to-end through deterministic, evidence-grounded
paths and shows *"LLM unavailable"* with the reason.

**First run:** create a project with a specific topic (e.g. *"road anomaly detection using
YOLO for Indian roads"*), press **Run**, then approve the draft in the **Draft** tab to
unlock the manuscript exports.

## Prove it works (no clicking)

```bash
cd backend
python -m pytest -q                 # 80 offline tests (84 with RUN_LIVE_TESTS=1)
python smoke_e2e.py "federated learning for IoT security"   # 15/15 live checks
```

Recorded results: **80 passed, 4 skipped in 20.03 s** (offline);
**84 passed** with `RUN_LIVE_TESTS=1`; **15/15** live smoke checks in 7.0 s.

## The seven agents

| Agent | Does | Never does |
|---|---|---|
| Literature Discovery | plan → 3 sources in parallel → dedupe/rank → save → observe → revise (≤3) | invent a record to reach the target |
| Gap & Idea | evidence matrix → gaps with verbatim quotes → research ideas | state a gap no retrieved paper supports |
| Deep Research | sub-questions → extra retrieval → evidence table + uncertainties | summarise papers it did not retrieve |
| Drafting | proposal manuscript, bibliography = stored records only | cite anything outside the project |
| Citation & Compliance | DOI resolution, marker↔bibliography, unused entries | claim the source supports the claim |
| Funding & Journal Radar | venues (ISSN), funders (award ids), collaborators | produce deadlines, impact factors, acceptance rates |
| Collaboration | author overlap with justification | imply availability |
| Export (gatekeeper) | 9 artefact kinds, 3 approval-gated (HTTP 403) | export a manuscript before a human approves |

## Deliverables

| # | Deliverable | File |
|---|---|---|
| 1 | Repository audit | [`docs/01_repository_audit.md`](docs/01_repository_audit.md) |
| 2 | Architecture and workflow | [`docs/02_architecture_and_workflow.md`](docs/02_architecture_and_workflow.md) |
| 3 | Labs 1–14 → real files | [`docs/03_labs_1_14_mapping.md`](docs/03_labs_1_14_mapping.md) |
| 4 | File list | [`docs/04_file_list.md`](docs/04_file_list.md) |
| 5 | Dependencies and environment | [`docs/05_dependencies_and_env.md`](docs/05_dependencies_and_env.md) |
| 6 | Database setup | [`docs/06_database_setup.md`](docs/06_database_setup.md) |
| 7 | Windows PowerShell commands | [`docs/07_windows_powershell.md`](docs/07_windows_powershell.md) |
| 8 | Test commands and actual results | [`docs/08_testing_and_results.md`](docs/08_testing_and_results.md) |
| 9 | End-to-end demo transcript | [`docs/09_end_to_end_demo.md`](docs/09_end_to_end_demo.md) |
| 10 | Known limitations | [`docs/10_known_limitations.md`](docs/10_known_limitations.md) |
| 11 | Academic justification | [`docs/11_academic_justification.md`](docs/11_academic_justification.md) |

## Repository layout

```
backend/
  app/
    api/        projects, runs, workspace, governance, system
    agents/     the seven agents + domain packs + export gatekeeper
    skills/     reusable, schema-validated skills (LLM optional)
    tools/      14 tools: specs → registry → runner (MCP or direct)
    mcp/        stdio MCP server + client
    services/   scholarly (OpenAlex/Crossref/arXiv), llm (Ollama), embeddings, health
    graph/      LangGraph state + 13-node workflow
    runtime/    step timing, retries, tracing, hash-chained audit log
    db/         17 tables (SQLite + SQLAlchemy async)
  tests/        17 modules, 84 tests
  smoke_e2e.py  live end-to-end proof
  static/ui/    built SPA served at /
frontend/       React 18 + TS + Vite + Tailwind (9 tabs)
docs/           the eleven deliverables
```

## API at a glance

`GET /api/health` · `GET /api/system/status` · `GET /api/system/tools` · `GET /api/labs`
`POST /api/projects` · `POST /api/projects/{id}/runs` · `GET /api/runs/{id}/events` (SSE)
`GET /api/projects/{id}/{papers,gaps,ideas,draft,citations,radar,exports,audit}`
`POST /api/projects/{id}/citations/audit` · `POST /api/approvals/{id}/decide` ·
`POST /api/projects/{id}/exports` (403 until approved) · interactive docs at `/docs`

## Honesty contract (the part that matters)

1. Different topics retrieve different papers — verified in
   `test_lab14_capstone_e2e::test_two_topics_produce_different_results_and_stay_isolated`.
2. A failing source is reported as failing (`failed_sources`, per-source status), never
   replaced by demo data.
3. Every gap carries `evidence_paper_ids` and verbatim quotes; unsupported gaps are dropped.
4. The citation audit distinguishes *DOI resolves* from *claim supported* and states
   `claim_level_evidence_check: "not_evaluated"`.
5. Exports are real files with sizes on disk; the audit log is hash-chained and
   tamper-evident.
6. When Ollama is absent: `llm_available=false`, `generated_by="heuristic"`, and an explicit
   warning in the run — never invented model output.

Known weaknesses are listed bluntly in [`docs/10_known_limitations.md`](docs/10_known_limitations.md).
