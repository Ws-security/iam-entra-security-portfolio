# Lab 08 – Entitlement Management & Access Packages

**Datum:** Mars 2026  
**Miljö:** Microsoft 365 Business Premium + Microsoft Entra ID P2  
**Tenant:** \<tenant\>.onmicrosoft.com  
**Svårighetsgrad:** Medel  
**Tidsåtgång:** ~1.5 timmar

---

## Scenario

Det här labbet konfigurerar Entitlement Management och kör ett fullständigt Access Package-flöde med intern testanvändare. Eve begär åtkomst till Finance-resurser via myaccess.microsoft.com och admin godkänner via Approvals-flödet.

---

## Mål

- Skapa en Catalog med resurser
- Skapa ett Access Package med godkännandeflöde och livscykelhantering
- Testa hela flödet från begäran till godkännande som Eve
- Verifiera i Assignments

---

## Förutsättningar

- Slutfört Lab 01–07
- Finance-Users-gruppen skapad i Lab 01
- Eve konfigurerad med Entra ID P2-licens

---

## Azure-prenumerationskrav

Från den 15 januari 2026 kräver Entitlement Management-funktioner för gästanvändare en länkad Azure-prenumeration. Labbet genomfördes med interna användare, vilket inte berörs av detta krav. Scenariot med externa gäster (Diana) dokumenteras men testades inte.

---

## Implementation

### 1. Skapa Catalog

```
Entra admin center → Identity Governance → Entitlement Management → Catalogs → + New catalog
```

| Inställning | Värde |
|-------------|-------|
| Name | Finance-Catalog |
| Description | Resurser för Finance-avdelningen |
| Enabled | Yes |
| Enabled for external users | No |

### 2. Lägg till resurser i Catalog

```
Finance-Catalog → Resources → + Add resources → Groups and Teams → Finance-Users
```

Finance-Users-gruppen lades till som resurs i cataloget.

### 3. Skapa Access Package

```
Finance-Catalog → Access packages → + New access package
```

**Basics:**

| Inställning | Värde |
|-------------|-------|
| Name | Finance-Team-Access |
| Catalog | Finance-Catalog |
| Description | Åtkomst till Finance-resurser |

**Resource roles:**

| Resurs | Roll |
|--------|------|
| Finance-Users | Member |

**Requests:**

| Inställning | Värde |
|-------------|-------|
| Who can get access | For users, service principals, and agent identities in your directory |
| Select specific scope | Specific users and groups: Finance-Users |
| Who can request access | Self + Admin |
| Require approval | Yes |
| Require requestor justification | Yes |
| First Approver | Johan Lab (specific approver) |

**Lifecycle:**

| Inställning | Värde |
|-------------|-------|
| Expiration | 90 days |
| Users can request specific timeline | No |
| Require access reviews | No |

### 4. Testa flödet som Eve

Loggade in som Eve i privat webbläsarfönster på `myaccess.microsoft.com`.

Finance-Team-Access var synligt under Tillgängligt (1). Eve klickade Begäran, valde "Mig själv" och fyllde i motivering. Begäran skickades med status **Pending approval**.

### 5. Godkänna som admin

```
myaccess.microsoft.com → Approvals → Eve Erikssons begäran → Approve
```

Eves begäran godkändes och åtkomsten aktiverades omedelbart.

---

## Troubleshooting

### "Det finns inga tillgängliga åtkomstpaket" i myaccess.microsoft.com
**Orsak 1:** Who can get access var satt till "None (administrator direct assignments only)" vid skapandet.  
**Lösning:** Finance-Team-Access → Policies → Edit → Who can get access: For users in your directory.

**Orsak 2:** "Self" var inte ikryssad under Who can request access.  
**Lösning:** Kryssa i Self i policy-inställningarna. Utan Self kan användare se paketet men inte begära det.

### Approve-knappen saknades i Requests-vyn
**Orsak:** Godkännande hanteras inte i admin-portalen utan i myaccess.microsoft.com.  
**Lösning:** Logga in som approver (Johan Lab) på myaccess.microsoft.com → Approvals.

---

## Verifiering

| Kontroll | Status |
|----------|--------|
| Finance-Catalog skapad | ✅ |
| Finance-Users tillagd som resurs i Catalog | ✅ |
| Finance-Team-Access Access Package skapat | ✅ |
| Eve begärde åtkomst via myaccess.microsoft.com | ✅ |
| Admin godkände via myaccess.microsoft.com Approvals | ✅ |
| Access Package för externa gäster | ⚠️ Kräver Azure-prenumeration |

---

## Reflektioner

Felet med Self-checkbox var lätt att missa. Portalen ger inget tydligt felmeddelande, det ser bara ut som att paketet inte finns. I produktion är det viktigt att testa hela flödet som slutanvändare direkt efter konfiguration.

Approve-flödet via myaccess.microsoft.com är inte intuitivt. Man förväntar sig att kunna godkänna direkt i admin-portalen, men det är inte hur det fungerar. Värt att kommunicera till kunder vid implementation.

---

## Referenser

- [Entitlement Management – Översikt](https://learn.microsoft.com/en-us/entra/id-governance/entitlement-management-overview)
- [Skapa ett Access Package](https://learn.microsoft.com/en-us/entra/id-governance/entitlement-management-access-package-create)
- [Godkänna Access Package-begäranden](https://learn.microsoft.com/en-us/entra/id-governance/entitlement-management-request-approve)
