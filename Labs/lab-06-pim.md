# Lab 06 – Privileged Identity Management (PIM)

**Datum:** Mars 2026
**Miljö:** Microsoft 365 Business Premium + Microsoft Entra ID P2
**Tenant:** \<tenant\>.onmicrosoft.com
**Svårighetsgrad:** Medel–Avancerad
**Tidsåtgång:** ~1.5 timmar

---

## Scenario

Privilegierade konton är de mest attraktiva måltavlorna för angripare. En Global Administrator med permanent aktiv roll är ett konto som, om det komprometteras, ger angriparen total kontroll över hela tenanten – dygnet runt, utan tidsbegränsning.

Privileged Identity Management (PIM) löser detta genom **Zero Standing Privilege** – principen att ingen användare ska ha privilegierad åtkomst längre än nödvändigt. Istället för permanent adminåtkomst aktiverar användaren rollen tillfälligt när den behövs, med MFA-krav, motivering och godkännande.

I det här labbet konfigureras PIM för Global Administrator-rollen och hela aktiveringsflödet testas från begäran till godkännande.

---

## Mål

- Tilldela Alice rollen Global Administrator som Eligible via PIM
- Konfigurera role settings med 4 timmars maxaktivering och krav på godkännande
- Testa hela aktiveringsflödet som Alice
- Godkänna begäran som admin och verifiera i Resource audit

---

## Förutsättningar

- Slutfört Lab 01–05
- Alice konfigurerad som testanvändare med Entra ID P2-licens
- MFA registrerat för Alice (Lab 02)

---

## Teoriöversikt: Zero Standing Privilege

Traditionellt har adminanvändare **permanent aktiva** adminroller – de är alltid Global Administrator, dygnet runt. Det innebär att om kontot komprometteras finns det inget att stjäla utanför ett aktiveringsfönster.

PIM introducerar konceptet **Eligible** – användaren *kan* aktivera rollen men är inte aktiv admin förrän de explicit begär det:

```
Utan PIM: Angripare stjäl lösenord → Permanent Global Admin-åtkomst
Med PIM:  Angripare stjäl lösenord → Eligible-tilldelning utan aktiv roll
          → Måste fortfarande aktivera via MFA + motivering + godkännande
```

**Eligible vs Active:**
| | Eligible | Active |
|--|----------|--------|
| Har adminåtkomst direkt | Nej | Ja |
| Måste aktivera | Ja | Nej |
| Kräver MFA vid aktivering | Ja (konfigurerbart) | Nej |
| Tidsbegränsad | Ja (1–24h) | Kan vara permanent |
| Rekommenderat för | Alla adminroller | Undantagsfall |

---

## Implementation

### 1. Tilldela Alice som Eligible för Global Administrator

```
PIM → Microsoft Entra roles → Roles → Global Administrator → + Add assignments
```

**Membership-fliken:**

| Inställning | Värde |
|-------------|-------|
| Select role | Global Administrator |
| Select member | Alice Andersson |
| Scope type | Directory |

**Setting-fliken:**

| Inställning | Värde |
|-------------|-------|
| Assignment type | Eligible |
| Permanently eligible | Ja |

> **Varför Permanently eligible?** I en verklig miljö skulle man sätta ett utgångsdatum (t.ex. 1 år) för att tvinga fram regelbunden omprövning. För labbet används permanent eligible för enkelhetens skull.

### 2. Konfigurera Role Settings för Global Administrator

```
PIM → Microsoft Entra roles → Settings → Global Administrator → Edit
```

| Inställning | Värde | Motivering |
|-------------|-------|------------|
| Activation maximum duration | 4 timmar | Begränsar exponeringstiden vid aktivering |
| On activation, require | Azure MFA | Kräver MFA även om användaren redan är inloggad |
| Require justification | Yes | Tvingar fram dokumentation av varför rollen aktiverades |
| Require approval to activate | Yes | Lägger till ett mänskligt kontrollsteg |
| Approvers | Admin (1 member) | Admin godkänner alla aktiveringar |

> **Varför 4 timmar?** En typisk arbetsuppgift som kräver Global Administrator tar sällan mer än några timmar. 8 timmar (standardvärdet) är onödigt länge – om uppgiften är klar ska rollen deaktiveras. 4 timmar är en bra balans mellan praktisk användbarhet och säkerhet.

### 3. Testa aktiveringsflödet som Alice

Loggade in som Alice på **entra.microsoft.com** i ett privat webbläsarfönster.

```
PIM → My roles → Eligible assignments → Global Administrator → Activate
```

Alice fyllde i:
- **Motivering:** (beskrivning av varför rollen behövs)
- **Duration:** upp till 4 timmar

Status efter begäran: **"Pending approval"** – Alice inväntar godkännande från admin.

### 4. Godkänna begäran som admin

```
PIM → Microsoft Entra roles → Approve requests
```

Admin såg Alices begäran, granskade motiveringen och godkände den. Alice fick omedelbart tillfällig Global Administrator-åtkomst.

### 5. Verifiera i Resource audit

```
PIM → Microsoft Entra roles → Resource audit
```

Hela flödet loggades med fullständig revisionsspårning:

| Tid | Requestor | Action | Target | Status |
|-----|-----------|--------|--------|--------|
| 12:39 | Johan Lab | Add eligible member to role in PIM requested | Alice Andersson | ✅ |
| 12:39 | Johan Lab | Add eligible member to role in PIM completed | Alice Andersson | ✅ |
| 12:42 | Johan Lab | Update role setting in PIM | Global Administrator | ✅ |
| 12:47 | Alice Andersson | Add member to role requested (PIM activation) | Global Administrator | ✅ |
| 12:47 | Alice Andersson | Add member to role approval requested | Global Administrator | ✅ |
| 12:49 | Johan Lab | Add member to role request approved | Alice Andersson | ✅ |
| 12:49 | Alice Andersson | Add member to role completed (PIM activation) | Global Administrator | ✅ |

---

## Verifiering – sammanfattning

| Kontroll | Status |
|----------|--------|
| Alice tilldelad som Eligible för Global Administrator | ✅ |
| Role settings konfigurerade (4h, MFA, godkännande) | ✅ |
| Alice begärde aktivering via PIM | ✅ |
| Admin godkände begäran | ✅ |
| Aktivering loggad i Resource audit | ✅ |

---

## Säkerhetsreflektioner

**Zero Standing Privilege eliminerar den största risken:** Med PIM finns det inget permanent privilegierat konto att stjäla. Även om en angripare komprometterar Alices konto har de bara en Eligible-tilldelning – de kan inte aktivera rollen utan att passera MFA-kravet och vänta på admin-godkännande.

**Audit trail är obligatoriskt för compliance:** Varje PIM-aktivering loggas med vem som begärde, när, varför och vem som godkände. Det är exakt den revisionsspårning som krävs för NIS2, ISO 27001 och SOC 2-compliance.

**Godkännandeflödet är ett komplement, inte ett hinder:** I verkligheten kan godkännare vara tillgängliga via Microsoft Authenticator-appen och svara på sekunder. Det är en minimal fördröjning för en stor säkerhetsvinst.

**Break-glass är fortfarande kritiskt:** Break-glass-kontot (Lab 01) ska aldrig hanteras via PIM – det ska ha permanent Global Administrator för att fungera i nödfall när PIM-infrastrukturen kanske inte är tillgänglig.

---

## Nästa steg

➡️ **Lab 07** – Identity Protection och riskbaserade policies
➡️ **Lab 08** – Entitlement Management och Access Packages

---

## Referenser

- [Vad är Privileged Identity Management?](https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/pim-configure)
- [Zero Standing Privilege](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/best-practices)
- [PIM – Godkännandeflöden](https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/pim-approval-workflow)
