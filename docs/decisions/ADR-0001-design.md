# ADR-0001 — qsag-canonical Design

| Field | Value |
|---|---|
| **ADR Number** | 0001 |
| **Title** | qsag-canonical Design |
| **Status** | Accepted |
| **Date Proposed** | 2026-05-08 |
| **Date Accepted** | 2026-05-08 |
| **Author** | Muhammad Zaid Naeem (Maintainer, AIXYBER TECH LTD trading as Neoxyber) |
| **Approver** | Muhammad Zaid Naeem (Sole Director, AIXYBER TECH LTD) |
| **Supersedes** | None (this is the first artefact-level ADR) |
| **Superseded By** | None |
| **Related Master ADRs** | ADR-0031 (Open-Source Cryptographic and Audit Substrate Programme) in the private neoxyber-qsag repository |
| **Scope** | qsag-canonical v0.1 design surface |
| **Licence** | Apache License 2.0 |

---

## 1. Context

### 1.1 What this ADR locks

This ADR is the design decision record for `qsag-canonical`, the second artefact in the Q-SAG open-source substrate programme. It commits to the v0.1 functional surface, the standards conformance posture, the threat model, the testing discipline (bidirectional clause-to-test traceability), and the integration pattern with sibling Q-SAG artefacts and downstream consumers.

The master programme ADR (ADR-0031 in the private neoxyber-qsag repository) commits to the existence of this artefact and to its place in the eleven-artefact substrate programme. This ADR-0001 commits to *how* the artefact is designed.

Per the locked ADR discipline in ADR-0031 §4.1, this ADR-0001 must be committed before any source code lands in `src/`. This is being committed as part of the initial repository scaffolding.

### 1.2 Why this artefact exists

JSON canonicalisation is load-bearing in cryptographic systems. Two implementations of RFC 8785 that disagree on Unicode normalisation, IEEE 754 number formatting, or property ordering produce different byte sequences for the same logical input. Different bytes produce different hashes. Different hashes produce signatures that cannot be cross-verified. The audit chain breaks silently — there is no error, just a quiet divergence between what the signer signed and what the verifier reconstructs.

The Q-SAG substrate signs canonicalised JSON in many places:

- Every Signed Statement submitted to `qsag-anchors` is canonicalised before signing.
- Every audit-pack export from `qsag-evidence` (forthcoming) is canonicalised before hashing.
- Every delegation token in Q-SAG main is canonicalised before issuance.
- Every checkpoint published by the Transparency Service is canonicalised.
- Every receipt verification step canonicalises the original statement.

A bug in canonicalisation cascades to every dependent operation. Q-SAG cannot afford ambiguity.

The existing Python implementations of RFC 8785 are correct in the common case. Trail of Bits' `rfc8785` package (Apache 2.0) is the most commonly cited. It does not, however, publish a bidirectional clause-to-test traceability matrix — the explicit mapping from each clause of RFC 8785 to the test cases that exercise it, and from each test case back to the clause(s) it validates. The Go reference implementation `json-canon` (Apache 2.0, Anders Rundgren) does publish such a matrix. `qsag-canonical` ports `json-canon`'s test discipline to Python, plus extends it with edge cases relevant to AI-agent audit chains.

### 1.3 Forcing functions

The timeline pressure from ADR-0031 §1.4 applies to this artefact specifically because every other Q-SAG artefact downstream depends on canonicalisation correctness. `qsag-canonical` must ship before `qsag-anchors` v0.1 publishes its first PyPI release, because `qsag-anchors` will declare a runtime dependency on `qsag-canonical`. The week-2 target in the substrate programme timeline therefore makes `qsag-canonical` v0.1 a hard prerequisite, not a parallel work item.

### 1.4 Build philosophy applied

Per ADR-0031 §1.5 and §2.4, this artefact wraps and ports rather than reimplementing from scratch:

- The core algorithm follows RFC 8785 directly. There is no derivative algorithm.
- The test corpus imports `json-canon`'s public test vectors verbatim (Apache 2.0 attribution preserved in NOTICE).
- The Python implementation uses the standard library `json` module for parsing where possible; the canonicalisation layer is implemented on top.
- The bidirectional traceability matrix is the artefact's distinguishing engineering contribution.
- A drop-in API compatibility with Trail of Bits' `rfc8785` package is preserved for the core `canonicalize()` call.

---

## 2. Decision

### 2.1 v0.1 functional surface

`qsag-canonical` v0.1 ships the following Python package with public API:

- **`qsag_canonical.canonicalize(value, *, strict=True) -> bytes`** — the primary public function. Takes any JSON-serialisable Python object and returns the canonical UTF-8 byte sequence. Strict mode (the default) rejects inputs that contain non-canonical numeric forms, non-Unicode strings, or other deviations that RFC 8785 cannot represent unambiguously.
- **`qsag_canonical.canonicalize_stream(value, writer, *, strict=True) -> None`** — streaming variant for large inputs. Writes canonical bytes to a file-like object that supports `write(bytes) -> int`.
- **`qsag_canonical.verify_canonical(input_bytes) -> bool`** — given a byte sequence, returns whether the bytes are already in canonical form. Useful for input validation in audit pipelines and for catching upstream re-canonicalisation bugs.
- **`qsag_canonical.NonCanonicalInputError`** — the exception raised by strict-mode operations when input cannot be canonicalised unambiguously. Subclass of `ValueError` for compatibility with Trail of Bits' `rfc8785`.

The package is type-annotated (Python 3.12+ with `from __future__ import annotations`), documented with sphinx-style docstrings, and covered by unit, integration, and conformance tests.

### 2.2 Strict-mode behaviour by default

Strict mode is enabled by default. Strict-mode behaviour:

- Numbers must round-trip through the IEEE 754 double-precision representation per RFC 8785 §3.2.2. Numbers that cannot be exactly represented (e.g., values outside the IEEE 754 normal range) raise `NonCanonicalInputError`.
- Strings must be valid Unicode. Surrogate halves and other invalid sequences raise `NonCanonicalInputError`.
- Object keys must be unique. Duplicates raise `NonCanonicalInputError` (RFC 8785 §3.2.3 requires sorted unique-key output but is silent on input deduplication; we choose to fail fast in strict mode).
- The special values `NaN` and `Infinity` are rejected. JSON has no representation for them; RFC 8785 inherits this restriction.

Permissive mode (`strict=False`) is supported for legacy migration paths only. It accepts duplicate keys (last-write-wins), normalises non-canonical numeric forms where possible, and emits `NonCanonicalInputError` only for genuinely unrepresentable inputs. Permissive-mode output is documented as not byte-identical to strict-mode output for the same input.

### 2.3 Conformance test corpus (locked for v0.1)

The v0.1 conformance test corpus comprises **three layers**:

1. **Official RFC 8785 test vectors** (4 vectors, hosted as a normative annex to the RFC). All must pass byte-identical.
2. **Imported `json-canon` (Go) test corpus** (Apache 2.0, attribution preserved). Imported as JSON test fixtures with each fixture annotated to its originating `json-canon` test name and the RFC 8785 clauses it exercises.
3. **Q-SAG-specific edge cases** newly authored for this artefact. The minimum v0.1 set includes:
   - Non-ASCII identifiers in agent IDs (UTF-8 multi-byte characters in object keys).
   - Deeply-nested delegation trees (recursion-depth limits exercised).
   - Large IEEE 754 numbers from telemetry (boundary tests near `Number.MAX_SAFE_INTEGER` and around the double-precision boundary).
   - Malformed UTF-8 in adversarial inputs (rejection-tested in strict mode; permissive-mode behaviour explicitly documented).
   - Property-ordering edge cases involving Unicode-equivalent forms (NFC vs NFD normalisation).

Every test fixture in every layer is mapped bidirectionally:

- **Forward**: each fixture cites the RFC 8785 clause(s) it exercises.
- **Backward**: each RFC 8785 clause's `traceability.md` entry lists every fixture that exercises it.

Building or maintaining the traceability matrix is a CI-gated step: pull requests that add new fixtures without updating the matrix fail the build. This matrix is the artefact's distinguishing engineering contribution.

### 2.4 Threat model (v0.1)

Documented in detail in [THREAT_MODEL.md](THREAT_MODEL.md) (forthcoming with v0.1). Summary of v0.1 commitments:

**Defended against:**

- Canonicalisation divergence on conformant input (i.e., the v0.1 test corpus passes byte-identical against `json-canon` Go and the official RFC 8785 vectors).
- Parser denial-of-service on adversarial input: stack overflow on deeply-nested input is bounded by an explicit recursion-depth limit; quadratic-behaviour attacks on string keys are mitigated by a documented input-size limit.
- Type confusion in the Python implementation: strict typing enforced via mypy, no untyped boundaries.
- Silent data loss: duplicate keys raise in strict mode rather than silently last-write-wins.

**Not defended against (acknowledged limits):**

- Memory exhaustion on legitimately-large inputs. The streaming API (`canonicalize_stream`) is the v0.1 mitigation; users of the non-streaming API must size-limit their inputs.
- Side-channel attacks against the Python implementation. v0.1 is pure Python; no constant-time guarantees are claimed. Constant-time canonicalisation is reserved for v1.0 (PyO3 / Rust acceleration path).
- Adversarial-input DoS where adversary controls both the input and the parsing budget. Input-size limits are documented but enforcement is the integrating application's responsibility.
- Theoretical attacks on RFC 8785 itself. Out of scope for this artefact; report such issues to the IETF JSON WG.

### 2.5 Integration points

- **With `qsag-anchors`**: every Signed Statement submitted to `qsag-anchors` is canonicalised through `qsag-canonical` before signing. `qsag-anchors` declares a runtime dependency on `qsag-canonical` once both are at v0.1.
- **With `qsag-evidence`** (forthcoming sibling artefact): every audit-pack export is canonicalised through `qsag-canonical` before hashing for inclusion in the regulator-facing report.
- **With `qsag-pq-primitives`** (forthcoming sibling artefact): no direct dependency. `qsag-canonical` is a pure-Python pre-cryptographic primitive; PQC operations happen downstream after canonicalisation.
- **With Q-SAG main** (private neoxyber-qsag repository): once `qsag-canonical` v0.1 ships, Q-SAG main migrates from its current Trail of Bits' `rfc8785` dependency to `qsag-canonical`. The migration is API-compatible per §1.4 above.
- **With Microsoft AGT and Asqav SDK**: indirect — these consume canonicalised JSON via the Transparency Service API, not via `qsag-canonical` directly.

### 2.6 Out of scope for v0.1

The following are explicitly out of scope for v0.1 and reserved for later versions:

- **Asynchronous API** (`async def canonicalize_async`). Reserved for v0.5; the v0.1 streaming API uses synchronous file-like objects.
- **Native Rust acceleration** via PyO3. Reserved for v0.5; v0.1 is pure Python. Performance characteristics are documented but not benchmarked competitively.
- **Permissive-mode being the default**. Reserved for never; strict-mode-by-default is a programme-wide commitment.
- **Streaming for very large objects** (multi-gigabyte inputs). The v0.1 streaming API supports inputs that fit in memory. True multi-gigabyte streaming is reserved for v0.5+.
- **Constant-time canonicalisation**. Reserved for v1.0 once PyO3 / Rust acceleration lands.
- **Custom number-format strategies**. v0.1 implements RFC 8785 §3.2.2 exactly and offers no configurability; this is intentional.
- **Backward-compatibility shims for non-RFC 8785 canonicalisation schemes** (e.g., older "JSON Canonical Form" variants). Reserved for v0.5 if there is genuine demand; until then, applications wanting alternate schemes use other libraries.

---

## 3. Consequences

### 3.1 What this ADR makes true

- The v0.1 functional surface is locked. Adding to or changing the surface requires a new ADR superseding the relevant clause.
- Strict mode is the default and is the supported posture for production use. Permissive mode is for legacy migration only.
- The bidirectional clause-to-test traceability matrix is a CI-gated requirement. Pull requests that add behaviour without updating the matrix fail the build.
- API-compatibility with Trail of Bits' `rfc8785` for the core `canonicalize()` call is a v0.1 commitment.

### 3.2 What this ADR makes required

- Every release of `qsag-canonical` from v0.1 onward must include the official RFC 8785 test vectors and the imported `json-canon` corpus, all passing byte-identical.
- Every release must publish a CycloneDX 1.6 SBOM and SLSA Level 3 build provenance.
- THREAT_MODEL.md, STANDARDS.md, and `docs/conformance/traceability.md` must be present and accurate at v0.1 ship time.
- The v0.1 release must include a worked end-to-end example demonstrating: (1) canonicalising a complex JSON object, (2) verifying byte-stability against a recorded test vector, (3) hashing the canonical bytes for downstream signing.

### 3.3 What this ADR makes false (or removes)

- Any prior assumption that `qsag-canonical` would offer permissive-mode-by-default behaviour. It will not.
- Any prior assumption that v0.1 would include native acceleration. It will not — that is v0.5.

### 3.4 Risks documented

- **Risk of subtle divergence from `json-canon`.** The Go reference implementation is the de facto cross-language conformance target. Mitigation: import its full test corpus, run on every CI build, and require byte-identical output. Any divergence found post-release is a P0 incident.
- **Risk of input-size DoS via the non-streaming API.** Mitigation: documented input-size limits, integrating-application enforcement. v0.5 will add enforced limits at the library boundary.
- **Risk of Python `json` parser bugs propagating.** Mitigation: the parsing layer is decoupled from canonicalisation; if a Python `json` bug surfaces, we can switch to an alternative parser without breaking the canonicalisation API.
- **Risk that strict-mode rejection breaks existing applications during migration from Trail of Bits' `rfc8785`.** Mitigation: documented migration guide; `strict=False` available for legacy codepaths during transition.

---

## 4. Compliance and conformance

This artefact aligns with:

- **IETF RFC 8785** — JSON Canonicalisation Scheme (the primary specification).
- **ECMA-262** — ECMAScript Language Specification (object semantics).
- **ECMA-404** — JSON Data Interchange Syntax (the underlying JSON grammar).
- **IEEE 754** — Standard for Floating-Point Arithmetic (number representation per RFC 8785 §3.2.2).
- **IETF RFC 8259** — The JavaScript Object Notation (JSON) Data Interchange Format (the input grammar).
- **Unicode Standard, latest version** — for string handling and normalisation tests.

Detailed clause-level mappings will live in [STANDARDS.md](STANDARDS.md) and the bidirectional matrix in [docs/conformance/traceability.md](docs/conformance/traceability.md), both to be committed at v0.1.

---

## 5. References

- ADR-0031 — Open-Source Cryptographic and Audit Substrate Programme (private neoxyber-qsag repository, commit 21490f1d4e046040330c2e360cfe16cb39e702a2).
- IETF RFC 8785 — JSON Canonicalisation Scheme (JCS).
- IETF RFC 8259 — JSON Data Interchange Format.
- ECMA-262 — ECMAScript Language Specification.
- ECMA-404 — JSON Data Interchange Syntax.
- IEEE Std 754-2019 — IEEE Standard for Floating-Point Arithmetic.
- Anders Rundgren — `json-canon` (Go reference implementation of RFC 8785) — https://github.com/cyberphone/json-canonicalization
- Trail of Bits — `rfc8785` (Python implementation) — https://github.com/trailofbits/rfc8785

---

## 6. Approval

Approved by the sole director of AIXYBER TECH LTD, Muhammad Zaid Naeem, on 8 May 2026. This ADR is committed GPG-signed and DCO-signed alongside the initial repository scaffolding for `qsag-canonical`.

---

*© 2026 AIXYBER TECH LTD (Company No. 16826340), trading as Neoxyber. Registered in England and Wales. ICO Registration: ZC071900. Released under the Apache License, Version 2.0.*
