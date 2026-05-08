# Security Policy

> **Coordinated Vulnerability Disclosure (CVD) for `qsag-canonical`** — part of the Q-SAG open-source substrate programme operated by AIXYBER TECH LTD (trading as Neoxyber).

We take security seriously. `qsag-canonical` is a cryptographic substrate primitive: every signature in the broader Q-SAG ecosystem ultimately commits to bytes produced by this library. A bug in canonicalisation that produces divergent bytes for the same input cascades silently into signature failures, audit-chain breaks, and unverifiable Receipts. Please disclose responsibly.

---

## Reporting a vulnerability

**Email:** `security@neoxyber.com`

**Encryption (strongly recommended):** Encrypt your report with our PGP key.

**PGP key fingerprint:** `A65AF5B7F02C9EB5B98023D70DB861BBF30F0D7B`

**Fetch the public key:**

```
gpg --keyserver keys.openpgp.org --recv-keys A65AF5B7F02C9EB5B98023D70DB861BBF30F0D7B
```

The key is an Ed25519 key issued to `Muhammad Zaid Naeem (AIXYBER TECH LTD) <zaidnaeem@neoxyber.com>`, valid until April 2028.

**Please include in your report:**

- A description of the vulnerability and the affected component(s).
- A minimal failing JSON input that demonstrates the bug.
- The canonical output `qsag-canonical` produces for that input.
- The canonical output you expected (with citation to the relevant RFC 8785 clause if applicable).
- The version, commit hash, or release tag where you observed the issue.
- Any potential impact you are aware of (e.g., known signature systems that rely on this library).
- Whether you have disclosed this elsewhere, and if so, when.
- How you would like to be credited (real name, handle, organisation, or anonymous).

**Please do not:**

- Open a public GitHub issue for security disclosures.
- Post details on social media, forums, or mailing lists before coordinated disclosure.
- Publish proof-of-concept inputs that produce signature divergence on production systems before remediation.

---

## What you can expect from us

| Stage | Target time | Notes |
|---|---|---|
| **Acknowledgement** | Within 72 hours | A human reply confirming receipt and assigning a tracking reference. |
| **Initial assessment** | Within 7 days | Severity triage, scope confirmation, and a target patch date. |
| **Status updates** | Every 14 days | Until the issue is resolved or otherwise closed. |
| **Coordinated disclosure window** | 90 days from acknowledgement | The default window for public disclosure of a fixed vulnerability. May be extended by mutual agreement for complex issues, or shortened for actively exploited vulnerabilities. |
| **Credit** | At public disclosure | We name reporters in release notes, the project changelog, and the GitHub Security Advisory unless the reporter prefers to remain anonymous. |

We commit to:

- Treating every report seriously, regardless of perceived severity.
- Not pursuing legal action against good-faith security research conducted in compliance with this policy.
- Working collaboratively with reporters on coordinated disclosure timing.
- Being transparent about our findings and the fix in public release notes once disclosure is appropriate.
- For canonicalisation-divergence bugs specifically: publishing the failing test vector in our conformance test corpus once the fix lands, so other RFC 8785 implementations can guard against the same class of bug.

---

## Safe harbour

We support good-faith security research. If you make a good-faith effort to comply with this policy, we will:

- Consider your research to be authorised in the meaning of the **UK Computer Misuse Act 1990 (Section 1)** and analogous legislation in your jurisdiction (including, for relevant cases, US Computer Fraud and Abuse Act and EU NIS2 / GDPR Art 32 considerations for security research).
- Not initiate civil or criminal proceedings against you for the research itself.
- Work with you if a third party (e.g., a hosting provider, CDN, or signature-verifying relying party) initiates legal action, to the extent we are able.

To benefit from this safe-harbour:

- Make a good-faith effort to avoid privacy violations, data destruction, and service disruption.
- Only test on infrastructure that you own, infrastructure provided by us for testing, or systems where you have explicit permission from the owner.
- Do not access, copy, modify, or download user data beyond the minimum necessary to demonstrate the vulnerability.
- Report the vulnerability to us promptly via the channel above and do not disclose publicly until we have had a reasonable time to remediate.

This safe-harbour does **not** apply if you:

- Violate any other applicable law (e.g., access systems you have no right to test, attempt to defraud, or harm a third party).
- Demand payment as a condition of disclosure (extortion).
- Knowingly disclose details of a vulnerability publicly before coordinated disclosure.

---

## Scope

**In scope:**

- The source code in this repository (`qsag-canonical`) at any released version or commit on the `main` branch.
- Any binary artefacts published by the maintainer to PyPI under the `qsag-canonical` name.
- Bugs in the canonicalisation algorithm itself: incorrect property ordering, incorrect Unicode handling, incorrect IEEE 754 number formatting, incorrect string escaping, incorrect handling of special values (`null`, `true`, `false`, NaN, Infinity).
- Denial-of-service vulnerabilities in the parser: stack-overflow on deeply-nested input, quadratic-behaviour on adversarial inputs, memory exhaustion on large inputs.
- Type-confusion or memory-safety issues in any C-extension or Rust-backed code path (relevant once v0.5+ adds native acceleration).

**Out of scope:**

- Vulnerabilities in upstream Python `json` module — please report those to CPython at https://bugs.python.org.
- Theoretical attacks on RFC 8785 itself — please report those to the IETF JSON WG.
- Vulnerabilities in third-party dependencies — please report to the relevant upstream project; we will track and update our pinned versions as upstream fixes ship.
- Social engineering, physical attacks, or denial-of-service attacks against our infrastructure.
- Vulnerabilities affecting versions of `qsag-canonical` that are no longer supported (typically anything older than the most recent stable minor release once we reach v1.0).
- Reports that consist only of automated scanner output without a working proof of concept.

---

## What is **not** a vulnerability

The following are not vulnerabilities for the purposes of this policy and do not warrant a CVD report:

- Differences in canonical output between `qsag-canonical` and other RFC 8785 implementations, unless one of them is provably wrong against a specific RFC 8785 clause. Such differences are interesting bug reports but should be filed as regular GitHub issues with a citation to the relevant RFC 8785 clause and the official RFC 8785 test vectors.
- Performance differences between `qsag-canonical` and other implementations. Performance issues belong in regular GitHub issues.
- API design preferences ("you should support feature X"). These belong in regular GitHub issues or Discussions.
- Compliance gaps (e.g., "you should be ISO 42001 certified"). These belong in regular GitHub issues.

---

## Bug bounty programme

We do not currently operate a paid bug bounty programme. We expect to add one once Tier 1 grant funding lands (target Q3 2026) and earmarks a portion for security research compensation. Until then, all CVD work is unpaid; we credit reporters publicly and offer letters of recommendation for academic and professional contexts.

---

## Hardening commitments

We commit to the following ongoing security practices for `qsag-canonical`:

- Every release is GPG-signed by the maintainer (fingerprint above).
- Every release is Sigstore-signed via GitHub Actions OIDC; the Rekor v2 transparency-log entry is recorded.
- Every release ships with a CycloneDX 1.6 SBOM.
- Every release ships with SLSA Level 3 build provenance.
- Dependencies are pinned and audited; updates are applied within 14 days for any advisory rated High or Critical.
- We participate in the GitHub Security Advisory database and assign CVE numbers via GitHub's CNA process for every accepted vulnerability.
- Every fixed canonicalisation-divergence bug is committed to the public conformance test corpus as a regression test, citing the discovery and the affected versions.

---

## Contact summary

| Purpose | Address |
|---|---|
| **Security disclosures (PGP recommended)** | security@neoxyber.com |
| **PGP fingerprint** | `A65AF5B7F02C9EB5B98023D70DB861BBF30F0D7B` |
| **General correspondence** | contact@neoxyber.com |
| **Code of Conduct reports** | zaidnaeem@neoxyber.com |
| **Public information** | info@neoxyber.com |

---

*Maintainer: Muhammad Zaid Naeem (Neoxyber). For company details see [COMPANY_FACTS.md](COMPANY_FACTS.md).*

*© 2026 AIXYBER TECH LTD (Company No. 16826340), trading as Neoxyber. Registered in England and Wales. ICO Registration: ZC071900. Released under the Apache License, Version 2.0.*
