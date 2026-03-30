# Lab 05 – B2B Gäståtkomst & External Identities

**Datum:** Mars 2026  
**Miljö:** Microsoft 365 Business Premium + Microsoft Entra ID P2  
**Tenant:** \<tenant\>.onmicrosoft.com  
**Svårighetsgrad:** Medel  
**Tidsåtgång:** ~1.5 timmar

---

## Scenario

Det här labbet konfigurerar B2B-gäståtkomst och sätter upp en löpande Access Review för externa användare. Diana bjuds in som extern gästanvändare, placeras i External-Guests-gruppen och en månadsvis review konfigureras som automatiskt tar bort gäster om ingen ansvarar för dem.

---

## Mål

- Konfigurera External Collaboration Settings enligt säker baseline
- Bjuda in Diana som extern B2B-gästanvändare
- Lägga till Diana i External-Guests-gruppen
- Konfigurera och starta en månadsvis Access Review för gästanvändare

---

## Förutsättningar

- Slutfört Lab 01–04
- Tillgång till ett externt e-postkonto (Gmail eller liknande)
- External-Guests-gruppen skapad i Lab 01

---

## Hur B2B fungerar

B2B-gäster autentiserar med sin egen identitetsleverantör (Gmail, Microsoft-konto) men får åtkomst till resurser i tenanten. Deras UPN slutar med `#EXT#@<tenant>.onmicrosoft.com`.

---

## Implementation

### 1. Konfigurera External Collaboration Settings

```
Entra admin center → External Identities → External collaboration settings
```

| Inställning | Valt värde | Motivering |
|-------------|------------|------------|
| Guest user access | Restricted to own directory objects | Gäster ska inte kunna enumera hela katalogens användare |
| Guest invite restrictions | Only users assigned to specific admin roles | Minimerar okontrollerade gästinbjudningar |
| Enable guest self-service sign up | No | Förhindrar självregistrering |
| Allow external users to remove themselves | Yes | Gäster kan avsluta åtkomst själva |
| Collaboration restrictions | Allow invitations to any domain | Öppet för labbet |

### 2. Bjuda in Diana som B2B-gäst

```
Entra admin center → Identity → Users → + Invite external user
```

| Inställning | Värde |
|-------------|-------|
| Display name | Diana Guest |
| Email | \<externt e-postkonto\> |

Diana fick ett inbjudningsmail och skapades automatiskt som gästkonto i tenanten efter att inbjudan accepterades.

**Gästkontots UPN-format:**
```
diana_<extern-domän>#EXT#@<tenant>.onmicrosoft.com
```

**Verifiering:**
```
Entra admin center → Identity → Users → filtrera på User type: Guest
```
Diana visas med User type: Guest ✅

### 3. Lägga till Diana i External-Guests-gruppen

```
Identity → Groups → External-Guests → Members → + Add members → Diana Guest
```

Dedikerade gästgrupper gör det enklare att styra åtkomst via CA-policies och köra Access Reviews per grupp istället för per användare.

### 4. Konfigurera Access Review för gästanvändare

```
External Identities → Access reviews → + New access review → Resource review
```

| Inställning | Värde |
|-------------|-------|
| Reviewers | Group owner(s) |
| Duration | 7 dagar |
| Review recurrence | Monthly |
| Start date | 2026-03-27 |
| End | Never |

| Inställning | Värde |
|-------------|-------|
| Auto apply results | Ja |
| If reviewers don't respond | Remove access |
| Action on denied guests | Remove user's membership |
| No sign-in within 30 days | Aktiverat |
| Justification required | Ja |

"Remove access" som default om reviewern inte svarar är ett medvetet val. En reviewer som inte svarar ger ingen trygghet om att gästen faktiskt behöver åtkomsten. Det är enklare att låta användaren begära om den om den faktiskt behövs.

---

## Troubleshooting

### Access Review visade "Not started" och inget att granska
**Orsak:** Reviewen hade precis skapats och hade inte aktiverats ännu.  
**Lösning:** Access Reviews propagerar med fördröjning. Granskning blir tillgänglig när reviewen startar aktivt.

### Inga reviews synliga i myaccess.microsoft.com
**Orsak:** External-Guests-gruppen saknade en owner. Reviewers var satt till "Group owner(s)" men ingen owner fanns.  
**Lösning:** Lade till admin-kontot som owner under Identity → Groups → External-Guests → Owners.

### Azure-prenumeration krävs för Access Reviews för gäster
Microsoft meddelade i januari 2026 att Access Reviews för gästanvändare kräver en länkad Azure-prenumeration. Påverkar labbmiljöer utan Azure-subscription.

---

## Verifiering

| Kontroll | Status |
|----------|--------|
| External Collaboration Settings konfigurerade | ✅ |
| Diana inbjuden och accepterat B2B-inbjudan | ✅ |
| Diana visas som Guest i Entra ID | ✅ |
| Diana tillagd i External-Guests-gruppen | ✅ |
| Månadsvis Access Review konfigurerad | ✅ |
| Owner tillagd på External-Guests-gruppen | ✅ |

---

## Reflektioner

Felet med saknad group owner var oväntat. Det är ett av de fall där Entra inte ger ett tydligt felmeddelande, det ser bara ut som att ingenting händer. I produktion bör alla grupper ha minst en utsedd owner från start, annars fungerar inte Access Reviews med "Group owner(s)".

CA-policyn från Lab 04 gäller även Diana. Om hon loggar in från utanför Sverige blockeras hon. Det är värt att informera externa användare om om de arbetar från andra länder.

---

## Referenser

- [Microsoft Entra B2B – Översikt](https://learn.microsoft.com/en-us/entra/external-id/what-is-b2b)
- [External Collaboration Settings](https://learn.microsoft.com/en-us/entra/external-id/external-collaboration-settings-configure)
- [Access Reviews för gäster](https://learn.microsoft.com/en-us/entra/id-governance/access-reviews-overview)
