# NLnet Funder Onboarding

**For:** Michiel Leenaars and the NLnet Foundation review team
**Re:** Application 2026-04-3f6 (Commons Fund · PopChain Network)
**From:** Paweł Piekut, founder (pawel.a1a5)
**Issued:** 2026-05-16

---

## Why this document

This document onboards the NLnet team to how communication and project
verification works in the current state of the project.

Since the application was submitted on 2026-04-03, the project has evolved
significantly. This document explains the evolution, points to verifiable
infrastructure, and explains how to reach the founder through the sovereign
contact channel that the project itself demonstrates.

## Project evolution since application

### What the application described

The 2026-04-3f6 application described **PopChain** — an industrial Layer-1
blockchain using Proof-of-Process (PoP) to verify machine-generated data
(welding, CNC, laser cutting) and produce work-backed tokens.

Key features at submission:
- Deterministic Rust implementation
- 5-anchor MBW (Monetary Base Weight) computation
- Cross-architecture bit-identical verification
- Native CLI tooling, validator coordination

### What has happened since

Between application submission (2026-04-03) and today (2026-05-16):

**Phase 1 (April):** PopChain continued operations. Validators ran across 4
machines. Cross-architecture determinism verified empirically (EXP-015).
PCIP License v1.0 anchored on chain at block 82112 — first self-anchored
license in post-quantum infrastructure.

**Phase 2 (Early May):** Recognition that PopChain's deterministic foundation
was solid, but two missing primitives prevented broader adoption:
- **Post-quantum identity** (XMSS signatures, NIST-standardized)
- **Sovereign naming** (independent of DNS/ICANN, immune to traditional
  censorship vectors)

**Phase 3 (2026-05-16, today):** PopChain operations concluded cleanly at
block 84,455. The 86,073-block chain was archived as **popchain-archive** —
85 cryptographically verifiable epoch artifacts. Generation 2 of the project
— **a1a5NT** — begins with the missing primitives added.

### Why this is a maturity progression, not abandonment

PopChain proved the foundational claims:
- Deterministic L1 is buildable in Rust by one person
- Cross-architecture consensus works empirically
- Industrial Proof-of-Process is computable
- 86,073 blocks of operational data archived

a1a5NT preserves all foundational work and adds the next layer:
- XMSS WOTS+ signatures (post-quantum, NIST-standardized)
- ANS (Address Namespace System) — sovereign identity, no DNS dependency
- Universal sovereign object courier (BWalletX) — wallets transfer any
  signed object, not just currency

The same deterministic philosophy. Same open-source approach. Same single
founder. Expanded primitives.

## What you can verify right now

All infrastructure is live and publicly accessible. No authentication needed.
Verification can be done with curl from any machine.

### Verify ANS server

```bash
curl http://37.97.202.97:8443/list
curl http://37.97.202.97:8443/count
```

Should return 92 registered namespaces (7 personal + 85 archive epochs).

### Verify Foundation namespace

```bash
curl http://37.97.202.97:8443/lookup/a1a5fndn
```

Returns Foundation public key. This is the Foundation cryptographic identity.

### Verify founder namespace

```bash
curl http://37.97.202.97:8443/lookup/pawel
```

Returns:
```json
{
  "name": "pawel.a1a5",
  "xmss_root": "f7d4d034adba761b09d1c277a3d6e5178a2f79127f5b057a9433e90867d5939d",
  "pub_seed":  "a442f5583b1df6ae6289bc56e2d8ae2eecc20fd0af53426fcb1fbbfe7d027d58"
}
```

These are real, working post-quantum public keys.

### View 85 archive artifacts

```bash
curl http://37.97.202.97:8444/v1/archive/epochs
```

Or open in browser: http://37.97.202.97:8444/

Each epoch artifact is a unique SVG card with cryptographic chain-of-custody
to popchain catalog blake3:
`aa2c0dd44071526901ae6fbb1ed471646dcdc3f5129b4bcf9b2124a553adf2a3`

### Verify a public message

The first XMSS-signed sovereign message in a1a5NT is publicly readable:

```bash
curl http://37.97.202.97:8443/v1/inbox/senobg
```

This message was sent from `pawel.a1a5` to `senobg.a1a5` and is verifiable
against the published public keys.

## How to communicate with the founder

Two channels are available. **Email remains supported** for institutional
correspondence; sovereign channel is the project's strategic direction.

### Channel 1: Email (recommended for NLnet workflow)

```
info@proof-of-process.org
```

This is the email address on the original 2026-04-3f6 application. It is
**active and monitored** by the founder. Use this for:
- Grant decision communication
- Formal funder correspondence
- Documents requiring SMTP delivery / institutional records
- Any case where your workflow expects standard email

### Channel 2: Sovereign address (demonstrating the project)

```
pawel.a1a5
```

This is the cryptographic identity of the founder in a1a5NT. Use this if
you want to test the project's actual infrastructure (recommended for at
least one exchange — it's how the project demonstrates its premise).

#### Option A: Read public inbox

```bash
curl http://37.97.202.97:8443/v1/inbox/pawel
```

Public messages addressed to pawel.a1a5 are visible without authentication.
Founder reads inbox regularly.

#### Option B: Send unsigned advisory message

If you do not have ANS tooling yet, you may POST a JSON message without XMSS
signature. The server accepts it as "advisory" — visible but not verified.

```bash
curl -X POST http://37.97.202.97:8443/v1/send \
  -H 'Content-Type: application/json' \
  -d '{
    "from": "nlnet.advisory",
    "to": "pawel",
    "body": "Re: 2026-04-3f6 — message text here",
    "timestamp": <current unix>
  }'
```

The founder will receive this as a notice in the public inbox.

#### Option C: Register nlnet.a1a5 (recommended for ongoing collaboration)

The Foundation has **reserved `nlnet.a1a5` namespace as a gift** for the NLnet
team. Upon NLnet's request, the namespace will be transferred to NLnet
custody with an XMSS keypair generated for the NLnet team.

After namespace handover, NLnet can send XMSS-signed messages to pawel.a1a5,
fully verified, with non-repudiation, replay-protected.

This is the **recommended channel** for any sensitive communication (grant
decisions, technical questions, project updates) once setup completes.

#### Option D: Use BWalletX (when v1.0 ships)

BWalletX (in active development) will be the recommended client for reading
and writing in a1a5NT. v1.0 release scheduled Q3 2026. Until then, options
A/B/C are functional.

## What to expect from the project

### Near-term (2026-Q2)

- **Stichting A1A5 registration** at notary in Almelo (NL) — week of 2026-05-19
- **Foundation Declaration v1.0 anchored** on a1a5NT
- **License v1.0 anchored** on a1a5NT (12-month binding, AI co-author clause,
  anti-patent defensive)
- **Public repository `a1a3`** with full protocol specifications

### Mid-term (2026-Q3)

- **IMASS algorithm specification** finalized
- **POP-3 Token specification** finalized
- **BWalletX v1.0** release
- **a1a5NT Genesis ceremony** — minting 85 tokenized epoch artifacts
- **Marketplace** opens for tokenized artifacts

### Long-term (2026-Q4 onwards)

- Federation between independent ANS servers
- Industry pilots (Plastechniek metal-fabrication workflows)
- Open-source ANS client implementations
- Expansion of POP-3 token primitive

## What we ask of NLnet

1. **Decision on application 2026-04-3f6** — original PopChain scope
2. **Tolerance for the maturity progression** to a1a5NT — same philosophy,
   expanded primitives, all open
3. **Formal amendment** after Stichting registration — we will submit updated
   project description matching current state
4. **Recognition of evidence over promise** — 86,073 blocks of operational
   PopChain stand as proof-of-execution; live a1a5NT infrastructure stands as
   proof-of-direction

We do not ask for additional funding beyond the original €45,000 request.
The scope expansion was done with available resources.

## What the Foundation gives in return

If grant is approved:

1. **`nlnet.a1a5` namespace** registered to NLnet (gift, regardless of
   funding outcome)
2. **One tokenized epoch artifact** from popchain-archive transferred to NLnet
   namespace at Genesis ceremony — choice of NLnet which artifact
3. **Public attribution** as Commons Fund supporter in all repositories
4. **Continued open-source development** with public spec in `a1a3` repo
5. **Quarterly status reports** delivered as XMSS-signed messages to
   nlnet.a1a5 inbox (sovereign channel demonstration)

If grant is not approved:

The project continues. PopChain demonstrated the technology works without
funding. a1a5NT has been built on personal resources. NLnet support would
accelerate but is not blocking.

## Founder background

**Paweł Piekut**
- Solo developer, 15+ months of full-time work on this project
- Background: industrial metal fabrication (ZZP Plastechniek, NL)
- Member: Metaalunie Jong Management (Dutch metal industry association)
- Location: Almelo, Overijssel, Netherlands
- Open-source commitment: Apache 2.0 / A1A5 Sovereign License v1.0

This is a one-person project at the technical core, with AI co-authorship
explicitly recognized in the License (Claude/Anthropic Opus 4.7 and prior
versions as substantive co-authors of cryptographic infrastructure).

## Contact summary

**Email (legacy infrastructure, supported):** `info@proof-of-process.org`
**Sovereign contact (project demonstration):** `pawel.a1a5`
**Read sovereign inbox:** `GET http://37.97.202.97:8443/v1/inbox/pawel`
**Foundation namespace:** `a1a5fndn.a1a5`
**Reserved for NLnet:** `nlnet.a1a5` (gift, awaiting NLnet handover)
**Public spec repository:** `github.com/a1a5NT/a1a3` (push pending Stichting)
**Live infrastructure:** http://37.97.202.97:8443 + 8444

Both channels are real. Use email for institutional workflows; sovereign for
testing the project itself.

---

**Issued by pawel.a1a5 · 2026-05-16**
**Document version:** v1.0
**License:** A1A5 Sovereign License v1.0
