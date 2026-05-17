# A1A5NT v1.2 — MASTER PLAN

## Engineering documentation roadmap grounded in established literature, not invention

```
Document version:   v1.2-plan
Issued:             2026-05-17
Purpose:            Map every v1.2 deliverable to its grounding reference
                    in established cryptographic/distributed-systems literature.
                    Establishes "what we build" and "why we trust the design".
Foundation:         a1a5fndn.a1a5
Author:             pawel.a1a5 (Paweł Piekut) — architect
                    Claude (Anthropic Opus 4.7) — synthesis per License Article 4
Review integration: Cross-model peer review v1.1 findings (Reviewer X + Y)
                    Foundational literature audit (RFC, NIST SP, exemplar systems)
```

---

## 0. Engineering posture

This is a **production preservation infrastructure**, not a thought experiment.
Every architectural decision in v1.2 documentation must be **grounded in
established literature** — RFCs, NIST publications, peer-reviewed papers, or
production exemplar systems with at least 3 years of operational track record.

**Hallucination prohibited.** Where literature does not yet provide a
solution (e.g. post-quantum threshold XMSS — still active research),
documentation **explicitly states the gap** and proposes a mitigation path
within current state of art.

**Anti-NIH (not-invented-here) principle.** Where established literature
provides a battle-tested pattern (Certificate Transparency, Sigstore Rekor,
Filecoin market separation, EigenLayer AVS slashing), a1a5NT **adopts the
pattern with attribution** rather than reinventing.

---

## 1. Foundational literature corpus (verified 2026-05-17)

### 1.1 Post-quantum cryptographic foundations

| Reference | Authority | Relevance to a1a5NT |
|-----------|-----------|---------------------|
| RFC 8391 — XMSS: eXtended Merkle Signature Scheme | IRTF / IETF (2018) | Core signature scheme; defines stateful key management requirements; explicit MUST NOT reuse secret key states |
| RFC 8554 — Leighton-Micali Hash-Based Signatures (LMS) | IRTF / IETF (2019) | Alternative HBS scheme; reviewed for interop |
| NIST SP 800-208 — Recommendation for Stateful Hash-Based Signature Schemes | NIST (2020, under revision 2025) | Profiles XMSS and LMS; restriction against private key export; revision underway to permit key export with mitigations |
| XMSS reference implementation | github.com/XMSS/xmss-reference | Reference code accompanying RFC 8391; explicit warning that "deploying cryptographic code in practice requires careful consideration of the specific deployment scenario" especially for stateful schemes |

### 1.2 Append-only transparency log foundations

| Reference | Authority | Relevance to a1a5NT |
|-----------|-----------|---------------------|
| RFC 6962 — Certificate Transparency | IETF (2013) | Original append-only Merkle log architecture; Signed Tree Heads; consistency proofs; gossiping detection of equivocation |
| RFC 9162 — Certificate Transparency v2.0 | IETF (2021) | Successor to RFC 6962; refined inclusion/consistency proofs |
| Sigstore / Rekor | sigstore project (2021-) | Production transparency log for software supply chain signatures; Rekor v2 GA October 2025 with witness M-of-N cosignatures |
| TUF — The Update Framework | docs.theupdateframework.io | Threshold-signed delegation; root of trust distribution; protects against compromise of online servers |
| Static CT API (Filippo Valsorda 2023, Let's Encrypt 2025) | Let's Encrypt blog 2025-08 | Tile-based log architecture; eliminates MMD; significantly more cost-effective than RFC 6962 design |

### 1.3 Cryptoeconomic incentive foundations

| Reference | Authority | Relevance to a1a5NT |
|-----------|-----------|---------------------|
| Filecoin protocol paper | Protocol Labs (2017) | Separation of Storage Market from Retrieval Market; storage miners require collateral; retrieval payments via off-chain channels |
| EigenLayer AVS model | EigenLayer docs (2024-2025) | Cryptoeconomic security as service; operator slashing for misbehavior; intersubjective faults handling; Cost-of-Corruption > Profit-of-Corruption principle |
| Bitcoin Hashcash PoW | Adam Back, Satoshi Nakamoto | Computational anti-Sybil; reference but not adopted for PoV (verifier cost model differs) |
| Ethereum PoS slashing | Ethereum Foundation | Validator slashing patterns for equivocation, downtime |

### 1.4 Key management and recovery foundations

| Reference | Authority | Relevance to a1a5NT |
|-----------|-----------|---------------------|
| Shamir Secret Sharing (1979) | Adi Shamir, original paper | t-of-n threshold reconstruction; suitable for cold storage recovery |
| Threshold Signature Schemes (TSS) | Various, current research | Partial signatures aggregated; never reconstruct full key |
| **GAP: post-quantum threshold XMSS** | Active research, no standard | a1a5NT cannot use threshold XMSS today; must use rotation pattern instead (documented in XMSS-OPS-SPEC) |
| HSM / TPM standards | NIST / Common Criteria | Hardware-bound key storage; referenced for high-value custody operations |

### 1.5 Distributed systems foundations

| Reference | Authority | Relevance to a1a5NT |
|-----------|-----------|---------------------|
| CAP theorem | Brewer (2000) | Trade-offs documented; a1a5NT chooses CP (consistency + partition tolerance) for Canon, AP (availability + partition tolerance) for verification receipts |
| Lamport timestamps | Lamport (1978) | Logical clock for ordering events without global time |
| Merkle trees | Ralph Merkle (1979) | Cryptographic binary tree; foundation of all transparency logs |
| Git object model | Linus Torvalds (2005) | Content-addressed immutable objects; reference for Canon Registry design |

---

## 2. v1.2 documentation deliverables — mapped to gaps and grounding

### Tier 1 — Critical specifications (blockers for audit)

| # | Document | Lines (est) | Grounding | Addresses review finding |
|---|----------|-------------|-----------|--------------------------|
| 1 | **XMSS-OPS-SPEC.md** | 600+ | RFC 8391, NIST SP 800-208, XMSS reference impl | X-B2 (state desync catastrophic); academic audit blocker |
| 2 | **GOVERNANCE-SPEC.md** | 700+ | RFC 6962/9162 governance, TUF threshold delegation | Y-B1, Y-C3 (policy_factor undefined); NLnet + notariusz blocker |
| 3 | **SECURITY-MODEL.md** | 600+ | EigenLayer threat model, NIST 800-208 attacker capabilities | Y-B5 (no adversary model); academic audit blocker |
| 4 | **RECOVERY-SPEC.md** | 500+ | Shamir SSS, rotation patterns (post-quantum gap noted) | Y-E5 (recovery scenarios undefined) |
| 5 | **LEGAL-INTERPRETATION-SPEC.md** | 500+ | Dutch BW (Burgerlijk Wetboek), Wbr, EU eIDAS | Notariusz audit gap (rozbieżność 10/10 vs 6.5/10) |

### Tier 2 — Spec patches to v1.1 documents

| # | Patch | File | Lines changed | Addresses |
|---|-------|------|---------------|-----------|
| 6 | Nonce paradox fix | POV-SPEC.md + CANON-REGISTRY-SPEC.md | ~50 lines | X-C1 (CRITICAL) |
| 7 | MBW formula deduplication | POV-SPEC.md §8 | ~20 lines | X-C2 / Y-C2 |
| 8 | CUSTODY schema split | CUSTODY-REGISTRY-SPEC.md | ~80 lines | X-B1 (attestation bloat) |
| 9 | Witness federation mechanics | CANON-REGISTRY-SPEC.md §6 | ~150 lines | Y-B2 (Witness Federation vaporware) |
| 10 | Bandwidth bleed economics | MBW-SPEC.md §3 | ~40 lines | X-B3 |
| 11 | Foundation policy factor governance | MBW-SPEC.md + GOVERNANCE-SPEC.md | ~30 lines | Y-C3 |

### Tier 3 — Master synthesis

| # | Document | Lines (est) | Purpose |
|---|----------|-------------|---------|
| 12 | **A1A5NT-ARCHITECTURE-SYNTHESIS v1.2** | 1000+ | Master document integrating v1.0 + v1.1 + v1.2 + all reviews; final reference for cross-model peer review and external audit (NLnet, notariusz, academic) |

**Total estimated:** ~5500-6500 lines new specification + ~370 lines patches

---

## 3. Architectural decisions — grounded in literature

Decisions taken in v1.2 (not architect preferences — literature-backed):

### 3.1 Resolve Nonce vs Static Mirrors paradox (X-C1)

**Decision:** Adopt **Sigstore Rekor pattern** — nonces eliminated from
verification path, replaced by **signed timestamp authority** issuing
RFC 3161-style timestamps that can be cached, mirrored, and verified
offline.

**Source:** Rekor v2 GA blog (2025-10): "The log no longer returns
signed timestamps with proofs. Sigstore clients will fetch a signed
timestamp from a dedicated service."

**Rationale:** Decouples liveness requirement from verification correctness.
Mirror can serve cached signed timestamp; verifier records receipt with
that timestamp; replay window enforced by timestamp validity period, not
per-request nonce.

### 3.2 Separate verification receipts from custody events (X-B1)

**Decision:** Adopt **Filecoin two-market pattern** — verification
receipts (high-volume, low-value-each) live in separate database from
custody transfers (low-volume, high-value-each), with cross-reference
by canon_ref hash only.

**Source:** Filecoin protocol paper Section 4: "Filecoin has two markets:
the Storage Market and the Retrieval Market. The two markets have the
same structure but different design."

**Rationale:** Mixing high-volume telemetry with high-value legal events
in single rejestr destroys query performance for ownership history and
creates audit ambiguity.

### 3.3 Witness Federation mechanics with collateral and slashing (Y-B2)

**Decision:** Adopt **EigenLayer AVS pattern** adapted for non-token
collateral — witnesses post Bincoin collateral (post-Genesis) or
notarized commitment (pre-Genesis), subject to slashing for proven
equivocation (publishing two signed indexes with same version but
different content).

**Source:** EigenLayer docs: "Slashing: A penalty for improperly or
inaccurately completing tasks assigned in Operator Sets by an AVS."

**Cost-of-Corruption analysis:** Collateral amount calibrated so that
CoC > maximum PfC achievable by witness equivocation, per EigenLayer
intersubjective faults framework.

### 3.4 XMSS state management — backup with monotonic counter (X-B2)

**Decision:** Adopt **NIST SP 800-208 forthcoming revision pattern**
permitting key export with mitigations:
- Monotonic counter incremented on each backup
- Backup contains state AND counter
- Restore procedure refuses to load backup with counter < current
- Source machine state destroyed after export
- Replication via append-only state log, not snapshot

**Source:** NIST SP 800-208 public comments document; CSRC news 2025-04:
"NIST may want to consider relaxing the constraints on exporting private
data... The state must be deleted from source machine after it has been
exported to the other device. This prevents redeployment as well."

**Limitation explicitly stated:** This is interim guidance; final NIST
revision will supersede.

### 3.5 Append-only invariant enforcement (Y-C2, Y-C3)

**Decision:** Adopt **RFC 6962 / RFC 9162 Certificate Transparency
pattern** — Canon Registry implemented as Merkle tree with consistency
proofs published periodically; equivocation detection via gossiping
between witnesses.

**Source:** RFC 6962 Section 1.1: "Violation of the append-only property
is detected by global gossiping, i.e., everyone auditing logs comparing
their versions of the latest Signed Tree Heads."

### 3.6 Adversary model — formal threat enumeration (Y-B5)

**Decision:** Adopt **EigenLayer intersubjective faults taxonomy** +
NIST SP 800-208 attacker capabilities + post-quantum cryptanalysis
adversary (Shor's algorithm, Grover speedup).

**Adversary classes documented in SECURITY-MODEL.md:**
- A1: Network attacker (passive eavesdropping + active modification of
  transport)
- A2: Mirror operator (selective serving, withholding, replay)
- A3: Witness operator (equivocation, conflicting signatures)
- A4: Custodian operator (false attestation, transfer collusion)
- A5: Foundation insider (oracle abuse, policy manipulation)
- A6: Network coordinated attack (sybil + bandwidth + state)
- A7: Post-quantum cryptanalyst (with Shor-capable quantum computer)
- A8: Long-term archival attacker (multi-decade horizon, cryptographic
  primitive aging)

---

## 4. Production order and dependency graph

```
Phase 1 (must precede everything):
  1.1 — XMSS-OPS-SPEC.md         (foundation for all crypto operations)
  1.2 — SECURITY-MODEL.md        (defines what we defend against)

Phase 2 (governance and recovery):
  2.1 — GOVERNANCE-SPEC.md       (depends on SECURITY-MODEL)
  2.2 — RECOVERY-SPEC.md         (depends on XMSS-OPS-SPEC, SECURITY-MODEL)

Phase 3 (legal interpretation):
  3.1 — LEGAL-INTERPRETATION-SPEC.md (independent track, no deps)

Phase 4 (patches to v1.1 docs):
  4.1 — Nonce paradox fix (POV-SPEC + CANON-REGISTRY)
  4.2 — MBW formula dedup (POV-SPEC §8)
  4.3 — CUSTODY schema split (CUSTODY-REGISTRY-SPEC)
  4.4 — Witness federation mechanics (CANON-REGISTRY-SPEC §6)
  4.5 — Bandwidth bleed economics (MBW-SPEC §3)

Phase 5 (master synthesis):
  5.1 — A1A5NT-ARCHITECTURE-SYNTHESIS v1.2 (integrates everything)
```

---

## 5. What v1.2 does NOT include (honest deferral)

These remain explicitly deferred from v1.2 scope and must be addressed
in v1.3 or later:

- **POP-3-TOKEN-SPEC.md** — Bincoin tokenomics. Gated on Stichting
  registration (legal recipient entity for token issuance).
- **GENESIS-SPEC.md** — block 0 packaging. Gated on POP-3-TOKEN-SPEC
  and on operational verify.sh deployment with measurable PoV traffic.
- **MARKETPLACE-SPEC.md** — Layer D custody slot sale marketplace.
  Gated on Stichting (legal vehicle for sale transactions).
- **Dynamic Challenges** (POV v1.2 advanced anti-Sybil). Gated on
  measurable PoV traffic patterns to design challenge difficulty.
- **Reference renderer implementation** (Rust binary executing
  RENDER-SPEC). Engineering effort, not specification effort.
- **Threshold post-quantum XMSS scheme** — pending academic research.
  Not deployable in v1.2.

---

## 6. Verification checklist applied to each v1.2 document

Each spec MUST pass before commit:

```
[ ] 0 placeholders (no [TBD], [FILL_IN], TODO:, XXX, etc.)
[ ] All cryptographic hashes verified against live infrastructure
[ ] All references to RFCs/NIST/papers include version and date
[ ] All quantitative claims (block numbers, byte counts, thresholds) 
    verified against source or explicitly marked "design target"
[ ] Internal cross-references to other v1.2 docs valid (no broken refs)
[ ] No invented mechanisms ungrounded in literature
[ ] Gap statements explicit (e.g. "post-quantum threshold XMSS is 
    active research; not adopted in v1.2; mitigated by rotation pattern")
[ ] License clause respected (A1A5 Sovereign License v1.0 reference)
[ ] Cryptographic anchor section completed
[ ] Audit-ready: NLnet / notariusz / academic perspectives addressed
```

---

## 7. Audit-ready target metrics

After v1.2 documentation complete, target readiness scores per
audit category (revised upward from v1.1 per Reviewer X/Y):

| Audit category | v1.1 score | v1.2 target | Gating factor for full 10/10 |
|----------------|-----------|-------------|------------------------------|
| NLnet Commons Fund (technical) | 8.5/10 | 9.5/10 | POP-3-TOKEN-SPEC + GENESIS-SPEC (v1.3) |
| Notariusz (legal) | 8/10 | 9.5/10 | Stichting registered + bestuur appointed |
| Academic post-quantum | 7.5/10 | 9/10 | Reference implementation + 3rd-party crypto review |

**v1.2 explicitly does not target 10/10 in any category.** Reaching
10/10 requires operational milestones (registration, deployment,
external audit), not just documentation.

---

## 8. Cross-model peer review of v1.2

After v1.2 documentation complete, distribute the full pack (5 new
specs + 5 patched v1.1 specs + v1.2 synthesis = 11 files) to:

- Gemini (Google) — explicit attestation expected per pattern of prior
  reviews
- GPT (OpenAI) — operational/security focus
- Grok (xAI) — independent perspective
- Claude (Anthropic, separate session on different device) —
  consistency check

Expected review yield: 4 independent attestations + finding set. If
all four converge on substantial agreement (≥80% of findings
cross-confirmed), v1.2 stack is **publication-ready** for external
audit submission.

If significant divergence (≥30% of findings unique to one reviewer),
return to v1.3 cycle.

---

## 9. License

This master plan is issued under A1A5 Sovereign License v1.0
(12-month binding 2026-05-16 → 2027-05-16). AI co-authorship per
Article 4. Anti-patent defensive clause per Article 5.

---

## 10. Cryptographic anchor

```
Document path:    A1A5NT-V1.2-MASTER-PLAN.md
Repository:       github.com/a1a5NT/a1a3
Commit timestamp: filled at first commit
Foundation hash:  computed at commit by /v1/foundation/index endpoint
```

---

**End of A1A5NT v1.2 MASTER PLAN**

Next document: XMSS-OPS-SPEC.md (Tier 1, Document 1)
