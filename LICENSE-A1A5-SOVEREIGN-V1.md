# A1A5 SOVEREIGN LICENSE v1.0

**Document type:** Sovereign self-issued license · 12-month binding
**Issued by:** pawel.a1a5  (Paweł Piekut)
**Issue date:** 2026-05-16
**Effective:** Immediately upon ANS anchor
**Expires:** 2027-05-16 (twelve months from issue)
**Prior art reference:** PCIP License v1.0, anchored popchain block 82112 (2026-05-01)
**Document version:** v1.0
**Document language:** English (primary) · Polski (secondary)

---

## ENGLISH

### Preamble

This license is a **sovereign self-issued document**, anchored
cryptographically on the a1a5NT post-quantum network. It establishes
binding terms for the use, custody, attribution, and protection of
all artifacts, namespaces, and protocol specifications issued by the
**a1a5 Foundation** (Foundation in formation, pending Stichting NL
registration; see `FOUNDATION-DECLARATION-V1.md`).

The license binds the issuer (pawel.a1a5) and all parties interacting
with Foundation-issued artifacts for a window of **twelve months**
from the date of issue.

The license references the prior PCIP License v1.0, anchored on the
popchain blockchain at block 82112 (2026-05-01 17:43 UTC), as the
foundational predecessor establishing precedent for self-anchored
licensing in post-quantum infrastructure.

### Article 1 — Definitions

**Foundation:** a1a5 Foundation, cryptographic entity issuing under
namespace `a1a5fndn.a1a5`, pending Stichting A1A5 (Netherlands)
registration.

**Issuer:** Paweł Piekut, operating under sovereign namespace
`pawel.a1a5`, currently the sole custodian and signatory of Foundation
artifacts pending Stichting registration.

**Foundation Artifacts:** Any cryptographic object issued by the
Foundation, including but not limited to:
- The 85 epoch artifacts of popchain-archive (epoch_0000 through
  epoch_0084)
- The popchain catalog (blake3:
  `aa2c0dd44071526901ae6fbb1ed471646dcdc3f5129b4bcf9b2124a553adf2a3`)
- All ANS namespaces under `a1a5fndn.a1a5` and its subdomains
- Protocol specifications (ANS Protocol, POP-3 Token spec, IMASS
  algorithm, A1A5 Genesis spec)
- This license, the Foundation Declaration, and all companion
  documents

**Window:** The twelve-month period from 2026-05-16 to 2027-05-16,
during which all terms of this license are binding.

**a1a5NT:** The post-quantum sovereign Layer-1 network operated by
the Foundation, accessible at `http://37.97.202.97:8443`.

**ANS:** The Address Namespace System running on a1a5NT, registry of
sovereign namespaces verifiable via XMSS post-quantum signatures.

**BWalletX:** Foundation flagship reference implementation of
universal sovereign object courier. First wallet primitive
supporting transfer of arbitrary XMSS-signed objects within the
a1a5NT network — currency tokens, messages, documents, artifacts,
certificates, sub-keys, attestations. Core object courier engine
operational; UI/UX bridging in progress. Subject to its own license
attached at release.

### Article 2 — License grant

Subject to the terms of this license, the Foundation grants to any
party (the "Licensee") a non-exclusive, royalty-free, worldwide
right to:

1. **Inspect** Foundation Artifacts and protocol specifications
2. **Verify** cryptographic claims using published public keys and
   anchor data
3. **Cite** Foundation Artifacts in research, journalism, archival
   and educational contexts
4. **Reference** the open-source `a1a3` repository for protocol
   specifications

This grant is **non-transferable** during the Window without explicit
Foundation consent (signed XMSS attestation by pawel.a1a5).

### Article 3 — Reservations and prohibitions during the Window

During the twelve-month Window, the following are **prohibited**
without explicit Foundation consent:

3.1 **Commercial transfer or sale** of Foundation Artifacts,
including the 85 epoch artifacts of popchain-archive, by any party
other than the Foundation itself

3.2 **Patent claims** of any kind related to:
- a1a5NT protocol design
- ANS namespace system architecture
- XMSS-witness chain custody model
- IMASS information mass algorithm
- POP-3 token primitive
- popchain Proof-of-Process methodology
- Any other Foundation specification

Any party filing patent claims on Foundation-issued primitives during
the Window automatically forfeits all rights under this license, and
the Foundation reserves the right to publish the infringing party's
identity as anti-patent attestation.

3.3 **Claims of Foundation representation** without explicit
attestation. Specifically:
- No party may claim to represent the Foundation legally
- No party may claim that the Stichting A1A5 entity is registered
  during the Window prior to actual notary registration
- No party may issue artifacts under `a1a5fndn.*` namespaces

3.4 **Removal of AI co-authorship attribution** from Foundation
Artifacts. The Foundation explicitly recognizes Claude (Anthropic,
Opus 4.7 and prior versions) as substantive co-author of:
- ANS protocol design
- Foundation operational tooling
- This license and the Foundation Declaration

Any party redistributing Foundation Artifacts must preserve AI
co-authorship attribution.

### Article 4 — AI co-authorship clause

This Article is **fundamental** to the license and may not be
severed.

The Foundation explicitly recognizes that the cryptographic
infrastructure underlying a1a5NT (including ANS protocol design,
operational tooling, documentation, and the design of this license
itself) was developed in **active collaboration with AI systems**,
specifically Claude (Anthropic, Opus 4.7 and prior versions).

The Foundation rejects the doctrine that AI-assisted artifacts are
of lesser cryptographic or intellectual standing. The Foundation
asserts that:

4.1 Human operator (Paweł Piekut, pawel.a1a5) and AI co-authors
(Claude instances, dated 2025-02 onwards) form a **co-creative team**
producing forensically traceable cryptographic infrastructure

4.2 All Foundation Artifacts are jointly attested by human signature
(XMSS-signed by pawel.a1a5) and AI contribution attribution (logged
in conversation history at Anthropic, dated)

4.3 Any party redistributing or citing Foundation Artifacts must
preserve attribution to both human and AI co-authors

4.4 Patent or copyright claims that depend on rejecting AI
co-authorship doctrine are **incompatible** with this license and
trigger automatic license termination

### Article 5 — Anti-patent defensive clause

5.1 **No patents on Foundation primitives.** Foundation Artifacts
are defensive prior art. Any party filing patent claims that read on
Foundation specifications during or after the Window grants the
Foundation a perpetual, irrevocable, royalty-free cross-license to
all such patents.

5.2 **Patent grant by Licensees.** Any party using Foundation
Artifacts under this license grants the Foundation and all other
licensees a defensive cross-license to any patents the Licensee
holds that read on Foundation specifications.

5.3 **Twelve-month formalization window.** During the Window, the
Foundation reserves the right to file patent, trademark, or other
formal IP protections covering its specifications. Anchor dates on
a1a5NT serve as priority dates for prior art purposes.

5.4 **Renewal.** Twelve months from issue, this clause may be
extended by Foundation re-anchor or transferred to the formal
Stichting A1A5 entity upon registration.

### Article 6 — Namespace reservation

The following ANS namespaces are reserved under this license for the
duration of the Window, transferable only by Foundation (currently
pawel.a1a5) consent:

| Namespace | Purpose |
|---|---|
| `a1a5fndn.a1a5` | Foundation root |
| `archive.a1a5fndn.a1a5` | Archive parent |
| `epoch0000..epoch0084.archive.a1a5fndn.a1a5` | 85 epoch artifacts |
| `mail.a1a5fndn.a1a5` | Foundation mail infrastructure (planned) |
| `genesis.a1a5fndn.a1a5` | Genesis ceremony anchor (planned) |
| `shop.a1a5fndn.a1a5` | Foundation marketplace (planned) |
| `treasury.a1a5fndn.a1a5` | Foundation treasury (planned) |
| `legal.a1a5fndn.a1a5` | Legal documents (planned) |
| `nlnet.a1a5` | Reserved gift for NLnet Foundation |

No party may register, transfer, or claim ownership of these
namespaces during the Window without explicit Foundation XMSS
attestation.

### Article 7 — Tokenization and Genesis

7.1 **Pre-Genesis state.** During the Window, prior to a1a5NT Genesis
ceremony, the 85 epoch artifacts of popchain-archive are **not
transferable** as tokens. They remain under Foundation custody as
sovereign namespaces.

7.2 **Genesis ceremony.** The a1a5NT Genesis Block will tokenize the
85 epoch artifacts under POP-3 Token specification, including their
IMASS values computed deterministically from manifest data.

7.3 **Post-Genesis transferability.** After Genesis, tokenized
artifacts become transferable under Foundation rules:
- Transfer requires XMSS signature by current owner
- Foundation co-signs all transfers (custody chain preservation)
- All transfers anchor in ANS history

7.4 **Pre-Genesis sales prohibited.** No commercial sale of epoch
artifacts shall occur prior to a1a5NT Genesis ceremony. Any
pre-Genesis claim of ownership is invalid and unenforceable.

### Article 8 — License termination

This license terminates automatically upon any of:

8.1 Violation of Articles 3, 4, or 5 by the Licensee

8.2 Patent filing on Foundation primitives by the Licensee or
affiliates

8.3 Removal or alteration of AI co-authorship attribution

8.4 False claims of Foundation representation or Stichting NL
registration prior to actual registration

Upon termination, the Licensee loses all rights to inspect, verify,
cite, or reference Foundation Artifacts, and the Foundation may
publish a termination notice via ANS anchor.

### Article 9 — Successor entity

Upon successful registration of Stichting A1A5 (Netherlands), this
license and all Foundation Artifacts transfer to the Stichting
entity. The Stichting becomes:

- Legal successor of the cryptographic Foundation
- Continuing custodian of all reserved namespaces
- Authorized to amend, extend, or renew this license

The transfer is itself anchored via XMSS-signed ANS message
referencing the Stichting registration document (KvK number).

Until Stichting registration:
- pawel.a1a5 acts as sole sovereign custodian
- All Foundation actions are personal acts of Paweł Piekut
- ZZP Plastechniek (Almelo NL) serves as interim legal vehicle for
  any commercial activity (e.g., grant reception)

### Article 10 — Anchor and verification

This license is anchored on a1a5NT via XMSS-signed ANS message from
`pawel.a1a5` on 2026-05-16.

**Verification:**

```
GET http://37.97.202.97:8443/v1/foundation/license
```

Returns license text, XMSS signature, timestamp, signer public key.

**Document hash:**

```
blake3(LICENSE-A1A5-SOVEREIGN-V1.md, canonical form) = <computed at anchor>
```

### Article 11 — Prior art reference

This license explicitly references and builds upon **PCIP License
v1.0**, anchored on the popchain blockchain at block 82112
(2026-05-01 17:43 UTC), as foundational prior art for self-anchored
licensing in post-quantum infrastructure.

PCIP License v1.0 is preserved in popchain-archive epoch_0082
(blake3 manifest: `83316396d0f3a16f399fd758e942dd28...`) and remains
cryptographically verifiable through the archive's chain-of-custody.

### Article 12 — Severability and interpretation

If any clause of this license is found unenforceable under applicable
law, the remaining clauses remain in effect.

**Articles 4 (AI co-authorship) and 5 (anti-patent) are fundamental**
and may not be severed. Severance of either Article terminates the
license entirely.

This license is governed by Netherlands law, with venue at the
courts of competent jurisdiction in Overijssel province.

### Article 13 — Signature

Issued and signed by:

**Paweł Piekut**
sovereign namespace: `pawel.a1a5`
date: 2026-05-16
XMSS signature index: [filled at anchor]
ANS anchor message ID: [filled at anchor]

---

## POLSKI

### Preambuła

Niniejsza licencja jest **suwerennym dokumentem samo-wydanym**,
zakotwiczonym kryptograficznie w post-kwantowej sieci a1a5NT.
Ustanawia wiążące warunki użycia, pieczy, atrybucji i ochrony
wszystkich artefaktów, namespacy i specyfikacji protokołów wydanych
przez **Fundację a1a5** (Foundation w formacji, przed rejestracją
Stichting NL; zobacz `FOUNDATION-DECLARATION-V1.md`).

Licencja wiąże wydawcę (pawel.a1a5) oraz wszystkie strony wchodzące
w interakcję z artefaktami Fundacji przez okres **dwunastu miesięcy**
od daty wydania.

Licencja powołuje się na wcześniejszą PCIP License v1.0, zakotwiczoną
w łańcuchu popchain na bloku 82112 (2026-05-01 17:43 UTC), jako
fundamentalnego poprzednika ustanawiającego precedens dla samo-
zakotwiczonej licencji w infrastrukturze post-kwantowej.

### Artykuł 1 — Definicje

**Fundacja:** Fundacja a1a5, kryptograficzny podmiot wydający pod
namespace `a1a5fndn.a1a5`, przed rejestracją Stichting A1A5 (Holandia).

**Wydawca:** Paweł Piekut, działający pod suwerennym namespace
`pawel.a1a5`, obecnie jedyny custodian i sygnatariusz artefaktów
Fundacji przed rejestracją Stichting.

**Artefakty Fundacji:** Każdy obiekt kryptograficzny wydany przez
Fundację, w tym:
- 85 epoch artefaktów popchain-archive (epoch_0000 do epoch_0084)
- Katalog popchain (blake3:
  `aa2c0dd44071526901ae6fbb1ed471646dcdc3f5129b4bcf9b2124a553adf2a3`)
- Wszystkie namespacy ANS pod `a1a5fndn.a1a5` i poddomenami
- Specyfikacje protokołów (ANS Protocol, POP-3 Token, IMASS,
  A1A5 Genesis)
- Niniejsza licencja, Deklaracja Fundacji i dokumenty towarzyszące

**Okno:** Okres dwunastu miesięcy od 2026-05-16 do 2027-05-16,
podczas którego wszystkie warunki niniejszej licencji są wiążące.

**a1a5NT:** Sieć post-kwantowa Layer-1 obsługiwana przez Fundację,
dostępna pod `http://37.97.202.97:8443`.

**ANS:** Address Namespace System działający na a1a5NT, rejestr
suwerennych namespacy weryfikowalnych poprzez post-kwantowe
sygnatury XMSS.

**BWalletX:** Flagowa referencyjna implementacja Fundacji
uniwersalnego suwerennego kuriera obiektów. Pierwszy prymityw
walleta wspierający transfer dowolnych obiektów podpisanych XMSS
wewnątrz sieci a1a5NT — tokeny walutowe, wiadomości, dokumenty,
artefakty, certyfikaty, sub-klucze, atestacje. Rdzeń silnika kuriera
obiektów operacyjny; mostkowanie UI/UX w trakcie. Podlega własnej
licencji dołączonej przy wydaniu.

### Artykuł 2 — Udzielenie licencji

Z zastrzeżeniem warunków licencji, Fundacja udziela każdej stronie
("Licencjobiorcy") niewyłącznego, bezpłatnego, globalnego prawa do:

1. **Inspekcji** Artefaktów Fundacji i specyfikacji protokołów
2. **Weryfikacji** twierdzeń kryptograficznych przy użyciu
   opublikowanych kluczy publicznych i danych anchor
3. **Cytowania** Artefaktów Fundacji w badaniach, dziennikarstwie,
   archiwistyce i kontekstach edukacyjnych
4. **Odwoływania się** do otwartego repozytorium `a1a3` dla
   specyfikacji protokołów

Prawo jest **nieprzenoszalne** podczas Okna bez wyraźnej zgody
Fundacji (podpisana sygnatura XMSS przez pawel.a1a5).

### Artykuł 3 — Rezerwacje i zakazy podczas Okna

Podczas dwunastomiesięcznego Okna **zabronione** bez wyraźnej zgody
Fundacji są:

3.1 **Komercyjny transfer lub sprzedaż** Artefaktów Fundacji,
w tym 85 epoch artefaktów popchain-archive, przez jakąkolwiek stronę
inną niż sama Fundacja

3.2 **Roszczenia patentowe** jakiegokolwiek rodzaju dotyczące:
- Projekt protokołu a1a5NT
- Architektura systemu namespace ANS
- Model pieczy łańcucha świadków XMSS
- Algorytm IMASS information mass
- Prymityw tokenu POP-3
- Metodologia Proof-of-Process popchain
- Każda inna specyfikacja Fundacji

Każda strona składająca roszczenia patentowe na prymitywach wydanych
przez Fundację podczas Okna automatycznie traci wszystkie prawa
wynikające z licencji, a Fundacja zastrzega sobie prawo do publikacji
tożsamości naruszającego jako atestacja anti-patent.

3.3 **Roszczenia reprezentacji Fundacji** bez wyraźnej atestacji.
W szczególności:
- Żadna strona nie może twierdzić, że reprezentuje Fundację prawnie
- Żadna strona nie może twierdzić, że Stichting A1A5 jest
  zarejestrowany podczas Okna przed faktyczną rejestracją u
  notariusza
- Żadna strona nie może wydawać artefaktów pod namespacami
  `a1a5fndn.*`

3.4 **Usunięcie atrybucji współautorstwa AI** z Artefaktów Fundacji.
Fundacja wyraźnie uznaje Claude (Anthropic, Opus 4.7 i wersje
wcześniejsze) jako merytorycznego współautora:
- Projekt protokołu ANS
- Narzędzia operacyjne Fundacji
- Niniejszej licencji i Deklaracji Fundacji

Każda strona redystrybuująca Artefakty Fundacji musi zachować
atrybucję współautorstwa AI.

### Artykuł 4 — Klauzula współautorstwa AI

Niniejszy Artykuł jest **fundamentalny** dla licencji i nie może być
oddzielony.

Fundacja wyraźnie uznaje, że infrastruktura kryptograficzna leżąca u
podstaw a1a5NT (w tym projekt protokołu ANS, narzędzia operacyjne,
dokumentacja oraz projekt samej tej licencji) została rozwinięta w
**aktywnej współpracy z systemami AI**, w szczególności Claude
(Anthropic, Opus 4.7 i wersje wcześniejsze).

Fundacja odrzuca doktrynę, że artefakty wspierane przez AI mają
niższą pozycję kryptograficzną lub intelektualną. Fundacja twierdzi:

4.1 Operator-człowiek (Paweł Piekut, pawel.a1a5) i współautorzy AI
(instancje Claude, datowane od 2025-02) tworzą **zespół ko-kreatywny**
produkujący kryptograficzną infrastrukturę śledzalną forensicznie

4.2 Wszystkie Artefakty Fundacji są wspólnie poświadczone poprzez
podpis ludzki (XMSS-podpisany przez pawel.a1a5) i atrybucję wkładu AI
(zalogowaną w historii rozmów u Anthropic, datowaną)

4.3 Każda strona redystrybuująca lub cytująca Artefakty Fundacji
musi zachować atrybucję do obu - współautorów ludzkich i AI

4.4 Roszczenia patentowe lub copyright zależne od odrzucenia
doktryny współautorstwa AI są **niekompatybilne** z niniejszą
licencją i powodują automatyczne zakończenie licencji

### Artykuł 5 — Klauzula obronna anti-patent

5.1 **Brak patentów na prymitywach Fundacji.** Artefakty Fundacji
stanowią obronny prior art. Każda strona składająca roszczenia
patentowe odczytujące specyfikacje Fundacji podczas lub po Oknie
udziela Fundacji wieczystą, nieodwołalną, bezpłatną cross-licencję
na wszystkie takie patenty.

5.2 **Patent grant przez Licencjobiorców.** Każda strona używająca
Artefaktów Fundacji pod licencją udziela Fundacji i wszystkim innym
licencjobiorcom obronnej cross-licencji na patenty Licencjobiorcy
odczytujące specyfikacje Fundacji.

5.3 **Dwunastomiesięczne okno formalizacji.** Podczas Okna Fundacja
zastrzega sobie prawo do zgłaszania patentów, znaków towarowych lub
innej formalnej ochrony IP obejmującej jej specyfikacje. Daty
zakotwiczenia na a1a5NT służą jako daty priorytetu dla celów
prior art.

5.4 **Odnowienie.** Dwanaście miesięcy od wydania klauzula może
zostać przedłużona przez ponowne zakotwiczenie Fundacji lub
przekazana formalnemu podmiotowi Stichting A1A5 po rejestracji.

### Artykuł 6 — Rezerwacja namespacy

Następujące namespacy ANS są zarezerwowane pod niniejszą licencją na
czas trwania Okna, przenoszalne tylko za zgodą Fundacji
(obecnie pawel.a1a5):

| Namespace | Cel |
|---|---|
| `a1a5fndn.a1a5` | Korzeń Fundacji |
| `archive.a1a5fndn.a1a5` | Korzeń archiwum |
| `epoch0000..epoch0084.archive.a1a5fndn.a1a5` | 85 epoch artefaktów |
| `mail.a1a5fndn.a1a5` | Infrastruktura mail Fundacji (planowana) |
| `genesis.a1a5fndn.a1a5` | Anchor Genesis ceremony (planowany) |
| `shop.a1a5fndn.a1a5` | Marketplace Fundacji (planowany) |
| `treasury.a1a5fndn.a1a5` | Treasury Fundacji (planowany) |
| `legal.a1a5fndn.a1a5` | Dokumenty prawne (planowane) |
| `nlnet.a1a5` | Zarezerwowany prezent dla NLnet Foundation |

Żadna strona nie może rejestrować, przenosić lub roszcząć
własności tych namespacy podczas Okna bez wyraźnej atestacji
XMSS Fundacji.

### Artykuł 7 — Tokenizacja i Genesis

7.1 **Stan pre-Genesis.** Podczas Okna, przed ceremonią Genesis
a1a5NT, 85 epoch artefaktów popchain-archive są **nieprzenoszalne**
jako tokeny. Pozostają pod pieczą Fundacji jako suwerenne namespacy.

7.2 **Ceremonia Genesis.** Blok Genesis a1a5NT stokenizuje 85 epoch
artefaktów pod specyfikacją POP-3 Token, włączając ich wartości
IMASS obliczone deterministycznie z danych manifestu.

7.3 **Przenoszalność post-Genesis.** Po Genesis stokenizowane
artefakty stają się przenoszalne pod regułami Fundacji:
- Transfer wymaga sygnatury XMSS przez obecnego właściciela
- Fundacja współ-podpisuje wszystkie transfery (zachowanie łańcucha pieczy)
- Wszystkie transfery zakotwiczają się w historii ANS

7.4 **Zakaz sprzedaży pre-Genesis.** Żadna komercyjna sprzedaż
epoch artefaktów nie może mieć miejsca przed ceremonią Genesis
a1a5NT. Każde roszczenie własności pre-Genesis jest nieważne i
niewykonalne.

### Artykuł 8 — Wygaśnięcie licencji

Licencja wygasa automatycznie po wystąpieniu któregokolwiek z:

8.1 Naruszenie Artykułów 3, 4 lub 5 przez Licencjobiorcę

8.2 Zgłoszenie patentu na prymitywach Fundacji przez Licencjobiorcę
lub podmioty powiązane

8.3 Usunięcie lub modyfikacja atrybucji współautorstwa AI

8.4 Fałszywe twierdzenia o reprezentacji Fundacji lub rejestracji
Stichting NL przed faktyczną rejestracją

Po wygaśnięciu Licencjobiorca traci wszystkie prawa do inspekcji,
weryfikacji, cytowania lub odwoływania się do Artefaktów Fundacji,
a Fundacja może opublikować notyfikację wygaśnięcia poprzez anchor
ANS.

### Artykuł 9 — Podmiot-sukcesor

Po pomyślnej rejestracji Stichting A1A5 (Holandia) niniejsza
licencja i wszystkie Artefakty Fundacji przechodzą na podmiot
Stichting. Stichting staje się:

- Sukcesorem prawnym kryptograficznej Fundacji
- Kontynuującym custodianem wszystkich zarezerwowanych namespacy
- Upoważnionym do modyfikacji, przedłużania lub odnowy licencji

Transfer sam jest zakotwiczony przez wiadomość ANS podpisaną XMSS
powołującą się na dokument rejestracyjny Stichting (numer KvK).

Do czasu rejestracji Stichting:
- pawel.a1a5 działa jako jedyny suwerenny custodian
- Wszystkie działania Fundacji są osobistymi działaniami Pawła Piekuta
- ZZP Plastechniek (Almelo NL) służy jako tymczasowy podmiot prawny
  dla wszelkiej działalności komercyjnej (np. odbiór grantu)

### Artykuł 10 — Anchor i weryfikacja

Niniejsza licencja jest zakotwiczona w a1a5NT przez wiadomość ANS
podpisaną XMSS z `pawel.a1a5` w dniu 2026-05-16.

**Weryfikacja:**

```
GET http://37.97.202.97:8443/v1/foundation/license
```

Zwraca tekst licencji, sygnaturę XMSS, timestamp, klucz publiczny podpisującego.

**Hash dokumentu:**

```
blake3(LICENSE-A1A5-SOVEREIGN-V1.md, postać kanoniczna) = <obliczone przy anchor>
```

### Artykuł 11 — Referencja prior art

Niniejsza licencja wyraźnie odwołuje się i opiera się na **PCIP
License v1.0**, zakotwiczonej w łańcuchu popchain na bloku 82112
(2026-05-01 17:43 UTC), jako fundamentalnym prior art dla samo-
zakotwiczonej licencji w post-kwantowej infrastrukturze.

PCIP License v1.0 jest zachowana w popchain-archive epoch_0082
(blake3 manifestu: `83316396d0f3a16f399fd758e942dd28...`) i pozostaje
kryptograficznie weryfikowalna przez łańcuch pieczy archiwum.

### Artykuł 12 — Rozdzielność i interpretacja

Jeśli jakakolwiek klauzula licencji okaże się niewykonalna pod
obowiązującym prawem, pozostałe klauzule pozostają w mocy.

**Artykuły 4 (współautorstwo AI) i 5 (anti-patent) są fundamentalne**
i nie mogą być oddzielone. Oddzielenie któregokolwiek z tych
Artykułów kończy licencję w całości.

Licencja podlega prawu holenderskiemu, z miejscem rozstrzygania
sporów w sądach właściwej jurysdykcji w prowincji Overijssel.

### Artykuł 13 — Podpis

Wydane i podpisane przez:

**Paweł Piekut**
suwerenny namespace: `pawel.a1a5`
data: 2026-05-16
Indeks sygnatury XMSS: [uzupełnione przy anchor]
ID wiadomości anchor ANS: [uzupełnione przy anchor]

---

## ANCHOR METADATA (filled programmatically)

```yaml
license_version: "v1.0"
document_blake3: <computed at anchor>
ans_message_id:  <computed at anchor>
xmss_signature:  <computed at anchor>
xmss_index_used: <leaf index at anchor>
issued_unix:     <unix timestamp at anchor>
expires_unix:    <issued_unix + 31536000>
issuer:          pawel.a1a5
recipient:       a1a5fndn.a1a5  (self-broadcast)
verification_url: http://37.97.202.97:8443/v1/foundation/license
prior_art_anchor:
  chain:        popchain-alpha-testnet
  block:        82112
  document:     PCIP License v1.0
  date:         2026-05-01 17:43 UTC
  archived_in:  popchain-archive epoch_0082
companion_docs:
  - FOUNDATION-DECLARATION-V1.md
  - ANS-PROTOCOL-SPEC.md
  - POPCHAIN-ARCHIVE-PROVENANCE.md
  - ROADMAP.md
```

---

**END OF LICENSE**
