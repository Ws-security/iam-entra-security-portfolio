# Lab 01 – Microsoft Entra ID Tenant Setup & Identity Foundation

**Datum:** Mars 2026  
**Miljö:** Microsoft 365 Business Premium Trial + Microsoft Entra ID P2 Trial  
**Tenant:** entrajohanlabb.onmicrosoft.com  
**Svårighetsgrad:** Grundläggande  
**Tidsåtgång:** ~2 timmar

---

## Scenario

Det här labbet sätter upp den labbmiljö som används i alla kommande labb. Tenant, licenser, testanvändare, grupper och break-glass-konto. Det simulerar vad en IAM-konsult gör vid onboarding av en ny M365-kund.

---

## Mål

- Skapa en Microsoft 365-tenant med Entra ID P2-licenser
- Etablera ett break-glass-konto för nödåtkomst
- Skapa testanvändare med roller enligt Least Privilege
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

Navigerade till Microsoft 365 Business Premium trial-sidan och skapade en ny tenant. Valde att skapa ett nytt `.onmicrosoft.com`-domännamn istället för att använda ett befintligt Microsoft-konto. Admin center kräver ett organisationskonto, inte ett personligt konto.

**Tenant-detaljer:**
- Domän: `entrajohanlabb.onmicrosoft.com`
- Admin-konto: `admin@<tenant>.onmicrosoft.com`
- Licenser: 25 x Microsoft 365 Business Premium

### 2. Aktivera Microsoft Entra ID P2 Trial

```
Entra admin center → Fakturering → Licenser → Alla produkter → + Testa/köp
→ Microsoft Entra ID P2 → Kostnadsfri utvärderingsversion → Aktivera
```

P2 låser upp Conditional Access, PIM och Identity Protection, som används i kommande labb.

### 3. Skapa break-glass-konto

Break-glass är ett nödkonto för om alla andra adminbehörigheter förloras, till exempel om en felkonfigurerad CA-policy låser ut alla administratörer.

```
Entra admin center → Identitet → Användare → + Ny användare → Skapa ny användare
```

| Inställning | Värde |
|-------------|-------|
| UPN | breakglass@entrajohanlabb.onmicrosoft.com |
| Roll | Global Administrator (permanent) |
| Licens | Ingen |
| MFA | Ej registrerat |

> ⚠️ Break-glass-kontot ska alltid exkluderas från samtliga CA-policies. Lösenordet förvaras offline och används enbart i nödfall.

### 4. Skapa testanvändare

Fyra testanvändare med roller tilldelade enligt Least Privilege.

| Användare | UPN | Entra-roll | Syfte |
|-----------|-----|------------|-------|
| Alice Andersson | alice@entrajohanlabb.onmicrosoft.com | User Administrator | Testar PIM och admin-flöden |
| Bob Bengtsson | bob@entrajohanlabb.onmicrosoft.com | Ingen | Testar CA-policies ur användarperspektiv |
| Charlie Carlsson | charlie@entrajohanlabb.onmicrosoft.com | Helpdesk Administrator, Authentication Administrator | Testar delegerad administration |
| Eve Eriksson | eve@entrajohanlabb.onmicrosoft.com | Ingen | Testar Access Packages och Access Reviews |

> Alice tilldelas Global Administrator som Eligible via PIM i Lab 06, inte som permanent roll.

Alla testanvändare tilldelades Microsoft 365 Business Premium och Entra ID P2.

### 5. Skapa säkerhetsgrupper

```
Entra admin center → Identitet → Grupper → + Ny grupp
```

| Grupp | Typ | Entra-roller kan tilldelas | Medlemmar |
|-------|-----|---------------------------|-----------|
| IT-Admins | Säkerhet | Ja | Alice |
| Finance-Users | Säkerhet | Nej | Bob, Eve |
| External-Guests | Säkerhet | Nej | Diana läggs till i Lab 05 |

IT-Admins är roll-tilldelningsbar eftersom gruppen ska kunna användas med PIM för just-in-time adminåtkomst.

---

## Verifiering

**Licensöversikt**  
`Entra admin center → Fakturering → Licenser → Alla produkter`  
Microsoft 365 Business Premium (5/25 tilldelade) och Entra ID P2 (5/25 tilldelade). ✅

**Inloggningstest**  
Loggade in som Bob i privat webbläsarfönster på `myapps.microsoft.com`. Bob nådde app-portalen och fick uppmaning att byta lösenord vid första inloggning. ✅

**Rollverifiering**  
`Entra admin center → Identitet → Användare → [Alice] → Tilldelade roller`  
User Administrator ✓

`Entra admin center → Identitet → Användare → [Charlie] → Tilldelade roller`  
Helpdesk Administrator ✓, Authentication Administrator ✓ ✅

**Break-glass**  
Kontot har Global Administrator och ingen licens. Redo att exkluderas från CA-policies i Lab 04. ✅

---

## Troubleshooting

### Microsoft 365 Developer Program kräver Visual Studio-prenumeration
**Symptom:** developer.microsoft.com/microsoft-365/dev-program visade att ett kvalificerande program krävs.  
**Lösning:** Använde Microsoft 365 Business Premium Trial istället, som ger samma Entra ID-funktionalitet.

### Kunde inte logga in på admin.microsoft.com
**Symptom:** Inloggning med personligt Outlook-konto gav felmeddelandet "Du kan inte logga in här med ett personligt konto."  
**Orsak:** Admin center kräver ett organisationskonto (`.onmicrosoft.com`).  
**Lösning:** Öppnade privat webbläsarfönster och loggade in med `admin@<tenant>.onmicrosoft.com`.

### Entra admin center visade "Azure AD roles" i dokumentation
**Symptom:** Äldre dokumentation refererar till "Azure AD roles can be assigned" men portalen visar "Microsoft Entra roles can be assigned".  
**Förklaring:** Microsoft bytte namn från Azure Active Directory till Microsoft Entra ID 2023. Funktionaliteten är identisk.

---

## Reflektioner

Det enda som inte var självklart var att break-glass-kontot måste skapas innan CA-policies sätts upp. Gör man det i fel ordning och en policy är felpkonfigurerad finns det inget sätt att ta sig in igen utan Microsofts support. Rätt ordning: break-glass först, CA-policies sedan.

Trials löper ut i april 2026. Påminnelser är satta.

---

## Referenser

- [Microsoft Entra ID dokumentation](https://learn.microsoft.com/en-us/entra/identity/)
- [Hantera nödåtkomstkonton i Microsoft Entra ID](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/security-emergency-access)
- [Microsoft Entra ID P2-licenser](https://learn.microsoft.com/en-us/entra/id-governance/licensing-fundamentals)
