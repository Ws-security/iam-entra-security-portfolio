# Lab 02 – Authentication Methods: MFA, SSPR & Passwordless

**Datum:** Mars 2026  
**Miljö:** Microsoft 365 Business Premium + Microsoft Entra ID P2  
**Tenant:** entrajohanlabb.onmicrosoft.com  
**Svårighetsgrad:** Grundläggande/Medel  
**Tidsåtgång:** ~2 timmar

---

## Scenario

Det här labbet sätter upp autentiseringslagret i tenanten: MFA via Microsoft Authenticator för alla användare, SSPR för Finance-Users och passwordless inloggning för Alice. Det är samma konfiguration jag skulle göra vid onboarding av en ny M365-kund.

---

## Mål

- Aktivera MFA via Microsoft Authenticator för alla användare
- Konfigurera SSPR för Finance-Users-gruppen med två autentiseringsmetoder
- Aktivera och registrera passwordless inloggning för Alice
- Verifiera alla konfigurationer och granska Audit logs

---

## Förutsättningar

- Slutfört Lab 01 (tenant, användare och grupper konfigurerade)
- Microsoft Authenticator-appen installerad på mobiltelefon
- Microsoft Entra ID P2-licenser tilldelade till testanvändare

---

## Implementation

### 1. Aktivera MFA via Microsoft Authenticator

```
Entra admin center → Protection → Authentication methods → Policies → Microsoft Authenticator
```

| Inställning | Värde |
|-------------|-------|
| Enable | Yes |
| Target | All users |
| Authentication mode | Any (MFA + Passwordless) |

Valde Authenticator framför SMS av ett konkret skäl: Authenticator stödjer number matching, vilket gör att en användare måste mata in ett specifikt nummer från inloggningssidan för att godkänna. Det stoppar MFA fatigue-attacker där angripare skickar upprepade push-notiser och hoppas att användaren råkar trycka godkänn.

**Verifiering:** Loggade in som Alice i privat webbläsarfönster. MFA-prompt visades korrekt och krävde godkännande i Authenticator-appen. ✅

### 2. Aktivera SMS som SSPR-metod

```
Entra admin center → Protection → Authentication methods → Policies → SMS
```

| Inställning | Värde |
|-------------|-------|
| Enable | Yes |
| Target | All users |
| Use for sign-in | No |

SMS aktiverades enbart för SSPR, inte som inloggningsmetod. Det är ett aktivt val eftersom SMS som inloggningsmetod är svagare (sårbart för SIM-swapping) och inte bör användas när Authenticator finns tillgängligt.

**Notering:** Microsoft har centraliserat hanteringen av autentiseringsmetoder till Authentication methods → Policies. Password reset → Authentication methods visar numera enbart Security questions som legacy-alternativ och refererar till den centrala policyn för övriga metoder. Security questions fasas ut av Microsoft i mars 2027.

### 3. Konfigurera SSPR för Finance-Users

```
Entra admin center → Protection → Password reset → Properties
```

| Inställning | Värde |
|-------------|-------|
| Self-service password reset enabled | Selected (Finance-Users) |

```
Entra admin center → Protection → Password reset → Authentication methods
```

| Inställning | Värde |
|-------------|-------|
| Number of methods required to reset | 2 |

```
Entra admin center → Protection → Password reset → Registration
```

| Inställning | Värde |
|-------------|-------|
| Require users to register when signing in | Yes |
| Number of days before re-confirmation | 180 |

SSPR rullades ut till Finance-Users som en pilotgrupp. Två metoder krävs för återställning eftersom en enda metod (till exempel bara alternativ e-post) inte räcker om den kontot komprometteras.

### 4. Testa SSPR som Bob

Bob använde en extern privat e-postadress som registrerad SSPR-metod, vilket simulerar hur en vanlig användare registrerar en backup-adress.

**Testflöde:**
1. Navigerade till `aka.ms/sspr` i privat webbläsarfönster
2. Loggade in som `bob@entrajohanlabb.onmicrosoft.com`
3. Verifierade identitet via alternativ e-postadress
4. Återställde lösenordet utan admin-intervention

**Audit log-verifiering:**

```
Activity:     Self-service password reset flow activity progress
Date:         2026-03-26, 12:17
Status:       Success
Status reason: User successfully reset password
Initiated by: bob@entrajohanlabb.onmicrosoft.com
IP address:   <REDACTED>
Category:     UserManagement
```

### 5. Konfigurera Passwordless för Alice

```
Entra admin center → Protection → Authentication methods → Policies → Microsoft Authenticator
```

| Inställning | Värde |
|-------------|-------|
| Authentication mode | Passwordless |

**Användarregistrering:**
```
aka.ms/mysecurityinfo → + Lägg till inloggningsmetod → Microsoft Authenticator → Lösenordsfri inloggning
```

Alice registrerade passwordless phone sign-in i Authenticator-appen. Registreringen bekräftades i säkerhetsinformationssidan.

| Metod | Enhet | Status |
|-------|-------|--------|
| Microsoft Authenticator, lösenordsfri inloggning | [DEVICE REDACTED] | Registrerad ✅ |

> **Notering:** Passwordless-policyn kan ta 15–30 minuter att propagera. Inloggningsflödet kunde inte verifieras live under labbet, men registreringen bekräftades korrekt i `aka.ms/mysecurityinfo`.

---

## Verifiering

| Kontroll | Metod | Status |
|----------|-------|--------|
| MFA aktiverat för Alice | Inloggningstest i privat fönster | ✅ |
| SSPR aktiverat för Finance-Users | Inloggningstest som Bob via aka.ms/sspr | ✅ |
| SSPR loggat i Audit logs | Audit logs → UserManagement | ✅ |
| Passwordless registrerat för Alice | aka.ms/mysecurityinfo | ✅ |

---

## Troubleshooting

### SSPR visade bara Security questions som tillgänglig metod
**Orsak:** Microsoft har migrerat alla autentiseringsmetoder till den centrala Authentication methods-policyn. Password reset → Authentication methods visar inte längre Email och SMS som valbara alternativ.  
**Lösning:** Aktivera önskade metoder i Authentication methods → Policies. SSPR hämtar automatiskt tillgängliga metoder därifrån.

### Bob kunde inte använda SSPR direkt
**Orsak:** Bob hade inte registrerat några autentiseringsmetoder för SSPR.  
**Lösning:** Aktiverade "Require users to register when signing in" och lät Bob registrera en alternativ e-postadress via inloggningsflödet.

### Passwordless-inloggning visade fortfarande lösenordsfält
**Orsak:** Policy-propagation tar 15–30 minuter.  
**Lösning:** Vänta på propagation. Registreringen i Authenticator-appen bekräftades korrekt i `aka.ms/mysecurityinfo`.

---

## Reflektioner

Det som tog längst tid var att lista ut varför SSPR inte visade SMS som alternativ. Det beror på Microsofts migrering till centraliserade Authentication methods, vilket inte är tydligt dokumenterat i äldre guider. Värd att känna till inför kunduppdrag där miljön kan ha blandade policy-inställningar från olika epoker.

Propagationstiden för passwordless är också något att kommunicera till användare i verkligheten. Annars tror de att något är fel.

---

## Referenser

- [Microsoft Authenticator, number matching](https://learn.microsoft.com/en-us/entra/identity/authentication/how-to-mfa-number-match)
- [Konfigurera SSPR](https://learn.microsoft.com/en-us/entra/identity/authentication/tutorial-enable-sspr)
- [Passwordless authentication i Microsoft Entra](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-authentication-passwordless)
- [MFA fatigue attacks](https://learn.microsoft.com/en-us/entra/identity/authentication/how-to-mfa-additional-context)
