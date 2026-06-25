# ALF — Azail Layer Format

The open specification for portable, model-neutral expert reasoning layers.

**Canonical home:** [dotalf.com](https://dotalf.com) *(pending launch)*

---

## What is ALF?

ALF is a JSON file format that encodes how a domain expert thinks — the evaluative sequences, the things non-experts don't know to ask, the procedural constraints, and the private data to connect at runtime.

An `.alf` file works with Claude, GPT-4, Gemini, or any other language model. It is not a prompt. It is a structured container for cognitive expertise that any conformant platform can read and assemble into context before a query reaches the model.

## The format is open

The spec is here. Read it, implement it, write `.alf` files. No licence required. No gate.

Marketplace layers built on ALF are commercial assets governed by their own licence terms. The format is not.

## Repository structure

```
alf-spec/
├── README.md
├── CONTRIBUTING.md
├── CHANGELOG.md
├── llms.txt
├── spec/
│   ├── alf-v1.md          ← The full normative specification
│   └── schema-v1.json     ← JSON Schema for validation
├── examples/
│   ├── minimal-reference.alf.json
│   └── clinical-differential-reasoning.alf.json
└── community-layers/
    └── README.md
```

## Quick start

1. Read [spec/alf-v1.md](spec/alf-v1.md) — the full normative spec
2. Validate your files against [spec/schema-v1.json](spec/schema-v1.json)
3. See [examples/](examples/) for a minimal test file and a production-quality benchmark example
4. Contribute a community layer — see [CONTRIBUTING.md](CONTRIBUTING.md)

## Licence

This specification is published under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). The format is open. Individual `.alf` file content is governed by the `metadata.licence` field of each file.

Maintained by [Azail AB](https://azail.ai).
