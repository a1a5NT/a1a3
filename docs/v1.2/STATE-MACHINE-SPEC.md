# STATE-MACHINE-SPEC.md

## a1a5NT Formal State Machine and Transition Invariants

```
Document version:   v1.0
Issued:             2026-05-17
Status:             Tier 1 critical specification (v1.2 deliverable)
Authority basis:    Formal methods practice (CSP, TLA+ patterns),
                    distributed systems theory (Lamport timestamps,
                    causal ordering), append-only log theory (RFC 6962),
                    state machine replication (Schneider 1990)
Foundation:         a1a5fndn.a1a5
Addresses:          Claude PC CP-B2 (CRITICAL): "Brak formalnego STATE
                    TRANSITION MODEL — allowed transitions, invalid
                    transitions, invariants. Bez tego custody/canon
                    operations są niewerifikowalne"
                    + cross-confirmed by need for formal academic audit
                    grounding
Audit relevance:    REQUIRED for academic post-quantum review;
                    REQUIRED for cryptographic correctness verification
License:            A1A5 Sovereign License v1.0
```

---

## EXECUTIVE SUMMARY (1 page)

**What this document is:**

The formal state machine specification for a1a5NT, defining:
- All system states each major entity can be in
- All permitted state transitions
- The preconditions and postconditions of each transition
- Invariants that MUST hold at all times
- Detection procedures for invariant violation

**Why this matters:**

Without formal state semantics, a system's correctness cannot
be verified by external auditors. Operators don't know what they
may or may not do. Implementations may diverge silently. Disputes
become subjective.

This spec makes a1a5NT's behavior mathematically tractable. An
auditor can read it and determine: given any system state X and
proposed action Y, is Y permitted? What state Z must result?

**What entities have state:**

1. **Canon entries** — once issued, immutable; states: ACTIVE,
   TOMBSTONED, SUPERSEDED
2. **Custody records** — mutable; states per CUSTODY-REGISTRY-SPEC
   §3
3. **Devices** — per PoV; states: REGISTERED, ACTIVE, INACTIVE,
   BANNED
4. **Witnesses** — per GOVERNANCE-SPEC §5; states: ONBOARDING,
   PROBATION, ACTIVE, WITHDRAWN, REMOVED
5. **Master index versions** — states: PENDING, PUBLISHED,
   CO_SIGNED, FINALIZED
6. **XMSS keypairs** — per XMSS-OPS-SPEC §9; states: ACTIVE,
   DEPRECATED, REVOKED, COMPROMISED

**Six core invariants that hold throughout system lifetime:**

```
I-1: Canon Append-Only — no canon entry once issued is ever
     modified or deleted; tombstoning adds new entries
I-2: Master Index Monotonic — version numbers strictly increase
I-3: Custody Chain Connected — every custody record references
     prior custody record or initial issuance
I-4: Device Identity Unique — each device_id appears in exactly
     one device record at any time
I-5: Witness Independence — no two active witnesses share keypair
     or operational control
I-6: XMSS State Monotonic — every XMSS keypair's idx_next never
     decreases (per XMSS-OPS-SPEC Invariant I1)
```

**Failure mode if violated:**

Any invariant violation triggers EMERGENCY procedures per
SECURITY-MODEL §A6 and XMSS-OPS-SPEC §9.2.

---

## 0. Abstract

This document specifies the formal state machine of a1a5NT,
enabling mathematical verification of system behavior. It
enumerates entities, their permitted states, transitions, and
invariants.

The spec uses informal but rigorous notation suitable for direct
translation to formal specification languages (TLA+, Coq, CSP)
in future formal verification work.

---

## 1. Scope

### 1.1 In scope

- State enumeration for all major entity types
- Transition diagrams (textual representation)
- Preconditions and postconditions for each transition
- System-level invariants
- Invariant verification procedures
- Equivalence classes (e.g., "any device in INACTIVE state is
  equivalent for all protocol purposes")

### 1.2 Out of scope

- Implementation-specific data structures (covered by individual
  registry specs)
- Performance characteristics
- Mechanical formal verification (future work, not in v1.2 scope)
- Liveness properties (when transitions WILL happen, only what is
  PERMITTED)

### 1.3 Notation

This spec uses pseudo-formal notation:

```
state(entity)                Current state of entity
T(action)                    A transition labeled "action"
precondition(T)              What must hold before T is permitted
postcondition(T)             What MUST hold after T executes
invariant(I)                 What MUST hold at all times
forbidden                    Transition NEVER permitted
guarded                      Transition permitted under conditions
permitted                    Transition always permitted in named state
```

---

## 2. Normative references

| Reference | Title | Applicability |
|-----------|-------|---------------|
| RFC 2119 | Key words for use in RFCs | Normative wording |
| RFC 6962 | Certificate Transparency append-only log | Canon append-only invariant |
| CSP | Communicating Sequential Processes (Hoare) | Process algebra |
| TLA+ | Temporal Logic of Actions (Lamport) | Temporal invariants |
| XMSS-OPS-SPEC.md §5 | Invariants I1-I5 | Cryptographic state invariants |

---

## 3. Entity: Canon Entry

### 3.1 States

```
ENUM CanonEntryState {
  ACTIVE       — entry is current, content available, signature valid
  TOMBSTONED   — entry exists but content unavailable per legal
                 compulsion (per SECURITY-MODEL §A13)
  SUPERSEDED   — entry has been logically replaced by newer entry
                 of same class (e.g., master_index v_N+1 supersedes
                 v_N); supersede does NOT modify the entry, only
                 references it
}
```

### 3.2 Initial state

A canon entry's initial state upon issuance is **ACTIVE**.

### 3.3 Permitted transitions

```
T(tombstone): ACTIVE → TOMBSTONED
  precondition:
    - Valid legal compulsion documented
    - Foundation operator authorization
    - TOMBSTONE_NOTICE entry issued (new canon entry, not modification)
    - Original entry's blake3 hash and signature unchanged
  postcondition:
    - state == TOMBSTONED
    - Original entry's content NOT served by Foundation-operated mirrors
    - Original entry's blake3 hash STILL queryable
    - Original entry's signature STILL verifiable
    - HTTP 451 returned for content requests with TOMBSTONE_NOTICE body
  guarded: requires legal documentation reference

T(supersede): ACTIVE → SUPERSEDED
  precondition:
    - New entry of same class issued and reaches ACTIVE state
    - Foundation operator decision (informal supersede) OR
    - Bestuur decision (post-Stichting, formal supersede)
  postcondition:
    - state == SUPERSEDED
    - Original entry still readable and verifiable
    - New entry takes precedence in client logic
    - Cross-reference recorded
  permitted: per class-specific supersede rules

T(content_archival_loss): ACTIVE → SUPERSEDED [BY EFFECT]
  precondition:
    - Some external event causes Foundation to lose access to 
      content (catastrophic hardware loss, etc.)
    - Foundation publishes ARCHIVE_LOSS_NOTICE
  postcondition:
    - State formally changes to SUPERSEDED with reason
    - Witness mirrors continue to serve content
    - Investigation initiated per RECOVERY-SPEC

T(reverse_supersede): SUPERSEDED → ACTIVE [FORBIDDEN under most conditions]
  forbidden: except under bestuur quorum decision with public 
             justification published as canon entry, AND newer 
             SUPERSEDING entry must itself be SUPERSEDED first
  
T(reverse_tombstone): TOMBSTONED → ACTIVE [FORBIDDEN under most conditions]
  forbidden: except by court order or successful appeal of legal 
             compulsion, with COURT_REVERSAL canon entry as
             prerequisite
```

### 3.4 Invariants for canon entries

```
INV-CE-1: Hash immutability
  ∀ canon_entry E: blake3(E.content_at_issuance) is fixed forever
  even when E is TOMBSTONED or SUPERSEDED.
  
INV-CE-2: Signature immutability
  ∀ canon_entry E: E.foundation_signature is fixed forever and 
  verifiable forever against the issuing keypair's public root.
  
INV-CE-3: State transition causality
  ∀ state transition T(s1 → s2): there exists a triggering canon 
  entry (TOMBSTONE_NOTICE, SUPERSEDE_NOTICE, COURT_REVERSAL) that
  records the cause.
  
INV-CE-4: Append-only
  No canon_entry is ever deleted from the registry. Tombstoning 
  hides content but preserves the entry record.
```

---

## 4. Entity: Custody Record

### 4.1 States

Per CUSTODY-REGISTRY-SPEC §3 (canonical reference; summary here):

```
ENUM CustodyState {
  foundation_held              Custody held by Foundation pending sale
  foundation_held_pre_genesis  Initial state for pre-Genesis artifacts
  custodian_held_pre_sale      Held by appointed custodian
  in_active_sale               Listed for transfer to buyer
  buyer_held                   Held by buyer post-purchase
  transferred                  Transferred to subsequent custodian
  revoked                      Revocation per dispute resolution
  burned                       Voluntarily destroyed by holder
  lost                         Custody record exists but holder unreachable
                               for >365 days
}
```

### 4.2 State transition matrix

```
TRANSITIONS:

T(initial_assignment): NULL → foundation_held_pre_genesis
  precondition: Canon entry exists for the artifact; Foundation 
               operator decision
  postcondition: Custody record created; Foundation holds

T(appoint_custodian): foundation_held → custodian_held_pre_sale
  precondition: Custodian onboarded per GOVERNANCE-SPEC
  postcondition: Custody transferred to named custodian

T(list_for_sale): custodian_held_pre_sale → in_active_sale
  precondition: Custodian decision; price set (per POP-3-TOKEN-SPEC)
  postcondition: Artifact listed in marketplace

T(buyer_purchase): in_active_sale → buyer_held
  precondition: Bincoin payment verified; transfer authorized
  postcondition: Custody to buyer

T(transfer): buyer_held → buyer_held [with new buyer]
  precondition: Current holder consent; recipient consent; price/gift
               documented
  postcondition: Custody record links old and new holders

T(revoke): * → revoked
  precondition: Dispute resolution outcome per GOVERNANCE-SPEC §7.2
  postcondition: Custody revoked; future challenges resolved

T(burn): buyer_held → burned
  precondition: Current holder decision
  postcondition: Artifact effectively retired; canon entry remains

T(lost): * → lost [WITH TIMEOUT]
  precondition: 365 days without holder attestation
  postcondition: Listed as orphaned; bestuur may reclaim
                 (post-Stichting)

T(recover_from_lost): lost → previous_state [GUARDED]
  precondition: Holder appears with cryptographic proof of identity
  postcondition: Custody restored
  guarded: bestuur ratification (post-Stichting)
```

### 4.3 Forbidden transitions

```
T(transferred → foundation_held)
  forbidden: Foundation cannot reclaim custody from rightful buyer
  exception: Court order

T(buyer_held → in_active_sale [by Foundation])
  forbidden: Foundation cannot list buyer's artifact for sale

T(revoked → buyer_held [reverse])
  forbidden: Revocation is final
  exception: Court reversal
```

### 4.4 Invariants for custody records

```
INV-CR-1: Chain connectivity
  ∀ custody_record R: R.previous_custody_ref points to either NULL
  (initial) or to a valid previous custody record for same artifact.
  
INV-CR-2: Holder consent
  ∀ transition T involving holder change: T requires holder's 
  cryptographic consent (signature).
  
INV-CR-3: Canon reference integrity
  ∀ custody_record R: R.canon_ref points to a valid canon entry
  (ACTIVE or TOMBSTONED — not SUPERSEDED for custody purposes).
  
INV-CR-4: No double-spending
  ∀ artifact A: at any moment, exactly one custody_record for A 
  is in state {foundation_held, foundation_held_pre_genesis, 
  custodian_held_pre_sale, in_active_sale, buyer_held}; all other
  records for A are in terminal states {revoked, burned, lost} or
  superseded by next-in-chain.
```

---

## 5. Entity: Device

### 5.1 States

```
ENUM DeviceState {
  REGISTERED   Device registered, not yet verified active
  ACTIVE       Device has performed verifications, accumulating 
              MBW per maturity curve
  INACTIVE     No verification activity for 30+ days
  BANNED       Device banned for misbehavior (Sybil detection,
              etc.)
}
```

### 5.2 Permitted transitions

```
T(register): NULL → REGISTERED
  precondition: Device generates XMSS keypair; submits 
               REGISTRATION request per POV-SPEC
  postcondition: device_id assigned; record created
  invariant maintained: INV-D-1 (uniqueness)

T(activate): REGISTERED → ACTIVE
  precondition: First successful verification receipt submitted
  postcondition: MBW accrual begins per maturity curve

T(deactivate): ACTIVE → INACTIVE
  precondition: 30 days since last verification receipt
  postcondition: MBW accrual ceases; maturity progress preserved

T(reactivate): INACTIVE → ACTIVE
  precondition: New verification receipt submitted
  postcondition: MBW accrual resumes from previous maturity level

T(ban): * → BANNED
  precondition: Foundation operator decision based on Sybil 
               detection, fraud, or other misbehavior; documented
  postcondition: All future submissions from device_id refused;
                historical MBW credit voided per policy
  guarded: bestuur ratification within 7 days (post-Stichting)

T(unban): BANNED → INACTIVE [RARE]
  precondition: Successful appeal; bestuur reversal
  guarded: requires explicit canon entry
```

### 5.3 Invariants

```
INV-D-1: Identity uniqueness
  ∀ device_id ID: at most one device record exists with ID.
  
INV-D-2: Maturity monotonicity (within an activity window)
  ∀ device D: maturity_age(D) increases linearly while D is in
  state ACTIVE; freezes (not resets) when D enters INACTIVE; 
  resumes when D returns to ACTIVE. Reset to zero only on BAN.
  
INV-D-3: MBW accrual constrained
  ∀ device D: MBW credit per verification is bounded by formula
  in MBW-SPEC §3 and CANNOT exceed published maximum per device 
  per day.
```

---

## 6. Entity: Witness

### 6.1 States

Per GOVERNANCE-SPEC §5 (canonical; summary):

```
ENUM WitnessState {
  ONBOARDING   Foundation reviewing application
  PROBATION    Accepted, 90-day probation, signatures advisory
  ACTIVE       Full federation member, signatures counted
  WITHDRAWN    Voluntary withdrawal
  REMOVED      Removed per §5.5 grounds
}
```

### 6.2 State transition matrix

```
T(apply): NULL → ONBOARDING
  precondition: Operator submits ONBOARDING_REQUEST per 
               GOVERNANCE-SPEC §5.3
  postcondition: Foundation begins review

T(reject): ONBOARDING → NULL
  precondition: Foundation determines applicant fails Q1-Q6 criteria
  postcondition: Application closed; may re-apply after addressing

T(accept_to_probation): ONBOARDING → PROBATION
  precondition: Foundation/bestuur approval per §5.3 Phase 2;
               cryptographic registration per Phase 3 complete
  postcondition: Witness namespace registered; signatures begin
                being collected but not counted toward quorum

T(graduate): PROBATION → ACTIVE
  precondition: 90 days probation completed; SLA met (≥95% 
               availability); no equivocation incidents
  postcondition: Signatures now count toward federation quorum

T(probation_failure): PROBATION → REMOVED
  precondition: SLA failure during probation OR equivocation
               detected during probation
  postcondition: Removal per §5.5 procedure

T(voluntary_withdrawal): * → WITHDRAWN
  precondition: Operator publishes WITHDRAWAL_NOTICE signed by 
               witness keypair
  postcondition: Witness ceases participation; historical 
                signatures remain valid

T(removal): * → REMOVED
  precondition: Grounds per GOVERNANCE-SPEC §5.5 G1-G9
  postcondition: Federation membership terminated; historical 
                signatures remain valid

T(reinstatement): REMOVED → ONBOARDING [VERY RARE]
  precondition: Removal grounds addressed; appeal successful; 
               bestuur quorum decision
  postcondition: Must restart full onboarding
```

### 6.3 Invariants

```
INV-W-1: Independence
  ∀ pair of ACTIVE witnesses (W1, W2): W1.operator ≠ W2.operator
  AND W1.xmss_root ≠ W2.xmss_root.
  
INV-W-2: Probation prerequisites
  ∀ witness W in state PROBATION: W joined timestamp + 90 days 
  ≥ current time when W can advance to ACTIVE.
  
INV-W-3: Withdrawal finality
  ∀ witness W in state WITHDRAWN: W cannot transition to ACTIVE
  without restarting via ONBOARDING.
```

---

## 7. Entity: Master Index Version

### 7.1 States

```
ENUM MasterIndexState {
  PENDING      Newly issued, awaiting witness co-signatures
  PUBLISHED    Foundation signature complete, publicly available
  CO_SIGNED    Witness federation quorum has co-signed
  FINALIZED    Sufficient time has passed and no equivocation
              detected; locked as historical reference
}
```

### 7.2 Permitted transitions

```
T(issue): NULL → PENDING
  precondition: New canon entries available; publication interval
               reached
  postcondition: New master_index version computed; Foundation
                signature applied; entry created

T(publish): PENDING → PUBLISHED
  precondition: Foundation signature verified; entry committed to
               local registry
  postcondition: Available via /v1/foundation/master_index; gossiped
                to witnesses

T(co_sign): PUBLISHED → CO_SIGNED
  precondition: Sufficient witness signatures collected per quorum
               (post-federation-activation)
  postcondition: CO_SIGNED_NOTICE canon entry issued

T(finalize): CO_SIGNED → FINALIZED [WITH TIMEOUT]
  precondition: 7+ days have passed since CO_SIGNED; no equivocation
               evidence received
  postcondition: Master index version locked as historical record

T(equivocation_invalidation): * → [WITHDRAWN, separate state]
  precondition: Equivocation evidence published per
               NETWORK-PROTOCOL-SPEC §7
  postcondition: This version is no longer canonical; new version
                must be published correcting the equivocation
```

### 7.3 Invariants

```
INV-MI-1: Version monotonicity
  ∀ master_index version V_n: V_n exists only if V_(n-1) exists
  and V_(n-1) is in state PUBLISHED, CO_SIGNED, or FINALIZED.
  
INV-MI-2: Single canonical version per number
  ∀ version number n: at most one master_index entry exists in 
  state PUBLISHED+ for version n.
  
INV-MI-3: Foundation signature requirement
  ∀ master_index entry in state PUBLISHED+: contains Foundation
  signature verifiable against current Foundation keypair.
  
INV-MI-4: Quorum requirement for CO_SIGNED
  Master index reaches CO_SIGNED only when witness signature count
  ≥ quorum_threshold (per GOVERNANCE-SPEC §6.1).
```

---

## 8. Entity: XMSS Keypair

### 8.1 States

Per XMSS-OPS-SPEC §9 (canonical; summary):

```
ENUM XMSSKeypairState {
  ACTIVE         Keypair in use; signing permitted
  DEPRECATED     Keypair retired by routine rotation; signatures 
                from this keypair remain valid for historical 
                verification
  REVOKED        Keypair revoked by formal rotation; signatures 
                remain valid for historical verification but no 
                new signatures expected
  COMPROMISED    Keypair compromised; emergency revocation; 
                signatures dated AFTER compromise time are 
                INVALID for new attestations
}
```

### 8.2 Permitted transitions

```
T(generate): NULL → ACTIVE
  precondition: XMSS-OPS-SPEC §4 procedure; state file written
  postcondition: Keypair available for signing per XMSS-OPS-SPEC §5

T(retire_routine): ACTIVE → DEPRECATED
  precondition: Capacity threshold reached (75%); ROTATION_BEGIN
               issued; NEW_KEY generated and TRANSITION_NOTICE 
               published
  postcondition: New signing operations use NEW_KEY; OLD_KEY 
                preserved for verification

T(emergency_revoke): ACTIVE → COMPROMISED
  precondition: Compromise detected per XMSS-OPS-SPEC §9.2
  postcondition: All future signing operations halted; 
                EMERGENCY_REVOCATION_NOTICE published; signatures
                after compromise time invalidated per §9.2

T(formal_revoke): DEPRECATED → REVOKED
  precondition: Period of preserved deprecation (e.g., 30 days)
               complete
  postcondition: Formal revocation; state file preserved for
                forensic audit per XMSS-OPS-SPEC §9.1

T(rehabilitate): COMPROMISED → DEPRECATED [POSSIBLE IF FALSE ALARM]
  precondition: Investigation determines compromise did not actually
               occur; bestuur ratification
  postcondition: Keypair status corrected
```

### 8.3 Invariants

```
INV-XK-1: Monotonic state per XMSS-OPS-SPEC I1
  ∀ XMSS keypair K: idx_next(K) never decreases.
  
INV-XK-2: Single-state per keypair
  ∀ XMSS keypair K: K is in exactly one of {ACTIVE, DEPRECATED, 
  REVOKED, COMPROMISED} at any time.
  
INV-XK-3: Active uniqueness for role
  ∀ role R (Foundation, witness, custodian): at most one XMSS 
  keypair in state ACTIVE per role per signer identity at any 
  moment.
```

---

## 9. System-wide invariants

The following invariants MUST hold across the entire system at
all times:

### 9.1 Canon append-only invariant

```
INV-S-1: Canon append-only
  ∀ canon_entry E that was at some prior time in state ACTIVE:
  E.id, E.content_hash, E.foundation_signature, E.issuance_timestamp
  are NEVER modified.
  
  E.state MAY transition to TOMBSTONED or SUPERSEDED per §3.3,
  but this is a NEW canon entry referencing E, not modification 
  of E.
```

### 9.2 Master index monotonicity

```
INV-S-2: Master index version monotonic
  ∀ master_index version V_n that has reached state PUBLISHED:
  there is no future master_index entry with version number ≤ n
  in state PUBLISHED unless n is being replaced via equivocation
  invalidation (in which case the replacement has the SAME version
  number but resolves the equivocation).
```

### 9.3 Custody chain connectivity

```
INV-S-3: Every custody record connects to its lineage
  ∀ custody_record R: traversing R.previous_custody_ref backward
  reaches a custody record with previous_custody_ref == NULL 
  (initial) in finite steps. No cycles. No orphans.
```

### 9.4 Device identity uniqueness

```
INV-S-4: Device identity unique
  ∀ device_id ID: ∃! device record D with D.device_id == ID.
```

### 9.5 Witness independence

```
INV-S-5: Active witnesses operationally independent
  ∀ pair (W1, W2) of witnesses in state ACTIVE:
  - W1.operator_identity ≠ W2.operator_identity
  - W1.xmss_root ≠ W2.xmss_root
  - W1.host ≠ W2.host (different infrastructure)
```

### 9.6 XMSS state monotonicity

```
INV-S-6: XMSS state monotonic (per XMSS-OPS-SPEC §5 I1)
  ∀ XMSS keypair K: ∀ snapshots S1, S2 of K's state where S1 was
  taken at time T1 and S2 at time T2, T1 < T2:
  S2.idx_next ≥ S1.idx_next AND S2.monotonic_counter ≥ S1.monotonic_counter
```

---

## 10. Invariant verification

### 10.1 Continuous verification

Foundation operators SHOULD run continuous invariant checks:

```
INVARIANT_CHECK_FREQUENCY:

  INV-S-1 (Canon append-only):     On every canon entry issuance
  INV-S-2 (Master index monotonic): On every master index publication
  INV-S-3 (Custody chain):          On every custody operation
  INV-S-4 (Device unique):          On every device registration
  INV-S-5 (Witness independent):    On witness onboarding/changes
  INV-S-6 (XMSS state monotonic):   On every signing operation
                                    (per XMSS-OPS-SPEC §5)
  
  System-wide consistency check: weekly batch verification
```

### 10.2 Violation response

If any invariant violation detected:

```
INVARIANT_VIOLATION_RESPONSE:

  1. IMMEDIATELY halt any operation that would compound the 
     violation
  2. Snapshot system state for forensic preservation
  3. Log INVARIANT_VIOLATION event to GOVERNANCE_AUDIT_LOG
  4. Convene Foundation operator and (post-Stichting) bestuur
  5. Determine: was this an attack, a software bug, or operator 
     error?
  6. Execute response per SECURITY-MODEL adversary class identified:
     - INV-S-1 violation: Likely A6 (Foundation insider) or 
       software bug
     - INV-S-6 violation: A9 (XMSS state replay) — invoke 
       EMERGENCY_ROTATION per XMSS-OPS-SPEC §9.2
     - INV-W-1/S-5 violation: Likely witness collusion or 
       independence failure
  7. Publish INVARIANT_VIOLATION_NOTICE canon entry with findings
```

---

## 11. State machine in failure modes

### 11.1 Partial network failure

During network partition:

- Foundation may continue issuing canon entries (Foundation has 
  authority; partition affects propagation, not authorization)
- Witnesses may not be able to gossip; they continue accepting
  local Foundation STH publications
- Equivocation detection is degraded until partition heals
- Verification receipts may queue locally if device cannot reach
  Foundation; submitted on reconnection

State machine remains correct under partition; only liveness
is affected.

### 11.2 Mass witness failure

If majority of witnesses fail (e.g., natural disaster affecting
multiple operators simultaneously):

- Foundation continues operating but cannot achieve quorum for 
  CO_SIGNED state on master indexes
- Master indexes remain in PUBLISHED state without CO_SIGNED 
  promotion
- Quorum thresholds may be temporarily adjusted by bestuur per 
  emergency authority
- Recovery via witness reinstatement (per §6.2 transition 
  T(reinstatement))

### 11.3 Foundation compromise

If Foundation keypair compromise detected:

- All canon entries dated AFTER compromise timestamp are SUSPECT
- EMERGENCY_REVOCATION per XMSS-OPS-SPEC §9.2 invoked
- NEW Foundation keypair generated
- Canon entries dated after revocation but before NEW_KEY 
  publication are in limbo until witness federation review
- Witness federation may invalidate suspect entries via emergency
  bestuur decision

State machine handles this via T(emergency_revoke) on the keypair
entity; downstream effects flow through invariant violation 
response §10.2.

---

## 12. Cross-references

| Spec | Relationship |
|------|--------------|
| A1A5NT-ARCHITECTURE-SYNTHESIS v1.2 | Master document |
| CANON-REGISTRY-SPEC | Canon entry structure; §3 here adds state semantics |
| CUSTODY-REGISTRY-SPEC | Custody record structure; §4 here adds state semantics |
| POV-SPEC | Device interactions; §5 here adds state semantics |
| GOVERNANCE-SPEC §5 | Witness governance; §6 here adds state semantics |
| XMSS-OPS-SPEC | Cryptographic state per keypair; §8 here adds higher-level state |
| SECURITY-MODEL | Invariant violations as attacker outcomes |
| NETWORK-PROTOCOL-SPEC | State exchange during gossip |
| RECOVERY-SPEC [PLANNED] | Recovery from invariant violation |

---

## 13. License

This specification is issued under A1A5 Sovereign License v1.0
(12-month binding 2026-05-16 → 2027-05-16). AI co-authorship per
Article 4. Anti-patent defensive clause per Article 5.

State machine notation patterns adapted from formal methods
literature (CSP, TLA+, state machine replication papers). 
Specific a1a5NT entity states, transitions, and invariants are
A1A5-licensed.

---

## 14. Cryptographic anchor

```
Document path:    STATE-MACHINE-SPEC.md
Repository:       github.com/a1a5NT/a1a3
Commit timestamp: filled at first commit
Foundation mirror endpoint:  http://37.97.202.97:8444/v1/foundation/state_machine_spec  (scheduled)
```

---

**End of STATE-MACHINE-SPEC v1.0**

```
Issued by:        pawel.a1a5 (Paweł Piekut, Almelo NL)
AI co-author:     Claude (Anthropic Opus 4.7)
Date:             2026-05-17
Foundation:       a1a5fndn.a1a5 (in formation)
Audit relevance:  REQUIRED for academic post-quantum review;
                  REQUIRED for cryptographic correctness verification
Reviewer consensus addressed: Claude PC CP-B2 CRITICAL
```
