# Lab 04 – Conditional Access: Zero Trust i praktiken

**Datum:** Mars 2026  
**Miljö:** Microsoft 365 Business Premium + Microsoft Entra ID P2  
**Tenant:** entrajohanlabb.onmicrosoft.com  
**Svårighetsgrad:** Medel  
**Tidsåtgång:** ~2 timmar

---

## Scenario

Det här labbet sätter upp tre CA-policies som utgör en grundläggande Zero Trust-baseline: MFA för alla, geografisk blockering och phishing-resistant MFA för adminroller. Det är samma konfiguration som en IAM-konsult implementerar vid säkerhetsgenomgångar.

---

## Mål

- Inaktivera Security Defaults och aktivera Conditional Access
- Skapa en policy som kräver MFA för alla användare
- Skapa Named Location och blockera inloggningar utanför Sverige
- Skapa en policy med Phishing-resistant MFA för adminroller
- Verifiera alla policies med What If och Sign-in logs

---

## Förutsättningar

- Slutfört Lab 01–03
- Break-glass-konto konfigurerat
- Entra ID P2-licenser tilldelade

---

## Hur Conditional Access fungerar

CA utvärderar varje inloggningsförsök mot en uppsättning regler och bestämmer: tillåt, kräv MFA eller blockera. Viktigt att förstå: att "aktivera MFA" i Authentication methods räcker inte. Utan CA-policies kan applikationer fortfarande utfärda tokens utan MFA-krav, vilket Lab 03 visade via `amr: pwd` i token-claims.

---

## Implementation

### Steg 1: Inaktivera Security Defaults

Security Defaults och Conditional Access kan inte vara aktiva samtidigt.

```
Entra admin center → Identity → Overview → Properties → Manage security defaults
→ Security defaults: Disabled
→ Reason: "My organization is planning to use Conditional Access"
```

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

> ⚠️ Break-glass-kontot måste alltid exkluderas. En felkonfigurerad policy utan exkludering kan innebära total utestängning från tenanten.

**Verifiering med What If:**
- User: Bob Bengtsson, Cloud app: Exchange Online, Device: Windows, Client: Browser
- Resultat: REQUIRE-MFA-ALL-USERS tillämpas ✅

**Verifiering i Sign-in logs:**
Loggade in som Bob i privat webbläsarfönster. MFA-prompt visades. Sign-in logs bekräftade att policyn tillämpades. ✅

---

### Steg 2: Skapa Named Location

```
Conditional Access → Named locations → + Countries location
```

| Inställning | Värde |
|-------------|-------|
| Name | Allowed-Countries |
| Country lookup method | Determine location by IP address (IPv4 and IPv6) |
| Länder | Sweden |

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
- User: Bob, IP: `<REDACTED-RU-IP>`, Country: Russia
- Resultat: BLOCK-SIGNIN-OUTSIDE-SWEDEN tillämpas med Grant: Block ✅

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

Policyn är satt till Report-only tills adminanvändare registrerat FIDO2 eller Windows Hello. Aktiveras den på On utan att admins är redo låser man ut alla admins.

Phishing-resistant MFA kräver för adminroller specifikt eftersom vanlig MFA (push-notis) är sårbart för MFA fatigue och adversary-in-the-middle-attacker där tokens fångas upp i realtid. FIDO2 och Windows Hello är kryptografiskt bundna till enhet och domän, vilket stoppar dessa angreppstyper.

---

## Verifiering

| Policy | Verifiering | Status |
|--------|-------------|--------|
| REQUIRE-MFA-ALL-USERS | What If + inloggningstest + Sign-in logs | ✅ On |
| BLOCK-SIGNIN-OUTSIDE-SWEDEN | What If med rysk IP | ✅ On |
| REQUIRE-COMPLIANT-DEVICE-ADMINS | Konfigurerad och granskad | ✅ Report-only |

---

## Troubleshooting

### "Security defaults must be disabled to enable Conditional Access policy"
**Orsak:** Security Defaults och Conditional Access är ömsesidigt uteslutande.  
**Lösning:** Inaktivera Security Defaults under Identity → Overview → Properties.

### What If kräver att IP och land matchar
**Orsak:** What If-verktyget validerar att IP-adressen faktiskt tillhör det angivna landet.  
**Lösning:** Använd en IP-adress som tillhör det land du vill testa.

---

## Reflektioner

Report-only-läget är verkligen användbart. Jag körde policyn i Report-only i 30 minuter och tittade i Sign-in logs för att se exakt vilka inloggningar som hade blockerats eller fått MFA-krav. Det gav full insyn innan policyn sattes till On.

Geoblockering är inte en primär kontroll. En VPN eller proxy i Sverige kringgår den. Den tar bort en stor del av automatiserade attacker, men ska kombineras med MFA och Identity Protection för att ha verkligt värde.

---

## Referenser

- [Vad är Conditional Access?](https://learn.microsoft.com/en-us/entra/identity/conditional-access/overview)
- [Conditional Access – Best practices](https://learn.microsoft.com/en-us/entra/identity/conditional-access/best-practices)
- [Phishing-resistant MFA](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-authentication-strengths)
- [Named locations i Conditional Access](https://learn.microsoft.com/en-us/entra/identity/conditional-access/location-condition)
