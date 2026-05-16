# A1A5 FOUNDATION DECLARATION v1.0

**Document type:** Sovereign self-proclamation · Foundation in formation
**Issued by:** pawel.a1a5  (Paweł Piekut)
**Issue date:** 2026-05-16
**Effective:** Immediately upon ANS anchor
**Jurisdiction (interim):** Sole proprietorship · Almelo, Netherlands
**Jurisdiction (target):** Stichting A1A5, Netherlands (registration pending)
**Document version:** v1.0
**Document language:** English (primary) · Polski (secondary)

---

## ENGLISH

### Article I — Self-proclamation

I, **Paweł Piekut**, residing in Almelo, Netherlands, operating as
sole proprietor under ZZP Plastechniek (KvK NL), holding sovereign
namespace `pawel.a1a5` on the a1a5NT post-quantum network,
hereby declare the formation of:

> **a1a5 Foundation**
> (cryptographic Foundation entity, pending Stichting NL registration)
> Sovereign namespace: `a1a5fndn.a1a5`

This declaration is issued on **2026-05-16** and is anchored
cryptographically via XMSS-signed ANS message on the a1a5NT network.

### Article II — Purpose

The Foundation exists to:

1. **Custody** the popchain-archive cryptographic artifacts
   (85 epoch artifacts from popchain-alpha-testnet, 86,073 blocks,
   sealed 2026-05-16 at block 84455)

2. **Develop and maintain** a1a5NT — a post-quantum Layer-1 sovereign
   network featuring:
   - XMSS WOTS+ post-quantum signatures (NIST-standardized)
   - Sovereign naming via ANS (Address Namespace System)
   - Native messaging without SMTP dependency
   - Deterministic cross-architecture verification

3. **Promote** open infrastructure for verifiable computation,
   sovereign identity, and post-quantum cryptography.

4. **Steward** the transition from PopChain (generation 1, archived)
   to a1a5NT (generation 2, active development).

5. **Bridge** to formal legal entity (Stichting A1A5, Netherlands)
   upon notary registration, currently scheduled.

6. **Operate** BWalletX — the Foundation's flagship reference
   implementation of a **universal sovereign object courier** for
   a1a5NT. BWalletX is the first wallet primitive where the
   transferable unit is not limited to currency tokens, but extends
   to any XMSS-signed object: messages, documents, artifacts,
   certificates, sub-keys, attestations. Core object courier engine
   operational; UI/UX bridging in progress.

### Article III — Governance

**Interim governance (pre-Stichting):**

- Sole founder and custodian: **Paweł Piekut** (`pawel.a1a5`)
- All Foundation actions executed via XMSS-signed transactions
  under `pawel.a1a5` keypair
- Foundation namespace tree (`a1a5fndn.*`) reserved under
  self-proclamation, governed by the A1A5 Sovereign License v1.0

**Post-Stichting governance:**

- Stichting A1A5 (Stichting under Dutch law)
- Bestuur (board) composition to be established at notary
- All Foundation cryptographic assets and namespaces transfer to
  the Stichting upon successful registration
- This declaration serves as the cryptographic predecessor and
  founding intent document

### Article IV — Assets under custody (as of issue date)

**Sovereign namespaces (a1a5NT ANS):**

| Namespace | Type | Status |
|---|---|---|
| `a1a5fndn.a1a5` | Foundation root | Reserved · custody pawel.a1a5 |
| `archive.a1a5fndn.a1a5` | Archive parent | Reserved · custody pawel.a1a5 |
| `epoch0000..epoch0084.archive.a1a5fndn.a1a5` | Epoch artifacts (85) | Reserved · custody pawel.a1a5 |

**Cryptographic artifacts:**

- popchain-archive (86,073 blocks, 85 epochs, sealed 2026-05-16)
- Catalog blake3: `aa2c0dd44071526901ae6fbb1ed471646dcdc3f5129b4bcf9b2124a553adf2a3`
- 6/6 invariants PASS (cross-validation reproducible)

**Prior cryptographic work referenced:**

- PCIP License v1.0 (anchored popchain block 82112, 2026-05-01)
- POCC_ANCHOR_V1 = 417696

### Article V — Funding context

Funding application submitted to NLnet Foundation
under code **2026-04-3f6** (Commons Fund), pending review.

The original application described PopChain (generation 1, industrial
Proof-of-Process). Pivot to a1a5NT (generation 2, post-quantum sovereign
naming) constitutes a maturity progression of the same deterministic
philosophy. Amendment notice scheduled to NLnet post-Stichting.

### Article VI — License and reservation

All Foundation namespaces, artifacts, and protocol specifications
issued under the **A1A5 Sovereign License v1.0**, with twelve-month
binding window (expiring 2027-05-16) and post-quantum cryptographic
attestation.

Full license text: see `LICENSE-A1A5-SOVEREIGN-V1.md` (companion document).

### Article VII — Anchor and verification

This declaration is anchored on the a1a5NT network via XMSS-signed
ANS message issued by `pawel.a1a5`.

**Verification path:**

```
GET http://37.97.202.97:8443/v1/foundation/declaration
```

Returns: declaration text + XMSS signature + timestamp + signer pubkey.

**Public key (pawel.a1a5 XMSS root):**

```
root:     f7d4d034adba761b09d1c277a3d6e5178a2f79127f5b057a9433e90867d5939d
pub_seed: a442f5583b1df6ae6289bc56e2d8ae2eecc20fd0af53426fcb1fbbfe7d027d58
```

Any party may verify the authenticity of this declaration by checking
the XMSS signature against the published public key.

### Article VIII — Contact

**Sovereign contact (primary):** `pawel.a1a5`

**Read inbox (public):**
```
GET http://37.97.202.97:8443/v1/inbox/pawel
```

**Send message:** see `CONTACT-PROTOCOL.md` (BWalletX recommended client)

**Legal correspondence (interim fallback):** `info@proof-of-process.org`

This email address is maintained for legal continuity, institutional
correspondence, and parties without ANS tooling. It will be deprecated once
sovereign channel infrastructure (BWalletX v1.0, federated ANS) reaches
broader adoption. The Foundation's strategic position remains: email is
end-of-life infrastructure, and the project demonstrates the alternative.
Until the alternative is universally accessible, legacy infrastructure
remains supported.

**Physical address:** Almelo, Netherlands (full address available upon Stichting registration)

### Article IX — Signature and date

Signed by:

**Paweł Piekut**
sovereign namespace: `pawel.a1a5`
date: 2026-05-16
ANS anchor message ID: [filled at anchor time]

---

## POLSKI

### Artykuł I — Samo-proklamacja

Ja, **Paweł Piekut**, zamieszkały w Almelo, Holandia, działający
jako jednoosobowy przedsiębiorca pod ZZP Plastechniek (KvK NL),
posiadający suwerenny namespace `pawel.a1a5` w sieci post-kwantowej
a1a5NT, niniejszym deklaruję założenie:

> **Fundacji a1a5**
> (kryptograficzny podmiot Foundation, przed rejestracją Stichting NL)
> Suwerenny namespace: `a1a5fndn.a1a5`

Deklaracja wydana **2026-05-16**, zakotwiczona kryptograficznie
poprzez wiadomość ANS podpisaną XMSS w sieci a1a5NT.

### Artykuł II — Cel Fundacji

Fundacja istnieje aby:

1. **Sprawować pieczę** nad artefaktami kryptograficznymi
   popchain-archive (85 epoch artefaktów z popchain-alpha-testnet,
   86 073 bloki, zapieczętowane 2026-05-16 na bloku 84455)

2. **Rozwijać i utrzymywać** a1a5NT — suwerenną sieć Layer-1
   post-kwantową zawierającą:
   - Podpisy post-kwantowe XMSS WOTS+ (standaryzowane przez NIST)
   - Suwerenną przestrzeń nazw przez ANS
   - Natywny system wiadomości bez zależności SMTP
   - Deterministyczną weryfikację cross-architektury

3. **Promować** otwartą infrastrukturę dla weryfikowalnej obliczeń,
   suwerennej tożsamości i kryptografii post-kwantowej.

4. **Zarządzać** przejściem z PopChain (generacja 1, zarchiwizowana)
   do a1a5NT (generacja 2, aktywny rozwój).

5. **Połączyć z formalnym podmiotem prawnym** (Stichting A1A5,
   Holandia) po rejestracji u notariusza, zaplanowanej.

6. **Operować** BWalletX — flagowy referencyjny implementacja
   Fundacji **uniwersalnego suwerennego kuriera obiektów** dla
   a1a5NT. BWalletX jest pierwszym prymitywem walleta, w którym
   jednostka przenoszalna nie jest ograniczona do tokenów walutowych,
   lecz rozciąga się na dowolny obiekt podpisany XMSS: wiadomości,
   dokumenty, artefakty, certyfikaty, sub-klucze, atestacje.
   Rdzeń silnika kuriera obiektów operacyjny; mostkowanie UI/UX
   w trakcie.

### Artykuł III — Zarządzanie

**Zarządzanie tymczasowe (przed-Stichting):**

- Jedyny fundator i custodian: **Paweł Piekut** (`pawel.a1a5`)
- Wszystkie działania Fundacji wykonywane przez transakcje
  podpisane XMSS pod parą kluczy `pawel.a1a5`
- Drzewo nazw Fundacji (`a1a5fndn.*`) zarezerwowane pod
  samo-proklamacją, regulowane Licencją Suwerenności A1A5 v1.0

**Zarządzanie post-Stichting:**

- Stichting A1A5 (Stichting pod prawem holenderskim)
- Skład bestuur (zarząd) zostanie ustalony u notariusza
- Wszystkie aktywa kryptograficzne i namespacy Fundacji
  przejdą do Stichting po pomyślnej rejestracji
- Niniejsza deklaracja stanowi kryptograficznego poprzednika
  i dokument intencji założycielskiej

### Artykuł IV — Aktywa pod pieczą (na dzień wydania)

**Suwerenne namespacy (a1a5NT ANS):**

| Namespace | Typ | Status |
|---|---|---|
| `a1a5fndn.a1a5` | Korzeń Fundacji | Zarezerwowany · piecza pawel.a1a5 |
| `archive.a1a5fndn.a1a5` | Korzeń archiwum | Zarezerwowany · piecza pawel.a1a5 |
| `epoch0000..epoch0084.archive.a1a5fndn.a1a5` | Artefakty epok (85) | Zarezerwowane · piecza pawel.a1a5 |

**Artefakty kryptograficzne:**

- popchain-archive (86 073 bloki, 85 epok, zapieczętowane 2026-05-16)
- Catalog blake3: `aa2c0dd44071526901ae6fbb1ed471646dcdc3f5129b4bcf9b2124a553adf2a3`
- 6/6 niezmienników PASS (cross-validation reproducible)

**Wcześniejsza praca kryptograficzna:**

- PCIP License v1.0 (zakotwiczona popchain block 82112, 2026-05-01)
- POCC_ANCHOR_V1 = 417696

### Artykuł V — Kontekst finansowania

Wniosek o finansowanie złożony do NLnet Foundation pod kodem
**2026-04-3f6** (Commons Fund), w trakcie przeglądu.

Oryginalny wniosek opisywał PopChain (generacja 1, przemysłowy
Proof-of-Process). Przejście do a1a5NT (generacja 2, suwerenne nazewnictwo
post-kwantowe) stanowi progresję dojrzałości tej samej deterministycznej
filozofii. Notyfikacja zmiany zaplanowana do NLnet po Stichting.

### Artykuł VI — Licencja i rezerwacja

Wszystkie namespacy Fundacji, artefakty i specyfikacje protokołów
wydane pod **Licencją Suwerenności A1A5 v1.0**, z dwunastomiesięcznym
oknem wiążącym (do 2027-05-16) i kryptograficznym atestowaniem
post-kwantowym.

Pełna treść licencji: zobacz `LICENSE-A1A5-SOVEREIGN-V1.md` (dokument towarzyszący).

### Artykuł VII — Anchor i weryfikacja

Deklaracja zakotwiczona w sieci a1a5NT poprzez wiadomość ANS
podpisaną XMSS wydaną przez `pawel.a1a5`.

**Ścieżka weryfikacji:**

```
GET http://37.97.202.97:8443/v1/foundation/declaration
```

Zwraca: tekst deklaracji + sygnaturę XMSS + timestamp + klucz publiczny podpisującego.

**Klucz publiczny (XMSS root pawel.a1a5):**

```
root:     f7d4d034adba761b09d1c277a3d6e5178a2f79127f5b057a9433e90867d5939d
pub_seed: a442f5583b1df6ae6289bc56e2d8ae2eecc20fd0af53426fcb1fbbfe7d027d58
```

Każda strona może zweryfikować autentyczność tej deklaracji
sprawdzając sygnaturę XMSS względem opublikowanego klucza publicznego.

### Artykuł VIII — Kontakt

**Kontakt suwerenny (główny):** `pawel.a1a5`

**Odczyt inbox (publiczny):**
```
GET http://37.97.202.97:8443/v1/inbox/pawel
```

**Wysłanie wiadomości:** zobacz `CONTACT-PROTOCOL.md` (BWalletX rekomendowany klient)

**Korespondencja prawna (tymczasowy fallback):** `info@proof-of-process.org`

Ten adres email jest utrzymywany dla ciągłości prawnej, korespondencji
instytucjonalnej i stron bez narzędzi ANS. Zostanie zdeprecjonowany gdy
infrastruktura kanału suwerennego (BWalletX v1.0, federacja ANS) osiągnie
szersze przyjęcie. Strategiczna pozycja Fundacji pozostaje: email to
infrastruktura schyłkowa, a projekt demonstruje alternatywę. Dopóki
alternatywa nie jest powszechnie dostępna, legacy infrastructure pozostaje
wspierana.

**Adres fizyczny:** Almelo, Holandia (pełen adres dostępny po rejestracji Stichting)

### Artykuł IX — Podpis i data

Podpisane przez:

**Paweł Piekut**
suwerenny namespace: `pawel.a1a5`
data: 2026-05-16
ID wiadomości anchor ANS: [uzupełnione w momencie zakotwiczenia]

---

## ANCHOR METADATA (filled programmatically)

```yaml
declaration_version: "v1.0"
document_blake3: <computed at anchor>
ans_message_id:   <computed at anchor>
xmss_signature:   <computed at anchor>
xmss_index_used:  <leaf index at anchor>
issued_unix:      <unix timestamp at anchor>
issuer:           pawel.a1a5
recipient:        a1a5fndn.a1a5  (self-broadcast)
verification_url: http://37.97.202.97:8443/v1/foundation/declaration
companion_docs:
  - LICENSE-A1A5-SOVEREIGN-V1.md
  - ANS-PROTOCOL-SPEC.md
  - POPCHAIN-ARCHIVE-PROVENANCE.md
  - ROADMAP.md
```

---

**END OF DECLARATION**
