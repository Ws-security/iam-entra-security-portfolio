# Lab 02 – Authentication Methods: MFA, SSPR & Passwordless

**Datum:** Mars 2026  
**Miljö:** Microsoft 365 Business Premium + Microsoft Entra ID P2  
**Tenant:** entrajohanlabb.onmicrosoft.com  
**Svårighetsgrad:** Grundläggande–Medel  
**Tidsåtgång:** ~2 timmar

---

## Scenario

Svaga autentiseringsmetoder är en av de vanligaste attackvektorerna mot organisationer. Enligt Microsoft är komprometterade lösenord inblandade i majoriteten av identitetsrelaterade intrång. I det här labbet konfigureras tre lager av autentiseringsskydd som tillsammans utgör en modern autentiseringsbaseline:

1. **MFA** – förhindrar kontoövertagande även om lösenordet läckt
2. **SSPR** – minskar helpdesk-belastning och ger användare kontroll
3. **Passwordless** – eliminerar lösenordet som attackvektor helt

Detta är standardkonfiguration vid IAM-konsultuppdrag för Microsoft 365-miljöer.

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

Microsoft Authenticator valdes framför SMS och röstsamtal av säkerhetsskäl. SMS är sårbart för SIM-swapping och SS7-attacker, medan Authenticator-appen stödjer **number matching** och **additional context** – vilket motverkar MFA fatigue-attacker där angripare spammar godkännandeprompts i hopp om att användaren råkar acceptera.

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

> **Viktigt:** SMS aktiverades enbart som SSPR-metod, inte som inloggningsmetod. SMS som inloggningsmetod är säkerhetsmässigt svagare och rekommenderas inte i produktionsmiljöer.

**Notering:** Microsoft har centraliserat hanteringen av autentiseringsmetoder till Authentication methods → Policies. Password reset → Authentication methods visar numera enbart Security questions som legacy-alternativ, och refererar till den centrala policyn för övriga metoder. Security questions fasas ut av Microsoft i mars 2027.

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

**Varför Finance-Users specifikt?**  
I enterprise-miljöer rullas SSPR ofta ut gradvis – börja med en pilotgrupp, utvärdera, sedan bredda. Finance-avdelningen är ett typiskt pilotval eftersom de ofta hanterar känslig data och behöver snabb åtkomst utan att vara beroende av IT-helpdesk.

**Varför 2 metoder?**  
En enda metod (t.ex. bara email) är sårbart – om den komprometteras kan en angripare återställa lösenordet. Två metoder kräver att angriparen kontrollerar båda kanalerna simultant.

### 4. Testa SSPR som Bob

Bob använder en alternativ e-postadress (`bob.entralab@gmail.com`) som registrerad SSPR-metod, vilket simulerar hur användare i verkligheten registrerar en privat e-postadress som backup-autentisering.

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

✅ SSPR fungerar korrekt och loggas med fullständig revisionsspårning.

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

Alice registrerade passwordless phone sign-in i Microsoft Authenticator-appen på Samsung SM-F966B. Registreringen bekräftades i säkerhetsinformationssidan:

| Metod | Enhet | Status |
|-------|-------|--------|
| Microsoft Authenticator – Lösenordsfri inloggning | Samsung SM-F966B | Registrerad ✅ |

> **Notering:** Passwordless-policyn kan ta 15–30 minuter att propagera i systemet efter aktivering. Inloggningsflödet kunde inte verifieras live under labbet pga propagationstid, men registreringen bekräftades korrekt i `aka.ms/mysecurityinfo`.

---

## Säkerhetsreflektioner

**MFA fatigue:** Genom att använda Microsoft Authenticator med number matching elimineras risken för MFA fatigue-attacker. Angripare som spammar MFA-prompts kan inte längre lura användare att råksgodkänna – användaren måste matcha ett specifikt nummer som visas på inloggningssidan.

**Passwordless = eliminerad attackvektor:** När passwordless är fullt implementerat finns det inget lösenord att stjäla, phisha eller brute-forca. Det är den starkaste autentiseringsformen som är praktiskt genomförbar i enterprise-miljöer idag.

**SSPR och helpdesk-kostnad:** Lösenordsåterställningar är en av de vanligaste helpdesk-ärendena i stora organisationer. SSPR eliminerar denna kostnad och ger användare omedelbar självservice – särskilt värdefullt utanför kontorstid.

**Least privilege för SMS:** SMS aktiverades enbart för SSPR, inte som inloggningsmetod. Detta är ett medvetet säkerhetsbeslut – SMS som inloggningsmetod är svagare än Authenticator-appen och bör undvikas i miljöer med höga säkerhetskrav.

---

## Troubleshooting

### Problem: SSPR visade bara Security questions som tillgänglig metod
**Orsak:** Microsoft har migrerat alla autentiseringsmetoder till den centrala Authentication methods-policyn. Password reset → Authentication methods visar inte längre Email och SMS som valbara alternativ där.  
**Lösning:** Aktivera önskade metoder i Authentication methods → Policies. SSPR hämtar automatiskt tillgängliga metoder därifrån.

### Problem: Bob kunde inte använda SSPR direkt
**Orsak:** Bob hade inte registrerat några autentiseringsmetoder för SSPR.  
**Lösning:** Aktiverade "Require users to register when signing in" och lät Bob registrera en alternativ e-postadress via inloggningsflödet.

### Problem: Passwordless-inloggning visade fortfarande lösenordsfält
**Orsak:** Policy-propagation tar 15–30 minuter.  
**Lösning:** Vänta på propagation. Registreringen i Authenticator-appen bekräftades korrekt i `aka.ms/mysecurityinfo`.

---

## Verifiering – sammanfattning

| Kontroll | Metod | Status |
|----------|-------|--------|
| MFA aktiverat för Alice | Inloggningstest i privat fönster | ✅ |
| SSPR aktiverat för Finance-Users | Inloggningstest som Bob via aka.ms/sspr | ✅ |
| SSPR loggat i Audit logs | Audit logs → UserManagement | ✅ |
| Passwordless registrerat för Alice | aka.ms/mysecurityinfo | ✅ |

---

## Nästa steg

➡️ **Lab 03** – Conditional Access policies (Zero Trust)  
➡️ **Lab 04** – B2B Gäståtkomst och External Identities  
➡️ **Lab 05** – Privileged Identity Management (PIM)

---

## Referenser

- [Microsoft Authenticator – Number matching](https://learn.microsoft.com/en-us/entra/identity/authentication/how-to-mfa-number-match)
- [Konfigurera SSPR](https://learn.microsoft.com/en-us/entra/identity/authentication/tutorial-enable-sspr)
- [Passwordless authentication i Microsoft Entra](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-authentication-passwordless)
- [MFA fatigue attacks](https://learn.microsoft.com/en-us/entra/identity/authentication/how-to-mfa-additional-context)
