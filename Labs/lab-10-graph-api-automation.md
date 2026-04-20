# Lab 10 – Microsoft Graph API & IAM-automation

**Datum:** April 2026  
**Miljö:** Microsoft 365 Business Premium + Microsoft Entra ID P2  
**Tenant:** \<tenant\>.onmicrosoft.com  
**Svårighetsgrad:** Medel  
**Tidsåtgång:** ~1 timme

---

## Scenario

Det här labbet använder Microsoft Graph API via PowerShell för att hämta och analysera IAM-data i tenanten. Målet är att gå från att klicka i portalen till att kunna köra samma kontroller programmatiskt – något som är nödvändigt vid större kundmiljöer där manuell granskning inte är skalbar.

---

## Mål

- Installera Microsoft Graph PowerShell SDK
- Ansluta till tenanten med rätt scopes
- Köra queries för MFA-status, CA-policies, PIM och riskfyllda användare
- Förstå scope-modellen i Graph API

---

## Förutsättningar

- Slutfört Lab 01–09
- PowerShell 5.1 eller senare
- Admin-konto med Global Administrator

---

## Implementation

### 1. Installera Microsoft Graph SDK

```powershell
Install-Module Microsoft.Graph -Scope CurrentUser -Force
```

Installationen tar några minuter. Verifiera efteråt:

```powershell
Get-Module Microsoft.Graph -ListAvailable | Select-Object Name, Version
```

### 2. Anslut till tenanten

Graph API använder en scope-modell där varje behörighet måste deklareras explicit vid inloggning. Anslut med alla scopes som behövs för det här labbet:

```powershell
Connect-MgGraph -Scopes "User.Read.All","Policy.Read.All","RoleManagement.Read.Directory","AuditLog.Read.All","IdentityRiskEvent.Read.All","UserAuthenticationMethod.Read.All","IdentityRiskyUser.Read.All"
```

Ett webbläsarfönster öppnas för inloggning. Logga in med admin-kontot och godkänn behörigheterna.

### 3. Hämta alla användare

```powershell
Get-MgUser -All -Property "displayName,userPrincipalName,accountEnabled" | 
    Select-Object DisplayName, UserPrincipalName, AccountEnabled | 
    Format-Table -AutoSize
```

**Output:**

| DisplayName | UserPrincipalName | AccountEnabled |
|-------------|-------------------|----------------|
| Johan Lab | admin@\<tenant\>.onmicrosoft.com | True |
| Alice Andersson | alice@\<tenant\>.onmicrosoft.com | True |
| Bob Bengtsson | bob@\<tenant\>.onmicrosoft.com | True |
| Break Glass Emergency | breakglass@\<tenant\>.onmicrosoft.com | True |
| Charlie Carlsson | charlie@\<tenant\>.onmicrosoft.com | True |
| Eve Eriksson | eve@\<tenant\>.onmicrosoft.com | True |
| Diana Guest | \<redacted\>#EXT#@\<tenant\>.onmicrosoft.com | True |

### 4. MFA-status per användare

```powershell
$users = Get-MgUser -All -Property "id,displayName,userPrincipalName"

$results = foreach ($user in $users) {
    $methods = Get-MgUserAuthenticationMethod -UserId $user.Id
    $hasMFA = $methods | Where-Object { 
        $_.AdditionalProperties.'@odata.type' -ne "#microsoft.graph.passwordAuthenticationMethod" 
    }
    [PSCustomObject]@{
        Name = $user.DisplayName
        UPN  = $user.UserPrincipalName
        MFARegistered = if ($hasMFA) { "Ja" } else { "Nej" }
    }
}
$results | Format-Table -AutoSize
```

**Output:**

| Name | UPN | MFARegistered |
|------|-----|---------------|
| Johan Lab | admin@\<tenant\>.onmicrosoft.com | Ja |
| Alice Andersson | alice@\<tenant\>.onmicrosoft.com | Ja |
| Bob Bengtsson | bob@\<tenant\>.onmicrosoft.com | Ja |
| Break Glass Emergency | breakglass@\<tenant\>.onmicrosoft.com | Nej |
| Charlie Carlsson | charlie@\<tenant\>.onmicrosoft.com | Nej |
| Eve Eriksson | eve@\<tenant\>.onmicrosoft.com | Ja |
| Diana Guest | \<redacted\>#EXT#@\<tenant\>.onmicrosoft.com | Ja |

Break Glass saknar MFA avsiktligt. Charlie saknar MFA och bör följas upp.

### 5. Conditional Access-policies och status

```powershell
Get-MgIdentityConditionalAccessPolicy | 
    Select-Object DisplayName, State | 
    Format-Table -AutoSize
```

**Output:**

| DisplayName | State |
|-------------|-------|
| REQUIRE-MFA-ALL-USERS | enabled |
| BLOCK-SIGNIN-OUTSIDE-SWEDEN | enabled |
| REQUIRE-COMPLIANT-DEVICE-ADMINS | enabledForReportingButNotEnforced |
| USER-RISK-POLICY | enabledForReportingButNotEnforced |
| SIGNIN-RISK-POLICY | enabledForReportingButNotEnforced |

### 6. Eligible Global Admins via PIM

```powershell
$roleId = (Get-MgDirectoryRole | Where-Object { $_.DisplayName -eq "Global Administrator" }).RoleTemplateId

Get-MgRoleManagementDirectoryRoleEligibilitySchedule -Filter "roleDefinitionId eq '$roleId'" |
    ForEach-Object {
        $user = Get-MgUser -UserId $_.PrincipalId -ErrorAction SilentlyContinue
        [PSCustomObject]@{
            Användare = $user.DisplayName
            UPN = $user.UserPrincipalName
            Roll = "Global Administrator (Eligible)"
        }
    } | Format-Table -AutoSize
```

**Output:**

| Användare | UPN | Roll |
|-----------|-----|------|
| Alice Andersson | alice@\<tenant\>.onmicrosoft.com | Global Administrator (Eligible) |

### 7. Riskfyllda användare

```powershell
Get-MgRiskyUser -Filter "riskState eq 'atRisk'" |
    Select-Object UserDisplayName, UserPrincipalName, RiskLevel, RiskState |
    Format-Table -AutoSize
```

Ingen output i labbmiljön – inga aktiva riskhändelser. Korrekt beteende.

---

## Troubleshooting

### 403 Forbidden vid Get-MgUserAuthenticationMethod
**Orsak:** Scopet `UserAuthenticationMethod.Read.All` saknades vid inloggning.  
**Lösning:** Koppla upp igen med rätt scope inkluderat i `Connect-MgGraph`.

### 403 Forbidden vid Get-MgRiskyUser
**Orsak:** Scopet `IdentityRiskyUser.Read.All` saknades.  
**Lösning:** Samma som ovan – lägg till scopet och anslut på nytt.

### ParserError: An empty pipe element is not allowed
**Orsak:** PowerShell tillåter inte pipe direkt efter en foreach-loop.  
**Lösning:** Tilldela foreach-resultatet till en variabel och pipe variabeln separat.

---

## Verifiering

| Kontroll | Status |
|----------|--------|
| Graph SDK installerat | ✅ |
| Anslutning med delegerade scopes | ✅ |
| Användarlista hämtad | ✅ |
| MFA-status per användare | ✅ |
| CA-policies och status | ✅ |
| Eligible Global Admins via PIM | ✅ |
| Risky users query | ✅ |

---

## Reflektioner

Scope-modellen i Graph API var det som tog längst tid att förstå. Till skillnad från portalen där behörigheterna är implicita, måste varje scope deklareras explicit vid anslutning. Det innebär att man som konsult behöver planera vilka scopes som krävs för en given uppgift innan man kör något.

Det praktiska värdet är tydligt. MFA-statusqueryn körde mot sju användare på sekunder. I en kundmiljö med hundratals användare är det här enda sättet att få en snabb överblick.

Att foreach-loopen inte kan pipas direkt i PowerShell är ett syntaxfälla som är värd att komma ihåg.

---

## Referenser

- [Microsoft Graph PowerShell SDK](https://learn.microsoft.com/en-us/powershell/microsoftgraph/overview)
- [Graph API permissions reference](https://learn.microsoft.com/en-us/graph/permissions-reference)
- [Get-MgUser](https://learn.microsoft.com/en-us/powershell/module/microsoft.graph.users/get-mguser)
- [Get-MgRiskyUser](https://learn.microsoft.com/en-us/powershell/module/microsoft.graph.identity.signins/get-mgriskyuser)
