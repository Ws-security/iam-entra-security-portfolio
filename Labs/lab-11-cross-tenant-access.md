# Lab 11 – Cross-tenant Access Settings

**Datum:** April 2026  
**Miljö:** Microsoft 365 Business Premium + Microsoft Entra ID P2  
**Tenant:** \<tenant\>.onmicrosoft.com  
**Svårighetsgrad:** Medel  
**Tidsåtgång:** ~45 minuter

---

## Scenario

Det här labbet konfigurerar Cross-tenant Access Settings, som styr hur din tenant samarbetar med externa Microsoft Entra-tenants. I kundmiljöer används det för att hantera trust-inställningar mot partners och leverantörer, och för att undvika att externa gäster tvingas göra MFA flera gånger.

---

## Mål

- Förstå skillnaden mellan default settings och organisationsspecifika inställningar
- Konfigurera Trust settings på default-nivå
- Lägga till en organisation med anpassade inställningar
- Granska Microsoft cloud settings

---

## Förutsättningar

- Slutfört Lab 01–10
- Global Administrator-behörighet

---

## Teori: Inbound och Outbound

Cross-tenant Access Settings har två riktningar:

**Inbound** – vad externa Entra-tenants får göra mot din tenant. Påverkar gästanvändare som loggar in på dina resurser.

**Outbound** – vad dina användare får göra mot externa tenants. Påverkar dina egna användare när de loggar in hos externa organisationer.

Trust settings styr om din Conditional Access ska acceptera anspråk från externa tenants. Utan trust tvingas en extern användare göra MFA både i sin hemtenant och i din, vilket är onödigt i ett etablerat partnerskap.

---

## Implementation

### 1. Granska default settings

```
Entra admin center → External Identities → Cross-tenant access settings → Default settings
```

Standardinställningarna som visades:

**Inbound access settings:**

| Type | Applies to | Status |
|------|------------|--------|
| B2B collaboration | External users and groups | All allowed |
| B2B collaboration | Applications | All allowed |
| B2B direct connect | External users and groups | All blocked |
| B2B direct connect | Applications | All blocked |
| Trust settings | N/A | Disabled |

**Outbound access settings:**

| Type | Applies to | Status |
|------|------------|--------|
| B2B collaboration | Users and groups | All allowed |
| B2B collaboration | External applications | All allowed |
| B2B direct connect | Users and groups | All blocked |
| B2B direct connect | External applications | All blocked |

**Tenant restrictions:**

| Applies to | Status |
|------------|--------|
| External users and groups | All blocked |
| External applications | All blocked |

### 2. Aktivera Trust settings på default-nivå

```
Default settings → Edit inbound defaults → Trust settings
```

Aktiverade: **Trust multifactor authentication from Microsoft Entra tenants**

Det innebär att om en extern användare redan gjort MFA i sin hemtenant, accepterar din CA-policy det utan att kräva MFA igen. Relevant för alla externa samarbeten där dubblad MFA-prompt annars blir ett problem.

Sparade inställningen.

### 3. Lägg till organisationsspecifik inställning för Microsoft

```
Organizational settings → + Add an organization → microsoft.com
```

Microsoft-tenanten lades till och visades initialt med alla inställningar ärvda från default.

Klickade på **Inherited from default** under Inbound access och navigerade till **Trust settings**.

Valde **Customize settings** och aktiverade alla tre:

- Trust multifactor authentication from Microsoft Entra tenants
- Trust compliant devices
- Trust Microsoft Entra hybrid joined devices

Aktiverade även **Automatically redeem invitations with the tenant Microsoft** under Automatic redemption. Det innebär att användare från Microsoft-tenanten inte behöver manuellt acceptera en inbjudan vid sin första inloggning.

Sparade.

### 4. Granska Microsoft cloud settings

```
Cross-tenant access settings → Microsoft cloud settings
```

Visar möjligheten att aktivera B2B-samarbete med organisationer i separata Microsoft-moln:

- Microsoft Azure Government
- Microsoft Azure China (operated by 21Vianet)

Relevant för myndigheter och organisationer med dataresidens-krav. Inga ändringar gjordes här.

---

## Verifiering

| Kontroll | Status |
|----------|--------|
| Default Trust settings – MFA trust aktiverat | ✅ |
| Microsoft-tenant tillagd under Organizational settings | ✅ |
| Anpassade Trust settings för Microsoft (alla tre) | ✅ |
| Automatic redemption aktiverat för Microsoft | ✅ |
| Microsoft cloud settings granskad | ✅ |

---

## Troubleshooting

Inga problem uppstod under labbet. Konfigurationen var rakt fram.

---

## Reflektioner

Det som var mest lärorikt var distinktionen mellan default settings och organisationsspecifika inställningar. Default-nivån täcker alla externa tenants som inte har en specifik konfiguration. Organisationsspecifika inställningar används för partners där ett utökat trust är motiverat.

I kundmiljöer är Trust MFA den inställning som oftast behöver aktiveras direkt. Utan den klagar externa gäster på att de måste göra MFA två gånger, vilket skapar onödiga supportärenden.

Automatic redemption är värdefullt i täta partnerskap där gäster bjuds in frekvent. Utan det får varje ny gäst en inbjudan att manuellt acceptera, vilket försenar onboarding-flödet.

---

## Referenser

- [Cross-tenant access settings i Entra ID](https://learn.microsoft.com/en-us/entra/external-id/cross-tenant-access-settings-b2b-collaboration)
- [Trust settings för B2B-samarbete](https://learn.microsoft.com/en-us/entra/external-id/cross-tenant-access-settings-b2b-collaboration#to-change-inbound-trust-settings-for-mfa-and-device-claims)
- [Microsoft cloud settings](https://learn.microsoft.com/en-us/entra/external-id/cross-tenant-access-settings-b2b-direct-connect)
