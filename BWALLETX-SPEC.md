# BWalletX Specification v1.0

**Universal Sovereign Object Courier for a1a5NT**

```
Document version:  v1.0
Issued:           2026-05-16
Foundation:        a1a5fndn.a1a5  (in formation)
Status:            Core engine operational · UI/UX bridging in progress
License:           A1A5 Sovereign License v1.0
                   (BWalletX implementation under its own license at release)
```

---

## 0. Evolution: v0 → v1.0

BWalletX has gone through two generations:

**BWalletX v0** (PopChain era)
- Repository: `github.com/popchainnetwork/bwalletx` (public, JavaScript, PWA)
- Description: "Dual-chain non-custodial wallet for PopChain & Solana"
- Features: PWA, AES-256-GCM, COPS cross-chain swaps
- Scope: BIN tokens on PopChain ↔ pUSDC bridge to Solana
- Status: shipped, public

**BWalletX v1.0** (a1a5NT era — this specification)
- Repository: private `a1a5` Foundation repo (reference implementation)
- Description: "Universal sovereign object courier for a1a5NT"
- Features: XMSS signing, arbitrary signed object transport, ANS-native
- Scope: any XMSS-signed object (currency, message, document, artifact,
  certificate, sub-key, attestation)
- Status: core engine operational, UI/UX bridging in progress

v0 demonstrated that single-codebase multi-chain wallets are feasible. v1.0
generalizes the transport layer: instead of supporting N chains, support N
**object types** with uniform semantics.

This specification describes v1.0. v0 remains live and supported during the
transition window; users may continue using v0 for PopChain↔Solana
interactions until a1a5NT Genesis ceremony tokenizes the popchain-archive
artifacts for transfer via v1.0.

---

## 1. Overview

BWalletX is the **Foundation flagship reference implementation** of a universal
sovereign object courier for a1a5NT. It is the **first wallet primitive** where
the transferable unit is not limited to currency tokens but extends to any
XMSS-signed object.

This specification describes the protocol. The reference implementation lives
in the private `a1a5` repository (Foundation-internal). This specification
enables independent implementations.

## 2. Why "universal sovereign object courier" not "wallet"

Traditional wallets (Bitcoin, Ethereum, Solana) transport one type of object:
the currency token. Different object types require different wallet support or
external tooling.

BWalletX transports **any XMSS-signed object** with uniform semantics:

| Object type | Example | Currently transferable in BWalletX |
|---|---|---|
| Currency token | BIN, pUSDC | ✓ |
| Message | XMSS-signed plain text | ✓ |
| Document | Foundation Declaration, License | ✓ |
| Artifact | popchain-archive epoch card | ✓ |
| Certificate | Membership attestation | ✓ |
| Sub-key | Delegated XMSS keypair | ✓ |
| Attestation | AI co-author attribution | ✓ |
| Custom | User-defined signed object | ✓ |

The transport layer is **type-agnostic**. The wallet UI provides type-specific
viewers when receiving an object.

## 3. Architecture

### 3.1 Components

```
┌─────────────────────────────────────────┐
│  BWalletX UI                            │
│  (object viewers, key management)       │
└──────────────────────┬──────────────────┘
                       │
┌──────────────────────┴──────────────────┐
│  Object Courier Engine                  │
│  - serialize/deserialize objects        │
│  - XMSS sign/verify                     │
│  - multi-object batching                │
└──────────────────────┬──────────────────┘
                       │
┌──────────────────────┴──────────────────┐
│  ANS Transport Layer                    │
│  - HTTP API (port 8443)                 │
│  - signature relay                      │
│  - inbox/outbox                         │
└──────────────────────┬──────────────────┘
                       │
                  ┌────┴────┐
                  │ a1a5NT  │
                  └─────────┘
```

### 3.2 Object structure

Every BWalletX object has uniform serialization:

```json
{
  "type": "<object_type>",
  "version": "v1.0",
  "from": "<sender_namespace>",
  "to": "<recipient_namespace>",
  "timestamp": <unix>,
  "content": <type_specific_payload>,
  "xmss_signature": "<2468 bytes hex>",
  "xmss_index": <leaf_index>
}
```

Object types are extensible. Reserved types:
- `bin_payment` — currency transfer
- `message` — plain-text message
- `document` — markdown/text document with hash
- `artifact` — referenced artifact (e.g. popchain-archive epoch)
- `certificate` — attestation of some claim
- `sub_key` — delegated XMSS keypair issuance
- `attestation` — meta-claim about another object

## 4. Cryptographic primitives

### 4.1 XMSS WOTS+ signatures

BWalletX uses XMSS WOTS+ (RFC 8391) for all signatures:
- Hash function: SHA-256
- Tree height: 10 (1024 leaves)
- Winternitz parameter: w=16
- Signature size: 2468 bytes wire format

### 4.2 State management

BWalletX maintains `state.bin` for each XMSS keypair tracking used leaves.
**Leaf reuse is forbidden** and constitutes signature forgery vulnerability.

If state.bin is lost or corrupted, the affected keypair must be considered
compromised; user must register a new namespace.

### 4.3 Recovery & sub-keys

To mitigate single-point-of-failure for XMSS state:

- **Sub-key delegation:** parent keypair signs sub-keypair authorization
- **Time-limited sub-keys:** sub-keys may include expiry timestamp
- **Recovery sub-keys:** held offline, used only to revoke compromised keypair

Recovery and sub-key management are part of BWalletX v1.0 backend; UI bridging
in progress.

## 5. Transport via ANS

### 5.1 Sending an object

BWalletX serializes the object, signs with XMSS, and POSTs to ANS server:

```
POST http://37.97.202.97:8443/v1/send
Content-Type: application/json

{
  "type": "document",
  "version": "v1.0",
  "from": "pawel.a1a5",
  "to": "nlnet.a1a5",
  "timestamp": 1778950000,
  "content": {
    "document_blake3": "<32 bytes hex>",
    "document_inline": "<base64-encoded markdown>"
  },
  "xmss_signature": "<2468 bytes hex>",
  "xmss_index": 12
}
```

ANS server verifies signature, stores in recipient's inbox, returns
`message_id`.

### 5.2 Receiving objects

BWalletX polls recipient inbox:

```
GET http://37.97.202.97:8443/v1/inbox/<my_namespace>
```

For each unread message:
1. Verify XMSS signature against sender's public key (from `/lookup/<sender>`)
2. Verify leaf index is monotonically increasing for sender
3. Deserialize content based on type
4. Render in type-specific UI viewer
5. Mark as read

### 5.3 Multi-object batching

A single transaction may carry multiple objects:

```json
{
  "type": "batch",
  "version": "v1.0",
  "from": "a1a5fndn.a1a5",
  "to": "nlnet.a1a5",
  "timestamp": 1778950000,
  "objects": [
    {"type": "certificate", "content": {...}},
    {"type": "document", "content": {...}},
    {"type": "artifact", "content": {...}}
  ],
  "xmss_signature": "<2468 bytes hex over batch>",
  "xmss_index": 13
}
```

One XMSS signature covers the entire batch. One leaf consumed.

## 6. Object viewers in UI

BWalletX UI provides type-specific viewers:

- **bin_payment** → balance display, transaction history
- **message** → plain-text reader, reply button
- **document** → markdown renderer with hash verification badge
- **artifact** → embedded SVG card preview with metadata
- **certificate** → verified-claim display with issuer chain
- **sub_key** → sub-key management dashboard
- **attestation** → meta-claim viewer with linkage to attested object

When NLnet downloads BWalletX and receives a token from Foundation, the token
auto-opens as a readable document — not an opaque hex string.

## 7. Status of implementation

As of 2026-05-16:

### Operational

- Core object courier engine: serialization/deserialization for all 7 reserved
  types
- XMSS signing + verification: end-to-end working
- Sending/receiving via ANS: tested in testnet
- Multi-object batching: supported
- BIN payment integration: operational
- Validator-side object validation: beta-stable
- State transition handling: working

### UI in progress

- Basic flow (connect, sign, send, receive, view): operational
- Object viewer polish: in progress
- Object history search: not yet
- ANS namespace search: not yet
- Advanced delegated key management UI: backend ready, UI in progress
- Recovery/social flows: backend logic ready, UI in progress

### Reference deployment timeline

- **v0.x (internal beta):** present
- **v1.0 (Foundation members + NLnet):** Q3 2026
- **v1.x (public release):** post-Genesis ceremony

## 8. Independent implementations

This specification is **public** under A1A5 Sovereign License v1.0.

Third parties may implement BWalletX-compatible clients in any language. The
on-the-wire object format and XMSS scheme are deterministic; implementations
that follow this specification will interoperate.

Patent claims on the protocol are **prohibited** under License v1.0 Article 5.

Reference implementation source code remains under Foundation custody in
private `a1a5` repository for production-grade assurance (Ledger/Signal model).

## 9. Security considerations

### 9.1 XMSS state preservation

Loss of state.bin = compromised keypair. Users must back up state.bin
regularly (encrypted) and never run multiple BWalletX instances on the same
keypair simultaneously.

### 9.2 Server trust

BWalletX trusts the ANS server only for **transport**. All cryptographic
verification (XMSS signatures) happens client-side. A malicious ANS server
cannot forge messages — only refuse to relay them or reorder them
(detectable via leaf index monotonicity).

### 9.3 Replay protection

ANS server enforces monotonic leaf indices per namespace. BWalletX additionally
verifies leaf index sequencing on inbox polling.

### 9.4 No phone home

BWalletX does not transmit metadata to Foundation. All connections are direct
client → ANS server. No analytics, no telemetry, no third-party tracking.

## 10. Foundation flagship status

BWalletX is the **Foundation flagship reference implementation**. It serves
multiple purposes:

1. **Eat own dogfood:** Foundation uses BWalletX for all internal communication
2. **Reference for spec:** Implementation-level proof that the spec works
3. **NLnet onboarding:** When NLnet (or any funder) receives tokens, BWalletX
   provides the recommended reader
4. **Marketplace client:** When tokenized artifacts go on sale post-Genesis,
   BWalletX is the recommended buyer/seller client
5. **Industry pilot tool:** For Plastechniek + Metaalunie integrations

---

**License:** A1A5 Sovereign License v1.0
**Foundation:** a1a5fndn.a1a5 (in formation)
**Custodian:** pawel.a1a5
**Implementation:** private `a1a5` repository
**Specification:** public `a1a3` repository (this document)
