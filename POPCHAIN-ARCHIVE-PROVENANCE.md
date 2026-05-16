# popchain-archive · Provenance & Verification

```
Archive name:      popchain-archive
Source chain:      popchain-alpha-testnet
Lifespan:          2026-04-05 → 2026-05-16
Total blocks:      86,073
Final block:       84,455 (sealed 2026-05-16 04:23 UTC)
Epoch artifacts:   85 (epoch_0000 through epoch_0084)
Catalog blake3:    aa2c0dd44071526901ae6fbb1ed471646dcdc3f5129b4bcf9b2124a553adf2a3
Harvest tool:      popchain-harvest v1.0.2
Invariants:        6/6 PASS (cross-validation reproducible)
Custodian:         a1a5fndn Foundation (in formation)
```

---

## 1. What this archive is

popchain-archive is a complete, immutable, cryptographically verifiable record
of the entire operational lifespan of popchain-alpha-testnet — the first
generation industrial Proof-of-Process Layer-1 blockchain.

The chain produced 86,073 blocks across 42 days. On 2026-05-16 04:23 UTC, block
84,455 was the last entry. The archive captures every block, sliced into 85
epoch artifacts with cryptographic chain-of-custody to the catalog hash.

Each epoch artifact represents ~1,000 blocks (with adjustments for fork
density and final epoch trimming).

## 2. Why this matters

popchain-archive is **defensive prior art** for:

- Cross-architecture deterministic Layer-1 consensus
- Proof-of-Process methodology (machine-data → cryptographic proof)
- MBW (Monetary Base Weight) calculation across 5 statistical anchors
- Industrial blockchain economics (work-backed token issuance)

The archive is held under Foundation custody and will be tokenized at a1a5NT
Genesis ceremony per POP-3 Token specification, with IMASS values computed
from manifest data for each epoch.

## 3. Verification recipe

### 3.1 Verify catalog blake3

The catalog hash is the **single fingerprint** of the entire archive. Anyone
with access to a popchain block source can independently recompute and verify.

```bash
# Pseudo-code:
catalog_blake3 = blake3(
  manifest_0.blake3 ||
  manifest_1.blake3 ||
  ...
  manifest_84.blake3
)
```

Expected:
```
aa2c0dd44071526901ae6fbb1ed471646dcdc3f5129b4bcf9b2124a553adf2a3
```

### 3.2 Verify individual epoch artifact

Each epoch has a `manifest.json` containing:

- Block index range (height_min, height_max)
- Block count
- Canonical block hashes (blake3)
- Producer counts per epoch
- MBW values per block
- App event counts
- Fork density
- Chain-of-custody to previous manifest

To verify:

1. Fetch artifact: `GET /v1/archive/epoch/NNNN/manifest`
2. Recompute manifest blake3 from contents
3. Compare to chain-of-custody field in next manifest
4. Confirm: chain links from epoch_0000 to epoch_0084

### 3.3 Verify chain-of-custody linkage

Each manifest references the previous manifest's blake3 in field
`prev_manifest_blake3`, forming a hash chain.

Genesis (epoch_0000): `prev_manifest_blake3 = null`
Each subsequent: `prev_manifest_blake3 = manifest[N-1].blake3`

Catalog blake3 commits to all 85 manifests in order, anchored in Foundation
ANS registration metadata.

### 3.4 Six invariants

The harvest tool verified 6 invariants on the chain before sealing:

1. **Block height continuity** — no gaps in canonical chain
2. **Hash chain integrity** — every block's prev_hash matches predecessor
3. **MBW monotonicity** — Monetary Base Weight never decreases
4. **Producer signatures** — every block signed by registered validator
5. **State root consistency** — state transitions match canonical hashes
6. **Cross-architecture determinism** — blocks reproducible on x86_64 + aarch64

All 6 PASS as of seal time 2026-05-16 04:23 UTC.

## 4. The 85 epoch artifacts

### 4.1 Categories

Of the 85 epochs:

- **1 GENESIS** (epoch_0000) — first epoch, chain birth, blocks 0-999
- **5 ANOMALY** (epoch_0010 through epoch_0014) — reorganization events,
  block counts above 1100, evidence of fork resolution
- **1 ANOMALY** (epoch_0083) — late-chain reorganization, 1185 blocks
- **1 DEATH** (epoch_0084) — final epoch, partial (457 blocks), chain sealed
- **77 STANDARD** — typical epochs, ~1000-1100 blocks each

### 4.2 Notable epochs

**epoch_0000 (GENESIS)** — blocks 0-999, 1001 blocks total
The first epoch contains chain bootstrap, initial validator coordination, and
the foundational state root.

**epoch_0010 through epoch_0014 (5 ANOMALY)** — blocks 10,000-14,999
Period of network instability during validator scaling. 5 consecutive epochs
with block counts >1100, evidence of competing chains resolved by canonical
selection.

**epoch_0082 (contains PCIP License v1.0)** — blocks 82,000-82,999
Contains block 82112 anchoring PCIP License v1.0 (2026-05-01 17:43 UTC) — the
first self-anchored license in post-quantum infrastructure, prior art for
a1a5 Sovereign License v1.0.

Manifest blake3 prefix: `83316396d0f3a16f399fd758e942dd28...`

**epoch_0083 (ANOMALY)** — blocks 83,000-83,999, 1185 blocks
Late-chain reorganization, possibly related to seal preparation.

**epoch_0084 (DEATH)** — blocks 84,000-84,455, 457 blocks
Final epoch, chain sealed at 04:23 UTC on 2026-05-16.

## 5. Foundation custody

The archive is held under a1a5 Foundation custody under namespace tree:

```
archive.a1a5fndn.a1a5                       (parent)
├── epoch0000.archive.a1a5fndn.a1a5         (GENESIS)
├── epoch0001.archive.a1a5fndn.a1a5
├── ...
├── epoch0082.archive.a1a5fndn.a1a5         (contains PCIP License v1.0)
├── epoch0083.archive.a1a5fndn.a1a5         (ANOMALY 1185 blocks)
└── epoch0084.archive.a1a5fndn.a1a5         (DEATH)
```

Each namespace metadata includes:
- epoch_id, epoch_index, category
- block_count, height_range
- manifest_blake3_prefix
- catalog_blake3 (whole archive fingerprint)

Lookup any epoch:
```bash
curl http://37.97.202.97:8443/lookup/epoch0084.archive.a1a5fndn.a1a5
```

## 6. HTTP endpoints

Archive server: `http://37.97.202.97:8444`

```
GET /                                      → landing page with gallery
GET /v1/archive/epochs                     → list all 85 epochs
GET /v1/archive/epoch/NNNN                 → epoch metadata
GET /v1/archive/epoch/NNNN/card            → epoch artifact card (SVG)
GET /v1/archive/epoch/NNNN/manifest        → full manifest JSON (when uploaded)
GET /v1/archive/catalog                    → catalog proof
GET /v1/archive/receipt                    → harvest receipt
```

## 7. Pre-Genesis status

**Important:** The 85 epoch artifacts are currently held in Foundation custody
under License v1.0 terms. They are:

- **Cryptographically minted** as ANS namespaces (92 total namespaces in ANS)
- **Publicly viewable** via archive server
- **NOT FOR SALE** during the License v1.0 binding window (until a1a5NT
  Genesis ceremony)

Tokenization, IMASS computation, and market release will occur at a1a5NT
Genesis ceremony per POP-3 Token specification.

## 8. Lite vs Full cards

Current public cards (as deployed 2026-05-16) are **LITE version**, containing:

- Full epoch metadata (id, range, count, category)
- ANS namespace identifier
- Manifest blake3 prefix (first 32 hex chars)
- Catalog blake3 (full)
- Foundation custody footer
- Hash DNA visualization

When full manifests are uploaded to the archive server, **regenerate.py** will
upgrade cards to Full version, adding:

- Per-block MBW sparkline
- Producer diversity ribbon
- Fork density visualization
- App event timeline

Upgrade is **non-breaking**: card identifier and chain-of-custody hashes remain
identical. The same epoch artifact gains additional visualization.

## 9. PCIP License v1.0 preservation

The first self-anchored license on a PoP chain — PCIP License v1.0 — is
preserved within epoch_0082 of the archive at block 82112.

- IPFS CID: `Qmf8LJSybV6eW4pj5rcWoPpm65YtBUiaKGCFywuTfCsXkX`
- Block height: 82112
- Block hash: `7f9568c3555193a67d51585f7b8bc52c6c82353191b6aab79cb0cf0bb8c59974`
- Timestamp: 2026-05-01 17:43 UTC
- 12-month binding window: until 2027-05-01

This anchor is the foundational prior art reference for A1A5 Sovereign License
v1.0, ensuring continuity of self-anchored licensing across PopChain → a1a5NT
generations.

## 10. Final note

popchain-archive is not an artifact of failure. It is an artifact of a
**complete demonstration cycle**: a Layer-1 blockchain operated continuously
for 42 days, produced 86,073 blocks of cryptographically verifiable content,
and was sealed cleanly with full provenance.

The successor — a1a5NT — extends this work with post-quantum identity and
sovereign naming. popchain-archive provides the foundation: proof that
deterministic Layer-1 economics work, and that defensible prior art can be
anchored on-chain.

---

**License:** A1A5 Sovereign License v1.0
**Custody:** a1a5fndn Foundation (in formation)
**Document version:** v1.0 · 2026-05-16
