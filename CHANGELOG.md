# Changelog

All notable changes to the ALF specification are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).
Versioning follows [Semantic Versioning](https://semver.org/).

---

## [1.0.0-draft] — 2026-06-25

### Added
- Initial draft specification for ALF v1
- Nine-section JSON structure: metadata, trigger_conditions, required_inputs, enhancement_templates, conversation_map, blind_spots, error_correction_patterns, unknown_unknowns, search_tags
- Four layer types: reasoning, procedure, context, data
- Four display types: expert, editorial, institutional, community
- Conformance criteria for files and readers
- JSON Schema (spec/schema-v1.json)
- Minimal reference example (examples/minimal-reference.alf.json)
- Benchmark quality example: Clinical Differential Reasoning for chest pain triage (examples/clinical-differential-reasoning.alf.json)
- llms.txt for AI assistant discoverability
- Community contribution process (community-layers/)

---

*ALF is maintained by Azail AB. Canonical home: dotalf.com*
