# Lab 05 – B2B Gäståtkomst & External Identities

**Datum:** Mars 2026
**Miljö:** Microsoft 365 Business Premium + Microsoft Entra ID P2
**Tenant:** \<tenant\>.onmicrosoft.com
**Svårighetsgrad:** Medel
**Tidsåtgång:** ~1.5 timmar

---

## Scenario

I moderna organisationer samarbetar anställda regelbundet med externa parter – konsulter, partners, leverantörer och kunder. Microsoft Entra ID B2B (Business-to-Business) gör det möjligt att bjuda in externa användare till din tenant utan att de behöver ett konto i din organisation.

En IAM-konsult behöver förstå hur B2B-åtkomst konfigureras säkert, hur gäståtkomst begränsas och hur organisationen säkerställer att externa användare inte har mer åtkomst än nödvändigt – och att den åtkomsten regelbundet granskas.

---

## Mål

- Konfigurera External Collaboration Settings enligt säker baseline
- Bjuda in en extern B2B-gästanvändare (Diana)
- Lägga till gästanvändaren i en dedikerad säkerhetsgrupp
- Konfigurera och starta en månadsvis Access Review för gästanvändare

---

## Förutsättningar

- Slutfört Lab 01–04
- Tillgång till ett externt e-postkonto (Gmail eller liknande)
- External-Guests-gruppen skapad i Lab 01

---

## Teoriöversikt: B2B och Least Privilege för externa användare

B2B-gäster autentiserar med sin egen identitetsleverantör (t.ex. Gmail, Microsoft-konto) men får åtkomst till resurser i din tenant. Deras UPN i Entra ID slutar med `#EXT#@<tenant>.onmicrosoft.com` – vilket indikerar att de är externa.

Tre viktiga principer för säker B2B-hantering:

1. **Begränsa gästers directory-åtkomst** – gäster ska inte kunna se hela organisationens användarkatalogt
2. **Begränsa vem som får bjuda in** – inte alla anställda bör ha rätt att bjuda in externa användare
3. **Regelbunden granskning** – Access Reviews säkerställer att gäster inte behåller åtkomst längre än nödvändigt

---

## Implementation

### 1. Konfigurera External Collaboration Settings

```
Entra admin center → External Identities → External collaboration settings
```

| Inställning | Valt värde | Motivering |
|-------------|------------|------------|
| Guest user access | Restricted to own directory objects | Gäster ska inte kunna enumera hela organisationens användare |
| Guest invite restrictions | Only users assigned to specific admin roles | Minimerar risken för okontrollerade gästinbjudningar |
| Enable guest self-service sign up | No | Förhindrar okontrollerad självregistrering |
| Allow external users to remove themselves | Yes | Rekommenderat – gäster kan avsluta åtkomst själva |
| Collaboration restrictions | Allow invitations to any domain | Öppet för labbet – i produktion bör detta begränsas |

> **Säkerhetsprincip:** Standardinställningen "Guest users have the same access as members" är alltför generös för de flesta organisationer. "Most restrictive" är rätt startpunkt och kan öppnas upp vid behov.

### 2. Bjuda in Diana som B2B-gäst

```
Entra admin center → Identity → Users → + Invite external user
```

| Inställning | Värde |
|-------------|-------|
| Display name | Diana Guest |
| Email | \<externt e-postkonto\> |

Diana fick ett inbjudningsmail från Microsoft med ämnesraden *"You're invited to the Entra Lab organization"*. Efter att inbjudan accepterades skapades ett gästkonto automatiskt i tenanten.

**Gästkontots UPN-format:**
```
diana_<extern-domän>#EXT#@<tenant>.onmicrosoft.com
```

Detta format indikerar att användaren är extern och autentiserar via sin egen identitetsleverantör.

**Verifiering:**
```
Entra admin center → Identity → Users → filtrera på User type: Guest
```
Diana visas med **User type: Guest** ✅

### 3. Lägga till Diana i External-Guests-gruppen

```
Identity → Groups → External-Guests → Members → + Add members → Diana Guest
```

Dedikerade gästgrupper är best practice eftersom de:
- Möjliggör gruppstyrd CA-policy specifikt för gäster
- Förenklar Access Reviews – en review per grupp istället för per användare
- Ger tydlig översikt över alla externa användare i organisationen

### 4. Konfigurera Access Review för gästanvändare

```
External Identities → Access reviews → + New access review → Resource review
```

**Reviews-inställningar:**

| Inställning | Värde |
|-------------|-------|
| Reviewers | Group owner(s) |
| Duration | 7 dagar |
| Review recurrence | Monthly |
| Start date | 2026-03-27 |
| End | Never |

**Settings-inställningar:**

| Inställning | Värde | Motivering |
|-------------|-------|------------|
| Auto apply results | Ja | Automatisk borttagning utan manuell åtgärd |
| If reviewers don't respond | Remove access | Säker default – ingen respons = ingen åtkomst |
| Action on denied guests | Remove user's membership | Tas bort från gruppen automatiskt |
| No sign-in within 30 days | Aktiverat | Flaggar inaktiva gäster för reviewer |
| Justification required | Ja | Reviewer måste motivera sitt beslut |

> **Varför "Remove access" om reviewers inte svarar?** I verkligheten är en reviewer som inte svarar lika problematiskt som en som aktivt nekar åtkomst. Säker default är att ta bort åtkomst och låta användaren begära den igen vid behov.

---

## Troubleshooting

### Problem 1: Access Review visade "Not started" och inget att granska
**Orsak:** Reviewen hade precis skapats och hade inte aktiverats ännu.
**Lösning:** Access Reviews tar tid att propagera. Granskning blir tillgänglig när reviewen startar aktivt.

### Problem 2: Inga reviews synliga i myaccess.microsoft.com
**Orsak:** External-Guests-gruppen saknade en owner. Reviewers var satt till "Group owner(s)" men ingen owner fanns tilldelad.
**Lösning:** Lade till admin-kontot som owner på External-Guests-gruppen under Identity → Groups → External-Guests → Owners.

> **Viktig lärdom:** Access Reviews med "Group owner(s)" som reviewer kräver att gruppen faktiskt har en owner – annars skickas inga review-notifikationer och ingen kan genomföra granskningen. I produktionsmiljöer ska alla grupper ha minst en utsedd owner.

### Problem 3: Azure-prenumeration krävs för Access Reviews för gäster
**Notering:** Microsoft meddelade i januari 2026 att Access Reviews för gästanvändare kräver en länkad Azure-prenumeration för fakturering. Detta är en förändring som påverkar labbmiljöer utan Azure-subscription.

---

## Verifiering – sammanfattning

| Kontroll | Status |
|----------|--------|
| External Collaboration Settings konfigurerade | ✅ |
| Diana inbjuden och accepterat B2B-inbjudan | ✅ |
| Diana visas som Guest i Entra ID | ✅ |
| Diana tillagd i External-Guests-gruppen | ✅ |
| Månadsvis Access Review konfigurerad | ✅ |
| Owner tillagd på External-Guests-gruppen | ✅ |

---

## Säkerhetsreflektioner

**B2B är inte samma sak som säker gäståtkomst:** Att bjuda in en extern användare skapar bara ett konto – det kontrollerar inte vad de kan komma åt. Gästers åtkomst bör alltid styras via grupper och CA-policies, inte via direkta rolltilldelningar.

**Access Reviews är kritiska för compliance:** Utan regelbunden granskning tenderar gästkonton att ackumuleras över tid. En extern konsult som slutade arbeta med organisationen för ett år sedan kan fortfarande ha aktiv åtkomst. Access Reviews med automatisk borttagning eliminerar detta problem.

**Geoblockering gäller även gäster:** CA-policyn BLOCK-SIGNIN-OUTSIDE-SWEDEN (Lab 04) gäller även Dianas konto. Om Diana loggar in från ett land utanför Sverige blockeras inloggningen. Detta är viktigt att kommunicera till externa användare.

---

## Nästa steg

➡️ **Lab 06** – Privileged Identity Management (PIM)
➡️ **Lab 07** – Identity Protection och riskbaserade policies
➡️ **Lab 08** – Entitlement Management och Access Packages

---

## Referenser

- [Microsoft Entra B2B – Översikt](https://learn.microsoft.com/en-us/entra/external-id/what-is-b2b)
- [External Collaboration Settings](https://learn.microsoft.com/en-us/entra/external-id/external-collaboration-settings-configure)
- [Access Reviews för gäster](https://learn.microsoft.com/en-us/entra/id-governance/access-reviews-overview)
