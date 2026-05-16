# ANS Protocol Specification v1.0

**Address Namespace System for a1a5NT**

```
Document version:  v1.0
Issued:           2026-05-16
Foundation:        a1a5fndn.a1a5  (in formation)
Status:            Live infrastructure as of 2026-05-16
License:           A1A5 Sovereign License v1.0
```

---

## 1. Overview

ANS (Address Namespace System) is the sovereign naming layer of a1a5NT. It
provides:

- **Hierarchical namespaces** independent of DNS/ICANN
- **Post-quantum identity binding** via XMSS WOTS+ signatures
- **Public message routing** with cryptographic provenance
- **Subdomain support** for hierarchical Foundation namespace trees

ANS replaces email + DNS + traditional identity layers with a single sovereign
primitive.

## 2. Namespace format

### 2.1 Syntax

```
<label>.<parent>.<tld>
```

Where:
- `<tld>` is a top-level identifier (currently only `a1a5`)
- `<parent>` is the parent namespace (or omitted for top-level)
- `<label>` is the leaf identifier

Examples:
- `pawel.a1a5` (top-level)
- `a1a5fndn.a1a5` (Foundation root)
- `archive.a1a5fndn.a1a5` (Foundation subdomain)
- `epoch0084.archive.a1a5fndn.a1a5` (deep subdomain)

### 2.2 Constraints

- Labels: lowercase ASCII alphanumeric + hyphen, 1-63 characters
- Total length: max 253 characters
- Reserved tld: `a1a5` (Foundation-managed)
- Reserved subtree: `a1a5fndn.*` (Foundation-managed, see License v1.0)

### 2.3 Resolution

Namespaces resolve to:

```json
{
  "name": "pawel.a1a5",
  "xmss_root": "<32 bytes hex>",
  "pub_seed":  "<32 bytes hex>",
  "registered_unix": 1778906400,
  "metadata": {
    "issuer": "self",
    "parent_signature": null
  }
}
```

For Foundation-issued subdomains:

```json
{
  "name": "epoch0084.archive.a1a5fndn.a1a5",
  "parent_namespace": "archive.a1a5fndn.a1a5",
  "issuer_name": "a1a5fndn",
  "owner_name": "a1a5fndn",
  "metadata": { ... }
}
```

## 3. XMSS signatures

### 3.1 Parameters

- Scheme: XMSS WOTS+ (RFC 8391)
- Hash: SHA-256
- Tree height: 10 (1024 leaves per keypair)
- Winternitz parameter: w=16
- Signature size: 2468 bytes wire format

### 3.2 Per-namespace keypair

Each namespace is bound to one XMSS keypair. The public root + public seed are
published in the namespace metadata.

State (used leaves) is tracked by the namespace owner in `state.bin`. Each
signature consumes one leaf; reuse is forbidden and results in signature
forgery vulnerability.

### 3.3 Signature verification

Any party can verify a signature against the published public key without
network access:

```rust
xmss::verify(pubkey, signature, message) → bool
```

Returns true if signature is valid AND the leaf index has not been previously
seen for this public key (replay protection enforced at server level).

## 4. HTTP API

### 4.1 Server location

```
http://37.97.202.97:8443
```

(Foundation-operated reference instance. Anyone may operate ANS-compatible
servers; federation protocol TBD.)

### 4.2 Endpoints

#### `GET /list`

Returns all registered namespaces.

```json
[
  {"name": "pawel.a1a5", "registered_unix": 1778906400, ...},
  {"name": "boes.a1a5", "registered_unix": 1778907021, ...}
]
```

#### `GET /lookup/<name>`

Returns namespace public key + metadata.

```bash
curl http://37.97.202.97:8443/lookup/pawel
```

Response:
```json
{
  "name": "pawel.a1a5",
  "xmss_root": "f7d4d034adba761b09d1c277a3d6e5178a2f79127f5b057a9433e90867d5939d",
  "pub_seed":  "a442f5583b1df6ae6289bc56e2d8ae2eecc20fd0af53426fcb1fbbfe7d027d58",
  "registered_unix": 1778906400
}
```

#### `GET /count`

Returns total registered namespaces.

```json
{"count": 92}
```

#### `GET /v1/inbox/<name>`

Returns all public XMSS-signed messages addressed to `<name>.a1a5`.

```bash
curl http://37.97.202.97:8443/v1/inbox/senobg
```

Response:
```json
{
  "name": "senobg.a1a5",
  "messages": [
    {
      "id": "msg_001",
      "from": "pawel.a1a5",
      "body": "Hello from sovereign channel",
      "timestamp": 1778950000,
      "xmss_signature": "<2468 bytes hex>",
      "xmss_index": 7,
      "verified": true
    }
  ]
}
```

#### `POST /v1/register`

Register a new namespace. Requires XMSS-signed payload.

Request body:
```json
{
  "name": "newuser",
  "parent": "a1a5",
  "xmss_root": "<32 bytes hex>",
  "pub_seed":  "<32 bytes hex>",
  "registration_signature": "<2468 bytes hex>",
  "signature_index": 0
}
```

Server validates:
1. Signature verifies against published xmss_root
2. Name + parent are not already registered
3. For subdomain registration: parent owner must co-sign

#### `POST /v1/send`

Send signed message.

Request body:
```json
{
  "from": "pawel.a1a5",
  "to": "senobg.a1a5",
  "body": "message content",
  "timestamp": 1778950000,
  "xmss_signature": "<2468 bytes hex>",
  "xmss_index": 7
}
```

Server validates:
1. Signature verifies against from's xmss_root
2. xmss_index is greater than previously seen index (replay protection)
3. Recipient namespace is registered

## 5. Foundation subdomain extension

### 5.1 Schema

Server maintains `ans_metadata` table for Foundation-issued subdomains:

```sql
CREATE TABLE ans_metadata (
  namespace_name TEXT PRIMARY KEY,
  parent_namespace TEXT NOT NULL,
  issuer_name TEXT NOT NULL,
  owner_name TEXT NOT NULL,
  metadata_json TEXT NOT NULL,
  registered_unix INTEGER NOT NULL
);
```

### 5.2 Foundation bulk registration

Foundation (Foundation namespace `a1a5fndn.a1a5`) may bulk-register subdomain
trees under its control, e.g. all 85 popchain-archive epoch namespaces:

```
epoch0000.archive.a1a5fndn.a1a5
epoch0001.archive.a1a5fndn.a1a5
...
epoch0084.archive.a1a5fndn.a1a5
```

Each registration includes metadata pointing to underlying artifact:

```json
{
  "kind": "archive_epoch",
  "category": "STANDARD",
  "epoch_id": "epoch_0084",
  "block_count": 457,
  "height_range": [84000, 84455],
  "manifest_blake3_prefix": "a7d5593c5c00a270",
  "catalog_blake3": "aa2c0dd44071526901ae6fbb1ed471646dcdc3f5129b4bcf9b2124a553adf2a3"
}
```

### 5.3 Lookup with subdomain support

`GET /lookup/<full.name.path>` resolves through ans_metadata first, falling
back to base ans table.

## 6. Replay protection

Server enforces monotonic leaf index for each namespace:

- First message: index = 0
- Each subsequent message: index = N+1
- Server rejects: index ≤ last seen index for namespace

This prevents replay attacks across XMSS-signed messages. If a namespace owner
loses state.bin, they must register a new namespace (XMSS reuse with stale state
would produce forgeable signatures).

## 7. No email bridge

ANS is intentionally not bridged to SMTP. Communication occurs via:

1. Direct HTTP API access
2. CLI tools (`a1a5nt-ans-cli`)
3. BWalletX (when v1.0 ships)
4. Third-party clients implementing this specification

Foundation will not operate SMTP infrastructure. Parties wishing to communicate
with Foundation must use sovereign channel.

## 8. Federation

Federation protocol between independent ANS servers is reserved for v2.

For v1, a single Foundation-operated reference server at `37.97.202.97:8443`
provides canonical resolution. Mirror servers operating in read-only mode are
permitted under License v1.0 (citation/research use).

## 9. Implementation reference

**Public spec:** this document (`a1a3` repo)
**Reference implementation:** `a1a5` private repo (Foundation)
**CLI tool:** `a1a5nt-ans-cli` (planned public release post-Stichting)

Third-party implementations welcome under License v1.0 terms.

## 10. Live deployment

As of 2026-05-16:

- **92 namespaces** registered total
  - 7 personal: pawel, boes, metaalunie, plastechniek, piekut, senobg, a1a5fndn
  - 85 archive: epoch0000..epoch0084.archive.a1a5fndn.a1a5
- **1 public XMSS-signed message** transmitted (pawel → senobg)
- **Reference server** at 37.97.202.97:8443
- **Uptime since launch:** continuous

---

**License:** A1A5 Sovereign License v1.0
**Custodian:** pawel.a1a5
**Foundation:** a1a5fndn.a1a5 (in formation)
