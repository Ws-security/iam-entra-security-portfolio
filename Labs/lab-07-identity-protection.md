# Lab 07 – Identity Protection & riskbaserade policies

**Datum:** Mars 2026  
**Miljö:** Microsoft 365 Business Premium + Microsoft Entra ID P2  
**Tenant:** \<tenant\>.onmicrosoft.com  
**Svårighetsgrad:** Medel  
**Tidsåtgång:** ~1 timme

---

## Scenario

Det här labbet konfigurerar riskbaserade CA-policies via Identity Protection. Microsoft har migrerat User Risk Policy och Sign-in Risk Policy från Identity Protection till Conditional Access. Funktionaliteten är densamma men konfigurationen sker nu på ett ställe tillsammans med övriga CA-policies.

---

## Mål

- Konfigurera USER-RISK-POLICY via Conditional Access med User risk: High
- Konfigurera SIGNIN-RISK-POLICY via Conditional Access med Sign-in risk: Medium and above
- Dokumentera Log Analytics/KQL-konfiguration (kräver Azure-prenumeration)

---

## Förutsättningar

- Slutfört Lab 01–06
- Entra ID P2-licenser tilldelade

---

## Microsoft-ändring: Risk Policies migrerar till CA

Identity Protection visade följande meddelande under User risk policy:

> "This risk policy is now read-only and will be retired on October 1, 2026. To manage or modify it, migrate it to Conditional Access."

Riskbaserade policies konfigureras nu direkt i Conditional Access via Conditions → User risk / Sign-in risk. Det innebär att alla policybeslut samlas på ett ställe, vilket är ett bättre upplägg ur ett förvaltningsperspektiv.

---

## Implementation

### Policy 1: USER-RISK-POLICY

```
Conditional Access → + New policy
```

| Inställning | Värde |
|-------------|-------|
| Name | USER-RISK-POLICY |
| Users – Include | All users |
| Users – Exclude | breakglass@\<tenant\>.onmicrosoft.com |
| Target resources | All cloud apps |
| Conditions – User risk | High |
| Grant | Require password change |
| Enable policy | Report-only |

> **Notering:** Require password change kräver att Target resources är satt till All cloud apps, annars är alternativet grått i Grant-panelen.

### Policy 2: SIGNIN-RISK-POLICY

```
Conditional Access → + New policy
```

| Inställning | Värde |
|-------------|-------|
| Name | SIGNIN-RISK-POLICY |
| Users – Include | All users |
| Users – Exclude | breakglass@\<tenant\>.onmicrosoft.com |
| Target resources | All cloud apps |
| Conditions – Sign-in risk | Medium and above |
| Grant | Require multifactor authentication |
| Enable policy | Report-only |

---

## Log Analytics och KQL

Log Analytics kräver en Azure-prenumeration och konfigurerades inte i det här labbet. Nedan är konfigurationsstegen och queries för referens.

**Konfiguration:**
```
Azure Portal → Log Analytics workspaces → + Create

Entra admin center → Monitoring → Diagnostic settings → + Add diagnostic setting
  SignInLogs ✓
  AuditLogs ✓
  NonInteractiveUserSignInLogs ✓
  → Send to Log Analytics workspace
```

**Användbara queries:**

```kql
// Riskfyllda inloggningar senaste 7 dagarna
SignInLogs
| where TimeGenerated > ago(7d)
| where RiskLevelDuringSignIn in ("medium", "high")
| project TimeGenerated, UserPrincipalName, RiskLevelDuringSignIn, Location, IPAddress
| order by TimeGenerated desc

// Användare med aktiv hög risk
SignInLogs
| where TimeGenerated > ago(24h)
| where RiskLevelAggregated == "high"
| summarize count() by UserPrincipalName
| order by count_ desc

// Misslyckade inloggningar per användare
SignInLogs
| where TimeGenerated > ago(24h)
| where ResultType != 0
| summarize Failures = count() by UserPrincipalName, ResultDescription
| where Failures > 5
| order by Failures desc
```

---

## Verifiering

| Kontroll | Status |
|----------|--------|
| USER-RISK-POLICY konfigurerad i CA | ✅ |
| SIGNIN-RISK-POLICY konfigurerad i CA | ✅ |
| Log Analytics / KQL | ⚠️ Kräver Azure-prenumeration |

---

## Reflektioner

Att User Risk Policy och Sign-in Risk Policy nu bor i Conditional Access är en förbättring jämfört med det gamla upplägget. Tidigare hanterades riskpolicies på ett separat ställe i Identity Protection, vilket lätt ledde till att man tappade översikten. Nu syns alla policies på samma plats.

Meddelandet om att de gamla risk policies pensioneras den 1 oktober 2026 är värt att känna till vid kunduppdrag. En kund som har de gamla policies aktiverade behöver migrera dem till CA innan dess.

---

## Referenser

- [Identity Protection och riskbaserade CA-policies](https://learn.microsoft.com/en-us/entra/id-protection/howto-identity-protection-configure-risk-policies)
- [User risk-baserade CA-policies](https://learn.microsoft.com/en-us/entra/id-protection/howto-identity-protection-configure-risk-policies#user-risk-based-conditional-access-policy)
- [Sign-in risk-baserade CA-policies](https://learn.microsoft.com/en-us/entra/id-protection/howto-identity-protection-configure-risk-policies#sign-in-risk-based-conditional-access-policy)
