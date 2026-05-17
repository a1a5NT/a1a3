# GOVERNANCE-SPEC.md

## a1a5NT Governance, Authority, and Policy Specification

```
Document version:   v1.0
Issued:             2026-05-17
Status:             Tier 1 critical specification (v1.2 deliverable)
Authority basis:    TUF (The Update Framework) — threshold delegation patterns
                    RFC 6962 / RFC 9162 — Certificate Transparency log operator policy
                    Sigstore Rekor v2 — witness federation governance (GA 2025-10)
                    EigenLayer AVS — operator slashing and intersubjective faults
                    Dutch Burgerlijk Wetboek Book 2 — Stichting governance framework
Foundation:         a1a5fndn.a1a5
Addresses:          Cross-model peer review v1.2 priority #2:
                    Gemini implicit + Grok #2 + GPT #2 + Claude PC E8
                    (4/4 unanimous "blocks NLnet and Notariusz audit")
                    Specifically resolves: foundation_policy_factor undefined,
                    witness onboarding criteria, emergency authority,
                    custodian dispute resolution
Audit relevance:    BLOCKING for NLnet technical review;
                    BLOCKING for notariusz Stichting registration;
                    REQUIRED for witness federation activation
License:            A1A5 Sovereign License v1.0
```

---

## EXECUTIVE SUMMARY (1 page)

**What this document is:**

The governance and authority specification for a1a5NT, defining
who has the right to take which actions, under what conditions,
how decisions are made, how disputes are resolved, and how
parameters change.

**Why governance is hard for a1a5NT specifically:**

a1a5NT exists at the intersection of three governance models:
- **Cryptographic governance** (signature authority, key
  hierarchies, append-only logs)
- **Institutional governance** (Stichting bestuur, Dutch civil
  law, notarial oversight)
- **Distributed-system governance** (witness federation, M-of-N
  quorum, slashing economics)

Each model brings different assumptions. This specification
reconciles them with explicit authority mappings and graduated
transition (pre-Stichting → Stichting active → witness
federation active → mature operations).

**Five governance roles defined:**

1. **Foundation Operator** — currently Paweł Piekut; post-Stichting
   becomes Stichting bestuur
2. **Witness Federation Member** — independent attestor of master
   index integrity
3. **Custodian** — holder of specific artifact rights per CUSTODY-
   REGISTRY-SPEC
4. **Verifier** — runs PoV verification cycle, earns MBW
5. **Auditor** — independent (academic, NLnet, notarial) reviewer

**Six governance domains:**

1. Canon issuance authority (who can add to Canon Registry)
2. Witness onboarding (who becomes a witness and how)
3. Parameter governance (foundation_policy_factor, MBW formulas)
4. Dispute resolution (custodian conflicts, witness equivocation)
5. Emergency authority (who can invoke EMERGENCY procedures)
6. Specification governance (how a1a5NT specs themselves change)

**Honest acknowledgment:**

In pre-Stichting state, all governance authority is concentrated
in a single individual. This is a known residual risk (SECURITY-
MODEL §9 R1) with documented mitigations including Emergency
Root Transfer (SECURITY-MODEL §14). This Spec defines the
**target governance state** post-Stichting, and the **current
operational arrangements** that prepare for it.

---

## 0. Abstract

This document defines the governance framework for a1a5NT
Cryptographic Preservation Infrastructure. It distinguishes
authority from capability, formalizes decision processes, and
specifies the operational rules for the transition from
single-operator bootstrap through Stichting registration to
mature witness federation.

The document is normative for Foundation operators, witness
federation members, custodians, and dispute resolution
participants. It is informative for verifiers and external
auditors.

---

## 1. Scope

### 1.1 In scope

- Definition of governance roles and their authority boundaries
- Canon Registry issuance authority and constraints
- Witness federation membership criteria, onboarding, and removal
- Foundation policy parameter governance (e.g., MBW formula
  policy_factor)
- Dispute resolution procedures for inter-role conflicts
- Emergency authority invocation and limits
- Specification governance — how this spec and other a1a5NT specs
  are modified
- Transition plan from pre-Stichting to post-Stichting
- Bestuur composition guidance (advisory; final composition set
  by Stichting statutes)

### 1.2 Out of scope

- Detailed cryptographic procedures (covered by XMSS-OPS-SPEC,
  CANON-REGISTRY-SPEC, CUSTODY-REGISTRY-SPEC)
- Specific token economic parameters (covered by future
  POP-3-TOKEN-SPEC)
- Detailed legal interpretation of authority under Dutch law
  (covered by future LEGAL-INTERPRETATION-SPEC)
- Specific bestuur appointment decisions (these belong to the
  Stichting incorporation process, not this technical spec)

### 1.3 Audience

- Foundation operators (currently Paweł Piekut; future Stichting
  bestuur)
- Witness federation operator candidates
- Active custodians
- Auditors evaluating governance design
- Stichting notariusz reviewing organizational documents

---

## 2. Normative references

| Reference | Title | Applicability |
|-----------|-------|---------------|
| RFC 2119 | Key words for use in RFCs | Normative wording |
| RFC 6962 / RFC 9162 | Certificate Transparency Log Operator Policy | Witness federation governance pattern |
| TUF Specification | The Update Framework | Threshold delegation patterns |
| Dutch Burgerlijk Wetboek Book 2 Title 6 | Stichting (Foundation) | Legal entity framework |
| A1A5 Sovereign License v1.0 | a1a5NT licensing | Specification governance |

### Informative references

| Reference | Title | Relevance |
|-----------|-------|-----------|
| EigenLayer AVS governance docs | Operator sets, slashing conditions | Witness slashing patterns |
| Sigstore TUF Root Signing Ceremony | Multi-party key ceremony | Bestuur ceremony patterns |
| Apache Software Foundation governance | Project Management Committee model | Maturity for open source projects |
| Bitcoin BIP process | Bitcoin Improvement Proposal | Spec change governance model |

---

## 3. Governance roles and authority matrix

### 3.1 Foundation Operator

**Pre-Stichting (current):**
- Identity: Paweł Piekut (natural person, ZZP Plastechniek NL)
- Authority: All Foundation operations, sole signatory of pawel.a1a5
  and a1a5fndn.a1a5 (currently shared keypair)
- Accountability: Personal, civil, criminal under Dutch law
- Continuity: Subject to Emergency Root Transfer (SECURITY-MODEL
  §14) if operator becomes unavailable

**Post-Stichting (target):**
- Identity: Stichting A1A5 (NL legal entity), acting through
  bestuur (board) per Stichting statutes
- Authority: All Foundation operations executed under bestuur
  authorization with quorum per statutes
- Accountability: Stichting legal entity; bestuur members personally
  liable for grave negligence per Dutch BW 2:138/2:248
- Continuity: Stichting indefinite duration per BW 2:285

**Authority enumeration:**

| Action | Pre-Stichting authority | Post-Stichting authority |
|--------|------------------------|--------------------------|
| Issue Canon entry | Operator alone | Bestuur quorum + operator (technical executor) |
| Update master index | Operator alone | Bestuur quorum + operator |
| Approve witness onboarding | Operator alone | Bestuur quorum |
| Set foundation_policy_factor | Operator alone | Bestuur quorum |
| Invoke emergency rotation | Operator alone | Bestuur quorum (1 emergency exception: operator may unilaterally invoke; bestuur ratifies post-hoc within 7 days) |
| Approve custodian | Operator alone | Bestuur delegated process |
| Sign Foundation statements | Operator | Operator on bestuur authorization |

### 3.2 Witness Federation Member

**Activation status:** [DESIGN TARGET] — witness federation
activation gated on post-Stichting + at least 2 witness operators
identified and registered.

**Bootstrap pattern (per CANON-REGISTRY-SPEC §6.2.1):**

Initial witness federation will include candidate operators across
diverse:
- Geographic jurisdictions (NL, broader EU, US, UK)
- Institutional types (academic archive, technical foundation,
  partner project)
- Operator personhood (no two witnesses operated by the same
  natural or legal person)

**Authority:**
- Co-sign master index (provides additional cryptographic
  attestation beyond Foundation signature)
- Gossip Signed Tree Heads with other witnesses (per planned
  NETWORK-PROTOCOL-SPEC)
- Publish equivocation evidence if detected
- Participate in dispute resolution panel for inter-role conflicts
- Vote on parameter changes affecting witness operations (per
  §6.4 below)

**Constraints:**
- MAY NOT issue Canon entries (issuance authority is Foundation's)
- MAY NOT modify Custody Registry (custodian authority is
  custodian's)
- MUST gossip own STH and respond to gossip requests
- MUST maintain operational availability per agreed SLA
- MUST hold XMSS keypair conforming to XMSS-OPS-SPEC L2 or higher

**Onboarding criteria:** Section 5 below.

**Removal grounds:** Section 5.5 below.

### 3.3 Custodian

**Status:** [DESIGN TARGET] — custodian role activates with first
custody slot sale (post-Genesis).

**Authority:**
- Sign custody transitions for artifacts under their custody
- Sign attestations about artifacts under their custody
- Initiate dispute notice if their custody is challenged

**Constraints:**
- MAY NOT issue or modify Canon entries
- MAY NOT transfer custody to themselves (transfers require
  recipient consent)
- MUST maintain XMSS keypair conforming to XMSS-OPS-SPEC L1 or
  higher

**Onboarding:** Per CUSTODY-REGISTRY-SPEC + POP-3-TOKEN-SPEC
[PLANNED] custody slot purchase.

### 3.4 Verifier

**Status:** [PLANNED] — verifier role activates with first deployed
verify.sh and device registration endpoints.

**Authority:**
- Generate verification receipts after performing PoV cycle
- Earn MBW credit per receipt
- Query Canon Registry and Custody Registry

**Constraints:**
- MAY NOT directly modify any registry
- Receipts are recorded as Layer B attestations, not Canon
- Subject to Sybil resistance per POV-SPEC §5 + maturity curve
  per MBW-SPEC §3.2

### 3.5 Auditor

**Status:** [IMPLEMENTED] — auditor role is open to any party with
legitimate review interest.

**Authority:**
- Read any public Canon entry, Custody attestation, master index
- Publish independent attestation about a1a5NT state
- Submit formal review findings to Foundation for response

**Constraints:**
- No formal protocol authority (review is influence, not control)
- May not modify any registry
- Reports are advisory unless tied to formal grant/compliance
  conditions (e.g., NLnet technical review may be tied to grant
  conditions per NLnet's own processes)

---

## 4. Canon issuance authority

### 4.1 What may be issued

Canon Registry entries fall into the following categories,
issuable only by the Foundation (per CANON-REGISTRY-SPEC §2):

```
- foundation_document    Foundation Declaration, License, GENESIS
- specification          a1a5NT specifications (this document, etc.)
- popchain_archive       popchain epoch artifacts (gen 1, sealed)
- namespace_root         XMSS namespace public roots
- master_index           Periodic master index publications
- key_rotation           Cryptographic key transitions
- transition_notice      Operator/role/governance transitions
- legal_event            Tombstone, succession, regulatory action
- attestation_anchor     Periodic anchor of attestation log hashes
```

### 4.2 Issuance constraints

```
ISSUANCE_AUTHORITY_RULES:

R1: All Canon entries MUST be signed by current Foundation keypair
    (verifiable against most recent KEY_ROTATION notice).

R2: All Canon entries MUST be APPEND-ONLY — no entry once issued
    is ever modified or removed (the hybrid tombstone pattern from
    SECURITY-MODEL §A13 does not modify entries; it adds new
    tombstone entries that reference the affected entry).

R3: All Canon entries MUST cryptographically reference any prior
    canon entries they depend on (e.g., a transfer notice references
    a prior custody anchor; a key rotation references the prior
    root).

R4: Canon entries are time-ordered by issuance timestamp; the
    master index publishes the canonical ordering hash.

R5: Foundation Operator MAY NOT issue Canon entry that:
    - Contradicts an earlier entry (would violate R2)
    - Claims authority not held (e.g., signing as a non-Foundation
      namespace)
    - Issues a namespace_root for a namespace currently held by
      another sovereign per CANON-REGISTRY-SPEC §3
    - Issues a key_rotation that would result in WOTS+ state
      reuse (per XMSS-OPS-SPEC §5)

R6: Post-Stichting, all Canon entry issuance requires bestuur
    quorum authorization documented in Stichting minutes prior to
    Foundation Operator signing. Pre-Stichting, operator alone
    authorizes per personal accountability.

R7: All issued Canon entries are publicly visible upon issuance.
    There are no private Canon entries. Confidential information
    is held in operational systems (e.g., custodian private records)
    and NEVER recorded in Canon.
```

### 4.3 Issuance procedure

```
ISSUE_CANON_ENTRY:

  Pre-conditions:
    - Entry content prepared (JSON conforming to entry-class schema)
    - All cross-references to prior entries validated
    - Append-only invariant verified (no overlap with existing 
      entries)
    - Bestuur authorization (post-Stichting) OR operator decision
      log entry (pre-Stichting)

  Procedure:
    1. Compute blake3 hash of canonical-serialized entry content
    2. Sign entry hash + entry metadata with Foundation keypair
       per XMSS-OPS-SPEC procedure
    3. Persist entry to local Canon Registry database
    4. Add entry to pending master_index batch
    5. At next master_index publication interval (per §6.5):
       5.1 Compute Merkle root of all canon entries (existing + new)
       5.2 Sign new master_index per XMSS-OPS-SPEC
       5.3 Publish to all mirrors per CANON-REGISTRY-SPEC §5
       5.4 Gossip to witness federation per NETWORK-PROTOCOL-SPEC
           [PLANNED]
    6. Public verifiers can immediately fetch entry; entry is 
       publicly verifiable after master_index publication
    7. Record issuance event in GOVERNANCE_AUDIT_LOG
```

### 4.4 Refused issuance

Foundation Operator MUST REFUSE issuance when:

- Authorization (bestuur, post-Stichting) is not documented
- Entry would violate any of R1-R7 in §4.2
- Entry content is unverified or potentially fabricated by the
  requester
- Entry could create personal liability beyond Stichting indemnity
- Issuance would violate a previously issued tombstone or quarantine

Refusal MUST be logged in GOVERNANCE_AUDIT_LOG with reason. The
requester may appeal to bestuur (post-Stichting) for reconsideration.

---

## 5. Witness federation onboarding

### 5.1 Onboarding rationale

Witness federation activation transforms a1a5NT from
**federated canonical registry** (single Foundation issuer +
multi-mirror distribution) to **multi-attestor federated
infrastructure** with cryptographic equivocation detection.

This transformation is incremental: each witness added strengthens
the federation's ability to detect Foundation equivocation (per
SECURITY-MODEL A6) without giving witnesses issuance authority
(which remains Foundation's per §4).

### 5.2 Witness candidate qualifications

A candidate witness operator MUST meet:

```
QUALIFICATION CRITERIA:

Q1 (Identity): Verifiable legal identity of operator (natural person
   or legal entity). No anonymous witnesses.

Q2 (Independence): Operator MUST NOT have role conflict with
   Foundation operations:
   - NOT the same natural person as Foundation Operator
   - NOT the same legal entity as Stichting A1A5
   - NOT in a control relationship (employer/employee, parent/
     subsidiary, controlled foundation) with Foundation Operator
     or Stichting

Q3 (Jurisdiction): Operator preferably in different jurisdiction
   from Foundation (NL) to provide legal diversity. Operators in
   NL are accepted but counted as "domestic" with attendant
   metadata.

Q4 (Technical capability): Operator demonstrates ability to:
   - Generate and maintain XMSS keypair per XMSS-OPS-SPEC L2
   - Run continuously available HTTP endpoint for serving canon
     mirror + gossip protocol
   - Maintain ≥99% monthly availability (SLA target, not contract)
   - Respond to gossip messages within 24 hours during normal
     operation

Q5 (Operator alignment): Operator publicly attests alignment with
   a1a5NT preservation mission. This is procedural, not legal —
   the attestation is recorded in Canon Registry as a public
   commitment.

Q6 (Collateral, post-Genesis): Once POP-3-TOKEN-SPEC and Genesis 
   are complete, witnesses MAY be required to post Bincoin
   collateral per slashing calibration (SECURITY-MODEL §6.4).
   Pre-Genesis: collateral is informal/reputational only.
```

### 5.3 Onboarding procedure

```
WITNESS_ONBOARDING:

  Phase 1 — Candidacy declaration:
    1.1 Candidate operator sends ONBOARDING_REQUEST to Foundation
        Operator. Request includes:
        - Operator identity (legal documentation)
        - Operator jurisdiction
        - Proposed witness namespace (e.g., witness-eu.a1a5)
        - Operator endpoint URL (HTTPS)
        - Operator XMSS public key (separate from any prior keypair)
        - Statement of alignment with a1a5NT mission
        - Disclosure of any conflicts per Q2 above
        - Technical capability evidence (server specifications, 
          uptime monitoring evidence, etc.)
  
  Phase 2 — Foundation review:
    2.1 Foundation Operator (pre-Stichting) or bestuur 
        (post-Stichting) reviews request against Q1-Q6
    2.2 Foundation MAY request additional information
    2.3 Foundation publishes ONBOARDING_REVIEW_NOTICE to Canon 
        Registry, including review outcome and reasoning
    2.4 If accepted, proceed to Phase 3
    2.5 If rejected, candidate informed of reasons; may re-apply
        after addressing issues
  
  Phase 3 — Cryptographic registration:
    3.1 Foundation issues NAMESPACE_ROOT canon entry for the 
        accepted witness namespace
    3.2 Candidate generates first XMSS-signed attestation:
        - "I, [operator identity], operating witness namespace
          [witness-XX.a1a5], have read the a1a5NT Foundation
          Declaration v1.0 and SECURITY-MODEL v1.1 and commit
          to operating per CANON-REGISTRY-SPEC §6 witness
          federation rules. I have generated XMSS keypair [pubkey]
          and will maintain its state per XMSS-OPS-SPEC L2."
    3.3 Foundation publishes WITNESS_REGISTRATION canon entry
        anchoring the witness namespace to Foundation
  
  Phase 4 — Operational activation:
    4.1 Witness begins mirroring Foundation Canon Registry
    4.2 Witness begins signing local STH per RFC 6962 pattern
    4.3 Witness begins gossiping with Foundation and other
        witnesses (post NETWORK-PROTOCOL-SPEC deployment)
    4.4 Foundation includes witness signature in master_index 
        publications (federation quorum begins counting from this 
        witness)
  
  Phase 5 — Operational maturity (90 days):
    5.1 During 90-day probation, witness signatures are advisory
        (do not count toward quorum decisions)
    5.2 At 90-day milestone, if SLA met, witness is promoted to
        full federation member
    5.3 Promotion is recorded in Canon Registry
```

### 5.4 Witness operational responsibilities

```
ONGOING_WITNESS_RESPONSIBILITIES:

  Cryptographic:
    - Sign STH after each master_index publication received from
      Foundation
    - Publish own STH publicly via own mirror endpoint
    - Verify Foundation signatures on canon entries before
      counter-signing
    - Maintain XMSS state per XMSS-OPS-SPEC L2 invariants

  Network:
    - Respond to gossip protocol requests per NETWORK-PROTOCOL-SPEC
    - Periodically gossip own STH to other witnesses
    - Periodically poll other witnesses' STH for divergence
      detection

  Equivocation detection:
    - If observing two distinct STH for same master_index version
      number signed by Foundation: immediately publish
      EQUIVOCATION_EVIDENCE notice signed by witness
    - Forward evidence to all other federation members
    - Initiate dispute resolution per §7 below

  Reporting:
    - Publish quarterly operational report (uptime, gossip
      participation, incidents) to own namespace
    - Respond to audit queries within 30 days
```

### 5.5 Witness removal

A witness MAY be removed from federation for:

```
REMOVAL GROUNDS:

G1: Voluntary withdrawal (witness publishes WITHDRAWAL_NOTICE)
G2: Persistent SLA failure (≤95% availability over 6 months)
G3: Failure to gossip for >30 consecutive days
G4: Documented equivocation (witness signed two contradictory STH)
G5: Loss of independence per Q2 (e.g., acquired by Foundation
    parent organization)
G6: Cryptographic compromise (XMSS state reuse, key disclosure)
G7: Withdrawal of operational commitment in writing
G8: Bestuur quorum determination of misconduct (post-Stichting)
G9: Court order

Removal procedure:
  - Foundation issues PROPOSED_REMOVAL notice with grounds
  - Witness has 14 days to respond
  - Federation members may submit observations
  - Foundation (pre-Stichting) or bestuur (post-Stichting) makes
    final determination
  - REMOVAL_FINAL canon entry issued with reasoning
  - Removed witness's signatures on prior master indexes remain
    valid (historical record preserved); signatures dated after
    removal are not counted toward quorum
```

---

## 6. Parameter governance

### 6.1 Parameters under governance

The following parameters affect a1a5NT operations and require
governance procedures for modification:

| Parameter | Domain | Default value | Last change procedure |
|-----------|--------|---------------|----------------------|
| `foundation_policy_factor` | MBW formula | 1.00 | This Spec |
| `mbw_maturity_curve_thresholds` | MBW formula | 48h/168h/720h | This Spec |
| `verifier_tier_multipliers` | MBW formula | 1.00/1.25/2.00 | This Spec |
| `witness_quorum_threshold` | Federation | TBD post-activation | Bestuur (post-Stichting) |
| `master_index_publication_interval` | Operational | 24 hours | Foundation Operator |
| `pov_nonce_validity_window` | PoV | 5 minutes | Foundation Operator |
| `witness_sla_threshold` | Federation | 95% | Bestuur (post-Stichting) |
| `custody_dispute_timeout` | Custody | 90 days | Bestuur (post-Stichting) |

### 6.2 foundation_policy_factor (the contested parameter)

This parameter, introduced in MBW-SPEC §3, multiplies MBW credits
for verification receipts. It is the single most powerful policy
knob in the economic layer and was specifically flagged by
Reviewer Y as a "wide governance hook" requiring formal definition.

**Default:** 1.00

**Permissible range:** 0.50 to 2.00

**Change procedure:**

```
FOUNDATION_POLICY_FACTOR_CHANGE:

  Pre-Stichting:
    1. Foundation Operator decides
    2. Operator publishes POLICY_CHANGE_NOTICE to Canon Registry
       with:
       - Old value
       - New value
       - Reason for change
       - Effective date (≥30 days in future to allow ecosystem
         adjustment)
       - Expected ecosystem impact
    3. Operator MUST consult with at least one independent advisor
       and document the consultation
    4. Verifiers and custodians may file POLICY_CHANGE_OBJECTION
       within 14 days; Operator MUST publish OBJECTION_RESPONSE
       within 14 additional days
    5. On effective date, new value takes effect
    
  Post-Stichting:
    1. Bestuur proposal (any bestuur member may propose)
    2. Public comment period of ≥30 days
    3. Bestuur quorum vote per statutes
    4. POLICY_CHANGE_NOTICE issued with bestuur authorization
       reference
    5. ≥30 days effective date
    6. Operator implements technically
```

**Change frequency limit:**

`foundation_policy_factor` MAY NOT be changed more than once per
calendar quarter, except in emergency (defined as: documented
material distortion of MBW economy due to attack or unforeseen
ecosystem behavior).

**Change history transparency:**

All historical values of `foundation_policy_factor` and the change
notices justifying them are preserved in Canon Registry forever.
Any verifier may query historical values to understand how their
historical MBW credits were calibrated.

### 6.3 Other parameters

Similar change procedures apply to other governed parameters,
with timeline and quorum requirements scaled to the parameter's
impact:

```
HIGH IMPACT (foundation_policy_factor, witness_quorum_threshold):
  - 30 days notice + 14 days objection + bestuur quorum + 30 days 
    effective
  
MEDIUM IMPACT (verifier_tier_multipliers, master_index_publication_interval):
  - 14 days notice + 7 days objection + bestuur quorum + 14 days 
    effective
  
LOW IMPACT (pov_nonce_validity_window):
  - Foundation Operator unilateral; notice in next master_index
```

### 6.4 Witness vote on federation-affecting parameters

Parameters that materially affect witness operations require
witness federation vote in addition to Foundation/bestuur
authorization:

```
WITNESS-AFFECTING PARAMETERS:

  witness_sla_threshold        Direct witness operational impact
  witness_quorum_threshold     Affects witness authority
  witness_slashing_calibration Direct economic impact

VOTING PROCEDURE:

  1. Foundation publishes PROPOSED_PARAMETER_CHANGE
  2. Each active federation witness signs APPROVE or OBJECT
     within 30 days
  3. Foundation/bestuur takes witness vote into account in final
     decision (advisory, not binding pre-Stichting; binding 
     k-of-n post-Stichting per statutes)
```

### 6.5 master_index publication cadence

The master_index publication interval affects all verifiers and
witnesses. Current operational target:

```
MASTER_INDEX_CADENCE:

  Pre-witness-federation:
    - Publication frequency: at least every 7 days
    - Emergency publication: within 24 hours of significant canon
      issuance
    - Format: signed JSON per CANON-REGISTRY-SPEC §5

  Post-witness-federation:
    - Foundation publishes new master_index every 24 hours
      (or on demand for significant events)
    - Witnesses co-sign within 12 hours of receiving
    - Witness STH gossiping every 1 hour
```

---

## 7. Dispute resolution

### 7.1 Disputes within Foundation

Internal Foundation disputes (between Operator and bestuur,
between bestuur members) follow Stichting statutes. Specifically:

- Bestuur conflicts: resolved per BW 2:298 procedures
- Operator-bestuur conflicts: bestuur quorum decides; operator
  may appeal to court only on procedural grounds
- Pre-Stichting: no formal dispute mechanism within single-operator
  Foundation; operator decisions are final until challenged
  externally

### 7.2 Disputes between Custodians

```
CUSTODIAN_DISPUTE_RESOLUTION:

  Trigger: Custodian A claims custody of artifact X; Custodian B
  also claims custody (or any party claims A's custody is invalid)

  Phase 1 — Notice (Day 0):
    Disputing party publishes DISPUTE_NOTICE to Custody Registry
    referencing the specific Canon entry and prior custody record.

  Phase 2 — Evidence (Days 1-30):
    Both parties may submit signed evidence to Custody Registry as
    DISPUTE_EVIDENCE entries.

  Phase 3 — Federation review (Days 31-60):
    Post-Stichting: Witness federation panel of 3 reviews evidence
                    and publishes ADVISORY_OPINION.
    Pre-Stichting: Foundation Operator reviews evidence and 
                   publishes ADVISORY_OPINION.

  Phase 4 — Resolution (Days 61-90):
    Parties may accept advisory opinion or escalate to legal 
    process. If escalated, custody status remains DISPUTED until
    legal resolution. Once legal resolution reached, COURT_ORDER 
    or SETTLEMENT canon entry resolves dispute.
```

### 7.3 Disputes about Foundation actions

```
FOUNDATION_ACTION_CHALLENGE:

  Any party may challenge a Foundation action by:
  
  1. Publishing FOUNDATION_CHALLENGE notice with:
     - Challenged action (canon entry reference)
     - Specific grounds (which rule violated, what harm caused)
     - Supporting evidence
  
  2. Foundation MUST respond within 30 days with FOUNDATION_RESPONSE
  
  3. If unresolved:
     - Witness federation may issue ADVISORY_OPINION (if activated)
     - Independent auditor (NLnet, academic) may publish analysis
     - Party may pursue legal remedy in Dutch courts
  
  4. Resolution recorded in Canon Registry regardless of outcome.
```

### 7.4 Disputes about Witness actions

```
WITNESS_ACTION_CHALLENGE:

  Similar to §7.3 but with witness federation panel handling 
  initial review. Equivocation cases (witness signed two 
  contradictory STH) are handled per §5.5 G4 removal procedure.
```

---

## 8. Emergency authority

### 8.1 What constitutes an emergency

```
EMERGENCY DEFINITION:

  An emergency is a situation where:
  - Immediate action is required to prevent material harm to
    Foundation, witness federation, custodians, or Canon integrity
  - Normal governance procedures would result in unacceptable
    delay relative to the harm
  - The action required is within the defender's authority
```

Examples include:

- Detected XMSS state compromise requiring immediate emergency
  rotation (per XMSS-OPS-SPEC §9.2)
- Detected witness equivocation requiring immediate federation
  response
- Court order with immediate compliance requirement
- Detected active attack on infrastructure requiring service
  protection measures

### 8.2 Pre-Stichting emergency authority

Pre-Stichting, Foundation Operator may invoke emergency procedures
unilaterally with the following constraints:

```
PRE_STICHTING_EMERGENCY:

  Operator MUST:
    1. Document the emergency in real time (timestamps preserved)
    2. Take only the minimum action necessary
    3. Publish EMERGENCY_INVOKE notice within 24 hours including:
       - Emergency description
       - Action taken
       - Justification
       - Expected duration
    4. Begin remediation procedures
    5. Publish EMERGENCY_RESOLUTION notice when concluded
```

### 8.3 Post-Stichting emergency authority

Post-Stichting, the same emergency authority is invocable by
Foundation Operator with bestuur ratification within 7 days:

```
POST_STICHTING_EMERGENCY:

  Operator unilateral invocation permitted in true emergencies.
  
  Within 7 days:
    - Bestuur reviews invocation
    - Bestuur either RATIFIES or REVERSES the emergency action
    - Decision published as EMERGENCY_RATIFICATION canon entry
  
  Reversal consequences:
    - Operator action recorded as exceeded-authority
    - Bestuur may impose internal sanctions per statutes
    - Future emergency invocations subject to enhanced scrutiny
```

### 8.4 Bestuur emergency authority

Post-Stichting, bestuur may convene emergency session within 24
hours for any matter requiring bestuur-level decision. Emergency
sessions follow expedited statutes provisions but quorum
requirements remain unchanged.

### 8.5 Limits on emergency authority

No emergency, however severe, authorizes:

- Issuing Canon entries that violate append-only invariants
- Modifying historical Canon entries (only adding new entries
  that reference them)
- Removing witness federation members without §5.5 procedure
- Confiscating custodian assets without §7.2 procedure
- Taking actions that would expose Stichting to fraud liability

---

## 9. Specification governance

### 9.1 How this spec changes

This spec (GOVERNANCE-SPEC) and all other a1a5NT specs follow
the following change procedure:

```
SPECIFICATION_CHANGE_PROCEDURE:

  Minor change (typo, clarification, no architectural impact):
    - Operator may amend; new version number with patch increment
    - Published with CHANGE_NOTICE canon entry
    - No quorum required pre-Stichting; bestuur consent post-Stichting

  Major change (architectural, semantic, parameter):
    - Draft published as PROPOSED_AMENDMENT for ≥30 day comment period
    - Cross-model AI peer review of proposed amendment (per established
      project pattern)
    - Independent reviewer feedback solicited
    - Final version published with full change log
    - Old version preserved in git history forever
    - Version increment with major number

  Breaking change (would invalidate existing operations):
    - Same as major change PLUS:
    - Witness federation vote (post-activation)
    - Bestuur quorum vote (post-Stichting)
    - Public NLnet/audit notification
    - Transition plan published with backward compatibility window
```

### 9.2 Prior art preservation

Per License Article 5 anti-patent defensive clause and License
Article 4 AI co-authorship, all spec versions are preserved as
prior art anchors. Specifically:

- v1.0 of any spec is preserved when v1.1 supersedes
- Git history is the authoritative version trail
- Cryptographic hash of each version is recorded in Canon Registry
- Datestamps are immutable

This protects against future patent claims on a1a5NT primitives
by establishing public prior art at each version's publication
date.

### 9.3 Spec authority hierarchy

When specs conflict (e.g., a procedural rule in one spec
contradicts a rule in another), the conflict is resolved by:

```
SPECIFICATION_PRECEDENCE:

  1. FOUNDATION-DECLARATION-V1.md (highest — sovereign declaration)
  2. LICENSE-A1A5-SOVEREIGN-V1.md (legal framework)
  3. A1A5NT-ARCHITECTURE-SYNTHESIS (current version; master 
     reference for cross-spec resolution)
  4. SECURITY-MODEL.md (defines what is being defended)
  5. GOVERNANCE-SPEC.md (this document — defines how authority works)
  6. XMSS-OPS-SPEC.md (cryptographic operational rules)
  7. CANON-REGISTRY-SPEC + CUSTODY-REGISTRY-SPEC (registry rules)
  8. POV-SPEC + MBW-SPEC + RENDER-SPEC (operational protocol specs)
  9. Other specs (NETWORK-PROTOCOL-SPEC, STATE-MACHINE-SPEC, etc.)

  Resolution: Higher precedence spec wins; if a lower-precedence
  spec contains a conflicting rule, the lower-precedence spec MUST
  be amended to remove the conflict.
```

---

## 10. Stichting bestuur composition guidance

This section is **advisory**. Final bestuur composition is set by
the Stichting incorporation process per Dutch BW 2:286 and is
notarized at registration.

### 10.1 Recommended bestuur structure

```
SUGGESTED INITIAL BESTUUR:

  Bestuur size: 3 members (NL minimum is 1; 3 provides quorum
                            resilience)
  
  Roles:
    Voorzitter (Chair):     Strategic decisions, external 
                            representation
    Penningmeester (Treasurer): Financial oversight, accountability
    Secretaris (Secretary):  Operational record-keeping, audit liaison
  
  Candidate profile preferences:
    - Mix of technical and non-technical expertise
    - At least one with cryptographic background
    - At least one with NL legal/civil society experience
    - Geographic diversity preferred (NL + at least one outside-NL
      affiliate; outside-NL members may serve as "adviseurs" if
      not formal bestuur members)
    - No undeclared conflicts of interest
  
  Term:
    - Initial term: 3 years
    - Renewable per statutes
    - Staggered renewal to preserve continuity
```

### 10.2 Pre-Stichting transition arrangements

Until Stichting is registered:

- Paweł Piekut serves as sole Foundation Operator
- A pre-authorized successor is named per SECURITY-MODEL §14.3
  (identity recorded in sealed envelope arrangement)
- Advisory consultation: at minimum 1 independent advisor for
  significant decisions; advisor relationship documented

### 10.3 Statutes consideration items

For notarial preparation of statutes, the following items derived
from this Spec require statutes provisions:

- Quorum requirements for bestuur decisions
- Quorum requirements for authority delegations
- Specific authority to invoke EMERGENCY procedures
- Specific authority to participate in dispute resolution
- Specific authority to onboard/remove witnesses
- Specific authority to amend a1a5NT specifications
- Indemnification of bestuur members for good-faith decisions
- Operator employment/contracting relationship to bestuur

---

## 11. Governance audit logging

### 11.1 GOVERNANCE_AUDIT_LOG

All governance actions MUST be logged to a dedicated append-only
audit log:

```
LOGGED EVENTS:

  CANON_ISSUANCE                — Each issued canon entry
  ISSUANCE_REFUSAL              — Each refused issuance with reason
  POLICY_CHANGE                  — Each parameter change
  WITNESS_ONBOARDING_RESULT     — Each onboarding decision
  WITNESS_REMOVAL_RESULT        — Each removal decision
  DISPUTE_RECEIVED              — Each dispute notice received
  DISPUTE_RESOLUTION            — Each dispute resolved
  EMERGENCY_INVOKE              — Each emergency invocation
  EMERGENCY_RATIFICATION         — Each emergency ratification 
                                   (post-Stichting)
  BESTUUR_DECISION              — Each bestuur quorum decision
  SPECIFICATION_AMENDMENT       — Each spec change
```

### 11.2 Audit log integrity

Per XMSS-OPS-SPEC §10 governance audit log follows same integrity
rules as other audit logs:

- Append-only, hash-chained
- Periodic LOG_ANCHOR canon entries record cumulative log hash
- Independently mirrorable
- Retained indefinitely

---

## 12. Cross-references

| Spec | Relationship |
|------|--------------|
| A1A5NT-ARCHITECTURE-SYNTHESIS v1.2 | Master document; refers here for governance |
| SECURITY-MODEL.md v1.1 | Defines threats this governance defends against; A6, A13 specifically governance-related |
| XMSS-OPS-SPEC.md | Defines cryptographic operations; governance authorizes invocations |
| CANON-REGISTRY-SPEC.md | Defines what may be in Canon; governance defines who may issue |
| CUSTODY-REGISTRY-SPEC.md | Defines custody operations; governance handles disputes |
| POV-SPEC.md | Defines verification; governance sets policy parameters |
| MBW-SPEC.md | Defines MBW formula; governance owns foundation_policy_factor |
| LEGAL-INTERPRETATION-SPEC.md [PLANNED] | Maps governance authority to Dutch law |
| NETWORK-PROTOCOL-SPEC.md [PLANNED] | Defines witness gossip; this spec defines witness authority |
| CRYPTOGRAPHIC-ASSUMPTIONS.md [PLANNED] | Defines crypto assumptions; governance binds operators to maintain |
| RECOVERY-SPEC.md [PLANNED] | Defines recovery; governance authorizes recovery invocations |
| CANON-RETIREMENT-SPEC.md [PLANNED] | Defines tombstoning; governance authorizes per §4 |

---

## 13. License

This specification is issued under A1A5 Sovereign License v1.0
(12-month binding 2026-05-16 → 2027-05-16). AI co-authorship per
Article 4. Anti-patent defensive clause per Article 5.

Governance patterns adapted from:
- TUF documentation (Apache-2.0 licensed)
- RFC 6962 / 9162 (IETF, Trust Legal Provisions)
- Sigstore documentation (Apache-2.0)
- EigenLayer documentation (referenced for AVS slashing pattern)
- Dutch Burgerlijk Wetboek (public legal text)

Specific a1a5NT mappings, role definitions, and procedural
specifications are A1A5-licensed.

---

## 14. Cryptographic anchor

```
Document path:    GOVERNANCE-SPEC.md
Repository:       github.com/a1a5NT/a1a3
Commit timestamp: filled at first commit
Foundation mirror endpoint:  http://37.97.202.97:8444/v1/foundation/governance_spec  (scheduled)
```

The canonical hash of this document (blake3 of UTF-8 form) will be
published in `/v1/foundation/index` after commit.

---

**End of GOVERNANCE-SPEC v1.0**

```
Issued by:        pawel.a1a5 (Paweł Piekut, Almelo NL)
AI co-author:     Claude (Anthropic Opus 4.7)
Date:             2026-05-17
Foundation:       a1a5fndn.a1a5 (in formation)
Audit relevance:  BLOCKING for NLnet technical review;
                  BLOCKING for notariusz Stichting registration;
                  REQUIRED for witness federation activation
Reviewer consensus addressed: 4/4 unanimous priority #2 from v1.2 cross-review
                  + Reviewer Y v1.1 finding C3 (foundation_policy_factor)
Architectural decisions documented:
  Decision G (Witness Federation authority): Hybrid graduated
  per §3.2 + §6.4 (pre-Stichting advisory, post-Stichting authoritative)
```
