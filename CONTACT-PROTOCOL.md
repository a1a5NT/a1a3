# Contact Protocol · How to communicate with a1a5fndn Foundation

```
Document version:  v1.0
Issued:           2026-05-16
Foundation:        a1a5fndn.a1a5  (in formation)
Sovereign address: pawel.a1a5
Email:             — none, by design
License:           A1A5 Sovereign License v1.0
```

---

## TL;DR

```bash
# Option 1: Email (legacy infrastructure, supported during transition)
mail info@proof-of-process.org

# Option 2: Read public sovereign inbox
curl http://37.97.202.97:8443/v1/inbox/pawel

# Option 3: Send sovereign message (unsigned advisory)
curl -X POST http://37.97.202.97:8443/v1/send \
  -H 'Content-Type: application/json' \
  -d '{
    "from": "your.name.or.org",
    "to": "pawel",
    "body": "Your message here",
    "timestamp": '$(date +%s)'
  }'
```

That's it. Three channels: legacy email, public sovereign inbox, signed
sovereign message. Pick whichever fits your infrastructure today.

---

## 1. Email vs sovereign channel

The Foundation maintains **both** during the transition period:

**Legacy email** (`info@proof-of-process.org`) — supported for:
- Parties without ANS tooling
- Institutional correspondence (notary, banking, legal)
- NLnet and other funder communication
- Anyone for whom curl-based messaging is impractical today

**Sovereign channel** (`pawel.a1a5`) — recommended for:
- Technical collaborators
- Foundation members with ANS access
- Anyone testing the post-quantum stack
- Future-state correspondence as adoption grows

**Strategic direction:** The Foundation's long-term position is that email is
end-of-life infrastructure incompatible with post-quantum sovereign identity.
The project exists in part to demonstrate the alternative. However, the
alternative requires tooling (BWalletX) and adoption that doesn't yet exist
at scale.

Until BWalletX v1.0 ships (Q3 2026) and ANS federation matures, legacy email
remains supported. This is **pragmatism, not retreat** — the world hasn't
caught up yet, and the Foundation needs to be reachable.

## 2. The sovereign address: pawel.a1a5

The Foundation founder maintains a single sovereign address:

```
pawel.a1a5
```

This is a namespace in ANS (Address Namespace System), bound to an XMSS
post-quantum keypair.

**Public key (XMSS root):**
```
c9a630a533a417230bcf8f32570824fe924381eb9ee028362b18fb60aad89bee
```

**Public seed:**
```
1a1340137ee6f5007ad572ff65d9e49641402ada856720083c39abbff10ef233
```

These are real, working post-quantum public keys verifiable at:
```bash
curl http://37.97.202.97:8443/lookup/pawel
```

## 3. Four ways to communicate

### Option 0: Email (legacy, supported)

```
info@proof-of-process.org
```

Use this if:
- You don't have ANS tooling
- You need standard SMTP delivery (NLnet, notary, banks)
- You're sending institutional correspondence
- You want familiar infrastructure

Limitations:
- No post-quantum verification
- No cryptographic non-repudiation
- Standard SMTP risks (spam filters, delivery delays)
- Will be deprecated once sovereign channel matures (likely 2027+)

Response priority: high. Founder reads regularly.

### Option A: Read public sovereign inbox (zero setup)

```bash
curl http://37.97.202.97:8443/v1/inbox/pawel
```

You see what others have sent. Public, no auth.

### Option B: Send unsigned advisory message (1 minute setup)

POST a JSON message. No XMSS signature required. Server accepts as advisory
(visible but not cryptographically verified).

```bash
curl -X POST http://37.97.202.97:8443/v1/send \
  -H 'Content-Type: application/json' \
  -d '{
    "from": "your.identifier",
    "to": "pawel",
    "body": "Your message text here",
    "timestamp": '$(date +%s)'
  }'
```

Use this for:
- Initial outreach
- Public notes
- Brief technical questions
- Press / media inquiries

Limitations:
- No cryptographic non-repudiation
- No verification you are who you say you are
- Server may rate-limit unsigned messages

### Option C: Register your own namespace + sign XMSS (recommended for ongoing)

For serious correspondence (legal, funding, technical depth):

1. **Register your namespace** via XMSS-signed POST to `/v1/register`
2. **Generate your XMSS keypair** (1024 leaves, RFC 8391 compliant)
3. **Sign each message** with XMSS, consuming one leaf per message
4. **Send signed messages** via `/v1/send`

Tools available:
- `a1a5nt-ans-cli` (CLI tool, public release post-Stichting)
- BWalletX (when v1.0 ships, Q3 2026)
- Independent implementations (encouraged, per License v1.0)

Until BWalletX v1.0, technical users may use the CLI or implement their own
client against the [ANS Protocol Specification](ANS-PROTOCOL-SPEC.md).

## 4. What gets a response

Founder reads pawel.a1a5 inbox regularly. Response priority:

**High priority (24-72h response):**
- Funding decisions, grants
- Legal matters (Stichting, contracts)
- Notary / official correspondence
- Critical security issues in the project
- NLnet team communications

**Medium priority (~1 week response):**
- Technical questions from collaborators
- Project integration inquiries
- Industry partnerships
- Press / media (legitimate outlets)

**Low / no priority:**
- Marketing pitches
- Unsolicited "we can help your project" emails
- AI-generated mass outreach
- Anything with a deadline implying time pressure

The Foundation is not for sale and does not respond to acquisition offers
during the License v1.0 binding window.

## 5. Special channels

### For NLnet

The Foundation has reserved `nlnet.a1a5` namespace as a courtesy for the NLnet
Foundation review team. Upon NLnet's request, this namespace will be
transferred to NLnet custody with a generated keypair.

See [NLnet-FUNDER-ONBOARDING.md](NLnet-FUNDER-ONBOARDING.md) for full
onboarding.

### For independent researchers

Open-source contributors and security researchers are welcome to reach out
via Option B (unsigned advisory) for initial contact.

For security disclosures, please use a clearly-labeled "SECURITY" prefix in
the message body. The Foundation will respond with secure channel
coordination (option C with one-time-use sub-key, or in-person meeting in
Almelo NL if appropriate).

### For Stichting/notary correspondence (post-registration)

Once Stichting A1A5 is registered (post-2026-05-19), the KvK-registered
business address will be the legal correspondence channel for matters
requiring registered mail or physical document delivery.

Cryptographic correspondence continues via pawel.a1a5 / future Foundation
namespace.

## 6. What we will not do

- **No phone** number publicly listed (founder's mobile is on the NLnet
  application form for emergencies only)
- **No social media DMs** as primary channel (Twitter/X used for broadcasts,
  not bilateral conversations)
- **No Discord / Telegram / Slack** for Foundation business
- **No third-party identity providers** (OAuth, "sign in with Google")
- **No proxy email services** (Mailgun, SendGrid as primary inbox — only
  for outbound notifications if needed)

Email (`info@proof-of-process.org`) is supported as transitional infrastructure
during the period before sovereign-channel adoption matures. It is not the
Foundation's preferred channel, but it is honest infrastructure that exists
today and meets institutional needs.

## 7. Anti-spam / anti-abuse

The ANS server enforces:
- Rate limiting on unsigned messages (per source IP)
- Mandatory XMSS verification for signed messages (replay protection)
- Public visibility of all inbox content (transparency by default)
- Spam scoring on unsigned messages (high-volume mass outreach filtered)

If you send legitimate correspondence and it doesn't appear in the public
inbox, the spam filter likely flagged it. Resubmit with clearer subject /
identifiable from-address.

## 8. Verification

You can verify the founder has received and read a public message in two ways:

1. **Reply broadcast:** Founder posts XMSS-signed reply to recipient's inbox
   (if recipient has a registered namespace)
2. **Status update:** Founder may post a digest of recent correspondence as
   XMSS-signed public message in own outbox

Both verifiable cryptographically.

## 9. Quick reference card

```
LEGACY EMAIL (supported during transition)
   info@proof-of-process.org

SOVEREIGN ADDRESS
   pawel.a1a5

PUBLIC KEY (XMSS root)
   c9a630a533a417230bcf8f32570824fe924381eb9ee028362b18fb60aad89bee

PUBLIC SEED
   1a1340137ee6f5007ad572ff65d9e49641402ada856720083c39abbff10ef233

READ INBOX
   GET http://37.97.202.97:8443/v1/inbox/pawel

VERIFY KEY
   GET http://37.97.202.97:8443/lookup/pawel

SEND ADVISORY (no signature needed)
   POST http://37.97.202.97:8443/v1/send

FOUNDATION NAMESPACE
   a1a5fndn.a1a5

PHYSICAL LOCATION (NL Stichting post-registration)
   Almelo, Overijssel, Netherlands
```

---

**License:** A1A5 Sovereign License v1.0
**Foundation:** a1a5fndn.a1a5 (in formation)
**Custodian:** pawel.a1a5
**Email and sovereign channel both available during transition. Sovereign preferred.**
