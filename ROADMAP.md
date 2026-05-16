# ROADMAP

**Document version:** v1.0
**Issued:** 2026-05-16
**Foundation:** a1a5fndn.a1a5  (in formation)

---

## Origins

```
2025-Q1    PPlastechniek (metalworking ZZP, Almelo NL)
           → Founder begins exploring blockchain for industrial process verification
           → Goal: enable cryptographic proof of completed work for shop-floor machines

2025-Q2    Proof-of-Process concept crystallizes
           → Domain registered: proof-of-process.org
           → Email infrastructure stood up: info@proof-of-process.org
           → Initial whitepaper sketched

2025-Q3    PopChain implementation begins
           → Rust Layer-1 from scratch, no SDK, no fork
           → POP-1 (deterministic binary analysis) + POP-2 (MBW calculation)
           → WOTS+ post-quantum signatures from day one
           → Repository: github.com/popchainnetwork/popchain (Apache-2.0)
           → Website: popchain.tech (live dashboard)
           → BWalletX v0 (dual-chain PopChain↔Solana):
             github.com/popchainnetwork/bwalletx

2026-Q1    a1a5 — brand and technology stack maturation
           → Recognition that PopChain's foundation was solid but missing:
             • Sovereign identity layer (independent of DNS/ICANN)
             • Post-quantum ANS (Address Namespace System)
             • Universal sovereign object courier (BWalletX v1.0)
           → Stack rename: a1a5NT (Network Technology, full stack)
           → Public spec naming: a1a3 (open-source specifications)
           → Sovereign founder identity: pawel.a1a5

2026-05    Formalization
           → PopChain testnet sealed at block 84,455 (86,073 blocks total)
           → popchain-archive cryptographically preserved (85 epoch artifacts)
           → a1a5NT begins as gen 2
           → GitHub account established: github.com/a1a5NT
           → Foundation Declaration v1.0 issued
           → A1A5 Sovereign License v1.0 issued
           → Stichting A1A5 registration scheduled at NL notary
```

## Trajectory

```
2026-05-16 ────  PopChain sealed at block 84455 (final block)
                 86,073 blocks total, archived in 85 epoch artifacts
2026-05-16 ────  a1a5NT begins · post-quantum sovereign L1
                 Foundation Declaration v1.0 anchored
                 License v1.0 anchored
                 85 cards minted in ANS, Foundation custody
   ↓
2026-05 W3-W4   Stichting A1A5 registration (notary, Almelo NL)
                Foundation legal entity formed
                Custody chain transfers from pawel.a1a5 to Stichting
   ↓
2026-Q3         BWalletX v1.0 release
                IMASS algorithm specification finalized
                POP-3 Token spec finalized
                a1a5NT Genesis ceremony
                85 cards tokenized with IMASS values
   ↓
2026-Q4         Marketplace opens
                Tokenized epoch artifacts on sale
                Token transfer protocol active
   ↓
2027-05-16      License v1.0 12-month binding window expires
                Renewal or successor license issued
```

---

## Generation 1: PopChain (2025-02 → 2026-05-16)

**Status:** ARCHIVED. 86,073 blocks sealed. 6/6 invariants PASS.

**What it was:**

PopChain was an industrial Layer-1 blockchain designed around Proof-of-Process
(PoP) — a system where machine-generated data (welding, CNC, laser cutting) is
transformed into cryptographically verifiable proofs via deterministic
statistical anchors:

- A1: Shannon entropy
- A2: Mean Absolute Deviation
- A3: Character uniqueness
- A4: Length fitness
- A5: NIST probabilistic signature

The five anchors compute MBW (Monetary Base Weight) — a single u64 fingerprint
per data submission. Cross-architecture bit-identical determinism was verified
empirically on x86_64 and aarch64 across 4 machines (EXP-015, 2026-05-12).

**What it produced:**

- 86,073 blocks across 42 days of operation
- 85 epoch artifacts (sliced via popchain-harvest 1.0.2)
- Catalog blake3: `aa2c0dd44071526901ae6fbb1ed471646dcdc3f5129b4bcf9b2124a553adf2a3`
- PCIP License v1.0 (first self-anchored license on PoP chain, block 82112)
- POCC_ANCHOR_V1 = 417696 (cross-validator consensus checkpoint)

**Why it was sealed:**

PopChain achieved its primary demonstration goals: deterministic L1 in pure
Rust, cross-architecture consensus, working Proof-of-Process pipeline. Continued
operation would have consumed resources better directed at the post-quantum
sovereign layer (a1a5NT). The chain was sealed cleanly with all 86,073 blocks
preserved as auditable archive.

**What remains:**

popchain-archive: 85 cryptographically verifiable epoch artifacts, held under
Foundation custody, to be tokenized at a1a5NT Genesis ceremony with IMASS
algorithm computation.

See [POPCHAIN-ARCHIVE-PROVENANCE.md](POPCHAIN-ARCHIVE-PROVENANCE.md) for
verification recipe.

---

## Generation 2: a1a5NT (2026-05-16 → present)

**Status:** ACTIVE. Live infrastructure. Foundation in formation.

**What it is:**

a1a5NT extends PopChain's deterministic philosophy with three post-quantum
primitives:

### 1. XMSS WOTS+ signatures

NIST-standardized post-quantum signature scheme (RFC 8391). 1024-leaf XMSS
tree, 2468-byte wire format, authentication path verification. Each ANS
namespace is bound to an XMSS root, with each transaction consuming one leaf.
Public seed and tree height published, signatures verifiable offline by any
party.

### 2. ANS (Address Namespace System)

Sovereign naming layer operating independently of DNS/ICANN. Namespaces are
hierarchical (`name.parent.tld`), registered via XMSS-signed transaction,
resolved via HTTP API. SQLite-backed, deterministic, single-write append-only.
Subdomain support enables Foundation-issued namespace trees.

Live as of 2026-05-16: 7 personal namespaces, 85 archive epoch namespaces.

### 3. Universal sovereign object courier (BWalletX)

First wallet primitive where the transferable unit is not limited to currency
tokens but extends to any XMSS-signed object: messages, documents, artifacts,
certificates, sub-keys, attestations. Core object courier engine operational;
UI/UX bridging in progress.

See [BWALLETX-SPEC.md](BWALLETX-SPEC.md) for protocol details.

---

## Foundation status

**Now (Foundation in formation):**

- Self-proclaimed via Foundation Declaration v1.0 (2026-05-16)
- Cryptographic entity under namespace `a1a5fndn.a1a5`
- Sole founder and custodian: pawel.a1a5 (Paweł Piekut)
- Interim legal vehicle: ZZP Plastechniek (KvK NL, Almelo)
- Sovereign-only contact: no email, no SMTP

**Scheduled:**

- **Stichting A1A5 registration** at notary (Almelo NL), week of 2026-05-19
- **Bestuur (board) composition** to be established at notary
- **Transfer of cryptographic custody** from pawel.a1a5 to Stichting upon
  successful registration, anchored via XMSS-signed ANS message

**Post-Stichting:**

- KvK number anchored in ANS as Foundation legitimacy proof
- Updated Foundation Declaration with Stichting reference
- Banking and accounting infrastructure
- NLnet amendment notification (if grant approved)

---

## NLnet Foundation context

**Application:** Commons Fund 2026-04-3f6
**Original subject:** PopChain Network — Proof-of-Process Industrial Blockchain
**Status:** Submitted 2026-04-03 · review in progress
**Requested amount:** €45,000

**Evolution since application:**

The application described PopChain as the working system. Since submission:

- PopChain completed its operational phase (86,073 blocks produced)
- Project evolved to a1a5NT (post-quantum L1 + sovereign naming) as natural
  successor
- Foundation Declaration issued formalizing the entity
- 85 epoch artifacts of popchain-archive held as proof-of-work and prepared
  for tokenization

**Amendment plan:**

Once Stichting A1A5 is registered (post-notary), formal amendment notice will
be issued to NLnet:

1. Updated project scope: a1a5NT with PopChain archived as gen 1 reference
2. Same deterministic philosophy, expanded primitives (post-quantum + sovereign
   naming)
3. Stichting A1A5 as legal entity (with KvK number)
4. Continued open-source approach (public spec in `a1a3` repo)
5. Sovereign contact: `pawel.a1a5` (with BWalletX as recommended reader)

**NLnet gift:**

The Foundation has reserved `nlnet.a1a5` namespace as a courtesy gift for the
NLnet team. Upon grant acceptance and Stichting registration, one tokenized
epoch artifact from popchain-archive will be transferred to the NLnet team
namespace as a record of historical collaboration.

See [NLnet-FUNDER-ONBOARDING.md](NLnet-FUNDER-ONBOARDING.md) for technical
details on accessing the sovereign channel.

---

## Key deliverables · 2026 Q3

### IMASS algorithm specification

**IMASS (Information Mass)** — single u64 fingerprint per epoch artifact
computed deterministically from manifest data. Used as cryptographic property
of each tokenized card at Genesis.

Components (designed, not yet finalized):

- block_count × canonical_ratio (canonical density)
- mbw_delta_in_epoch (monetary base weight accretion)
- producer_diversity (Shannon entropy of producer_counts)
- fork_density (forked_heights / block_count)
- app_event_density (total_app_events / block_count)

Formula will be published as `IMASS-SPEC-V1.md` before Genesis ceremony. Every
IMASS value verifiable by re-running the algorithm against the manifest.

### POP-3 Token specification

a1a5NT-native token primitive for tokenization of Foundation artifacts.

- Token = 1 epoch artifact = 1 indivisible NFT-like asset
- Issuance: Foundation XMSS root signature
- Transfer: current owner XMSS sig + Foundation co-signature
- Storage: SQLite table `tokens` with chain-of-custody
- Endpoints: `GET /token/<id>`, `POST /token/transfer`

### a1a5NT Genesis ceremony

Single event minting:

- 85 tokenized epoch artifacts with IMASS values
- Foundation Declaration anchor
- License v1.0 anchor
- All under Foundation XMSS root signature

Output: single blake3 hash = a1a5NT genesis fingerprint, linking back to
popchain catalog blake3 as parent. After Genesis, tokens become transferable;
marketplace opens for tokenized artifact sales.

### BWalletX v1.0

Universal sovereign object courier — first wallet primitive supporting transfer
of arbitrary XMSS-signed objects.

**Status (2026-05-16):**

- Core object courier engine operational
- XMSS signing + verification end-to-end
- Multi-type object serialization (BIN, Message, Artifact, Card, SubKey,
  Attestation)
- Sending/receiving in testnet, including multi-object batch transactions
- BIN payment integration + bridge hooks
- Validator-side object validation, state transition handling
- UI: basic flow operational, polish in progress
- Missing: advanced delegated key management UI, recovery/social flows

See [BWALLETX-SPEC.md](BWALLETX-SPEC.md) for technical specification.

---

## Risk and dependencies

**Known risks:**

1. **Bus factor:** Sole founder. Mitigation: open spec in `a1a3`, anchored
   cryptographic proofs, planned Stichting bestuur expansion post-registration.

2. **NLnet decision:** Grant pending. Mitigation: development continues
   regardless; commercial revenue path via tokenized artifact sales post-Genesis.

3. **Stichting registration timing:** Dependent on notary scheduling.
   Mitigation: Foundation Declaration provides cryptographic predecessor; all
   pre-Stichting activity attributable to pawel.a1a5 personally.

4. **BWalletX UI polish:** v1.0 release target Q3 2026. Mitigation: Public HTTP
   endpoints (port 8443/8444) provide full functionality for technical users
   pre-BWalletX.

5. **Patent landscape:** Aggressive defensive clause in License v1.0 + prior
   art anchored on popchain block 82112. Mitigation: All Foundation primitives
   timestamped via blockchain anchors as defensive prior art.

---

## License

This roadmap is part of the public `a1a3` repository, issued under
[A1A5 Sovereign License v1.0](LICENSE-A1A5-SOVEREIGN-V1.md).

**Issued by a1a5fndn Foundation (in formation) · Custodian: pawel.a1a5**
**Document anchor:** scheduled · ANS message ID will be appended post-anchor
