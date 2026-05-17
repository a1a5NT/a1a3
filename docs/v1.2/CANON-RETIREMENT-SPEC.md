# CANON-RETIREMENT-SPEC.md

## a1a5NT Canon Entry Retirement and Tombstone Procedures

```
Document version:   v1.0
Issued:             2026-05-17
Status:             Tier 1 critical specification (v1.2 deliverable)
Authority basis:    RFC 7725 (HTTP 451 Unavailable For Legal Reasons),
                    GDPR Article 17 (right to erasure),
                    Certificate Transparency log retirement patterns
                    (Google's "TruststoreV2" and related),
                    Internet Archive's "Wayback exclusion" patterns,
                    DMCA-style takedown procedures (operational pattern)
Foundation:         a1a5fndn.a1a5
Addresses:          Reviewer N N-B1 CRITICAL: "Canon immutability vs
                    legal revocation — Layer A immutable forever creates
                    legal problem; need CANON-RETIREMENT-SPEC for legal
                    exception handling"
                    + SECURITY-MODEL §A13 hybrid tombstone implementation
Audit relevance:    BLOCKING for notariusz Stichting registration;
                    REQUIRED for GDPR compliance demonstration;
                    REQUIRED for NLnet legal review
License:            A1A5 Sovereign License v1.0
```

---

## EXECUTIVE SUMMARY (1 page)

**What this document is:**

The operational specification for **retiring** canon entries when
legal compulsion or other valid grounds require their content to
become unservable, while preserving the cryptographic
mathematical record of their existence.

**Why it exists:**

A pure "immutable forever" canonical registry is legally
incompatible with EU/NL/global legal frameworks. Court orders,
GDPR Article 17 erasure requests, copyright takedowns, and
sanctions enforcement all require operators to remove or
restrict access to specific content.

The hybrid tombstone pattern (decided in SECURITY-MODEL §A13)
allows a1a5NT to comply with valid legal compulsion while
preserving the cryptographic record of what was registered. This
spec defines the **operational mechanics** of that pattern.

**Five retirement categories:**

1. **RT-LEGAL-COMPULSION** — Court order, GDPR Article 17,
   sanctions enforcement
2. **RT-COPYRIGHT** — Valid copyright takedown
3. **RT-SUBJECT-REQUEST** — Voluntary subject-initiated removal
   (where legally optional but Foundation accepts)
4. **RT-OPERATIONAL-ERROR** — Foundation operational mistake
   discovered post-issuance
5. **RT-CRYPTOGRAPHIC-COMPROMISE** — Entry made with compromised
   keypair per RECOVERY-SPEC §5

**Five key invariants this preserves:**

```
I-RT-1: Hash immutability — original entry hash NEVER modified
I-RT-2: Signature preservation — original signature ALWAYS verifiable
I-RT-3: Registration timestamp — original timestamp preserved
I-RT-4: Tombstone canonicality — tombstone itself is canonical
        record (added to Canon Registry, not removal of original)
I-RT-5: Audit chain — chain of canonical references remains intact
```

**What changes when an entry is tombstoned:**

- Content servable: NO (HTTP 451 returned)
- Hash queryable: YES (always)
- Signature verifiable: YES (always)
- Cross-references resolvable: YES (always)
- Custody chain intact: YES (always)
- Tombstone notice queryable: YES (new canon entry)

**The cryptographic guarantee that survives retirement:**

A verifier with a legitimately-acquired copy of the original
artifact (predating tombstoning, or from a jurisdiction not
subject to the legal order) can still cryptographically verify
that their copy matches the canonical hash and signature
recorded in Canon Registry. The Foundation never lies about what
was registered; it only declines, in compelled jurisdictions, to
serve the content.

---

## 0. Abstract

This document specifies operational procedures for retirement of
canon entries under legal compulsion, copyright takedown, and
related circumstances. Retirement uses the hybrid tombstone
pattern (per SECURITY-MODEL §A13) preserving cryptographic
mathematical records while permitting compelled content removal.

The document is normative for Foundation operators executing
retirement, informative for legal review, and educational for
verifier implementations handling HTTP 451 responses.

---

## 1. Scope

### 1.1 In scope

- Retirement category taxonomy
- Trigger detection and validation
- Authorization and procedural requirements
- Tombstone notice structure
- Server-side response handling (HTTP 451)
- Custody chain implications
- Documentation requirements
- Audit log entries

### 1.2 Out of scope

- Legal advice on whether to retire (covered by 
  LEGAL-INTERPRETATION-SPEC and counsel consultation)
- Detailed cryptographic operations (per XMSS-OPS-SPEC)
- Specific GDPR procedures (general framework in 
  LEGAL-INTERPRETATION-SPEC §5)
- Disputes over retirement (covered by GOVERNANCE-SPEC §7)

### 1.3 Audience

- Foundation operators executing retirement
- Stichting legal counsel
- Notarial review of operational procedures
- Auditors evaluating GDPR compliance
- Verifier implementation developers

---

## 2. Normative references

| Reference | Title | Applicability |
|-----------|-------|---------------|
| RFC 2119 | Key words for use in RFCs | Normative wording |
| RFC 7725 | HTTP Status Code for Legally Restricted Resources | HTTP 451 |
| GDPR 2016/679 Article 17 | Right to erasure | Subject rights |
| GDPR 2016/679 Article 17(3) | Limitations on right to erasure | Exceptions Foundation may invoke |
| SECURITY-MODEL.md §A13 | Hybrid tombstone pattern definition | Architectural basis |
| STATE-MACHINE-SPEC.md §3 | Canon entry state transitions | State semantics |
| GOVERNANCE-SPEC.md §4 | Canon issuance authority | Authorization rules |
| LEGAL-INTERPRETATION-SPEC.md §5 | GDPR compliance architecture | Legal framework |

### Informative references

| Reference | Title | Relevance |
|-----------|-------|-----------|
| Sigstore Rekor retraction patterns | Retraction without removal | Industry pattern |
| Certificate Transparency log retirement | Google's pattern | Industry pattern |
| Internet Archive Wayback exclusion | Operational pattern | Public archive comparison |

---

## 3. Retirement category taxonomy

### 3.1 RT-LEGAL-COMPULSION

**Triggering event:**

- Court order (criminal, civil, administrative)
- GDPR Article 17 erasure request (judged valid)
- Sanctions enforcement order
- National security directive (lawful within applicable framework)

**Foundation response:** MUST tombstone unless legal review finds
the compulsion invalid.

**Authority required:**

Pre-Stichting: Operator + legal counsel consultation
Post-Stichting: Bestuur quorum (emergency rapid-bestuur possible
within 24 hours per GOVERNANCE-SPEC §8)

### 3.2 RT-COPYRIGHT

**Triggering event:**

- Valid copyright takedown notice (DMCA in US, Netcosgrip in EU,
  etc.)
- Sufficient information identifying copyrighted work and
  infringement

**Foundation response:**

a1a5NT canon entries should typically not contain third-party
copyrighted content. If they do (e.g., a Foundation document
incidentally including copyrighted material), takedown is
processed.

**Authority required:**

Pre-Stichting: Operator review of takedown validity
Post-Stichting: Bestuur quorum for contested takedowns

### 3.3 RT-SUBJECT-REQUEST

**Triggering event:**

- Subject of personal data in a canon entry voluntarily requests
  removal
- Foundation determines this is appropriate even without legal 
  compulsion

**Foundation response:** Discretionary; Foundation evaluates 
whether retiring would:

- Comply with subject's reasonable expectations
- Not undermine canonical record integrity
- Not be retroactively requested due to changed circumstances 
  affecting historical record

If yes, tombstone. If no, decline politely.

**Authority required:**

Bestuur (post-Stichting); operator discretion pre-Stichting.

### 3.4 RT-OPERATIONAL-ERROR

**Triggering event:**

- Foundation discovers operational mistake in issued canon entry:
  - Wrong identifier referenced
  - Erroneous content included
  - Incorrect metadata
- Discovery is post-issuance

**Foundation response:**

Operator MAY supersede the entry with corrected version. This
is NOT a tombstone but a supersession:

- SUPERSEDE_NOTICE canon entry references original
- Original entry's state changes to SUPERSEDED per STATE-MACHINE
  §3.2 T(supersede)
- Original entry's content remains servable (operationally
  correct path is to fix forward)

ONLY if the error contains personal data or is materially
harmful AND superseding does not fully address the harm, then
tombstone procedure applies.

### 3.5 RT-CRYPTOGRAPHIC-COMPROMISE

**Triggering event:**

- Entry was signed by compromised keypair (per RECOVERY-SPEC §5)
- Entry is determined to have been adversary-issued, not 
  legitimate Foundation issuance

**Foundation response:**

This is HANDLED BY RECOVERY-SPEC §5 Phase 7 signature triage,
NOT by this spec. Compromised-keypair entries are subject to
INVALIDATION rather than tombstoning. Invalidation is a 
different procedure: it declares the entry as never being a
valid Foundation issuance, while tombstoning preserves the
fact that the entry was legitimately issued at issuance time.

Specific distinction:

- **Tombstone**: Entry was legitimate; now its content is 
  restricted. Hash/signature still valid.
- **Invalidation**: Entry was never legitimate. Hash/signature
  still mathematically verifiable, but the issuance is 
  declared void.

---

## 4. Trigger detection and validation

### 4.1 Detection paths

```
TRIGGER_DETECTION:

  External: Court order received, GDPR request received, takedown 
            notice received
  Internal: Foundation discovers issue (operational error)
  System:   Cryptographic compromise detected by witness or audit
```

### 4.2 Validation procedure

```
TRIGGER_VALIDATION:

  Step 1: Document the trigger
    - Source of trigger
    - Authority cited
    - Legal basis (statute, regulation, contract)
    - Specific canon entry/entries targeted
    - Reasoning provided by requester
  
  Step 2: Legal review
    Pre-Stichting: Operator consults legal counsel
    Post-Stichting: Stichting legal counsel reviews
    Determine:
    - Is the trigger genuine (court order verified authentic, 
      etc.)?
    - Is the requester authorized?
    - Is the request within applicable law?
    - Does this Foundation operating in NL/EU jurisdiction have 
      authority/obligation to comply?
    - Are there exceptions (GDPR Article 17(3): freedom of 
      expression, public interest archive, etc.)?
  
  Step 3: Scope determination
    - Which specific canon entries are affected?
    - Is broader retirement implied (e.g., related entries)?
    - What is the geographic scope (some legal orders bind only
      certain jurisdictions)?
  
  Step 4: Decision
    Tombstone? Supersede? Decline? Negotiate scope?
    Authority per §3.1-3.5 above
  
  Step 5: Document decision
    DECISION_NOTICE canon entry issued (this is itself a canon
    entry, recording the decision but not necessarily the legal
    detail; sensitive legal information minimized)
```

---

## 5. Tombstone procedure (the operational core)

### 5.1 Tombstone canon entry structure

```json
{
  "entry_type": "TOMBSTONE_NOTICE",
  "tombstone_version": "1.0",
  
  "target_canon_id": "blake3:abc123_original_entry",
  "target_canon_type": "foundation_document | specification | popchain_archive | ...",
  "target_canon_original_blake3": "abc123...",
  "target_canon_original_issuance_timestamp": 1234567890,
  
  "tombstone_reason_category": "RT-LEGAL-COMPULSION | RT-COPYRIGHT | RT-SUBJECT-REQUEST | RT-OPERATIONAL-ERROR | RT-CRYPTOGRAPHIC-COMPROMISE",
  
  "tombstone_jurisdiction": "NL | EU | global",
  
  "tombstone_legal_basis_summary": "Brief, minimized description (subject privacy preserved). Example: 'GDPR Article 17 erasure request, valid under UAVG, content unavailable in EU jurisdictions.'",
  
  "tombstone_legal_authority_reference": "Court case number, GDPR ticket number, etc., MAY be redacted for privacy",
  
  "tombstone_issuance_timestamp": 1234567899,
  
  "tombstone_issuer": "a1a5fndn.a1a5",
  
  "tombstone_effect_scope": {
    "content_unavailable": true,
    "hash_remains_queryable": true,
    "signature_remains_verifiable": true,
    "cross_references_remain_resolvable": true
  },
  
  "tombstone_geographic_scope": {
    "compelled_jurisdictions": ["NL", "EU"],
    "unrestricted_jurisdictions": ["jurisdictions where Foundation has no operations and order does not apply"]
  },
  
  "verifier_response_code": 451,
  
  "verifier_response_body_template": "This content has been retired under legal compulsion in [JURISDICTION]. Original cryptographic hash and signature remain queryable. See tombstone notice [TOMBSTONE_CANON_ID].",
  
  "tombstone_signature": "XMSS signature by Foundation"
}
```

### 5.2 Execution sequence

```
TOMBSTONE_EXECUTION:

  Phase 1 — Authorization (Day 0):
    1.1 Trigger validated per §4
    1.2 Authority obtained per §3.1-3.5
    1.3 Bestuur ratification scheduled if post-Stichting
        (rapid bestuur within 24-48 hours if urgent)
  
  Phase 2 — Tombstone preparation (Day 0 - Day 1):
    2.1 TOMBSTONE_NOTICE canon entry drafted with §5.1 structure
    2.2 Legal counsel review of tombstone notice content
        (especially minimization of personal data in notice itself)
    2.3 Sensitive identifying information minimized
  
  Phase 3 — Issuance (Day 1):
    3.1 TOMBSTONE_NOTICE signed by Foundation
    3.2 Issued per ISSUE_CANON_ENTRY procedure in GOVERNANCE-SPEC §4.3
    3.3 Added to next master_index publication
    3.4 Recorded in GOVERNANCE_AUDIT_LOG
  
  Phase 4 — Content access change (Day 1 - immediate):
    4.1 Foundation-operated mirrors stop serving content of 
        affected entry
    4.2 HTTP 451 response configured for affected entry per §6
    4.3 Witness federation mirrors notified of tombstone
    4.4 Witness federation mirrors in compelled jurisdictions 
        also stop serving content (within 24 hours)
    4.5 Verifier-side caching expires (CDN/cache TTL respected)
  
  Phase 5 — Documentation (Day 1 - Day 7):
    5.1 Tombstone notice publicly announced via:
        - Foundation public statement on website
        - Twitter
        - NLnet correspondence (if NLnet-related entry)
        - Direct notification to known interested parties
    5.2 Internal documentation of decision rationale
    5.3 If subject is known and reachable: notification to subject
        that tombstone has been issued
  
  Phase 6 — Verification (Day 7):
    6.1 External verifier confirms HTTP 451 returned correctly
    6.2 External verifier confirms hash/signature still queryable
    6.3 Internal verification: tombstone reflected in master_index
    6.4 Tombstone procedure complete
```

---

## 6. HTTP 451 response handling

### 6.1 Server-side response

When a verifier requests content of a tombstoned entry:

```
HTTP/1.1 451 Unavailable For Legal Reasons
Content-Type: application/json
Link: <tombstone_canon_id_url>; rel="blocked-by"
Cache-Control: must-revalidate

{
  "type": "https://a1a5nt.org/errors/canon-tombstoned",
  "title": "Canon entry retired under legal compulsion",
  "status": 451,
  "detail": "The requested canon entry has been retired and is 
            no longer servable. Original cryptographic hash and 
            signature remain queryable through standard endpoints.",
  "instance": "/v1/canon/blake3_of_original",
  "tombstone_canon_id": "blake3_of_tombstone_notice",
  "tombstone_reason_category": "RT-LEGAL-COMPULSION",
  "tombstone_jurisdiction": "NL/EU",
  "verifier_action": "Refer to tombstone_canon_id for full details; 
                     hash and signature of original entry remain 
                     queryable at /v1/canon/{id}/hash and 
                     /v1/canon/{id}/signature respectively."
}
```

### 6.2 Verifier-side handling

Verifier implementations MUST:

- Handle HTTP 451 gracefully (do not retry)
- Display tombstone notice to user
- Allow user to query hash and signature via separate endpoints
- Distinguish HTTP 451 from HTTP 404 or 410 (resource missing 
  vs intentionally retired)

### 6.3 Hash endpoint

Even for tombstoned entries:

```
GET /v1/canon/{canon_id}/hash

HTTP/1.1 200 OK
Content-Type: application/json

{
  "canon_id": "blake3:...",
  "original_blake3_hash": "blake3:...",
  "issuance_timestamp": 1234567890,
  "state": "TOMBSTONED",
  "tombstone_canon_id": "blake3:...",
  "tombstone_reason_category": "RT-LEGAL-COMPULSION"
}
```

This endpoint returns 200 OK with the cryptographic hash even
when the content endpoint returns 451.

### 6.4 Signature endpoint

```
GET /v1/canon/{canon_id}/signature

HTTP/1.1 200 OK

{
  "canon_id": "blake3:...",
  "foundation_signature": {
    "algorithm": "XMSS-SHA2_20_256",
    "signature": "base64:...",
    "signer_xmss_root": "blake3:..."
  },
  "issuance_timestamp": 1234567890,
  "state": "TOMBSTONED"
}
```

The signature itself remains verifiable: any party with a 
legitimately-acquired copy of the original content can verify
that their copy matches the canonical hash.

---

## 7. Custody chain implications

### 7.1 Tombstoned entries in custody chain

If a canon entry referenced by custody records is tombstoned:

```
CUSTODY_TOMBSTONE_HANDLING:

  Effect on custody chain:
  - Custody records continue to reference the original canon_id
  - Custody chain remains intact (cryptographic references valid)
  - But: the underlying content cannot be served by Foundation
    mirrors in compelled jurisdictions
  
  Custodian's position:
  - Custodian retains their custody right (registered, signed)
  - Custodian may have legitimately-acquired copy of content
    predating tombstone; that copy can be verified against hash
  - Custodian cannot ask Foundation to re-serve content
  - Custodian may need to seek their own legal advice if 
    affected by tombstone
  
  Foundation duty:
  - Foundation notifies custodian of tombstone within 7 days
  - Foundation provides custodian with tombstone notice details
    (within legal constraints)
  - Foundation does not unilaterally void custody record
```

### 7.2 Tombstone of custody record itself

Custody records may themselves be subject to tombstoning (e.g.,
if a custody record contains personal data subject to GDPR
erasure):

- Custody record content tombstoned per §5
- Custody chain reference preserved
- Subsequent custody transitions can still reference the 
  tombstoned custody record's id and signature
- Operational continuity maintained

---

## 8. Tombstone reversal

### 8.1 When tombstone reversal is permitted

A tombstone MAY be reversed if:

- Court order is itself reversed (e.g., on appeal)
- GDPR request is withdrawn or determined invalid post-hoc
- Operational mistake in issuing tombstone is discovered
- Subject of erasure request later consents to publication

### 8.2 Tombstone reversal procedure

```
TOMBSTONE_REVERSAL:

  Phase 1 — Authorization:
    1.1 Bestuur (post-Stichting) or operator + legal counsel 
        (pre-Stichting) determines reversal is appropriate
    1.2 Reversal MUST be supported by clear evidence:
        - Court order reversal (formal court document)
        - Withdrawal of GDPR request (signed by original requester)
        - Mistake documentation (formal analysis)
  
  Phase 2 — Issuance:
    2.1 TOMBSTONE_REVERSAL canon entry issued
    2.2 Structure mirrors §5.1 with additional fields:
        - "reversal_authority_reference": basis for reversal
        - "reversed_tombstone_canon_id": reference to original 
          tombstone
        - "reversal_reasoning": explanation
    2.3 Original entry state transitions back to ACTIVE per
        STATE-MACHINE §3.2 T(reverse_tombstone)
    2.4 Content servable again on Foundation mirrors
  
  Phase 3 — Notification:
    3.1 Public announcement
    3.2 Notification to all parties previously notified
    3.3 NLnet correspondence updated
```

### 8.3 Tombstone reversal is rare

In practice, reversals are uncommon. Most tombstones are
permanent. Reversal is reserved for clear corrections.

---

## 9. Tombstone scope considerations

### 9.1 Geographic scope

Some legal orders bind only certain jurisdictions:

- US-only DMCA takedowns: Foundation in NL may not be subject
  to compulsion; case-by-case legal review
- EU GDPR orders: Foundation IS subject; tombstone applies
  EU-wide
- Local NL court orders: Foundation IS subject; tombstone applies
  NL/EU-wide for compliance

The TOMBSTONE_NOTICE explicitly documents geographic scope.
Witness federation members in non-compelled jurisdictions are
NOT required to suspend serving content based on geographic
scope of compulsion.

### 9.2 Time-bounded vs permanent

Some legal orders are time-bounded:

- Temporary restraining orders
- Time-limited sanctions
- Embargo periods that may lift

The TOMBSTONE_NOTICE documents whether tombstone is permanent or
time-bounded. Time-bounded tombstones automatically reverse upon
expiration of the underlying compulsion, with TOMBSTONE_REVERSAL
issued at that time.

### 9.3 Granularity

Tombstone applies to a specific canon entry. If a Foundation
Declaration contains a portion subject to compulsion:

- Option A: Tombstone the entire Declaration
- Option B: Issue a REDACTED version (which is operationally a
  new canon entry that supersedes the original; the original is
  still tombstoned for its sensitive portion)

Granularity is decided per-case based on legal advice and 
operational practicality.

---

## 10. Audit and logging requirements

### 10.1 GOVERNANCE_AUDIT_LOG entries

Every tombstone action MUST be logged:

```
LOGGED_EVENTS:

  TOMBSTONE_REQUEST_RECEIVED   When trigger arrives
  TOMBSTONE_VALIDATION         Validation procedure performed
  TOMBSTONE_DECISION           Decision made (proceed or decline)
  TOMBSTONE_ISSUED             TOMBSTONE_NOTICE issued
  TOMBSTONE_PROPAGATED         Witness federation notified
  TOMBSTONE_REVERSAL_REQUESTED  Reversal requested
  TOMBSTONE_REVERSED            Reversal executed
```

### 10.2 Public-facing transparency

Foundation publishes aggregate statistics on tombstone activity:

```
PUBLIC_TRANSPARENCY:

  Quarterly: 
    - Total tombstones issued
    - By category (RT-LEGAL-COMPULSION, RT-COPYRIGHT, etc.)
    - Jurisdiction breakdown
    - Number of court orders processed
    - Number of GDPR requests processed
  
  Per-incident:
    - For high-public-interest tombstones (e.g., specifications
      affecting many parties): individual public notice
    - For privacy-sensitive tombstones (e.g., GDPR personal data):
      aggregate only, subject identity protected
```

### 10.3 NLnet reporting

For tombstones affecting NLnet-related entries (e.g., grant 
documentation), specific NLnet correspondence notification is 
required.

---

## 11. Limitations of tombstone approach

### 11.1 What this approach cannot do

The hybrid tombstone pattern is robust but has limits:

- Cannot prevent witness federation members in non-compelled
  jurisdictions from serving content
- Cannot prevent users from having local copies of content
  predating tombstone
- Cannot prevent hash from being computed from content (the hash
  remains a valid output)
- Cannot fully address situations where a court order specifically
  requires hash erasure (unprecedented but conceivable)

### 11.2 Hash erasure scenarios

If a court order specifically compels hash erasure:

- This is an unusual order; Foundation should challenge in court
- If court order survives challenge, this is a Cat-3 event per
  SECURITY-MODEL §8 (irreversible loss of cryptographic record)
- Compliance requires deletion of canonical record itself
- Restoration impossible

This scenario is documented but not expected in practice. Hashes
are mathematical facts about a piece of data; orders to erase
mathematical facts are unprecedented in legal history.

### 11.3 Sanctions enforcement

OFAC and EU sanctions may require:

- Cessation of services to designated parties
- Custody transfer halts to sanctioned recipients
- Removal of designated parties' identifiers from canon entries

These are operational compliance matters; tombstone procedures
may apply to specific entries naming sanctioned parties.

---

## 12. Cross-references

| Spec | Relationship |
|------|--------------|
| A1A5NT-ARCHITECTURE-SYNTHESIS v1.2 | Master document |
| SECURITY-MODEL.md §A13 | Hybrid tombstone pattern definition |
| STATE-MACHINE-SPEC.md §3 | Canon entry state transitions |
| GOVERNANCE-SPEC.md §4 | Canon issuance authority |
| GOVERNANCE-SPEC.md §8 | Emergency authority for tombstone |
| LEGAL-INTERPRETATION-SPEC.md §5 | GDPR compliance framework |
| LEGAL-INTERPRETATION-SPEC.md §10 | International compliance |
| CANON-REGISTRY-SPEC.md | Canon entry structure |
| RECOVERY-SPEC.md §5 | Cryptographic compromise (different from tombstone) |
| NETWORK-PROTOCOL-SPEC.md §6 | Witness federation notification |

---

## 13. License

This specification is issued under A1A5 Sovereign License v1.0
(12-month binding 2026-05-16 → 2027-05-16). AI co-authorship per
Article 4. Anti-patent defensive clause per Article 5.

Retirement patterns adapted from:
- RFC 7725 (IETF, Trust Legal Provisions)
- Industry practice (Certificate Transparency, Sigstore Rekor,
  Internet Archive)

Specific a1a5NT tombstone notice structure and operational
procedures are A1A5-licensed.

---

## 14. Cryptographic anchor

```
Document path:    CANON-RETIREMENT-SPEC.md
Repository:       github.com/a1a5NT/a1a3
Commit timestamp: filled at first commit
Foundation mirror endpoint:  http://37.97.202.97:8444/v1/foundation/canon_retirement_spec  (scheduled)
```

---

**End of CANON-RETIREMENT-SPEC v1.0**

```
Issued by:        pawel.a1a5 (Paweł Piekut, Almelo NL)
AI co-author:     Claude (Anthropic Opus 4.7)
Date:             2026-05-17
Foundation:       a1a5fndn.a1a5 (in formation)
Audit relevance:  BLOCKING for notariusz Stichting registration;
                  REQUIRED for GDPR compliance demonstration;
                  REQUIRED for NLnet legal review
Reviewer consensus addressed: Reviewer N N-B1 CRITICAL +
                  SECURITY-MODEL §A13 hybrid tombstone implementation
```
