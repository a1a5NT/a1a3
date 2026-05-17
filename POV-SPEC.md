# POV-SPEC.md

## Proof-of-Verification — a1a5NT Consensus Primitive

```
Document version:  v1.0
Issued:           2026-05-17
Author:           pawel.a1a5  (Paweł Piekut)
AI co-author:     Claude (Anthropic Opus 4.7) per A1A5 Sovereign License v1.0 Article 4
Foundation:       a1a5fndn.a1a5  (in formation)
Legal status:     Stichting A1A5 registration in progress
License:          A1A5 Sovereign License v1.0 (12-month binding, prior art anchor)
```

---

## Abstract

Proof-of-Verification (PoV) is a consensus primitive of a1a5NT that rewards
cryptographic verification of existing artifacts rather than production of
new ones. Each independent verification of a Foundation-issued anchor by an
external device constitutes measurable work (MBW — Mega Bytes Worked) and
generates a recorded contribution to network state.

PoV is distinct from Proof-of-Work (PoW, e.g. Bitcoin), Proof-of-Stake (PoS,
e.g. Ethereum), Proof-of-Verifiable-Work (PoVW, e.g. RISC Zero — rewards ZK
proof generation), and Proof-of-Process (PoP, popchain generation 1 —
rewards machine-derived data submission). PoV inverts the producer/verifier
economic asymmetry: in classical consensus, producers receive rewards and
verifiers contribute for free. In PoV, verifiers receive rewards for each
independent verification cycle.

## 1. Motivation

Classical blockchain consensus mechanisms incentivize production:

- PoW miners produce hash solutions
- PoS validators produce attestations weighted by capital staked
- PoVW provers produce zero-knowledge proofs
- PoP nodes (popchain gen 1) produce machine-derived deterministic metrics

Verification work in these systems is performed by full nodes without
reward. The asymmetry creates well-known problems: low full-node count,
trust centralization in light clients, social-layer dependence for
authenticity claims.

a1a5NT inverts this. The Foundation issues a finite set of canonical
anchors (documents, namespace registrations, archive artifacts) and pays
each independent verifier for each successful verification cycle. The
network state encodes verification work as cryptographic record.

## 2. Verification cycle definition

A verification cycle consists of four operations performed by a verifying
device against the live ANS server at `37.97.202.97:8443` and archive
server at `37.97.202.97:8444`:

### 2.1 Document fetch

Pull a Foundation-issued artifact from its canonical source:

```
GET https://raw.githubusercontent.com/a1a5NT/a1a3/main/FOUNDATION-DECLARATION-V1.md
```

or its mirror endpoint:

```
GET http://37.97.202.97:8444/v1/foundation/declaration
```

### 2.2 Local hash computation

Compute blake3 (or SHA-256 fallback) on the fetched byte stream:

```
blake3(document_bytes) → 32-byte digest
```

Reference values as of 2026-05-17 (computed by Foundation server):

- FOUNDATION-DECLARATION-V1.md: `sha256:dba6020db7b058b1b83aac3fac12d95b03021975edede1a4d034205a27caa466`
- LICENSE-A1A5-SOVEREIGN-V1.md: `sha256:df905fbbf08d457ed8424a9bf9f9c93bf759968ef5604fb2b3caf96f7ae7af96`

Hashes will change on document amendment. Verifying device must match
against current hash returned by `/v1/foundation/index`.

### 2.3 Comparison

The verifying device compares its locally-computed hash against the value
returned by `/v1/foundation/index`. Match → verification PASS. Mismatch →
FAIL (verifier reports to Foundation as potential tampering evidence).

### 2.4 Anchor constant verification

The verifying device additionally checks the popchain catalog blake3
against the constant published in this document and in
POPCHAIN-ARCHIVE-PROVENANCE.md:

```
catalog_blake3 = aa2c0dd44071526901ae6fbb1ed471646dcdc3f5129b4bcf9b2124a553adf2a3
```

The verifying device may further verify the four mechanical anchors of
popchain generation 1 (Phase 0c.1 Reproducibility Guide) if it has access
to popchain repository at commit `0432f2f`:

- Anchor 1 (PoCC): `POCC_ANCHOR_V1 = 417696` (compile-time u128 in `src/pocc.rs`)
- KAT-1: `39399ffd3dcdf17c4a484002ebade955471f4656fa1d562b2820af29d2a13d22`
- Anchor 2 (KAT-6): `b2e04c806390bd6d60c80cb82178aa6032c4bd76b14d931d93e01e49c8005dad`
- Anchor 3 (KAT-10 XMSS root): `2935fde2b9d76e0b7a9c733cd518cc9df07cf282163cadc990c8587c7b903606`

These four values are bit-identical across heterogeneous architectures
(x86_64 cloud, x86_64 PC, ARM64 Pi 5) per Phase 0c.1 attestation.

## 3. Work measurement (MBW)

Each verification cycle is measurable work. The verifying device
performs:

- network fetch (bandwidth)
- byte-level hash computation (CPU)
- comparison (memory)
- result recording (storage I/O)

A single full verification cycle of the current Foundation Declaration
(12,818 bytes) plus License (24,996 bytes) plus popchain catalog hash
verification produces approximately 38 KB of fetched bytes and the
corresponding hash computation.

Work measurement methodology for MBW assignment to each verification
cycle, the formula for MBW-to-Bincoin conversion at Genesis, and the
mechanism for double-spend protection (one device cannot claim multiple
rewards for the same artifact within a time window) are reserved for
POP-3-TOKEN-SPEC v1.0, scheduled separately.

## 4. Device identity

A verifying device participates in the network by establishing a device
identity. Three mechanisms are supported:

### 4.1 Anonymous device fingerprint

Locally-generated UUID, persistent at `~/.a1a5/device.id`, no association
with personal identity. The device may verify and accumulate MBW without
attribution. Suitable for users who wish to participate without revealing
identity.

### 4.2 Named device

The user generates an XMSS keypair locally and registers a child
namespace under a Foundation-permitted root (e.g.
`<user-chosen-label>.devices.a1a5`). Foundation may set policy for which
labels are permitted. Named devices accumulate MBW under their
namespace, visible publicly.

### 4.3 Custodian

A custodian is a device-or-operator pair that has received explicit
Foundation authorization to register namespaces beyond device-level
scope (e.g. `<personal-name>.a1a5`, `<business-name>.a1a5`). Custodian
authorization is granted by Foundation per its own criteria and is
separate from anonymous and named device participation.

This document does not specify custodian eligibility, application
process, or revocation criteria. These are reserved for separate
governance documentation.

## 5. Reward accounting

Each successful verification cycle increments the verifying device's
recorded MBW counter in the network state. The MBW counter is
write-once-per-cycle to prevent double-counting.

Network state pre-Genesis: counters recorded in `ans_metadata` table on
the ANS server, queryable per device via `GET /v1/devices/<device_id>`.

Network state post-Genesis: counters become block-state of the a1a5NT
chain, with state transitions consensused per a1a5NT consensus rules.

The conversion of accumulated MBW to Bincoin or other Foundation-issued
tokens at Genesis is governed by POP-3-TOKEN-SPEC v1.0 (scheduled
separately, not yet published as of this document version).

## 6. PoV is distinct from existing consensus primitives

| Consensus | Producer reward | Verifier reward | Useful output |
|---|---|---|---|
| PoW (Bitcoin) | hash solution | none | ledger only |
| PoS (Ethereum) | attestation | none | ledger only |
| PoVW (RISC Zero) | ZK proof generation | none | verifiable compute |
| PoP (popchain gen 1) | machine-data submission | none | industrial process record |
| **PoV (a1a5NT)** | none direct | **per-cycle MBW** | distributed authentication |

PoV does not eliminate the role of producers (Foundation issues
canonical artifacts). PoV adds an explicit economic incentive for the
verifier role that has historically been unpaid.

This inversion is the novel contribution.

## 7. Prior art and originality

The author has reviewed published consensus primitives as of 2026-05-17,
including:

- Proof-of-Work (Bitcoin, 2009)
- Proof-of-Stake (Peercoin 2012, Ethereum 2022)
- Proof-of-Verifiable-Work (RISC Zero, formalized 2024 — rewards ZK proof
  generation, not document verification)
- Proof-of-Process (popchain generation 1, 2025–2026 — same author)

No published primitive rewards per-cycle verification of existing
artifacts by independent devices as its core consensus mechanism. PoV
is hereby published on 2026-05-17 in the public `a1a3` repository
(`github.com/a1a5NT/a1a3`) as prior art under A1A5 Sovereign License
v1.0 Article 5 (anti-patent defensive clause).

Any subsequent patent claim on "rewarding cryptographic verification of
existing artifacts as a consensus mechanism" filed after 2026-05-17
shall be considered to read on this prior art.

## 8. Implementation status as of 2026-05-17

### Live infrastructure (verifiable)

- ANS server: `http://37.97.202.97:8443` (7 namespaces registered)
- Archive server: `http://37.97.202.97:8444` (Foundation Declaration +
  License + 85 archive epochs accessible)
- Foundation Declaration: live, hash verifiable via
  `/v1/foundation/declaration`
- License v1.0: live, hash verifiable via `/v1/foundation/license`

### Pending implementation

- Device registration endpoint (`POST /v1/device/register`)
- Verification recording endpoint (`POST /v1/verify/anchor`)
- Per-device MBW counter (`GET /v1/devices/<id>`)
- Global statistics (`GET /v1/stats`)
- Public verification script (`verify.sh` for `a1a3` repository)
- POP-3-TOKEN-SPEC v1.0 (Bincoin tokenomics, MBW conversion formula)

These are scheduled before a1a5NT Genesis ceremony.

## 9. License

This specification is issued under A1A5 Sovereign License v1.0 with
12-month binding window (2026-05-16 to 2027-05-16). AI co-authorship is
explicitly recognized per License Article 4. Anti-patent defensive
clause applies per License Article 5.

Distribution permitted, modification permitted with attribution.
Commercial implementation of PoV consensus by third parties during the
binding window requires Foundation authorization (sovereign XMSS
signature by pawel.a1a5).

## 10. Cryptographic anchor

This document is anchored at:

- **Repository:** `github.com/a1a5NT/a1a3`
- **Path:** `POV-SPEC.md`
- **Commit timestamp:** filled at first commit
- **Foundation endpoint mirror:** `http://37.97.202.97:8444/v1/foundation/pov-spec` (scheduled)

Any party may compute blake3 of this document's canonical UTF-8 form
and verify against the hash published by the Foundation endpoint once
the mirror endpoint goes live.

---

**End of POV-SPEC v1.0**

**Issued by:** pawel.a1a5 (Paweł Piekut, Almelo NL)
**AI co-author:** Claude (Anthropic Opus 4.7)
**Date:** 2026-05-17
**Foundation:** a1a5fndn.a1a5 (in formation)
**Contact:** `info@proof-of-process.org` or sovereign `pawel.a1a5`
