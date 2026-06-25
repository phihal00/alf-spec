# ALF v1 — Azail Layer Format Specification

**Version:** 1.0.0-draft  
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
13. [Conformance](#13-conformance)
14. [The open/private boundary](#14-the-openprivate-boundary)
15. [Versioning](#15-versioning)
16. [Licence](#16-licence)
17. [Reference examples](#17-reference-examples)

---

## 1. Concepts and definitions

**Layer.** A named, structured encoding of expert reasoning for a specific domain or task. A layer is the unit of ALF — one `.alf` file contains one layer.

**Layer type.** One of four values: `reasoning`, `procedure`, `context`, `data`.

**Conformant platform.** A software system that reads `.alf` files and assembles their contents into context for a language model, following the rules in this specification.

**Enhancement template.** A named instruction that reshapes how the model responds to a class of query. The primary mechanism by which a layer modifies model behaviour.

**Unknown unknown.** A named insight representing something an expert would raise that the non-expert would not know to ask. The most intellectually distinctive element of a layer.

**Trigger condition.** A semantic rule that determines whether a layer should activate for a given query.

**Layer pointer.** A reference to an external data source, used exclusively in `data` layer types. The pointer is in the file; the data is not.

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

An ALF file contains exactly nine top-level keys. All nine are required for a fully conformant file.

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
  "search_tags": { ... }
}
```

---

## 4. Section 1: metadata

Identifies the layer and its structural properties. All fields are required.

| Field | Type | Constraints |
|---|---|---|
| `alf_version` | string | `"1.0"` for v1 files |
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

## 13. Conformance

### Conformant ALF v1 file

A file is conformant if:

1. It is valid JSON (RFC 8259)
2. It contains all nine required top-level sections
3. All required fields are present and of the correct type
4. Minimum counts met: trigger_conditions ≥ 2, required_inputs ≥ 2, enhancement_templates ≥ 5, conversation_map ≥ 2, blind_spots ≥ 3, error_correction_patterns ≥ 2, unknown_unknowns ≥ 4, search_tags.primary ≥ 8, search_tags.secondary ≥ 6, search_tags.contextual ≥ 5
5. `metadata.alf_version` is `"1.0"`
6. `metadata.layer_id` is a valid UUID v4
7. `metadata.layer_type` is one of the four defined values
8. `metadata.display_type` is one of the four defined values
9. All `unknown_unknowns[].label` values follow the `"On [3–5 word] —"` format

### Conformant ALF v1 reader

A reader is conformant if:

1. It accepts any conformant ALF v1 file without error
2. It ignores unknown keys at any level (forward compatibility)
3. It rejects files that are not valid JSON
4. It does not require fields beyond those defined in this spec

### Validation

A JSON Schema is published at `spec/schema-v1.json` in this repository and at dotalf.com/spec/v1/schema.json.

---

## 14. The open/private boundary

The most common misconception about ALF: "open format" does not mean "layers are free to copy." The format is open. Individual layer content is governed by `metadata.licence`. And the thing that makes a specific layer instance valuable is largely not in the file at all.

**What is in the file:**
- The nine sections — structure, labels, enhancement templates, unknown unknowns, search tags

**What is not in the file:**
- The author's verified practitioner credential (platform-level signal, not a file attribute)
- Layer performance metrics (computed from live usage, not stored in the file)
- For data layers: the actual data (only a pointer is in the file; data stays on the institution's infrastructure)
- The ongoing refinement (a copied file is a snapshot; the live layer is continuously updated)

**The structural protection:** A copied `.alf` file carries the structure but not the four properties that make the original valuable: verified authorship, continuous refinement, live platform metrics, and private runtime data.

**Important:** ALF v1 does not define any encryption-at-rest or per-install watermarking mechanism. The protection is structural, not cryptographic. Implementations that claim ALF provides cryptographic copy protection are misrepresenting this specification.

---

## 15. Versioning

`metadata.alf_version` carries major.minor as a string: `"1.0"`, `"1.1"`, `"2.0"`.

- **Minor versions** (1.0 → 1.1): additive only. New optional fields, new enum values. v1.0 files remain valid. v1.0 readers must handle v1.x files (unknown keys are ignored per §2).
- **Major versions** (1.x → 2.0): may include breaking changes. A v2.0 reader is not required to process v1.x files.

---

## 16. Licence

This specification is published under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

The format is open. No licence is required to implement a conformant reader, writer, or validator. No licence is required to write a `.alf` file.

Individual `.alf` file content is governed by `metadata.licence`.

---

## 17. Reference examples

Two reference examples are provided in the `examples/` directory:

**[examples/minimal-reference.alf.json](../examples/minimal-reference.alf.json)**  
A fully conformant ALF v1 file with minimal content, suitable for testing reader implementations. Synthetic — not a quality layer.

**[examples/clinical-differential-reasoning.alf.json](../examples/clinical-differential-reasoning.alf.json)**  
The benchmark quality example. Clinical differential diagnosis for chest pain triage. Demonstrates production-quality content across all nine sections: five enhancement templates, five blind spots, three error correction patterns, five unknown unknowns grounded in named clinical research (Groopman, Kahneman/Klein, IRAD). Free to use under CC BY 4.0.

---

*ALF v1 Specification — draft — Azail AB — 2026-06-25*  
*Published under CC BY 4.0. The format is open.*  
*Canonical location (pending launch): dotalf.com/spec/v1*
