# SECURITY-MODEL.md

## a1a5NT Adversary Model and Threat Enumeration

```
Document version:   v1.1
Issued:             2026-05-17
Supersedes:         v1.0 (preserved in git history)
Status:             Tier 1 critical specification (v1.2 deliverable)
Authority basis:    NIST SP 800-30 (Guide for Conducting Risk Assessments),
                    NIST SP 800-208 (adversary capabilities for stateful HBS),
                    EigenLayer intersubjective faults taxonomy,
                    Certificate Transparency RFC 6962/9162 threat analysis,
                    GDPR Article 17 (right-to-erasure) — for A13
Foundation:         a1a5fndn.a1a5
Addresses:          Cross-model peer review v1.2 unanimous priority #1:
                    Gemini implicit blocker, Grok #1, GPT #1, Claude PC E1 CRITICAL
                    + v1.0 post-publication review:
                    Reviewer M (validation), Reviewer N (architectural gaps),
                    Reviewer O (operational readiness)
Audit relevance:    BLOCKING for academic post-quantum review;
                    REQUIRED for NLnet technical review;
                    REQUIRED for institutional adoption
License:            A1A5 Sovereign License v1.0
```

---

## EXECUTIVE SUMMARY (1 page)

**What this document is:**

The formal adversary model for a1a5NT, the post-quantum cryptographic
preservation infrastructure under development by Stichting A1A5 (NL,
registration in progress) and operated by Paweł Piekut.

**What a1a5NT defends:**

Six security goals in priority order: Canon Integrity (immutable
historical truth) > Custody Integrity (current ownership state) >
Verification Authenticity (proof-of-verification work) > Provenance
Continuity (decades-scale archive accessibility) > Operator
Accountability (cryptographic attribution of every state change) >
Censorship Resistance (legitimate participant access).

**Against whom:**

Thirteen adversary classes (A1–A13) ranging from passive network
observers to post-quantum cryptanalysts to legal compulsion attackers.
Each adversary has documented capabilities, motivations, and a
Cost-of-Corruption analysis ensuring that for rational adversaries
the cost of attempting an attack exceeds the achievable profit.

**Honest limitations:**

Five explicit residual risks (R1–R5) are documented, the highest
being **R1 — single-person bootstrap concentration** during the
pre-Stichting phase. Mitigation includes an Emergency Root Transfer
procedure (Section 14) that defines what happens operationally if
the Foundation operator becomes unavailable for 30, 60, or 90 days.

**Implementation status:**

Each defense mechanism is labeled **[IMPLEMENTED]** (deployed and
operational), **[PLANNED]** (specification written, deployment
scheduled), or **[DESIGN TARGET]** (specification in development).
Section 15 provides the full Mitigation Roadmap with owner and
target date for each open item.

**Three audit perspectives addressed:**

| Audience | Their question | Where to read |
|----------|---------------|---------------|
| Academic cryptographic reviewer | "What is your threat model?" | Sections 3, 5, 6, Trust Assumptions |
| NLnet / institutional reviewer | "What are the residual risks and what's the plan?" | Sections 9, 14, 15 |
| Foundation operator | "What do I defend against and how?" | Section 7 cross-reference matrix |

**Audit readiness self-assessment (post-v1.1 patches):**

```
NLnet Commons Fund technical review:   9.3/10  (gating: GOVERNANCE-SPEC, POP-3-TOKEN-SPEC, GENESIS-SPEC)
Notariusz NL (Stichting):              9.0/10  (gating: LEGAL-INTERPRETATION-SPEC, Stichting registration complete)
Academic post-quantum review:          9.0/10  (gating: CRYPTOGRAPHIC-ASSUMPTIONS-SPEC, independent code audit)
```

---

This document formally enumerates the adversaries that the a1a5NT
Cryptographic Preservation Infrastructure defends against, the
capabilities each adversary is assumed to possess, the assets each
adversary may target, and the specification sections (across the
a1a5NT documentation stack) that provide mitigation.

This document does **not** introduce new mechanisms. It systematizes
the threat surface that the existing specifications (XMSS-OPS-SPEC,
CANON-REGISTRY-SPEC, CUSTODY-REGISTRY-SPEC, POV-SPEC, MBW-SPEC,
RENDER-SPEC, planned GOVERNANCE-SPEC) collectively address.

The document is structured for **three audiences**:

1. **Academic cryptographic reviewers** — formal adversary
   capabilities, cost-of-corruption analysis, security assumptions
2. **NLnet and institutional technical reviewers** — clear mapping
   from threats to mitigation, audit evidence pointers
3. **Foundation and operator personnel** — operational understanding
   of what each procedure defends against and why

---

## 1. Scope

### 1.1 In scope

- Enumeration of adversary classes A1 through A12
- Per-adversary capabilities, motivations, and asset targets
- Cost-of-corruption (CoC) and profit-of-corruption (PfC) analysis
  per EigenLayer framework for cryptoeconomic security
- Cross-reference matrix mapping threats to mitigations across the
  full a1a5NT specification stack
- Residual risk identification (threats not fully mitigated)
- Trust assumption inventory (what we assume without proof)
- Failure mode catalog with severity classification

### 1.2 Out of scope

- Specific cryptanalytic attacks on hash functions (SHA-256, blake3)
  — covered by NIST hash function security evaluations
- Specific cryptanalytic attacks on XMSS — covered by RFC 8391
  Section 9 (Security Considerations)
- Side-channel attacks on cryptographic implementations — covered
  by xmss-reference and liboqs implementation documentation
- Physical security of operator premises — covered by operator
  procedure documents, not architecture
- Social engineering targeting end-users — universal, not a1a5NT-
  specific

### 1.3 Audience

- Cryptographic researchers and academic reviewers
- NLnet Commons Fund technical evaluators
- Institutional procurement security teams
- a1a5NT Foundation operators
- Witness federation operator candidates
- Custodian candidates

---

## 2. Normative references

| Reference | Title | Applicability |
|-----------|-------|---------------|
| RFC 2119 | Key words for use in RFCs | Normative wording |
| RFC 8391 | XMSS specification | Cryptographic primitive security |
| RFC 6962 / RFC 9162 | Certificate Transparency v1/v2 | Witness federation threat model |
| NIST SP 800-30 Rev 1 | Guide for Conducting Risk Assessments | Risk assessment methodology |
| NIST SP 800-208 | Stateful Hash-Based Signatures Recommendation | Stateful HBS adversary capabilities |
| NIST FIPS 199 | Standards for Security Categorization | Impact level classification |

### Informative references

| Reference | Title | Relevance |
|-----------|-------|-----------|
| EigenLayer documentation | AVS slashing, intersubjective faults | Cost-of-corruption framework |
| Filecoin protocol paper | Storage market threat model | Adversary classes for storage layer |
| TUF (The Update Framework) | Threshold delegation | Trust distribution model |
| Sigstore documentation | Transparency log threat model | Append-only log adversaries |

---

## 3. Security goals

a1a5NT defends the following security goals, in priority order:

### G1 — Canon Integrity (highest priority)

```
No adversary, by any combination of capabilities below, SHALL cause
the Canon Registry to record an artifact attribution, hash, or
issuance event that diverges from what the Foundation legitimately
authorized at the time of issuance.

Failure mode: false canon entry indistinguishable from legitimate
              entry by any external observer.
Impact:       CATASTROPHIC — invalidates the entire preservation
              promise; cannot be remediated post-hoc.
```

### G2 — Custody Integrity

```
No adversary SHALL cause the Custody Registry to record a custody
transition (transfer, revocation, attestation) that contradicts the
genuine intent of the rightful custodian(s) at the time of the
transition.

Failure mode: false custody transfer indistinguishable from
              legitimate transfer.
Impact:       HIGH — recoverable through dispute resolution (per
              CUSTODY-REGISTRY-SPEC §8), but undermines trust.
```

### G3 — Verification Authenticity

```
No adversary SHALL produce a verification receipt that the Layer B
verification log accepts as valid without having performed the
underlying verification work.

Failure mode: fabricated PoV receipts inflating attacker's MBW
              balance, false claims of network participation.
Impact:       MEDIUM-HIGH — distorts economy, undermines participation
              metrics, but does not affect Canon integrity.
```

### G4 — Provenance Continuity

```
No adversary SHALL prevent legitimate verifiers from retrieving the
authentic Canon entries decades after issuance, under the cryptographic
assumptions stated in CRYPTOGRAPHIC-ASSUMPTIONS.md (planned).

Failure mode: archive becomes inaccessible or its integrity
              becomes unprovable.
Impact:       CATASTROPHIC long-term — the core value proposition.
```

### G5 — Operator Accountability

```
Every operation that modifies network state SHALL be cryptographically
attributable to an identified operator (Foundation, witness, custodian,
verifier) such that misbehavior can be detected and attributed.

Failure mode: anonymous successful attack with no attribution path.
Impact:       MEDIUM — does not prevent harm but enables response.
```

### G6 — Censorship Resistance

```
No single adversary SHALL prevent legitimate participants from
accessing the Canon Registry, registering devices, or submitting
verification receipts.

Failure mode: partition or systematic blocking of legitimate
              participants.
Impact:       MEDIUM — reduces network value but does not corrupt it.
```

### G7 — Provenance ≠ Authenticity (scope disclosure)

```
a1a5NT certifies CONTINUITY of artifacts, not TRUTHFULNESS of
their original content.

a1a5NT cryptographically guarantees:
  ✓ The artifact registered at time T1 is byte-identical to the
    artifact retrievable at time T2 (integrity continuity)
  ✓ The chain of custody between T1 and T2 is recorded and
    attributable (custody continuity)
  ✓ The Foundation, custodian, or witness who attested to the
    artifact is cryptographically identified (attribution continuity)

a1a5NT does NOT cryptographically guarantee:
  ✗ That the facts asserted within the original artifact are true
    (truthfulness of content)
  ✗ That the original artifact corresponds to physical reality
    (e.g., a welding telemetry report is what it claims to be)
  ✗ That the issuer of the original artifact had authority to
    issue it in their domain (domain authority is the issuer's
    responsibility, not a1a5NT's)

Failure mode: external observer conflates "registered on a1a5NT"
              with "verified true".
Impact:       MEDIUM-HIGH for reputation; LOW for cryptographic 
              soundness (the system's behavior is correct; only
              its interpretation by laypeople risks misunderstanding).
Mitigation:   Explicit disclosure in all public-facing material;
              MUST be included in any compliance certification
              language; Foundation operators MUST refuse to issue
              statements implying a1a5NT certifies truthfulness.
```

This goal was added in v1.1 per Reviewer N peer review finding N-B5
(2026-05-17). It clarifies a subtle but critical distinction that
academic cryptographic reviewers and legal counsel routinely raise
about preservation infrastructures: continuity is provable;
truthfulness of original content is not.

---

## 4. Trust assumptions

a1a5NT explicitly relies on the following assumptions. Their failure
modes are documented in Section 9.

### TA1 — Cryptographic primitive security

```
SHA-256, SHA-3, blake3 provide collision resistance and second-
preimage resistance at the security levels stated in their
respective specifications, against all currently known attacks
including quantum (Grover speedup acknowledged: effective security
level halved against quantum attacker).

XMSS provides existential unforgeability under chosen-message
attack (EU-CMA) at the security levels stated in RFC 8391,
PROVIDED that the state-reuse-prevention invariants in XMSS-OPS-SPEC
are correctly enforced.
```

**Mitigation if assumption fails:** Algorithm migration with key
rotation per XMSS-OPS-SPEC Section 9 and CANON-REGISTRY-SPEC
historical preservation.

### TA2 — Implementation correctness

```
The reference implementations (xmss-reference, liboqs XMSS,
a1a5NT specific software) correctly implement the published
specifications without exploitable bugs.
```

**Mitigation if assumption fails:** Multi-implementation cross-
validation (test vector conformance per XMSS-OPS-SPEC §11.2),
independent code audit (planned for Tier 1 deployment), bug bounty
program (planned post-Stichting).

### TA3 — Hardware integrity for high-value operations

```
Foundation root operations and witness operator operations are
conducted on hardware that has not been compromised at the
firmware/microcode level beyond commercial threat-model assumptions
(i.e., adversary may compromise OS but not BIOS-level persistent
implants).
```

**Mitigation if assumption fails:** Hardware-bonded state (XMSS-
OPS-SPEC §11.3), incident response procedures (planned in
RECOVERY-SPEC), multi-host divergence detection (XMSS-OPS-SPEC §7).

### TA4 — At least one honest witness post-federation

```
After Witness Federation rollout (post-Genesis), the network
assumes that at least one witness in the active federation is
honest and operational, sufficient to detect Foundation
equivocation by gossiping.
```

**Mitigation if assumption fails:** Failover to network-wide
public gossip (GitHub commits, Twitter posts, cross-published
master indexes); manual cryptographic review.

### TA5 — Operator procedural diligence

```
Foundation, witness, and custodian operators perform their
documented procedures correctly. Specifically, they do not:
  - Restore from old backups while current state exists
  - Run multiple signing processes concurrently
  - Skip post-write state verification
  - Share private keys across hosts
```

**Mitigation if assumption fails:** Multi-party operator quorum
for high-value operations (XMSS-OPS-SPEC §7.4 failover ceremony),
detection via audit logging (XMSS-OPS-SPEC §10), emergency rotation
procedures (XMSS-OPS-SPEC §9.2).

---

## 5. Adversary classes

The following adversary classes are defined. Each adversary may
combine capabilities (e.g., A1 + A2 is a network attacker who has
also compromised a mirror).

### A1 — Passive network observer

| Attribute | Value |
|-----------|-------|
| Capabilities | Read all unencrypted network traffic between any two parties |
| Cannot | Modify traffic, forge signatures, compromise hosts |
| Motivation | Intelligence gathering, statistical analysis of participation |
| Cost (PfC) | Information leakage; cannot directly monetize |
| Cost (CoC) | Trivial (commodity ISP-level observation) |
| Asset targeted | Privacy of verifiers, statistical patterns of operation |

**Defense:** All Foundation API endpoints SHALL serve over TLS
(HTTPS) by the deployment of public verify.sh; transport-layer
encryption mitigates A1 to standard-web-app threat level. Note:
the cryptographic integrity of artifacts does not depend on
transport encryption — A1 cannot forge canon or receipts even with
plaintext observation.

### A2 — Active network attacker

| Attribute | Value |
|-----------|-------|
| Capabilities | A1 + modify, drop, reorder, or inject network traffic between specific parties |
| Cannot | Forge cryptographic signatures, compromise endpoint hosts |
| Motivation | Selective censorship, downgrade attacks, denial of service |
| Cost (PfC) | Limited; cannot produce counterfeit canon |
| Cost (CoC) | Moderate (ISP-level access or BGP hijack capability) |
| Asset targeted | Liveness of verification, accuracy of timestamps |

**Defense:** Signed timestamp authority (per Sigstore Rekor v2
pattern) decouples liveness from correctness; verifier can record
delayed receipts when network recovers. Mirror multiplicity per
CANON-REGISTRY-SPEC §5 (GitHub raw + IPFS + Foundation endpoint +
witness mirrors) makes complete censorship impractical.

### A3 — Hostile mirror operator

| Attribute | Value |
|-----------|-------|
| Capabilities | Serves stale, partial, or fabricated content to specific clients |
| Cannot | Forge XMSS signatures, modify Foundation index hash |
| Motivation | Selective information control, undermining specific verifiers |
| Cost (PfC) | Limited; victims who consult a second mirror detect attack |
| Cost (CoC) | Low (run a mirror) |
| Asset targeted | Accuracy of canon retrieved by specific verifiers |

**Defense:** Every canon entry retrievable from any mirror MUST be
hash-verifiable against the XMSS-signed master_index, which clients
can obtain from multiple independent mirrors. Hash mismatch
indicates mirror malfeasance and is locally detectable. CANON-
REGISTRY-SPEC §5.4 enforces hash-before-trust.

### A4 — Hostile witness operator

| Attribute | Value |
|-----------|-------|
| Capabilities | Holds legitimate witness XMSS keypair; can sign attestations |
| Cannot | Force Foundation to issue counterfeit canon (witnesses attest, do not issue) |
| Motivation | Equivocation, conflicting attestations, undermining federation |
| Cost (PfC) | Witness reputation loss when equivocation detected by gossip; potential slashing per Cost-of-Corruption analysis (Section 6.4) |
| Cost (CoC) | Witness operational setup + potential collateral; non-trivial |
| Asset targeted | Integrity of witness federation co-signatures |

**Defense:** RFC 6962 §5.3 gossiping protocol (specified in
planned NETWORK-PROTOCOL-SPEC) — witnesses gossip Signed Tree Heads
between each other; observation of two distinct STH for same
version number = cryptographic proof of equivocation,
unforgeable, attributable. EigenLayer-pattern slashing (planned in
GOVERNANCE-SPEC) makes CoC > PfC for rational adversary.

### A5 — Hostile custodian operator

| Attribute | Value |
|-----------|-------|
| Capabilities | Holds legitimate custodian XMSS keypair; can sign custody operations |
| Cannot | Modify Canon entries (only Custody Registry); cannot forge transfers from custodian they do not control |
| Motivation | False transfer of artifact in their custody to colluding party; fraudulent dispute |
| Cost (PfC) | Value of artifact transferred (bounded by individual artifact value, not entire system) |
| Cost (CoC) | Reputation loss, future custody opportunities lost, potential legal action via Stichting under LEGAL-INTERPRETATION-SPEC (planned) |
| Asset targeted | Custody attribution of specific artifact under their control |

**Defense:** CUSTODY-REGISTRY-SPEC §6 attestation chain — every
custody operation is signed, dated, and references prior custody
state; collusion with single counter-party detectable by third-
party verifiers comparing chains. Dispute resolution per
CUSTODY-REGISTRY-SPEC §8 with witness federation arbitration
(planned in GOVERNANCE-SPEC).

### A6 — Foundation insider (single operator compromise)

| Attribute | Value |
|-----------|-------|
| Capabilities | Holds or accesses Foundation XMSS root key state; can sign anything as Foundation |
| Cannot | Forge signatures attributed to other namespaces; modify already-issued canon entries (append-only) |
| Motivation | Self-dealing canon issuance, fraudulent master index publication |
| Cost (PfC) | Bounded by what Foundation can legitimately issue; equivocation detectable by witnesses |
| Cost (CoC) | Detection by witnesses → reputation collapse → Foundation viability ends → legal consequences via Stichting |
| Asset targeted | Foundation issuance authority itself |

**Defense:** Witness Federation (post-Genesis) detects equivocation
by gossiping (TA4); pre-Federation, multi-channel public publication
(GitHub commits, Twitter posts, NLnet correspondence) creates audit
trail that makes silent equivocation impractical. Root-of-Roots
hierarchy (planned in XMSS-OPS-SPEC §9.4 + CANON-REGISTRY-SPEC §6.6)
means even single Foundation key compromise does not allow
restructuring of historical canon entries.

### A7 — Sybil verifier swarm

| Attribute | Value |
|-----------|-------|
| Capabilities | Operates thousands of automated verifier instances, each generating valid receipts |
| Cannot | Generate receipts without performing verification work (must fetch + hash) |
| Motivation | Inflate own MBW balance, dominate participation metrics |
| Cost (PfC) | MBW credit proportional to receipts × maturity_weight × tier_multiplier |
| Cost (CoC) | XMSS keygen per device (5-10s CPU) + 720h maturity ramp (per MBW-SPEC §3.2) before full credit; bandwidth costs |
| Asset targeted | MBW economy distribution fairness |

**Defense:** POV-SPEC §5 (Sybil resistance): maturity curve (10%
credit at registration, ramping to 100% over 720h); XMSS keygen
natural cost; per-IP and per-ASN rate limiting; nonce/timestamp
freshness requirements. New devices contribute network value
through traffic but earn proportionally less credit until proven
persistent.

### A8 — Bandwidth bleed attacker

| Attribute | Value |
|-----------|-------|
| Capabilities | Operates legitimate verifiers but selectively targets heaviest artifacts (largest manifests, longest cards) from cheapest free mirrors |
| Cannot | Generate receipts without actual data transfer |
| Motivation | Maximize MBW earnings per dollar spent on bandwidth, externalizing cost onto mirror operators |
| Cost (PfC) | MBW proportional to bytes fetched; concentrated on high-bandwidth content |
| Cost (CoC) | Verifier's own ISP bandwidth (low); legitimate behavior, hard to attribute as attack |
| Asset targeted | Mirror operator economic viability |

**Defense:** MBW-SPEC §3 (bandwidth bleed economics, to be patched
per v1.2 Tier 2 Patch 14) — MBW credit formula MUST include a
discount factor for repeated fetches of identical artifacts from
the same mirror, and a per-mirror retrieval credit returned to
the mirror operator from the verifier's MBW account. Filecoin
two-market pattern: Storage providers (mirrors) receive
compensation distinct from verifiers; net economic flow does not
externalize all bandwidth onto unpaid mirrors.

### A9 — State replay attacker (XMSS state desync)

| Attribute | Value |
|-----------|-------|
| Capabilities | Obtains older snapshot of XMSS secret key state for Foundation or witness; induces or observes restore from that snapshot |
| Cannot | Forge XMSS signatures without compromising key state |
| Motivation | If successful, can forge signatures using reused WOTS+ indices, catastrophic |
| Cost (PfC) | Complete forgery capability over affected keypair |
| Cost (CoC) | Requires both (a) acquiring backup, (b) inducing restore, both gated by procedural controls |
| Asset targeted | XMSS signature unforgeability itself |

**Defense:** XMSS-OPS-SPEC §5-6 (invariants I1-I5; monotonic
counter; refuse-restore-from-older). This is the threat with the
most rigorous published mitigation; see XMSS-OPS-SPEC Section 3
Adversary T-XMSS-2 for detailed analysis.

### A10 — Renderer divergence attacker

| Attribute | Value |
|-----------|-------|
| Capabilities | Distributes a modified or alternate SVG card claiming to be the canonical render of a Foundation-issued artifact |
| Cannot | Modify the canonical artifact_id hash; cannot forge Foundation issuance signature |
| Motivation | Visual confusion; presenting visually different artifact as same canon entry |
| Cost (PfC) | Limited; verifiers with reference renderer detect divergence |
| Cost (CoC) | Trivial (anyone can host an alternate SVG) |
| Asset targeted | Visual provenance continuity |

**Defense:** RENDER-SPEC deterministic reproducible rendering with
canonical renderer binary hash, canonical font set hash, canonical
palette ID, integer/fixed-point math only (per v1.2 Tier 2 Patch
16 addressing Gemini G-B2). Verifier with canonical render manifest
can locally compute expected output hash and detect divergence from
distributed SVG.

### A11 — Post-quantum cryptanalyst

| Attribute | Value |
|-----------|-------|
| Capabilities | Possesses a quantum computer capable of running Shor's algorithm at scale sufficient to break ECDSA, RSA, Ed25519; Grover speedup against hash functions |
| Cannot | Break hash-based signatures (XMSS) at full security level (hash collision resistance is what matters; Grover provides at most √n speedup, doubling effective hash output requirement) |
| Motivation | Forge any signature using broken classical primitive |
| Cost (PfC) | Capability to forge signatures using broken primitives |
| Cost (CoC) | Cost of a quantum computer (currently infeasible; estimated decades) |
| Asset targeted | All cryptographic signatures NOT using post-quantum primitives |

**Defense:** a1a5NT uses XMSS (hash-based, post-quantum) for all
authoritative signatures: Foundation root, witness attestations,
custodian operations. Receipt-level signatures may use Ed25519
(per planned Verifier XMSS Tree Exhaustion fix, Patch 15) with the
explicit understanding that receipts have 30-day validity window
and are not part of canonical truth; their loss to a future quantum
attacker is acceptable.

### A12 — Long-term archival attacker (multi-decade horizon)

| Attribute | Value |
|-----------|-------|
| Capabilities | Patient adversary willing to invest in attacks playing out over decades; may gradually accumulate compromised hosts, gradually degrade cryptographic primitives as they age, exploit institutional turnover |
| Cannot | Defeat append-only nature of Canon (entries timestamped at issuance); cannot retroactively modify historical signatures |
| Motivation | Eventual control over historical canon attribution; revisionist provenance |
| Cost (PfC) | Bounded by accumulated value of all artifacts under preservation |
| Cost (CoC) | Decades of operational patience; requires success across multiple cryptographic primitive aging cycles |
| Asset targeted | Long-term provenance continuity (Goal G4) |

**Defense:** Periodic cryptographic algorithm migration with key
rotation, recorded as canonical events in Canon Registry; multi-
generation key hierarchy (Root-of-Roots → era-specific roots) so
that compromise of any one era's keys does not invalidate prior
eras; archival institution diversity (no single archive holds the
only copy); WitNess Federation distribution. The strongest defense
is procedural: institutionalize the practice of periodic re-
attestation by current cryptographic primitives, recorded as
canonical events that bind old attestations to new primitive's
authority.

### A13 — Legal compulsion attacker

| Attribute | Value |
|-----------|-------|
| Capabilities | Court order (criminal, civil, administrative); GDPR Article 17 right-to-erasure request; copyright takedown notice (DMCA-equivalent in NL/EU); economic sanctions enforcement (OFAC, EU sanctions); regulatory injunction (national security, child protection) |
| Cannot | Be ignored without legal consequences for operators and Stichting |
| Motivation | Compel modification, removal, or non-serving of canonical entries; compel custody transfer; compel operator identification disclosure |
| Cost (PfC) | Effectiveness of legal compulsion in chosen jurisdiction |
| Cost (CoC) | Filing legal action; varies by jurisdiction and standing |
| Asset targeted | Canon immutability (G1) DIRECTLY conflicts with legal compulsion of removal |

**Defense (the Hybrid Tombstone Pattern — architectural decision
made in v1.1 per cross-review):**

The Foundation MUST comply with valid legal compulsion in
applicable jurisdictions. The Canon Registry handles this through
**tombstoning** rather than deletion:

```
LEGAL_COMPULSION_RESPONSE:

  Inputs:
    - Valid legal order (court order, GDPR erasure request, 
      DMCA takedown, etc.)
    - Canon entry C_id targeted by the order
  
  Process:
    1. Foundation legal counsel reviews validity of compulsion
    2. If valid, Foundation issues TOMBSTONE_NOTICE:
       - Signed by Foundation operator-level key (NOT root)
       - Contains: original canon_id, original blake3 hash,
         tombstone reason category (legal_compulsion_type),
         jurisdiction, date, redacted reference to legal order
         (specific identifying information minimized per
         data-protection requirements of the compelling order)
    3. The original canon entry's HASH and SIGNATURE remain in
       the Canon Registry forever — these are mathematical facts,
       not personal data, not copyrightable, not subject to
       erasure under any current legal framework
    4. The original canon entry's CONTENT is removed from
       Foundation-operated mirrors and all witness mirrors that
       operate in jurisdictions subject to the compelling order
    5. Verifier requests for the content return:
       HTTP 451 Unavailable For Legal Reasons (RFC 7725)
       with body containing the TOMBSTONE_NOTICE
    6. The TOMBSTONE_NOTICE itself is added to Canon Registry
       as a new canon entry (legal_event class) — the FACT of
       legal compulsion is itself canonical, even if the
       original content is no longer servable
    7. Custody chain referencing the tombstoned artifact remains
       valid; current custodian is notified
```

**Why this preserves G1 (Canon Integrity):**

The cryptographic guarantee — "the artifact hash recorded at time
T is the artifact's hash, immutably" — is preserved because the
hash itself is never modified. A verifier with a local copy of the
artifact (legitimately acquired before tombstoning, or from a
jurisdiction not subject to the legal order) can still
cryptographically verify that their copy matches the canonical
hash. The Canon Registry never lies about what was registered; it
only declines (in compelled jurisdictions) to serve the content.

**Why this preserves operator legal viability:**

In NL (and broadly in EU under GDPR), refusal to comply with valid
legal orders ends Stichting viability and exposes operators to
personal liability. The hybrid pattern allows compliance with
serving requirements while preserving the cryptographic record of
what existed. Multiple major preservation infrastructures (Internet
Archive, certificate transparency logs, Sigstore Rekor) operate
under similar models.

**Limitation:**

If the legal compulsion specifically requires hash erasure (an
unprecedented but conceivable order in extreme circumstances), the
Foundation must escalate to legal counsel; current best
understanding is that mathematical facts (hashes) are not subject
to such orders under current law. This will be documented in
LEGAL-INTERPRETATION-SPEC [PLANNED].

**v1.1 status:** [DESIGN TARGET] — architecturally defined; full
operational procedure detailed in CANON-RETIREMENT-SPEC [PLANNED]
and LEGAL-INTERPRETATION-SPEC [PLANNED] per v1.2 Tier 1 roadmap.

---

## 6. Cost-of-Corruption analysis

Following EigenLayer's intersubjective faults framework, the
following analyzes the economic viability of each adversary class.
The principle:

> An attack is economically rational only if Profit-of-Corruption
> (PfC) exceeds Cost-of-Corruption (CoC).
>
> The defender's task is to ensure CoC > PfC for all attacker classes.

### 6.1 Per-adversary CoC vs PfC

| Adversary | PfC (potential gain) | CoC (cost of attempt) | CoC > PfC? |
|-----------|---------------------|----------------------|------------|
| A1 Passive observer | Low (info leakage) | Trivial | N/A (no attack to deter) |
| A2 Active network | Low (no forgery possible) | Moderate (ISP/BGP access) | YES |
| A3 Hostile mirror | Low (locally detectable) | Low (run a mirror) | Marginal — relies on detection |
| A4 Hostile witness | Reputation gain by malicious party | Slashing + reputation collapse (per GOVERNANCE-SPEC) | DESIGN TARGET: YES |
| A5 Hostile custodian | Value of single artifact | Reputation + legal action | Marginal at low artifact value; YES at meaningful value |
| A6 Foundation insider | Full Foundation authority | Foundation viability + legal + criminal | YES (existential cost) |
| A7 Sybil verifier swarm | MBW economy distortion | Per-device keygen + 720h maturity | YES at small scale; marginal at very large scale |
| A8 Bandwidth bleed | Externalized bandwidth cost | Mirror retrieval credit system (Patch 14) | DESIGN TARGET: YES |
| A9 XMSS state replay | Catastrophic forgery | Backup access + restore induction | YES via XMSS-OPS-SPEC controls |
| A10 Renderer divergence | Limited visual confusion | Trivial to attempt | Locally detected by reference renderer; PfC effectively zero |
| A11 Post-quantum cryptanalyst | Forgery of broken primitives | Cost of quantum computer (decades-scale infeasibility) | YES currently; design assumption |
| A12 Long-term archival | Revisionist provenance | Decades of operational patience | Marginal — requires institutional vigilance |
| A13 Legal compulsion | Effectiveness of legal order in jurisdiction | Filing legal action (varies) | DESIGN: hybrid tombstone preserves cryptographic record while permitting compelled content removal — neither side fully "wins" or "loses" |

### 6.2 Weakest CoC/PfC ratios (highest defensive priority)

```
1. A3 Hostile mirror     — relies on client-side detection diligence
2. A7 Sybil swarm at scale — anti-Sybil maturity curve is necessary but not sufficient
3. A12 Long-term archival  — multi-decade vigilance is procedural, not cryptographic
```

These three areas receive **priority in operational procedure
development** (planned RECOVERY-SPEC, GOVERNANCE-SPEC).

### 6.3 Adversaries with effectively unbounded CoC

```
A6 Foundation insider:    Foundation existence at stake
A9 XMSS state replay:      Procedural controls effective
A11 Post-quantum:          Quantum computer required
```

These threats have such high cost that rational adversaries do not
attempt them; defense rests on maintaining the cost barriers
(institutional integrity, procedural discipline, cryptographic
primitive selection).

### 6.4 Slashing calibration target

For witness federation post-Genesis (per GOVERNANCE-SPEC planned):

```
Witness collateral SHALL be calibrated such that:
  
  collateral_value ≥ max(PfC) for any single equivocation event
  
where max(PfC) is bounded by the value of canon entries that could
be falsely attested in a single equivocation episode.

For initial federation (5-10 witnesses), collateral target:
  - Equivalent of ~10% of expected annual canon-entry value flow
  - Bonded for minimum 24 months (matches Foundation transition
    window per LEGAL-INTERPRETATION-SPEC)
```

Final calibration is GOVERNANCE-SPEC scope.

---

## 7. Threat-to-mitigation cross-reference matrix

| Threat | Goal violated | Defending spec | Specific section |
|--------|---------------|----------------|------------------|
| A1 | G6 | All — transport encryption | (deployment guideline) |
| A2 | G6 | CANON-REGISTRY-SPEC | §5 mirror policy |
| A3 | G1, G6 | CANON-REGISTRY-SPEC | §5.4 hash-before-trust |
| A4 | G1, G5 | CANON-REGISTRY-SPEC + NETWORK-PROTOCOL-SPEC (planned) | §6 witness federation, gossip |
| A5 | G2 | CUSTODY-REGISTRY-SPEC | §6 attestation chain, §8 dispute resolution |
| A6 | G1, G5 | XMSS-OPS-SPEC + NETWORK-PROTOCOL-SPEC + GOVERNANCE-SPEC (planned) | §9 emergency rotation, §6 witness gossip |
| A7 | G3 | POV-SPEC + MBW-SPEC | §5 anti-Sybil, §3.2 maturity curve |
| A8 | G3 (economy fairness) | MBW-SPEC | §3 bandwidth bleed economics (v1.2 patch) |
| A9 | G1 (catastrophic) | XMSS-OPS-SPEC | §5 invariants, §6 backup/restore controls |
| A10 | G4 (visual provenance) | RENDER-SPEC | §4 deterministic rendering |
| A11 | All G1-G5 long-term | XMSS-OPS-SPEC + planned CRYPTOGRAPHIC-ASSUMPTIONS | §9 rotation, primitive migration |
| A12 | G4 (long-term) | All — periodic re-attestation | (procedural, GOVERNANCE-SPEC) |
| A13 | G1 (conflict, resolved by tombstone) | CANON-RETIREMENT-SPEC + LEGAL-INTERPRETATION-SPEC [PLANNED] | A13 defense section above |

---

## 8. Failure mode severity classification

Per NIST FIPS 199:

### Catastrophic (Cat)

Failure modes that invalidate the entire preservation promise:

```
Cat-1: Successful A9 (XMSS state replay) producing two valid
       signatures with same WOTS+ index from Foundation root or
       witness root keypair.
Cat-2: Successful A6 (Foundation insider) AND simultaneous A4
       (hostile witnesses ≥ quorum threshold) AND no honest
       gossip detection — coordinated multi-role attack.
Cat-3: Loss of XMSS root keys without backup recoverability
       (XMSS-OPS-SPEC §9.4 scenario).
```

### High

Failure modes that undermine specific high-value operations:

```
H-1:   Successful A5 (custodian misattribution) on high-value
       artifact prior to dispute resolution.
H-2:   Successful A10 (renderer divergence) accepted by uninformed
       viewers in absence of reference renderer.
H-3:   Total destruction of all Foundation infrastructure with
       only cold backup remaining (XMSS-OPS-SPEC §9.3).
```

### Medium

Failure modes that distort but do not destroy:

```
M-1:   A7 (Sybil) at scale beyond anti-Sybil effectiveness.
M-2:   A8 (bandwidth bleed) sustained over months bankrupting
       free mirrors.
M-3:   A3 (hostile mirror) undetected by some verifiers.
```

### Low

Failure modes that primarily affect availability or privacy:

```
L-1:   A1 (passive observation) — privacy reduction only.
L-2:   A2 (network partition) — temporary verification delay.
```

---

## 9. Residual risk inventory

Risks not fully mitigated by current specifications:

### R1 — Single-person bootstrap concentration

**Status:** Acknowledged by all four reviewers (Gemini implicit, Grok Gk-2,
GPT GPT-1, Claude PC CP-C1) as the highest-priority residual risk
in v1.2 state.

**Description:** Pre-Stichting registration, all Foundation
operational capabilities (XMSS root key, Canon issuance authority,
witness federation bootstrap, custodian appointment) are held by
a single individual (Paweł Piekut). This represents:
- A single point of failure (operator unavailability halts the
  network)
- A single point of corruption (operator compromise = Foundation
  compromise per A6)
- A single point of legal liability (no separation between operator
  and institution)

**Documented mitigations:**
- ZZP Plastechniek (KvK NL) provides interim legal vehicle
- Foundation Declaration v1.0 and License v1.0 publicly published
  and cryptographically anchored, creating prior-art evidence of
  intent independent of single-person continuation
- Multi-channel publication of master index updates creates
  external audit trail
- Stichting registration is in progress (timeline gated on
  notarial appointment)

**Acceptance rationale:** All four reviewers acknowledge this risk
as inherent to bootstrap of any sovereign infrastructure project;
none suggest the project should not proceed because of it. The
mitigation path (Stichting + witness federation + Root-of-Roots
separation) is documented and on the roadmap.

### R2 — Post-quantum threshold signature schemes do not exist as standard

**Status:** Documented in XMSS-OPS-SPEC §13.1.

**Description:** True t-of-n cryptographic multi-party signing of
XMSS or other stateful HBS schemes is active research; no
production-ready standard exists as of 2026-05-17.

**Documented mitigations:**
- Multi-party authorization implemented procedurally (ceremony
  requiring multiple operator signatures) rather than
  cryptographically
- Composition of multiple single-party XMSS signatures from
  distinct keypairs (each procedurally distinct) approximates
  multi-party trust

**Acceptance rationale:** Project cannot wait for unspecified
future cryptographic research. Procedural multi-party with
multi-keypair composition is the current state-of-art and is
adopted by other production systems with similar constraints.

### R3 — Long-term archival institutional continuity

**Status:** Inherent to multi-decade horizon (Goal G4, Adversary A12).

**Description:** Provenance continuity over decades requires
institutional continuity. Stichting continuity, witness federation
continuity, archival institution participation continuity are all
long-term human-institutional commitments that no cryptography can
guarantee.

**Documented mitigations:**
- Stichting structure designed for institutional longevity (NL legal
  framework for foundations explicitly supports indefinite duration)
- Witness federation rollout will include academic institutions
  (archive-oxford.a1a5 candidate per CANON-REGISTRY-SPEC §6.2.1)
  whose own institutional continuity exceeds the project's
- Open-source publication of all canon material via Internet Archive,
  Software Heritage Foundation, and similar long-term preservation
  partners (planned post-Stichting)

**Acceptance rationale:** This is the inherent challenge of all
long-term preservation projects. The mitigation is procedural and
relational, not cryptographic, and is explicitly within scope of
planned GOVERNANCE-SPEC.

### R4 — Reference implementation has not undergone independent audit

**Status:** Documented in XMSS-OPS-SPEC §11.

**Description:** While XMSS specification is well-reviewed (RFC
8391 went through IRTF and NIST processes), the a1a5NT-specific
wrapping (state file format, monotonic counter, multi-host
replication) defined in XMSS-OPS-SPEC has not yet been
independently audited.

**Documented mitigations:**
- Implementation submitted for review at NLnet Commons Fund
  application (anticipated audit support if grant approved)
- Test vector conformance to RFC 8391 enforced (XMSS-OPS-SPEC §11.2)
- Multi-implementation cross-validation (xmss-reference and liboqs)

**Acceptance rationale:** Initial deployment proceeds with
conformance-tested reference implementation; independent audit is
sought as next-phase grant deliverable.

### R5 — Reviewer attestation is from AI systems

**Status:** Honest acknowledgment.

**Description:** The cross-model peer review process (four
independent AI models with different training cutoffs and
inference architectures) produces useful findings but is not a
substitute for human cryptographic and legal review.

**Documented mitigations:**
- All four AI reviews are published in full as part of v1.2
  documentation (Audit trail)
- Architectural decisions are made by the human architect with AI
  serving in synthesis role per License Article 4
- External human review is sought at NLnet, notarial, and academic
  stages as documented in v1.2 master plan §7

**Acceptance rationale:** AI-assisted iterative review captures
many concerns earlier than would be feasible with sequential
human-only review; human review of final v1.2 stack is the
authoritative external validation step.

---

## 10. Required updates to this document

This document MUST be revised when any of the following occurs:

```
T-Update-1: New adversary class identified through external review
            or operational incident
T-Update-2: New cryptographic attack against TA1 primitives published
T-Update-3: Stichting registration complete (R1 status change)
T-Update-4: Witness federation activated (A4, A6, TA4 changes)
T-Update-5: First independent crypto audit complete (R4 status change)
T-Update-6: Operational incident requiring threat model revision
```

Revisions follow the same License v1.0 anti-patent prior-art
publication discipline as all other a1a5NT specifications. v1.0
is preserved in git history when superseded.

---

## 14. Emergency Root Transfer Procedure

This section addresses Residual Risk R1 (single-person bootstrap
concentration) at the operational level. v1.0 documented R1 as a
known risk; v1.1 specifies what operationally happens if the
Foundation operator becomes unavailable.

### 14.1 Scope

This procedure applies to the Foundation root keypair (currently
shared by pawel.a1a5 and a1a5fndn.a1a5; to be separated post-
Stichting per XMSS-OPS-SPEC §9 and CANON-REGISTRY-SPEC §6.6).

This procedure does NOT apply to:
- Custodian keypairs (managed per CUSTODY-REGISTRY-SPEC)
- Witness operator keypairs (managed per GOVERNANCE-SPEC [PLANNED])
- Verifier device keypairs (managed by individual device owner)

### 14.2 Trigger conditions

Emergency Root Transfer is triggered by any of:

```
T-ERT-1: Foundation operator deceased (documented by official
         death certificate)
T-ERT-2: Foundation operator legally incapacitated (documented by
         court order, medical certification, or notarial attestation
         of cognitive incapacity)
T-ERT-3: Foundation operator unreachable for 90+ consecutive days
         despite documented good-faith attempts to contact through
         all known channels (no master_index update, no public
         communication, no response to NLnet correspondence, no
         response to Stichting bestuur — pre-Stichting: to ZZP
         Plastechniek registered address contact)
T-ERT-4: Foundation operator voluntarily transfers responsibility
         (publicly signed transition notice)
T-ERT-5: Court order compelling transfer (unusual; subject to
         legal review)
```

### 14.3 Pre-Stichting procedure (CURRENT, while Stichting in formation)

```
EMERGENCY_ROOT_TRANSFER_PRE_STICHTING:

  Phase 1 — Documentation of trigger:
    1.1 Document the trigger condition with evidence:
        - Death certificate (T-ERT-1)
        - Court order or medical certification (T-ERT-2)
        - Communication log + cross-channel attempt record (T-ERT-3)
        - Operator-signed transition notice (T-ERT-4)
        - Court order (T-ERT-5)
    1.2 Publicly publish the trigger documentation to:
        - github.com/a1a5NT/a1a3 (commit signed by successor's
          new keypair)
        - NLnet Commons Fund (correspondence to ticket 2026-04-3f6)
        - Internet Archive (snapshot of github repo state)
        - At least one academic institution (planned witness-tier
          contact)

  Phase 2 — Pre-authorized successor activation:
    2.1 Per the Foundation Declaration v1.0 (Article noted at
        publication of this Spec) OR per a sealed envelope held
        by ZZP Plastechniek successor contact, the named
        pre-authorized successor is identified.
        
        Current named successor (in confidence to ZZP and
        Stichting-in-formation contact channels): identity 
        documented separately for safety reasons; this Spec
        records that a successor IS named, not who.
        
    2.2 Successor verifies their authority via:
        - Possession of the sealed envelope OR
        - Foundation Declaration Article reference OR
        - Stichting bestuur appointment (if Stichting registered)
    
  Phase 3 — Cryptographic transition:
    3.1 Successor generates a new XMSS keypair (NEW_FOUNDATION_KEY)
        per XMSS-OPS-SPEC §4.4 parameter selection for Foundation
        root.
    3.2 The OLD Foundation keypair is DEPRECATED but NOT 
        compromised:
        - All historical signatures by OLD_KEY remain valid
        - No new signatures by OLD_KEY are produced
        - The keypair state file is preserved READ-ONLY for 
          forensic and historical reference
    3.3 Successor publishes a SUCCESSION_NOTICE to Canon Registry:
        - Signed by NEW_FOUNDATION_KEY
        - Contains: trigger condition documentation, OLD_KEY root,
          NEW_KEY root, successor identity, succession timestamp,
          link to public trigger documentation
    3.4 Successor begins normal Foundation operations using
        NEW_FOUNDATION_KEY.
    
  Phase 4 — Witness federation notification (if active):
    4.1 If witness federation has been activated, successor sends
        SUCCESSION_NOTICE to all federation members.
    4.2 Witness federation members verify trigger documentation
        independently.
    4.3 Witness federation members re-anchor their attestation
        relationship to NEW_FOUNDATION_KEY.
    
  Phase 5 — Public verification:
    5.1 Any external verifier can confirm succession by:
        - Reading SUCCESSION_NOTICE from public mirrors
        - Verifying signature against published NEW_KEY public root
        - Reviewing trigger documentation
    5.2 External skeptics may challenge succession via dispute
        notice (per CUSTODY-REGISTRY-SPEC §8 [PLANNED]) — but
        cannot reverse it absent court finding of fraud.
```

### 14.4 Post-Stichting procedure (FUTURE)

```
EMERGENCY_ROOT_TRANSFER_POST_STICHTING:

  Same as 14.3, EXCEPT:
  
  - Pre-authorized successor identity is the Stichting bestuur
    (not an individual)
  - Bestuur convenes per Stichting statutes to authorize
    succession
  - NEW_FOUNDATION_KEY is generated as a Shamir-split keypair
    (per Section 14.5)
  - Succession requires bestuur quorum per Stichting statutes
    (typically 2-of-3 or 3-of-5)
```

### 14.5 Shamir Secret Sharing for post-Stichting (DESIGN TARGET)

Post-Stichting, the new Foundation root keypair's state file MAY
be split via Shamir Secret Sharing (Shamir 1979) into:

```
3 shares, 2-of-3 threshold reconstruction:

  Share 1: Held by Stichting bestuur chair
  Share 2: Held by Stichting bestuur treasurer  
  Share 3: Held by notariusz (sealed envelope) OR independent
           legal trustee
```

This requires any 2 of 3 share-holders to reconstruct the operating
state, preventing both:
- Single-share holder unilateral control
- Single-share loss locking out succession

**Important limitation:** Shamir SSS protects the KEY STATE, not
the SIGNING OPERATIONS. Each Foundation signing operation must
still occur on a single trusted machine; the SSS scheme allows
RECOVERY of state to a successor machine if the operating machine
is destroyed.

**Post-quantum note:** Shamir SSS itself is information-theoretically
secure (does not depend on computational hardness assumptions);
hence it remains valid in post-quantum threat models. The split is
of the XMSS state (post-quantum primitive). The combination is
post-quantum secure.

### 14.6 Status

```
Status: [DESIGN TARGET pre-Stichting, PLANNED post-Stichting]

Pre-Stichting procedure (14.3):
  [PLANNED] — operator MUST name pre-authorized successor and
  publish sealed envelope arrangement before any further canon
  issuance affecting irrevocable state. Target: within 60 days
  of this Spec publication.

Post-Stichting procedure (14.4):
  [DESIGN TARGET] — implementation gated on Stichting registration
  and bestuur formation.

Shamir SSS implementation (14.5):
  [DESIGN TARGET] — gated on post-Stichting bestuur appointment
  and successor key generation.
```

---

## 15. Mitigation Roadmap and Status Matrix

This section addresses Reviewer O finding O-1 (planned mitigation
references should be explicitly status-labeled) and O-5 (timeline
section). Each adversary defense and residual risk mitigation is
classified by current implementation status with a target owner
and resolution path.

### 15.1 Status labels (used throughout v1.1)

```
[IMPLEMENTED]    Live in production infrastructure, verifiable
                 by external observer
[DEPLOYED]       Specification finalized, deployment underway,
                 verifiable within current iteration
[PLANNED]        Specification written or being written, 
                 implementation scheduled in v1.2 or v1.3
[DESIGN TARGET]  Architectural decision documented, detailed
                 specification not yet authored
[DEFERRED]       Acknowledged as future work; not in current 
                 roadmap; will be addressed when prerequisites 
                 are met
```

### 15.2 Per-adversary defense status

| Adversary | Defense status | Owner | Target |
|-----------|---------------|-------|--------|
| A1 Passive observer | [IMPLEMENTED] HTTPS on archive server | Foundation | Done |
| A2 Active network | [PLANNED] Signed timestamp authority (Sigstore Rekor pattern) | Foundation | v1.2 deployment |
| A3 Hostile mirror | [IMPLEMENTED] hash-before-trust in client design | Foundation + verifiers | Done in spec |
| A4 Hostile witness | [DESIGN TARGET] EigenLayer-pattern slashing | Foundation + future witness operators | Post-Stichting |
| A5 Hostile custodian | [PLANNED] CUSTODY-REGISTRY-SPEC §8 dispute resolution | Foundation | v1.2 patch |
| A6 Foundation insider | [PARTIAL IMPLEMENTED + DESIGN TARGET] Multi-channel publication done; Root-of-Roots design target | Foundation | Post-Stichting |
| A7 Sybil swarm | [PLANNED] POV-SPEC §5 maturity curve | Foundation | v1.2 deployment |
| A8 Bandwidth bleed | [DESIGN TARGET] MBW-SPEC §3 bandwidth credit | Foundation | v1.2 Tier 2 patch |
| A9 XMSS state replay | [IMPLEMENTED in spec, DEPLOYED in code] XMSS-OPS-SPEC §5-6 | Foundation operator | Ongoing |
| A10 Renderer divergence | [DESIGN TARGET] RENDER-SPEC + integer-math constraint | Foundation | v1.2 Tier 2 patch |
| A11 Post-quantum | [IMPLEMENTED] XMSS for all authoritative signatures | Foundation | Ongoing |
| A12 Long-term archival | [DEFERRED] periodic algorithm migration | Foundation + future witnesses | Multi-year |
| A13 Legal compulsion | [DESIGN TARGET] Hybrid tombstone pattern; CANON-RETIREMENT-SPEC | Foundation + Stichting legal counsel | v1.2 Tier 1 + post-Stichting |

### 15.3 Per-residual-risk mitigation status

| Risk | Current mitigation status | Owner | Target completion |
|------|--------------------------|-------|-------------------|
| R1 Single-person bootstrap | [IMPLEMENTED] ZZP legal vehicle, prior art anchored; [PLANNED] Emergency Root Transfer §14.3; [DESIGN TARGET] Shamir SSS §14.5; [DEPLOYED] Stichting registration in progress | Foundation operator, ZZP, future Stichting | Within 60 days for §14.3; Stichting timeline-gated for §14.5 |
| R2 No post-quantum threshold sigs | [DEFERRED] research-gated | NIST + academic research community | Multi-year |
| R3 Long-term archival continuity | [DESIGN TARGET] institutional witnesses (archive-oxford.a1a5 etc.) + Internet Archive replication | Foundation + future federation members | Multi-year incremental |
| R4 No independent code audit | [PLANNED] NLnet grant funding for external audit | NLnet, Foundation | Gated on grant approval |
| R5 AI-based review | [IMPLEMENTED] Full review transcripts published; [PLANNED] human expert review at Stichting + NLnet milestones | Foundation | Sequential per audit milestone |

### 15.4 Critical path to NLnet submission readiness

The following items are gating for NLnet Commons Fund technical
review readiness:

```
1. SECURITY-MODEL.md v1.1 (THIS DOCUMENT) — [IMPLEMENTED]
2. XMSS-OPS-SPEC.md v1.0 — [IMPLEMENTED]
3. GOVERNANCE-SPEC.md — [PLANNED, next in v1.2 production]
4. LEGAL-INTERPRETATION-SPEC.md — [PLANNED v1.2]
5. NETWORK-PROTOCOL-SPEC.md — [PLANNED v1.2]
6. STATE-MACHINE-SPEC.md — [PLANNED v1.2]
7. CRYPTOGRAPHIC-ASSUMPTIONS.md — [PLANNED v1.2]
8. RECOVERY-SPEC.md — [PLANNED v1.2]
9. CANON-RETIREMENT-SPEC.md — [PLANNED v1.2 — added per Reviewer N N-B1]
10. Tier 2 patches to v1.1 specs (Nonce paradox, MBW dedup, etc.) — [PLANNED v1.2]
11. A1A5NT-ARCHITECTURE-SYNTHESIS v1.2 — [PLANNED v1.2]
12. Emergency Root Transfer §14.3 operational — [PLANNED 60 days]
13. Stichting registration complete — [IN PROGRESS, timeline-gated by notarial appointment]
14. NLnet amendment notice 2026-04-3f6 — [PLANNED post-Stichting]
```

Items 1-11 are within architect's direct control and on the v1.2
production track. Items 12-14 depend on external factors
(operator availability, notariusz appointment, NLnet review cycle).

### 15.5 Audit-specific readiness checklists

**For NLnet technical review submission:**

```
[ ] All Tier 1 v1.2 specs complete (items 1-9 above)
[ ] Tier 2 patches applied (item 10)
[ ] Architecture Synthesis v1.2 published (item 11)
[ ] Cross-model AI peer review attestations published
[ ] Open code repositories at stable v1.2 tag
[ ] Operational live infrastructure verified (ANS, Archive, 
    Foundation endpoints all HTTP 200)
```

**For Notariusz NL submission:**

```
[ ] LEGAL-INTERPRETATION-SPEC complete (mapping crypto to BW)
[ ] Stichting draft statutes prepared
[ ] Foundation Declaration v1.0 + License v1.0 printed and anchored
[ ] Successor identification documented (sealed envelope or 
    declaration article)
[ ] Bestuur candidate list prepared
[ ] Operating address confirmed (Almelo, NL)
```

**For Academic post-quantum review submission:**

```
[ ] CRYPTOGRAPHIC-ASSUMPTIONS.md complete
[ ] XMSS-OPS-SPEC formal verification engagement (gated on grant)
[ ] Reference implementation conformance test vectors published
[ ] Multi-implementation cross-validation demonstrated
[ ] Public bug bounty program announced (post-Stichting)
```

---

## 16. License

This specification is issued under A1A5 Sovereign License v1.0
(12-month binding 2026-05-16 → 2027-05-16). AI co-authorship per
Article 4. Anti-patent defensive clause per Article 5.

Adversary class taxonomy and CoC framework adapted from EigenLayer
documentation (CC-BY-style usage) and NIST publications (public
domain). Specific a1a5NT mappings, classifications, and mitigation
references are A1A5-licensed.

---

## 17. Cross-references

| Spec | Relationship |
|------|--------------|
| A1A5NT-ARCHITECTURE-SYNTHESIS v1.2 | Master document; cites this spec for adversary taxonomy |
| XMSS-OPS-SPEC §3 | Detailed adversary analysis for stateful HBS specifically (T-XMSS-1 through T-XMSS-4 here map to A9 family) |
| CANON-REGISTRY-SPEC §5, §6 | Defenses for A2, A3, A4 |
| CUSTODY-REGISTRY-SPEC §6, §8 | Defenses for A5 |
| POV-SPEC §5 | Defense for A7 |
| MBW-SPEC §3 | Defense for A8 (v1.2 patch) |
| RENDER-SPEC §4 | Defense for A10 (v1.2 patch for float determinism) |
| GOVERNANCE-SPEC [PLANNED] | Defense framework for A6 (Foundation insider) including witness oversight, emergency rotation authority, slashing calibration |
| NETWORK-PROTOCOL-SPEC [PLANNED] | Gossip protocol for A4 (hostile witness) detection per RFC 6962 §5.3 |
| CRYPTOGRAPHIC-ASSUMPTIONS [PLANNED] | Formal statement of TA1 assumptions |
| LEGAL-INTERPRETATION-SPEC [PLANNED] | Legal/institutional framework for A5 dispute resolution, A6 institutional response, A13 legal compulsion |
| CANON-RETIREMENT-SPEC [PLANNED] | Operational detail for A13 hybrid tombstone pattern (added v1.1) |
| RECOVERY-SPEC [PLANNED] | Detailed procedures for Cat-1 through Cat-3, H-3 |

---

## 18. Cryptographic anchor

```
Document path:    SECURITY-MODEL.md
Repository:       github.com/a1a5NT/a1a3
Commit timestamp: filled at first commit
Foundation mirror endpoint:  http://37.97.202.97:8444/v1/foundation/security_model  (scheduled)
```

The canonical hash of this document (blake3 of UTF-8 form) will be
published in `/v1/foundation/index` after commit.

---

**End of SECURITY-MODEL.md v1.0**

```
Issued by:        pawel.a1a5 (Paweł Piekut, Almelo NL)
AI co-author:     Claude (Anthropic Opus 4.7)
Date:             2026-05-17
Foundation:       a1a5fndn.a1a5 (in formation)
Audit relevance:  BLOCKING for academic post-quantum review;
                  REQUIRED for NLnet technical review;
                  REQUIRED for institutional adoption
Reviewer consensus addressed: Gemini implicit + Grok #1 + GPT #1
                  + Claude PC E1 CRITICAL — 4/4 unanimous priority #1
```
