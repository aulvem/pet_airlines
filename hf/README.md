---
language:
  - en
  - ja
license: cc-by-4.0
tags:
  - travel
  - pets
  - aviation
  - regulations
  - structured-data
  - japan
size_categories:
  - n<1K
task_categories:
  - text-classification
  - question-answering
pretty_name: Pet Air Travel Dataset (Japan Outbound)
configs:
  - config_name: default
    data_files:
      - split: train
        path: pet_airlines_v0.1.csv
---

# Pet Air Travel Dataset (Japan Outbound)

A structured, machine-readable dataset of pet travel policies for major international airlines, focused on flights departing from Japan. Designed to be directly consumable by AI agents and downstream applications without scraping.

## Overview

The web is full of human-readable pet travel guides; almost none are usable by an AI agent or a typed application without per-airline scraping. `pet_airlines` provides the missing structured layer: cabin / baggage / cargo policies, brachycephalic-breed restrictions, weight and carrier-size limits, required documents, fees, and reservation procedures — all under one schema, with English as the master and Japanese as a localized view.

Every record cites a single official source URL and carries an explicit `last_checked_at` date plus a `confidence` rating. Unknown values are marked `unknown` / `null` / `[]` rather than guessed.

## What's in v0.1

| Metric | Value |
|---|---|
| Records | 10 |
| Geographic focus | Japan outbound |
| Language coverage | English (master) + Japanese (view) |
| Confidence: high | 6 |
| Confidence: medium | 4 |
| Last checked | 2026-04-27 |
| Schema version | 0.1 |
| License | CC-BY 4.0 |

The 10 airlines covered:

1. Japan Airlines (JAL)
2. All Nippon Airways (ANA)
3. ZIPAIR
4. Korean Air
5. Asiana Airlines
6. Singapore Airlines
7. Cathay Pacific
8. EVA Air
9. Lufthansa
10. United Airlines

## How to use

### Python (`datasets` library)

```python
from datasets import load_dataset

ds = load_dataset("Aulvem/pet-airlines")
print(ds["train"][0])
# {'id': 'pet_airline_001', 'airline_name_en': 'Japan Airlines', ...}
```

### Direct JSON fetch (preserves nested structure)

The CSV is the default split, but the JSON file preserves the original nested objects (`weight_limit.scope`, `brachycephalic_restriction.applies`, etc.) and is the recommended source if your downstream code needs typed objects rather than flattened columns.

```python
import json, urllib.request

url = "https://huggingface.co/datasets/Aulvem/pet-airlines/resolve/main/pet_airlines_v0.1.json"
data = json.load(urllib.request.urlopen(url))

for r in data["records"]:
    print(r["id"], r["airline_name_en"], r["confidence"])
```

### Schema validation

```python
import json, urllib.request, jsonschema  # pip install jsonschema

base = "https://huggingface.co/datasets/Aulvem/pet-airlines/resolve/main"
schema  = json.load(urllib.request.urlopen(f"{base}/schema.json"))
dataset = json.load(urllib.request.urlopen(f"{base}/pet_airlines_v0.1.json"))
jsonschema.validate(dataset, schema)   # raises on first failure
```

## Schema

The full JSON Schema (Draft 2020-12) ships in this dataset as `schema.json`. Top-level fields per record:

| Field | Type | Notes |
|---|---|---|
| `id` | string | Stable identifier, e.g. `pet_airline_001` |
| `airline_name_en` / `airline_name_ja` | string | English master + Japanese view |
| `official_url` | string (URI) | Single source page; required for every record |
| `source_language` | enum | `en` / `ja` / `zh` / `ko` / `de` / `multi` |
| `pet_type` | array | `dog`, `cat`, `bird`, `rabbit`, `small_mammal`, `service_animal`, `emotional_support_animal`, `other` |
| `cabin_allowed` / `checked_baggage_allowed` / `cargo_allowed` | enum | `yes` / `no` / `conditional` / `unknown` |
| `brachycephalic_restriction` | object | `{ applies, details_en, details_ja }` |
| `weight_limit` | object | `{ value, unit (kg/lb), scope, notes_en, notes_ja }` |
| `carrier_size_limit` | object | `{ length_cm, width_cm, height_cm, sum_cm, notes_en, notes_ja }` |
| `reservation_method_en/ja` | string | Free text |
| `required_documents_en/ja` | array of string | One document per element |
| `fee_info_en/ja` | string | Original currency preserved verbatim |
| `notes_en/ja` | string | Seasonal / route carve-outs (ISO date ranges) |
| `last_checked_at` | string (ISO date) | Verification date |
| `confidence` | enum | `high` / `medium` / `low` / `unknown` |

The CSV at `pet_airlines_v0.1.csv` is a flattened 33-column view of the same data (nested objects expanded with prefixed column names; arrays joined with ` | `). Both files are kept bit-identical with the canonical sources in the GitHub repository.

## Important notice

> **Reference only. Always verify with official sources before travel.**

Pet travel rules change frequently and are subject to per-route, per-aircraft, and per-season conditions, and may differ between an airline's published policy and the experience at the check-in counter. Confirm directly with the airline, the airport, and the animal-quarantine authority of every country on your route before booking. The dataset maintainers accept no liability for travel outcomes derived from this data; see the source repository's `TERMS.md` for the full disclaimer.

## Update cadence

The dataset is automatically re-fetched and re-extracted weekly (Mondays 03:00 UTC) by a GitHub Actions workflow in the source repository. Each run produces a pull request with a structured diff against the prior snapshot. Updates are merged into the canonical dataset only after human review, and the merged result is mirrored here on Hugging Face.

You can pin to a specific revision using the standard HF dataset versioning, e.g. `load_dataset("Aulvem/pet-airlines", revision="<commit-sha>")`.

## Roadmap

| Version | Focus |
|---|---|
| v0.1 (current) | 10 airlines, Japan outbound, JSON + CSV |
| v0.2 | Schema normalization (unit / currency harmonization), 25-30 airlines |
| v0.3 | Country-level quarantine and import rules |
| v1.0 | Public REST API with route / breed / season query semantics |
| v2.0 | MCP server for direct AI-agent integration (Claude, ChatGPT, Cursor, Claude Code) |

Schema changes are additive — v0.1-pinned consumers will not be broken by later versions.

## Source repository

Canonical source, weekly extraction pipeline, schema definition, and contribution process all live on GitHub:

**https://github.com/aulvem/pet_airlines**

Issues, pull requests, and inaccuracy reports should go there. This Hugging Face dataset is a downstream mirror.

## How to cite

If you use this dataset in research, products, or downstream tooling, please cite as:

> pet_airlines contributors (2026). *pet_airlines: AI-readable Pet Air Travel Dataset* (v0.1). Available at https://huggingface.co/datasets/Aulvem/pet-airlines (mirror) and https://github.com/aulvem/pet_airlines (canonical), licensed under CC-BY 4.0.

CC-BY 4.0 requires attribution; the citation above — or any substantively equivalent form that names the dataset, version, and license — satisfies that requirement.

## License

Released under [Creative Commons Attribution 4.0 International (CC-BY 4.0)](https://creativecommons.org/licenses/by/4.0/). You are free to share and adapt, including for commercial use, provided you give appropriate credit.

The underlying airline policy text remains the property of each airline. This dataset extracts factual information (numerical limits, document lists, yes / no flags) under principles applicable to factual extraction; it does not redistribute the airlines' original prose. If you republish raw airline-authored text obtained via the source URLs, observe each airline's own terms.

## Contact

Questions, bug reports, and corrections via GitHub issues:

**https://github.com/aulvem/pet_airlines/issues**

Please cite the official source page when reporting an inaccuracy.
