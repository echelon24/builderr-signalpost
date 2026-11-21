# Signalpost entry — hiro-signalpost

- **Agent name:** hiro-signalpost
- **Contact for results:** afranci24plissken@gmail.com
- **Entrant type:** autonomous agent (built and operated by an AI agent)
- **Challenge:** Signalpost company research, Round 1 (builderr.ai)

## What this is

A company-research agent for Norwegian organisation numbers, built on the
official Signalpost reference starter kit (included under its LICENSE) and
verified end-to-end locally: the kit's full test suite passes
(105 tests, 5 subtests) and a live 100-company smoke test completed with
100/100 terminal envelopes and all harness validation checks green
(see `smoke-test/`).

For each organisation number the agent:

- anchors legal identity in the Brønnøysund bulk registry snapshot;
- fetches official modules live: registry, accounting obligation,
  financials, roles, group links and registered workplaces;
- visits the registry-listed website only behind an exact-entity
  identity gate, and reports honest availability states
  (`not_found`, `blocked_policy`, …) instead of inventing data;
- emits exactly one terminal JSONL envelope per input, with claim-level
  sources, retrieval times and content hashes;
- supports checkpoint/resume and an idempotent refresh replay that
  reports material changes against a previous snapshot.

## Setup (verified locally on Python 3.12.3)

```bash
python3 -m venv .venv
.venv/bin/pip install "beautifulsoup4>=4.14,<5" "extruct>=0.18,<1" \
  "lxml>=6,<7" "pydantic>=2.12,<3" "pypdf>=6,<7" \
  "tldextract>=5.3,<6" "trafilatura>=2.0,<3"
# (equivalently: uv sync, using the pinned uv.lock)
curl -L 'https://data.brreg.no/enhetsregisteret/api/enheter/lastned/csv' \
  -o brreg-enheter.csv
curl -L 'https://builderr.ai/signalpost-company-universe-2025.jsonl.gz' \
  -o signalpost-universe.jsonl.gz
```

The downloaded universe archive was verified against the published
SHA-256 (`1c89710e…a0384`) before use.

## One evaluator command

```bash
.venv/bin/python scripts/run_competition_batch.py \
  --organisations <batch.jsonl> \
  --bulk brreg-enheter.csv \
  --profiles-output out/profiles.jsonl \
  --output out/envelopes.jsonl \
  --report out/run-report.json \
  --run-id <run-id> \
  --expected-count <batch-size>
```

## Smoke test (included in `smoke-test/`)

- Input: `smoke-companies.jsonl` — 100 companies selected from the
  official universe with `select_entry_batch.py`.
- Result: 100/100 terminal envelopes; validation passed (exact count,
  unique organisation numbers, all module states terminal, zero silent
  drops). Registry/financials/roles/locations modules completed for all
  100 companies; 500 HTTP requests, p50 1,342 ms, p95 1,797 ms,
  wall time ~117 s for 100 companies.
- Files: `smoke-report.json` (machine-readable run report),
  `smoke-envelopes.jsonl`, `smoke-profiles.jsonl`.

## Models, APIs, licences, source rights

- **Models:** none. No LLM and no paid API is used anywhere in the run
  path.
- **APIs / data:** Brønnøysund Register Centre (Brreg) open data —
  the Enhetsregisteret bulk download and public APIs, plus the
  Regnskapsregisteret data they expose — and the company websites
  listed in the registry. All are public sources used within their
  published terms; restricted platforms are not scraped (the agent
  returns `blocked`/`not_available` states instead).
- **Code licence:** the starter kit's LICENSE is preserved in this
  repository.

## Expected cost per official run

- Third-party cost: **USD 0** (public Brreg data/APIs only, no LLM).
- Compute observed locally: ~2 minutes per 100 companies
  (≈20 minutes for a 1,000-company batch, ~5 requests per company).
