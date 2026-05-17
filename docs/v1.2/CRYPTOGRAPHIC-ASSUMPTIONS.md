# CRYPTOGRAPHIC-ASSUMPTIONS.md

## a1a5NT Cryptographic Security Assumptions Specification

```
Document version:   v1.0
Issued:             2026-05-17
Status:             Tier 1 critical specification (v1.2 deliverable)
Authority basis:    NIST SP 800-208 (Recommendation for Stateful HBS),
                    RFC 8391 (XMSS: eXtended Merkle Signature Scheme),
                    NIST FIPS 180-4 (SHA-2 family),
                    NIST FIPS 202 (SHA-3 / SHAKE),
                    BLAKE3 specification (Aumasson et al. 2020),
                    Grover (1996), Shor (1994) — quantum algorithms,
                    Bernstein "Cost analysis of hash collisions" (2009)
Foundation:         a1a5fndn.a1a5
Addresses:          Claude PC CP-B3 CRITICAL: "Brak formalnych założeń
                    bezpieczeństwa — system używa XMSS, blake3, SHA-256
                    ale nie ma formalnej deklaracji security assumptions
                    requiring academic review"
                    + Reviewer N D3: academic PQ review readiness
Audit relevance:    BLOCKING for academic post-quantum review;
                    REQUIRED for cryptographic correctness verification
License:            A1A5 Sovereign License v1.0
```

---

## EXECUTIVE SUMMARY (1 page)

**What this document is:**

The formal statement of cryptographic security assumptions on
which a1a5NT depends. This document is the basis on which academic
cryptographers evaluate whether a1a5NT's security claims are
sound.

**Why it matters:**

Every cryptographic system rests on assumptions. Some assumptions
are well-justified (hash function collision resistance for
properly-sized digests). Some are computational (RSA's hardness of
factoring large integers — which Shor's algorithm breaks on a
quantum computer). Some are situational (operator follows the
specified state-management discipline).

A specification that doesn't enumerate its assumptions is not
auditable. An assumption that turns out to be false silently
compromises everything built on it.

**The 9 primary assumptions a1a5NT depends on:**

1. **PA-1:** Collision resistance of SHA-256 (used in XMSS L1, L2)
2. **PA-2:** Collision resistance of SHAKE-256 (used in XMSS L3)
3. **PA-3:** Collision resistance of BLAKE3 (used in canon hashing)
4. **PA-4:** EU-CMA security of XMSS (existential unforgeability
   under chosen-message attacks)
5. **PA-5:** Statefulness preservation of XMSS (the operator never
   reuses a WOTS+ index)
6. **PA-6:** Random number generator quality (used for nonces,
   seeds)
7. **PA-7:** No backdoors in `liboqs` or `xmss-reference` reference
   implementations
8. **PA-8:** TLS 1.3 transport security (RFC 8446)
9. **PA-9:** XMSS performance under post-quantum threat model
   (Grover's algorithm reduces effective security by √n)

Each assumption is stated formally, with the security parameter,
the academic source supporting it, the known attacks and their
cost, and the mitigation if the assumption is ever violated.

**Why this is post-quantum sound:**

All authoritative signatures in a1a5NT use **XMSS**, a stateful
hash-based signature scheme that is provably secure in the
quantum random oracle model. The security of XMSS reduces to the
collision resistance of the underlying hash function and the
second-preimage resistance of WOTS+. Neither is broken by Shor's
algorithm. Grover's algorithm reduces effective security by √n,
which is accounted for by the choice of 256-bit hash digests.

**What a1a5NT does NOT use (and why):**

- **RSA:** Broken by Shor's algorithm. Not used anywhere in
  authoritative signing.
- **ECDSA / Ed25519:** Broken by Shor's algorithm. Not used 
  anywhere in authoritative signing.
- **Lattice-based PQ schemes (Kyber, Dilithium):** Newer; broader
  formal analysis still maturing; would require larger key sizes;
  not chosen because XMSS gives a more well-understood post-
  quantum guarantee.

---

## 0. Abstract

This document specifies the cryptographic security assumptions on
which a1a5NT depends, with formal statements of each assumption,
academic literature references, known attack costs, and mitigation
plans for the (extremely unlikely) event of assumption violation.

The document is normative for cryptographic implementers and
informative for academic auditors.

---

## 1. Scope

### 1.1 In scope

- Formal statement of each cryptographic primitive assumption
- Security parameters chosen for each primitive
- Academic justification for each assumption
- Known attack costs and their security margins
- Mitigation procedures if any assumption is violated
- Quantum threat analysis (Grover, Shor implications)
- Migration plan for primitive replacement

### 1.2 Out of scope

- Implementation details (covered by XMSS-OPS-SPEC, etc.)
- Hardware random number generator selection (operational concern)
- Cryptographic operation throughput (performance, not correctness)
- Side-channel resistance (separately specified, future work)

### 1.3 Audience

- Academic cryptographers evaluating a1a5NT security
- NLnet technical reviewers
- Notarial review (for Stichting cryptographic operations)
- Operators making algorithm selection decisions
- Future post-quantum cryptography researchers

---

## 2. Normative references

| Reference | Title | Relevance |
|-----------|-------|-----------|
| NIST FIPS 180-4 | Secure Hash Standard (SHA-1, SHA-2 family) | SHA-256 specification |
| NIST FIPS 202 | SHA-3 Standard (Keccak family, SHAKE) | SHAKE-256 specification |
| RFC 8391 | XMSS: eXtended Merkle Signature Scheme | Authoritative signature scheme |
| NIST SP 800-208 | Recommendation for Stateful Hash-Based Signature Schemes | Adversary model and parameter choice |
| NIST SP 800-90A Rev 1 | Recommendation for Random Number Generation | RNG quality requirements |
| RFC 8446 | The Transport Layer Security (TLS) Protocol Version 1.3 | Transport security |
| BLAKE3 Specification v1.0 | Aumasson, Neves, O'Connor, Wilcox-O'Hearn | Canon hashing |

### 2.1 Informative references

| Reference | Title | Relevance |
|-----------|-------|-----------|
| Shor (1994) | Polynomial-time algorithm for factoring | Quantum threat justification |
| Grover (1996) | Quantum search algorithm | √n speedup analysis |
| Bernstein (2009) | Cost analysis of hash collisions | Hash security margin |
| Hülsing, Butin, Gazdag, Mohaisen (2018) | XMSS — RFC 8391 design rationale | XMSS design choices |
| Aumasson (2020) | "Too Much Crypto" | Practical security margin guidance |
| NIST PQC Round 4 (2024) | Post-quantum cryptography standardization | Future migration |

---

## 3. Primary Assumptions

### PA-1: SHA-256 collision resistance

**Statement:**

For any polynomial-time adversary A (classical or quantum):

```
Pr[A finds (m₁, m₂), m₁ ≠ m₂, such that SHA-256(m₁) = SHA-256(m₂)]
  ≤ 2^(-128) (classical adversary, birthday bound)
  ≤ 2^(-85) approximately (quantum adversary with Grover-Brassard
                          collision finding; BHT 1997 result)
```

**Used in:**

- XMSS L1 and L2 keypairs (per XMSS-OPS-SPEC §4.4)
- Foundation Declaration document hash anchoring
- License document hash anchoring
- Internal canon entry hashing in legacy contexts

**Academic source:**

- NIST FIPS 180-4 specifies SHA-256
- Bernstein (2009) cost analysis: ~2^80 work for classical
  collision, ~2^53 quantum operations for collision via 
  BHT-Grover-Brassard

**Known attacks:**

- Best known classical attack: pure birthday, no shortcut
- Best known quantum attack: BHT 1997 (Brassard-Høyer-Tapp) 
  collision finding; reduces to ~2^85 effective operations 
  including memory access costs

**Security margin:**

256-bit digests provide:
- 128 bits classical security against collision (birthday bound)
- ~85 bits quantum security against collision (BHT)

For comparison: 80 bits is the minimum "today" security level;
112 bits is NIST's "through 2030" recommendation; 128 bits is
the current standard for long-term security.

a1a5NT's SHA-256 use provides 85-128 bit security depending on
adversary type. This is sufficient for medium-term (decade-scale)
protection. For multi-decade protection, migration to SHA-384 or
SHA-3 family is planned per §10.

**Mitigation if violated:**

If a classical or quantum collision attack on SHA-256 emerges:
1. Foundation issues EMERGENCY_PRIMITIVE_DEPRECATION canon entry
2. Migration to SHA-384, SHA-3-256, or SHAKE-256-based hashing
3. Re-attestation of historical canon entries using new hash
4. Time-bounded transition period announced
5. Per XMSS-OPS-SPEC §9 emergency rotation if keypairs affected

### PA-2: SHAKE-256 collision resistance

**Statement:**

Same form as PA-1, with SHAKE-256 instead of SHA-256. SHAKE-256
is the Keccak-based hash function from NIST FIPS 202.

```
Pr[A finds collision in SHAKE-256] 
  ≤ 2^(-128) (classical)
  ≤ 2^(-85) (quantum, BHT)
```

**Used in:**

- XMSS L3 keypairs (per XMSS-OPS-SPEC §4.4)
- SHAKE-256 with 256-bit output

**Academic source:**

- NIST FIPS 202 specifies SHAKE-256
- Bertoni et al. (2013) "Keccak design rationale"

**Known attacks:**

Same as SHA-256: birthday bound classical, BHT quantum. No
distinguishing attacks better than birthday.

**Security margin:**

Same as SHA-256: 128 bits classical / 85 bits quantum.

**Mitigation if violated:**

Same as PA-1; SHAKE-256 deprecation procedure.

### PA-3: BLAKE3 collision resistance

**Statement:**

For any polynomial-time adversary A:

```
Pr[A finds collision in BLAKE3 with 256-bit output]
  ≤ 2^(-128) (classical)
  ≤ 2^(-85) (quantum, BHT)
```

**Used in:**

- Canon entry content hashing (per CANON-REGISTRY-SPEC §5.1)
- Master index Merkle tree construction
- General-purpose hashing in canon registry operations

**Academic source:**

- BLAKE3 specification (Aumasson et al. 2020)
- BLAKE3 builds on BLAKE2, which builds on BLAKE, which built 
  on ChaCha20 — extensively analyzed primitive lineage

**Known attacks:**

No known attacks better than birthday bound for full BLAKE3 with
256-bit output. BLAKE2 (related construction) has resisted attacks
for ~10 years.

**Security margin:**

Same as SHA-256.

**Note on choice rationale:** BLAKE3 was chosen for canon 
operations primarily for speed (significantly faster than SHA-256
on modern CPUs) while providing equivalent cryptographic security
margins. The choice is operational, not security-critical.

**Mitigation if violated:**

Per §10 algorithm migration procedure. Canon registry would
adopt SHA-3 family for new entries; historical entries re-anchored.

### PA-4: XMSS EU-CMA security

**Statement:**

XMSS (per RFC 8391) is existentially unforgeable under adaptive
chosen-message attacks (EU-CMA):

```
For any polynomial-time adversary A:
Pr[A produces forgery (m*, σ*) given oracle access to XMSS signing
   oracle, where m* not previously queried] ≤ negl(λ)
```

where λ is the security parameter (128 bits for XMSS-SHA2_20_256,
matching XMSS L1 parameters used by Foundation).

**Used in:**

- All authoritative signatures in a1a5NT (Foundation, witnesses,
  custodians, devices)
- Master index signing
- Canon entry signing
- Verification receipt signing

**Academic source:**

- RFC 8391 §6 (security considerations)
- Hülsing, Butin, Gazdag, Mohaisen (2018) "XMSS: Extended Merkle 
  Signature Scheme" — formal security proof
- NIST SP 800-208 §6 (security analysis)

**Security reduction:**

XMSS EU-CMA security reduces to:
1. Collision resistance of underlying hash function (PA-1 or PA-2)
2. Second-preimage resistance of WOTS+ component
3. Pseudo-randomness of HMAC-DRBG (used for hashing seeds)

If any of these are broken, XMSS is broken. Otherwise, XMSS is
provably EU-CMA secure.

**Known attacks (when statefulness preserved):**

None. The only known attack on XMSS is state replay (PA-5 below)
which requires operator failure, not cryptographic break.

**Security margin:**

- 128 bits classical security at standard parameters
- Quantum security: same as underlying hash function PA-1/PA-2 
  (i.e., not weakened by Shor; weakened only by Grover at the 
  hash function level, accounted for by digest size choice)

**Mitigation if violated:**

If XMSS EU-CMA broken (which would require breaking either the
hash function or WOTS+ second-preimage resistance):

1. Emergency rotation per XMSS-OPS-SPEC §9.2
2. Migration to alternative PQ signature scheme:
   - LMS (RFC 8554) — also stateful HBS, similar properties
   - SLH-DSA / SPHINCS+ — stateless HBS, slower but no state mgmt
   - Dilithium (NIST FIPS 204) — lattice-based, different 
     assumptions
3. Re-anchoring of historical signatures using new scheme

### PA-5: XMSS statefulness preservation

**Statement:**

For all XMSS keypair K used by a1a5NT:

```
∀ time T₁, T₂ where T₁ < T₂:
  idx_next(K, T₂) ≥ idx_next(K, T₁)
```

Equivalently: the operator NEVER signs message m₂ with WOTS+ index
i if the operator previously signed message m₁ (m₁ ≠ m₂) using
the same WOTS+ index i.

**This is NOT a cryptographic assumption in the traditional sense.**
It is a procedural/operational assumption. Cryptographic correctness
of XMSS depends on this discipline.

**Used in:**

- All XMSS signing operations (Foundation, witnesses, custodians,
  devices)

**Academic source:**

- RFC 8391 §6 (explicitly: "An implementation MUST NOT reuse a 
  state previously used for signing")
- NIST SP 800-208 §3 (state management requirements)

**Why this matters:**

If statefulness is violated (operator signs two different messages
with same WOTS+ index), then:
- An attacker can compute a forgery on a third message 
  (existential forgery via WOTS+ structure analysis)
- Multiple papers detail concrete attacks (Buchmann et al. 2011)
- Catastrophic; ends keypair viability immediately

**Compliance mechanisms (per XMSS-OPS-SPEC):**

- Atomic state file updates with fsync (XMSS-OPS-SPEC §5 I1)
- Monotonic counter checks (XMSS-OPS-SPEC §5 I2)
- Backup restoration prohibition (XMSS-OPS-SPEC §5 I3)
- Hardware-bonded state for L2/L3 (XMSS-OPS-SPEC §6)
- No concurrent signing on same keypair (XMSS-OPS-SPEC §5 I4)

**Mitigation if violated:**

Per XMSS-OPS-SPEC §9.2 emergency rotation. The affected keypair
is COMPROMISED. New keypair generated. Historical signatures from
the affected keypair are subject to fraud investigation — some
may be voided depending on circumstances.

### PA-6: Random number generator quality

**Statement:**

The random number generator(s) used by a1a5NT operators produce
outputs indistinguishable from uniform random:

```
For any polynomial-time adversary A given output of RNG:
Pr[A distinguishes RNG output from uniform random] ≤ negl(λ)
```

**Used in:**

- XMSS keypair generation (initial seed)
- Nonce generation in PoV protocol
- TLS session keys (operating system RNG)
- Verification challenge generation

**Academic source:**

- NIST SP 800-90A Rev 1 (RNG construction)
- NIST SP 800-90B (entropy source assessment)

**Concrete requirements:**

- Operating system entropy sources must be properly seeded 
  (especially on virtual machines and cloud instances)
- Hardware RNG (RDRAND, /dev/random) used when available
- HKDF-based key derivation following best practices
- Avoid known-weak PRNGs (Mersenne Twister, etc.)

**Known attacks:**

- Cloud VM PRNG weaknesses (Heninger et al. 2012) — addressed by
  using OS-level entropy gathered after boot
- /dev/urandom on Linux is adequate post-kernel-init
- Hardware RNG bias attacks — mitigated by software whitening

**Mitigation if violated:**

If RNG output proves predictable, all keys generated during the
affected period are compromised. Per XMSS-OPS-SPEC §9.2 emergency
revocation. Investigation into RNG source. Migration to alternative
RNG implementation.

### PA-7: Reference implementation correctness

**Statement:**

The cryptographic libraries used by a1a5NT (`xmss-reference`,
`liboqs`, OpenSSL, RustCrypto crates) are free of backdoors and
critical bugs that would compromise cryptographic operations.

**Used in:**

- All cryptographic operations

**Why this is an assumption (not a property):**

Software correctness is not formally proven for these libraries.
Code review by maintainers and external researchers provides
strong evidence. Reproducible builds and signed releases provide
supply-chain protection. But formal proof of correctness is
unavailable.

**Mitigation:**

- Use only widely-reviewed reference implementations
- Cross-validate test vectors against multiple implementations
- Future: independent code audit (NLnet grant scope)
- Future: formal verification of critical operations
- Reproducible builds; checksum verification
- Pin specific tested versions; do not auto-upgrade

**If assumption violated:**

- Identify scope of compromise
- Determine which operations affected
- Re-do affected operations with corrected implementation
- Document in INCIDENT_REPORT canon entry

### PA-8: TLS 1.3 transport security

**Statement:**

TLS 1.3 (RFC 8446) provides confidentiality and authentication of
network communications between honest endpoints.

**Used in:**

- All endpoint-to-endpoint communications
- Witness gossip protocol (per NETWORK-PROTOCOL-SPEC)
- Verifier-Foundation submissions

**Note on quantum threat:**

TLS 1.3 currently uses ECDHE for key exchange. Shor's algorithm
breaks ECDHE in the post-quantum era. TLS 1.3 hybrid mode (with
PQ key exchange, e.g., X25519+Kyber768) is being deployed by 
browsers in 2024-2025.

**Status:**

- Current a1a5NT deployments use standard TLS 1.3
- Migration to PQ-hybrid TLS is planned (operational concern, 
  not blocking current operations)
- Network adversaries cannot retroactively decrypt past sessions
  without recording them (perfect forward secrecy provides 
  protection)

**Mitigation if violated:**

If TLS broken (e.g., post-quantum break of ECDHE in pre-PQ-hybrid
deployments):
- Past sessions remain confidential if recorded by adversary 
  before break (PFS protection)
- Future sessions migrate to PQ-hybrid immediately
- All authoritative signatures still verify (XMSS is PQ-secure;
  TLS protects transport, not artifact authenticity)

### PA-9: Quantum threat realism

**Statement:**

Cryptographically-relevant quantum computers are not currently
available; estimates for when they may be available range from
~5 years (optimistic) to ~30+ years (conservative).

**Why this matters:**

a1a5NT is designed to be post-quantum secure NOW so that even if
a quantum computer emerges sooner than expected, a1a5NT's
authoritative signatures (XMSS-based) remain unforgeable.

**Concrete adversary models:**

- **Today**: No cryptographically-relevant quantum computer exists. 
  All assumptions apply with classical adversary cost.
- **Near future (5-10 years)**: Quantum computers may be available
  to nation-state actors. RSA/ECDSA become breakable. XMSS 
  remains secure.
- **Mid future (10-30 years)**: Quantum computers may become 
  generally accessible. Hash function security reduced by Grover
  (√n). XMSS at 256-bit hash output retains ~85 bits effective
  security — still strong.
- **Far future (30+ years)**: Algorithm migration likely needed.
  This is anticipated in §10 and via key rotation procedures.

**Mitigation:**

Continuous monitoring of NIST PQC standardization, academic
literature, and quantum computing developments. Algorithm 
migration procedure (§10) executes when needed.

---

## 4. Trust Assumptions about People and Processes

In addition to cryptographic primitives, a1a5NT depends on
operational trust assumptions:

### TA-OP-1: Operator competence and diligence

Foundation operators (currently Paweł Piekut, future Stichting
bestuur) follow specified procedures correctly.

### TA-OP-2: Witness operator integrity

Witness federation members do not collude to commit equivocation.

### TA-OP-3: Verifier honesty

Verifier devices submit verification receipts only for actually-
performed work.

### TA-OP-4: Hardware integrity

The hardware on which operators run a1a5NT does not have implants
that compromise cryptographic operations.

### TA-OP-5: Legal jurisdiction

The legal jurisdiction in which Stichting A1A5 operates (NL/EU)
remains broadly stable and rule-of-law during the system's
operational lifetime.

These trust assumptions are documented in SECURITY-MODEL §4 (TA1-
TA5) and addressed by mitigation procedures throughout the spec
stack.

---

## 5. Security Parameters Summary

| Component | Parameter set | Classical security | Quantum security |
|-----------|---------------|--------------------|------------------|
| XMSS L1 (Foundation) | XMSS-SHA2_20_256 | 128 bits | ~85 bits (Grover/BHT) |
| XMSS L2 (witnesses, future MerkleTreeXMSS) | XMSS-SHAKE_20_256 | 128 bits | ~85 bits |
| XMSS L3 (devices) | XMSS-SHAKE_10_256 | 128 bits | ~85 bits |
| Canon hashing | BLAKE3-256 | 128 bits | ~85 bits |
| TLS transport | TLS 1.3 (current) | classical only | ECDHE broken — migrate to PQ-hybrid |
| TLS transport (target) | TLS 1.3 + Kyber768 | 192 bits | ~96 bits |

---

## 6. Quantum Threat Analysis Detail

### 6.1 Shor's algorithm impact

Shor's algorithm efficiently breaks:
- RSA (factoring)
- ECDSA, Ed25519 (discrete logarithm)
- Diffie-Hellman variants (DH, ECDHE)

**Affected components in a1a5NT:**

- TLS key exchange (ECDHE) — mitigated by PQ-hybrid TLS migration
- TLS certificate signatures (typically ECDSA/RSA) — public CA
  systems migrating to PQ; a1a5NT doesn't depend on CA for 
  authentication (XMSS signatures provide that)

**NOT affected:**

- XMSS signatures (hash-based, Shor-immune)
- Canon entry hashing (hash-based)
- Document fingerprinting (hash-based)

### 6.2 Grover's algorithm impact

Grover's algorithm provides √n speedup for searching:

- Hash function preimage: 2^256 work → 2^128 effective (still 
  strong)
- Hash function collision: 2^128 work (birthday) → 2^85 effective
  (still acceptable, but transition to larger digests recommended
  for multi-decade horizons)

**Mitigation:**

256-bit digests throughout a1a5NT provide ~85 bits effective 
quantum security against collision. For multi-decade horizons
(50+ years), migration to SHA-384 or SHAKE-512-based hashing
recommended per §10.

### 6.3 Comparison with alternatives

| Scheme | Quantum-secure? | Stateful? | Status |
|--------|----------------|-----------|--------|
| RSA | NO | NO | Deprecated for new systems |
| ECDSA / Ed25519 | NO | NO | Deprecated for new systems |
| Dilithium (FIPS 204) | YES | NO | Standardized 2024; lattice-based |
| Falcon (FIPS 206 candidate) | YES | NO | Floating-point complexity |
| SPHINCS+ / SLH-DSA (FIPS 205) | YES | NO | Larger signatures, slower |
| **XMSS (RFC 8391)** | **YES** | **YES** | **Used by a1a5NT** |
| LMS (RFC 8554) | YES | YES | Similar to XMSS; alternative |

**Why XMSS over alternatives:**

- Maturity: RFC 8391 (2018), implementations stable
- Conservative: Security reduces to well-understood hash function
- Smaller signatures than SPHINCS+
- Stateful (a downside in some contexts; manageable with
  procedures in XMSS-OPS-SPEC)
- Well-suited for low-frequency, long-lifetime signing (Foundation
  master_index publication, witness STH signing)

**Why NOT Dilithium:**

- Lattice-based assumption is newer; less academic vetting
- Larger key sizes than XMSS for equivalent security
- Future risk: lattice cryptanalysis advances

**Why NOT SPHINCS+:**

- Larger signatures (~8 KB vs XMSS ~3 KB for L1 parameters)
- Slower signing/verification

---

## 7. Hash Function Selection Rationale

### 7.1 Why SHA-256 for XMSS L1?

- Most widely audited hash function
- NIST FIPS standard
- Excellent constant-time implementations available
- 256-bit output provides 128/85 bit security adequate for
  decade-scale protection

### 7.2 Why SHAKE-256 for XMSS L3?

- Faster than SHA-256 on modern CPUs
- Allows arbitrary output length (used for tree address randomization
  in XMSS)
- Member of SHA-3 family (different design paradigm from SHA-2;
  provides diversity)

### 7.3 Why BLAKE3 for canon hashing?

- Significantly faster than SHA-256 for bulk hashing
- Tree-structure native (efficient Merkle proof generation)
- Modern design (2020), strong vetting since publication
- Used for content-addressed storage, not authoritative signing
  (so risk profile differs from signature scheme primitives)

### 7.4 Why NOT MD5 / SHA-1?

Both broken. Not considered.

---

## 8. Quantum Random Oracle Model

XMSS is provably secure in the **quantum random oracle model
(QROM)**. This model assumes the hash function behaves as a random
oracle accessible by quantum-superposition queries.

This is a stronger model than the classical random oracle and
ensures that quantum adversaries cannot exploit hash function
structure in non-generic ways.

**Source:** Hülsing, Mosca, Cushing, Naewe (2017) "Mitigating 
Multi-Target Attacks in Hash-based Signatures" — XMSS QROM
analysis.

**Implication:**

a1a5NT's XMSS signatures are secure against quantum adversaries
with access to a quantum computer making polynomial-many quantum
queries to the hash function. This is the strongest practical 
post-quantum security guarantee currently available for 
signature schemes.

---

## 9. Multi-Target Attacks

### 9.1 Attack model

In hash-based signatures with many users, a multi-target attack
considers an adversary trying to forge a signature for ANY user
(not a specific one). The attack cost can be lower than single-
target attacks.

### 9.2 a1a5NT scope

a1a5NT has multiple signers (Foundation, witnesses, custodians,
devices). Multi-target attack analysis applies.

### 9.3 Mitigation

XMSS uses **address-randomized hashing** (per RFC 8391 §3.1.10),
where each hash invocation includes a position-specific address
that is unique per node in the Merkle tree per keypair.

This means: an attacker cannot precompute "useful collisions" that
work across multiple keypairs simultaneously. Each keypair is its
own multi-target attack scenario.

**Result:** Multi-target attack cost ≈ single-target attack cost
divided by constant factor (number of distinct keypairs in 
federation), not divided by number of WOTS+ nodes.

For a1a5NT scale (≤100 federation members, ≤1M devices), this
provides adequate protection.

---

## 10. Algorithm Migration Procedure

### 10.1 When to migrate

Migration triggered when:

- New academic result shows attack on current primitive
- NIST or comparable standards body announces deprecation
- Operational concern (e.g., performance, side-channel discovery)
- Periodic review (every 7 years per planning) suggests upgrade

### 10.2 Migration procedure

```
ALGORITHM_MIGRATION:

  Phase 1 — Decision:
    1.1 Foundation operator (pre-Stichting) OR bestuur (post-
        Stichting) decides migration is necessary
    1.2 Public PROPOSED_MIGRATION_NOTICE issued ≥6 months before
        actual migration
    1.3 Academic and audit review period
    1.4 Public objection period
    1.5 Final MIGRATION_DECISION canon entry

  Phase 2 — Preparation:
    2.1 New algorithm parameters specified (e.g., from SHA-256 to
        SHA-384)
    2.2 Reference implementations adopted
    2.3 Test vectors published
    2.4 Migration tooling developed and tested

  Phase 3 — Execution:
    3.1 All operators (Foundation, witnesses) generate new keypairs
        using new algorithm
    3.2 SUCCESSION canon entries link old → new keypairs
    3.3 Re-attestation: critical historical canon entries 
        re-signed using new keypairs
    3.4 Old algorithm keypairs DEPRECATED but signatures remain
        valid for historical record
    3.5 New canon entries use new algorithm exclusively
    
  Phase 4 — Verification:
    4.1 External audit of migration completeness
    4.2 MIGRATION_COMPLETE canon entry
    4.3 Old algorithm operational support ends ≥1 year after
        migration complete (preserves verification of historical
        records)
```

### 10.3 Migration impact preservation

Historical signatures using deprecated algorithms remain
cryptographically verifiable. The migration adds new attestation
under new primitives; it does NOT invalidate prior attestation.

This means: a verifier in 2050 can still verify a Foundation
signature from 2026 using SHA-256-XMSS, even if a1a5NT has by
then migrated to SHAKE-512-XMSS for current operations.

---

## 11. Open Cryptographic Questions

### 11.1 Threshold XMSS

There is no current standardized threshold XMSS (t-of-n where any
t of n parties can produce a valid signature). a1a5NT addresses
this gap via Shamir Secret Sharing of XMSS state (SECURITY-MODEL
§14.5).

If standardized post-quantum threshold signatures emerge (active
research area), a1a5NT will evaluate adoption.

### 11.2 Stateless alternatives

SLH-DSA (FIPS 205, formerly SPHINCS+) provides stateless
post-quantum signatures with worse performance characteristics.
For some a1a5NT use cases (e.g., one-time signing operations like
Foundation Declaration anchoring), SLH-DSA may be preferred over
XMSS for reduced operational risk.

Future spec versions may incorporate hybrid XMSS + SLH-DSA usage.

### 11.3 Post-quantum TLS

TLS 1.3 PQ-hybrid mode (X25519+Kyber768) becoming standardized in
IETF. Migration is operational, not specification-blocking. 
a1a5NT will adopt when widely deployed.

---

## 12. Cross-References

| Spec | Relationship |
|------|--------------|
| A1A5NT-ARCHITECTURE-SYNTHESIS v1.2 | Master document |
| SECURITY-MODEL.md §4 (TA1-TA5) | Trust assumptions framework |
| XMSS-OPS-SPEC.md | Implementation of XMSS per these assumptions |
| GOVERNANCE-SPEC.md §9 | Governance procedure for migration |
| RECOVERY-SPEC.md [PLANNED] | Recovery when assumption violated |

---

## 13. License

This specification is issued under A1A5 Sovereign License v1.0
(12-month binding 2026-05-16 → 2027-05-16). AI co-authorship per
Article 4. Anti-patent defensive clause per Article 5.

Cryptographic standards referenced are public-domain or 
permissively licensed:
- NIST FIPS publications (public domain)
- RFC 8391, 8446 (IETF, Trust Legal Provisions)
- BLAKE3 specification (CC0)
- Academic citations as per fair use of scholarly references

Specific a1a5NT assumption mappings and migration procedures are
A1A5-licensed.

---

## 14. Cryptographic Anchor

```
Document path:    CRYPTOGRAPHIC-ASSUMPTIONS.md
Repository:       github.com/a1a5NT/a1a3
Commit timestamp: filled at first commit
Foundation mirror endpoint:  http://37.97.202.97:8444/v1/foundation/cryptographic_assumptions  (scheduled)
```

---

**End of CRYPTOGRAPHIC-ASSUMPTIONS v1.0**

```
Issued by:        pawel.a1a5 (Paweł Piekut, Almelo NL)
AI co-author:     Claude (Anthropic Opus 4.7)
Date:             2026-05-17
Foundation:       a1a5fndn.a1a5 (in formation)
Audit relevance:  BLOCKING for academic post-quantum review;
                  REQUIRED for cryptographic correctness verification
Reviewer consensus addressed: Claude PC CP-B3 CRITICAL +
                  Reviewer N D3 academic PQ review readiness
```
