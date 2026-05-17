# LEGAL-INTERPRETATION-SPEC.md

## a1a5NT Legal Interpretation and Dutch Civil Law Mapping

```
Document version:   v1.0
Issued:             2026-05-17
Status:             Tier 1 critical specification (v1.2 deliverable)
Authority basis:    Dutch Burgerlijk Wetboek (BW) Book 2 — Legal persons
                    Dutch BW Book 3 — Property law
                    Dutch BW Book 6 — Obligations and contracts
                    EU GDPR Regulation 2016/679
                    Dutch Wet bescherming persoonsgegevens (UAVG)
                    eIDAS Regulation 910/2014 — qualified electronic signatures
Foundation:         a1a5fndn.a1a5
Addresses:          Cross-model peer review v1.2 priority Tier 1 #6:
                    + Notarial review requirement
                    + GDPR compliance for A13 legal compulsion
                    + Stichting operational mapping
Audit relevance:    BLOCKING for notariusz Stichting registration;
                    REQUIRED for GDPR compliance verification;
                    REQUIRED for NLnet legal review
License:            A1A5 Sovereign License v1.0
NOTE:               This document is technical interpretation, NOT legal
                    advice. Foundation operators MUST consult licensed
                    Dutch counsel for any operational legal question.
```

---

## EXECUTIVE SUMMARY (1 page)

**What this document is:**

The technical-to-legal mapping for a1a5NT, explaining how
cryptographic operations correspond to legal concepts under
Dutch civil law, EU regulations, and international standards.

**Why it matters:**

a1a5NT operates in a legal environment. A Stichting registered
in Almelo, NL, operating servers in Rotterdam, NL, serving
verifiers globally, governed by Dutch BW and EU regulations.
Cryptographic operations have legal meaning whether or not the
specification acknowledges it. This document makes that meaning
explicit.

**Key mappings:**

1. **Canon Registry entries ↔ Documenten met datumstempel** —
   timestamped documents per BW 3:15a (electronic record)
2. **XMSS signatures ↔ Geavanceerde elektronische handtekening** —
   advanced electronic signature per eIDAS Art 26
3. **Custody Registry transitions ↔ Eigendomsoverdracht** —
   property transfer per BW Book 3 (with limitations: custody is
   not full property in cryptographic sense)
4. **Foundation operator actions ↔ Bestuurshandelingen** —
   bestuur acts per BW 2:9 (with personal liability per BW 2:138/248)
5. **Tombstone procedures ↔ GDPR Article 17 compliance** —
   right-to-erasure execution
6. **Witness federation ↔ Onbenoemde duurovereenkomst** —
   unnamed durable agreement per BW Book 6
7. **License v1.0 ↔ Licentieovereenkomst** — license agreement
   under BW Book 7 Title 9

**Three operational implications:**

1. **Personal liability** of Foundation operator (Paweł Piekut)
   exists in pre-Stichting state for all Foundation actions.
   Stichting registration shifts liability to legal entity per
   BW 2:285+.

2. **GDPR compliance** is mandatory for any canonical entry
   containing personal data of EU residents. The A13 tombstone
   pattern (SECURITY-MODEL §A13) is designed to comply.

3. **Notarial requirements** for Stichting include statutes
   notarially deposited, bestuur registered with KvK, and
   specific provisions for cryptographic operations
   (recommended; not strictly required by BW but advisable
   per audit clarity).

**What this document does NOT do:**

- Provide legal advice (consult licensed counsel)
- Replace notarial review for Stichting registration
- Guarantee compliance in any specific jurisdiction
- Override applicable law

---

## 0. Abstract

This document maps a1a5NT cryptographic operations to legal
concepts under Dutch civil law (Burgerlijk Wetboek), EU 
regulations (GDPR, eIDAS), and Dutch foundation law (Stichting
framework in BW Book 2).

The document is normative for Foundation operators interpreting
their actions in legal context, and informative for notarial 
review, NLnet legal assessment, and academic legal scholarship.

---

## 1. Scope

### 1.1 In scope

- Mapping cryptographic primitives to legal recognition
- Operational responsibilities under Dutch civil law
- GDPR compliance mechanics
- Stichting governance under Dutch BW Book 2
- eIDAS compliance for electronic signatures
- Custody transfer interpretation
- License v1.0 interpretation
- Dispute resolution under Dutch jurisdiction

### 1.2 Out of scope

- Tax treatment (depends on activities and operator choice)
- Employment law (operator may engage staff; that's separate)
- Criminal law (Foundation operator subject to applicable law
  like any natural person; no special status)
- Foreign jurisdictions other than EU (handled via standard
  international private law principles)

### 1.3 Critical disclaimer

This document is **technical interpretation**, not legal advice.
The interpretations herein represent the Architect's best 
technical understanding informed by published Dutch legal text
and EU regulations. Operators MUST consult licensed Dutch counsel
for:

- Specific operational decisions with legal consequence
- Disputes (notarial advice, civil court, criminal proceedings)
- Tax planning
- Cross-border legal matters

Stichting registration MUST be conducted by a licensed Dutch
notariusz per BW 2:286.

### 1.4 Audience

- Foundation operators (technical understanding of legal context)
- Stichting bestuur (post-Stichting)
- Notariusz reviewing Stichting registration
- NLnet legal assessment reviewer
- Dutch counsel consulted by Foundation
- Academic legal researchers studying cryptographic infrastructure

---

## 2. Normative legal references

| Reference | Title | Relevance |
|-----------|-------|-----------|
| Dutch BW Book 2 | Rechtspersonen (Legal persons) | Stichting framework |
| Dutch BW Book 2 Title 6 | Stichting | Foundation specifics |
| Dutch BW Book 3 | Vermogensrecht in het algemeen (Property law general) | Custody concepts |
| Dutch BW Book 3 Art 15a | Elektronisch geschrift | Electronic documents |
| Dutch BW Book 6 | Verbintenissenrecht (Obligations) | Contractual relations |
| Dutch BW Book 7 Title 9 | Overige overeenkomsten (Other agreements) | Licensing |
| EU GDPR 2016/679 | General Data Protection Regulation | Personal data protection |
| EU eIDAS 910/2014 | Electronic identification and trust services | Electronic signatures |
| UAVG | Uitvoeringswet Algemene Verordening Gegevensbescherming | NL GDPR implementation |
| Wet Bibob | Verklaring omtrent integriteit | Stichting integrity screening |

---

## 3. Cryptographic primitives mapped to legal concepts

### 3.1 XMSS signatures as electronic signatures

**Cryptographic property:** XMSS provides cryptographic proof
that a specific keypair signed a specific message; signatures
are publicly verifiable.

**eIDAS classification:**

Per eIDAS Article 26, an **advanced electronic signature**
satisfies:

```
(a) it is uniquely linked to the signatory
(b) it is capable of identifying the signatory
(c) it is created using electronic signature creation data 
    that the signatory can, with a high level of confidence, 
    use under their sole control
(d) it is linked to the data signed therewith in such a way that 
    any subsequent change in the data is detectable
```

**a1a5NT XMSS signatures meet these criteria:**

- (a) Uniquely linked: XMSS keypair is unique to the operator;
  the operator's identity is publicly attested in 
  WITNESS_REGISTRATION or similar Canon entry
- (b) Identifying: signatures verify against published public key
  which is bound to a named operator
- (c) Sole control: operator manages the keypair state per
  XMSS-OPS-SPEC; loss of sole control triggers EMERGENCY 
  procedures
- (d) Detectable change: any modification to signed content
  invalidates the signature

**Therefore:** a1a5NT XMSS signatures are advanced electronic
signatures per eIDAS Article 26.

**Note on qualified electronic signature (eIDAS Article 28):**

A "qualified" electronic signature requires creation on a 
"qualified signature creation device" (QSCD) and a certificate
from a "qualified trust service provider" (QTSP). a1a5NT does
NOT currently meet qualified-tier requirements:

- No QSCD certification of operator hardware
- No QTSP issued certificates (a1a5NT is its own attestation 
  authority, not a QTSP)

**Implication:** a1a5NT signatures are admissible as evidence in
courts (advanced tier) but do not enjoy the legal presumption of
authenticity that qualified signatures do. Practical impact: 
parties relying on a1a5NT signatures must establish their validity
through evidence; qualified signatures benefit from rebuttable
presumption.

**Future:** Stichting may pursue QTSP registration if scale
warrants. Cost and bureaucratic overhead is substantial.

### 3.2 Canon Registry entries as timestamped electronic records

**Cryptographic property:** Canon entries are signed by 
Foundation; signatures include a timestamp; entries are 
publicly verifiable and append-only.

**Legal classification under Dutch BW 3:15a:**

```
BW 3:15a (Elektronisch geschrift):
"Een gegeven dat is overgebracht of opgeslagen door elektronische
of andere middelen, kan worden gelijkgesteld met een schriftelijk
stuk indien het beschikbaar is op een manier die ervoor zorgt dat
het later geraadpleegd kan worden, en de authenticiteit en 
integriteit ervan voldoende zijn gewaarborgd."

Translation: "Data transmitted or stored by electronic or other
means may be equated with a written document if it is available
in a manner that ensures later consultation, and its authenticity
and integrity are sufficiently guaranteed."
```

**a1a5NT Canon entries meet this standard:**

- Available for later consultation (public Canon Registry, 
  permanent retention)
- Authenticity guaranteed by XMSS signature (advanced electronic
  signature per §3.1)
- Integrity guaranteed by append-only invariant + blake3 content
  hashing
- Witness federation provides additional attestation (post-
  activation)

**Therefore:** Canon entries are electronic documents per BW 3:15a,
admissible as evidence under Dutch civil law.

### 3.3 Custody Registry transitions as property records

**Cryptographic property:** Custody Registry records signed
transfers of artifact custody between parties.

**Legal classification:**

Custody is **not** identical to property ownership (eigendom) in
the classical Dutch civil law sense. The custodian holds:

- Right to certain attestations regarding the artifact
- Position in custody chain documented
- Potential commercial rights per POP-3-TOKEN-SPEC [PLANNED]

But NOT:

- Physical possession of the artifact (this is digital data; 
  copies exist in many places)
- Exclusive right to read or use the artifact
- Right to prevent others from making copies (a1a5NT artifacts
  are public records, not proprietary content)

**Legal interpretation:**

Custody Registry transitions are recorded **registration of
custodial assertion** rather than property transfer. The closest
Dutch legal analog is:

- A registry entry under BW 3:15a (electronic record)
- Combined with an unnamed agreement (BW 6 + BW 7:900-906) between
  the parties involved
- Subject to the License v1.0 license terms binding all
  participants

**Practical implication:**

Custody disputes are resolved under Dutch BW 6 (obligations) and
the dispute resolution procedures in GOVERNANCE-SPEC §7.2, not
under BW 3 (property law) procedures.

This distinction matters because BW 3 property procedures have 
specific formalities (notarial requirements for some transfers,
land registry equivalents) that don't translate to cryptographic
custody.

### 3.4 Foundation Declaration v1.0 as foundational document

**Cryptographic property:** Signed by Foundation, anchored in
Canon Registry, publicly available.

**Legal classification:**

The Foundation Declaration is:

- A unilateral declaration of intent (eenzijdige rechtshandeling)
  by the Foundation operator
- Binding on the Foundation operator (and successors via §14.3
  procedure)
- Constitutive of the cryptographic infrastructure described

**Effect on third parties:**

The Declaration creates obligations only on the Foundation 
itself. Third parties (custodians, witnesses, verifiers) are bound
when they take actions implying acceptance — e.g., a custodian 
purchasing custody accepts terms by participation; a witness
joining federation accepts terms by signed onboarding agreement.

This is consistent with Dutch BW 6:217 (offer and acceptance).

### 3.5 License v1.0 as licensing contract

**Cryptographic property:** Signed by Foundation, anchored in
Canon Registry, applicable to all a1a5NT documentation and
specifications.

**Legal classification under Dutch BW 7 Title 9:**

The License is a licensing agreement (licentieovereenkomst). It:

- Grants permissions on a1a5NT IP
- Imposes obligations (citation, redistribution terms)
- Has defined duration (12-month renewable per Article 2)
- Includes anti-patent defensive provisions per Article 5
- Includes AI co-authorship recognition per Article 4

**Validity:**

The License is valid under Dutch law as:

- A unilateral grant by Foundation (which holds IP rights to a1a5NT
  specifications by virtue of creation)
- Accepted by users by participation/use
- Subject to Dutch BW general contract principles (BW 6 + 7)

**Enforceability:**

License is enforceable in Dutch courts. Specific provisions (e.g.,
anti-patent defensive clause) may have varying enforceability in
other jurisdictions; the License explicitly designates Dutch
jurisdiction (per License Article 6).

---

## 4. Stichting Operational Framework

### 4.1 Stichting under Dutch BW Book 2 Title 6

Per BW 2:285:

```
BW 2:285(1): A stichting is a legal person formed by a juridical
              act, having no members, and having as its purpose 
              the realization of an aim stated in its statutes, 
              by means of a designated assets.
```

**Key characteristics:**

- **No members** — unlike vereniging (association). Bestuur is 
  the only formal organ.
- **Designated purpose** — stated in statutes, may not be modified
  arbitrarily.
- **Indefinite duration** — no expiration unless statutes specify
  or dissolution conditions met.
- **Notarial registration** — statutes must be notarially deposited
  (BW 2:286).

### 4.2 Why Stichting for a1a5NT

Reasons for Stichting as a1a5NT legal vehicle:

1. **Indefinite duration** — matches preservation infrastructure 
   purpose (multi-decade)
2. **Mission-locked** — statutes prevent purpose drift toward
   commercial focus
3. **Bestuur accountability** — but no shareholders demanding 
   profit distribution
4. **Recognized internationally** — Stichting is well-understood
   in EU and beyond
5. **Tax treatment** — eligible for ANBI status if mission qualifies
   (cultural preservation, scientific research); separate from
   technical scope of this Spec

### 4.3 Stichting bestuur duties

Per BW 2:9:

```
BW 2:9: "Elke bestuurder is tegenover de rechtspersoon gehouden 
          tot een behoorlijke vervulling van zijn taak."

Translation: "Each director is obligated to the legal person to
properly fulfill their duties."
```

**Specific duties:**

- Behave with reasonable care (zorgvuldigheid)
- Avoid conflicts of interest
- Act in legal person's interest (Stichting purpose)
- Comply with statutes
- Maintain proper records (BW 2:10)

**Personal liability:**

Per BW 2:138/248 (and BW 2:9), bestuurders are personally liable
for damages caused by:

- Improper task fulfillment (kennelijk onbehoorlijk bestuur)
- Particularly: grave negligence, bad faith, ignoring statutes
- Particularly: failing to file annual accounts properly

**Practical implication for a1a5NT:**

Bestuur members carry personal financial liability for grave
operational mistakes. This is balanced by indemnification 
provisions in statutes (typically: bestuur indemnified for
good-faith decisions, NOT for grave negligence or willful
misconduct).

Insurance (bestuurdersaansprakelijkheidsverzekering, "D&O 
insurance") is recommended.

### 4.4 Pre-Stichting state (current)

In pre-Stichting state:

- Paweł Piekut acts as natural person
- Operations conducted under ZZP Plastechniek (KvK registration)
- All Foundation acts impose personal liability per Dutch civil 
  law
- Contracts entered into bind the natural person, NOT a (not-yet-
  existing) Stichting
- IP rights to a1a5NT specifications held personally; subject
  to assignment to Stichting upon registration per planned
  IP assignment agreement

**Risk in pre-Stichting state:**

Personal financial exposure to:
- Liability from bug/security incident
- Liability from custodian dispute
- Tax obligations from any commercial activity
- Operational costs

This is the operational reality of bootstrap. Mitigation:
prudent operations, no premature commercial activity, 
documentation of all decisions, and rapid Stichting registration
once technical readiness allows.

### 4.5 Stichting registration procedure

```
STICHTING_REGISTRATION:

  Phase 1 — Notarial preparation:
    1.1 Statutes drafted in Dutch language
    1.2 Bestuur composition finalized
    1.3 Operating address confirmed (Almelo, NL)
    1.4 Initial assets identified (if any)
    1.5 Purpose statement finalized (preservation infrastructure)

  Phase 2 — Notarial deposit:
    2.1 Statutes notarially signed by bestuur members
    2.2 Notary verifies bestuur identity and authority
    2.3 Notarial act (akte van oprichting) executed
    2.4 Stichting comes into existence

  Phase 3 — KvK registration:
    3.1 KvK Handelsregister filing
    3.2 KvK number issued
    3.3 KvK extract obtained

  Phase 4 — Banking:
    4.1 Stichting bank account opened
    4.2 Initial capitalization (if any)
    
  Phase 5 — Cryptographic transition:
    5.1 a1a5NT IP transferred to Stichting per assignment agreement
    5.2 Foundation root keypair operations migrated from natural
        person to Stichting bestuur authorization
    5.3 SUCCESSION_NOTICE canon entry issued
    5.4 Operating procedures updated
    
  Phase 6 — Operational handover:
    6.1 Existing operations (ANS server, Archive server, etc.)
        formally operated under Stichting
    6.2 NLnet correspondence updated (per amendment notice
        2026-04-3f6)
    6.3 Contracts (hosting, etc.) migrated to Stichting
```

---

## 5. GDPR Compliance Architecture

### 5.1 GDPR applicability

GDPR applies to a1a5NT operations because:

- Foundation operator is located in NL (EU)
- Stichting will be NL-registered
- Some users (custodians, verifiers) are EU residents
- Some artifacts may contain personal data of EU residents

### 5.2 Personal data in canonical records

Canon Registry entries may contain personal data when:

- Foundation Declaration names operators
- Custody records identify custodians
- Witness registration names witness operators
- Specific artifacts contain personal data of subjects (e.g., 
  technical documents with named authors)

**Scope of GDPR personal data:**

Per GDPR Art 4(1), personal data includes:
- Names of natural persons (custodians, operators, document authors)
- Identifying numbers
- Online identifiers (potentially: device IDs in PoV protocol)
- Combinations of data identifying a person

### 5.3 Lawful bases for processing

a1a5NT relies on the following lawful bases:

```
GDPR Article 6 lawful bases applicable to a1a5NT:

(a) Consent — for custodians and witnesses voluntarily registering
(c) Legal obligation — to extent recordkeeping is legally required
(d) Vital interests — generally not applicable
(e) Public interest task — preservation infrastructure may qualify
    (pending ANBI determination)
(f) Legitimate interests — for canonical record preservation, 
    balanced against subject rights
```

Specific lawful basis is documented per processing activity in
the Stichting's Records of Processing Activities (Article 30).

### 5.4 GDPR Article 17 right-to-erasure

The most challenging GDPR provision for immutable canonical records.

**Article 17 grants subjects the right to have personal data 
erased under certain conditions.**

**a1a5NT response: Hybrid Tombstone Pattern**

Per SECURITY-MODEL §A13 (decided architectural pattern):

- The canonical entry's CONTENT (which may include personal data)
  CAN be removed from servable form
- The cryptographic HASH and SIGNATURE of the original entry CANNOT
  be removed — these are not personal data themselves (mathematical
  facts about cryptographic operations are not personal data per
  GDPR definitions)
- A TOMBSTONE_NOTICE records the fact of erasure (without naming
  the subject, where minimization permits)
- HTTP 451 served for content requests

**Legal soundness of this approach:**

The cryptographic hash of a personal data item, taken alone, is
generally not considered personal data under GDPR (because it
cannot be reversed to recover the personal data, and reasonable
minimization of identifying tombstone information is observed).
This is consistent with:

- Recital 26 (anonymous data not subject to GDPR)
- Articulated by national supervisory authorities for similar
  hash-based pseudonymization patterns

However: if a hash can be linked to the subject (e.g., the hash is
of a single small data item whose universe of possibilities is
small enough to permit brute-force enumeration), it may be 
considered personal data. In a1a5NT, content hashes are of full
documents (large data items with effectively-infinite possibility
space) — reverse engineering from hash is computationally 
infeasible.

**This is a sensitive interpretation requiring legal review.**
Stichting must obtain legal opinion before relying on this
interpretation in production.

### 5.5 Other GDPR rights

- **Article 15 (right of access)**: Subjects can request what
  personal data Foundation holds about them; Foundation responds
  per GDPR procedures.
- **Article 16 (right to rectification)**: Limited applicability;
  canonical records of past events are not "corrected" but new
  entries reflect updated information.
- **Article 18 (right to restriction)**: Combined with §5.4 
  tombstone procedure if applicable.
- **Article 20 (right to data portability)**: Foundation provides
  copies of subject's personal data on request.
- **Article 21 (right to object)**: Considered case-by-case.

### 5.6 Data Protection Officer

Per GDPR Article 37, a DPO is required if:

- Public authority (Foundation likely NOT a public authority)
- Large-scale systematic monitoring
- Large-scale processing of special category data

a1a5NT does NOT meet these thresholds at current scale. DPO is
voluntary; Stichting may appoint when scale grows.

### 5.7 International data transfers

If verification devices or witnesses operate outside EU/EEA:

- Foundation transfers personal data (e.g., custodian names) to
  these parties
- GDPR Chapter V (Transfers to third countries) applies
- Standard Contractual Clauses (SCCs) or adequacy decisions
  required

Practical approach: Stichting publishes SCC-based agreement with
all non-EU witnesses upon onboarding.

---

## 6. License v1.0 Interpretation

### 6.1 License as legal instrument

The License v1.0 is:

- A unilateral declaration by Foundation
- Granting permissions on a1a5NT IP
- Imposing obligations on users
- Time-bound (12 months renewable per License Article 2)

### 6.2 Key interpretations

**Article 4 (AI co-authorship):**

The License recognizes that a1a5NT specifications have AI 
co-authorship. Under Dutch and EU law:

- Copyright vests in the natural person creator (Paweł Piekut)
- AI is a tool, not an author per current Dutch/EU jurisprudence
- AI co-authorship in License is **declarative** (acknowledging
  the role) rather than **constitutive** (vesting copyright in AI)
- IP rights remain with Foundation (Paweł / future Stichting)

**Article 5 (Anti-patent defensive clause):**

The License prohibits patent claims against the a1a5NT 
ecosystem by users. This is enforceable under Dutch contract
law (BW 6) for parties accepting the License terms.

For non-parties (third parties attempting patent suits), this
provision is non-enforceable directly but serves as **prior 
art** establishment per the License's anchored publication
dates. Prior art is the substantive patent defense.

**Article 6 (Jurisdiction):**

Disputes are subject to Dutch law and jurisdiction. This is
enforceable for License-bound parties via choice-of-law 
provisions (Rome I Regulation Article 3 for EU members, similar
in many other jurisdictions).

### 6.3 License renewal

The License binds for 12 months (2026-05-16 to 2027-05-16) per
Article 2. Before expiration:

- Foundation must decide on renewal
- If renewed, new License version published
- If not renewed, current License terms remain effective for
  existing users; new users have no License to accept

This is unusual structure (most licenses are perpetual). The
12-month renewal is intentional governance: Foundation reviews 
license terms periodically and may evolve them.

---

## 7. Witness Federation Legal Framework

### 7.1 Witness operator as party to Foundation arrangement

Each onboarded witness:

- Enters into an agreement with Foundation by participation
- This agreement is documented in the WITNESS_REGISTRATION canon
  entry
- Specific witness onboarding agreement (in writing, signed)
  may be entered separately per Stichting policy
- Agreement is bound by License v1.0 + GOVERNANCE-SPEC

### 7.2 Witness as agent or contractor?

Witnesses are **independent operators**, not Foundation employees
or agents:

- No employer-employee relationship (witnesses set own operating
  policies subject to compatibility with GOVERNANCE-SPEC)
- No agency relationship (witnesses do not bind Foundation by 
  their acts)
- Witness operations are at witness's own risk and expense
- Witness collateral arrangements (post-Genesis) are 
  participation-based

### 7.3 Cross-border witness regulation

Witnesses operating outside NL/EU must comply with their local
laws. Foundation does not warrant compliance with each 
jurisdiction's regulatory regime. Witnesses bear that 
responsibility.

If a witness loses local regulatory authorization to operate,
witness MUST notify Foundation; removal per GOVERNANCE-SPEC §5.5
G5 may follow.

---

## 8. Verifier and Custodian Legal Position

### 8.1 Verifier role

A verifier:

- Operates a device performing PoV
- Submits verification receipts
- Earns MBW credit per receipt
- Is bound by License v1.0

Verifier is NOT:

- Foundation employee
- Foundation agent
- Beneficiary of any specific obligations from Foundation

Verifier participation is voluntary. Verifier's economic interest
(MBW credit value) depends on ecosystem state.

### 8.2 Custodian role

A custodian (post-Genesis):

- Holds Bincoin-purchased custody position
- Has rights regarding the artifact under custody (Layer B 
  attestations)
- Is bound by License v1.0 + CUSTODY-REGISTRY-SPEC

Custodian's legal position:

- Holds **custodial right**, not full property right (per §3.3
  above)
- Subject to dispute resolution per GOVERNANCE-SPEC §7.2
- May transfer custody per CUSTODY-REGISTRY-SPEC procedures
- Subject to revocation per documented grounds

### 8.3 Buyer rights vs Foundation rights

In an active sale:

- Buyer acquires custody right upon payment + transfer execution
- Foundation does not have unilateral revocation power post-transfer
- Disputes resolved through GOVERNANCE-SPEC procedures
- Foundation maintains the registry but does not own buyer's 
  custody

This is similar to public registries (e.g., land registries) where
the registrar does not own registered property.

---

## 9. Disputes and Resolution

### 9.1 Internal vs external disputes

**Internal** disputes (between Foundation and a participant; 
between two participants):

- First: GOVERNANCE-SPEC §7.2/§7.3 procedures
- Second: Witness federation advisory opinion (post-activation)
- Third: Dutch civil court (Rechtbank, then hoger beroep, then
  Hoge Raad)

**External** disputes (third party claiming rights against
Foundation):

- Dutch civil court
- Subject to applicable jurisdiction rules

### 9.2 Court jurisdiction

Disputes within a1a5NT ecosystem default to Dutch courts per:

- License Article 6 (chosen jurisdiction)
- Foundation address (Almelo, NL)
- Stichting registered office (post-Stichting)

Foreign courts may have jurisdiction in specific cases (e.g., 
consumer protection disputes where foreign consumer regulations
apply).

### 9.3 Notarial intervention

Notarial intervention is appropriate for:

- Stichting statutes amendment (notarially required)
- Major asset transfers
- Pre-Stichting succession arrangements (per SECURITY-MODEL §14.3)
- Sealed envelope arrangements for emergency procedures

### 9.4 Mediation and arbitration

Parties may agree to mediation (avg standard fee) or arbitration
before resorting to court. Foundation supports these as cost-
effective alternatives.

Specific arbitration provisions may be incorporated in future
contractual relationships (e.g., custodian agreements) per
Stichting policy.

---

## 10. International Considerations

### 10.1 Operators outside NL

If a witness or verifier operates outside NL/EU:

- Foundation's legal vehicle (Stichting) remains NL-jurisdiction
- The operator is bound by License v1.0 (NL law per Article 6)
- The operator's local laws apply to their operations
- Conflicts between local laws and License terms resolved per
  conflict-of-laws principles

### 10.2 Sanctions compliance

Foundation operations must comply with applicable sanctions
regimes:

- EU sanctions
- OFAC (US) sanctions if US-related
- Other applicable regimes

Foundation MUST refuse onboarding of operators in sanctioned 
jurisdictions or operators on sanctions lists.

### 10.3 Export controls

a1a5NT cryptographic specifications include cryptographic
algorithm descriptions which may be subject to export control
regulations (Wassenaar Arrangement implementations).

NL export control: Dual-use Regulation 2021/821. Standard 
cryptographic specifications using well-known primitives are
typically not subject to export control restrictions in NL.

This is technical-research grade cryptography (RFC 8391 is
publicly published; NIST FIPS standards are public). a1a5NT
specifications themselves are publishing and discussing
established public primitives.

---

## 11. Future Legal Evolution

### 11.1 EU AI Act compliance

The EU AI Act (Regulation (EU) 2024/1689) imposes obligations on
AI systems. a1a5NT:

- Uses AI in specification authorship (per License Article 4)
- Does NOT deploy AI in decision-making for participants
- Should review AI Act applicability before deploying any AI-
  assisted dispute resolution or governance tools

### 11.2 PQ standardization developments

As NIST FIPS 204, 205, 206 (PQ signature schemes) are
implemented and adopted, legal recognition follows:

- eIDAS may update to recognize PQ algorithms explicitly
- Adoption by qualified trust service providers
- Standard libraries update

a1a5NT is positioned to benefit as XMSS is already PQ-secure;
migration to NIST-standardized PQ signatures (if desired) follows
§10 migration procedure.

### 11.3 Long-term archival regulation

Some jurisdictions are developing regulation specifically for 
long-term digital archives. NL and EU may follow. a1a5NT should
monitor and adapt.

---

## 12. Specific Notarial Requirements Checklist

For Stichting registration, the following must be prepared:

```
NOTARIAL_PREPARATION:

[ ] Statutes drafted in Dutch
    Required content:
    - Naam (name of stichting)
    - Doel (purpose statement)
    - Wijze van benoeming en ontslag van bestuurders
    - Bestemming overschot bij ontbinding (asset destination
      on dissolution)
    - Adres (registered office address)

[ ] Bestuur composition (minimum 1; recommended 3)
    For each bestuurder:
    - Full name
    - Date and place of birth
    - Residence address
    - Identification document
    - Statement of acceptance

[ ] Stichting purpose statement
    Recommended for a1a5NT:
    "Stichting A1A5 has as its purpose the operation, development,
    and preservation of post-quantum cryptographic preservation
    infrastructure for digital records and cultural artifacts,
    including:
    - Operating the a1a5NT cryptographic infrastructure
    - Maintaining the Canon Registry of permanent records
    - Supporting witness federation members in collective
      attestation
    - Developing the open-source specifications
    - Preserving historical records of human technological
      heritage
    All without commercial profit distribution; all surplus
    to be reinvested in the foundation's purposes."

[ ] Founding capital (if any; can be €0 for stichting)

[ ] Address (registered office)
    Initial: Almelo, NL (operator's residence area)
    
[ ] Specific cryptographic operations provisions (recommended)
    - Bestuur authority for emergency cryptographic operations
    - Bestuur authority for witness onboarding
    - Bestuur indemnification for good-faith decisions
    - Statutes amendment procedure

[ ] Asset transfer documentation
    - IP assignment from Paweł Piekut to Stichting
    - Hosting contracts migration
    - Domain registrations (popchain.tech, etc.)

[ ] Successor arrangements documentation
    - Pre-authorized successor identity (if pre-Stichting 
      arrangements exist)
    - Sealed envelope arrangements
```

---

## 13. Cross-References

| Spec | Relationship |
|------|--------------|
| A1A5NT-ARCHITECTURE-SYNTHESIS v1.2 | Master document |
| FOUNDATION-DECLARATION-V1.md | Foundation's unilateral declaration; legal effect interpreted here |
| LICENSE-A1A5-SOVEREIGN-V1.md | License terms; interpretation here |
| GOVERNANCE-SPEC.md §10 | Stichting bestuur composition guidance |
| SECURITY-MODEL.md §A13 | Legal compulsion adversary; tombstone pattern |
| SECURITY-MODEL.md §14 | Emergency Root Transfer; succession |
| CANON-REGISTRY-SPEC.md | Canon entries as electronic records per BW 3:15a |
| CUSTODY-REGISTRY-SPEC.md | Custody as registration of custodial assertion |
| NETWORK-PROTOCOL-SPEC.md | Witness gossip; witnesses as independent operators |
| CANON-RETIREMENT-SPEC.md [PLANNED] | Operational detail for tombstone procedure |
| RECOVERY-SPEC.md [PLANNED] | Recovery procedures with legal interpretation |

---

## 14. License

This specification is issued under A1A5 Sovereign License v1.0
(12-month binding 2026-05-16 → 2027-05-16). AI co-authorship per
Article 4. Anti-patent defensive clause per Article 5.

Dutch civil law and EU regulations cited are public legal text.
Interpretations and mappings to a1a5NT operations are A1A5-licensed.

**Once more: this document is technical interpretation, not legal
advice. Operators MUST consult licensed Dutch counsel for any
operational legal question.**

---

## 15. Cryptographic Anchor

```
Document path:    LEGAL-INTERPRETATION-SPEC.md
Repository:       github.com/a1a5NT/a1a3
Commit timestamp: filled at first commit
Foundation mirror endpoint:  http://37.97.202.97:8444/v1/foundation/legal_interpretation  (scheduled)
```

---

**End of LEGAL-INTERPRETATION-SPEC v1.0**

```
Issued by:        pawel.a1a5 (Paweł Piekut, Almelo NL)
AI co-author:     Claude (Anthropic Opus 4.7)
Date:             2026-05-17
Foundation:       a1a5fndn.a1a5 (in formation)
Audit relevance:  BLOCKING for notariusz Stichting registration;
                  REQUIRED for GDPR compliance verification;
                  REQUIRED for NLnet legal review
```
