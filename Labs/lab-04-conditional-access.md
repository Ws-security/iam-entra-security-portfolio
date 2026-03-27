# Lab 04 – Conditional Access: Zero Trust i praktiken

**Datum:** Mars 2026
**Miljö:** Microsoft 365 Business Premium + Microsoft Entra ID P2
**Tenant:** entrajohanlabb.onmicrosoft.com
**Svårighetsgrad:** Medel
**Tidsåtgång:** ~2 timmar

---

## Scenario

Conditional Access är kärnan i en modern Zero Trust-strategi. Istället för att lita på att en användare är säker bara för att de befinner sig innanför nätverket, utvärderar Conditional Access varje inloggningsförsök dynamiskt – vem är användaren, varifrån loggar de in, vilken enhet använder de, och vilken risk är kopplad till inloggningen?

I det här labbet konfigureras tre CA-policies som tillsammans utgör en grundläggande Zero Trust-baseline för en organisation. Det är exakt den typ av konfiguration en IAM-konsult implementerar vid säkerhetsgenomgångar hos kunder.

---

## Mål

- Inaktivera Security Defaults och aktivera Conditional Access
- Skapa en policy som kräver MFA för alla användare
- Skapa Named Location och blockera inloggningar utanför Sverige
- Skapa en policy med Phishing-resistant MFA för adminroller
- Verifiera alla policies med What If-verktyget och Sign-in logs

---

## Förutsättningar

- Slutfört Lab 01–03
- Break-glass-konto konfigurerat och redo att exkluderas
- Entra ID P2-licenser tilldelade

---

## Teoriöversikt: Conditional Access som Zero Trust-motor

Conditional Access fungerar som en intelligent beslutspunkt som utvärderar varje inloggningsförsök mot en uppsättning regler. Det är Zero Trust i praktiken – "lita aldrig, verifiera alltid".

```
Vem loggar in? + Varifrån? + Med vilken enhet? + Vilken risk? 
→ Tillåt / Kräv MFA / Blockera
```

En viktig insikt: det räcker inte att "aktivera MFA" i Entra. Utan CA-policies kan applikationer fortfarande utfärda tokens utan MFA-krav. CA enforcar MFA på policynivå, inte bara på användarnivå.

---

## Implementation

### Steg 1: Inaktivera Security Defaults

Security Defaults och Conditional Access kan inte vara aktiva samtidigt – Security Defaults är Microsofts förenklade säkerhetsinställningar för organisationer utan CA-licens.

```
Entra admin center → Identity → Overview → Properties → Manage security defaults
→ Security defaults: Disabled
→ Reason: "My organization is planning to use Conditional Access"
```

> **Varför:** Security Defaults ger grundläggande skydd men är inte konfigurerbart. CA ger granulär kontroll och är rätt val för alla organisationer med Entra ID P2.

---

### Policy 1: REQUIRE-MFA-ALL-USERS

```
Conditional Access → Policies → + New policy
```

| Inställning | Värde |
|-------------|-------|
| Name | REQUIRE-MFA-ALL-USERS |
| Users – Include | All users |
| Users – Exclude | breakglass@\<tenant\>.onmicrosoft.com, admin@\<tenant\>.onmicrosoft.com |
| Target resources | All cloud apps |
| Grant | Require multifactor authentication |
| Enable policy | Report-only → verifiering → On |

**Verifiering med What If:**
- User: Bob Bengtsson
- Cloud app: Office 365 Exchange Online
- Device platform: Windows
- Client app: Browser
- Resultat: REQUIRE-MFA-ALL-USERS tillämpas ✅

**Verifiering i Sign-in logs:**
Loggade in som Bob i privat webbläsarfönster. MFA-prompt visades korrekt. Sign-in logs visade att CA-policyn tillämpades med resultatet Success.

> ⚠️ **Break-glass är kritiskt:** Utan exkludering av break-glass-kontot riskerar man total utestängning om MFA-registreringen misslyckas för admin-kontot. Break-glass ska alltid exkluderas från ALLA CA-policies.

---

### Steg 2: Skapa Named Location – Allowed-Countries

```
Conditional Access → Named locations → + Countries location
```

| Inställning | Värde |
|-------------|-------|
| Name | Allowed-Countries |
| Country lookup method | Determine location by IP address (IPv4 and IPv6) |
| Länder | Sweden |

Named Locations används för att definiera betrodda geografiska områden. Inloggningar från länder utanför denna lista behandlas som potentiellt misstänkta.

---

### Policy 2: BLOCK-SIGNIN-OUTSIDE-SWEDEN

```
Conditional Access → Policies → + New policy
```

| Inställning | Värde |
|-------------|-------|
| Name | BLOCK-SIGNIN-OUTSIDE-SWEDEN |
| Users – Include | All users |
| Users – Exclude | breakglass@\<tenant\>.onmicrosoft.com, admin@\<tenant\>.onmicrosoft.com |
| Target resources | All cloud apps |
| Conditions – Locations – Include | Any location |
| Conditions – Locations – Exclude | Allowed-Countries |
| Grant | Block access |
| Enable policy | Report-only → verifiering → On |

**Verifiering med What If:**
- User: Bob Bengtsson
- Cloud app: Office 365 Exchange Online
- Device platform: Windows
- Client app: Browser
- IP address: `<REDACTED-RU-IP>`
- Country: Russia
- Resultat: BLOCK-SIGNIN-OUTSIDE-SWEDEN tillämpas med Grant: Block ✅

> **Notering:** What If-verktyget kräver att IP-adress och land matchar varandra. Verktyget validerar att IP-adressen faktiskt tillhör det angivna landet.

---

### Policy 3: REQUIRE-COMPLIANT-DEVICE-ADMINS

```
Conditional Access → Policies → + New policy
```

| Inställning | Värde |
|-------------|-------|
| Name | REQUIRE-COMPLIANT-DEVICE-ADMINS |
| Users – Include | Directory roles: Global Administrator, User Administrator, Helpdesk Administrator |
| Users – Exclude | breakglass@\<tenant\>.onmicrosoft.com, admin@\<tenant\>.onmicrosoft.com |
| Target resources | All cloud apps |
| Grant | Require authentication strength: Phishing-resistant MFA |
| Enable policy | Report-only |

**Varför Phishing-resistant MFA för adminroller?**

Vanlig MFA (SMS, push-notification) är sårbart för:
- **MFA fatigue attacks** – angripare spammar MFA-prompts tills användaren råkar acceptera
- **Adversary-in-the-middle (AiTM)** – angripare kan fånga upp MFA-tokens i realtid

Phishing-resistant MFA (FIDO2/Passkey, Windows Hello) eliminerar dessa attackvektorer eftersom autentiseringen är kryptografiskt bunden till den specifika enheten och domänen.

> **Varför Report-only?** Policyn kräver att adminanvändare registrerat FIDO2 eller Windows Hello innan den aktiveras – annars riskerar man att låsa ut alla admins. I en verklig miljö sätts den till On efter att alla admins genomfört registreringen.

---

## Verifiering – sammanfattning

| Policy | Verifiering | Status |
|--------|-------------|--------|
| REQUIRE-MFA-ALL-USERS | What If + inloggningstest + Sign-in logs | ✅ On |
| BLOCK-SIGNIN-OUTSIDE-SWEDEN | What If med rysk IP | ✅ On |
| REQUIRE-COMPLIANT-DEVICE-ADMINS | Konfigurerad och granskad | ✅ Report-only |

---

## Troubleshooting

### Problem: "Security defaults must be disabled to enable Conditional Access policy"
**Orsak:** Security Defaults och Conditional Access är ömsesidigt uteslutande.
**Lösning:** Inaktivera Security Defaults under Identity → Overview → Properties → Manage security defaults.

### Problem: What If kräver både IP och land
**Orsak:** What If-verktyget validerar att IP-adressen tillhör det angivna landet.
**Lösning:** Använd en IP-adress som faktiskt tillhör det land du vill testa.

---

## Säkerhetsreflektioner

**Report-only är inte valfritt – det är best practice:** Att aktivera en CA-policy direkt på On utan verifiering är ett vanligt misstag som kan låsa ut hela organisationen. Report-only låter dig se exakt vad policyn *skulle* göra mot verklig inloggningstrafik innan den aktiveras.

**Exkludering av break-glass är icke-förhandlingsbart:** Varje CA-policy ska exkludera break-glass-kontot. En felkonfigurerad policy utan exkludering kan innebära total utestängning från tenanten.

**Geoblockering är inte foolproof:** En angripare med tillgång till en VPN eller proxy i Sverige kan kringgå geografisk blockering. Geoblockering är ett komplement till andra kontroller, inte en primär säkerhetsmekanism.

**Phishing-resistant MFA för adminroller:** Adminroller är de mest attraktiva måltavlorna för angripare. Vanlig MFA räcker inte – phishing-resistant MFA eliminerar de vanligaste attackvektorerna mot privilegierade konton.

---

## Nästa steg

➡️ **Lab 05** – B2B Gäståtkomst och External Identities
➡️ **Lab 06** – Privileged Identity Management (PIM)
➡️ **Lab 07** – Identity Protection och riskbaserade policies

---

## Referenser

- [Vad är Conditional Access?](https://learn.microsoft.com/en-us/entra/identity/conditional-access/overview)
- [Conditional Access – Best practices](https://learn.microsoft.com/en-us/entra/identity/conditional-access/best-practices)
- [Phishing-resistant MFA](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-authentication-strengths)
- [Named locations i Conditional Access](https://learn.microsoft.com/en-us/entra/identity/conditional-access/location-condition)
