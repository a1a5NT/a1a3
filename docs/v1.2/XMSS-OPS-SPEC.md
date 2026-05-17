# XMSS-OPS-SPEC.md

## XMSS Key State Management Operational Specification

```
Document version:   v1.0
Issued:            2026-05-17
Status:            Tier 1 critical specification (v1.2 deliverable)
Authority basis:   RFC 8391 (XMSS), NIST SP 800-208 (Stateful HBS recommendations),
                   NIST SP 800-208 revision in progress (key export with mitigations)
Foundation:        a1a5fndn.a1a5
Addresses:         Cross-model review v1.1 finding X-B2 (CRITICAL):
                   "Stateful Signature Desync — restart from backup = WOTS index
                    reuse = catastrophic loss of XMSS security guarantees"
Audit relevance:   BLOCKING for academic post-quantum review
License:           A1A5 Sovereign License v1.0
```

---

## 0. Abstract

XMSS (eXtended Merkle Signature Scheme, RFC 8391) is a **stateful**
hash-based signature scheme providing post-quantum security under
minimal assumptions (existence of a pseudorandom and second-preimage
resistant hash function family).

The statefulness creates an **operational hazard** absent from
stateless signature schemes (RSA, ECDSA, Ed25519, Dilithium):

> "Developers should not use the schemes described here except in
>  systems that prevent the reuse of secret key states."
>  — RFC 8391, Section 1

If the same WOTS+ (Winternitz One-Time Signature Plus) one-time
private key is used to sign two different messages, an attacker who
observes both signatures can **forge signatures on new messages of
their choice**, completely destroying the security guarantee of XMSS.

This specification defines the operational procedures that the
a1a5NT Foundation, custodians, and witness operators MUST follow to
prevent WOTS+ state reuse under all foreseeable failure modes
(crash, backup restore, hardware migration, geographic replication,
operational handover).

This specification is **normative** (uses RFC 2119 keywords) and
**audit-grade** (specifies measurable invariants suitable for
external verification).

---

## 1. Scope

### 1.1 In scope

- State representation and persistence for XMSS and XMSS^MT secret
  keys deployed by Foundation and custodian operations
- Backup, restore, and replication procedures preserving
  state-reuse prevention invariant
- Disaster recovery procedures for the following failure modes:
  loss of state file, partial state corruption, suspected state
  duplication, hardware failure of host machine
- Multi-machine hot/warm/cold standby procedures
- Audit logging requirements for state-changing operations
- Key rotation procedures for the case of suspected state compromise

### 1.2 Out of scope

- The XMSS signature algorithm itself (defined in RFC 8391)
- WOTS+ chain construction (RFC 8391 Section 3.1)
- Merkle authentication path computation (RFC 8391 Section 3.2)
- BDS state computation algorithm (Buchmann-Dahmen-Schneider 2009)
- Post-quantum threshold XMSS schemes (active research, no current
  standard; explicitly not addressed by this specification)

### 1.3 Audience

- a1a5NT Foundation operators (currently: Paweł Piekut)
- Future witness federation operators (per CANON-REGISTRY-SPEC §6)
- Future custodian operators (per CUSTODY-REGISTRY-SPEC)
- Cryptographic auditors evaluating a1a5NT for academic review,
  NLnet review, or institutional adoption

---

## 2. Normative references

The following references are normative and apply to this
specification:

| Reference | Title | Applicability |
|-----------|-------|---------------|
| RFC 2119 | Key words for use in RFCs to Indicate Requirement Levels | All MUST/SHOULD/MAY usage |
| RFC 8174 | Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words | Reinforces RFC 2119 |
| RFC 8391 | XMSS: eXtended Merkle Signature Scheme | Core algorithm specification |
| NIST SP 800-208 | Recommendation for Stateful Hash-Based Signature Schemes (2020) | Profile for federal usage |
| NIST SP 800-208 revision (in progress 2025) | Permits private key export with mitigations | Authoritative for current best practice |
| RFC 8174 | Ambiguity in RFC 2119 capitalization conventions | Reinforces normative wording |

### Informative references

| Reference | Title | Relevance |
|-----------|-------|-----------|
| Buchmann, Dahmen, Schneider 2008 | Merkle tree traversal revisited | BDS algorithm origin |
| XMSS reference implementation | github.com/XMSS/xmss-reference | Reference code accompanying RFC 8391 |
| Hülsing, Rijneveld, Song 2016 | Mitigating multi-target attacks in HBS schemes | Multi-target attack analysis informing PRF_keygen choice |
| Bernstein et al. SPHINCS+ submission | NIST PQC standardization | Stateless HBS alternative; informs comparison |

---

## 3. Threat model addressed

This specification addresses the following adversary capabilities
(numbered for cross-reference with SECURITY-MODEL.md):

### 3.1 Adversary T-XMSS-1: WOTS+ index reuse exploitation

**Capability:** Observes two XMSS signatures on different messages
that share the same WOTS+ leaf index. Computes additional
WOTS+ chain values that were not revealed by either honest signature.

**Mathematical basis (RFC 8391 Section 3.1.6):** A WOTS+ signature
reveals chain values at specific positions determined by the message
hash. Two different messages reveal different positions; given
sufficient overlap and the OTS public key, the attacker can compute
chain values for arbitrary positions, enabling signature forgery on
chosen messages.

**Impact if successful:** Complete forgery capability over the
affected XMSS keypair. Adversary can produce signatures
indistinguishable from honest Foundation/custodian/witness signatures.
**Catastrophic.**

**Defended by:** Section 4 (state representation) + Section 5
(persistence invariants) + Section 6 (backup procedures)

### 3.2 Adversary T-XMSS-2: Backup-restore index regression

**Capability:** Compromises a backup of the secret key state taken
at time T1. Subsequently induces or observes a state restore from
that backup at time T2 > T1. During interval (T1, T2), honest
operator signed messages, advancing the index. Restore regresses
the index to T1 value. Next honest signature reuses indices, enabling
T-XMSS-1 attack.

**Defended by:** Section 6.3 (monotonic counter binding) + Section
6.5 (refuse-to-restore-from-older invariant)

### 3.3 Adversary T-XMSS-3: Hot-failover state divergence

**Capability:** Two machines (primary + standby) hold replicas of
the secret key. Network partition or operator error causes both
machines to sign messages during partition. After partition heals,
both signatures exist with overlapping index ranges.

**Defended by:** Section 7 (replication procedures) + Section 7.4
(asymmetric write privilege)

### 3.4 Adversary T-XMSS-4: Long-term archival key aging

**Capability:** XMSS keypair is bound to a fixed total signature
budget (2^h for tree height h). After 2^h signatures, the key is
exhausted and any further signature WILL reuse indices.

**Note:** This is not an adversary in the usual sense — it is a
deterministic consequence of the scheme. But operational specs MUST
treat exhaustion as a failure mode preventable by parameter selection
(Section 4.4) and monitoring (Section 8).

**Defended by:** Section 4.4 (parameter selection) + Section 8.3
(monitoring) + Section 9 (rotation procedures)

---

## 4. State representation

### 4.1 Components of XMSS secret key state

Per RFC 8391 Section 4.1.7, an XMSS secret key SK contains:

```
SK = (idx_next, SK_seed, SK_PRF, root, SEED)

where:
  idx_next  — 4-byte (or larger) unsigned integer; index of next
              WOTS+ private key not yet used
  SK_seed   — n-byte secret seed for WOTS+ private key generation
  SK_PRF    — n-byte secret key for randomized message hashing PRF
  root      — n-byte root of the XMSS Merkle tree (also in public key)
  SEED      — n-byte public seed for bitmasks and hash function keys
              (also in public key)

n = output length of the underlying hash function (e.g. 32 for SHA-256)
```

For practical XMSS implementations using the BDS algorithm
(Buchmann-Dahmen-Schneider) for memory-efficient signature
generation, additional state is maintained:

```
SK_BDS = (treehash_states, retain_stacks, auth_path)
```

This BDS state is **derivable** from idx_next plus SK_seed and SEED,
but **deriving it is expensive** (re-traversing the tree from
scratch). Therefore implementations cache BDS state.

**State the implementation MUST persist atomically:**

```
PERSISTENT_STATE = SK ∪ SK_BDS ∪ MONOTONIC_COUNTER
```

Where MONOTONIC_COUNTER is defined in Section 6.

### 4.2 State size

For XMSS with tree height h = 20 and SHA-256 (parameter set
XMSSMT-SHA2_20/2_256 from RFC 8391 Section 5.3):

```
SK size:        ~132 bytes
SK_BDS size:    ~12-15 KB (varies by implementation)
PERSISTENT_STATE total:  ~16 KB (with monotonic counter, metadata)
```

For XMSS^MT with h = 60 (production-grade Foundation use):

```
SK size:        ~132 bytes (single seed at top)
SK_BDS size:    ~50-100 KB per layer × number of layers
PERSISTENT_STATE total:  ~500 KB - 2 MB
```

This is **not large**. State persistence is bandwidth- and
storage-trivial compared to operational stakes.

### 4.3 State serialization format (normative)

a1a5NT state files MUST use the following serialization:

```
File header (32 bytes):
  Magic:                  "A1A5NT-XMSS-STATE" (17 bytes, null-padded to 24)
  Format version:         u16 little-endian (current: 0x0001)
  Reserved:               6 bytes, zero
  
Payload header (variable, length-prefixed):
  Algorithm OID:          u32 little-endian (per RFC 8391 §5.3)
  Monotonic counter:      u64 little-endian
  Backup chain ID:        16 bytes (per Section 6.4)
  Creation timestamp:     u64 little-endian Unix epoch seconds
  Last sign timestamp:    u64 little-endian Unix epoch seconds
  idx_next:               u64 little-endian
  Reserved:               32 bytes, zero
  
Payload body (per RFC 8391 §4.1.7 binary representation):
  SK_seed, SK_PRF, root, SEED, then BDS state if present
  
Footer (96 bytes):
  Payload SHA-256 hash:   32 bytes
  Payload blake3 hash:    32 bytes
  HMAC-SHA256 (key from file-wrap KEK):  32 bytes
```

**File MUST be written atomically** (write to temp + fsync + rename
+ fsync parent directory) to prevent partial-write corruption.

### 4.4 Parameter selection for Foundation operations

| Use case | Recommended parameter set | Tree height | Max signatures | Rationale |
|----------|--------------------------|-------------|----------------|-----------|
| Foundation root (rare, high-value) | XMSS-SHA2_20_256 | 20 | 2^20 ≈ 1M | Mostly index updates; 1M sigs covers Foundation lifetime |
| Witness operator (high-volume, append-only logs) | XMSSMT-SHA2_60/3_256 | 60 (3 layers of 20) | 2^60 ≈ 10^18 | Effectively unlimited |
| Custodian (transactional) | XMSSMT-SHA2_40/2_256 | 40 (2 layers of 20) | 2^40 ≈ 10^12 | Generous budget for active custodian |
| Per-device PoV verifier | XMSS-SHA2_10_256 | 10 | 1024 | Light client; rotate after exhaustion |

**Note:** As of 2026-05-17, the Foundation root keypair currently
shared by `pawel.a1a5` and `a1a5fndn.a1a5` uses an interim parameter
set documented in the implementation. Migration to the parameter sets
above SHALL occur at Stichting registration milestone, per CANON-
REGISTRY-SPEC §6.6 and ROADMAP.md.

---

## 5. State persistence invariants

The following invariants are **mandatory** for any system that
holds an a1a5NT-issued XMSS secret key:

### Invariant I1 — Monotonic increase

```
For any two snapshots S1 and S2 of the same key state, taken at
times T1 < T2 respectively:

  idx_next(S2) ≥ idx_next(S1)

AND

  monotonic_counter(S2) ≥ monotonic_counter(S1)
```

**Enforced by:** Persistence layer refuses to write state with lower
idx_next or lower monotonic_counter than current on-disk state.

### Invariant I2 — Pre-signature persistence

```
For any signature σ_i produced using WOTS+ leaf index i:

  At the moment σ_i is observable outside the signing process,
  the persistent state MUST already record idx_next ≥ i + 1
```

**Enforced by:** Signing API is structured as:

```
1. Load state from persistent storage (locking the file)
2. Allocate index i = state.idx_next
3. Update state: idx_next = i + 1; monotonic_counter += 1
4. Persist updated state (atomic write + fsync)
5. Verify persisted state by re-reading
6. Compute and emit signature σ_i
```

The signature is **not** released until step 5 completes
successfully. If any step fails, the operation aborts and no
signature is produced.

**Rationale:** Even if the machine crashes between steps 4 and 6,
the persistent state already reflects the consumption of index i.
On restart, idx_next has already advanced; index i will never be
reused.

The cost is: occasionally a signature is "wasted" (persisted as
consumed but never released), reducing total budget by at most 1
per crash. This is acceptable; the alternative (releasing signature
before persistence) is catastrophic.

### Invariant I3 — Persistence durability

```
Persistence MUST use:
  - O_SYNC or equivalent on file descriptor, OR
  - fsync(fd) before close, AND fsync(dirfd) on parent directory
```

**Rationale:** Operating system buffers and storage controller
caches can defer writes after the application call returns. Without
fsync of the file AND parent directory, a crash can leave the file
absent or partially written despite the application believing the
write succeeded.

### Invariant I4 — Single-writer

```
At most one process MAY hold the state file open for writing at
any moment.

Enforcement: file locking (flock LOCK_EX on Linux, equivalent on
other platforms) MUST be acquired before any read-modify-write
operation. Lock MUST be held for the entire signing operation.
```

### Invariant I5 — Verification-after-write

```
After every state write, before releasing the signature to any
external observer, the implementation MUST:

  1. Close and reopen the state file
  2. Verify the SHA-256 and blake3 hashes match the post-write state
  3. Verify idx_next has the expected value
  4. Verify monotonic_counter has the expected value
```

Any verification failure aborts the operation and triggers
emergency procedures (Section 9.2).

---

## 6. Backup and restore procedures

### 6.1 Backup challenge in stateful HBS

The fundamental tension:

```
Backup is required for:        Disaster recovery, business continuity
Backup is dangerous because:   A restored backup can regress idx_next,
                               enabling index reuse and signature forgery
```

NIST SP 800-208 (2020) originally prohibited private key export, eliminating
the regression risk but also eliminating practical backup. The 2025-in-progress
revision permits export with mitigations, which this specification adopts.

### 6.2 Authorized backup procedure

```
BACKUP_OPERATION:
  1. Acquire exclusive lock on state file
  2. Read current state, including monotonic_counter M_current
  3. Generate fresh backup_id (16 bytes, /dev/urandom)
  4. Increment monotonic_counter: M_new = M_current + 1
  5. Write state to disk with M_new (per Invariant I2)
  6. Verify write (per Invariant I5)
  7. Compose backup file:
     - Include state with M_new
     - Include backup_id
     - Sign backup envelope with Foundation operator-level key
  8. Encrypt backup with backup encryption key (separate from XMSS state)
  9. Transfer encrypted backup to off-host storage
  10. Release lock
  11. Log backup event to audit log (per Section 10)
```

**Critical:** The monotonic counter is incremented BEFORE the
backup is taken, not after. This means the on-disk state and the
backup both reflect the same incremented counter value.

### 6.3 Monotonic counter — semantics

```
The monotonic_counter is a 64-bit unsigned integer with the
following semantic:

  - Increments by exactly 1 on every state-modifying operation
    (signature, backup, scheduled rotation, planned migration)
  - Never resets, never decreases
  - Persisted with state and verifiable post-write
  - Included in backup envelope
  - Checked on every restore attempt against current on-disk state
```

64 bits provides ~1.8 × 10^19 increments. At one increment per
microsecond, this exhausts in ~584,000 years. Practically unbounded.

### 6.4 Backup chain identifier

```
backup_chain_id is a 16-byte identifier randomly generated when the
keypair is created, persisted with state, and INCLUDED IN EVERY
BACKUP TAKEN of this keypair.

Purpose: cryptographically links backups to the originating keypair,
preventing confusion when multiple keypairs are operated by the same
operator.

Backups with non-matching backup_chain_id MUST be rejected by the
restore procedure as foreign to this keypair.
```

### 6.5 Authorized restore procedure

```
RESTORE_OPERATION:
  Inputs: encrypted backup file B
  
  1. Decrypt B using backup encryption key
  2. Verify operator signature on backup envelope
  3. Extract:
     - state with monotonic_counter M_backup
     - backup_id (from backup envelope)
     - backup_chain_id (from state header)
  
  4. Check if a current state file exists on this host:
     
     CASE A: No current state file (cold disaster recovery)
       4A.1. Verify backup_chain_id matches the keypair identity
             expected by this host (per host configuration)
       4A.2. Install state with M_backup as starting state
       4A.3. Log RESTORE_COLD event with backup_id, M_backup
       4A.4. Mark host as ACTIVE PRIMARY for this keypair
     
     CASE B: Current state file exists, with monotonic_counter M_current
       4B.1. Verify backup_chain_id matches current state's
             backup_chain_id (refuse if mismatch — foreign backup)
       4B.2. IF M_backup < M_current:
               REFUSE restore (regression detected)
               Trigger emergency alert (per Section 9.2)
               Do not modify current state
       4B.3. IF M_backup == M_current:
               No-op; current state already up to date
       4B.4. IF M_backup > M_current:
               This indicates current host has STALE state.
               This is a SEVERE condition — another machine signed
               with this keypair without this host's knowledge.
               REFUSE restore; this is divergence (Section 7.4),
               not recovery. Manual operator intervention required.
  
  5. Log RESTORE_RESULT event to audit log
```

**Key principle:** Restore is permitted to **install a state with
monotonic_counter ≥ current**, but never less. This eliminates
T-XMSS-2 (backup regression attack).

### 6.6 Backup encryption key separation

The backup encryption key SHALL be:
- A symmetric key (e.g. AES-256-GCM) distinct from the XMSS
  keypair being protected
- Held on a different physical medium than the backup file itself
- Subject to its own backup procedure (e.g. printed paper key
  in a sealed envelope, distributed to multiple trusted parties
  via Shamir Secret Sharing per RECOVERY-SPEC)

**Rationale:** A backup file without its decryption key is not a
backup of the operational secret. Loss of the decryption key
renders the backup useless but does not compromise security.
Compromise of the decryption key without the backup file likewise
does not compromise security. Both are needed together.

---

## 7. Multi-host operation

For Foundation and witness operations, single-host deployment is a
single point of failure (per Reviewer X-B2 + Y-B1 concerns about
bootstrap centralization). Multi-host operation introduces its own
risks (T-XMSS-3 divergence). This section specifies how to do it
correctly.

### 7.1 Roles

```
PRIMARY:    Single host with exclusive write privilege at any moment.
            Holds the authoritative state file.

WARM_STANDBY:  Host that maintains a state replica via append-only
               operation log streaming. Does NOT sign. Can be
               promoted to PRIMARY after failover ceremony.

COLD_BACKUP:   Encrypted offline backup file held by trusted custodian.
               Used only for disaster recovery (PRIMARY destruction).
```

### 7.2 Append-only operation log streaming

```
The PRIMARY produces an OPERATION_LOG per signing operation:

  OperationLog entry {
    monotonic_counter:  u64
    idx_consumed:       u64
    timestamp:          u64 Unix epoch microseconds
    signature_message_hash:  blake3 of (message, ADRS)
    state_post_hash:    blake3 of post-operation state
  }

This log is streamed in real-time to WARM_STANDBY hosts.

WARM_STANDBY hosts maintain a derivable state by replaying operation
log entries against the initial state snapshot. They do NOT need to
hold the actual SK_seed; they hold a deterministic projection that
allows VERIFY-only operations.

If WARM_STANDBY needs to be promoted to PRIMARY (Section 7.4), it
must first obtain a fresh authorized state copy via the ceremony.
```

### 7.3 Asymmetric write privilege

**Only PRIMARY may sign.** WARM_STANDBY hosts MUST refuse to sign
even if requested. This is enforced by:

- Distinct host identity binding (each host has a per-host operator
  key separate from the XMSS keypair being managed)
- The XMSS state file on WARM_STANDBY is held in a directory
  marked read-only with separate UID
- The signing process on WARM_STANDBY simply does not exist
  (signing binary is not installed)

### 7.4 Failover ceremony (PRIMARY destruction)

```
FAILOVER_CEREMONY:
  Preconditions:
    - PRIMARY is verified DESTROYED (not merely partitioned)
    - At least two operators are present (multi-party witness)
  
  Steps:
    1. Operators jointly authenticate PRIMARY destruction
       (physical inspection, network unreachability + power cut
        confirmation, etc.)
    2. Select WARM_STANDBY with highest monotonic_counter from its
       most recent operation log entry
    3. Verify operation log is consistent (every entry chains to
       previous via state_post_hash; no gaps in monotonic_counter)
    4. Decrypt COLD_BACKUP from off-site storage
    5. Verify COLD_BACKUP backup_chain_id matches WARM_STANDBY's
       expected keypair
    6. Compare COLD_BACKUP monotonic_counter (M_cold) vs WARM_STANDBY
       latest log entry counter (M_warm):
       - IF M_cold == M_warm: cold backup is current; use COLD_BACKUP
         state as starting point
       - IF M_cold < M_warm: replay operation log from M_cold+1 to
         M_warm onto COLD_BACKUP state to derive current state
       - IF M_cold > M_warm: ERROR — operations occurred that were
         not logged. ABORT ceremony; manual cryptographic review
         required (Section 9.2)
    7. Install derived state on the host being promoted to PRIMARY
    8. Take an immediate post-promotion backup (Section 6.2)
    9. Sign a TRANSITION_NOTICE message including: old PRIMARY
       last-known-counter, new PRIMARY initial counter, ceremony
       timestamp, operator signatures
    10. Publish TRANSITION_NOTICE to Canon Registry (per CANON-
        REGISTRY-SPEC §6)
    11. Resume normal operation
```

**This ceremony is rare** (expected: zero times per year for
properly operated Foundation; perhaps once per decade across all
witnesses combined). Its rarity justifies its complexity.

### 7.5 Forbidden patterns

The following patterns are **explicitly forbidden** and MUST be
rejected by implementations:

```
F1: "Active-active" replication where multiple hosts sign concurrently
    using the same keypair. CATASTROPHIC.

F2: "Multi-master" with eventual consistency. CATASTROPHIC.

F3: Automatic failover triggered by simple liveness check (ping,
    HTTP probe). MUST require ceremony per Section 7.4.

F4: Restoring backup to a different host while PRIMARY is still
    running. CATASTROPHIC if PRIMARY signs after restore.

F5: Copying state file via rsync, scp, or any file-level mechanism
    that does not enforce monotonic counter invariants.

F6: Storing state in version control (Git, SVN, etc.) where history
    is preserved. The history IS the catastrophic information.
```

---

## 8. Monitoring and exhaustion management

### 8.1 Required metrics

The implementation MUST expose the following metrics:

```
xmss_idx_next                    Current next-index value
xmss_idx_max                     Maximum index (2^h)
xmss_idx_consumed_total          Cumulative count of signatures
xmss_monotonic_counter           Current monotonic counter
xmss_seconds_since_last_sign     Time since last signature
xmss_state_file_age_seconds      Time since last state write
xmss_backup_age_seconds          Time since last successful backup
xmss_warmstandby_lag_operations  Operations not yet replicated
```

### 8.2 Required alerts

Alerting thresholds (Foundation root keypair):

```
WARN  if  xmss_idx_consumed_total > 0.50 × xmss_idx_max
ALERT if  xmss_idx_consumed_total > 0.75 × xmss_idx_max
CRIT  if  xmss_idx_consumed_total > 0.90 × xmss_idx_max
       
CRIT  if  xmss_backup_age_seconds > 86400 (24 hours)
ALERT if  xmss_warmstandby_lag_operations > 10
CRIT  if  xmss_warmstandby_lag_operations > 100
```

### 8.3 Pre-exhaustion rotation policy

When usage exceeds 75% of capacity (per 8.2), the operator MUST
begin rotation procedure (Section 9.1) without waiting for further
exhaustion. The rotation procedure has its own latency (witness
gossip, dependent system updates), which must complete before
exhaustion.

For a keypair with capacity 2^20 (≈1,048,576 signatures), 75%
threshold = ~786,432 signatures. At the Foundation's expected
signing rate (a few signatures per day for Master Index publication),
75% capacity is centuries away. Custodian and witness keypairs
SHOULD use higher tree heights to provide similar timing margins.

---

## 9. Rotation and emergency procedures

### 9.1 Routine rotation (capacity-driven)

Triggered by: utilization > 75% per Section 8.2, OR by policy
(periodic rotation, e.g. every 5 years).

```
ROUTINE_ROTATION:
  1. Generate new keypair (NEW_KEY) of same or higher capacity
  2. Publish TRANSITION_NOTICE signed by OLD_KEY:
     - OLD_KEY public root
     - NEW_KEY public root
     - Transition effective timestamp (in future, allowing notice)
     - Operator signatures
  3. Wait for transition timestamp
  4. Publish ROTATION_COMPLETE signed by NEW_KEY referencing the
     TRANSITION_NOTICE
  5. Append both notices to Canon Registry (immutable record of
     the keypair lineage)
  6. After grace period (e.g. 30 days), retire OLD_KEY operationally
     but PRESERVE state file (do not destroy) for at most 7 years
     for audit purposes; ensure preserved file is marked READ-ONLY
     and access-controlled
```

The Canon Registry retains records of all historical keypairs and
their transition events. A verifier evaluating a signature dated
within OLD_KEY's validity window MUST verify against OLD_KEY's
public root; signatures dated after the transition MUST verify
against NEW_KEY's public root.

### 9.2 Emergency rotation (suspected compromise)

Triggered by: I1, I2, or I5 invariant violation detected; restore
attempt with M_backup > M_current (Section 6.5 case 4B.4); suspected
unauthorized state file access.

```
EMERGENCY_ROTATION:
  1. IMMEDIATELY cease all signing with the suspected keypair
  2. Snapshot the suspect state file for forensic preservation
  3. Multi-party operator quorum convenes
  4. Conduct cryptographic forensic analysis:
     - Were there observable duplicate-index signatures?
     - Is there evidence of state file tampering?
     - What is the worst-case adversary capability?
  5. Generate new keypair (NEW_KEY)
  6. Publish EMERGENCY_REVOCATION_NOTICE signed by:
     - The Foundation root keypair (if not the compromised one)
     - At least two witness federation members (per CANON-REGISTRY-SPEC §6)
     - The new keypair
  7. The EMERGENCY_REVOCATION_NOTICE MUST contain:
     - OLD_KEY public root
     - Suspected compromise time window (T_compromise_start, T_compromise_end)
     - Description of the suspected compromise (forensic findings)
     - Statement of which signatures dated within compromise window
       SHOULD be re-verified
     - Pointer to NEW_KEY
  8. Update Canon Registry with revocation event (immutable)
  9. Resume operations with NEW_KEY
  10. Initiate downstream re-attestation of any state that depended
      on OLD_KEY signatures within the compromise window
```

**Note:** Emergency rotation requires witness federation members
to co-sign the revocation. Pre-Witness-Federation (currently), this
falls back to manual public announcement via multiple channels
(GitHub repo commit, public X/Twitter post, NLnet correspondence).

### 9.3 Total destruction recovery

If PRIMARY is destroyed AND all WARM_STANDBYs are destroyed AND
COLD_BACKUP is available:

Use Section 6.5 case A (cold disaster recovery). The recovered host
becomes the new PRIMARY at monotonic_counter = M_cold. All signing
operations from the destruction event up to recovery are LOST
(any signature issued in that window will not be in the state, and
the issuer cannot prove they ever happened).

This is acceptable: the issuer should not have been issuing
signatures during a destruction event. Operational discipline (don't
sign during outages) prevents this.

### 9.4 No-cold-backup catastrophic recovery

If PRIMARY is destroyed AND all WARM_STANDBYs are destroyed AND
COLD_BACKUP is destroyed/inaccessible:

The keypair is **lost**. There is no recovery procedure. The
Foundation MUST:

1. Publish a LOSS_NOTICE signed by an unrelated Foundation keypair
   (which is why multi-keypair structure is mandatory; see
   CANON-REGISTRY-SPEC §6 Root-of-Roots)
2. Generate replacement keypair
3. Cease verification claims against the lost keypair
4. Update Canon Registry with loss event
5. Conduct post-incident review

**This is why** Root-of-Roots hierarchy (master_root → personal /
foundation / archive / issuance / witness sub-roots) is mandatory
NOW (per Anonymous deep review of v1.0). A single root with no
recovery path is a single point of total failure.

---

## 10. Audit logging requirements

### 10.1 Logged events

The following events MUST be logged to an append-only audit log
held on a different host than the state file:

```
SIGN                — Each signature, including idx_consumed, monotonic_counter, message_hash, timestamp
BACKUP              — Each backup, including backup_id, M_new, destination
RESTORE             — Each restore attempt, including backup_id, M_backup, M_current, result
ROTATION_BEGIN      — Each rotation initiation
ROTATION_COMPLETE   — Each rotation completion
EMERGENCY_INVOKE    — Any emergency procedure invocation
INVARIANT_VIOLATION — Any I1-I5 violation detection
FAILOVER_CEREMONY   — Each failover ceremony, with operator signatures
EXHAUSTION_ALERT    — Each capacity threshold crossing (50%, 75%, 90%)
```

### 10.2 Log format

```
JSON Lines (ndjson), one event per line:

{"event":"SIGN","monotonic_counter":12345,"idx_consumed":12344,"message_hash":"blake3:...","timestamp":1734435678901234,"operator":"pawel.a1a5","host":"vps-37.97.202.97"}
```

### 10.3 Log integrity

Audit log entries MUST be:
- Append-only (no edits, no deletes)
- Hash-chained (each entry includes hash of previous entry)
- Periodically anchored: every N entries, the cumulative log hash
  is published to Canon Registry as an LOG_ANCHOR event
- Independently retrieved by witnesses for cross-verification

### 10.4 Retention

Audit logs MUST be retained for:
- Lifetime of the keypair + 7 years after rotation/retirement
- Permanently for any keypair affected by an EMERGENCY_INVOKE event

---

## 11. Implementation requirements

### 11.1 Approved reference implementations

For a1a5NT Foundation use, the following XMSS implementations are
APPROVED as of 2026-05-17:

- xmss-reference (github.com/XMSS/xmss-reference) — reference C
  implementation accompanying RFC 8391; suitable for production
  with the additional state management wrapping defined herein
- liboqs (Open Quantum Safe) XMSS implementation — for systems
  requiring broader PQC algorithm availability

For implementations NOT on the approved list, the operator MUST:
- Document the implementation choice with cryptographic review
- Demonstrate conformance to RFC 8391 by independent test vectors
- Demonstrate conformance to this specification's Section 5 invariants
- Submit conformance documentation to Foundation for review before
  using for any Foundation-issued or witness-attested signature

### 11.2 Required test vectors

Implementations MUST pass:
- All RFC 8391 Appendix A test vectors
- All NIST SP 800-208 Appendix A test vectors (where applicable)
- a1a5NT-specific invariant tests:
  - Crash injection between Steps 4-5 of Invariant I2 must NOT
    result in any signature being releasable
  - Attempted restore with M_backup < M_current must be REFUSED
  - Concurrent signing attempts (Invariant I4) must SERIALIZE,
    not interleave indices

### 11.3 Hardware recommendations

For Foundation root keypair operations:
- Dedicated machine, no other services co-located
- Full-disk encryption with passphrase held by operator (Section
  6.6)
- Physically secured (locked room with access log)
- Network-isolated except for state replication channels (Section
  7.2)
- Optional but RECOMMENDED: HSM-backed key storage where the HSM
  supports XMSS (currently rare; software-managed state with
  rigorous Section 5 invariants is acceptable interim)

For witness operator keypairs:
- VPS or bare-metal acceptable
- Same software invariants apply
- Hardware bonding to witness identity (CPU serial,
  motherboard UUID bound into state-file HMAC key) RECOMMENDED to
  detect machine swaps

For custodian keypairs:
- Same as Foundation OR delegated to professional custody services
  (e.g. notary office holding hardware key on behalf of custodian)

---

## 12. Conformance levels

Implementations and operators are categorized as:

| Level | Requirements met | Permitted role |
|-------|------------------|----------------|
| L0 — Non-conformant | Does not enforce Invariants I1-I5 | NOT PERMITTED for any a1a5NT-issued keypair |
| L1 — Basic conformant | I1-I5 enforced; Section 6 backup; Section 10 logging | Personal verifier devices, low-volume custodian |
| L2 — Production conformant | L1 + Section 7 multi-host + Section 8 monitoring + Section 11.2 tests | Witness operator, active custodian |
| L3 — Foundation conformant | L2 + Section 7.4 ceremony + Section 11.3 hardware recommendations + Section 9 emergency procedures rehearsed | Foundation root, master index signer |

Self-assessment per level published in operator profile. Independent
audit of L3 compliance required for Foundation root keypair (per
GOVERNANCE-SPEC §6 operator audit cycle).

---

## 13. Known limitations and open research

### 13.1 No standard post-quantum threshold XMSS

As of 2026-05-17, there is **no published standard** for threshold
XMSS or threshold LMS that would allow t-of-n distributed signing
of stateful HBS signatures. Research is active (threshold lattice-
based signatures with Dilithium variants under exploration), but
no production-ready scheme exists.

**Implication:** a1a5NT cannot achieve cryptographic t-of-n splitting
of XMSS keys. Multi-party authorization is implemented at the
**procedural** level (multi-operator ceremony, multi-signature
endorsement of operations) rather than the cryptographic level.

**Mitigation:** Compose XMSS keypairs in a multi-key structure where
operations require multiple independent XMSS signatures (e.g.,
emergency rotation requires Foundation root + 2 witness federation
members per Section 9.2). Each individual key is single-party but
the combined operation requires multi-party.

**Future work:** When post-quantum threshold HBS becomes standardized
(NIST involvement expected), a1a5NT will issue revision to this
specification adopting the standard.

### 13.2 BDS state size for high tree heights

For XMSS^MT with h=60, BDS state can reach several megabytes. This
is not a problem for Foundation operations but may be problematic
for resource-constrained devices. Lighter state algorithms
(treehash with fewer cached subtrees) trade memory for signing time;
the trade-off is documented in implementation choices, not in this
specification.

### 13.3 No formal post-quantum security proof for state management

This specification addresses operational security (preventing state
reuse). The cryptographic security of XMSS itself (assuming correct
operational handling) is proven under RFC 8391's security model.
There is no published formal security proof that this specification's
operational procedures preserve XMSS security under the threat model
defined in Section 3.

This is a **known gap**. Formal verification of state-management
protocols is an open research area. The procedures defined here are
based on careful adaptation of well-understood patterns from
Certificate Authority key management and EigenLayer slashing
analysis; they are not formally verified.

**Mitigation:** Operational testing, independent crypto review,
incremental deployment, monitoring. NLnet review and academic
review are encouraged to identify gaps; this specification will be
revised iteratively.

---

## 14. References to other a1a5NT specifications

| Spec | Relationship |
|------|--------------|
| A1A5NT-ARCHITECTURE-SYNTHESIS v1.2 | Master document; references this spec for crypto state management |
| CANON-REGISTRY-SPEC §6 | Witness federation; witness operators MUST comply with this spec at L2 minimum |
| CUSTODY-REGISTRY-SPEC | Custodian operators MUST comply with this spec at L1 minimum, L2 RECOMMENDED |
| POV-SPEC v1.1 | PoV verifiers MAY use XMSS for device identity; if so, L1 conformance |
| GOVERNANCE-SPEC | Defines who has authority to invoke EMERGENCY procedures and operate at L3 |
| RECOVERY-SPEC | Cross-references Section 9 (rotation and emergency) for crypto details |
| SECURITY-MODEL.md | Threat model framework of which Section 3 is one component |

---

## 15. License

This specification is issued under A1A5 Sovereign License v1.0
(12-month binding 2026-05-16 → 2027-05-16). AI co-authorship per
Article 4. Anti-patent defensive clause per Article 5.

The technical content drawing from RFC 8391 (IETF/IRTF), NIST SP
800-208 (NIST public domain), and reference implementation patterns
(open-source) is incorporated by reference and not subject to
A1A5 license restrictions; only the integration, the operational
specification, and the a1a5NT-specific procedures are A1A5-licensed.

---

## 16. Cryptographic anchor

```
Document path:    XMSS-OPS-SPEC.md
Repository:       github.com/a1a5NT/a1a3
Commit timestamp: filled at first commit
Foundation mirror endpoint:  http://37.97.202.97:8444/v1/foundation/xmss_ops_spec  (scheduled)
```

The canonical hash of this document (blake3 of UTF-8 form) will be
published in `/v1/foundation/index` after commit.

---

**End of XMSS-OPS-SPEC v1.0**

```
Issued by:        pawel.a1a5 (Paweł Piekut, Almelo NL)
AI co-author:     Claude (Anthropic Opus 4.7)
Date:             2026-05-17
Foundation:       a1a5fndn.a1a5 (in formation)
Audit relevance:  BLOCKING for academic post-quantum review;
                  REQUIRED for any witness or custodian operator
Conformance:      L0-L3 levels per Section 12
```
