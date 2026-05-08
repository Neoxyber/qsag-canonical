# Contributing to qsag-canonical

Thank you for your interest in contributing to `qsag-canonical`, part of the Q-SAG open-source substrate programme operated by AIXYBER TECH LTD (trading as Neoxyber).

This document explains how to contribute code, documentation, and ideas. It is short by design — the goal is to make contributing low-friction while keeping the project legally clean and technically sound.

---

## Before you start

1. **Read the [README](README.md)** to understand what this artefact does and where it sits in the broader Q-SAG substrate programme.
2. **Read the [Code of Conduct](CODE_OF_CONDUCT.md)** — Contributor Covenant 2.1. Reports route to `zaidnaeem@neoxyber.com`.
3. **Read the [Security Policy](SECURITY.md)** if your contribution relates to security. Vulnerabilities go to `security@neoxyber.com` (PGP encryption recommended), not to public GitHub issues.

---

## Ways to contribute

- **Report bugs.** Open a GitHub Issue with a clear title, a minimal failing JSON input, the canonical output you got, the canonical output you expected, and the version or commit hash you observed.
- **Propose features.** Open a GitHub Issue or Discussion describing the use case and rough design. For substantial changes (e.g., adding a new compatibility mode), expect to write or co-author an Architecture Decision Record (ADR) — see below.
- **Submit code.** Fork the repository, create a feature branch, push your changes, and open a Pull Request. Details below.
- **Improve documentation.** Same workflow as code. Documentation improvements are welcome and reviewed with the same care as source changes.
- **Improve test vectors.** New edge-case test inputs that demonstrate or guard against canonicalisation bugs are some of the most valuable contributions to a project like this.
- **Review pull requests.** Constructive technical review from anyone is welcome.

---

## Pull request workflow

### 1. Fork and clone

```
git clone https://github.com/<your-username>/qsag-canonical.git
cd qsag-canonical
git remote add upstream https://github.com/Neoxyber/qsag-canonical.git
```

### 2. Create a feature branch

Branch naming convention: `<type>/<short-description>`. Types: `feat/`, `fix/`, `docs/`, `test/`, `refactor/`, `chore/`.

```
git checkout -b fix/numeric-edge-case-negative-zero
```

### 3. Make your changes

- Keep commits focused. One logical change per commit. Multiple commits per PR are fine if each is independently coherent.
- Follow the existing code style. (Style guide will be expanded with v0.1.)
- Add or update tests for any code change. **Every behavioural change must be covered by a test that maps to a specific RFC 8785 clause or a documented edge case.**
- Update documentation if your change affects user-visible behaviour or the clause-to-test traceability matrix.

### 4. Sign your commits (DCO)

Every commit must include a `Signed-off-by:` line attesting that you have the right to contribute the code under the project's Apache 2.0 licence. This is the **Developer Certificate of Origin** (DCO) — see https://developercertificate.org for the full text.

Add the sign-off automatically by using the `-s` flag:

```
git commit -s -m "fix(numeric): handle negative-zero per RFC 8785 §3.2.2"
```

The DCO sign-off looks like this in the commit message:

```
Signed-off-by: Your Name <your.email@example.com>
```

The name and email must match a real identity you can be reached at. Pull requests with unsigned commits will not be merged.

We do **not** require a Contributor License Agreement (CLA). Contributors retain copyright to their contributions and licence them to the project under Apache 2.0 via the DCO sign-off.

### 5. Sign your commits (GPG, recommended)

GPG signing is encouraged for all contributors and required for maintainer commits. If you have a GPG key configured, add `-S` to your commit:

```
git commit -S -s -m "fix(numeric): handle negative-zero per RFC 8785 §3.2.2"
```

This produces commits that show as "Verified" on GitHub, providing cryptographic attestation that the commit came from you.

### 6. Push and open a PR

```
git push origin fix/numeric-edge-case-negative-zero
```

Then open a PR against `main` on GitHub. Fill in the PR template (forthcoming with v0.1) — at minimum:

- A clear summary of what changed and why.
- The specific RFC 8785 clause(s) the change implements or affects.
- Links to any related issues, ADRs, or external standards.
- A note on testing performed, including which test vectors were added or updated.
- A note on any breaking changes (we use semantic versioning, and a single canonicalisation bug fix may produce different bytes for previously-mishandled inputs).

### 7. Review

- The maintainer will review within roughly 7 days for non-trivial PRs.
- Discussion happens in line comments on the PR.
- Be open to feedback. Constructive disagreement is welcome and expected.
- Once approved, the maintainer will merge — typically as a squash-merge for clean history, occasionally as a merge commit when commit-by-commit history is meaningful.

---

## Architecture Decision Records (ADRs)

Substantial design changes — new public APIs, changes to canonicalisation behaviour, deviations from RFC 8785, additions of compatibility modes for non-strict JSON producers, anything that affects the threat model — must be accompanied by an ADR.

ADRs live in `docs/decisions/` and follow the format documented in [ADR-0001](docs/decisions/ADR-0001-design.md). The format is:

- Header table (number, title, status, dates, author, approver, supersedes, superseded by, related ADRs, scope)
- Context (why this decision is being made now)
- Decision (what is being committed to)
- Consequences (what becomes true / false / required after this decision)
- References (primary sources, prior ADRs, related issues)

ADR numbering is sequential. New ADRs in this repository start at the next available number. ADRs are never deleted — they are superseded by later ADRs that explicitly reference the predecessor.

If you propose a change that needs an ADR but you are not sure how to write one, open a Discussion or draft PR — the maintainer will help you co-author it.

---

## Standards and conformance

`qsag-canonical` aligns with the following standards:

- **IETF RFC 8785** — JSON Canonicalisation Scheme (the primary specification this artefact implements).
- **ECMA-262** — ECMAScript Language Specification (defines the object semantics RFC 8785 builds on, particularly for property ordering and number serialisation).
- **ECMA-404** — JSON Data Interchange Syntax (the underlying JSON grammar).
- **IEEE 754** — Standard for Floating-Point Arithmetic (defines the number representation that RFC 8785 §3.2.2 references).

Contributions that affect standards-conformance must:

- Cite the specific RFC 8785 clause (or supporting ECMA / IEEE clause) the change implements or affects.
- Include or update conformance tests in `tests/conformance/`.
- Pass the official RFC 8785 test vectors and the imported `json-canon` (Go reference implementation) test corpus.
- Update the bidirectional clause-to-test traceability matrix when adding new tests or behaviour.

The full standards alignment matrix lives in [STANDARDS.md](STANDARDS.md) (forthcoming with v0.1).

---

## Style and tooling (v0.1 forthcoming)

The following will be enforced once v0.1 ships:

- **Python**: `ruff` for linting, `black` for formatting, `mypy --strict` for type checking, `pytest` for tests.
- **Markdown**: `markdownlint` with the project's configuration.
- **Pre-commit hooks**: a `.pre-commit-config.yaml` will run all of the above plus secret scanning before each commit.
- **CI**: GitHub Actions matrix on Python 3.12 and 3.14, x86_64 and aarch64.

Until v0.1, contributors should follow the conventions of the existing code in their PR.

---

## Communication

- **GitHub Issues**: bugs, feature requests, focused technical discussions on a single change.
- **GitHub Discussions** (when enabled): broader design discussions, questions, ideas not yet a concrete proposal.
- **Email**: `contact@neoxyber.com` for general correspondence; `info@neoxyber.com` for public information; `security@neoxyber.com` for vulnerability disclosure (PGP recommended); `zaidnaeem@neoxyber.com` for Code of Conduct reports and director-direct correspondence.

We aim to acknowledge issues and PRs within 7 days. Substantive review may take longer for complex changes.

---

## Recognition

Contributors are credited in:

- The git history (DCO sign-off and GPG signature, where applicable).
- Release notes for the version in which their contribution shipped.
- The `CHANGELOG.md` entry for that release.
- The `CONTRIBUTORS.md` file (forthcoming with v0.1).

We do not currently operate a paid bug-bounty programme — see [SECURITY.md](SECURITY.md) for our roadmap on that.

---

## Legal summary

- All contributions are licensed under [Apache License 2.0](LICENSE).
- Contributors retain copyright to their contributions; the DCO sign-off licenses them to the project.
- The legal entity behind the project is AIXYBER TECH LTD (Company No. 16826340), trading as Neoxyber. Full company facts in [COMPANY_FACTS.md](COMPANY_FACTS.md).

---

*Maintainer: Muhammad Zaid Naeem (Neoxyber) — zaidnaeem@neoxyber.com*

*© 2026 AIXYBER TECH LTD (Company No. 16826340), trading as Neoxyber. Registered in England and Wales. ICO Registration: ZC071900. Released under the Apache License, Version 2.0.*
