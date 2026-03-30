# Lab 06 – Privileged Identity Management (PIM)

**Datum:** Mars 2026  
**Miljö:** Microsoft 365 Business Premium + Microsoft Entra ID P2  
**Tenant:** \<tenant\>.onmicrosoft.com  
**Svårighetsgrad:** Medel/Avancerad  
**Tidsåtgång:** ~1.5 timmar

---

## Scenario

Det här labbet konfigurerar PIM för Global Administrator-rollen och kör hela aktiveringsflödet från begäran till godkännande. Alice tilldelas rollen som Eligible och aktiverar den sedan tillfälligt via ett godkänt flöde.

---

## Mål

- Tilldela Alice rollen Global Administrator som Eligible via PIM
- Konfigurera role settings med 4 timmars maxaktivering och krav på godkännande
- Testa hela aktiveringsflödet som Alice
- Godkänna begäran som admin och verifiera i Resource audit

---

## Förutsättningar

- Slutfört Lab 01–05
- Alice konfigurerad med Entra ID P2-licens och MFA

---

## Hur PIM fungerar

Utan PIM är adminroller permanenta. Om ett konto med Global Administrator komprometteras har angriparen full kontroll, dygnet runt, utan tidsbegränsning.

PIM gör roller Eligible istället för aktiva. Användaren kan aktivera rollen vid behov men är inte admin annars.

```
Utan PIM: Angripare stjäl lösenord → Permanent Global Admin-åtkomst
Med PIM:  Angripare stjäl lösenord → Eligible-tilldelning, ingen aktiv roll
          → Måste aktivera via MFA + motivering + godkännande
```

**Eligible vs Active:**

| | Eligible | Active |
|--|----------|--------|
| Har adminåtkomst direkt | Nej | Ja |
| Måste aktivera | Ja | Nej |
| Kräver MFA vid aktivering | Ja | Nej |
| Tidsbegränsad | Ja (1–24h) | Kan vara permanent |
| Rekommenderat för | Alla adminroller | Undantagsfall |

---

## Implementation

### 1. Tilldela Alice som Eligible för Global Administrator

```
PIM → Microsoft Entra roles → Roles → Global Administrator → + Add assignments
```

| Inställning | Värde |
|-------------|-------|
| Select role | Global Administrator |
| Select member | Alice Andersson |
| Scope type | Directory |
| Assignment type | Eligible |
| Permanently eligible | Ja |

I en riktig miljö sätter man ett utgångsdatum (t.ex. 1 år) för att tvinga regelbunden omprövning. För labbet används permanent eligible.

### 2. Konfigurera Role Settings för Global Administrator

```
PIM → Microsoft Entra roles → Settings → Global Administrator → Edit
```

| Inställning | Värde |
|-------------|-------|
| Activation maximum duration | 4 timmar |
| On activation, require | Azure MFA |
| Require justification | Yes |
| Require approval to activate | Yes |
| Approvers | Admin (1 member) |

4 timmar valdes eftersom en typisk admin-uppgift sällan tar längre tid. Standardvärdet är 8 timmar, vilket ger onödigt lång exponering om uppgiften är klar tidigare.

### 3. Testa aktiveringsflödet som Alice

Loggade in som Alice på entra.microsoft.com i privat webbläsarfönster.

```
PIM → My roles → Eligible assignments → Global Administrator → Activate
```

Alice fyllde i motivering och vald duration (upp till 4 timmar). Status efter begäran: "Pending approval".

### 4. Godkänna begäran som admin

```
PIM → Microsoft Entra roles → Approve requests
```

Granskade Alices begäran och motivering, godkände. Alice fick omedelbart tillfällig Global Administrator-åtkomst.

### 5. Verifiera i Resource audit

```
PIM → Microsoft Entra roles → Resource audit
```

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

## Verifiering

| Kontroll | Status |
|----------|--------|
| Alice tilldelad som Eligible för Global Administrator | ✅ |
| Role settings konfigurerade (4h, MFA, godkännande) | ✅ |
| Alice begärde aktivering via PIM | ✅ |
| Admin godkände begäran | ✅ |
| Aktivering loggad i Resource audit | ✅ |

---

## Troubleshooting

Inga problem uppstod under labbet. Flödet fungerade som förväntat.

---

## Reflektioner

Resource audit-loggen är det tydligaste exemplet på varför PIM är värdefullt utöver ren säkerhet. Varje aktivering är loggad med vem som begärde, när, varför och vem som godkände. Det är precis vad som krävs vid NIS2- och ISO 27001-revisioner.

Break-glass-kontot ska aldrig hanteras via PIM. Det ska ha permanent Global Administrator för att kunna användas i nödfall även om PIM-infrastrukturen av någon anledning inte är tillgänglig.

---

## Referenser

- [Vad är Privileged Identity Management?](https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/pim-configure)
- [Zero Standing Privilege](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/best-practices)
- [PIM – Godkännandeflöden](https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/pim-approval-workflow)
