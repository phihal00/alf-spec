# ALF Specification Changelog

## v1.1.0-draft (2026-06-26)

Enterprise readiness additions. All changes are optional fields — full backwards compatibility with v1.0 files maintained.

### Added

- `metadata.distribution_scope` (enum: `public` | `org` | `private`, default `public`) — DR-598
- `metadata.issuer_org_id` (string|null) — machine-readable enterprise tenant identifier — DR-599
- `metadata.content_version` (semver string) — tracks intellectual content version independently of format version — DR-600
- `required_inputs[].pointer_config` object for `source: runtime` inputs — fields: `endpoint`, `auth_scheme`, `cache_allowed` — DR-601
- `audit_requirements` top-level optional section — fields: `log_layer_invocations`, `log_template_ids_fired`, `log_pointer_fetches`, `platform_obligations_note` — DR-602
- Conformance rule: platform must honour `distribution_scope`; `org`-scoped layers require non-null `issuer_org_id`
- Conformance rule: platform must treat data fetched with `cache_allowed: false` as strictly ephemeral
- `alf_version` pattern now accepts `"1.1"` in addition to `"1.0"`

### Changed

- §14 (Conformance) updated with two new platform conformance requirements
- §15 (Open/private boundary) updated to reflect v1.1 metadata additions
- §16 (Versioning) updated with content versioning guidance and v1.1 changelog note
- Table of contents updated to include §13 (audit_requirements)

### Not changed

All nine required sections and their minimum/maximum counts are unchanged. v1.0 files remain fully conformant.

---

## v1.0.0-draft (2026-06-25)

Initial draft of the ALF v1 specification.
