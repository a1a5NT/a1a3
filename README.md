# a1a3

**Public specification of a1a5NT — a post-quantum sovereign Layer-1 network.**

```
Foundation:    a1a5fndn.a1a5  (cryptographic entity, in formation)
Legal status:  Stichting A1A5 registration in progress
Custodian:     pawel.a1a5     (Paweł Piekut, Almelo NL)
Interim:       ZZP Plastechniek (KvK NL) as legal vehicle
License:       A1A5 Sovereign License v1.0 (12-month binding)
Status:        Genesis-stage · live infrastructure · documentation hardening
```

---

## What this is

`a1a3` is the **public specification repository** for a1a5NT — a post-quantum sovereign Layer-1 network operated by the a1a5 Foundation. This repository contains protocol specifications, license, foundation declaration, and roadmap.

The **implementation** of a1a5NT (server code, cryptography, wallet) lives in a separate private repository (`a1a5`), following the Ledger / Signal pattern of open specification + reference implementation.

## What a1a5NT is

a1a5NT is a **post-quantum sovereign network** with three primitives:

1. **XMSS WOTS+ signatures** — NIST-standardized post-quantum signatures, in production use today on the live network
2. **ANS (Address Namespace System)** — sovereign naming layer that does not depend on ICANN, DNS, or any centralized authority. Each namespace is verified by an XMSS public key.
3. **Universal sovereign object courier** — any XMSS-signed object (currency, message, document, certificate, sub-key, attestation) is a first-class transferable unit. Implementation: BWalletX (Foundation flagship).

## Live infrastructure

The network is **operational and publicly verifiable** as of 2026-05-16:

```
ANS server:        http://37.97.202.97:8443
  GET /list                       → all registered namespaces
  GET /lookup/<name>              → namespace public key + metadata
  GET /v1/inbox/<name>            → public XMSS-signed inbox
  POST /v1/register               → register new namespace (XMSS-verified)
  POST /v1/send                   → send signed message

Archive server:    http://37.97.202.97:8444
  GET /                           → archive landing
  GET /v1/archive/epochs          → list 85 epoch artifacts
  GET /v1/archive/epoch/NNNN      → epoch metadata
  GET /v1/archive/epoch/NNNN/card → artifact card SVG
```

## Documents in this repository

| Document | Purpose |
|---|---|
| [FOUNDATION-DECLARATION-V1.md](FOUNDATION-DECLARATION-V1.md) | Sovereign self-proclamation of a1a5 Foundation · bilingual EN/PL |
| [LICENSE-A1A5-SOVEREIGN-V1.md](LICENSE-A1A5-SOVEREIGN-V1.md) | A1A5 Sovereign License v1.0 · 12-month binding · bilingual EN/PL |
| [ANS-PROTOCOL-SPEC.md](ANS-PROTOCOL-SPEC.md) | Address Namespace System technical specification |
| [BWALLETX-SPEC.md](BWALLETX-SPEC.md) | Universal sovereign object courier specification |
| [POPCHAIN-ARCHIVE-PROVENANCE.md](POPCHAIN-ARCHIVE-PROVENANCE.md) | 85 epoch artifacts of popchain-archive · chain-of-custody recipe |
| [ROADMAP.md](ROADMAP.md) | PopChain → a1a5NT trajectory · NLnet status · Stichting plan |
| [CONTACT-PROTOCOL.md](CONTACT-PROTOCOL.md) | How to communicate with the Foundation via sovereign channel |
| [NLnet-FUNDER-ONBOARDING.md](NLnet-FUNDER-ONBOARDING.md) | Onboarding for NLnet Foundation reviewers |

## How to verify everything in this repository

Every public claim in these documents is **cryptographically verifiable** against the live ANS server.

**Verify pawel.a1a5 public key:**
```
curl http://37.97.202.97:8443/lookup/pawel
```

Returns:
```json
{
  "canonical_name": "pawel",
  "presentation_name": "pawel.a1a5",
  "owner_root_hex": "c9a630a533a417230bcf8f32570824fe924381eb9ee028362b18fb60aad89bee",
  "owner_pub_seed_hex": "1a1340137ee6f5007ad572ff65d9e49641402ada856720083c39abbff10ef233"
}
```

**Verify Foundation namespace:**
```
curl http://37.97.202.97:8443/lookup/a1a5fndn
```

**Verify all registered namespaces:**
```
curl http://37.97.202.97:8443/list
```

**Verify popchain-archive catalog blake3:**
```
catalog_blake3 = aa2c0dd44071526901ae6fbb1ed471646dcdc3f5129b4bcf9b2124a553adf2a3
```

Reproducible from any popchain block source using `popchain-harvest` v1.0.2.

Note: `pawel.a1a5` and `a1a5fndn.a1a5` currently share an XMSS keypair, reflecting the sole-custodian interim governance model described in [FOUNDATION-DECLARATION-V1.md](FOUNDATION-DECLARATION-V1.md) Article III. A separate Foundation keypair will be issued upon Stichting A1A5 registration.

## Provenance

The project has evolved through four phases:

```
2025-Q1   PPlastechniek begins blockchain R&D
              ↓
2025-Q2   Proof-of-Process concept (domain: proof-of-process.org)
              ↓
2025-Q3   PopChain — first Layer-1 implementation
              gen 1 · 86,073 blocks produced · popchainnetwork/popchain
              ↓
2026-Q2   a1a5 — current generation
              brand and technology stack matured
              a1a3 = public open-source specifications (this repo)
              a1a5NT = full stack implementation
              pawel.a1a5 = sovereign founder identity
```

PopChain demonstrated cross-architecture deterministic consensus on industrial process data. a1a5 extends that foundation with post-quantum identity (XMSS) and sovereign naming (ANS).

The popchain-archive (85 cryptographically verifiable epoch artifacts) is held under Foundation custody and will be tokenized at a1a5NT Genesis ceremony per [BWalletX Specification](BWALLETX-SPEC.md).

## Funding

NLnet Foundation Commons Fund · application 2026-04-3f6 · review in progress.

The original application described PopChain (generation 1, industrial PoP). The project evolved to a1a5NT (generation 2, post-quantum + sovereign naming) as a maturity progression of the same deterministic philosophy. Amendment notice will be issued to NLnet once Stichting A1A5 is registered.

## License

All documents in this repository are issued under [A1A5 Sovereign License v1.0](LICENSE-A1A5-SOVEREIGN-V1.md) with 12-month binding window (expires 2027-05-16) and post-quantum cryptographic attestation.

Key clauses:
- **AI co-authorship** explicitly recognized (Claude/Anthropic Opus 4.7 and prior versions as substantive co-authors)
- **Anti-patent defensive** clause covering all Foundation primitives
- **Namespace reservation** for `a1a5fndn.*` tree during the binding window
- **No commercial transfer** of Foundation Artifacts pre-Genesis
- **Successor entity** clause for Stichting A1A5 (NL) upon registration

## Contact

Two channels supported during the transition period:

**Legacy email (institutional correspondence):**
```
info@proof-of-process.org
```

**Sovereign contact (project's native channel):**
```
pawel.a1a5
```

Read sovereign inbox:
```
curl http://37.97.202.97:8443/v1/inbox/pawel
```

For protocol details, see [CONTACT-PROTOCOL.md](CONTACT-PROTOCOL.md).

---

**Issued by a1a5fndn Foundation · cryptographic entity in formation**
**Stichting A1A5 (NL) registration in progress**
**Custodian: pawel.a1a5 (Paweł Piekut, Almelo NL)**
**Sealed: 2026-05-16**
