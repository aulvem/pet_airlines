---
name: pet-airline-extraction
description: Extract structured pet travel policy data from airline official web pages into the v0.1 schema. Used by scripts/extract.js when processing fetched HTML.
---

# Pet Airline Policy Extraction Skill (v0.1)

This skill defines how to convert an airline's official pet/animal travel policy page (HTML or plain text) into a single JSON record that conforms to the `pet_airlines_v0.1` schema. It is the core knowledge asset of this dataset.

The extractor (LLM or rule-based) MUST follow every section of this document. When in doubt, prefer `unknown` over a guess.

---

## 1. Target schema

Every output record MUST contain exactly these top-level fields. Field order is not significant, but no field may be omitted.

| Field | Type | Notes |
|---|---|---|
| `id` | string | Stable identifier, format `pet_airline_NNN` (zero-padded 3 digits). Assigned by the dataset maintainer; never invent. |
| `airline_name_en` | string | Official English name as printed on the airline's own site. |
| `airline_name_ja` | string | Official Japanese name. If the airline has no JA name, romanize and mark `confidence: medium` for this field via `notes_ja`. |
| `official_url` | string (URL) | The single canonical page that was the primary source for this record. |
| `source_language` | enum: `en`, `ja`, `zh`, `ko`, `de`, `multi` | Primary language of `official_url`. |
| `pet_type` | array of enum | Allowed values: `dog`, `cat`, `bird`, `rabbit`, `small_mammal`, `service_animal`, `emotional_support_animal`, `other`. Empty array = no pets accepted. |
| `cabin_allowed` | enum: `yes`, `no`, `conditional`, `unknown` | In-cabin travel. `conditional` if route/aircraft/weight dependent. |
| `checked_baggage_allowed` | enum: `yes`, `no`, `conditional`, `unknown` | Pet as accompanied checked baggage. |
| `cargo_allowed` | enum: `yes`, `no`, `conditional`, `unknown` | Pet as cargo (separate AWB). |
| `brachycephalic_restriction` | object | See §1.1. |
| `weight_limit` | object | See §1.2. |
| `carrier_size_limit` | object | See §1.3. |
| `reservation_method_en` / `reservation_method_ja` | string | Free text, one paragraph max. |
| `required_documents_en` / `required_documents_ja` | array of string | One document per array element. |
| `fee_info_en` / `fee_info_ja` | string | Keep original currency and unit. |
| `notes_en` / `notes_ja` | string | Anything material that does not fit elsewhere (seasonal embargo, route exceptions, COVID-era residue). |
| `last_checked_at` | string (ISO-8601 date, `YYYY-MM-DD`) | The date the source page was fetched. |
| `confidence` | enum: `high`, `medium`, `low`, `unknown` | Record-level confidence. See §3. |

### 1.1 `brachycephalic_restriction` (short-nosed breed restriction)

```json
{
  "applies": "yes" | "no" | "conditional" | "unknown",
  "details_en": "string or 'unknown'",
  "details_ja": "string or 'unknown'"
}
```

If the policy mentions short-nosed/snub-nosed/brachycephalic/pug/bulldog/Persian-cat in a restrictive context, set `applies: "yes"`. If it explicitly says no such restriction, `"no"`. Silence ≠ "no" — use `"unknown"`.

### 1.2 `weight_limit`

```json
{
  "value": number | null,
  "unit": "kg" | "lb" | null,
  "scope": "pet_only" | "pet_plus_carrier" | "carrier_only" | "unknown",
  "notes_en": "string or 'unknown'",
  "notes_ja": "string or 'unknown'"
}
```

`scope` distinguishes what the published numeric limit refers to:

- `pet_only` — limit applies to the animal alone (e.g., "the pet must weigh under 8 kg").
- `pet_plus_carrier` — limit applies to pet + carrier combined (e.g., "8 kg including carrier").
- `carrier_only` — limit applies to the carrier itself, independent of the animal. This case is real: some airlines publish a soft-carrier empty-weight cap, and some cargo programs weight-check the carrier separately from a hold-loaded pet. Use this when the page's numeric limit is unambiguously about the container, not the animal or the combined load.
- `unknown` — the page does not state which interpretation applies.

NEVER convert units. If the page says "8 kg including carrier", emit `value: 8, unit: "kg", scope: "pet_plus_carrier"`. Do not normalize lb → kg. `notes_en` and `notes_ja` mirror each other (per §2 rule 9); a fact appearing in only one language is copied verbatim with a `[ja: untranslated]` or `[en: untranslated]` tag rather than guessed.

### 1.3 `carrier_size_limit`

```json
{
  "length_cm": number | null,
  "width_cm": number | null,
  "height_cm": number | null,
  "sum_cm": number | null,
  "notes_en": "string or 'unknown'",
  "notes_ja": "string or 'unknown'"
}
```

If the source uses inches, convert to cm and round to one decimal place — but record the original in `notes_en` (e.g., `"original: 19 x 13 x 9 in"`) and mirror in `notes_ja` (e.g., `"原表記: 19 x 13 x 9 in"`). This is the ONLY unit conversion permitted; it is allowed because cm is the dataset's normalized unit for length.

If only a sum-of-dimensions is given (e.g., "sum of 3 sides ≤ 115 cm"), populate `sum_cm` and leave the individual dimensions `null`.

---

## 2. Extraction rules (HARD RULES — do not violate)

1. **No invention.** If a field is not stated on the source page, output `"unknown"` (or `null` for numeric fields, or `[]` for arrays). Do not infer from sister airlines, prior versions, or general knowledge.
2. **No paraphrasing of policy-bearing facts.** For numeric limits, fees, and document names, copy the source language wording into the matching language field. Translation into the other language is allowed and expected, but must not introduce new claims.
3. **Preserve units and currencies.** Yen stays yen, USD stays USD. Do not convert.
4. **Seasonal/temperature embargoes** must be expressed as ISO dates in `notes_en` / `notes_ja`. Example: `"Heat embargo on cargo pets between 2026-06-01 and 2026-09-30 (approximate; verify yearly)."` Never use "summer" alone.
5. **Route-specific carve-outs** (e.g., "not allowed on US routes") go in `notes_en` / `notes_ja`, and the relevant top-level enum becomes `"conditional"`.
6. **Service animals and emotional-support animals** are tracked separately via `pet_type`. Their fee/weight/size rules typically differ; if the policy distinguishes them, capture the general-pet rule in the structured fields and put the service-animal carve-out in `notes_en` / `notes_ja`.
7. **One source URL per record.** If a page links out to a PDF that is the actual policy, use the PDF URL as `official_url` and note the entry-page URL in `notes_en`.
8. **Date discipline.** `last_checked_at` is the fetch date, not the page's "last updated" date. Page-claimed update dates, if any, go in `notes_en`.
9. **Language fields are mirrors, not duplicates.** `notes_en` and `notes_ja` should contain the same factual claims, translated. If a fact is only available in one language and you cannot translate it confidently, mirror it verbatim with a `[ja: untranslated]` tag.
10. **Empty is meaningful.** An empty `required_documents_en: []` means "the page lists none," not "we did not check." If you did not check, use `confidence: "low"` and say so in `notes_en`.

---

## 3. Confidence rubric

Assign exactly one of `high` / `medium` / `low` / `unknown` to the record:

- **high** — Source is the airline's own official site in a language the extractor reads natively, the page directly addresses pet travel, every required field is populated from the page (or explicitly absent), and no internal contradictions were found.
- **medium** — Official site, but: page is a summary that links to a PDF not retrieved, OR translation was required from a non-primary language, OR 1–2 fields are `unknown` for unobvious reasons, OR the page's own "last updated" is older than 18 months.
- **low** — Information was assembled from multiple sub-pages, OR the policy was unclear and required interpretation, OR more than 2 fields are `unknown`, OR the source page is a help-center article rather than a policy page.
- **unknown** — Could not access the source, or the source page does not actually state pet policy.

A record with `confidence: low` is still publishable; it signals to downstream users that they should re-verify before acting.

---

## 4. Validation (run after extraction, before commit)

The extractor MUST run these checks and either fix the record or downgrade `confidence`:

1. **Enum check.** All enum fields contain only allowed values.
2. **Contradiction check.**
   - If `cabin_allowed = "no"` AND `checked_baggage_allowed = "no"` AND `cargo_allowed = "no"`, then `pet_type` MUST be `[]`.
   - If `pet_type = []`, then all three transport enums MUST be `"no"`.
   - If `weight_limit.value` is set, `weight_limit.unit` MUST NOT be `null`.
   - If any of `length_cm` / `width_cm` / `height_cm` is set, all three SHOULD be set; if not, `notes_en` must explain.
3. **Unit check.** No mixed-unit text inside numeric fields. (e.g., `"8 kg or 18 lb"` is not allowed in `value`; pick one and put the other in `notes_en`.)
4. **URL check.** `official_url` returns HTTP 200 within the last fetch cycle. If 4xx/5xx, mark `confidence: "low"` and add to `notes_en`.
5. **Date check.** `last_checked_at` is not in the future and not more than 14 days older than the run date.

If a hard validation fails and cannot be repaired, do not delete — emit the record with `confidence: "low"` and a `notes_en` entry beginning `"VALIDATION:"`.

---

## 5. LLM extraction prompt (canonical)

`scripts/extract.js` should send the following system prompt to the LLM, with the page text and target schema appended:

> You are a structured-data extractor for the `pet_airlines` open dataset. You will receive the text of one airline's official pet-travel policy page. Output exactly one JSON object conforming to the v0.1 schema described below. Follow these rules without exception:
>
> 1. If a field is not stated on the page, output `"unknown"` (string fields), `null` (numeric fields), or `[]` (array fields). Never infer from outside knowledge.
> 2. Preserve the source's units and currencies verbatim. The only exception is carrier dimensions: convert inches to cm, but record the original in `notes_en`.
> 3. Express seasonal restrictions as ISO date ranges in `notes_en` and `notes_ja`. Never write "summer" alone.
> 4. Mirror facts between `*_en` and `*_ja` fields. Translate, do not add. If a fact appears only in one language and you are not confident in the translation, copy it verbatim with a `[ja: untranslated]` or `[en: untranslated]` tag.
> 5. Set `confidence` per the rubric in §3. Be honest. A low-confidence record is more useful than a wrong high-confidence one.
> 6. Run the validation checks in §4. If a check fails, fix the record if possible; otherwise prefix `notes_en` with `"VALIDATION:"` and downgrade `confidence`.
> 7. Output only the JSON object — no prose, no Markdown fences, no commentary.
>
> Schema: <inject schema definition here>
> Source URL: <inject URL>
> Source language: <inject language>
> Page text: <inject text>

The extractor wrapper code is responsible for parsing the JSON response, re-running validation server-side, and writing the result to `data/pet_airlines_v0.1.json`.

---

## 6. Out of scope for v0.1

Explicitly NOT extracted in this version (deferred to v0.2 / v0.3):

- Country-specific quarantine and import requirements (handled per-destination, not per-airline).
- Per-route pricing tables.
- Airport-specific facility info (relief areas, vet on call).
- Travel-class differences (e.g., First-class pet allowances).

These are tracked in [docs/roadmap.md](../../docs/roadmap.md).
