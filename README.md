# pet_airlines — AI-readable Pet Air Travel Dataset

A structured, machine-readable dataset of pet travel policies for major international airlines, with a focus on flights departing from Japan.

The web is full of human-readable pet travel guides; almost none of them are usable by an AI agent or a downstream application without scraping and re-parsing. `pet_airlines` is the missing structured layer.

## What this is

- A JSON + CSV dataset (`data/`) following a documented schema.
- An extraction skill (`skills/pet-airline-extraction/SKILL.md`) that defines exactly how each field is derived from airline source pages.
- Tooling (`scripts/`) that re-fetches sources and re-extracts on a weekly cadence.
- Diff-based change tracking via Git, so consumers can audit *what* changed and *when*.

## v0.1 scope

| | |
|---|---|
| Airlines | 10 (full list in [data/pet_airlines_v0.1.json](data/pet_airlines_v0.1.json)) |
| Geographic focus | Outbound from Japan |
| Schema | v0.1 — see [skills/pet-airline-extraction/SKILL.md](skills/pet-airline-extraction/SKILL.md) |
| Update cadence | Weekly (Mondays 03:00 UTC) via GitHub Actions |
| License | CC-BY 4.0 — see [LICENSE](LICENSE) |

The 10 airlines covered in v0.1:

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

## Disclaimer

> **Reference only. Always verify with official sources before travel.**

Pet travel rules change, are season- and route-dependent, and frequently differ between an airline's published policy and the experience at the check-in counter. Treat this dataset as a starting point for research, not as the final word. See [DISCLAIMER.md](DISCLAIMER.md) and [TERMS.md](TERMS.md).

## Usage

```bash
# Read the dataset directly
cat data/pet_airlines_v0.1.json

# Or as CSV
cat data/pet_airlines_v0.1.csv
```

Programmatic consumers should pin to a specific schema version (`v0.1`) — breaking schema changes will be released as `v0.2`, `v0.3`, etc., not as in-place edits.

## License & attribution

The dataset is released under [Creative Commons Attribution 4.0 International (CC-BY 4.0)](LICENSE). You are free to share and adapt, including for commercial use, provided you give appropriate credit.

The underlying airline policy text is the property of each airline. We extract structured facts under fair-use principles for the purpose of interoperability; the resulting dataset is our contribution. If you republish raw airline-authored text, observe each airline's own terms.

## How to cite

If you use this dataset in research, products, or downstream tooling, please cite as:

> pet_airlines contributors (2026). *pet_airlines: AI-readable Pet Air Travel Dataset* (v0.1). Available at https://github.com/aulvem/pet_airlines, licensed under CC-BY 4.0.

CC-BY 4.0 requires attribution; the citation above — or any substantively equivalent form that names the dataset, version, and license — satisfies that requirement.

## Roadmap

- **v0.1 (now)** — 10 airlines, Japan outbound, JSON/CSV.
- **v0.2** — Schema normalization, units harmonization, expanded airline coverage.
- **v0.3** — Country-level quarantine and import requirements.
- **v1.0** — Public REST API.
- **v2.0** — MCP server for direct AI agent consumption.

Full roadmap: [docs/roadmap.md](docs/roadmap.md).

## Contributing

Bug reports and corrections are welcome via GitHub issues. Please cite the official source page when reporting an inaccuracy.

## See also

- [日本語 README](README.ja.md)
