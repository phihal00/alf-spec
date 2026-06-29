# ALF v1 — Azail Layer Format Specification

**Version:** 1.1.0-draft  
**Status:** Draft — not yet published  
**Canonical home:** dotalf.com/spec/v1 (pending launch)  
**Maintained by:** Azail AB  
**Licence:** The format specification is open. See §16.

---

## Preamble

The Azail Layer Format (ALF) is an open file format for encoding portable, model-neutral expert reasoning layers. An `.alf` file is a JSON document that tells an AI model how a domain expert thinks — the evaluative sequences, the things non-experts don't know to ask, the procedural constraints, and the private data to connect at runtime.

ALF is not a model. It is not a prompt. It is a structured container for cognitive expertise that any conformant platform can read and assemble into context before a query reaches the model. The same `.alf` file works with Claude, GPT-4, Gemini, or any other language model.

This document is the complete normative specification for ALF v1. A developer reading this document alone should be able to implement a conformant ALF reader, writer, and validator without asking anyone for clarification.

---

## Table of contents

1. [Concepts and definitions](#1-concepts-and-definitions)
2. [File format](#2-file-format)
3. [Top-level structure](#3-top-level-structure)
4. [Section 1: metadata](#4-section-1-metadata)
5. [Section 2: trigger_conditions](#5-section-2-trigger_conditions)
6. [Section 3: required_inputs](#6-section-3-required_inputs)
7. [Section 4: enhancement_templates](#7-section-4-enhancement_templates)
8. [Section 5: conversation_map](#8-section-5-conversation_map)
9. [Section 6: blind_spots](#9-section-6-blind_spots)
10. [Section 7: error_correction_patterns](#10-section-7-error_correction_patterns)
11. [Section 8: unknown_unknowns](#11-section-8-unknown_unknowns)
12. [Section 9: search_tags](#12-section-9-search_tags)
13. [Section 10: audit_requirements](#13-section-10-audit_requirements)
14. [Conformance](#14-conformance)
15. [The open/private boundary](#15-the-openprivate-boundary)
16. [Versioning](#16-versioning)
17. [Licence](#17-licence)
18. [Reference examples](#18-reference-examples)

---

## 1. Concepts and definitions

**Layer.** A named, structured encoding of expert reasoning for a specific domain or task. A layer is the unit of ALF — one `.alf` file contains one layer.

**Layer type.** One of four values: `reasoning`, `procedure`, `context`, `data`.

**Conformant platform.** A software system that reads `.alf` files and assembles their contents into context for a language model, following the rules in this specification.

**Enhancement template.** A named instruction that reshapes how the model responds to a class of query. The primary mechanism by which a layer modifies model behaviour.

**Unknown unknown.** A named insight representing something an expert would raise that the non-expert would not know to ask. The most intellectually distinctive element of a layer.

**Trigger condition.** A semantic rule that determines whether a layer should activate for a given query.

**Layer pointer.** A reference to an external data source, used exclusively in `data` layer types. The pointer is in the file; the data is not.

**Distribution scope.** The intended audience for a layer: `public` (anyone), `org` (a single issuing organisation), or `private` (the authoring user only). Declared in the file; enforced by the platform.

**Content version.** A semver string tracking the version of the layer's intellectual content, independent of the ALF format version.

---

## 2. File format

- File extension: `.alf`
- Encoding: UTF-8
- Format: JSON (RFC 8259)
- MIME type: `application/vnd.azail.alf+json`
- Maximum file size (conformant readers must support): 512 KB
- The file must be valid JSON. A conformant reader must reject files that are not valid JSON.
- The file must contain exactly one JSON object at the root level.
- Key names are case-sensitive. All keys defined in this specification are lowercase with underscores.
- Unknown keys at any level must be ignored by conformant readers (forward-compatible extension mechanism).

---

## 3. Top-level structure

An ALF file contains nine required top-level keys and one optional top-level key.

```json
{
  "metadata": { ... },
  "trigger_conditions": [ ... ],
  "required_inputs": [ ... ],
  "enhancement_templates": [ ... ],
  "conversation_map": [ ... ],
  "blind_spots": [ ... ],
  "error_correction_patterns": [ ... ],
  "unknown_unknowns": [ ... ],
  "search_tags": { ... },
  "audit_requirements": { ... }
}
```

`audit_requirements` is optional. All other keys are required.

---

## 4. Section 1: metadata

Identifies the layer and its structural properties.

### Required fields

| Field | Type | Constraints |
|---|---|---|
| `alf_version` | string | `"1.0"` or `"1.1"` for v1 files |
| `layer_id` | string | UUID v4, set at creation, never changed |
| `title` | string | Max 120 chars. Describes what the layer does, not who authored it |
| `description` | string | 1–3 sentences, max 500 chars. Written for a non-expert reader |
| `domain` | string | Max 60 chars. Free text (e.g. `"clinical medicine"`, `"corporate finance"`) |
| `layer_type` | enum | `reasoning` \| `procedure` \| `context` \| `data` |
| `display_type` | enum | `expert` \| `editorial` \| `institutional` \| `community` |
| `attribution_label` | string | Max 120 chars. Practitioner name, editorial entity, institution, or handle |
| `author_handle` | string\|null | Platform author identifier. `null` for editorial and community layers |
| `created_at` | string | ISO 8601 date (YYYY-MM-DD) |
| `updated_at` | string | ISO 8601 date, ≥ created_at |
| `language` | string | BCP 47 tag (e.g. `"en"`, `"sv"`) |
| `licence` | string | Free text (e.g. `"CC BY 4.0"`, `"Azail Marketplace Terms"`) |

### Optional fields (v1.1)

| Field | Type | Constraints |
|---|---|---|
| `distribution_scope` | enum | `public` \| `org` \| `private`. Default: `public`. See §4.1. |
| `issuer_org_id` | string\|null | UUID v4 or verified domain string identifying the owning enterprise tenant. `null` for public/community layers. See §4.2. |
| `content_version` | string | Semver string (e.g. `"1.0.0"`) tracking version of the layer's intellectual content independently of `alf_version`. See §4.3. |

### §4.1 distribution_scope

`distribution_scope` declares the intended audience for this layer.

- `public` — The layer may be indexed, installed, and invoked by any user on any conformant platform. This is the default if the field is absent.
- `org` — The layer is restricted to users belonging to the issuing organisation identified by `issuer_org_id`. A conformant platform must not make an `org`-scoped layer available to users outside that organisation. `issuer_org_id` must be present and non-null when `distribution_scope` is `org`.
- `private` — The layer is restricted to the authoring user only. A conformant platform must not share or transfer a `private`-scoped layer to any other user.

The distribution scope is a declaration of intent. The platform is responsible for enforcing it. A conformant platform must honour declared scope. A platform that ignores `distribution_scope` is not conformant.

### §4.2 issuer_org_id

`issuer_org_id` is a machine-readable identifier for the enterprise tenant that owns the layer. It should be a UUID v4 matching the platform's internal organisation record, or a verified domain string (e.g. `"acme-corp.com"`). Its presence enables automated billing settlement, enterprise-wide revocation on contract expiry, and deterministic ownership attribution in multi-tenant deployments. It is `null` for public, editorial, and community layers.

### §4.3 content_version

`content_version` tracks the version of the layer's intellectual content using semver (`MAJOR.MINOR.PATCH`). It is distinct from `alf_version`, which tracks the format schema.

- Incrementing `PATCH` (e.g. `1.0.0` → `1.0.1`): editorial corrections, rewording, no change to reasoning logic.
- Incrementing `MINOR` (e.g. `1.0.0` → `1.1.0`): new enhancement templates, blind spots, or unknown unknowns added; existing logic unchanged.
- Incrementing `MAJOR` (e.g. `1.0.0` → `2.0.0`): material change to reasoning logic, trigger conditions, or conversation map. Enterprise deployments should treat a MAJOR increment as requiring re-validation before continued use.

When `content_version` is present, `updated_at` must also be updated to reflect the date of the version increment.

### Layer types

- `reasoning` — encodes a method of thinking; how an expert evaluates a situation
- `procedure` — encodes a sequence of steps with a defined correct order
- `context` — carries standing facts, precedents, or constraints
- `data` — primarily a pointer to an external runtime data source

### Display types

- `expert` — authored or co-built with a named, verified practitioner
- `editorial` — produced by an editorial process from public sources
- `institutional` — produced by or for a named institution
- `community` — contributed by the open community under a permissive licence

---

## 5. Section 2: trigger_conditions

Defines when the layer should activate. **Minimum: 2. Maximum: 10.**

```json
{
  "trigger_conditions": [
    {
      "type": "semantic",
      "description": "Plain-language description of what triggers this condition (max 200 chars)",
      "examples": ["Example query 1", "Example query 2"]
    }
  ]
}
```

**`type`** — `"semantic"` | `"keyword"` | `"context"`. Prefer `"semantic"`.  
**`examples`** — 2–5 example queries, max 150 chars each.

---

## 6. Section 3: required_inputs

What the platform needs before the layer can fire usefully. **Minimum: 2. Maximum: 8.**

```json
{
  "required_inputs": [
    {
      "id": "snake_case_identifier",
      "label": "Human-readable name (max 60 chars)",
      "description": "What this is and why the layer needs it (max 200 chars)",
      "source": "user_provided",
      "required": true
    }
  ]
}
```

**`source`** — `"user_provided"` | `"inferred"` | `"profile"` | `"runtime"`

### §6.1 pointer_config (v1.1, optional)

For inputs with `source: "runtime"`, an optional `pointer_config` object may be provided to describe the external data connection.

```json
{
  "id": "customer_record",
  "label": "Customer record",
  "description": "Live customer data fetched at prompt assembly time",
  "source": "runtime",
  "required": true,
  "pointer_config": {
    "endpoint": "https://api.example.com/customers/{customer_id}",
    "auth_scheme": "bearer",
    "cache_allowed": false
  }
}
```

| Field | Type | Constraints |
|---|---|---|
| `endpoint` | string | URI template for the data source. May contain `{variable}` placeholders. Max 500 chars. |
| `auth_scheme` | enum | `bearer` \| `api_key` \| `mtls` \| `none`. Describes the credential type; the credential itself is never stored in the file. |
| `cache_allowed` | boolean | Whether a conformant platform may cache the fetched data between prompt assemblies. |

**`cache_allowed: false` is required for conformant enterprise data layers.** A conformant platform must treat data fetched for a `runtime` input with `cache_allowed: false` as strictly ephemeral: held in memory for the duration of one prompt assembly and discarded immediately after the model call completes. The data must not be logged, stored, or re-used across calls. This implements the data minimisation principle under GDPR and matches the Azail platform's Path A runtime data guarantee.

A platform that caches data when `cache_allowed` is `false` is not conformant.

---

## 7. Section 4: enhancement_templates

The primary mechanism of a layer. **Minimum: 5. Maximum: 20.**

```json
{
  "enhancement_templates": [
    {
      "id": "snake_case_identifier",
      "name": "Human-readable name (max 80 chars)",
      "activation": "Condition under which this template fires (max 200 chars)",
      "modifier": "Instruction added to model context (max 500 chars)",
      "expected_outcome": "What the user experiences after this fires (max 200 chars)"
    }
  ]
}
```

**Quality standard:** A template that adds a checklist the user could have found themselves is not a conformant enhancement template. The test: does this template cause the model to do something a thoughtful but non-expert user could not have prompted for on their own?

---

## 8. Section 5: conversation_map

Multi-turn structured sequences. **Minimum: 2 steps. Maximum: 10.**

```json
{
  "conversation_map": [
    {
      "step": 1,
      "name": "Step name (max 60 chars)",
      "purpose": "What this step establishes (max 200 chars)",
      "model_instruction": "Instruction at this step (max 400 chars)",
      "next_step_condition": "Condition to proceed, or null for final step"
    }
  ]
}
```

---

## 9. Section 6: blind_spots

Things the average person does not know they do not know. Surfaced proactively. **Minimum: 3. Maximum: 12.**

```json
{
  "blind_spots": [
    {
      "id": "snake_case_identifier",
      "title": "Short gap name (max 80 chars)",
      "description": "What the gap is and why it matters (max 300 chars)"
    }
  ]
}
```

---

## 10. Section 7: error_correction_patterns

Common mistakes and how to catch them. **Minimum: 2. Maximum: 10.**

```json
{
  "error_correction_patterns": [
    {
      "id": "snake_case_identifier",
      "error": "Description of the mistake (max 200 chars)",
      "detection": "How to recognise it (max 200 chars)",
      "correction": "What the model should do instead (max 300 chars)"
    }
  ]
}
```

---

## 11. Section 8: unknown_unknowns

The intellectual core of a layer. Named insights an expert would raise that the non-expert would not know to ask. **Minimum: 4. Maximum: 12.**

```json
{
  "unknown_unknowns": [
    {
      "label": "On [3–5 word description] —",
      "content": "The insight itself (100–400 chars)"
    }
  ]
}
```

### Label format — hard constraint

`label` must follow exactly: `"On [3–5 word description] —"`

The word count between "On " and " —" must be exactly 3 to 5 words. This is a hard constraint, not a style preference.

Valid: `"On anchoring errors in diagnosis —"` (4 words)  
Invalid: `"On anchoring —"` (2 words)  
Invalid: `"On the way anchoring errors manifest —"` (6 words)

### Content standards

- 100–400 characters
- Must open with the insight itself, not a definition of the label
- Minimum 2 entries across the layer must cite a named academic researcher, clinical framework, or named institutional body
- Never cite productivity bloggers, founder opinions, podcast hosts, or Wikipedia as primary sources

### Quality standard

Both must be true: a senior practitioner reads it and says "yes, that's exactly the thing people miss" AND a non-expert reads it and says "I would never have known to ask about that." If a thoughtful generalist would have included this in their own prompt, it is not an unknown unknown.

---

## 12. Section 9: search_tags

Semantic and taxonomic tags for platform matching.

```json
{
  "search_tags": {
    "primary": ["direct semantic descriptors"],
    "secondary": ["adjacent concepts"],
    "contextual": ["situational and user-context descriptors"]
  }
}
```

**`primary`** — Minimum 8 tags. Direct semantic match = strong activation signal.  
**`secondary`** — Minimum 6 tags. Secondary match alone is a weaker signal.  
**`contextual`** — Minimum 5 tags. User types, scenarios, adjacent activities.

All tags: lowercase, max 60 chars, no duplicates across groups.

---

## 13. Section 10: audit_requirements

Optional. Signals what audit logging a conformant platform should perform when invoking this layer. The presence of this section does not create binding legal obligations on the platform from within the file — the platform's data processing agreement governs retention periods and legal obligation. This section provides a machine-readable signal for ecosystem partners and a human-readable note for compliance reviewers.

```json
{
  "audit_requirements": {
    "log_layer_invocations": true,
    "log_template_ids_fired": true,
    "log_pointer_fetches": true,
    "platform_obligations_note": "Informational prose for compliance reviewers (max 500 chars)"
  }
}
```

| Field | Type | Description |
|---|---|---|
| `log_layer_invocations` | boolean | Whether the platform should log each time this layer is activated for a query. |
| `log_template_ids_fired` | boolean | Whether the platform should log which specific enhancement template IDs fired for each invocation. |
| `log_pointer_fetches` | boolean | Whether the platform should log each external data fetch performed for `runtime` inputs (endpoint and timestamp only; fetched data must never be logged). |
| `platform_obligations_note` | string | Human-readable note for compliance reviewers. Max 500 chars. Not machine-enforceable. |

**Important:** `log_pointer_fetches: true` means the platform logs that a fetch occurred (endpoint, timestamp, success/failure). It does not mean the platform logs the content of the fetched data. Logging fetched content from a `runtime` input with `cache_allowed: false` is a conformance violation regardless of this field.

All four fields are optional within `audit_requirements`. An empty `audit_requirements` object `{}` is valid.

> **Security Officer note:** This section should be reviewed by the platform's security and legal teams before deploying layers with `audit_requirements` in regulated contexts (EU AI Act, HIPAA, FCA). The `platform_obligations_note` is informational; it cannot substitute for a formal DPA or processing agreement.

---

## 14. Conformance

### Conformant ALF v1 file

A file is conformant if:

1. It is valid JSON (RFC 8259)
2. It contains all nine required top-level sections
3. All required fields are present and of the correct type
4. Minimum counts met: trigger_conditions ≥ 2, required_inputs ≥ 2, enhancement_templates ≥ 5, conversation_map ≥ 2, blind_spots ≥ 3, error_correction_patterns ≥ 2, unknown_unknowns ≥ 4, search_tags.primary ≥ 8, search_tags.secondary ≥ 6, search_tags.contextual ≥ 5
5. `metadata.alf_version` is `"1.0"` or `"1.1"`
6. `metadata.layer_id` is a valid UUID v4
7. `metadata.layer_type` is one of the four defined values
8. `metadata.display_type` is one of the four defined values
9. All `unknown_unknowns[].label` values follow the `"On [3–5 word] —"` format
10. If `metadata.distribution_scope` is `"org"`, then `metadata.issuer_org_id` must be present and non-null
11. For any `required_inputs` item with `source: "runtime"` and `pointer_config.cache_allowed: false`, the platform must treat fetched data as ephemeral

### Conformant ALF v1 reader

A reader is conformant if:

1. It accepts any conformant ALF v1 file without error
2. It ignores unknown keys at any level (forward compatibility)
3. It rejects files that are not valid JSON
4. It does not require fields beyond those defined in this spec
5. It honours `metadata.distribution_scope` — it does not make `org` or `private` layers available outside their declared scope
6. It treats data fetched for `runtime` inputs with `cache_allowed: false` as strictly ephemeral

### Validation

A JSON Schema is published at `spec/schema-v1.json` in this repository and at dotalf.com/spec/v1/schema.json.

---

## 15. The open/private boundary

The most common misconception about ALF: "open format" does not mean "layers are free to copy." The format is open. Individual layer content is governed by `metadata.licence`. And the thing that makes a specific layer instance valuable is largely not in the file at all.

**What is in the file:**
- The nine sections — structure, labels, enhancement templates, unknown unknowns, search tags
- Distribution scope and org identity (v1.1)
- Content version (v1.1)

**What is not in the file:**
- The author's verified practitioner credential (platform-level signal, not a file attribute)
- Layer performance metrics (computed from live usage, not stored in the file)
- For data layers: the actual data (only a pointer is in the file; data stays on the institution's infrastructure)
- The ongoing refinement (a copied file is a snapshot; the live layer is continuously updated)
- Authentication credentials for runtime data sources (auth_scheme declares the type; credentials are provisioned by the platform)

**The structural protection:** A copied `.alf` file carries the structure but not the four properties that make the original valuable: verified authorship, continuous refinement, live platform metrics, and private runtime data.

**Important:** ALF v1 does not define any encryption-at-rest or per-install watermarking mechanism. The protection is structural, not cryptographic. Implementations that claim ALF provides cryptographic copy protection are misrepresenting this specification.

---

## 16. Versioning

### Format versioning

`metadata.alf_version` carries major.minor as a string: `"1.0"`, `"1.1"`, `"2.0"`.

- **Minor versions** (1.0 → 1.1): additive only. New optional fields, new enum values. v1.0 files remain valid. v1.0 readers must handle v1.x files (unknown keys are ignored per §2).
- **Major versions** (1.x → 2.0): may include breaking changes. A v2.0 reader is not required to process v1.x files.

### Content versioning

When `metadata.content_version` is present, it tracks the layer's intellectual content using semver independently of `alf_version`. See §4.3 for increment rules.

v1.1 additions (DR-598–DR-602): `distribution_scope`, `issuer_org_id`, `content_version` added to metadata as optional fields; `pointer_config` added to `required_inputs` for `runtime` sources; `audit_requirements` added as optional top-level section.

---

## 17. Licence

This specification is published under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

The format is open. No licence is required to implement a conformant reader, writer, or validator. No licence is required to write a `.alf` file.

Individual `.alf` file content is governed by `metadata.licence`.

---

## 18. Reference examples

Two reference examples are provided in the `examples/` directory:

**[examples/minimal-reference.alf.json](../examples/minimal-reference.alf.json)**  
A fully conformant ALF v1 file with minimal content, suitable for testing reader implementations. Synthetic — not a quality layer.

**[examples/clinical-differential-reasoning.alf.json](../examples/clinical-differential-reasoning.alf.json)**  
The benchmark quality example. Clinical differential diagnosis for chest pain triage. Demonstrates production-quality content across all nine sections. Free to use under CC BY 4.0.

---

*ALF v1.1 Specification — draft — Azail AB — 2026-06-26*  
*Published under CC BY 4.0. The format is open.*  
*Canonical location (pending launch): dotalf.com/spec/v1*  
*v1.1 additions: DR-598, DR-599, DR-600, DR-601, DR-602*
