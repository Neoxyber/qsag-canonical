# qsag-canonical

> Strict RFC 8785 (JSON Canonicalisation Scheme) implementation in Python with full bidirectional clause-to-test traceability. Part of the Q-SAG open-source substrate programme.

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)
[![Status](https://img.shields.io/badge/status-v0.0.0%20scaffolding-orange.svg)](#status)
[![Standards](https://img.shields.io/badge/standards-RFC%208785%20%7C%20ECMA--262%20%7C%20ECMA--404%20%7C%20IEEE%20754-informational.svg)](#standards)

`qsag-canonical` is the second artefact in the Q-SAG open-source substrate programme — a family of post-quantum cryptographic and audit primitives for AI-agent governance, designed for the 2027-and-onwards horizon. The substrate is maintained by AIXYBER TECH LTD (trading as Neoxyber) under [Apache License 2.0](LICENSE).

---

## What this artefact does

`qsag-canonical` deterministically converts a JSON value into a single, byte-stable canonical form per IETF RFC 8785 (JSON Canonicalisation Scheme, JCS). Two independently-developed implementations of `qsag-canonical` consuming the same input must produce the same byte sequence — and that byte sequence is the input to any subsequent hash, signature, or anchoring operation.

At v0.1, the published surface includes:

- **`qsag_canonical.canonicalize(value) -> bytes`** — the primary public function. Takes any JSON-serialisable Python object and returns the canonical byte sequence.
- **`qsag_canonical.canonicalize_stream(value, writer)`** — streaming variant for large inputs; writes canonical bytes to a file-like object.
- **`qsag_canonical.verify_canonical(input_bytes) -> bool`** — given a byte sequence, returns whether it is already in canonical form (useful for input validation in audit pipelines).
- **Strict-mode behaviour by default** — non-canonical inputs raise `NonCanonicalInputError` rather than silently re-canonicalising; an explicit `strict=False` flag opts into permissive parsing for legacy migration paths.

## Why this gap matters

Canonicalisation sounds boring. It is also load-bearing.

If two implementations of RFC 8785 disagree on how to canonicalise the same JSON object — different Unicode normalisation, different IEEE 754 number formatting, different property ordering — they produce different byte sequences. The hashes diverge. Signatures made by one implementation cannot be verified by the other. The audit chain breaks silently.

The Q-SAG substrate signs canonicalised JSON in dozens of places. Every Signed Statement submitted to `qsag-anchors` is canonicalised before signing. Every audit-pack export from `qsag-evidence` is canonicalised before hashing. Every ML-DSA-44 / SLH-DSA signature anywhere in the substrate ultimately commits to canonicalised bytes. A bug in canonicalisation cascades to every dependent operation.

The existing Python implementations of RFC 8785 (notably Trail of Bits' `rfc8785` package) are correct but do not publish a bidirectional clause-to-test traceability matrix. The Go reference implementation `json-canon` does. This artefact ports `json-canon`'s test discipline to Python, plus extends it with edge cases relevant to AI-agent audit chains (non-ASCII identifiers in agent IDs, deeply-nested delegation trees, large IEEE 754 numbers from telemetry, malformed UTF-8 in adversarial inputs).

## Status

This repository is at **v0.0.0 — scaffolding**. The repository structure, ADR discipline, security posture, and contribution guidelines are in place. The v0.1 implementation is in progress, targeting **May 2026** per the locked Q-SAG substrate programme roadmap.

Q-SAG itself is under active development. This artefact is being built as part of that ongoing work. Production use is not recommended until v0.5 at the earliest, by which point the conformance test corpus will have been audited against the official RFC 8785 vectors, the imported `json-canon` corpus, and at least one independent Python interoperability target.

## Installation

> Not yet available. v0.1 will publish to PyPI as `qsag-canonical`.

## Quick start

> Not yet available. v0.1 will include a minimal end-to-end example demonstrating: (1) canonicalising a complex JSON object, (2) verifying the byte-stability against a recorded test vector, (3) hashing the canonical bytes for downstream signing.

## Documentation

- [Threat model](THREAT_MODEL.md) — what this artefact defends and what it does not (forthcoming with v0.1)
- [Standards alignment](STANDARDS.md) — RFC 8785, ECMA-262, ECMA-404, IEEE 754 mappings (forthcoming with v0.1)
- [Architecture](docs/architecture.md) — design overview (forthcoming with v0.1)
- [Architecture Decision Records](docs/decisions/) — ADR-0001 design decision and onward
- [Conformance test vectors](docs/conformance/test-vectors/) — official RFC 8785 vectors plus the imported `json-canon` corpus (forthcoming with v0.1)
- [Clause-to-test traceability matrix](docs/conformance/traceability.md) — the bidirectional mapping between RFC 8785 clauses and test cases (forthcoming with v0.1)

## Standards

`qsag-canonical` aligns with the following standards:

- **IETF RFC 8785** — JSON Canonicalisation Scheme (the primary specification this artefact implements)
- **ECMA-262** — ECMAScript Language Specification (defines the object semantics RFC 8785 builds on)
- **ECMA-404** — JSON Data Interchange Syntax (the underlying JSON grammar)
- **IEEE 754** — Standard for Floating-Point Arithmetic (defines the number representation RFC 8785 §3.2.2 references)

Detailed clause-level mappings live in [STANDARDS.md](STANDARDS.md) (forthcoming with v0.1).

## Differences from existing implementations

| Implementation | Language | Strict by default | Clause-to-test traceability | Q-SAG-specific edge cases |
|---|---|---|---|---|
| **qsag-canonical** | Python | Yes | Yes (bidirectional) | Yes |
| Trail of Bits `rfc8785` | Python | Yes | Partial | No |
| `json-canon` | Go | Yes | Yes (bidirectional) | No |
| Various JS libraries | JavaScript | Variable | No | No |

`qsag-canonical` is API-compatible with Trail of Bits' `rfc8785` for the core `canonicalize()` call, so existing Python users can substitute without code changes. The added value is the test discipline and the audit-chain edge cases.

## Related artefacts

`qsag-canonical` is one of eleven sibling artefacts in the Q-SAG open-source substrate programme. The full list, in shipping order:

1. [qsag-anchors](https://github.com/Neoxyber/qsag-anchors) — federated SCITT Transparency Service primitives
2. **qsag-canonical** *(this repository)* — strict RFC 8785 JCS implementation in Python
3. [pg-qsag-audit-tle](https://github.com/Neoxyber/pg-qsag-audit-tle) — pg_tle SQL-only Postgres extension (closes the seven-year SHA3 gap)
4. [pg-qsag-audit](https://github.com/Neoxyber/pg-qsag-audit) — pgrx-native Postgres extension
5. [qsag-pq-primitives](https://github.com/Neoxyber/qsag-pq-primitives) — PyO3 wrapper, profile-aware dispatch
6. [qsag-evidence](https://github.com/Neoxyber/qsag-evidence) — regulator-facing audit-pack export (EU AI Act Annex IV, DORA, CRA SRP, C2PA 2.3)
7. [qsag-ocsf](https://github.com/Neoxyber/qsag-ocsf) — OCSF v1.8.0 ai_operation event emitter
8. [qsag-cascade](https://github.com/Neoxyber/qsag-cascade) — verifiable cascading-kill primitive
9. [qsag-coalition](https://github.com/Neoxyber/qsag-coalition) — graph-based coalition / Sybil detection
10. [qsag-aibom](https://github.com/Neoxyber/qsag-aibom) — CycloneDX 1.6 + SPDX 3.0 + EAT-AI emitter
11. [qsag-confidential](https://github.com/Neoxyber/qsag-confidential) — TEE attestation receipt format

## Contributing

We welcome contributions from the cryptography, RFC 8785, and AI-agent-governance communities. Before contributing, please read:

- [Contributing guide](CONTRIBUTING.md) — DCO sign-off, PR process, ADR discipline
- [Code of Conduct](CODE_OF_CONDUCT.md) — Contributor Covenant 2.1
- [Security policy](SECURITY.md) — coordinated vulnerability disclosure

All commits must be DCO-signed (`git commit -s`). Maintainer commits are additionally GPG-signed.

New test vectors (especially edge cases that demonstrate or guard against canonicalisation bugs) are some of the most valuable contributions to this artefact.

## Security

Security disclosures: **security@neoxyber.com**

PGP key fingerprint: `A65AF5B7F02C9EB5B98023D70DB861BBF30F0D7B`

Fetch the public key:

```
gpg --keyserver keys.openpgp.org --recv-keys A65AF5B7F02C9EB5B98023D70DB861BBF30F0D7B
```

For full disclosure procedure, acknowledgement window, and safe-harbour terms, see [SECURITY.md](SECURITY.md).

## Maintainer

Maintainer: **Muhammad Zaid Naeem (Neoxyber)** — zaidnaeem@neoxyber.com

## Legal

`qsag-canonical` is licensed under the [Apache License 2.0](LICENSE). See the [NOTICE](NOTICE) file for required attribution and third-party component acknowledgements.

For company facts (legal entity, registration, ICO), see [COMPANY_FACTS.md](COMPANY_FACTS.md).

The hosted Q-SAG demo at [qsag.neoxyber.com](https://qsag.neoxyber.com) is provided free of charge for research, education, and testing purposes only. It is not a commercial product, has no SLA, and is not suitable for production deployment on safety-critical systems.

---

© 2026 AIXYBER TECH LTD (Company No. 16826340), trading as Neoxyber.
Registered in England and Wales. ICO Registration: ZC071900.
Released under the Apache License, Version 2.0.
