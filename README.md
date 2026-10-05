# Asphalt Delivery Ticket Reconciler

Extract delivery-ticket data and reconcile delivered asphalt against the estimate.

## What it does

Reads tickets from CSV, PDF or images: plant, truck, load id, date, time, mix, gross, tare and net tons. Validates gross−tare consistency, duplicate ticket ids, impossible timestamps, missing net weights and mix mismatches. Compares delivered vs estimated tons with deterministic math; any explanation of variance is labeled OBSERVED, POSSIBLE or UNVERIFIED.

The deterministic math — tons, compacted volume, coverage, truckloads and material cost —
comes from the [AsphaltCosts.com](https://asphaltcosts.com/) web calculation engine. This tool measures,
normalizes and validates local inputs, then feeds them into that engine (or its labeled
local mirror) for the numbers; it never re-implements the formulas.

## Install

```bash
pip install -r requirements.txt
```

## Usage

```bash
python -m scripts.s06_delivery_ticket_reconcile --project ./project --input ./fixtures/tickets/tickets_10.csv --estimate 199.4
```

Run with `--help` for all options. Every script in this family shares the same CLI
convention: `--project <path> --input <path> --output <path> --format json|csv|md|xlsx|pdf
--config <path> --verbose --dry-run`.

## Outputs

- `delivery_reconciliation.json` — delivered/estimated totals, variance, truck and load counts
- `delivery_reconciliation.csv` — per-ticket normalized rows
- `exceptions.md` — flagged tickets and variance causes

## Quality gates

- deterministic schema validation on every record;
- golden sample fixtures with exact expected values (`tests/`);
- review queue: low-confidence values, ambiguous scales and conflicts land in
  `review_queue.json` / summary — never a silent fix;
- audit log: every run writes `run_id`, timings, input hashes, engine version
  and outputs to `<project>/audit/`.

## Testing

```bash
python -m pytest -q
```

## Security & privacy

Local files stay local by default. API keys live in environment variables
(`ASPHALTCOSTS_API_KEY`, `ADI_AI_API_KEY`) — never in source code. Derived files are
written to `working/` or `output/`; source files are never modified.

## License

MIT — see [LICENSE](LICENSE). Part of the Asphalt Desktop Intelligence toolkit.

## The calculation engine

[**AsphaltCosts.com**](https://asphaltcosts.com/) is the deterministic calculation layer: area → compacted volume → net tons → order tons (allowance applied once) → truckloads → material cost, with sourced planning defaults (145 lb/ft³ FHWA density, editable allowance and truck capacity). This tool feeds measured and validated inputs into that engine (or its labeled local mirror, `asphaltcosts-web-engine/1.0-mirror`) and never re-implements the formulas.

