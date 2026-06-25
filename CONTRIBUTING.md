# Contributing to alf-spec

Thank you for your interest in the Azail Layer Format. Contributions are welcome in four forms: spec questions, implementation reports, feature proposals, and community layers.

---

## How decisions are made

Azail AB maintains the specification. Community input shapes it, but Azail reviews and accepts all changes. This is a stewarded open standard. Spec changes go through a public proposal process (issue → discussion → decision → PR). No spec change is accepted without a prior issue.

---

## Issue templates

Use the templates in `.github/ISSUE_TEMPLATE/`:

- **Spec question** — if you are implementing a reader or writer and the spec is unclear
- **Implementation report** — if you have built a conformant tool and want to report your experience
- **Feature proposal** — if you want to propose a change or addition to the spec

---

## Contributing a community layer

Community layers live in `community-layers/`. Requirements:

1. **Conformant.** Must pass validation against `spec/schema-v1.json`.
2. **Licensed.** `metadata.licence` must be `"CC BY 4.0"` or another open licence.
3. **`display_type` is `"community"`.** Expert or editorial display types are not accepted.
4. **Quality standard.** `unknown_unknowns` must meet the spec quality bar — named insights with at least 2 citations. Tips and reminders do not qualify.
5. **Original.** Your own work or properly attributed open-licensed material.
6. **Filename.** `community-layers/<domain>-<short-title>.alf.json` in lowercase with hyphens.

### Process

1. Fork this repository
2. Add your `.alf.json` to `community-layers/`
3. Validate: `npx ajv validate -s spec/schema-v1.json -d community-layers/your-file.alf.json`
4. Open a PR with the title: `[community layer] <layer title>`
5. Azail will review within 2 weeks

---

## Proposing a spec change

1. Open a **Feature Proposal** issue
2. Allow 2 weeks for community discussion
3. If accepted, Azail will indicate readiness for a PR
4. Submit a PR modifying `spec/alf-v1.md` and `spec/schema-v1.json` together
5. PRs modifying the spec without a prior accepted issue will be closed

---

*Azail AB · dotalf.com*
