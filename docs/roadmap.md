# Roadmap

Long-form companion to the brief roadmap in [README.md](../README.md). Versions describe project capability level. Each version ships when the previous is genuinely stable, not on a calendar deadline.

## v0.1 — Skeleton + 10 airlines (current)

Demonstrates that the schema and extraction pipeline work end-to-end on a real, weekly-refreshing dataset.

- 10 major airlines, Japan-outbound focus
- v0.1 schema (English master + Japanese view)
- Weekly automated re-fetch + re-extract via GitHub Actions
- Public release under CC-BY 4.0

## v0.2 — Schema normalization

Goal: make the dataset directly queryable without per-airline unit handling.

- Unit harmonization (parallel normalized fields)
- Currency harmonization
- Tighter enums for common patterns
- Expanded airline coverage

Migration policy: v0.1 → v0.2 is additive, not destructive. v0.1 fields are preserved.

## v0.3 — Country-level rules

Adds arrival-side requirements (quarantine, import rules, breed restrictions at the country level) as a separate table cross-referenced with airline records.

## v1.0 — Public REST API

Hosted query interface so applications do not need to redownload snapshots. The dataset itself remains free under CC-BY 4.0; the API is a convenience layer.

## v2.0 — MCP server

Direct integration with AI agents (Claude, ChatGPT, Cursor). Primary tool: structured query joining airline policy and country rules.

## How versions interact with downstream consumers

| Version transition | Schema impact | Action required |
| --- | --- | --- |
| v0.1 → v0.1.x patch | None | None |
| v0.1 → v0.2 | Additive | Optional |
| v0.2 → v0.3 | Additive | Optional |
| v0.3 → v1.0 | None for dataset | Optional |
| v1.0 → v2.0 | None for dataset / API | Optional |

The intent across all versions is **never break a v0.1-pinned consumer.**
