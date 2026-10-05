# Asphalt Delivery Ticket Reconciler

**Reconcile asphalt delivery tickets against the estimate** — extract ticket data from CSV, PDF or images, validate it, and compare delivered vs estimated tons.

[![Python 3.12](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![CLI](https://img.shields.io/badge/CLI-command--line-blue)](#usage)
[![Platform: Windows](https://img.shields.io/badge/Platform-Windows-0078D6)](https://www.microsoft.com/windows)

**English** | [简体中文](./README.zh-CN.md)

---

## What it solves

Delivery tickets pile up on site in every format — CSV exports, PDF scans, phone photos — and reconciling what was actually delivered against what was estimated is manual, slow and error-prone. This tool **reads tickets from CSV, PDF or images**: plant, truck, load id, date, time, mix, gross, tare and net tons, then validates gross−tare consistency, duplicate ticket ids, impossible timestamps, missing net weights and mix mismatches.

It compares **delivered vs estimated tons** with deterministic math; any explanation of variance is labeled OBSERVED, POSSIBLE or UNVERIFIED — never invented.

## Features

- **Multi-format ticket ingestion** — CSV / PDF / image (photo) delivery tickets
- **Field extraction** — plant, truck, load id, date, time, mix, gross, tare, net tons
- **Deterministic validation** — gross−tare consistency, duplicate ids, impossible timestamps, missing net weights, mix mismatches
- **Delivered vs estimated reconciliation** — variance with labeled explanations
- **Review queue** — uncertain tickets flagged, never silently fixed

## Install

```bash
pip install -r requirements.txt
```

## Usage

```bash
python -m scripts.s06_delivery_ticket_reconcile --project ./project --input ./fixtures/tickets/tickets_10.csv --estimate 199.4
```

Run with `--help` for all options. Every script in this family shares the same CLI convention: `--project <path> --input <path> --output <path> --format json|csv|md|xlsx|pdf --config <path> --verbose --dry-run`.

## Outputs

- `delivery_reconciliation.json` — delivered/estimated totals, variance, truck and load counts
- `delivery_reconciliation.csv` — per-ticket normalized rows
- `exceptions.md` — flagged tickets and variance causes

## How it works

- **Standard lifecycle** — `DISCOVER → INGEST → EXTRACT → NORMALIZE → VALIDATE → CALCULATE/CLASSIFY → OUTPUT → AUDIT`
- **Evidence state machine** — every ticket value moves `EXTRACTED → NORMALIZED → VALIDATED → VERIFIED`; stopping earlier is valid
- **Honest variance labels** — explanations are OBSERVED, POSSIBLE or UNVERIFIED

## Quality gates

- deterministic schema validation on every record;
- golden sample fixtures with exact expected values (`tests/` — incl. a 100-ticket suite);
- review queue: low-confidence values, ambiguous scales and conflicts land in `review_queue.json` / summary — never a silent fix;
- audit log: every run writes `run_id`, timings, input hashes, engine version and outputs to `<project>/audit/`.

## Testing

```bash
python -m pytest -q
```

## Security & privacy

Local files stay local by default. API keys live in environment variables (`ASPHALTCOSTS_API_KEY`, `ADI_AI_API_KEY`) — never in source code. Derived files are written to `working/` or `output/`; source files are never modified.

## License

MIT — see [LICENSE](LICENSE). Part of the [Asphalt Desktop Intelligence](https://github.com/anleeylee/asphalt-desktop-intelligence) toolkit.

## The calculation engine

[**AsphaltCosts.com**](https://asphaltcosts.com/) is the deterministic calculation layer: area → compacted volume → net tons → order tons (allowance applied once) → truckloads → material cost, with sourced planning defaults (145 lb/ft³ FHWA density, editable allowance and truck capacity). This tool feeds measured and validated inputs into that engine (or its labeled local mirror, `asphaltcosts-web-engine/1.0-mirror`) and never re-implements the formulas.
