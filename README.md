# qsag-canonical

> Canonical serialisation, pre-hash, and binding primitives for post-quantum signing in Python. Part of the Q-SAG open-source substrate programme.

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)
[![Status](https://img.shields.io/badge/status-pre--v0.1%20scaffolding-orange.svg)](#status)
[![Standards](https://img.shields.io/badge/standards-RFC%208785%20%7C%20RFC%209881%20%7C%20FIPS%20204-informational.svg)](#standards)

`qsag-canonical` is part of the **Q-SAG open-source substrate programme** — a research effort to build a set of post-quantum cryptographic and audit primitives for AI-agent governance. The substrate is stewarded by AIXYBER TECH LTD (trading as Neoxyber) under the [Apache License 2.0](LICENSE).

This repository is in the structuring and scaffolding stage. The code, tests, and documentation will be built up iteratively under public version control. There is no shipped release yet.

---

## What this library is for

`qsag-canonical` provides the small set of primitives that sit between an application-level data structure and a post-quantum signature.

In plain terms: before any data can be signed in a way that is verifiable, deterministic, and survives long-term audit, several things must happen in a strict order — serialisation must be deterministic, the value to be signed must be derived in a standard way, the binding between the signed value and the signature must be recorded, and the policy under which the signing was performed must be captured. These steps are individually simple and individually well-specified by published standards. They are not currently available as a coherent, fail-loud, audit-aligned Python library.

`qsag-canonical` is intended to be that library.

The intended public surface at the first numbered release consists of three functions:

```python
sign_payload(record, key, policy=None) -> (signature, binding, hybrid_signature_or_None)
verify_payload(record, signature, binding, pubkey, hybrid_pubkey=None) -> bool
prepare_external_signing(record, public_key, policy=None) -> (signing_input, binding)
```

`prepare_external_signing` is the bridge to hardware security modules (HSMs) that perform ML-DSA signing using the External µ scheme defined in FIPS 204 and standardised for X.509 in RFC 9881. The intended scope is to make External µ usable from Python correctly, fail-loud, and standards-aligned.

---

## Why this gap exists

Three things have converged.

First, NIST finalised the post-quantum signature standards (FIPS 204 ML-DSA, FIPS 205 SLH-DSA) in August 2024. Second, the supporting X.509 algorithm-identifier standard (RFC 9881) was published in October 2025. Third, hardware security modules and managed key services have begun supporting ML-DSA with External µ — a pre-hash variant designed so that an HSM does not have to receive the entire message it is signing.

In Python, no library brings these three together cleanly. The pieces exist — there are RFC 8785 canonicalisers, there are wrappers around lower-level cryptographic libraries, there are ad-hoc helpers for hardware-backed signing — but nothing exposes the External µ flow end-to-end with strict canonicalisation, a stable binding record, policy capture, and fail-loud error handling. The result is that Python applications that want to sign canonical JSON with ML-DSA on an HSM end up writing their own glue, often with subtle bugs that are hard to detect until the signatures need to be verified by a different party.

`qsag-canonical` exists to be that glue, written once, carefully, in the open, with traceable test vectors and an explicit threat model.

---

## What this library is not

It is important to be honest about scope.

This library is not a re-implementation of ML-DSA or any other post-quantum algorithm. The cryptographic mathematics is deferred to vetted upstream implementations (see `qsag-pq-primitives` in the related-libraries list below). `qsag-canonical` is the layer above that, concerned with canonicalisation, pre-hashing, binding, and policy — not the lattice arithmetic.

It is not an agent-identity system. Complete agent-identity systems require orchestration, lifecycle management, capability descriptors, kill-switch coordination, and many other components. Substrate libraries provide the cryptographic primitives that such systems can be built on top of.

It is not a discovery or migration tool. Tools that scan codebases or infrastructure for legacy cryptography and produce Cryptographic Bills of Materials (CBOMs) are a separate, well-served category. `qsag-canonical` does not compete with them; it emits CycloneDX 1.6 SBOMs (including a CBOM section) as a release artefact, which those tools can consume.

It is not a hosted service. There is no SaaS component. There is no telemetry. The library runs entirely within the application that uses it.

---

## Locked v0.1 scope

The first numbered release will consist of exactly six modules:

| Module | Purpose |
|---|---|
| `canon` | Strict RFC 8785 (JSON Canonicalisation Scheme) implementation. Vendored from a reference implementation lineage. Fail-loud on inputs that cannot be canonicalised. |
| `prehash` | Three-variant dispatch (`pure`, `external_mu`, `internal_prehash`). Algorithm-agnostic so the same surface can be extended to future signature schemes. |
| `binding` | A Pydantic v2 schema that records what was signed, under what policy, and with what algorithm. Designed so that a v1 binding record remains verifiable under any future v2 or later schema version. |
| `policy` | YAML profile blocks for the four primary post-quantum cryptographic regimes referenced by current national guidance. |
| `errors` | A fail-loud exception hierarchy. No silent fallbacks. Any cryptographic step that cannot proceed must raise. |
| `compat.kms` | Integration with cloud key management services that support ML-DSA with External µ. The only compat module in v0.1. |

Other compat modules (PKCS#11, content-provenance metadata, supply-chain transparency, selective disclosure, agent-to-agent protocols, model-context protocols) are explicitly out of scope for v0.1. They are deferred to later minor releases so that v0.1 can ship narrow.

---

## Status

This repository is in the pre-v0.1 structuring phase. The work currently in progress is:

- Locking the architecture and committing the structure document (in the master programme repository)
- Migrating cryptographic key material off developer disk to FIPS 140-3 Level 3 key storage
- Producing the Phase 0 documentation set (structure, research log, constraint coverage matrix, competitive landscape, security policy)
- Writing the v0.1 module scaffolding

No code beyond scaffolding is being claimed as functional yet. There is no Python package on PyPI. There is no Zenodo DOI. There is no published test vector pass log. These will appear when the v0.1 release tag is cut and the release hygiene checklist is satisfied.

Statements such as "first Python implementation of X" appear in this repository's documentation in the future tense. They become citable only when v0.1 ships with passing test vectors against the relevant reference suites.

---

## Standards alignment

The intended alignment, to be verified by published test results when v0.1 ships:

- **NIST FIPS 204** (ML-DSA): the pre-hash module implements the External µ formula from §6
- **IETF RFC 9881** (Algorithm Identifiers for ML-DSA in X.509 PKI, October 2025): the binding module uses RFC 9881 algorithm identifiers
- **IETF RFC 8785** (JSON Canonicalisation Scheme): the canon module is a strict implementation, with the silent-fallback path explicitly removed and disclosed in the changelog
- **ECMA-262** (ECMAScript Language Specification): the number-formatting in canon follows the ECMA-262 algorithm referenced by RFC 8785
- **ECMA-404** (JSON Data Interchange Syntax): the underlying JSON grammar
- **IEEE 754** (Floating-Point Arithmetic): the number representation referenced by RFC 8785 §3.2.2
- **NIST CSWP 39** (Crypto Agility, December 2025): policy module structure designed for algorithm rotation by configuration, not code

Detailed clause-level traceability matrices will be published with v0.1 in `docs/conformance/`.

---

## How this fits the broader substrate programme

The Q-SAG open-source substrate programme is a planned set of ten libraries. Each is independently usable; together they cover post-quantum primitives, canonical signing, trust anchors, audit chains, and a set of reserved future libraries that activate when relevant external standards finalise.

The four front-of-priority libraries, in dependency order:

1. **qsag-pq-primitives** — wrappers and dependency-checklist verification over vetted upstream post-quantum implementations
2. **qsag-canonical** *(this library)* — canonicalisation, pre-hash, binding, policy
3. **qsag-anchors** — post-quantum trust-anchor primitives, including RFC 9794 hybrid-chain taxonomy
4. **pg-qsag-audit** — append-only SHA-3 audit chain primitives for PostgreSQL, in dual TLE and pgrx flavours

Six reserved libraries are structured but not yet implemented. Each will activate when its triggering external standard, hardware availability, or peer-reviewed academic result lands:

5. `qsag-threshold` · 6. `qsag-composite` · 7. `qsag-attest` · 8. `qsag-fn-dsa` · 9. `qsag-incident` · 10. `qsag-zk-attest`

The overall programme is documented in the substrate-programme overview at the steward's `.well-known` location.

---

## Honest gaps

This section is mandatory for every substrate library and will grow over time as more is learned.

- **Side-channel resistance** is a property of the underlying signing device, not of this library. When signing happens in software using a generic CPython interpreter, secret-dependent timing leakage cannot be ruled out. This library targets HSM-backed signing for production; software signing exists for development and testing only.
- **CPython memory model** does not allow reliable zeroisation of immutable `bytes` objects. Key material handling in pure Python is best-effort; the canonical pattern is to perform key operations inside an HSM and never have private-key bytes resident in the Python heap.
- **JSON canonicalisation** is fragile in the small. RFC 8785 is precise but the gap between "precise specification" and "every implementation produces the same bytes" is where audit chains have historically broken. This library uses traced clause-to-test mapping and an oracle vector suite drawn from a reference implementation lineage.

---

## Contributing

Contributions are welcome from cryptographers, standards practitioners, and Python engineers with relevant experience. Before contributing, please read:

- [Contributing guide](CONTRIBUTING.md) — Developer Certificate of Origin sign-off, contributor ladder, AI-assistance disclosure
- [Code of Conduct](CODE_OF_CONDUCT.md) — Contributor Covenant 2.1
- [Security policy](SECURITY.md) — coordinated vulnerability disclosure

All commits must be DCO-signed (`git commit -s`). Maintainer commits are additionally GPG-signed.

The most valuable contributions at this stage are: test vectors that exercise canonicalisation edge cases; prior-art citations the maintainer has missed in standards or academic literature; threat-model review.

---

## Security

Security disclosures: **`security@aixybertech.com`**

PGP key fingerprint: `A65AF5B7F02C9EB5B98023D70DB861BBF30F0D7B`

Fetch the public key:

```
gpg --keyserver keys.openpgp.org --recv-keys A65AF5B7F02C9EB5B98023D70DB861BBF30F0D7B
```

For the full disclosure procedure, acknowledgement window, and safe-harbour terms, see [SECURITY.md](SECURITY.md).

---

## Maintainer

Maintainer: **Muhammad Zaid Naeem** — `zaidnaeem@aixybertech.com`

---

## Legal

`qsag-canonical` is licensed under the [Apache License 2.0](LICENSE). See the [NOTICE](NOTICE) file for required attribution and third-party component acknowledgements.

---

© 2026 AIXYBER TECH LTD (Company No. 16826340), trading as Neoxyber. Registered in England and Wales. ICO Registration: ZC071900. Released under the Apache License, Version 2.0.
