# Lab 01 – Microsoft Entra ID Tenant Setup & Identity Foundation

**Datum:** Mars 2026  
**Miljö:** Microsoft 365 Business Premium Trial + Microsoft Entra ID P2 Trial  
**Tenant:** entrajohanlabb.onmicrosoft.com  
**Svårighetsgrad:** Grundläggande  
**Tidsåtgång:** ~2 timmar

---

## Scenario

Innan man kan arbeta med Identity & Access Management i Microsoft Entra ID behöver man en fungerande tenant med rätt licenser och en kontrollerad användarstruktur. I enterprise-miljöer är detta grunden för allt IAM-arbete – utan en korrekt uppsatt tenant kan man inte konfigurera Conditional Access, PIM eller Identity Protection.

Det här labbet simulerar den initiala uppsättningen av en ny Microsoft 365-miljö, som en IAM-konsult typiskt utför vid onboarding av en ny kund.

---

## Mål

- Skapa en Microsoft 365-tenant med Entra ID P2-licenser
- Etablera ett break-glass-konto för nödåtkomst
- Skapa testanvändare med rätt roller enligt principen om minsta möjliga behörighet (Least Privilege)
- Skapa säkerhetsgrupper som speglar en verklig organisationsstruktur

---

## Förutsättningar

- Ett Microsoft-konto (Outlook/Hotmail)
- Microsoft 365 Business Premium Trial (gratis, 30 dagar)
- Microsoft Entra ID P2 Trial (gratis, 30 dagar)

> **OBS:** Microsoft 365 Developer Program kräver numera en aktiv Visual Studio Professional/Enterprise-prenumeration. Business Premium Trial är det enklaste alternativet för labbmiljöer.

---

## Implementation

### 1. Skapa tenant via Microsoft 365 Business Premium Trial

Navigerade till Microsoft 365 Business Premium trial-sidan och skapade en ny tenant. Under registreringen valde jag att skapa ett nytt `.onmicrosoft.com`-domännamn istället för att använda ett befintligt Microsoft-konto – detta är kritiskt eftersom admin center kräver ett organisationskonto, inte ett personligt konto.

**Tenant-detaljer:**
- Domän: `entrajohanlabb.onmicrosoft.com`
- Admin-konto: `admin@entrajohanlabb.onmicrosoft.com`
- Licenser: 25 x Microsoft 365 Business Premium

### 2. Aktivera Microsoft Entra ID P2 Trial

```
Entra admin center → Fakturering → Licenser → Alla produkter → + Testa/köp
→ Microsoft Entra ID P2 → Kostnadsfri utvärderingsversion → Aktivera
```

P2-licensen låser upp följande funktioner som används i kommande labb:
- **Conditional Access** – policystyrd åtkomstkontroll
- **Privileged Identity Management (PIM)** – just-in-time adminåtkomst
- **Identity Protection** – riskbaserad autentisering

### 3. Skapa break-glass-konto

Break-glass är ett nödkonto som används om alla andra adminbehörigheter förloras – till exempel om en felkonfigurerad Conditional Access-policy låser ut alla administratörer.

```
Entra admin center → Identitet → Användare → + Ny användare → Skapa ny användare
```

| Inställning | Värde |
|-------------|-------|
| UPN | breakglass@entrajohanlabb.onmicrosoft.com |
| Roll | Global Administrator (permanent) |
| Licens | Ingen (nödkonto behöver ej Office-appar) |
| MFA | Ej registrerat (nödåtkomst måste fungera utan MFA) |

> ⚠️ **Säkerhetsprincip:** Break-glass-kontot ska alltid exkluderas från samtliga Conditional Access-policies. Lösenordet förvaras offline och används enbart i nödfall.

### 4. Skapa testanvändare

Fyra testanvändare skapades för att representera olika roller i organisationen. Rollerna är tilldelade enligt principen om minsta möjliga behörighet.

| Användare | UPN | Entra-roll | Syfte |
|-----------|-----|------------|-------|
| Alice Andersson | alice@entrajohanlabb.onmicrosoft.com | User Administrator | Privilegierad användare – testar PIM och admin-flöden |
| Bob Bengtsson | bob@entrajohanlabb.onmicrosoft.com | Ingen | Vanlig slutanvändare – testar CA-policies ur användarperspektiv |
| Charlie Carlsson | charlie@entrajohanlabb.onmicrosoft.com | Helpdesk Administrator, Authentication Administrator | Junior IT – testar delegerad administration och behörighetsgränser |
| Eve Eriksson | eve@entrajohanlabb.onmicrosoft.com | Ingen | Finance-användare – testar Access Packages och Access Reviews |

> **Notering om Alice:** Alice tilldelas senare rollen Global Administrator som *Eligible* via PIM (inte permanent) – detta konfigureras i Lab 03.

Alla testanvändare tilldelades **Microsoft 365 Business Premium**- och **Entra ID P2**-licenser.

### 5. Skapa säkerhetsgrupper

```
Entra admin center → Identitet → Grupper → + Ny grupp
```

| Grupp | Typ | Microsoft Entra-roller kan tilldelas | Medlemmar |
|-------|-----|--------------------------------------|-----------|
| IT-Admins | Säkerhet | Ja | Alice |
| Finance-Users | Säkerhet | Nej | Bob, Eve |
| External-Guests | Säkerhet | Nej | (Diana läggs till i Lab 04 – B2B) |

> **Varför "Microsoft Entra roles can be assigned: Yes" för IT-Admins?**  
> En roll-tilldelningsbar grupp kan användas i PIM för att ge just-in-time adminåtkomst till hela gruppen, istället för att hantera varje individ separat. Detta är best practice i enterprise-miljöer med många adminanvändare.

---

## Verifiering

Verifierade att miljön är korrekt uppsatt genom följande kontroller:

**✅ Kontroll 1 – Licensöversikt**  
`Entra admin center → Fakturering → Licenser → Alla produkter`  
Visar: Microsoft 365 Business Premium (5/25 tilldelade) och Microsoft Entra ID P2 (5/25 tilldelade)

**✅ Kontroll 2 – Inloggningstest**  
Loggade in som Bob i ett privat webbläsarfönster på `myapps.microsoft.com`. Bob nådde sin app-portal utan problem och fick uppmaning att byta lösenord vid första inloggning (förväntat beteende).

**✅ Kontroll 3 – Rollverifiering**  
`Entra admin center → Identitet → Användare → [Alice] → Tilldelade roller`  
Visar: User Administrator ✓

`Entra admin center → Identitet → Användare → [Charlie] → Tilldelade roller`  
Visar: Helpdesk Administrator ✓, Authentication Administrator ✓

**✅ Kontroll 4 – Break-glass exkludering**  
Break-glass-kontot har Global Administrator och ingen licens. Kontot är redo att exkluderas från Conditional Access-policies i Lab 03.

---

## Troubleshooting

Under labbets genomförande uppstod följande problem och lösningar:

### Problem 1: Microsoft 365 Developer Program kräver Visual Studio-prenumeration
**Symptom:** developer.microsoft.com/microsoft-365/dev-program visade att ett kvalificerande program krävs.  
**Lösning:** Använde Microsoft 365 Business Premium Trial istället, vilket ger samma Entra ID-funktionalitet för labbändamål.

### Problem 2: Kunde inte logga in på admin.microsoft.com
**Symptom:** Inloggning med personligt outlook-konto gav felmeddelandet "Du kan inte logga in här med ett personligt konto."  
**Orsak:** Admin center kräver ett organisationskonto (`.onmicrosoft.com`), inte ett personligt Microsoft-konto.  
**Lösning:** Öppnade privat webbläsarfönster och loggade in med `admin@entrajohanlabb.onmicrosoft.com`.

### Problem 3: Entra admin center visade "Azure AD roles" i dokumentation
**Symptom:** Äldre dokumentation refererar till "Azure AD roles can be assigned" men portalen visar "Microsoft Entra roles can be assigned".  
**Förklaring:** Microsoft bytte namn från Azure Active Directory till Microsoft Entra ID 2023. Funktionaliteten är identisk – bara namngivningen har ändrats.

---

## Säkerhetsreflektioner

**Least Privilege:** Ingen testanvändare fick fler behörigheter än vad som krävs för sitt syfte. Bob och Eve har inga Entra-roller alls – de är representativa för majoriteten av användare i en verklig organisation.

**Break-glass:** Utan ett break-glass-konto riskerar man total utestängning vid felkonfigurerade Conditional Access-policies. Detta är en av de vanligaste misstagen vid initial Entra-konfiguration och ett kritiskt steg som alltid bör utföras **innan** CA-policies aktiveras.

**Trial-hantering:** Båda trials (M365 Business Premium och Entra ID P2) löper ut i april 2026. Påminnelser är satta för att avsluta prenumerationerna i god tid för att undvika oavsiktlig fakturering.

---

## Nästa steg

➡️ **Lab 02** – Konfigurera MFA, SSPR och Authentication Methods  
➡️ **Lab 03** – Conditional Access policies (Zero Trust)  
➡️ **Lab 04** – B2B Gäståtkomst och External Identities

---

## Referenser

- [Microsoft Entra ID dokumentation](https://learn.microsoft.com/en-us/entra/identity/)
- [Hantera nödåtkomstkonton i Microsoft Entra ID](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/security-emergency-access)
- [Microsoft Entra ID P2-licenser](https://learn.microsoft.com/en-us/entra/id-governance/licensing-fundamentals)
