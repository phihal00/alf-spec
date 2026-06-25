# Contributing to ALF

Thank you for your interest in contributing to the Azail Layer Format.

## Ways to contribute

**Spec feedback** — Open an issue using the Spec Question or Feature Proposal template.

**Implementation reports** — Built a conformant reader, writer, or validator? Open an Implementation Report issue.

**Community layers** — Contribute an open `.alf` layer to the community directory.

## Contributing a community layer

1. Write a conformant `.alf` file. Validate it against [spec/schema-v1.json](spec/schema-v1.json).
2. Set `metadata.display_type` to `"community"` and `metadata.licence` to `"CC BY 4.0"`.
3. Name the file descriptively: `domain-layer-type.alf.json` (e.g. `corporate-finance-reasoning.alf.json`).
4. Open a pull request adding the file to `community-layers/`.
5. A maintainer will review for structural conformance and content quality bar.

## Content quality bar

- All `unknown_unknowns` follow the `"On [3–5 word] —"` label format
- At least 2 `unknown_unknowns` carry citations to named academic researchers, clinical frameworks, or named institutional bodies
- Enhancement templates cause the model to do something a thoughtful non-expert could not have prompted for themselves
- The layer encodes genuine domain expertise, not a prompt wrapper

## Licence

By contributing, you agree that your contributions are licensed under CC BY 4.0 unless explicitly stated otherwise.

The specification itself is © Azail AB, licensed under CC BY 4.0.
