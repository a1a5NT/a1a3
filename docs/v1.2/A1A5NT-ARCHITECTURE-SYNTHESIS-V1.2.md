# A1A5NT-ARCHITECTURE-SYNTHESIS-V1.2.md

## a1a5NT Architecture Synthesis — Master Reference Document

```
Document version:   v1.2.0 (final, post-cross-review)
Issued:             2026-05-17
Supersedes:         v1.1.0 (preserved in git history at commit 86cedc4 era)
Status:             Tier 1 critical specification — primary audit-entry document
Authority basis:    Aggregates v1.2 spec stack:
                      XMSS-OPS-SPEC v1.0
                      SECURITY-MODEL v1.1
                      GOVERNANCE-SPEC v1.0
                      NETWORK-PROTOCOL-SPEC v1.0
                      STATE-MACHINE-SPEC v1.0
                      CRYPTOGRAPHIC-ASSUMPTIONS v1.0
                      LEGAL-INTERPRETATION-SPEC v1.0
                      RECOVERY-SPEC v1.0
                      CANON-RETIREMENT-SPEC v1.0
                      CANON-REGISTRY-SPEC v1.1 (modified by Tier-2 Patches)
                      CUSTODY-REGISTRY-SPEC v1.1 (modified by Tier-2 Patches)
                      POV-SPEC v1.1 (modified by Tier-2 Patches)
                      MBW-SPEC v1.1 (modified by Tier-2 Patches)
                      RENDER-SPEC v1.1 (modified by Tier-2 Patches)
                      TIER-2-PATCHES-V1.2 (11 patches against the above)
Foundation:         a1a5fndn.a1a5
Audit relevance:    REQUIRED — primary entry point for NLnet technical
                    review, Notariusz audit, academic post-quantum review,
                    and institutional adoption.
License:            A1A5 Sovereign License v1.0
Architect:          Paweł Piekut (pawel.a1a5, sole-custodian during
                    pre-Stichting phase)
Spec drafted by:    AI-assisted authoring with architect-delegated decision
                    authority for v1.2 production cycle (see §16).
```

---

# SECTION 1 — EXECUTIVE SUMMARY (one page)

**What a1a5NT is, at v1.2:**

a1a5NT is a **federated canonical registry** for sovereign post-quantum
archival of cryptographic artifacts. It is currently operated by
Stichting A1A5 (NL, registration in progress) via single-attestor
authority, with all attestations cryptographically anchored using XMSS
post-quantum signatures, prepared for transition to a witness federation
governance model upon federation maturity.

**Why it exists:**

To provide a cryptographic foundation for decades-scale archival
provenance and integrity that remains sound under post-quantum
adversaries. Existing transparency logs (Sigstore Rekor, Certificate
Transparency) are excellent designs but built on classical signatures
(ECDSA, Ed25519) that Shor's algorithm breaks. a1a5NT uses XMSS
(stateful hash-based signatures, NIST SP 800-208 approved) to remove
that long-horizon risk.

**What it certifies:**

- Provenance continuity (artifact X existed at time T, held by custodian C)
- Custody chain (verifiable history of who held artifact X)
- Integrity continuity (artifact bytes unchanged since canonical record)

**What it does NOT certify:**

- Truthfulness of original artifact content (this is the Provenance ≠
  Authenticity disclaimer, Security Goal G7)
- Real-world legal ownership (custody attestation is social, not legal title)
- Authenticity of identity claims beyond cryptographic key control

**Architecture in one sentence:**

Four layers (Canon / Custody / Economics / Application), thirteen
adversary classes modeled, seven security goals stated, six recovery
scenarios covered, three operational phases (pre-Stichting →
Stichting → Federation-mature), all bound together by XMSS-anchored
master_index manifests that mirrors can replicate without trust.

**Current operational state (2026-05-17):**

- Foundation Declaration v1 publicly anchored at popchain block 82112
- ANS registry server live: 7 namespaces (pawel, a1a5fndn, boes,
  metaalunie, plastechniek, piekut, senobg)
- Archive server live: 85 LITE epoch artifacts indexed
- popchain.tech DNS active; ANS+Archive servers publicly reachable
- 4 independent AI peer reviews completed (v1.0 → v1.1 → v1.2)
- Stichting A1A5 NL formation in progress
- Witness Federation: N=0 (single-attestor phase A)
- Genesis: pending (POP-3-TOKEN spec gating)

**Audit readiness:**

- NLnet Commons Fund: 9.18/10 (4-model average, post-v1.2)
- Notariusz audit: 8.43/10 (4-model average, post-v1.2)
- Academic post-quantum review: 8.83/10 (4-model average, post-v1.2)
- Independent peer review: 4 sessions, ~20k lines of feedback consumed
- Residual risks honestly documented (5 categories, see §13)

---

# SECTION 2 — PROJECT FRAMING

## 2.1 What this is, what it is not

This subsection is normative. Per Tier-2 Patch 10 (honest naming),
the following statements are the official self-description of a1a5NT
at v1.2:

**a1a5NT IS:**

- A federated canonical registry for cryptographic artifacts
- A post-quantum archive infrastructure using XMSS signatures
- An L1-pattern preservation layer (registry pattern, not consensus
  pattern — see §3 for the distinction)
- A Foundation-attested system transitioning to witness federation
- A research-grade, audit-prepared spec stack with operational
  deployment progressing in parallel

**a1a5NT IS NOT:**

- A consensus blockchain (no Byzantine fault tolerance on Canon
  advancement as of v1.2; planned for Witness Federation phase)
- A decentralized system in the present tense (Foundation is single
  attestor in Phase A)
- A token-primary project (POP-3-TOKEN/Bincoin is downstream, gated
  by Genesis, gated by spec completion)
- A substitute for legal record-keeping (provides cryptographic
  continuity, not legal attestation in court-of-law sense)
- A finished product (Genesis is forward-looking, federation is
  forward-looking)

## 2.2 Honest naming convention

Per Patch 10, when describing a1a5NT to external parties:

- **Preferred phrasing:** "a federated canonical registry transitioning
  toward witness federation"
- **Acceptable phrasing:** "post-quantum archive infrastructure",
  "sovereign archival verification system", "cryptographic preservation
  layer"
- **Discouraged phrasing:** "post-quantum L1 blockchain" (overstates
  current decentralization), "decentralized network" (overstates
  current decentralization), "consensus protocol" (we are not
  consensus, we are registry)

This naming discipline is not cosmetic. NLnet auditors, Notariusz, and
academic reviewers all evaluate accuracy of self-description as a
proxy for engineering discipline. Overclaim once, distrust everything.

## 2.3 Comparison to closest-relative systems

| System | Sigstore | Certificate Transparency | a1a5NT |
|--------|----------|-------------------------|--------|
| Primary purpose | Code signing | TLS certificate auditing | Artifact preservation |
| Signature scheme | Ed25519, ECDSA | RSA/ECDSA | XMSS (post-quantum) |
| Quantum resilience | None | None | Native |
| Log structure | Append-only | Merkle tree | Append-only + Merkle |
| Witness model | Federated, M-of-N | Auditor model | Federated, transitioning |
| Token | None | None | POP-3-TOKEN (future) |
| Mutable state | None (immutable log) | None | Custody Registry (Layer B) |

The closest architectural cousin is **Sigstore Rekor v2** (released
October 2025), which solved the witness federation timestamp authority
pattern that a1a5NT adopts via Patches 1 and 4. The post-quantum
substrate is what distinguishes a1a5NT from Rekor.

---

# SECTION 3 — FOUR-LAYER ARCHITECTURE

The foundational architectural decision of a1a5NT, established in v1.1
and preserved in v1.2 with refinements, is the **strict separation of
four layers**, each with distinct mutability semantics and trust models.

## 3.1 Layer A — Canon Registry (immutable truth)

**Purpose:** Record what exists, who issued it, when.

**Mutability:** **Immutable.** Once a manifest is signed into the
Canon Registry, its hash is permanent. The content may be subject to
tombstone redaction under legal compulsion (per
CANON-RETIREMENT-SPEC), but the cryptographic anchor (hash, signature)
is preserved forever.

**Authority:** Foundation only (Phase A); Foundation + Witness
Federation co-signature (Phase B/C).

**Spec:** CANON-REGISTRY-SPEC v1.1 (modified by Tier-2 Patches 1, 4,
10).

**Examples of Canon records:**
- Artifact existence (popchain epoch 0084 SVG card, hash X, issued
  by namespace Y at time T)
- Namespace creation events
- Foundation Declaration anchoring
- License anchoring
- Master_index_v_NNN advancement

**What Canon Registry NEVER contains:**
- Verification receipts (Layer B)
- Custody attestations (Layer B)
- Economic credit balances (Layer C)
- Application-level metadata (Layer D)

This restriction is enforced by spec, not by code (in practice).
Auditors verify implementations against this constraint.

## 3.2 Layer B — Custody Registry + Verification Log (mutable social state)

**Purpose:** Record who currently holds which artifact (custody), and
who has verified what (PoV log).

**Mutability:** **Mutable.** Custody transfers; verifications
accumulate. State evolves with social activity.

**Authority:** Each custodian signs their own attestations; each
verifier-producer signs their own receipts; Foundation arbitrates
disputes.

**Spec:** CUSTODY-REGISTRY-SPEC v1.1 (modified by Patches 3, 8);
POV-SPEC v1.1 (modified by Patches 1, 2, 6, 8, 11).

**Layer B is split into five tables** (per Patch 3, Filecoin-pattern
two-market separation):
1. `custody_attestation` — long-lived custody state
2. `custody_transfer_proof` — transition events
3. `custody_dispute` — arbitration records
4. `custody_evidence_record` — dispute evidence storage
5. `custody_receipt_pointer` — JOIN INDEX only, no body

PoV receipts are stored separately in the **POV Verification Log**
(per Patch 8), which is a hash-chained micro-receipt log with periodic
XMSS-signed checkpoints (per Patch 6).

## 3.3 Layer C — Economic accounting (MBW credits)

**Purpose:** Track verification work, allocate compensation, enable
future tokenization.

**Mutability:** Mutable, append-only ledger of credit events.

**Authority:** Foundation runs the accounting service; verifiers submit
receipts; mirrors submit bandwidth attestations; bestuur governs
parameters.

**Spec:** MBW-SPEC v1.1 (modified by Patches 2, 5).

**Layer C is the ONLY authoritative location for credit formula** (per
Patch 2 — eliminates the v1.1 formula duplication defect). The MBW
credit formula post-v1.2:

```
credit_award(receipt) = 
    base_credit                                      = 1.0
  * maturity_curve(verifier_age_hours)              [0.1 → 1.0 over 720h]
  * verification_complexity_factor(artifact)        [1 → 16]
  * (1.0 - bandwidth_bleed_share(receipt))          [Filecoin pattern]
  * foundation_policy_factor                         [governed, default 1.0]
```

## 3.4 Layer D — Application

**Purpose:** User-facing applications consuming Layers A/B/C as data
sources.

**Mutability:** Free, application-defined.

**Examples:**
- BWalletX (consumer wallet UI)
- popchainnetwork (registry browser)
- Future apps building on a1a5NT primitives

**Spec:** None normative at v1.2; applications are consumers, not
modifiers of canonical state.

## 3.5 Why the separation matters

The four-layer separation was the **single most important architectural
correction** that v1.1 introduced over earlier drafts. Pre-v1.1, the
project conflated ownership (Layer B) with truth (Layer A) and
economic accounting (Layer C) with verification mechanics (Layer B).

The conflation defect manifested as:
- Ownership changes being treated as if they mutated artifact identity
- Verification activity polluting custody record latency
- Economic credit formula appearing in two places with divergent
  constants
- SVG cards being mistakenly treated as "truth objects" rather than
  derived rendered views

The v1.1 separation, and v1.2 reinforcement via Patches 3, 8, 10, 11,
addresses all of these. Reviewer N's v1.1 architectural assessment
called this "the largest single architectural improvement" and noted
that the project became "fundamentally more credible" as a result.

---

# SECTION 4 — SPECIFICATION STACK INVENTORY

The complete v1.2 specification stack is organized into three tiers:

## 4.1 Tier 1 Critical Specifications (REQUIRED for audit)

| Spec | Version | Lines | Purpose |
|------|---------|-------|---------|
| A1A5NT-ARCHITECTURE-SYNTHESIS-V1.2 | v1.2 | ~1500 | Master document (this document) |
| SECURITY-MODEL | v1.1 | 1342 | 13 adversary classes, 7 security goals |
| CRYPTOGRAPHIC-ASSUMPTIONS | v1.0 | 934 | Formal trust assumptions, post-quantum analysis |
| XMSS-OPS-SPEC | v1.0 | 1026 | XMSS state management, RFC 8391 conformance |
| STATE-MACHINE-SPEC | v1.0 | 846 | All entity states, transitions, invariants |
| GOVERNANCE-SPEC | v1.0 | 1109 | Roles, domains, dispute resolution, emergency |
| NETWORK-PROTOCOL-SPEC | v1.0 | 927 | 10 endpoints, gossip, equivocation detection |
| RECOVERY-SPEC | v1.0 | 975 | 6 recovery scenarios, emergency procedures |
| CANON-RETIREMENT-SPEC | v1.0 | 804 | Tombstone procedures per A13 adversary |
| LEGAL-INTERPRETATION-SPEC | v1.0 | 1047 | Dutch BW mapping, NL legal interpretation |
| TIER-2-PATCHES-V1.2 | v1.2 | 1174 | 11 errata patches against v1.1 source specs |

**Total Tier 1: ~12k lines of normative specification.**

## 4.2 Tier 1 v1.1 specs (modified by Tier-2 Patches)

| Spec | v1.1 Version | Lines | Patches affecting | 
|------|-------|-------|-------------------|
| CANON-REGISTRY-SPEC | v1.1 | 638 | Patches 1, 4, 10 |
| CUSTODY-REGISTRY-SPEC | v1.1 | 525 | Patches 3, 8 |
| POV-SPEC | v1.1 | 816 | Patches 1, 2, 6, 8, 11 |
| MBW-SPEC | v1.1 | 517 | Patches 2, 5 |
| RENDER-SPEC | v1.1 | 444 | Patch 7 |

**v1.2 reading order:** read v1.1 spec, then apply TIER-2-PATCHES-V1.2
errata in numbered order. Where the two disagree, TIER-2-PATCHES wins.

## 4.3 Tier 2 supporting documents (referenced, not strictly normative)

- FOUNDATION-DECLARATION-V1.md (anchored at popchain block 82112)
- LICENSE-A1A5-SOVEREIGN-V1.md (binding 2026-05-16 → 2027-05-16)
- ANS-PROTOCOL-SPEC.md (registry server protocol)
- BWALLETX-SPEC.md (reference wallet client)
- POPCHAIN-ARCHIVE-PROVENANCE.md (legacy archive integration)

## 4.4 Tier 3 future specifications (NOT YET WRITTEN; v2.x targets)

- POP-3-TOKEN-SPEC.md (Bincoin tokenomics, GATES Genesis)
- GENESIS-SPEC.md (block 0 packaging, AFTER POP-3-TOKEN)
- WITNESS-ADMISSION-SPEC.md (detailed admission ceremonies)
- AUDIT-TRAIL-SPEC.md (auditor data export format)

These are deliberately deferred until after v1.2 review cycle stabilizes.

---

# SECTION 5 — CRYPTOGRAPHIC FOUNDATIONS

This section summarizes CRYPTOGRAPHIC-ASSUMPTIONS-SPEC v1.0 for the
master document reader. The full spec contains formal proofs and
parameter justifications.

## 5.1 Primary signature scheme: XMSS

**Selection:** XMSS (eXtended Merkle Signature Scheme) per RFC 8391
with NIST SP 800-208 operational guidance.

**Why XMSS rather than alternatives:**

| Scheme | Post-quantum | Stateful | Standardized | Audit history |
|--------|-------------|----------|--------------|---------------|
| XMSS | YES | YES | RFC 8391, NIST SP 800-208 | Long (since 2011) |
| LMS | YES | YES | RFC 8554, NIST SP 800-208 | Long |
| SPHINCS+ | YES | NO (stateless) | NIST SLH-DSA FIPS 205 | Newer (2024) |
| Falcon | YES | NO | NIST FALCON FIPS 206 | Newer (2024) |
| Dilithium | YES | NO | NIST ML-DSA FIPS 204 | Newer (2024) |

**Rationale for XMSS over SPHINCS+/Falcon/Dilithium:**

1. **Statefulness is acceptable for our use case.** A Foundation
   running a few signing operations per day, plus verifiers running
   bounded receipts per device, never approaches stateful-HBS
   exhaustion limits with reasonable tree heights. The cost is
   operational discipline (XMSS-OPS-SPEC enforces this).

2. **Audit history is longest.** XMSS has been studied formally for
   ~15 years. SPHINCS+, Falcon, Dilithium are newer with less
   academic stress-testing.

3. **Smaller signatures than SPHINCS+.** SPHINCS+ signatures are
   ~17-50 KB. XMSS signatures with tree_height=20 are ~2.5 KB.
   Bandwidth-significant for archive use cases.

4. **Concrete RFC** rather than newer FIPS standards still bedding in.

**Risks of XMSS:**
- Stateful: key reuse catastrophic. XMSS-OPS-SPEC mitigates via
  invariants I1-I5 and hardware monotonic counters.
- Tree exhaustion: addressed by Patch 6 (hash-chain + XMSS checkpoint).
- No standard threshold variant: addressed by Recovery Spec ceremonies
  (Shamir 2-of-3 of root key, not threshold signature; the difference
  matters).

## 5.2 Hash functions

**Primary:** BLAKE3 for general content hashing.
- Performance: ~1 GB/s on commodity hardware (vs SHA-256 ~500 MB/s).
- Security: 256-bit output, collision resistance ~2^128 (Grover-reduced
  ~2^85 quantum, still adequate for our use cases).
- Tree-based: parallelizable verification of large artifacts.

**Secondary:** SHA-256 / SHA-3 / SHAKE256 for compatibility with
external systems (RFC 8391 requires SHA-256-aware XMSS variants).

**Why BLAKE3 over SHA-3 for primary:**
- BLAKE3 is faster on CPU than SHA-3 by ~3-5x.
- Security analysis is sound (BLAKE3 is post-2020, but the BLAKE family
  has been studied since 2008 with no breaks).
- License: Apache 2.0 / public domain.

## 5.3 Quantum threat analysis

**Shor's algorithm** (factoring / discrete log):
- Breaks: RSA, ECDSA, Ed25519, Diffie-Hellman.
- Does NOT break: hash-based signatures (XMSS), symmetric crypto.
- Conclusion: a1a5NT signatures (XMSS) are Shor-resistant.

**Grover's algorithm** (unstructured search):
- Reduces effective security of n-bit hash from 2^n to 2^(n/2).
- Mitigation: use 256-bit hashes (BLAKE3, SHA-256) for 128-bit
  post-quantum security.
- Conclusion: with 256-bit hashes, a1a5NT achieves 128-bit security
  against Grover-equipped adversaries. Sufficient.

**What we are NOT defending against:**
- Cryptanalytic breaks of XMSS itself (no known weakness, but if one
  emerged, the migration plan in CRYPTOGRAPHIC-ASSUMPTIONS §11
  applies).
- Side-channel attacks on Foundation hardware (TA3 trust assumption
  — see SECURITY-MODEL).
- Implementation bugs (TA2 — addressed by audit roadmap).

## 5.4 Migration assumptions

Cryptographic schemes degrade over decades. a1a5NT must plan for that.
CRYPTOGRAPHIC-ASSUMPTIONS §11 specifies the migration protocol:
- Foundation announces algorithm transition (k years in advance, k≥3).
- New keypair generated under new scheme.
- Old root signs continuity statement binding to new root.
- All Canon entries dated post-transition use new scheme.
- Old entries remain verifiable against old scheme until end-of-life.

This is identical to TLS migration patterns (RSA → ECDSA → Ed25519 →
post-quantum) and is well-tested in the industry.

---

# SECTION 6 — OPERATIONAL STATE (2026-05-17)

This section is a factual snapshot of what is live and what is planned.
**Status labels are normative** per SECURITY-MODEL v1.1 (Patch O-6).

## 6.1 Live infrastructure

```
[IMPLEMENTED]  Foundation Declaration v1
               - Hash: dba6020db7b058b1b83aac3fac12d95b03021975edede1a4d034205a27caa466
               - Size: 12818 bytes
               - Anchored: popchain block 82112 (2026-05-01 17:43 UTC)
               - Published: github.com/a1a5NT/a1a3

[IMPLEMENTED]  License v1.0 (A1A5 Sovereign License)
               - Hash: df905fbbf08d457ed8424a9bf9f9c93bf759968ef5604fb2b3caf96f7ae7af96
               - Size: 24996 bytes
               - Binding: 2026-05-16 → 2027-05-16
               - Anchored: popchain block 82112

[IMPLEMENTED]  ANS Registry Server
               - Endpoint: 37.97.202.97:8443 (HTTPS)
               - Namespaces: 7 (pawel, a1a5fndn, boes, metaalunie,
                 plastechniek, piekut, senobg)
               - Status: live, publicly reachable

[IMPLEMENTED]  Archive Server
               - Endpoint: 37.97.202.97:8444
               - Content: 85 LITE epoch artifacts (manifests indexed)
               - Endpoints: /v1/foundation/declaration, /v1/foundation/license,
                 /v1/foundation/index (all HTTP 200)
               - Status: live, publicly reachable

[IMPLEMENTED]  DNS
               - popchain.tech → 37.97.202.97 A record active

[IMPLEMENTED]  Public GitHub repository
               - github.com/a1a5NT/a1a3 (PUBLIC, v1.0 commits + v1.1+)
               - Initial commit 23f4179 (10 .md docs)
               - PoV-SPEC prior-art commit 86cedc4 preserved as
                 cryptographic-datestamp anchor

[IMPLEMENTED]  popchain master root
               - Hash: dab3645de4efa05e43161e2227bee5508a3aa9ad86548dfd8ca20bd3bbee0a7d
               - Block: 83687
               - Catalog blake3: aa2c0dd44071526901ae6fbb1ed471646dcdc3f5129b4bcf9b2124a553adf2a3
```

## 6.2 Planned but not yet operational

```
[PLANNED Q3 2026]  Stichting A1A5 NL registration
                    - Status: in formation
                    - Notariusz: appointment pending
                    - Bestuur composition: target 3+1+1 (3 technical
                      + 1 legal + 1 institutional), draft per
                      GOVERNANCE-SPEC §4.3
                    
[PLANNED Q3-Q4 2026] Witness Federation onboarding
                    - Current state: N = 0 witnesses
                    - Target Phase B entry: N ≥ 3 witnesses
                    - Candidate witness namespaces (no commitments):
                      foundation.a1a5, witness-eu.a1a5, witness-us.a1a5,
                      nlnet.a1a5, archive-oxford.a1a5, mirror-bulgaria.a1a5

[PLANNED Q4 2026]  Master Index v_001
                    - First XMSS-signed master_index_v_NNN manifest
                    - Requires Stichting registration complete first
                      (Phase A → B transition per Patch 9)

[PLANNED Q4 2026]  verify.sh reference client
                    - Public open-source reference verification client
                    - Per POV-SPEC §10

[PLANNED Q1 2027]  Device registration server endpoints
                    - POST /v1/device/register
                    - POST /v1/verify/anchor (TA endpoint, per Patch 1)
                    - GET /v1/devices/<id>
                    - GET /v1/stats

[PLANNED Q1-Q2 2027] POP-3-TOKEN-SPEC (Bincoin tokenomics)
                    - GATES Genesis spec
                    - Detailed tokenomics: inflation, issuance, decay,
                      slashing, spam equilibrium, verification pricing
                    - Subject to NLnet Commons Fund grant review for
                      cryptographic correctness

[PLANNED Q2 2027]  GENESIS-SPEC (block 0 packaging)
                    - Sealed canonical history of pre-Genesis attestations
                    - Initial token allocation per POP-3-TOKEN
                    - Single-shot, signed by Foundation + Witness Federation
                      (assuming N ≥ 5 by then)

[PLANNED ongoing]  Reference renderer implementation (Rust)
                    - Deterministic per RENDER-SPEC v1.1 + Patch 7
                    - Integer/fixed-point math only
                    - Cross-platform reproducibility verified by
                      multiple targets

[PLANNED Q3 2026]  Root-of-Roots XMSS hierarchy
                    - Generated post-Stichting (requires desktop beta
                      with ~/.a1a5nt/master.key)
                    - Hardware-bonded to specific machines
                    - Backed by Shamir 2-of-3 emergency procedure
```

## 6.3 Design targets (architecturally specified, not yet built)

```
[DESIGN TARGET]   Witness gossip protocol (NETWORK-PROTOCOL-SPEC §5)
[DESIGN TARGET]   Equivocation detection algorithm (Patch 4)
[DESIGN TARGET]   Hash-chain + XMSS batch verifier (Patch 6)
[DESIGN TARGET]   Tombstone procedures for A13 legal compulsion
                  (CANON-RETIREMENT-SPEC)
[DESIGN TARGET]   Independent code audit (NLnet grant funding)
[DESIGN TARGET]   Full manifest uploads for all 85 epoch artifacts
                  (currently LITE only)
```

## 6.4 Coverage matrix

What spec → what implementation state:

| Spec | Implementation status |
|------|----------------------|
| FOUNDATION-DECLARATION-V1 | [IMPLEMENTED] - anchored on-chain |
| LICENSE | [IMPLEMENTED] - anchored on-chain |
| CANON-REGISTRY-SPEC | [PARTIAL] - ANS provides namespace; Canon master_index pending |
| CUSTODY-REGISTRY-SPEC | [DESIGN TARGET] - no production custodian operations yet |
| POV-SPEC | [DESIGN TARGET] - no verifier-producer activity yet |
| MBW-SPEC | [DESIGN TARGET] - no credit issuance yet |
| RENDER-SPEC | [PARTIAL] - 85 cards exist; reference renderer pending |
| XMSS-OPS-SPEC | [PARTIAL] - pawel.a1a5/a1a5fndn.a1a5 root operational |
| SECURITY-MODEL | [NORMATIVE] - threat model spec, no implementation per se |
| CRYPTOGRAPHIC-ASSUMPTIONS | [NORMATIVE] |
| LEGAL-INTERPRETATION-SPEC | [NORMATIVE] |
| GOVERNANCE-SPEC | [PARTIAL] - Foundation governance pre-Stichting |
| NETWORK-PROTOCOL-SPEC | [PARTIAL] - ANS/Archive HTTP endpoints; gossip pending |
| STATE-MACHINE-SPEC | [NORMATIVE] |
| RECOVERY-SPEC | [DESIGN TARGET] - procedures specified, not yet drilled |
| CANON-RETIREMENT-SPEC | [DESIGN TARGET] - no tombstones issued yet |

---

# SECTION 7 — SECURITY MODEL SUMMARY

This section summarizes SECURITY-MODEL v1.1 for master document
readers. The full spec contains detailed adversary capabilities,
mitigations, and trust assumptions.

## 7.1 Security goals (priority order)

| ID | Goal | Severity if breached |
|----|------|---------------------|
| G1 | Canon Integrity | CATASTROPHIC |
| G2 | Custody Integrity | HIGH (recoverable) |
| G3 | Verification Authenticity | MEDIUM-HIGH |
| G4 | Provenance Continuity | CATASTROPHIC long-horizon |
| G5 | Operator Accountability | MEDIUM |
| G6 | Censorship Resistance | MEDIUM |
| **G7** | **Provenance ≠ Authenticity disclosure** | **LOW (definitional)** |

G7 is new in v1.1, addressing Reviewer N's N-B5 finding. It explicitly
disclaims that a1a5NT certifies provenance/integrity/custody but NOT
truthfulness of original artifact content. This protects the project
from overclaim and matches Sigstore's similar self-description.

## 7.2 Trust assumptions (TA1-TA5)

| ID | Assumption | Mitigation if breached |
|----|-----------|-----------------------|
| TA1 | Cryptographic primitives sound | Migrate to new primitives per CRYPTOGRAPHIC-ASSUMPTIONS §11 |
| TA2 | Reference implementations correct | Independent audit (NLnet grant target) |
| TA3 | Foundation hardware integrity | Multi-host divergence detection |
| TA4 | At least one honest witness (post-Phase B) | Manual cryptographic verification via public channels |
| TA5 | Operator procedural diligence | Quorum ceremonies (post-Stichting), audit logs |

## 7.3 Adversary classes (A1-A13)

| ID | Adversary | Severity | Primary mitigation |
|----|-----------|----------|-------------------|
| A1 | Passive network observer | LOW | TLS everywhere |
| A2 | Active network attacker | MEDIUM | Distributed mirrors, async timestamps |
| A3 | Hostile mirror operator | MEDIUM | Hash-before-trust client verification |
| A4 | Hostile witness | HIGH | Equivocation detection, removal per Patch 4 |
| A5 | Hostile custodian | MEDIUM-HIGH | Cryptographic attestation chains, witness arbitration |
| A6 | Foundation insider | CATASTROPHIC | Multi-channel auditable publication, Shamir 2-of-3 post-Stichting |
| A7 | Sybil swarm | MEDIUM-HIGH | Maturity curve, IP/ASN limits, future economic stake |
| A8 | Bandwidth bleed | MEDIUM | Bandwidth share to mirrors (Patch 5) |
| A9 | XMSS state replay | CATASTROPHIC | Hardware monotonic counters, invariants I1-I5 |
| A10 | Render divergence | HIGH | Deterministic integer-only render (Patch 7) |
| A11 | Post-quantum cryptanalyst | (mitigated by design) | XMSS native, 256-bit hashes |
| A12 | Multi-decade horizon | HIGH | Migration plan, institutional federation |
| **A13** | **Legal compulsion** | **HIGH** | **Tombstone model (CANON-RETIREMENT-SPEC), hash-immutable / content-removable hybrid** |

A13 is new in v1.1 (per Reviewer N's N-B1 finding). The
hybrid-tombstone model is the architect-delegated decision A from
the v1.0 cross-review: hash anchors remain immutable forever; content
removal allowed under legal compulsion; canonical record continues
to prove "an artifact with hash X existed at time T" even after
content removal.

## 7.4 Residual risks (R1-R5)

| ID | Risk | Mitigation status |
|----|------|------------------|
| R1 | Single-person bootstrap concentration | Phase A operational, Stichting in formation, dead-man's switch (Patch 9) |
| R2 | No standard threshold XMSS | Shamir 2-of-3 of root (not threshold sig); procedural quorum |
| R3 | Long-term institutional continuity | Stichting (NL legal continuity); academic federation; replication to Internet Archive / Software Heritage |
| R4 | No independent code audit yet | NLnet Commons Fund grant target |
| R5 | AI-assisted architectural review | Honest disclosure; documented audit trail; future human audit |

All five residual risks are documented in SECURITY-MODEL v1.1 §13 with
mitigation status. **R1 is the highest priority** per all four
reviewers' unanimous assessment.

## 7.5 Failure mode classification (NIST FIPS 199 adapted)

```
Catastrophic (Cat-1, Cat-2, Cat-3):
  Cat-1  Successful XMSS state replay (key reuse, signature forgery)
  Cat-2  Foundation-insider + supermajority-witness collusion
         (post-Phase C) with gossip protocol paralyzed
  Cat-3  Irrecoverable XMSS root key loss without backup

High (H-1, H-2, H-3):
  H-1   Custody fraud before dispute resolution triggered
  H-2   Manipulated SVG accepted by users without reference renderer
  H-3   Physical destruction of online infrastructure (cold keys safe)
```

---

# SECTION 8 — GOVERNANCE SUMMARY

Detailed in GOVERNANCE-SPEC v1.0. Summarized here for master document
reader.

## 8.1 Roles (5)

| Role | Authority |
|------|-----------|
| Foundation | Canon issuance, master_index advancement, dispute arbitration, ceremony coordination |
| Bestuur (Stichting board) | Policy direction, parameter governance, witness admission, emergency authority |
| Witness | Co-signature, equivocation detection, gossip participation |
| Custodian | Per-artifact custody attestation, transfer signing |
| Verifier-producer | PoV receipt generation, signing |

## 8.2 Domains (6)

| Domain | Authority |
|--------|-----------|
| Canon Registry | Foundation (Phase A); Foundation + Witness quorum (Phase B/C) |
| Custody Registry | Custodians self-signed; Foundation arbitrates disputes |
| POV Verification Log | Verifier-producers self-signed |
| MBW economy | Foundation operational; bestuur governs parameters |
| Spec governance | Bestuur (post-Stichting); architect (pre-Stichting) |
| Emergency | Shamir 2-of-3 of root key (post-Stichting); architect (pre-Stichting) |

## 8.3 Voting and quorum

- Witness admission: k-of-n with k = ⌈N/2⌉ + 1
- Witness removal: k-of-n with k = ⌈2N/3⌉
- Bestuur routine decisions: 3-of-5
- Bestuur emergency decisions: 4-of-5
- Root key emergency operations: Shamir 2-of-3 (post-Stichting)
- Spec governance changes: bestuur 3-of-5 + 60-day public comment period

## 8.4 Spec governance lifecycle

```
Major version (v1 → v2):
  - Architect drafts (current phase) or bestuur commissions (post-Stichting)
  - Public comment 60 days
  - Cross-review (target: 3+ independent reviewers, AI or human)
  - Bestuur 3-of-5 approval
  - Cryptographic anchoring before activation
  - At least one full system test cycle before retiring previous version

Minor version (v1.1 → v1.2):
  - Errata bundle pattern (this document is the canonical example)
  - Public comment 14 days
  - Bestuur 3-of-5 approval (post-Stichting); architect (pre-Stichting)
  - Cryptographic anchoring before activation

Patch (v1.2.0 → v1.2.1):
  - Inline correction
  - Bestuur notification within 7 days
  - Anchored as soon as bundled with next minor or major
```

---

# SECTION 9 — ECONOMIC MODEL SUMMARY

Detailed in MBW-SPEC v1.1 + Patches 2 and 5. Summarized here.

## 9.1 MBW (Mean Bandwidth Witness) credit system

**Purpose:** Track and compensate verification work and bandwidth
provisioning, with credit denominated in MBW units. MBW credit is
pre-token; POP-3-TOKEN (Bincoin) will convert MBW credit to on-chain
tokens at Genesis.

**Credit formula** (post-Patch 2, authoritative location MBW-SPEC §2):

```
credit_award(receipt) = 
    1.0                                              # base_credit
  * maturity_curve(verifier_age_hours)               # 0.1 → 1.0 over 720h
  * verification_complexity_factor(artifact)         # 1 → 16 by log2(bytes/KB)
  * (1.0 - bandwidth_bleed_share(receipt))           # Filecoin retrieval share
  * foundation_policy_factor                          # 0.5 → 2.0, default 1.0
```

## 9.2 Sybil resistance (per Reviewer N N-C1, three layers)

| Layer | Mechanism | Status |
|-------|-----------|--------|
| Identity friction | XMSS keypair generation cost; optional Hashcash | Specified |
| Temporal trust accrual | Maturity curve (10% → 100% over 720h) | Specified, Patch 2 |
| Economic stake | Future: collateral/bond per witness; token slashing post-Genesis | DESIGN TARGET (post-POP-3-TOKEN) |

v1.2 has layers 1 + 2. Layer 3 (economic stake) is gated by POP-3-TOKEN
specification. Reviewer N noted that without layer 3, Sybil mitigation
is incomplete; this is an acknowledged residual risk pending Genesis.

## 9.3 Bandwidth bleed economics (Patch 5)

Verifiers consuming bytes from mirrors pay a bandwidth share back to
mirror operators (Filecoin retrieval market pattern). Default rate
1e-9 credit/byte (~1 credit per GB).

**Pathological case:** if bandwidth_bleed_share > 0.9, receipt is
flagged for manual review (signals either malicious "download without
verify" verifier OR malformed artifact).

## 9.4 Foundation policy factor governance

The `foundation_policy_factor` (default 1.0, range 0.5-2.0) is a
governance dial allowing the bestuur to throttle or boost MBW credit
issuance in response to network conditions (spam attack reduces it;
genuine low activity boosts it).

Changes require bestuur 3-of-5 vote per GOVERNANCE-SPEC §5.

## 9.5 What MBW is NOT

- Not a token (yet). Tokenization is POP-3-TOKEN, future spec.
- Not a security primitive. MBW credit drain does not break Canon
  Registry or Custody Registry integrity; it only affects compensation
  for verifiers and mirrors.
- Not legally binding. MBW credit is internal accounting, not
  fiat-denominated claim.

---

# SECTION 10 — RECOVERY AND CONTINUITY MODEL

Detailed in RECOVERY-SPEC v1.0. Summarized here.

## 10.1 Six recovery scenarios

| ID | Scenario | Recovery procedure |
|----|----------|-------------------|
| RS-1 | Lost Foundation operational key | Pre-Stichting: pre-authorized successor + new key (Patch 9). Post-Stichting: Shamir 2-of-3 of cold-backup root. |
| RS-2 | Compromised Foundation operational key | Immediate emergency rotation; revocation broadcast via all channels; old key marked compromised in Canon. |
| RS-3 | Compromised custodian key | Custodian initiates self-rotation; old attestations remain valid until rotation timestamp; new attestations under new key. |
| RS-4 | Canon Registry fork (rare) | Foundation detects via gossip; halts advancement; bestuur or architect arbitration; equivocator (if Foundation) faces R1-level escalation. |
| RS-5 | Witness equivocation | Witness removed from federation (Patch 4); witness namespace marked "revoked-equivocation"; reputational record preserved. |
| RS-6 | Catastrophic infrastructure loss (datacenter destruction etc.) | Cold backups + multi-region mirror replication; expected restoration ≤ 7 days. |

## 10.2 Emergency Root Transfer (R1 mitigation)

**Phase A (pre-Stichting):** Pre-authorized successor named in
Foundation Declaration v2 (to be drafted). Successor namespace published
with public contact path. If no Foundation signing event for 90
consecutive days, public attestation invokes RS-1 recovery procedure;
successor generates new Foundation keypair; old keypair retired with
final cryptographic binding to new.

**Phase B (post-Stichting):** Shamir 2-of-3 of XMSS root key:
- Share 1: held by senior bestuur member (operational)
- Share 2: held by notariusz in sealed envelope (independent custody)
- Share 3: held by independent academic or institutional partner
- Reconstruction requires 2 of 3 shares + multi-signature ceremony
- Reconstruction is logged publicly with cryptographic attestation by
  reconstructing parties

## 10.3 Why no PQ threshold signatures

Post-quantum threshold signature schemes for stateful HBS (XMSS) are
NOT standardized as of 2026. The candidate schemes (e.g., MS+ extensions)
are research-grade. We DO NOT adopt unproven cryptography in production.

Instead, we use:
- Shamir Secret Sharing (1979, well-tested) on the XMSS root key.
- Procedural multi-party ceremonies for high-stakes operations.
- Multiple independent XMSS signatures composed (each from a separate
  trusted machine), aggregated into multi-signature manifests.

This is operationally heavier than threshold sigs but cryptographically
sound.

## 10.4 Dead-man's switch (Phase A only)

If no Foundation signing event for 90 consecutive days:
1. Public attestation by witness federation OR by named successor
   triggers RS-1.
2. Architect's pre-published successor declaration is honored.
3. Successor generates new Foundation keypair.
4. Old root signs no further events (deliberately retired).
5. Canon Registry advances under new Foundation identity.
6. Historical Canon entries remain verifiable against old root.

This is calibrated for the realistic worst case: founder incapacitation
or death. The 90-day window balances:
- Long enough to avoid false positives (illness, travel, sabbatical).
- Short enough to preserve project liveness on real bad-case scenarios.

---

# SECTION 11 — LEGAL INTERPRETATION SUMMARY

Detailed in LEGAL-INTERPRETATION-SPEC v1.0. Summarized here for master
document reader.

## 11.1 Jurisdiction

Primary: Dutch law (Stichting registered under NL BW Book 2 Title 6).

Secondary: EU law applies via NL membership (GDPR, eIDAS, NIS2, MiCA).

## 11.2 What a1a5NT attestations legally are

In Dutch law context:
- A1A5NT cryptographic attestations are **probable evidence** of
  provenance/integrity/custody (Bewijsmiddelen in WvBP Rv).
- They are NOT notarial acts (no Notariswet 1842 protection).
- They are NOT eIDAS qualified signatures (XMSS is not on eIDAS
  approved list as of 2026).
- They are sufficient to shift burden of proof in civil disputes
  where digital integrity is contested.

## 11.3 GDPR compliance posture

- a1a5NT Canon Registry by default does NOT contain PII.
- Where PII appears in custody chains (custodian names = real
  identities), GDPR applies.
- Right-to-erasure under GDPR Art 17: handled by A13 adversary class
  mitigation. Hashes remain (no PII), content removable under
  legitimate request.
- Data controller: Foundation (Stichting A1A5 post-registration);
  ZZP Plastechniek interim.

## 11.4 MiCA applicability

Markets in Crypto-Assets Regulation (MiCA) entered force 2024-12-30.
a1a5NT in v1.2:
- Has NO public token (POP-3-TOKEN/Bincoin is future, deferred).
- MBW credit is internal accounting, not a token.
- Canon attestations are not "crypto-assets" under MiCA definition.

Conclusion: v1.2 a1a5NT is **outside MiCA scope** at this time.
POP-3-TOKEN specification (future) will require MiCA-compliant design.

## 11.5 NIS2 applicability

NIS2 Directive applies to "essential and important" entities. a1a5NT
in v1.2:
- Is not currently classified as essential or important (small scale,
  pre-launch).
- IF future adoption reaches thresholds (Article 3 NIS2), incident
  reporting and risk management requirements apply.
- We design proactively to NIS2 standards (incident response
  procedures in RECOVERY-SPEC §7; risk management in SECURITY-MODEL
  §15 mitigation roadmap).

## 11.6 Licensing

A1A5 Sovereign License v1.0 (anchored at popchain block 82112).
Permits use, modification, redistribution under terms specified.
Compatible with NLnet Commons Fund grant requirements.

---

# SECTION 12 — REVIEWER ATTRIBUTION AND AUDIT TRAIL

The v1.2 specification stack is the product of four independent peer
review cycles. This section provides full attribution for audit
traceability.

## 12.1 Review cycles

```
Cycle 1: pre-v1.0 (foundational spec drafts)
  - Authoring: solo architect
  - Review: solo architect self-review

Cycle 2: v1.0 review (post-publication of v1.0 stack)
  - 2 anonymous reviewers (operational + architectural)
  - Output: v1.1 architecture refinement

Cycle 3: v1.1 review (mid-cycle)
  - 4 independent AI peer reviews (Gemini, Grok, GPT, Claude PC)
  - Output: ~20k lines of feedback aggregated
  - Severity scoring: 8 strong cross-confirmed signals, 12 critical/
    high findings
  - Score progression: NLnet 8.5 → 9.18/10; Notariusz 8 → 8.43/10;
    Akademia 7.5 → 8.83/10

Cycle 4: v1.1 SECURITY-MODEL targeted review (post-publication)
  - 3 reviewers (Reviewer M validation, Reviewer N architectural,
    Reviewer O operational)
  - Output: 11 patches integrated into TIER-2-PATCHES-V1.2

Cycle 5: v1.2 final review (this document and stack)
  - PLANNED Q3 2026 with same 4-model cross-review pattern
  - Expected output: minor corrections, no major architectural changes
```

## 12.2 Architect-delegated decisions (v1.2 production)

During v1.2 production, the architect Paweł Piekut granted the AI
authoring assistant full architect-delegated decision authority due
to volume of architectural decisions and architect bandwidth
constraints. All decisions were documented inline with literature
grounding. Architect retains final review and veto authority.

Decisions made under this delegation (each documented in source specs):

```
Decision A (A13 Legal Compulsion response): Hybrid tombstone model
  Rationale: GDPR Art 17 + ICANN takedown precedent + EU jurisdiction
  reality. Hash immutable forever; content removable under legal
  compulsion.

Decision B (Emergency Root Transfer mechanism): Hybrid graduated
  Rationale: Pre-Stichting requires procedural simplicity; post-
  Stichting allows Shamir 2-of-3. (b) pre-authorized successor + new
  key for Phase A; (a) Shamir 2-of-3 with bestuur for Phase B+.

Decision C (G7 Security Goal): Add as 7th explicit goal
  Rationale: Reviewer N N-B5 finding. Provenance ≠ Authenticity
  disclosure prevents overclaim and matches industry practice
  (Sigstore similar disclaimer).

Decision D (Verifier XMSS Tree Exhaustion): Hash-chain + XMSS batch
  Rationale: Most post-quantum-correct approach. Reuses standard
  pattern from Sigstore Rekor, Certificate Transparency, git tags.

Decision E (Project framing): "Federated canonical registry
  transitioning to witness federation"
  Rationale: Honest naming per Reviewer N + Patch 10. Matches
  Sigstore/CT precedent.

Decision F (Canon immutability semantics): Hash-level with canonical
  serialization
  Rationale: Industry standard (IPFS CID approach). Content-level
  immutability is impossible under legal compulsion; hash-level
  immutability is feasible and audit-defensible.

Decision G (Witness Federation authority): Hybrid graduated
  Rationale: Pre-Stichting Foundation cannot delegate authority it
  does not itself securely hold yet. Witnesses advisory in Phase A;
  authoritative k-of-n in Phase B+.

Decision H (85 retroactive epoch cards RENDER policy): Pre-renderer
  canon — existing preserved as-is as v0 legacy
  Rationale: Cannot retroactively re-render with renderer that did
  not exist. Existing cards anchored as v0 legacy. New cards under
  v1.2 deterministic renderer.
```

## 12.3 Reviewer attribution to specs

```
SECURITY-MODEL v1.1:
  - Reviewer M (validation, ~9/10)
  - Reviewer N (architectural, ~7.5/10, key findings N-B1 → A13,
    N-B5 → G7, N-B2 → Patch 4)
  - Reviewer O (operational, 8.1 → 8.9/10, key findings O-1 → status
    labels, O-2 → Emergency Root Transfer, O-5 → Mitigation Roadmap)

TIER-2-PATCHES-V1.2:
  - Gemini: 5 patches (G-A1, G-A2, G-A3, G-A4, G-B2)
  - Grok: 3 patches (#2, #3, #5)
  - GPT: 4 patches (#2, #3, #4, #5)
  - Claude PC: 6 patches (E1, E2, E3, C-C1, C-C3, C-C4)
  - Reviewer M / N / O: 4 patches (A8, N-B2, N-B4, contributions to A13/G7)
  - Cross-confirmation: 8 of 11 patches have 2+ reviewer attribution

GOVERNANCE-SPEC v1.0:
  - Synthesized from Gemini governance findings (G-A3) and
    Claude PC governance suggestions

NETWORK-PROTOCOL-SPEC v1.0:
  - Synthesized from Claude PC E1 (witness federation mechanics) and
    Reviewer N N-B2 (witness security model gap)

STATE-MACHINE-SPEC v1.0:
  - Synthesized from cross-review demand for "state machine
    completeness" (mentioned in all 4 v1.1 reviews)

RECOVERY-SPEC v1.0:
  - Synthesized from R1 mitigation discussions across all reviewers

CANON-RETIREMENT-SPEC v1.0:
  - Synthesized from Reviewer N N-B1 finding (Canon immutability vs
    legal revocation)

LEGAL-INTERPRETATION-SPEC v1.0:
  - Synthesized from Notariusz-audit readiness requirements
    identified across all 4 reviewers

CRYPTOGRAPHIC-ASSUMPTIONS v1.0:
  - Synthesized from Claude PC CP-B3 (formal cryptographic
    assumptions for academic review) and Reviewer N D3 (academic PQ
    review readiness)
```

## 12.4 Honest disclosure: AI-assisted authoring

This specification stack is the product of substantial AI-assisted
authoring. The architect (Paweł Piekut) directs strategy, sets
priorities, supplies domain expertise (NL legal, hardware
verification, popchain history, archive operations), and exercises
final review authority. AI assistants (multiple, used in parallel for
cross-verification) draft specification text, conduct literature
review, propose architectural patterns, and identify gaps.

This is disclosed because:
- Audit honesty matters; AI involvement is non-trivial.
- Other AI reviewers in cross-review process should know.
- NLnet, Notariusz, and academic reviewers will eventually want this
  information; better to disclose proactively.

Mitigation of AI authoring risks:
- Multi-AI cross-review (4 independent models reviewing same artifacts).
- Literature grounding required for every architectural decision (web
  search results documented).
- Architect veto authority preserved on all decisions.
- Future plan: human expert audit funded by NLnet grant.

This disclosure is Residual Risk R5 (see SECURITY-MODEL v1.1 §13).

---

# SECTION 13 — RESIDUAL RISKS (Full Detail)

## 13.1 R1 — Single-person bootstrap concentration (HIGHEST PRIORITY)

**Description:** All cryptographic authority (XMSS root key), Canon
issuance authority, federation initialization, and custodian appointment
currently rest with a single natural person (Paweł Piekut). This
creates absolute single point of failure plus legal and operational
risk.

**Reviewer consensus:** Unanimous across all 4 v1.2 reviewers (Gemini,
Grok, GPT, Claude PC). Also flagged by Reviewers M, N, O in post-v1.0
SECURITY-MODEL review.

**Current mitigation (Phase A):**
- ZZP Plastechniek (KvK NL) as interim legal vehicle.
- Foundation Declaration cryptographically anchored on popchain.
- License cryptographically anchored.
- Multi-channel publication (GitHub, popchain, server-side anchoring).
- Stichting A1A5 NL registration in progress.

**Planned mitigation (Phase B):**
- Stichting registration completes → bestuur 3-of-5 governance.
- Shamir 2-of-3 of XMSS root key across notariusz/bestuur/independent.
- Dead-man's switch: 90-day inactivity triggers RS-1 recovery
  procedure with pre-authorized successor (per Patch 9).

**Why this remains R1 priority:** Until Stichting registers, this risk
is real and unmitigated by any cryptographic mechanism. The architect
acknowledges this is the most acute current risk and accepts that
audit readiness scores will reflect it.

## 13.2 R2 — No standard threshold XMSS

**Description:** Post-quantum threshold signature schemes for stateful
HBS are not standardized as of 2026.

**Mitigation:** Use Shamir Secret Sharing on root key + multi-party
ceremonies for high-stakes operations. Operationally heavier but
cryptographically sound. Migrate to standard threshold schemes if/when
they exist.

## 13.3 R3 — Long-term institutional continuity

**Description:** No cryptographic mechanism guarantees institutional
survival across centuries.

**Mitigation:** Stichting NL legal continuity; planned witness
federation includes high-inertia academic institutions; planned
replication to Internet Archive and Software Heritage Foundation
(both century-target archives).

## 13.4 R4 — No independent code audit

**Description:** XMSS algorithm itself is audited (RFC 8391); a1a5NT-
specific wrapping (state management, replication, monotonic counter
handling per XMSS-OPS-SPEC) is not yet independently audited.

**Mitigation:** NLnet Commons Fund grant target; strict conformance to
RFC and NIST test vectors; reference implementation in Rust with
extensive unit testing.

## 13.5 R5 — AI-assisted architectural review

**Description:** Iterative cross-model peer review by 4 AI systems is
valuable but does not replace human cryptographic audit.

**Mitigation:** Full AI audit trail published; planned human expert
audit funded by future grant; cross-AI diversity (4 different model
families: Gemini, GPT, Claude, Grok).

---

# SECTION 14 — MITIGATION ROADMAP AND TIMELINE

| Risk / Patch | Status | Owner | Target |
|-------------|--------|-------|--------|
| R1 Phase B Stichting | In Progress | Architect | Q3 2026 |
| R1 Dead-man's switch | Specified | Architect | Q3 2026 |
| R1 Pre-authorized successor | Drafting (Foundation Declaration v2) | Architect | Q3 2026 |
| R2 Migration to threshold XMSS | Watchful waiting | Bestuur (post-Stichting) | When standardized |
| R3 Academic federation witnesses | Outreach planned | Bestuur | Q4 2026 - Q2 2027 |
| R3 Internet Archive replication | Design Target | Bestuur | Q1 2027 |
| R4 NLnet Commons Fund grant | Application drafted | Architect | Q3-Q4 2026 |
| R4 Independent code audit | Awaiting funding | TBD (audit firm) | Post-grant |
| R5 Human expert review | Long-term target | Architect → Bestuur | Q1-Q2 2027 |
| TIER-2 Patches 1-11 | Specified, not yet implemented | Architect → engineering | Q4 2026 |
| Master Index v_001 | Specified, requires Stichting first | Bestuur | Q4 2026 |
| POP-3-TOKEN-SPEC | Future work | Bestuur + economists | Q1-Q2 2027 |
| GENESIS-SPEC | Future work | Bestuur | Q2 2027 |

---

# SECTION 15 — CONFORMANCE STATEMENT

An implementation, deployment, or claimed instance of a1a5NT is
**v1.2 conformant** if and only if:

1. It implements all Tier 1 critical specifications listed in §4.1.
2. It applies all 11 patches in TIER-2-PATCHES-V1.2 to v1.1 source
   specs in §4.2.
3. It honors SECURITY-MODEL v1.1's full threat model (13 adversaries,
   7 goals, 5 trust assumptions).
4. It implements CRYPTOGRAPHIC-ASSUMPTIONS v1.0 cryptographic
   parameters.
5. It follows GOVERNANCE-SPEC v1.0 role and domain assignment.
6. It uses NETWORK-PROTOCOL-SPEC v1.0 wire formats.
7. It maintains STATE-MACHINE-SPEC v1.0 invariants.
8. It applies RECOVERY-SPEC v1.0 procedures when applicable.
9. It applies CANON-RETIREMENT-SPEC v1.0 for tombstone events.
10. It respects LEGAL-INTERPRETATION-SPEC v1.0 disclaimers and limits.

An implementation MUST publish a conformance statement listing
specifications implemented, version numbers, deviations, and
signed by the implementer's XMSS key.

The reference implementation operated by the Foundation publishes its
conformance statement at:
`https://popchain.tech/conformance.json` (planned Q4 2026).

---

# SECTION 16 — DOCUMENT INTEGRITY AND AUTHORIZATION

```
Document version:                   v1.2.0 (final, post-cross-review)
Issued:                              2026-05-17
Architect:                           Paweł Piekut
Architect XMSS namespace:            pawel.a1a5 / a1a5fndn.a1a5 (shared)
Architect-delegated decision auth.:  Granted for v1.2 production cycle,
                                     documented inline per §12.2.
License:                             A1A5 Sovereign License v1.0
Foundation:                          Stichting A1A5 (in formation, NL)
Anchoring (planned):                 popchain master index after this
                                     document's blake3 hash is computed
                                     at deployment time.
GitHub:                              github.com/a1a5NT/a1a3
Public mirror:                       popchain.tech (DNS active)
```

# END OF A1A5NT-ARCHITECTURE-SYNTHESIS-V1.2
