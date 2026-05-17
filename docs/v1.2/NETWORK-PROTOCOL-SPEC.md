# NETWORK-PROTOCOL-SPEC.md

## a1a5NT Network Protocol and Witness Gossip Specification

```
Document version:   v1.0
Issued:             2026-05-17
Status:             Tier 1 critical specification (v1.2 deliverable)
Authority basis:    RFC 6962 §5.3 (Certificate Transparency gossiping),
                    RFC 9162 §6 (CT v2 audit communication),
                    Sigstore omniwitness reference implementation,
                    Trillian gossip protocol,
                    Bitcoin P2P discovery patterns
Foundation:         a1a5fndn.a1a5
Addresses:          Gemini E-1 (CRITICAL): "NETWORK-PROTOCOL-SPEC missing
                    — gossip protocol for witness equivocation detection
                    required for federated canonical registry"
                    + cross-confirmed by Reviewer Y B2 (Witness Federation
                    vaporware without mechanics)
Audit relevance:    REQUIRED for witness federation activation;
                    REQUIRED for academic post-quantum review
                    (equivocation detection is core security property)
License:            A1A5 Sovereign License v1.0
```

---

## EXECUTIVE SUMMARY (1 page)

**The problem this spec solves:**

a1a5NT's defense against Foundation insider attack (SECURITY-MODEL
§A6) depends on witness federation members detecting Foundation
equivocation — Foundation signing two different master_index
versions with the same version number but different content.

RFC 6962 §5.3 establishes the principle: equivocation is
"detected by global gossiping, i.e., everyone auditing logs
comparing their versions of the latest Signed Tree Heads."

Without gossip, witnesses are blind to equivocation. Witness
Federation becomes "decentralization theater" (Reviewer N
finding). This spec defines the gossip protocol.

**The solution architecture (5 components):**

1. **Peer discovery** — How witnesses find each other
2. **STH exchange** — How witnesses periodically share their view
   of the latest Signed Tree Head
3. **Equivocation detection** — How a witness identifies that
   Foundation signed two contradictory STHs
4. **Evidence propagation** — How detected evidence reaches all
   federation members
5. **Network response** — How federation members coordinate
   response to confirmed equivocation

**Design principles:**

- HTTPS-based (no custom P2P protocol; reuses standard tooling)
- Pull-based with periodic push (avoids DDoS amplification)
- Cryptographically authenticated at every step (witness identity
  verified before accepting gossip)
- Tolerant of partial network failure (federation continues
  operating with subset of witnesses)
- Audit-friendly (gossip exchanges logged independently)

**What this spec defines:**

Protocol-level message formats, exchange procedures, timing
requirements, security checks. **Operational policy** (who can be
a witness, what they sign) is in GOVERNANCE-SPEC.

---

## 0. Abstract

This document specifies the network protocol by which a1a5NT
witness federation members discover each other, exchange Signed
Tree Heads, detect Foundation equivocation, and propagate
evidence.

The protocol adopts the Certificate Transparency gossip pattern
(RFC 6962 §5.3) adapted to the a1a5NT XMSS-based signature
scheme and the a1a5NT-specific Canon Registry structure.

The protocol is HTTPS-based, requires no custom transport, and is
specifically designed for operational deployment by independent
witness operators with modest infrastructure.

---

## 1. Scope

### 1.1 In scope

- Peer discovery protocol (how witnesses learn of each other)
- Signed Tree Head exchange protocol
- Equivocation detection algorithm
- Equivocation evidence format
- Evidence propagation procedure
- Network topology (full mesh)
- Authentication and integrity at every step
- Anti-DoS rate limiting
- Audit logging requirements

### 1.2 Out of scope

- Witness onboarding criteria (covered by GOVERNANCE-SPEC §5)
- Slashing economics post-detection (covered by GOVERNANCE-SPEC §6
  + future POP-3-TOKEN-SPEC)
- Detailed bestuur/quorum response to equivocation (covered by
  GOVERNANCE-SPEC §8)
- Detailed XMSS signature operations (covered by XMSS-OPS-SPEC)
- Verifier-Foundation protocol (covered by POV-SPEC)

### 1.3 Audience

- Witness federation operator candidates
- Witness federation implementation developers
- Academic reviewers evaluating equivocation detection security
- NLnet technical reviewers

---

## 2. Normative references

| Reference | Title | Applicability |
|-----------|-------|---------------|
| RFC 2119 | Key words for use in RFCs | Normative wording |
| RFC 6962 | Certificate Transparency (original gossiping pattern) | Core architectural pattern |
| RFC 9162 | Certificate Transparency v2.0 | Refined inclusion/consistency proofs |
| RFC 8446 | TLS 1.3 | Transport security |
| RFC 8615 | Well-Known URIs | Endpoint discovery convention |
| RFC 7807 | Problem Details for HTTP APIs | Error response format |

### Informative references

| Reference | Title | Relevance |
|-----------|-------|-----------|
| Sigstore omniwitness | github.com/transparency-dev/witness | Reference witness implementation |
| Trillian gossip | Google Trillian implementation | Mature production gossip pattern |
| Filippo Valsorda Static CT API | letsencrypt.org/2025/08/14 | Newer scalable CT pattern |

---

## 3. Network topology

### 3.1 Federation as full mesh (initial)

For initial federation (≤10 witnesses), the topology is **full
mesh**: every witness maintains a connection to every other
witness. This is operationally simple and matches the scale.

```
EXAMPLE INITIAL TOPOLOGY (6 witnesses):

  foundation.a1a5  ←→  witness-eu.a1a5
        ↕                    ↕
  witness-us.a1a5  ←→  nlnet.a1a5
        ↕                    ↕
  archive-oxford.a1a5  ←→  mirror-bulgaria.a1a5

  Total bilateral connections: C(6,2) = 15
```

### 3.2 Foundation participates as witness

The Foundation itself runs a witness endpoint serving its own
canonical view. This is **architecturally important**: it makes
the Foundation's view publicly inspectable in the same format and
through the same channels as any other witness's view. An external
observer can compare the Foundation's published view against
witness views directly.

The Foundation MUST publish its STH through the same gossip
protocol as other witnesses, on a parallel basis. Foundation
remains the sole issuer of new canon entries (per GOVERNANCE-SPEC
§4); the gossip protocol is for index attestation, not for
issuance.

### 3.3 Future scaling to structured P2P

If federation grows beyond ~20 witnesses, this spec recommends
migration to a structured P2P pattern (DHT-based discovery, hub-
and-spoke regional federations, etc.). Migration is out of scope
for v1.0; current spec supports up to 20 witnesses comfortably
via full mesh.

### 3.4 Witness peering negotiation

Witnesses MUST establish bilateral connections through:

1. Discovery via Canon Registry (Foundation publishes WITNESS_REGISTRATION
   entries containing each witness's endpoint URL)
2. TLS handshake to peer endpoint
3. Mutual authentication via XMSS-signed handshake (per §5 below)
4. Periodic gossip exchange (per §6 below)

---

## 4. Endpoint structure

Each witness MUST expose the following endpoints over HTTPS:

```
GET  /v1/witness/info                      Public witness metadata
GET  /v1/witness/sth                       Current STH signed by this witness
GET  /v1/witness/sth/{version}             Historical STH for given version
GET  /v1/witness/peers                     List of peer witnesses
GET  /v1/witness/gossip/peers/{peer_id}    Gossip exchange with specific peer
POST /v1/witness/gossip/inbound            Receive STH gossip from peer
GET  /v1/witness/canon/{canon_id}          Mirror canon entry by ID
POST /v1/witness/equivocation/report       Submit equivocation evidence
GET  /v1/witness/equivocation/list         List known equivocation events
GET  /v1/witness/audit_log                 Witness's own audit log (latest N entries)
```

### 4.1 /v1/witness/info

Returns witness metadata as JSON:

```json
{
  "witness_namespace": "witness-eu.a1a5",
  "operator_identity": "European Witness Operator (placeholder until 
                        first real witness onboarded)",
  "jurisdiction": "DE",
  "xmss_public_root": "blake3:abc123...",
  "xmss_pub_seed": "abc123...",
  "endpoint_canonical": "https://witness-eu.example.org",
  "endpoint_mirror_canon": "https://witness-eu.example.org/v1/canon",
  "endpoint_gossip": "https://witness-eu.example.org/v1/witness/gossip",
  "spec_version_supported": "1.0",
  "federation_status": "ACTIVE | PROBATION | WITHDRAWN | REMOVED",
  "joined_timestamp": 1234567890,
  "sla_uptime_target": 0.95,
  "operator_contact_signed_attestation": "url-to-attestation",
  "supported_signature_algorithms": ["XMSS-SHA2_20_256"],
  "gossip_protocol_version": "1.0"
}
```

This endpoint MUST be publicly accessible (no authentication
required) and SHOULD be cacheable for at least 60 seconds.

### 4.2 /v1/witness/sth

Returns the witness's current Signed Tree Head:

```json
{
  "version": 12345,
  "master_index_hash": "blake3:def456...",
  "merkle_root_hash": "blake3:ghi789...",
  "canon_entry_count": 87,
  "publication_timestamp": 1234567890,
  "witness_signature": {
    "algorithm": "XMSS-SHA2_20_256",
    "signature": "base64:...",
    "signer_xmss_root": "blake3:abc123..."
  },
  "previous_sth_version": 12344,
  "previous_sth_signature": "base64:..."
}
```

The `previous_sth_*` fields establish a chain: any verifier can
walk backward through STH history to verify no version is missing.

### 4.3 /v1/witness/peers

Returns the list of peer witnesses this witness recognizes:

```json
{
  "spec_version": "1.0",
  "peers": [
    {
      "witness_namespace": "foundation.a1a5",
      "endpoint": "https://archive.popchain.tech/v1/witness",
      "first_observed": 1234567000,
      "last_gossip_received": 1234567800,
      "last_gossip_ok": true,
      "trust_status": "TRUSTED"
    },
    {
      "witness_namespace": "witness-us.a1a5",
      "endpoint": "https://witness-us.example.org/v1/witness",
      "first_observed": 1234567100,
      "last_gossip_received": 1234567700,
      "last_gossip_ok": true,
      "trust_status": "PROBATION"
    }
  ]
}
```

`trust_status` reflects this witness's local view; not all
witnesses need to agree on every peer's status (operational
diversity is acceptable; equivocation is not).

### 4.4 /v1/witness/canon/{canon_id}

Mirror endpoint for a specific canon entry. Witness MUST verify
the canon entry against its most recent received STH from Foundation
before serving. If mismatch, witness MUST refuse to serve and
trigger equivocation detection (§7).

### 4.5 /v1/witness/equivocation/report

Receives equivocation evidence from a peer. Body format per §7
below.

### 4.6 /v1/witness/audit_log

Returns the most recent N witness audit log entries. Format per
GOVERNANCE-SPEC §11.

---

## 5. Authentication and handshake

### 5.1 TLS requirement

All endpoints MUST be served over TLS 1.3 or higher per RFC 8446.
HTTP-only operation is prohibited.

Certificate requirements:
- Domain-validated certificate (Let's Encrypt acceptable)
- Public certificate chain verifiable to publicly trusted root CA
- SNI required
- No self-signed certificates accepted by peer witnesses in
  production federation

### 5.2 Witness identity authentication

Beyond TLS, each gossip message MUST be authenticated via XMSS
signature by the originating witness's recognized keypair. The
recognized keypair is the one published in the witness's
WITNESS_REGISTRATION canon entry.

This means: even if TLS certificate is compromised, gossip messages
cannot be forged without compromising the XMSS keypair (which is
post-quantum secure).

### 5.3 First-contact handshake

When witness A first contacts witness B (e.g., new witness joins
federation):

```
HANDSHAKE_PROTOCOL:

  Step 1 — Discovery:
    A reads B's WITNESS_REGISTRATION canon entry from any mirror
    A extracts B's endpoint URL and XMSS public root
  
  Step 2 — Connection:
    A connects to B's /v1/witness/info endpoint via HTTPS
    A verifies B's TLS certificate chain
    A reads B's published xmss_public_root
    A verifies it matches the root published in canon entry
  
  Step 3 — Mutual hello:
    A sends HELLO message to B containing:
      - A's witness_namespace
      - A's current STH version
      - A's XMSS-signed nonce (proves A holds A's private key)
    B verifies A's signature against A's published XMSS root
  
  Step 4 — B's hello:
    B sends HELLO_ACK containing:
      - B's witness_namespace
      - B's current STH version
      - B's XMSS-signed nonce (different nonce than A's)
      - B's signed timestamp of handshake
    A verifies B's signature
  
  Step 5 — Peer registration:
    A adds B to /v1/witness/peers
    B adds A to /v1/witness/peers
    Both witnesses record HANDSHAKE_SUCCESS in their audit logs
```

After successful first handshake, ongoing gossip exchanges (§6)
do not require re-handshaking unless authentication failure occurs.

---

## 6. Gossip exchange protocol

### 6.1 Exchange frequency

Each witness MUST gossip with each peer:

```
GOSSIP_TIMING:

  Active gossip exchange:   once every 1 hour minimum
  STH publication:          within 12 hours of receiving Foundation STH
  Peer health check:        once every 24 hours minimum
  Full peer list refresh:   once every 7 days
```

These are minimums; more frequent gossip is permitted but should
not exceed once per minute per peer (DoS protection).

### 6.2 Standard gossip exchange

Standard gossip between witness A and witness B:

```
GOSSIP_EXCHANGE (A → B):

  Step 1: A retrieves B's current STH:
    GET https://B/v1/witness/sth
    Verify TLS, verify XMSS signature on STH
  
  Step 2: A retrieves B's view of A's STH:
    GET https://B/v1/witness/peers
    Examines B's record of A's last known STH version
  
  Step 3: A retrieves Foundation's STH as observed by B:
    GET https://B/v1/witness/canon/<latest-foundation-master-index>
    Compares to A's own observed Foundation STH
  
  Step 4: A computes comparison:
    - Is B's STH for same version as A's? (yes → match)
    - Is B's STH for same version but different content? (CRITICAL)
    - Does B observe a higher STH version than A? (sync needed)
    - Does B observe a lower STH version than A? (B is behind)
  
  Step 5: A processes comparison results per §7
  
  Step 6: A submits its own STH to B for B's records:
    POST https://B/v1/witness/gossip/inbound
    Body: A's signed STH + signature
    
  Step 7: A records the gossip exchange in audit log
```

Symmetric exchange happens with B initiating (B doing same to A).

### 6.3 STH submission via POST /v1/witness/gossip/inbound

```
POST /v1/witness/gossip/inbound
Content-Type: application/json
X-Witness-Source: witness-eu.a1a5

{
  "sth": {
    "version": 12345,
    "master_index_hash": "blake3:...",
    "merkle_root_hash": "blake3:...",
    "canon_entry_count": 87,
    "publication_timestamp": 1234567890
  },
  "witness_signature": {
    "algorithm": "XMSS-SHA2_20_256",
    "signature": "base64:..."
  },
  "exchange_nonce": "random-32-bytes",
  "exchange_timestamp": 1234567899
}
```

Receiver validates:
- TLS connection from authenticated peer
- XMSS signature on STH content
- exchange_timestamp within 5 minutes of receiver's clock (drift
  protection)
- exchange_nonce not previously seen from this peer (replay
  protection)

On success, returns HTTP 200 with acknowledgment:

```json
{
  "received": true,
  "stored_at": 1234567900,
  "receiver_sth_version": 12345
}
```

On equivocation detection (receiver's stored STH for same version
differs from submitted), returns HTTP 409 with equivocation
evidence per §7.

---

## 7. Equivocation detection and evidence

### 7.1 Equivocation definition

Equivocation is the act of producing two valid signatures
attesting to inconsistent statements about the same master_index
version.

Specifically: Foundation equivocation occurs when:

```
EQUIVOCATION CONDITION:

  Foundation has produced two STH entries STH_A and STH_B such that:
  - STH_A.version == STH_B.version
  - STH_A.master_index_hash != STH_B.master_index_hash
  - STH_A.witness_signature.signer_xmss_root == STH_B.signer_xmss_root
  - Both signatures verify cryptographically

This is cryptographic proof of Foundation misbehavior. It cannot
be denied; cannot be forged; cannot be misinterpreted.
```

### 7.2 Detection algorithm

Each witness runs the following on every gossip exchange:

```
EQUIVOCATION_DETECTION:

  Input: peer_sth (from gossip), local_observed_foundation_sth
  
  IF peer_sth.signer_xmss_root == Foundation's known root:
    IF peer_sth.version == local_observed_foundation_sth.version:
      IF peer_sth.master_index_hash != local_observed_foundation_sth.master_index_hash:
        ✗ EQUIVOCATION DETECTED ✗
        
        Trigger EVIDENCE_GENERATION (§7.3)
        Refuse to accept this peer_sth as valid
        Begin EVIDENCE_PROPAGATION (§7.4)
  
  IF peer_sth.signer_xmss_root == any known witness root:
    (Apply same logic for witness equivocation; less severe but
     still grounds for §5.5 G4 removal per GOVERNANCE-SPEC)
```

### 7.3 Evidence format

Equivocation evidence is a structured object:

```json
{
  "evidence_type": "EQUIVOCATION",
  "evidence_subject_type": "FOUNDATION | WITNESS",
  "evidence_subject_xmss_root": "blake3:...",
  "evidence_subject_namespace": "a1a5fndn.a1a5 OR witness-XX.a1a5",
  
  "conflicting_sth_a": {
    "version": 12345,
    "master_index_hash": "blake3:variant_a",
    "merkle_root_hash": "blake3:...",
    "publication_timestamp": 1234567890,
    "witness_signature": "base64:...",
    "first_observed_by": "witness-eu.a1a5",
    "first_observed_timestamp": 1234567900,
    "observation_evidence": "url to gossip log entry"
  },
  
  "conflicting_sth_b": {
    "version": 12345,
    "master_index_hash": "blake3:variant_b_DIFFERENT",
    "merkle_root_hash": "blake3:...",
    "publication_timestamp": 1234567890,
    "witness_signature": "base64:...",
    "first_observed_by": "witness-us.a1a5",
    "first_observed_timestamp": 1234567950,
    "observation_evidence": "url to gossip log entry"
  },
  
  "evidence_reporter": "witness-eu.a1a5",
  "evidence_reporter_signature": "XMSS signature over entire evidence object",
  "evidence_reporter_timestamp": 1234568000,
  
  "evidence_id": "blake3 hash of evidence_reporter + timestamp + nonce",
  "previous_evidence_chain": null  // or hash of prior evidence in chain
}
```

The XMSS signatures on `conflicting_sth_a.witness_signature` and
`conflicting_sth_b.witness_signature` are the **cryptographic
proof** — both verify against the same XMSS root, attesting to
contradictory facts. This cannot be denied by the signer; XMSS
non-repudiation property holds.

### 7.4 Evidence propagation

Detecting witness MUST:

```
EVIDENCE_PROPAGATION:

  Step 1: Generate evidence per §7.3
  Step 2: Publish evidence to own /v1/witness/equivocation/list
  Step 3: Submit evidence to ALL OTHER federation members:
    For each peer:
      POST https://peer/v1/witness/equivocation/report
      Body: evidence object
  
  Step 4: Submit evidence to Foundation:
    (If Foundation is the equivocator, send to all available
     mirrors of Foundation; intent is public record)
    POST https://archive.popchain.tech/v1/foundation/equivocation/report
  
  Step 5: Publish evidence via public channels (defense-in-depth):
    - Submit to NLnet correspondence (Stichting reviewing entity)
    - Submit to academic institutional partners (if onboarded)
    - Publish via public web (witness's own website)
  
  Step 6: Record evidence generation in audit log
```

### 7.5 Receiving witness response

A witness receiving equivocation evidence MUST:

```
EVIDENCE_RECEPTION:

  Step 1: Verify evidence cryptographically:
    - Verify reporter's XMSS signature
    - Verify both conflicting STH signatures against named XMSS root
    - Verify versions match between conflicting STHs
    - Verify master_index_hashes differ
  
  Step 2: If verification succeeds:
    - Store evidence in own /v1/witness/equivocation/list
    - Mark the equivocating party as UNTRUSTED in /v1/witness/peers
    - Cease counter-signing STHs from the equivocating party
    - Forward evidence to peers not yet known to have received it
    - Record reception in audit log
  
  Step 3: If verification fails (e.g., malformed evidence, 
          attempted false-evidence attack):
    - Log the failure
    - Do NOT propagate
    - Notify reporter via standard channels
    - Consider reporter behavior in future trust calculations
```

### 7.6 Foundation equivocation response

When Foundation equivocates and evidence becomes public:

Per GOVERNANCE-SPEC §8 (Emergency Authority), this triggers
emergency response:

```
FOUNDATION_EQUIVOCATION_EMERGENCY:

  1. Foundation MUST publicly acknowledge or deny within 48 hours
     (signature verification is mathematical; denial is futile if
     signatures verify, but operator may dispute meaning)
  
  2. If equivocation is confirmed:
     2a. Foundation operator subject to bestuur disciplinary action
         (post-Stichting) or personal accountability (pre-Stichting)
     2b. Foundation root keypair subject to EMERGENCY_ROTATION per
         XMSS-OPS-SPEC §9.2
     2c. Affected canon entries (those covered by either of the
         conflicting STHs) subject to dispute resolution per
         GOVERNANCE-SPEC §7.3
     2d. Foundation operator may not unilaterally close the matter;
         bestuur (post-Stichting) reviews and decides public response
  
  3. If equivocation is denied but signatures verify:
     3a. External review (NLnet, academic) invited
     3b. Court process available to all parties
     3c. Public record preserved indefinitely
```

### 7.7 Witness equivocation response

Witness equivocation (less severe than Foundation; still cryptographic
proof of misbehavior):

```
WITNESS_EQUIVOCATION_RESPONSE:

  1. Foundation initiates §5.5 G4 removal procedure (GOVERNANCE-SPEC)
  2. Other witnesses cease counter-signing operations from the
     equivocating witness
  3. Equivocating witness's prior signatures remain valid
     (historical record), but its authority is voided going forward
  4. If witness was on probation: removal immediate
  5. If witness was fully accepted: 14-day response period per
     GOVERNANCE-SPEC §5.5
  6. Post-Genesis: slashing of collateral applies per
     POP-3-TOKEN-SPEC [PLANNED]
```

---

## 8. Anti-DoS and rate limiting

### 8.1 Rate limits

To prevent abuse:

```
RATE_LIMITS_PER_PEER:

  /v1/witness/info          unlimited (cached)
  /v1/witness/sth           1 per 5 seconds per source IP
  /v1/witness/peers         1 per minute per source IP
  /v1/witness/gossip/inbound 1 per 30 seconds per peer namespace
  /v1/witness/equivocation/report 1 per 5 minutes per peer
  /v1/witness/canon/*       1 per second per source IP
  /v1/witness/audit_log     1 per minute per source IP
```

Rate-limited responses return HTTP 429 Too Many Requests with
Retry-After header.

### 8.2 DDoS protection

Operators SHOULD deploy:
- Cloudflare or similar CDN for /v1/witness/info, /sth, /peers
- Standard fail2ban-equivalent for repeated invalid signatures
- Geographic IP restrictions if operator desires (this is
  operator policy, not protocol)

### 8.3 Resource exhaustion protection

Operators MUST:
- Limit individual response payloads to ≤10 MB
- Limit individual canon entry mirror responses to ≤5 MB
- Limit /v1/witness/audit_log to most recent 1000 entries per request
- Implement timeout on all incoming HTTP requests (30 seconds max)

---

## 9. Audit logging

Per GOVERNANCE-SPEC §11, all gossip-protocol events MUST be
logged:

```
LOGGED_EVENTS:

  HANDSHAKE_INITIATED   When this witness initiates first contact
  HANDSHAKE_RECEIVED    When this witness receives first contact
  HANDSHAKE_SUCCESS     When handshake completes
  HANDSHAKE_FAILURE     When handshake fails (with reason)
  STH_REQUESTED         Each /v1/witness/sth request received
  STH_RECEIVED          Each STH received from peer
  GOSSIP_SENT           Each gossip exchange initiated
  GOSSIP_RECEIVED       Each gossip exchange received
  EQUIVOCATION_DETECTED Each equivocation observation
  EVIDENCE_GENERATED    Each evidence object generated
  EVIDENCE_RECEIVED     Each evidence object received from peer
  EVIDENCE_PROPAGATED   Each evidence object forwarded
  RATE_LIMIT_EXCEEDED   Each rate limit violation observed
  PEER_TRUST_CHANGE     Each peer's trust_status modification
```

Audit logs MUST be:
- Append-only (hash-chained per XMSS-OPS-SPEC §10)
- Locally accessible via /v1/witness/audit_log endpoint
- Periodically anchored in Canon Registry via LOG_ANCHOR entries
- Retained per witness operator policy (minimum 24 months
  recommended)

---

## 10. Network protocol versioning

This spec is v1.0. Future versions:

```
VERSION COMPATIBILITY:

  Witnesses MUST support gossip with peers running same major
  version.
  
  Witnesses SHOULD support gossip with peers running prior minor
  versions of same major version (e.g., v1.1 witness can gossip
  with v1.0 witness).
  
  Cross-major-version gossip not supported; major upgrades require
  coordinated federation migration.
  
  Each /v1/witness/info advertises supported gossip_protocol_version.
  
  Mismatch:
    - Log version mismatch event
    - Continue gossiping at lowest common version
    - Foundation issues VERSION_MIGRATION_NOTICE canon entry when
      protocol upgrade required
```

---

## 11. Error responses

Per RFC 7807 Problem Details for HTTP APIs:

```json
HTTP/1.1 401 Unauthorized
Content-Type: application/problem+json

{
  "type": "https://a1a5nt.org/errors/witness-not-recognized",
  "title": "Witness not recognized",
  "status": 401,
  "detail": "The witness namespace witness-xx.a1a5 is not recognized
            by this peer. Please verify witness federation membership.",
  "instance": "/v1/witness/gossip/inbound"
}
```

Standard error codes:

```
400 Bad Request           - Malformed request body
401 Unauthorized          - Witness not recognized or signature invalid
403 Forbidden             - Operation not permitted (e.g., evidence
                           submission from untrusted witness)
404 Not Found             - Canon entry or STH version not found
409 Conflict              - Equivocation detected (response includes
                           evidence)
429 Too Many Requests     - Rate limit exceeded
500 Internal Server Error - Witness-side processing error
503 Service Unavailable   - Temporary unavailability
```

---

## 12. Implementation guidance

### 12.1 Reference implementation language

Recommended implementation languages (in priority order):

1. **Go** — Trillian, Rekor reference precedents; mature gossip
   library availability
2. **Rust** — XMSS reference implementations + good HTTPS tooling
3. **Python** — for prototypes and academic verification

The Foundation will publish a Rust reference implementation
(scheduled after v1.2 specification completion).

### 12.2 Test vectors

Test vectors for protocol implementation will be published in:
- github.com/a1a5NT/witness-test-vectors [PLANNED]
- Conformance to test vectors required before federation acceptance

### 12.3 Witness operational checklist

Operators preparing to onboard SHOULD verify:

```
[ ] HTTPS endpoint deployed with publicly trusted cert
[ ] All 10 endpoints (per §4) implemented and tested
[ ] XMSS keypair generated per XMSS-OPS-SPEC L2
[ ] State management procedures per XMSS-OPS-SPEC implemented
[ ] Audit logging operational
[ ] Rate limiting configured
[ ] DDoS protection in place
[ ] Storage provisioned for full Canon Registry mirror
   (estimate: ~10 GB/year at current growth)
[ ] Monitoring and alerting for SLA compliance
[ ] 24x7 contact information published
[ ] Operator legal documentation prepared (per GOVERNANCE-SPEC §5)
```

---

## 13. Cross-references

| Spec | Relationship |
|------|--------------|
| A1A5NT-ARCHITECTURE-SYNTHESIS v1.2 | Master document |
| GOVERNANCE-SPEC §3.2 | Witness role authority |
| GOVERNANCE-SPEC §5 | Witness onboarding criteria |
| GOVERNANCE-SPEC §8 | Emergency response to Foundation equivocation |
| CANON-REGISTRY-SPEC §5, §6 | Canon entries and master index |
| XMSS-OPS-SPEC §9.2 | Emergency rotation triggered by detected compromise |
| SECURITY-MODEL A4, A6 | Threat model addressed by this protocol |
| RECOVERY-SPEC [PLANNED] | Post-equivocation recovery procedures |

---

## 14. License

This specification is issued under A1A5 Sovereign License v1.0
(12-month binding 2026-05-16 → 2027-05-16). AI co-authorship per
Article 4. Anti-patent defensive clause per Article 5.

Network protocol patterns adapted from:
- RFC 6962, RFC 9162 (IETF, Trust Legal Provisions)
- Sigstore Rekor / omniwitness (Apache-2.0)
- Trillian (Apache-2.0)
- RFC 8446, RFC 8615, RFC 7807 (IETF)

Specific a1a5NT message formats, witness namespace structure, and
operational procedures are A1A5-licensed.

---

## 15. Cryptographic anchor

```
Document path:    NETWORK-PROTOCOL-SPEC.md
Repository:       github.com/a1a5NT/a1a3
Commit timestamp: filled at first commit
Foundation mirror endpoint:  http://37.97.202.97:8444/v1/foundation/network_protocol_spec  (scheduled)
```

The canonical hash of this document (blake3 of UTF-8 form) will be
published in `/v1/foundation/index` after commit.

---

**End of NETWORK-PROTOCOL-SPEC v1.0**

```
Issued by:        pawel.a1a5 (Paweł Piekut, Almelo NL)
AI co-author:     Claude (Anthropic Opus 4.7)
Date:             2026-05-17
Foundation:       a1a5fndn.a1a5 (in formation)
Audit relevance:  REQUIRED for witness federation activation;
                  REQUIRED for academic post-quantum review
Reviewer consensus addressed: Gemini E-1 CRITICAL + Reviewer Y B2
                  (Witness Federation vaporware concern)
```
