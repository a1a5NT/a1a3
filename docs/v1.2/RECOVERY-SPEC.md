# RECOVERY-SPEC.md

## a1a5NT Recovery and Incident Response Specification

```
Document version:   v1.0
Issued:             2026-05-17
Status:             Tier 1 critical specification (v1.2 deliverable)
Authority basis:    NIST SP 800-61 Rev 2 (Computer Security Incident
                    Handling Guide),
                    NIST SP 800-184 (Guide for Cybersecurity Event 
                    Recovery),
                    SECURITY-MODEL §8 failure mode classification,
                    XMSS-OPS-SPEC §9 emergency rotation,
                    GOVERNANCE-SPEC §8 emergency authority
Foundation:         a1a5fndn.a1a5
Addresses:          SECURITY-MODEL Cat-1, Cat-2, Cat-3 catastrophic
                    failure modes; H-1, H-2, H-3 high-severity modes;
                    Reviewer N N-B2 witness compromise recovery
Audit relevance:    REQUIRED for NLnet technical review;
                    REQUIRED for institutional adoption (operational
                    resilience demonstration)
License:            A1A5 Sovereign License v1.0
```

---

## EXECUTIVE SUMMARY (1 page)

**What this document is:**

The recovery and incident response specification for a1a5NT,
defining how the system recovers from cryptographic, operational,
and infrastructure failures.

**Why it matters:**

Every cryptographic system experiences incidents. Some are minor
(transient network failures). Some are existential (root key
compromise). What separates a mature infrastructure from a
fragile one is **recovery planning**.

This document defines what happens when things go wrong, who has
authority to invoke recovery, what procedures are followed, and
how the system returns to verified-correct state.

**Six recovery scenarios documented:**

1. **R-KEY-LOSS** — Foundation root keypair state file lost
2. **R-KEY-COMPROMISE** — Foundation root keypair compromised
3. **R-WITNESS-COMPROMISE** — Federation witness compromised
4. **R-CUSTODIAN-INCAPACITATION** — Custodian becomes unreachable
5. **R-CANON-FORK** — Two contradictory canonical histories exist
6. **R-INFRASTRUCTURE-LOSS** — Catastrophic loss of operating
   infrastructure (hosting, data center)

**Each scenario specifies:**

- Trigger detection
- Authority required for invocation
- Procedure (step-by-step)
- Verification of recovery
- Documentation requirements
- Long-term system state implications

**The fundamental recovery principle:**

a1a5NT's recovery is grounded in **monotonic history preservation**.
No recovery action ever modifies prior canonical records. Recovery
ADDS new entries that supersede, deprecate, or quarantine prior
entries; it never rewrites history.

**The recovery time objective (RTO):**

```
For incidents detected and confirmed:

  R-KEY-LOSS:                Recovery in <72 hours
  R-KEY-COMPROMISE:          New keypair operational in <24 hours
  R-WITNESS-COMPROMISE:      Removal in <72 hours
  R-CUSTODIAN-INCAPACITATION: Resolution per CUSTODY procedures (90 days)
  R-CANON-FORK:              Reconciliation in <14 days
  R-INFRASTRUCTURE-LOSS:     Service restoration in <14 days
```

These are targets, not contractual SLAs. Real incidents may exceed
these.

---

## 0. Abstract

This document specifies how a1a5NT recovers from cryptographic
and operational failures. It complements SECURITY-MODEL
(threat enumeration) and XMSS-OPS-SPEC §9 (emergency rotation)
with operational recovery procedures.

The document is normative for Foundation operators executing
recovery and informative for auditors evaluating resilience.

---

## 1. Scope

### 1.1 In scope

- Recovery procedures for cryptographic key incidents
- Recovery procedures for operational failures
- Authority requirements for recovery invocation
- Documentation requirements during recovery
- Verification procedures post-recovery
- Communication requirements (internal and external)

### 1.2 Out of scope

- Detailed cryptographic operations (covered by XMSS-OPS-SPEC)
- Incident detection mechanisms (operational concern)
- Backup procedures themselves (operational concern; recovery
  uses backups but doesn't define them)
- Performance recovery (e.g., reducing query latency)
- Legal proceedings (covered by LEGAL-INTERPRETATION-SPEC and
  separately handled by counsel)

### 1.3 Audience

- Foundation operators executing recovery
- Stichting bestuur (authorizing recovery, post-Stichting)
- Auditors evaluating recovery preparedness
- Witness operators (subject to recovery procedures)
- Custodians (potentially affected by recovery)

---

## 2. Normative references

| Reference | Title | Applicability |
|-----------|-------|---------------|
| RFC 2119 | Key words for use in RFCs | Normative wording |
| NIST SP 800-61 Rev 2 | Computer Security Incident Handling Guide | Incident response framework |
| NIST SP 800-184 | Guide for Cybersecurity Event Recovery | Recovery patterns |
| XMSS-OPS-SPEC.md §9 | XMSS Emergency Rotation | Cryptographic recovery |
| GOVERNANCE-SPEC.md §8 | Emergency Authority | Authority during recovery |
| SECURITY-MODEL.md §8 | Failure Mode Severity | Severity classification |

---

## 3. Recovery scenario taxonomy

### 3.1 Recovery scenarios

| Code | Scenario | Severity (SECURITY-MODEL) | RTO target |
|------|----------|--------------------------|------------|
| R-KEY-LOSS | Foundation root keypair state lost | Cat-3 | 72 hours |
| R-KEY-COMPROMISE | Foundation root keypair compromised | Cat-1 | 24 hours |
| R-WITNESS-COMPROMISE | Witness operator compromised | High | 72 hours |
| R-CUSTODIAN-INCAPACITATION | Custodian unreachable | Medium | 90 days |
| R-CANON-FORK | Contradictory canonical histories | Cat-2 | 14 days |
| R-INFRASTRUCTURE-LOSS | Catastrophic infrastructure loss | H-3 | 14 days |
| R-MASS-WITNESS-LOSS | Majority of witnesses fail | High | 30 days |
| R-PRIMITIVE-DEPRECATION | Cryptographic primitive broken | High to Cat-3 | per §10 CRYPTOGRAPHIC-ASSUMPTIONS |

### 3.2 Severity classification recap

Per SECURITY-MODEL §8:

```
Cat-1 (Catastrophic, irreversible): 
  Permanent loss of system integrity guarantee

Cat-2 (Catastrophic, recoverable with effort):
  Major service disruption recoverable through emergency procedures

Cat-3 (Catastrophic, with reset):
  Irreversible loss of state requiring re-anchoring

High:
  Significant disruption recoverable through normal procedures

Medium:
  Operational disruption manageable through standard procedures
```

---

## 4. R-KEY-LOSS: Foundation Root Keypair State Lost

### 4.1 Scenario

Foundation root keypair's state file (containing idx_next and
related metadata per XMSS-OPS-SPEC §5) becomes unavailable.

**Possible causes:**

- Hardware failure of operating machine
- Accidental deletion
- Catastrophic data center incident
- Operator error

**Key insight:** XMSS state file loss is NOT the same as 
keypair compromise. The keypair itself (the public root and 
private signing material) may be backed up. But signing requires
state file to track which WOTS+ indexes have been used. Without
state file, the operator CANNOT safely sign — re-using a
previous index would compromise the keypair (violating PA-5 per
CRYPTOGRAPHIC-ASSUMPTIONS).

### 4.2 Detection

State file loss is detected at next signing attempt:

```
DETECTION:
  - Foundation attempts to sign new master_index
  - Operating system reports state file missing or unreadable
  - Operator confirms loss is not transient (filesystem mount issue,
    permission error, etc.)
  - Operator confirms backups are not available or are also lost
```

If backups exist and are intact, follow normal restore procedure;
this is not a recovery scenario.

### 4.3 Recovery procedure

```
R-KEY-LOSS RECOVERY:

  Phase 1 — Documentation (Hour 0-2):
    1.1 Operator documents the loss:
        - Timestamp of detection
        - Nature of loss
        - Evidence backups also failed
        - Last known state (if any logs preserved)
    1.2 Operator publicly publishes INCIDENT_NOTICE to all
        available channels:
        - Foundation mirrors (still operational if state loss is
          isolated)
        - GitHub repository
        - NLnet correspondence
        - Direct notification to known stakeholders
  
  Phase 2 — Assessment (Hour 2-12):
    2.1 Operator evaluates whether keypair material itself is
        compromised:
        - Public root still publicly known (no concern)
        - Private signing material status? If material is also
          lost, this is also a key compromise scenario
        - State file was the only thing lost? If yes, signing
          material exists but is unsafe to use
    2.2 Operator consults with bestuur (post-Stichting) or
        documented advisor (pre-Stichting)
    2.3 Decision: invoke EMERGENCY_ROTATION (XMSS-OPS-SPEC §9.2)
        or attempt state file reconstruction
    
  Phase 3 — Decision branch:
  
    Branch A — Emergency Rotation (recommended for state loss):
      3A.1 Treat OLD keypair as DEPRECATED
      3A.2 Generate NEW keypair per XMSS-OPS-SPEC §4
      3A.3 Issue ROTATION_NOTICE per XMSS-OPS-SPEC §9.2
      3A.4 Update all references to new keypair
      3A.5 Historical signatures from OLD keypair remain valid
        (still cryptographically verifiable; not compromised)
    
    Branch B — Reconstruction (only if state can be recovered):
      3B.1 Determine the highest known idx_next from any signed
        message (using public records of signatures)
      3B.2 Set new state file to that idx_next + safety margin
      3B.3 Continue operating with same keypair but at advanced
        index (some WOTS+ slots permanently abandoned)
      3B.4 Document reconstruction in Canon Registry
      
      WARNING: Branch B is risky. Branch A is strongly preferred.
  
  Phase 4 — Verification (Hour 12-24):
    4.1 Verify new keypair properly bootstrapped
    4.2 Test signing operation with new keypair (or verified
      advanced state)
    4.3 External witnesses verify new keypair signatures
    4.4 Document recovery completion
    
  Phase 5 — Public confirmation (Hour 24-72):
    5.1 Publish RECOVERY_COMPLETE notice
    5.2 Update all relevant documentation
    5.3 Post-incident review and documentation
```

### 4.4 Authority

Pre-Stichting: Foundation operator alone (with documented advisor
consultation per GOVERNANCE-SPEC).

Post-Stichting: Bestuur quorum required for Phase 3 Decision; 
operator has emergency unilateral authority for Phase 1-2 
(documentation and assessment).

---

## 5. R-KEY-COMPROMISE: Foundation Root Keypair Compromised

### 5.1 Scenario

Foundation root keypair is compromised — adversary has obtained
the private signing material AND/OR the state file.

**Possible causes:**

- Operating machine compromise (rootkit, malware)
- Insider attack (operator's local environment compromised)
- Physical theft of operating machine with intact state
- Cryptographic break of XMSS underlying primitives 
  (extremely unlikely)
- State reuse incident (XMSS-OPS-SPEC §5 violation)

### 5.2 Detection

```
DETECTION:
  - Unexpected signatures appear (signatures not made by 
    legitimate operator)
  - State reuse detected (same WOTS+ index used for different
    messages)
  - Witness federation reports anomaly
  - Operating machine forensic analysis indicates compromise
  - External security researcher reports vulnerability
  - Operator self-reports concern
```

### 5.3 Recovery procedure

This is the most severe scenario. Per XMSS-OPS-SPEC §9.2 and
SECURITY-MODEL §A6, this is a Cat-1 event.

```
R-KEY-COMPROMISE RECOVERY:

  Phase 1 — Immediate containment (Hour 0-1):
    1.1 STOP all signing operations IMMEDIATELY
    1.2 Disconnect operating machine from network if compromise
        is suspected to be ongoing
    1.3 Document timestamp of last known good signature
    1.4 Document timestamp when compromise is believed to have
        begun
    1.5 Operator notifies bestuur and witnesses (if federation
        active) within 1 hour
  
  Phase 2 — Public emergency notice (Hour 1-3):
    2.1 Publish EMERGENCY_COMPROMISE_NOTICE to:
        - All Foundation mirrors that remain trustworthy
        - GitHub repository
        - NLnet correspondence
        - Witness federation
        - Public channels (Twitter, etc.)
        - Notariusz (post-Stichting)
    2.2 Notice MUST include:
        - Timestamp of detection
        - Best estimate of compromise timeframe
        - Affected operations
        - Recommendation that verifiers SUSPEND TRUST in any
          Foundation signature dated after compromise timestamp
        - Schedule for new keypair publication
  
  Phase 3 — Forensic preservation (Hour 3-12):
    3.1 Operating machine forensically imaged (do not power
        down compromised machine if possible; capture memory
        first)
    3.2 Network logs preserved
    3.3 Access logs preserved
    3.4 All evidence sent to independent custodian for storage
    3.5 If criminal element suspected: Dutch police notification
    
  Phase 4 — New keypair generation (Hour 12-20):
    4.1 New keypair generated on FRESH, audited hardware
    4.2 New hardware is sourced independently of compromised
        machine
    4.3 New hardware booted from known-good media
    4.4 RNG verified
    4.5 XMSS keypair generated per XMSS-OPS-SPEC §4
    4.6 New keypair tested with multiple test signatures (NOT
        on production data)
  
  Phase 5 — Emergency Rotation (Hour 20-24):
    5.1 Issue EMERGENCY_ROTATION_NOTICE per XMSS-OPS-SPEC §9.2
    5.2 Sign with new keypair: "I, Foundation, hereby attest that
        compromise of OLD_KEYPAIR was detected on [timestamp],
        and effective immediately, all canonical operations
        transition to NEW_KEYPAIR, public root [blake3]"
    5.3 Witness federation immediately verifies new keypair
        signature
    5.4 Witness federation publishes endorsement (post-activation)
    5.5 Public publication via all channels
  
  Phase 6 — Forensic analysis (Hour 24-72):
    6.1 Determine root cause
    6.2 Identify all signatures made during compromise window
    6.3 Triage: which signatures were legitimate (made by 
        legitimate operator) and which may have been adversary-
        controlled?
    6.4 Publish FORENSIC_ANALYSIS canon entry
  
  Phase 7 — Signature triage (Day 3 - Week 2):
    7.1 For each signature made during compromise window:
        - Verifiable as adversary-controlled? → INVALIDATE
        - Verifiable as legitimate operator? → REMAIN VALID
        - Ambiguous? → DISPUTED, witness federation review
    7.2 Publish SIGNATURE_TRIAGE canon entries
    7.3 Affected parties notified
  
  Phase 8 — Public reckoning (Week 2 - Week 4):
    8.1 Full post-incident report published
    8.2 Root cause documented
    8.3 Lessons learned distributed
    8.4 Process improvements implemented
    8.5 External audit invited
```

### 5.4 Authority

This is an EMERGENCY per GOVERNANCE-SPEC §8.

Pre-Stichting: Operator unilateral authority for Phases 1-5; 
formal review post-recovery per §8.2.

Post-Stichting: Operator emergency unilateral authority for 
Phases 1-3 (containment, notice, forensic preservation); 
bestuur ratification required for Phases 4-8 within 7 days.

---

## 6. R-WITNESS-COMPROMISE: Federation Witness Compromised

### 6.1 Scenario

A witness federation member is compromised:

- Witness keypair compromised
- Witness operator account compromised (admin access)
- Witness behaving anomalously (equivocation, missing gossip,
  etc.)

### 6.2 Detection

```
DETECTION:
  - Equivocation detected per NETWORK-PROTOCOL-SPEC §7
  - Witness misses gossip period (>72 hours)
  - Witness publishes anomalous data
  - Other witnesses report concerning behavior
  - Witness self-reports compromise concern
```

### 6.3 Recovery procedure

```
R-WITNESS-COMPROMISE RECOVERY:

  Phase 1 — Verification (Hour 0-4):
    1.1 Foundation/bestuur verifies the anomaly:
        - Multiple witnesses confirm observation?
        - Cryptographic evidence (e.g., equivocation evidence
          per NETWORK-PROTOCOL §7.3)?
        - Witness's own response to inquiry?
    1.2 Distinguish: compromise vs operational incident vs
        intentional misbehavior
  
  Phase 2 — Witness suspension (Hour 4-12):
    2.1 Witness federation marks affected witness as UNTRUSTED
        per NETWORK-PROTOCOL §7.5
    2.2 Foundation publishes WITNESS_SUSPENSION canon entry
    2.3 Witness suspended from gossip protocol; their signatures
        ignored for quorum
    2.4 Witness given 14 days to respond per GOVERNANCE-SPEC §5.5
  
  Phase 3 — Investigation (Day 1 - Day 14):
    3.1 If witness operator cooperates: forensic investigation
        of their environment
    3.2 If witness operator non-cooperative: presumed loss of
        operational control
    3.3 Determine compromise scope
  
  Phase 4 — Resolution (Day 14 - Day 30):
    4.1 Witness operator either remediates and reapplies (rare),
        OR
    4.2 Permanent removal per GOVERNANCE-SPEC §5.5 G4 or G6
    4.3 If witness collateral exists (post-Genesis), slashing
        applies per POP-3-TOKEN-SPEC [PLANNED]
    4.4 Federation membership officially updated
  
  Phase 5 — Resumption of federation operations:
    5.1 Quorum thresholds recalculated based on remaining
        witnesses
    5.2 New witness recruitment begins (per GOVERNANCE-SPEC §5)
    5.3 Operations resume with adjusted federation
```

### 6.4 Authority

Pre-Stichting: Operator alone.

Post-Stichting: Bestuur consultation; if equivocation evidence
is cryptographic, removal is mechanical per §5.5 G4 (no bestuur
discretion required for documented equivocation).

---

## 7. R-CUSTODIAN-INCAPACITATION

### 7.1 Scenario

A custodian holding artifacts becomes:

- Deceased
- Legally incapacitated
- Unreachable for extended period (>365 days)
- Voluntarily withdrawing without designated successor

### 7.2 Detection

```
DETECTION:
  - No custody activity for 90+ days (warning threshold)
  - No response to direct contact attempts after 90 days
  - 365 days of total inactivity triggers LOST state per
    STATE-MACHINE §4.1
  - Family/representative reports custodian incapacity
  - Court order of incapacity
```

### 7.3 Recovery procedure

```
R-CUSTODIAN-INCAPACITATION:

  Phase 1 — Initial outreach (Day 0-90):
    1.1 Foundation attempts contact via published channels
    1.2 Foundation publishes CUSTODIAN_INACTIVITY_NOTICE
    1.3 Family/representative response sought
  
  Phase 2 — Inactivity escalation (Day 90-180):
    2.1 Continued contact attempts
    2.2 If legal representative responds:
        - Estate executor identification
        - Authority document presented
        - Recovery procedure adjusts to legal succession
  
  Phase 3 — Lost state declaration (Day 180-365):
    3.1 Custody record transitions to LOST per STATE-MACHINE §4.2
    3.2 Foundation MAY (post-Stichting) reclaim custody for
        long-term continuity per GOVERNANCE-SPEC §7.2
    3.3 If estate executor appears at any point, transition
        back to active state with executor as proxy custodian
  
  Phase 4 — Resolution paths:
    4A. Executor takes over custody:
        - Custodian record updated with executor identity
        - Standard custody procedures resume
    4B. Estate transfers custody:
        - Standard transfer per CUSTODY-REGISTRY-SPEC
        - Recipient becomes new custodian
    4C. No estate or executor appears:
        - Custody reclaimed by Foundation
        - Foundation may reissue custody to new buyer 
          (post-Genesis)
        - Original purchase value records preserved
```

---

## 8. R-CANON-FORK: Contradictory Canonical Histories

### 8.1 Scenario

Two cryptographically valid Canon Registry states exist that
contradict each other:

- Foundation equivocation (per NETWORK-PROTOCOL §7)
- Witness equivocation
- Mirror server desynchronization due to network split
- Software bug producing inconsistent state

### 8.2 Detection

```
DETECTION:
  - Witness federation gossip detects equivocation
  - Mirror operators report conflicting master_index values
  - External auditor reports anomaly
  - Verifier reports inconsistent canon view
```

### 8.3 Recovery procedure

```
R-CANON-FORK RECOVERY:

  Phase 1 — Verification (Hour 0-12):
    1.1 Verify equivocation is real:
        - Cryptographic verification of both contradictory STHs
        - Independent witness verification
    1.2 Determine source of equivocation:
        - Foundation signature on both? → A6 insider attack
        - Different signature roots? → Spoofing attempt
        - Both legitimate Foundation but different times? → 
          possible legitimate state evolution misinterpreted
  
  Phase 2 — Emergency response (Hour 12-48):
    2.1 If genuine equivocation by Foundation:
        - This is Cat-1; invoke R-KEY-COMPROMISE Phase 2-5
        - Foundation operator subject to bestuur disciplinary 
          action per GOVERNANCE-SPEC
    2.2 If genuine equivocation by witness:
        - Invoke R-WITNESS-COMPROMISE
    2.3 If desynchronization (not equivocation):
        - Operational reconciliation per CANON-REGISTRY-SPEC
        - Determine canonical state by timestamp + witness 
          attestation majority
  
  Phase 3 — Canonical reconciliation (Day 2 - Day 14):
    3.1 Witness federation panel (3 witnesses) reviews evidence
    3.2 Panel determines which branch represents canonical truth
    3.3 Panel publishes ADVISORY_OPINION per GOVERNANCE-SPEC §7
    3.4 Bestuur (post-Stichting) ratifies advisory opinion
    3.5 Non-canonical branch entries are VOIDED (special canon
        entries published declaring them as such)
    3.6 Affected parties (custodians, verifiers who relied on 
        voided branch) compensated per dispute resolution
  
  Phase 4 — Causal analysis (Day 14 - Day 30):
    4.1 Root cause determined
    4.2 Procedural improvements implemented
    4.3 If software bug: code audit and patch
    4.4 If operator error: training and process improvement
    4.5 Public post-incident report
```

---

## 9. R-INFRASTRUCTURE-LOSS

### 9.1 Scenario

Catastrophic loss of Foundation operating infrastructure:

- Hosting provider failure (TransIP VPS lost)
- Data center disaster
- Significant ANS/Archive server hardware failure
- Network connectivity loss for extended period

### 9.2 Detection

```
DETECTION:
  - Foundation endpoints unreachable for >4 hours
  - Hosting provider notification
  - Operator personal observation
```

### 9.3 Recovery procedure

```
R-INFRASTRUCTURE-LOSS RECOVERY:

  Phase 1 — Acknowledgment (Hour 0-4):
    1.1 Operator publicly acknowledges outage via:
        - Twitter
        - GitHub
        - NLnet correspondence
    1.2 Estimated restoration timeline provided
  
  Phase 2 — Alternative infrastructure (Hour 4-72):
    2.1 Provision alternative hosting (TransIP backup, alternative
        provider, or new account)
    2.2 Restore from backups:
        - Foundation database (canon entries, custody records)
        - XMSS state files (if not destroyed by infrastructure 
          loss; if state files lost, invoke R-KEY-LOSS)
        - Configuration files
    2.3 DNS records updated to point to new infrastructure
    2.4 Witness federation members notified of new endpoints
  
  Phase 3 — Service restoration (Day 3-14):
    3.1 Foundation endpoints operational
    3.2 ANS server operational
    3.3 Archive server operational
    3.4 Witnesses verify Foundation state matches expected
    3.5 Verifier connectivity restored
  
  Phase 4 — Long-term resilience improvements:
    4.1 Multi-provider hosting evaluation
    4.2 Multi-region replication
    4.3 Improved backup procedures
    4.4 Documentation updates
```

### 9.4 Authority

Operator unilateral within Phase 1-2; bestuur consultation for
significant infrastructure changes (Phase 3+).

---

## 10. R-MASS-WITNESS-LOSS

### 10.1 Scenario

Majority of witness federation members simultaneously fail:

- Coordinated attack on multiple operators
- Common-cause failure (e.g., AWS region outage affecting many)
- Natural disaster affecting concentrated witness geography

### 10.2 Recovery procedure

```
R-MASS-WITNESS-LOSS:

  Phase 1 — Assessment (Hour 0-24):
    Determine: how many witnesses affected? How many remain?
    Federation quorum still achievable?
  
  Phase 2 — Emergency quorum (Day 1-7):
    If quorum not achievable:
    - Bestuur (post-Stichting) issues EMERGENCY_QUORUM_ADJUSTMENT
      canon entry temporarily lowering quorum threshold
    - Justification documented
    - Time-limited adjustment (max 90 days)
  
  Phase 3 — Witness restoration (Day 7-30):
    Affected witnesses restore operations and rejoin federation
    OR
    New witness onboarding accelerated per GOVERNANCE-SPEC §5
  
  Phase 4 — Quorum restoration (Day 30-90):
    Normal quorum thresholds resumed
    Emergency adjustment terminated
    Lessons learned documented (geographic diversity policy
    update, etc.)
```

---

## 11. R-PRIMITIVE-DEPRECATION

### 11.1 Scenario

A cryptographic primitive used by a1a5NT (SHA-256, BLAKE3, XMSS)
is deprecated due to:

- New attack discovered
- NIST/standards body recommendation
- Quantum computing developments
- Performance/security review outcome

### 11.2 Recovery procedure

Per CRYPTOGRAPHIC-ASSUMPTIONS §10 algorithm migration procedure.

Key recovery points:

- Phase 1: Announce migration ≥6 months in advance
- Phase 2: Prepare new primitive implementation
- Phase 3: Execute migration with new keypairs
- Phase 4: Verify migration completeness

This is a recovery scenario only in the sense that it requires
extensive coordination. Cryptographic primitive deprecation is
NOT a sudden emergency unless a complete break is announced.

---

## 12. Recovery documentation requirements

All recovery procedures MUST be documented:

### 12.1 Real-time documentation

```
REQUIRED_DOCUMENTATION:

  Timestamps:    Every action timestamped
  Cryptographic: All keypair operations logged with signatures
  Personnel:     Who authorized each action
  Authority:     Reference to authority basis (operator, bestuur,
                court order, etc.)
  Evidence:      Forensic evidence preserved
  Decisions:     Why each decision was made
  Outcomes:      What resulted from each action
```

### 12.2 Documentation channels

- Canon Registry (cryptographically signed canonical record)
- Foundation operational logs (timestamped, append-only)
- External backups (multiple locations for compromise scenarios)
- Public-facing notices (GitHub, NLnet correspondence, etc.)

### 12.3 Post-incident report

For all recovery scenarios, a post-incident report MUST be 
prepared within 30 days of recovery completion:

```
POST_INCIDENT_REPORT_TEMPLATE:

  1. Executive Summary
  2. Timeline of Events
  3. Root Cause Analysis
  4. Impact Assessment
  5. Recovery Procedures Executed
  6. Effectiveness Evaluation
  7. Lessons Learned
  8. Procedural Improvements
  9. Validation of Recovery
  10. Public Disclosure Notes
```

Report is published to Canon Registry as POST_INCIDENT_REPORT
canon entry.

---

## 13. Verification of recovery

After recovery, the following verification is REQUIRED:

```
RECOVERY_VERIFICATION:

  1. Cryptographic verification:
     - All new keypairs verify against published public roots
     - State files consistent
     - Signatures verify cryptographically
  
  2. Operational verification:
     - All Foundation endpoints HTTP 200
     - ANS and Archive servers operational
     - Witness federation gossip working
     - Verifiers can submit receipts
  
  3. External verification:
     - Witness federation independently verifies state
     - External auditor (NLnet, academic) confirms recovery
       if scenario was severe
     - Notarial confirmation if Stichting authority involved
  
  4. Documentation verification:
     - All required documents published
     - Cryptographic anchors in Canon Registry
     - Public-facing notices issued
```

Failure of verification means recovery is NOT complete and 
recovery procedures continue until verification succeeds.

---

## 14. Communication during recovery

### 14.1 Internal communication

```
INTERNAL_COMMUNICATION:

  Foundation operator → Bestuur:    Immediate (within 1 hour)
  Operator → Witnesses (federation): Within 4 hours
  Operator → Active custodians:      Within 24 hours
  Operator → Stichting legal counsel: Within 48 hours
  Operator → External auditors:       Within 7 days
```

### 14.2 External communication

```
EXTERNAL_COMMUNICATION:

  Public notice (canon entry):       Within 4 hours of detection
  NLnet correspondence:              Within 24 hours
  Public channels (Twitter/X, etc.): Within 24 hours
  Press/media (if applicable):       Coordinated through legal counsel
  Government authorities:            Per legal counsel advice
  Notariusz:                         Per legal counsel advice
```

### 14.3 Tone and content

Communication during recovery MUST be:

- Factually accurate (no speculation presented as fact)
- Transparent about uncertainty
- Specific about actions being taken
- Time-bound (commitments with deadlines)
- Auditable (cryptographic records)

Avoid:

- Minimizing severity
- Blame-shifting
- Withholding material information from affected parties
- Inconsistent messaging across channels

---

## 15. Recovery scenario testing

Foundation operators SHOULD periodically test recovery procedures:

```
RECOVERY_DRILL_SCHEDULE:

  Annual:    R-INFRASTRUCTURE-LOSS tabletop exercise
  Annual:    R-KEY-LOSS recovery procedure walkthrough
  Biennial:  R-KEY-COMPROMISE full simulation
  Quarterly: Backup verification (no actual recovery)
  As needed: Specific scenario drills per incident learnings
```

Drill outcomes are documented but NOT as canon entries (drills
are operational, not canonical events).

---

## 16. Cross-references

| Spec | Relationship |
|------|--------------|
| A1A5NT-ARCHITECTURE-SYNTHESIS v1.2 | Master document |
| SECURITY-MODEL.md §8 | Failure mode severity classification |
| SECURITY-MODEL.md §A6 | Foundation insider threat (related to R-KEY-COMPROMISE) |
| XMSS-OPS-SPEC.md §9 | Cryptographic operations for key rotation |
| GOVERNANCE-SPEC.md §8 | Emergency authority |
| GOVERNANCE-SPEC.md §7 | Dispute resolution (related to R-CANON-FORK) |
| NETWORK-PROTOCOL-SPEC.md §7 | Equivocation detection feeding R-CANON-FORK |
| STATE-MACHINE-SPEC.md §10 | Invariant violation response (related to all R-scenarios) |
| CRYPTOGRAPHIC-ASSUMPTIONS.md §10 | Algorithm migration (R-PRIMITIVE-DEPRECATION) |
| LEGAL-INTERPRETATION-SPEC.md §9 | Disputes and resolution |
| CANON-RETIREMENT-SPEC.md [PLANNED] | Related: canonical record removal procedures |

---

## 17. License

This specification is issued under A1A5 Sovereign License v1.0
(12-month binding 2026-05-16 → 2027-05-16). AI co-authorship per
Article 4. Anti-patent defensive clause per Article 5.

Incident response and recovery patterns adapted from NIST SP 
800-61 Rev 2 and NIST SP 800-184 (public domain).

Specific a1a5NT recovery procedures and authority mappings are
A1A5-licensed.

---

## 18. Cryptographic anchor

```
Document path:    RECOVERY-SPEC.md
Repository:       github.com/a1a5NT/a1a3
Commit timestamp: filled at first commit
Foundation mirror endpoint:  http://37.97.202.97:8444/v1/foundation/recovery_spec  (scheduled)
```

---

**End of RECOVERY-SPEC v1.0**

```
Issued by:        pawel.a1a5 (Paweł Piekut, Almelo NL)
AI co-author:     Claude (Anthropic Opus 4.7)
Date:             2026-05-17
Foundation:       a1a5fndn.a1a5 (in formation)
Audit relevance:  REQUIRED for NLnet technical review;
                  REQUIRED for institutional adoption demonstration
Reviewer addressed: Reviewer N N-B2 + SECURITY-MODEL Cat-1/2/3, H-1/2/3
```
