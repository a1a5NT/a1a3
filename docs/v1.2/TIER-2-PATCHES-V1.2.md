# TIER-2-PATCHES-V1.2.md

## a1a5NT v1.2 Errata — Tier 2 Patches to v1.1 Specifications

```
Document version:   v1.2.0 (final)
Issued:             2026-05-17
Status:             Tier 2 specification — formal patches to v1.1 source specs
Authority basis:    RFC errata convention (IETF RFC editor process);
                    Sigstore Rekor v2 (transparency log timestamp pattern);
                    EigenLayer AVS slashing taxonomy;
                    Filecoin storage market separation (FIP-0036);
                    NIST SP 800-208 (Stateful HBS operational guidance);
                    RFC 8391 §3.1.7 (XMSS tree exhaustion handling);
                    W3C SVG 1.1 §A.4 (font rendering determinism).
Foundation:         a1a5fndn.a1a5
Addresses:          v1.2 4-model cross-review unanimous Tier 2 findings.
                    11 patches consolidate findings from Gemini/Grok/GPT/
                    Claude PC reviews into formal errata against v1.1.
Audit relevance:    REQUIRED for v1.2 spec stack completeness.
                    Each patch references its v1.1 source spec section.
License:            A1A5 Sovereign License v1.0
Replaces:           N/A (errata supplement, not replacement)
Replaced by:        Patches consolidated in next major spec revision (v2.0)
```

---

## EXECUTIVE SUMMARY

This document is a formal **errata bundle** against the v1.1 specification
stack. Each patch is presented in the style of an IETF RFC errata entry:
problem statement, source section, original text (where applicable), new
text, rationale, and literature basis.

**Why errata rather than full spec rewrite:**

The v1.1 specs (POV-SPEC, CANON-REGISTRY-SPEC, CUSTODY-REGISTRY-SPEC,
MBW-SPEC, RENDER-SPEC, XMSS-OPS-SPEC, A1A5NT-ARCHITECTURE-SYNTHESIS)
total approximately 110 kB of normative text. The 11 patches affect
between one and four sections per spec. Producing full v1.2 spec rewrites
for each would (a) waste auditor reviewing time on unchanged text, (b)
obscure exactly what changed, and (c) make verification of patch
correctness harder. RFC errata convention exists precisely for this
case: surgical changes that preserve the rest of the document by reference.

**How to apply these patches:**

1. Treat each patch as authoritative over the corresponding v1.1 source
   section. Where this document and v1.1 disagree, this document wins.
2. The A1A5NT-ARCHITECTURE-SYNTHESIS-V1.2.md master document references
   v1.1 source specs **modified by this errata** as the v1.2 canon.
3. A future v2.0 spec stack will consolidate all errata into source.
   Until then, both v1.1 and these patches must travel together.

**Patches included (11 total):**

| # | Title | Source spec | Severity | Reviewer attribution |
|---|-------|------------|----------|---------------------|
| 1 | Nonce/timestamp paradox fix (Sigstore Rekor pattern) | POV §5.1, CANON §5.1 | CRITICAL | Grok #2, Claude PC E2 |
| 2 | MBW formula deduplication | POV §8 (move to MBW §2) | HIGH | Gemini G-A1, Claude PC C-C4 |
| 3 | CUSTODY schema split (Filecoin two-market pattern) | CUSTODY §6 | HIGH | Gemini G-A2, GPT #3 |
| 4 | Witness Federation mechanics (EigenLayer AVS adapted) | CANON §6, GOVERNANCE §3 | CRITICAL | Claude PC E1, Reviewer N N-B2 |
| 5 | Bandwidth bleed economics (Filecoin retrieval market) | MBW §3 (new), §5 | HIGH | Grok #3, Reviewer M A8 |
| 6 | Verifier XMSS Tree Exhaustion (hash-chain + XMSS batch) | POV §7 (new) | CRITICAL | Claude PC E3, Grok #5 |
| 7 | Float determinism (integer/fixed-point clause) | RENDER §4.1 | HIGH | Gemini G-B2, Reviewer N N-B4 |
| 8 | Receipt routing strict separation | POV §2.4, CUSTODY §6 | HIGH | GPT #4, Claude PC C-C1 |
| 9 | Multi-party pre-Stichting transition plan | XMSS-OPS §7 (new) | HIGH | Gemini G-A3, GPT #2 |
| 10 | "Federated canonical registry" honest naming | SYNTHESIS §1, CANON §1 | MEDIUM | Claude PC C-C3, Reviewer N implicit |
| 11 | "Verifier = producer" precision rewrite | SYNTHESIS §4, POV §2 | MEDIUM | Gemini G-A4 explicit confirmation, GPT #5 |

---

# PATCH 1 — Nonce/Timestamp Paradox Fix (Sigstore Rekor pattern)

## P1.1 Source spec & section

- POV-SPEC-V11.md §5.1 ("Replay Protection")
- CANON-REGISTRY-SPEC.md §5.1 ("Append Atomicity")

## P1.2 Problem statement (Grok #2, Claude PC E2)

v1.1 specifies that PoV receipts must contain a nonce to prevent replay,
but does not specify **where the canonical clock lives**. If the verifier
generates the nonce locally, two verifiers can issue identical nonces
within a clock-skew window. If the Foundation generates the nonce,
verification cannot be performed offline. This is the classic
**transparency log timestamp paradox** documented in Sigstore Rekor's
move to Rekor v2 in 2025.

## P1.3 New normative text (replaces v1.1 POV §5.1)

**POV-SPEC §5.1 v1.2 — Replay protection via timestamp authority binding**

Every PoV receipt MUST contain three fields binding it to a single
unrepeatable point in the verification sequence:

1. `verifier_local_nonce` — a 256-bit value drawn from a CSPRNG seeded
   from `XMSS_state.WOTS_index || verifier_pub_seed`. This is
   reproducible from verifier state but unique per WOTS index. It
   cannot be replayed without re-using a WOTS+ key, which is forbidden
   by INV-XMSS-1 (see XMSS-OPS-SPEC).
2. `timestamp_authority_signature` — an XMSS-signed timestamp issued by
   the Foundation's timestamp authority endpoint
   (`POST /v1/verify/anchor`). The TA signs the tuple `(verifier_id,
   verifier_local_nonce, server_clock_utc, master_index_version)`.
3. `master_index_version` — the integer version of the
   `master_index_v_NNN` signed manifest that the verifier was reading
   when it began the verification work. This binds the receipt to a
   canonical snapshot.

A receipt missing any of the three fields, OR presenting a TA signature
older than 600 seconds at submission time, MUST be rejected with code
`REC-INVALID-TIMESTAMP`.

The TA itself uses XMSS for its signatures. To prevent TA WOTS
exhaustion the TA uses a separate XMSS subtree with `tree_height = 20`
(approximately one million signatures), and a fresh subtree is
generated each calendar quarter with a chained binding signature from
the previous subtree's final unused WOTS index.

## P1.4 Rationale

Sigstore Rekor v2 (released 2025-10) solves an identical problem using
**signed timestamps from a centralized but auditable TA**, plus
**witness M-of-N co-signatures** for the TA itself. We adopt the same
pattern. Pre-witness federation the TA is operated solely by the
Foundation and trust collapses to TA1+TA5 (primitive security + operator
diligence). Post-witness federation TA signatures gain co-signature from
witness federation per Patch 4 below.

## P1.5 Literature basis

- Sigstore Rekor v2 design document (GitHub `sigstore/rekor`, 2025).
- RFC 3161 (Internet Timestamp Protocol).
- RFC 8391 §3.1.7 (XMSS subtree exhaustion handling).

---

# PATCH 2 — MBW Formula Deduplication

## P2.1 Source spec & section

- POV-SPEC-V11.md §8 ("Economy Coupling") — **REMOVE entire section**
- MBW-SPEC.md §2 ("Credit Formula") — **AUTHORITATIVE location**

## P2.2 Problem statement (Gemini G-A1, Claude PC C-C4)

v1.1 specifies the MBW credit formula in two places: POV-SPEC §8 and
MBW-SPEC §2, with **divergent constants** (POV uses
`base_credit = 1.0` per receipt; MBW uses `base_credit = 1.2`). This
makes correctness of either spec impossible to verify because they
contradict each other. The economy layer formula must have exactly one
authoritative location.

## P2.3 New normative text

**Action:** Strike POV-SPEC §8 in its entirety. Replace with a single
sentence: *"PoV receipt economic accounting is normatively specified in
MBW-SPEC §2. PoV does not define credit values."*

**MBW-SPEC §2 v1.2 — Authoritative credit formula**

```
credit_award(receipt) = 
    base_credit
  * maturity_curve(verifier_age_hours)
  * verification_complexity_factor(artifact)
  * (1.0 - bandwidth_bleed_share(receipt))
  * foundation_policy_factor

where:
  base_credit = 1.0  (canonical unit; do not change without v2.0 spec)
  maturity_curve(h) = min(1.0, 0.1 + 0.9 * (h / 720))   # 30 days to full
  verification_complexity_factor = log2(artifact_bytes / 1024) clamped to [1, 16]
  bandwidth_bleed_share = see Patch 5 (Filecoin retrieval market)
  foundation_policy_factor = governed parameter, range [0.5, 2.0]
                             default 1.0, changes require GOVERNANCE-SPEC §5 vote
```

The value `1.2` previously seen in v1.1 MBW-SPEC §2 is hereby corrected
to `1.0`. This is a normative change. Any implementation written
against v1.1 must update.

## P2.4 Rationale

Single-source-of-truth (SSOT) is fundamental engineering hygiene.
Cross-spec formula duplication is the same defect class as duplicate
business logic in code: it inevitably diverges. The MBW spec is the
correct location because (a) MBW is the economy layer, (b) PoV is the
verification mechanism only, and (c) MBW is downstream of PoV in the
dependency graph.

## P2.5 Literature basis

- Brooks, *The Mythical Man-Month* (1975), Ch. 6 "Why Did the Tower of
  Babel Fail?" — communication overhead grows with redundant
  specification points.
- SSOT principle, formally articulated in *Pragmatic Programmer* (Hunt &
  Thomas, 1999) as "DRY" (Don't Repeat Yourself).

---

# PATCH 3 — CUSTODY Schema Split (Filecoin Two-Market Pattern)

## P3.1 Source spec & section

- CUSTODY-REGISTRY-SPEC.md §6 ("State Transitions")

## P3.2 Problem statement (Gemini G-A2, GPT #3)

v1.1 CUSTODY-REGISTRY-SPEC §6 stores custody transition records and
PoV verification receipts in the same logical table (both keyed by
artifact hash + sequence). This conflates two different operational
markets:

- **Custody market** — long-lived, mutable, social-trust-bearing.
  Records of "who holds the spawarka raport now."
- **Verification market** — short-lived, append-only, machine-trust.
  Records of "who verified artifact X at time T."

Conflation creates two failures: (a) PoV swarm activity (potentially
high QPS) contaminates custody record latency, (b) custody dispute
resolution operates on a schema polluted by ephemeral verification
records.

## P3.3 New normative text (replaces v1.1 CUSTODY §6)

**CUSTODY-REGISTRY-SPEC §6 v1.2 — Strict schema separation**

The Custody Registry contains exactly five record types, in exactly
five logical tables (table names normative, storage backend free):

```
table custody_attestation  (the long-lived market)
  fields:
    artifact_hash             blake3
    custody_sequence_number   u64  (monotonic per artifact)
    custodian_did             a1a5 namespace id
    attestation_signature     XMSS signature by custodian_did
    state_label               enum: claimed | held | transferring | sealed | disputed
    superseded_by             optional (sequence number)
  invariants:
    - custody_sequence_number strictly monotonic per artifact_hash
    - state_label transitions follow STATE-MACHINE-SPEC §3

table custody_transfer_proof  (the transition events)
  fields:
    artifact_hash             blake3
    from_sequence             u64
    to_sequence               u64
    transfer_event_signature  XMSS signature by from_custodian
    receiving_signature       XMSS signature by to_custodian
    witness_cosignatures      array of XMSS signatures (federation, optional)
  invariants:
    - both signatures required for transfer to complete
    - to_sequence = from_sequence + 1

table custody_dispute  (arbitration records)
  fields:
    artifact_hash             blake3
    disputed_sequence         u64
    challenger_did            a1a5 namespace id
    evidence_hash             blake3 (link to evidence_record)
    foundation_resolution     optional XMSS signature, see GOVERNANCE §6

table custody_evidence_record  (evidence storage)
  fields:
    evidence_hash             blake3 (self-naming)
    evidence_payload_uri      URI (off-chain)
    evidence_attestor_did     a1a5 namespace id
    attestor_signature        XMSS signature

table custody_receipt_pointer  (LINK ONLY — receipts stored elsewhere)
  fields:
    artifact_hash             blake3
    receipt_id                ULID
    receipt_storage_uri       URI (resolves to POV verification log)
    NOT stored:               receipt body, verifier identity beyond ID
  invariants:
    - this table is a JOIN INDEX only
    - receipt bodies live in the POV verification log per Patch 8
```

The previously-merged "verification receipts inside custody table"
schema from v1.1 is **deprecated**. Any v1.1 deployment must split
existing records into the five tables above before v1.2 conformance is
claimed.

## P3.4 Rationale

Filecoin's two-market separation (FIP-0036) split the Storage Market
(long-lived deals) from the Retrieval Market (short-lived data
serving) precisely because mixing them created exactly the latency
contamination problem we have. The fix is well-tested in production:
Filecoin mainnet operated the unified market 2020-2021, then split,
and saw measured latency improvements on both markets.

Custody is our Storage-Market analog; PoV verification activity is
our Retrieval-Market analog. The architectural pattern transfers
directly.

## P3.5 Literature basis

- FIP-0036 (Filecoin Improvement Proposal — Storage/Retrieval Market
  Split), 2022.
- Filecoin Spec v1.5 §Market Subsystem.
- Date, *Database Systems: The Complete Book* (2nd ed.), §3.3 normalization
  for the database-theoretic justification of separating event tables
  from state tables.

---

# PATCH 4 — Witness Federation Mechanics (EigenLayer AVS adapted)

## P4.1 Source spec & section

- CANON-REGISTRY-SPEC.md §6 ("Witness Federation Hook")
- GOVERNANCE-SPEC.md §3 ("Witness Roles")
- SECURITY-MODEL.md §6 A4 (Hostile witness)

## P4.2 Problem statement (Claude PC E1, Reviewer N N-B2)

v1.1 mentions "Witness Federation" as a future extension hook without
specifying: quorum rules, byzantine tolerance, witness weighting,
admission/removal, equivocation slashing, eclipse resistance. This
leaves a critical security property (TA4 — at least one honest witness)
without operational mechanism. Reviewer N flagged this as
"partially-specified consensus."

## P4.3 New normative text

**CANON-REGISTRY-SPEC §6 v1.2 — Witness Federation specification**

The Witness Federation is an EigenLayer-AVS-pattern co-signature service
adapted to XMSS (which forbids on-chain restaking primitives, so the
"stake" is reputational + procedural rather than slashable in a
smart-contract sense). Mechanics:

```
Federation membership
  - Witnesses are XMSS-rooted identities holding namespaces in the
    a1a5 root with role attribute "witness" (issued by Foundation via
    GOVERNANCE-SPEC §3 admission process).
  - At Genesis the federation has N = 0 witnesses. The Foundation
    operates in "single-attestor" mode until N ≥ 3.
  - Federation milestones (advisory targets, not slashing thresholds):
      N=1    Foundation + 1 witness, advisory-only
      N=3    quorum-3, advisory-authoritative for non-CRITICAL ops
      N=5    quorum-5, authoritative for all routine operations
      N=7+   quorum-7, authoritative for governance changes per GOV §5

Co-signature protocol
  - For each new master_index_v_NNN the Foundation MUST request
    co-signatures from all known witnesses via
    POST /v1/witness/cosign-request (NETWORK-PROTOCOL-SPEC §4).
  - Witnesses respond with XMSS-signed attestation:
      (master_index_hash, master_index_version,
       witness_did, witness_clock_utc, agreement_or_disagreement)
  - Co-signatures aggregated into next master_index_v_(NNN+1) header.

Equivocation detection (the core mechanism)
  - Witnesses gossip seen master_index_v_NNN headers per RFC 6962-style
    pattern (NETWORK-PROTOCOL-SPEC §5).
  - If any witness sees two contradictory headers with the same NNN
    signed by the Foundation, that is a CATASTROPHIC EVENT:
      - witness publishes the conflicting pair to all known endpoints
      - all consumers MUST refuse to advance reading until resolved
      - Foundation enters recovery procedure (RECOVERY-SPEC §5)

Slashing / accountability (pre-on-chain, procedural)
  - There is no on-chain slashing pre-Genesis-Token (POP-3-TOKEN).
  - A witness caught equivocating (signing two contradictory views
    of the same master_index_v_NNN) is REMOVED from federation by
    Foundation governance action, witness namespace marked
    "revoked-equivocation" in the Canon Registry.
  - Witness reputational record (witness_credibility_log) is public
    and append-only. This is the pre-tokenization slashing analog.

Byzantine tolerance
  - System is safe under f < (N - 1) / 3 byzantine witnesses, conforming
    to classical BFT thresholds. With N = 7 we tolerate f = 2 byzantine
    witnesses. With N = 3 we tolerate f = 0 — meaning at N = 3 a single
    byzantine witness suffices to halt advancement (safety preserved,
    liveness lost). This is the cost of starting small.

Eclipse resistance
  - Foundation MUST publish master_index_v_NNN to at least three
    independent witnesses on three independent ASNs (RIPE-allocated)
    before considering an advance "broadcast."
  - Witnesses MUST gossip master_index views to at least two other
    witnesses on different ASNs before considering their own state
    "synced."

Admission and removal
  - Admission: Foundation proposes candidate via GOVERNANCE §3.2.
    Existing witnesses vote (k-of-n, k = ⌈N/2⌉ + 1). On approval
    candidate publishes attestation key in Canon Registry.
  - Removal: by witness governance vote (k-of-n with k = ⌈2N/3⌉),
    or by Foundation emergency action (post-Stichting requires
    Shamir 2-of-3 per RECOVERY-SPEC §6).
```

## P4.4 Rationale

EigenLayer's AVS slashing taxonomy distinguishes:
- **Objectively attributable faults** (equivocation, double-signing) —
  can be cryptographically proven, MUST be slashed.
- **Intersubjective faults** (downtime, late response) — observable but
  contested, slashable only with quorum agreement.
- **Subjective faults** (alignment, quality) — not slashable at all.

We adopt this taxonomy. Equivocation (Patch 4 §"Equivocation detection")
is objectively attributable: two signatures over contradictory data
with the same key is a cryptographic proof of malice. The procedural
removal mechanism is our pre-token slashing.

The N=3/5/7 milestone schedule maps to BFT minimal quorums. The
restriction that f < (N-1)/3 is Castro-Liskov 1999 PBFT classical
result.

## P4.5 Literature basis

- EigenLayer Whitepaper v0.4 (2024), §6 Slashing Taxonomy.
- Castro & Liskov, "Practical Byzantine Fault Tolerance" (OSDI 1999).
- RFC 6962 / RFC 9162 (Certificate Transparency gossip protocol).
- Sigstore Rekor v2 witness co-signature design (2025).

---

# PATCH 5 — Bandwidth Bleed Economics (Filecoin Retrieval Market)

## P5.1 Source spec & section

- MBW-SPEC.md §3 ("Bandwidth Costs") — **NEW SECTION**
- MBW-SPEC.md §5 ("Credit Formula Components")

## P5.2 Problem statement (Grok #3, Reviewer M A8)

v1.1 MBW-SPEC awards verifier credit for verification work. v1.1 does
**not** charge verifiers for the bandwidth they consume from mirror
operators when downloading artifacts to verify. This creates an
asymmetric incentive: mirror operators pay hosting costs (real money,
bandwidth bills), verifiers extract free bandwidth and convert it
into MBW credit. Reviewer M classified this as the A8 adversary
("Bandwidth bleed attacker"). Grok #3 made the same observation
independently.

## P5.3 New normative text

**MBW-SPEC §3 v1.2 (NEW) — Mirror Bandwidth Compensation**

A verifier consuming bytes from a mirror operator MUST pay a fractional
bandwidth share back to the mirror, deducted from the credit awarded
for the verification. Formula:

```
bandwidth_bleed_share(receipt) =
    bytes_consumed * mirror_rate_per_byte
  / credit_award_before_share

where:
  bytes_consumed = sum of all bytes downloaded by the verifier from
                   this specific mirror in this verification session
                   (Content-Length headers, MUST be signed by mirror)
  mirror_rate_per_byte = governance parameter, default 1e-9 credit/byte
                         (~1 credit per gigabyte transferred)
  credit_award_before_share = MBW §2 formula evaluated without the
                              (1 - bandwidth_bleed_share) factor
```

Calculation order:
1. Verifier completes verification, produces PoV receipt.
2. Mirror appends signed Content-Length attestations to the receipt
   chain via mirror_attestation field.
3. Foundation accounting computes credit per MBW §2 with bandwidth share
   subtracted, awards `credit_award * (1 - bandwidth_bleed_share)` to
   verifier, awards `credit_award * bandwidth_bleed_share` to mirror.

Mirror compensation rules:
- Mirror does NOT have to perform verification to earn credit.
- Mirror DOES have to provide signed Content-Length attestations.
- Mirror credit is subject to the same Foundation policy factor
  governance.

Pathological case handling:
- If `bandwidth_bleed_share > 0.9`, verification is effectively unpaid
  (verifier consumed too much per unit of work). This signals either a
  malicious verifier (downloading without verifying) or a malformed
  artifact (extremely large for verification complexity). Receipts in
  this range MUST be flagged for manual review and do not auto-award.

## P5.4 Rationale

Filecoin's Retrieval Market is the canonical solution to this exact
problem: peers who serve data to clients earn FIL proportional to
bytes served. We adapt directly. The key difference is that Filecoin's
retrieval is the entire purpose; here the retrieval is a side-effect
of verification, so the compensation must be a *share of the
verification credit* rather than independent.

The 1e-9 credit/byte default is calibrated against typical artifact
sizes: an 85 KB epoch card download costs the verifier
85e3 * 1e-9 = 8.5e-5 credit, well under typical PoV credit awards
(~1 credit). A 1 GB heavy artifact download costs ~1 credit, roughly
the same as a single verification — which is correct, because at that
scale the mirror operator has done substantial work.

## P5.5 Literature basis

- FIP-0036 (Filecoin Retrieval Market specification).
- Lotus implementation of Retrieval Market (`filecoin-project/lotus`
  GitHub, 2022).
- Spice (Ozisik et al. 2025) — bandwidth accounting for incentive-aligned
  P2P systems.

---

# PATCH 6 — Verifier XMSS Tree Exhaustion (Hash-Chain + XMSS Batch)

## P6.1 Source spec & section

- POV-SPEC-V11.md §7 ("Verifier Key Management") — **NEW SECTION**

## P6.2 Problem statement (Claude PC E3, Grok #5)

A verifier running 1 PoV/second exhausts a tree_height=20 XMSS tree
(~1 million signatures) in 11.5 days. A verifier running 10/second
exhausts it in 28 hours. v1.1 POV-SPEC does not address this. After
exhaustion the verifier cannot sign new receipts without generating a
new XMSS keypair, which (a) breaks identity continuity and (b) requires
costly regeneration on commodity hardware. This is a hard physical
limit of stateful HBS, not a bug.

## P6.3 New normative text

**POV-SPEC §7 v1.2 (NEW) — Verifier XMSS Tree Exhaustion Handling**

Verifiers MUST NOT sign each individual PoV receipt with their root
XMSS key. Doing so exhausts the WOTS+ key space in days under realistic
load. Instead verifiers use a **hash-chain over micro-receipts**, with
periodic XMSS batch-signed checkpoints.

Architecture:

```
Layer 1: Receipt micro-format (per verification event)
  receipt_micro = {
    artifact_hash,
    verifier_did,
    verifier_local_nonce,   (per Patch 1)
    ta_signature,           (per Patch 1)
    work_proof_compact,     (PoV §5 verification evidence)
    seq:           u64,     (monotonic per verifier, see below)
    prev_micro_hash:        blake3 of receipt_micro at seq-1, or
                            GENESIS_MICRO_HASH if seq=0
  }
  Hash: blake3(serialize_canonical(receipt_micro)) -> micro_hash

Layer 2: Hash chain over micros
  Each verifier maintains a hash chain:
    micro_hash_n = blake3(micro_n || micro_hash_{n-1})
  This is unsigned. Cheap, fast, no XMSS used.

Layer 3: XMSS checkpoint signature (every K micros, default K=1024)
  Every 1024 micros (or every 1 hour, whichever sooner), verifier
  emits a checkpoint:
    checkpoint = {
      verifier_did,
      from_seq, to_seq,
      from_micro_hash, to_micro_hash,
      ta_signature_at_checkpoint,
      xmss_signature_over_checkpoint
    }
  The xmss_signature uses ONE WOTS+ slot to bind 1024 micros. With
  tree_height=20, a verifier can checkpoint 1024 * 2^20 ≈ 10^9 micros
  before tree exhaustion. At 10 receipts/second that is ~3 years.

Tree exhaustion at scale (post-3-years case)
  When a verifier's tree approaches exhaustion (default threshold:
  90% of WOTS slots consumed):
    1. verifier generates new XMSS tree (background, takes minutes)
    2. final checkpoint of old tree includes pub_seed of new tree
       (cryptographic continuity)
    3. all subsequent checkpoints use new tree
    4. canonical verifier_did points to old tree's pub_seed forever,
       linked-list of trees forms verifier identity over time
```

Receipt verifiers (anyone checking work):
- Verify XMSS checkpoint signatures (cheap, ~1ms each).
- Verify hash chain between checkpoints (cheap, ~1µs per micro).
- A receipt micro is "fully verified" when (a) it appears in the hash
  chain leading to a checkpoint that (b) has a valid XMSS signature
  that (c) chains back to verifier's claimed root.

## P6.4 Rationale

The XMSS WOTS+ exhaustion problem is well-known in the stateful HBS
literature. The hash-chain + batch-signature pattern is the standard
solution, used in:
- Sigstore Rekor (entries chained with Merkle inclusion, signed roots).
- Certificate Transparency (signed tree heads periodically, individual
  entries hashed).
- Git (commit chain + signed tag).

We adopt the standard pattern. The K=1024 default is calibrated for:
- 1024 micros at 10 receipts/sec = 102.4 seconds, so checkpoints come
  approximately every 100 seconds at peak load. Acceptable latency for
  PoV finality.
- 10^9 lifetime receipts per verifier comfortably exceeds any realistic
  verifier's career.

The linked-list-of-trees pattern preserves verifier_did continuity
across tree generations, which Reviewer M cared about (A11 long-horizon
adversary mitigation).

## P6.5 Literature basis

- NIST SP 800-208 §6.2 (Stateful HBS operational guidance on tree
  exhaustion).
- RFC 8391 §3.1.7 (XMSS tree exhaustion handling).
- Sigstore Rekor v2 batch signing design (2025).
- RFC 6962 §3 (Certificate Transparency signed tree head pattern).

---

# PATCH 7 — Float Determinism (Integer / Fixed-Point Clause)

## P7.1 Source spec & section

- RENDER-SPEC.md §4.1 ("Coordinate System")

## P7.2 Problem statement (Gemini G-B2, Reviewer N N-B4)

v1.1 RENDER-SPEC §4.1 specifies SVG coordinates in floating-point with
no mandate for determinism. IEEE 754 floating-point arithmetic is
non-associative under reordering and produces architecture-dependent
results (x86 80-bit extended precision differs from ARM 64-bit, GPU
shaders differ from CPU). Two correct implementations of v1.1 RENDER
running on different machines can produce different SVG bytes, leading
to different blake3 hashes, breaking RENDER §3 hash-equality requirement.

## P7.3 New normative text (replaces v1.1 RENDER §4.1)

**RENDER-SPEC §4.1 v1.2 — Coordinate System (deterministic integer/fixed-point)**

All coordinates, dimensions, and rendering parameters in canonical
RENDER output MUST be expressed as **integer values in micro-units**.
One micro-unit = 1/1000 of a logical SVG user unit. The coordinate
system is:

```
- viewBox is always "0 0 W H" with W, H integers in micro-units.
- Standard card viewBox: "0 0 600000 800000" (representing 600×800
  user units with micro-unit precision).
- All <path> "d" attribute values use integer coordinates in
  micro-units only.
- Decimal literals (".", "0.5", etc.) are FORBIDDEN in canonical
  serialization. Half-pixel positioning is expressed as 500
  micro-units.
- Transforms using matrix(a b c d e f) require a,b,c,d as
  fixed-point Q16.16 (32-bit signed: integer * 65536), serialized
  as integer. Scaling 1.5x = "98304" not "1.5".
- Floating-point evaluation is PROHIBITED at any stage between
  manifest input and SVG byte output. Implementations MUST use
  integer arithmetic exclusively. (Internal int64 sufficient for
  all card geometry.)
- Sin, cos, tan, sqrt operations: not used in canonical card
  geometry. If future variants require trigonometry, use precomputed
  Q16.16 lookup tables specified per-variant.

Floating-point use is permitted in NON-canonical viewer code (browser
SVG renderers will internally convert micro-unit integers to display
floats; this happens after canonical hashing).
```

Font rendering deterministic-ness:
- Font files MUST be cited by blake3 hash, not by name.
- Glyph paths from fonts are pre-rendered to micro-unit integer SVG
  paths and stored in the font_paths_table referenced by hash.
- The card SVG embeds glyph paths by reference, not by font lookup.

Unicode normalization:
- All text content normalized to NFC (Normalization Form Canonical
  Composition) before glyph lookup.
- Bidi reordering applied at canonicalization time per UAX #9.

## P7.4 Rationale

Integer-only deterministic rendering is the standard solution to the
cross-platform floating-point reproducibility problem. Used in:
- IPFS (CID generation uses integer-only arithmetic).
- Bitcoin script (integer-only by design).
- Most blockchain VMs (EVM is 256-bit integer, no floats).

The micro-unit (1/1000) precision is calibrated as follows:
- Human display: 1 micro-unit ≈ 0.001 px at 600px width, well below
  perception threshold.
- Numerical headroom: int64 supports coordinates up to 9.2 × 10^18
  micro-units, vastly exceeding any reasonable card size.
- Q16.16 transforms: standard fixed-point representation, used in
  graphics for decades (Macintosh QuickDraw 1984, OpenType font
  hinting, etc.).

Font hash-citation rather than name-citation eliminates the "Arial on
Linux is metric-incompatible with Arial on Windows" class of bugs.

## P7.5 Literature basis

- IEEE 754-2019 §11 (Reproducibility, "default attribute settings
  insufficient for reproducibility").
- "Determinism in Distributed Systems" — Lamport, *Communications of
  the ACM* (1978).
- W3C SVG 1.1 §A.4 (Conformance — does NOT mandate determinism,
  hence this patch).
- W3C SVG Tiny 1.2 §appendix-B (cross-platform considerations).

---

# PATCH 8 — Receipt Routing Strict Separation

## P8.1 Source spec & section

- POV-SPEC-V11.md §2.4 ("Receipt Storage")
- CUSTODY-REGISTRY-SPEC.md §6 (modified by Patch 3 also)

## P8.2 Problem statement (GPT #4, Claude PC C-C1)

Combined with Patch 3's CUSTODY schema split, we need to be explicit
about exactly **where receipts live** versus **where custody records
live**, and how cross-references work. v1.1 leaves this implicit and
auditors will demand it explicit.

## P8.3 New normative text

**POV-SPEC §2.4 v1.2 — Receipt Storage (authoritative location)**

PoV verification receipts (micros and checkpoints per Patch 6) are
stored exclusively in the **POV Verification Log**, structurally
separated from the Custody Registry, Canon Registry, and Master Index.

```
Storage layout (normative, names canonical):

  /v1/pov/log/<verifier_did>/<year>/<month>/checkpoints.bin
      - Append-only XMSS-checkpointed receipt log
      - Format: see XMSS-OPS-SPEC §4 (CBOR canonical form)
      - Mirror-replicated per NETWORK-PROTOCOL-SPEC §5

  /v1/pov/log/<verifier_did>/<year>/<month>/micros.bin
      - Hash-chained unsigned micros between checkpoints
      - Format: see Patch 6 above

  /v1/pov/index/by-artifact/<artifact_hash>.json
      - JOIN INDEX from artifact_hash → list of (verifier_did,
        checkpoint_id, micro_seq) tuples
      - Populated by Foundation aggregator process
      - Used by clients to find "all verifications of this artifact"

Cross-references from Custody Registry:
  custody_receipt_pointer.receipt_storage_uri (per Patch 3) resolves to
  /v1/pov/log/<verifier_did>/<yyyy>/<mm>/micros.bin#<seq>
  
  CRUCIALLY: custody_receipt_pointer DOES NOT contain receipt body,
  only the URI. Receipt body retrieval is a separate operation.

Cross-references from Canon Registry:
  Canon Registry NEVER contains direct receipt pointers. Canon records
  are about artifact existence and provenance; verification activity
  is an INDEPENDENT system that observes Canon but does not modify it.
```

## P8.4 Rationale

This patch is the implementation-level enforcement of the Layer A vs
Layer B architectural separation established in
A1A5NT-ARCHITECTURE-SYNTHESIS-V1.1 §2. Layer A = Canon = immutable
truth. Layer B = custody + verification = mutable social state. Receipts
are firmly Layer B.

Implementations that previously stored receipts in custody_attestation
rows are non-conformant to v1.2 and must migrate.

## P8.5 Literature basis

- Codd, *The Relational Model for Database Management* (1990), §13
  (table-level separation of concerns).
- IPFS specification, separation of immutable content addressing
  (Layer A analog) from mutable IPNS records (Layer B analog).

---

# PATCH 9 — Multi-Party Pre-Stichting Transition Plan

## P9.1 Source spec & section

- XMSS-OPS-SPEC.md §7 ("Transition Plan") — **NEW SECTION**

## P9.2 Problem statement (Gemini G-A3, GPT #2)

v1.1 spec stack assumes Stichting A1A5 exists or is imminent. Reality:
Stichting is in formation but not yet registered. During the pre-
Stichting period (days/weeks/months), what is the operational identity
of "the Foundation"? v1.1 does not specify the transition mechanics.
Reviewer R1 (single-person bootstrap) is the residual risk
this patch operationalizes.

## P9.3 New normative text

**XMSS-OPS-SPEC §7 v1.2 (NEW) — Pre-Stichting → Stichting Transition Plan**

Operational identity of the Foundation in three phases:

```
PHASE A — Pre-Stichting (current state, ZZP Plastechniek as legal vehicle)
  - Foundation operational identity: pawel.a1a5 / a1a5fndn.a1a5
    (shared XMSS root per session record)
  - Legal vehicle: Plastechniek ZZP (KvK NL, Paweł Piekut)
  - Authority: sole-custodian, no quorum
  - Limits:
    * MUST NOT issue master_index containing PII subject to GDPR
      compliance challenges (those are deferred to Phase B)
    * MUST publish all signing events to public mirrors within 24h
    * Foundation Declaration v1 publicly anchored (existing)
  - Pre-authorized successor: Foundation Declaration v2 (to be
    drafted) names successor with namespace and contact path.
  - Dead-man's switch: if no Foundation signing event for 90
    consecutive days, public attestation may invoke RECOVERY-SPEC
    §3 emergency transfer.

PHASE B — Stichting (target state, target before block 100000)
  - Foundation operational identity: a1a5fndn.a1a5 (Foundation only;
    pawel.a1a5 becomes ordinary namespace holder)
  - Legal vehicle: Stichting A1A5 (Notariusz-registered, NL, BW Book 2)
  - Authority: bestuur 3-of-5 for routine operations,
    Shamir 2-of-3 of root key for emergency operations
  - Bestuur composition: 3 technical + 1 legal + 1 institutional
    (target; GOVERNANCE-SPEC §4 normative)
  - First-pass bestuur (advisory until Stichting registration):
    documented in GOVERNANCE-SPEC §4.3

PHASE C — Federation maturity (post-Stichting, post-7-witness)
  - Foundation operational identity: same as Phase B
  - Authority: most operations co-signed by Witness Federation
    quorum-7 per Patch 4
  - Foundation cannot unilaterally advance Canon Registry without
    witness quorum
  - Emergency operations still possible via Shamir 2-of-3

Transition triggers and sequence:
  A→B: Stichting registration confirmed by Notariusz.
       New keypair generated by bestuur ceremony.
       OLD XMSS root (pawel.a1a5+a1a5fndn.a1a5 shared) emits
       FINAL canonical attestation transferring Foundation role
       to NEW keypair.
       OLD root continues as ordinary namespace holder.
  B→C: 7 witnesses admitted and operating successfully for 90 days.
       Bestuur votes to enter Phase C (GOVERNANCE-SPEC §5).
```

## P9.4 Rationale

This patch operationalizes Reviewer M's "R1 - Single person bootstrap"
residual risk and Reviewer O's "Emergency Root Transfer" requirement.
It honestly documents the current pre-Stichting state without
pretending Foundation already has institutional structure it lacks.

The dead-man's switch (90 days) is calibrated against:
- Real-world institutional crisis windows (multi-month, not days).
- Severe enough that activation is visible to all observers.
- Short enough to keep liveness if founder becomes incapacitated.

Pre-authorized successor (Foundation Declaration v2, to be drafted)
gives the project survival pathway distinct from "Paweł always lives."

## P9.5 Literature basis

- Dutch BW Book 2 Title 6 (Stichting framework).
- Estate planning best practice — successor designation in
  closely-held entities (US treatise: Sitkoff & Dukeminier, *Wills,
  Trusts, and Estates* 11th ed.).
- Bitcoin Foundation historical example (founder transition 2014-2015,
  case study in institutional bootstrapping).

---

# PATCH 10 — "Federated Canonical Registry" Honest Naming

## P10.1 Source spec & section

- A1A5NT-ARCHITECTURE-SYNTHESIS.md §1 ("Project Framing")
- CANON-REGISTRY-SPEC.md §1 ("Overview")

## P10.2 Problem statement (Claude PC C-C3, Reviewer N implicit)

v1.1 framings include phrases like "post-quantum L1" and "decentralized
verification network" which **overstate present-state decentralization**.
Reality at v1.2 issuance: Foundation operates the only Canon-issuing
node; mirror network is replication, not consensus; witness federation
is N=0. The honest framing is:

> a1a5NT is a **federated canonical registry** transitioning toward
> **witness federation** governance, currently operating in
> single-attestor mode with cryptographic anchoring of all attestations
> to enable future decentralization without breaking prior
> commitments.

This framing is stronger because (a) it is accurate, (b) it does not
require backtracking when audited, and (c) it positions the project as
preservation infrastructure rather than as a crypto-token L1, which is
the NLnet-friendly framing per Reviewer N.

## P10.3 New normative text

**SYNTHESIS §1 v1.2 — Project Framing**

```
What a1a5NT is, at v1.2:

  a1a5NT is a federated canonical registry for sovereign post-quantum
  archival of cryptographic artifacts, currently operated by Stichting
  A1A5 (NL, registration in progress) via single-attestor authority,
  with all attestations cryptographically anchored using XMSS post-
  quantum signatures and prepared for transition to a witness
  federation governance model upon federation maturity.

What a1a5NT is NOT, at v1.2:

  - It is not yet a Layer-1 blockchain in the consensus sense. Canon
    advancement is single-attestor.
  - It is not yet decentralized. The Foundation is currently a single
    legal entity in formation.
  - It is not a token primarily. POP-3-TOKEN (Bincoin) is gated by
    Genesis and is downstream of preservation infrastructure.
  - It is not a substitute for legal record-keeping. It provides
    cryptographic continuity, not legal attestation.

What a1a5NT certifies:

  - Provenance continuity (a given artifact existed at a given time,
    held by a given custodian)
  - Custody chain (verifiable history of who held the artifact)
  - Integrity continuity (artifact content has not been modified since
    canonical record)

What a1a5NT does NOT certify:

  - Truthfulness of original artifact content (Patch 11; G7 in
    SECURITY-MODEL)
  - Real-world legal ownership (custody = social attestation, not
    title)
  - Authenticity of identity claims beyond cryptographic key control
```

**CANON-REGISTRY-SPEC §1 v1.2 — Overview (corresponding update)**

Strike "post-quantum L1 consensus layer" wherever appearing. Replace
with "post-quantum federated canonical registry." All other technical
substance unchanged.

## P10.4 Rationale

Honest naming is operationally important because:
1. NLnet auditors evaluate accuracy of self-description as part of
   technical review. Overclaiming triggers skepticism on all other
   claims.
2. Notariusz will not certify Stichting documents that describe
   capabilities not yet present.
3. Reviewer N: "v1.1 zrobiło jedną bardzo ważną rzecz: przestało
   udawać, że wszystko jest już rozwiązane." This patch completes
   that work.

The phrase "federated canonical registry transitioning to witness
federation" is precise: it describes (a) current architecture, (b)
direction of evolution, and (c) the mechanism (federation) clearly.
This is in line with Sigstore's self-description ("federated transparency
log") and Certificate Transparency's ("federated audit log").

## P10.5 Literature basis

- Sigstore project self-description (sigstore.dev "About").
- Certificate Transparency System Architecture (RFC 9162 §1).
- Mozilla CT policy framing (mozilla.org/security/ct/).

---

# PATCH 11 — "Verifier = Producer" Precision Rewrite

## P11.1 Source spec & section

- A1A5NT-ARCHITECTURE-SYNTHESIS.md §4 ("Verification Mechanics")
- POV-SPEC-V11.md §2 ("PoV Roles")

## P11.2 Problem statement (Gemini G-A4 explicit confirmation, GPT #5)

v1.1 SYNTHESIS §4 contains language conflating "verifier" with
"producer" in some places. Gemini explicitly noted: "verifier=producer
correct, but needs precision rewrite." GPT independently flagged that
in the PoV economics, "verifier" sometimes means "node doing
verification work" and sometimes means "entity producing verification
receipts." These are the same role at v1.1 (the entity doing the work
also signs the receipt), but the text reads as if there are sometimes
two parties. This causes misreading by auditors not deeply familiar
with the spec.

## P11.3 New normative text

**SYNTHESIS §4 v1.2 — Verification Mechanics (precision rewrite)**

In a1a5NT verification, exactly one party performs each verification:
the **verifier-producer**. This single role:

1. Receives a canonical artifact reference from the master_index_v_NNN
   signed manifest.
2. Downloads the artifact from one or more mirrors, accumulating
   signed Content-Length attestations from each.
3. Performs the verification computation specified in POV-SPEC §5
   (hash verification, structural validation, render reproducibility
   check if a card-type artifact, etc.).
4. Generates a PoV receipt micro per Patch 6, signing the eventual
   checkpoint with their own XMSS key.
5. Submits the receipt micro to the POV Verification Log per Patch 8.
6. Receives MBW credit per Patch 2/5 (less bandwidth bleed share to
   mirrors).

This role is named **verifier-producer** in v1.2 normative text. The
verifier IS the producer of the receipt. No separate "verifier" and
"producer" roles exist.

Adjacent but distinct roles (NOT verifier-producer):

- **Mirror operator**: serves artifact bytes, earns bandwidth share
  per Patch 5. Does NOT perform verification.
- **Custodian**: claims social-trust attestation over artifact (Custody
  Registry per CUSTODY-REGISTRY-SPEC). Does NOT perform verification
  (may incidentally verify but custody attestation is independent of
  PoV).
- **Witness**: co-signs Foundation's master_index per Patch 4. Does
  NOT perform per-artifact verification; witnesses verify INDEX state,
  not artifact state.
- **Foundation / TA**: issues canonical master_index, runs timestamp
  authority. Does NOT verify individual artifacts as part of routine
  operations.

**POV-SPEC §2 v1.2 — PoV Roles (corresponding update)**

Strike all instances of "verifier" used alone where the meaning is
"verifier-producer." Replace with "verifier-producer" or "v-p" abbrev
after first use per section.

## P11.4 Rationale

This is a documentation precision patch, not an architectural change.
The v1.1 architecture is correct; only the prose is imprecise. Cross-
review consensus (Gemini explicitly, GPT independently) identified the
issue. The fix is mechanical: use a compound term that cannot be
misread.

The four adjacent roles are listed explicitly so auditors have a clear
who-does-what map. This is standard practice in role-based access
control documentation and protocol specification.

## P11.5 Literature basis

- IETF protocol specification convention — explicit role definition
  (e.g., RFC 6749 §1.1 OAuth 2.0 Roles).
- W3C WebAuthn §1.2 (precise definition of authenticator, relying
  party, user).

---

# APPENDIX A — Patch Application Order

Patches are independent except where noted:

- Patch 3 (CUSTODY schema split) MUST be applied before Patch 8
  (Receipt routing strict separation) since Patch 8 references the
  custody_receipt_pointer table defined by Patch 3.
- Patch 1 (Nonce/TA) MUST be applied before Patch 6 (Verifier tree
  exhaustion) since Patch 6 references TA signature pattern.
- Patch 4 (Witness federation) modifies SECURITY-MODEL A4 adversary;
  apply after SECURITY-MODEL v1.1 is in place (it is).
- Patches 2, 5, 7, 9, 10, 11 are mutually independent.

Recommended application order: 1 → 3 → 8 → 6 → 4 → 2 → 5 → 7 → 9 → 10 → 11.

---

# APPENDIX B — Conformance Statement

An implementation is **v1.2 conformant** if and only if:

1. v1.1 source specs are implemented correctly per their normative
   text, EXCEPT where modified by patches in this document.
2. All 11 patches in this document are implemented faithfully.
3. The SECURITY-MODEL.md v1.1 (already includes A13, G7, Emergency
   Root Transfer) is implemented.
4. CRYPTOGRAPHIC-ASSUMPTIONS.md v1.0 (separate document) is honored
   as the formal trust basis.
5. LEGAL-INTERPRETATION-SPEC.md v1.0 governs legal mapping.
6. RECOVERY-SPEC.md v1.0 governs all recovery scenarios.
7. CANON-RETIREMENT-SPEC.md v1.0 governs tombstone procedures.
8. GOVERNANCE-SPEC.md v1.0 governs all role authority.
9. NETWORK-PROTOCOL-SPEC.md v1.0 governs all wire protocol.
10. STATE-MACHINE-SPEC.md v1.0 governs all state transitions.

An implementation claiming v1.2 conformance MUST publish a conformance
statement listing the patches implemented and any known divergences
from this document, signed by the implementer's XMSS key and published
in a publicly-accessible location.

---

# APPENDIX C — Reviewer Attribution Crosswalk

For audit traceability — which v1.2 reviewer findings produced which
patches:

| Reviewer (model) | Finding code | Severity | Patch # |
|------------------|------------|----------|---------|
| Gemini | G-A1 (MBW dedup) | HIGH | 2 |
| Gemini | G-A2 (CUSTODY schema) | HIGH | 3 |
| Gemini | G-A3 (transition plan) | HIGH | 9 |
| Gemini | G-A4 (verifier=producer confirmation) | MEDIUM | 11 |
| Gemini | G-B2 (SVG determinism) | HIGH | 7 |
| Grok | #2 (nonce/timestamp) | CRITICAL | 1 |
| Grok | #3 (bandwidth bleed) | HIGH | 5 |
| Grok | #5 (verifier tree exhaustion) | CRITICAL | 6 |
| GPT | #2 (transition plan) | HIGH | 9 |
| GPT | #3 (CUSTODY schema) | HIGH | 3 |
| GPT | #4 (receipt routing) | HIGH | 8 |
| GPT | #5 (verifier=producer) | MEDIUM | 11 |
| Claude PC | E1 (witness federation) | CRITICAL | 4 |
| Claude PC | E2 (nonce/timestamp) | CRITICAL | 1 |
| Claude PC | E3 (verifier tree exhaustion) | CRITICAL | 6 |
| Claude PC | C-C1 (receipt routing) | HIGH | 8 |
| Claude PC | C-C3 (honest naming) | MEDIUM | 10 |
| Claude PC | C-C4 (MBW dedup) | HIGH | 2 |
| Reviewer M (post-v1.0 SECURITY) | A8 (bandwidth bleed) | HIGH | 5 |
| Reviewer N (post-v1.0 SECURITY) | N-B2 (witness federation) | CRITICAL | 4 |
| Reviewer N (post-v1.0 SECURITY) | N-B4 (SVG determinism) | HIGH | 7 |

**Cross-confirmation count:**
- Patch 1: 2 reviewers (Grok #2 + Claude PC E2)
- Patch 3: 2 reviewers (Gemini G-A2 + GPT #3)
- Patch 4: 2 reviewers (Claude PC E1 + Reviewer N N-B2)
- Patch 6: 2 reviewers (Grok #5 + Claude PC E3)
- Patch 7: 2 reviewers (Gemini G-B2 + Reviewer N N-B4)
- Patch 8: 2 reviewers (GPT #4 + Claude PC C-C1)
- Patch 9: 2 reviewers (Gemini G-A3 + GPT #2)
- Patch 11: 2 reviewers (Gemini G-A4 + GPT #5)
- Patches 2, 5, 10: 1 reviewer each (still valid findings)

Patches with 2+ reviewer attribution are **cross-confirmed** and have
higher confidence. Patches with 1 reviewer are still authoritative
under the v1.2 architect-delegated decision authority granted for this
spec generation.

---

# END OF TIER-2-PATCHES-V1.2

```
Document hash (this document, after final write): 
  computed at deployment time, recorded in master_index_v_001
Authoring authority:                pawel.a1a5 / a1a5fndn.a1a5 (shared)
Architect-delegated decisions:      Patches 1-11 per autonomous authority
                                     granted by Paweł Piekut for v1.2
                                     production (see session record).
License:                            A1A5 Sovereign License v1.0
Public publication:                 GitHub a1a5NT/a1a3 (committed at
                                     deployment time)
```
