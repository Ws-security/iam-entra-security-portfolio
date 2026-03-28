# Zero Trust IAM Baseline – Microsoft Entra ID

> **Författare:** Johan Wallén-Sjöö  
> **Område:** Identity & Access Management | Microsoft Entra ID

Tio IAM-kontroller för Zero Trust i Microsoft Entra ID, sorterade efter vad som ger mest per investerad timme.

---

## Varför det här dokumentet finns

De flesta säkerhetsbristerna jag stöter på när jag granskar Entra-miljöer handlar inte om avancerade attacker. Det handlar om att grunderna saknas. Ingen MFA. Administrativa roller som ingen använder men ingen törs ta bort. Gästkonton som skapades för tre år sedan och aldrig granskades.

Zero Trust bygger på en enkel idé: lita aldrig, verifiera alltid. Identiteten är det nya perimetret. Det spelar ingen roll om användaren sitter på kontoret eller hemma, varje inloggningsförsök ska utvärderas på samma sätt.

Det här är min sammanfattning av de tio kontrollerna jag alltid börjar med. Varje kontroll har ett PowerShell-kommando du kan köra för att verifiera att det faktiskt fungerar. Det är en sak att klicka i ett gränssnitt, en annan att kunna bevisa det.

---

## Kontroll 1 – MFA för alla användare

Lösenord stjäls. Det är inte en fråga om om, det är en fråga om när. MFA tar fem minuter att sätta upp och blockerar enligt Microsoft över 99,9 % av automatiserade kontoattacker. Att inte ha MFA aktiverat 2026 är ungefär som att lämna ytterdörren olåst.

**Gör så här:**
```
Entra Admin Center → Protection → Authentication methods → Policies
→ Microsoft Authenticator → Enable: Yes → Target: All users
```

**Verifiera:**
```powershell
Connect-MgGraph -Scopes "Policy.Read.All"

$policy = Get-MgPolicyAuthenticationMethodPolicy
$authenticator = $policy.AuthenticationMethodConfigurations | 
    Where-Object { $_.Id -eq "MicrosoftAuthenticator" }

Write-Host "Microsoft Authenticator aktiverat: $($authenticator.State)"
Write-Host "Inkluderade användare: $($authenticator.IncludeTargets.Id)"
```

---

## Kontroll 2 – Break-glass-konto

Det här hoppar de flesta över tills det är för sent. En felaktig Conditional Access-policy kan låsa ut samtliga administratörer från tenanten, och då är ett break-glass-konto skillnaden mellan en stressig halvtimme och en riktigt dålig dag.

Kontot ska aldrig användas i det dagliga arbetet. Det finns bara för nödsituationer.

**Gör så här:**
```
Entra → Identity → Users → + New user

UPN: breakglass@<tenant>.onmicrosoft.com
Roll: Global Administrator (permanent)
Licens: Ingen
MFA: Ej registrerat
Exkludera från ALLA CA-policies
Spara lösenordet offline, inte i en lösenordshanterare
```

**Verifiera:**
```powershell
Connect-MgGraph -Scopes "User.Read.All", "RoleManagement.Read.Directory"

$breakGlass = Get-MgUser -Filter "displayName eq 'Break Glass'"
$globalAdminRole = Get-MgDirectoryRole | Where-Object { $_.DisplayName -eq "Global Administrator" }
$members = Get-MgDirectoryRoleMember -DirectoryRoleId $globalAdminRole.Id

$isAdmin = $members | Where-Object { $_.Id -eq $breakGlass.Id }
Write-Host "Break-glass är Global Admin: $($null -ne $isAdmin)"
```

---

## Kontroll 3 – Conditional Access: kräv MFA via policy

Security Defaults ger grundläggande MFA men saknar granularitet. Med en CA-policy bestämmer du exakt vem som berörs, för vilka appar och under vilka villkor. Det är också enda sättet att exkludera specifika konton, som ditt break-glass.

En sak jag ser hela tiden: folk sätter på policyn direkt utan att testa. Kör alltid Report-only först och använd What If-verktyget. Det tar en dag extra men sparar dig från att låsa ut dina egna användare.

**Gör så här:**
```
Entra → Protection → Conditional Access → + New policy

Namn: REQUIRE-MFA-ALL-USERS
Users: All users (exkludera break-glass-konto)
Target resources: All cloud apps
Grant: Require multifactor authentication
Kör Report-only i 48h → Kontrollera What If → Sätt sedan On
```

**Verifiera:**
```powershell
Connect-MgGraph -Scopes "Policy.Read.All"

$policies = Get-MgIdentityConditionalAccessPolicy
$mfaPolicy = $policies | Where-Object { 
    $_.DisplayName -like "*MFA*" -and $_.State -eq "enabled" 
}

foreach ($p in $mfaPolicy) {
    Write-Host "Policy: $($p.DisplayName)"
    Write-Host "Status: $($p.State)"
    Write-Host "Grant controls: $($p.GrantControls.BuiltInControls)"
    Write-Host "---"
}
```

---

## Kontroll 4 – Blockera inloggning från högriskländer

De flesta automatiserade attacker mot Microsoft 365 kommer från ett fåtal länder. Det här är en av de enklaste kontrollerna att aktivera och den skär bort en stor del av bruset utan att påverka användare i Sverige.

**Gör så här:**
```
Conditional Access → Named locations → + Countries location

Namn: Allowed-Countries
Länder: Sverige (+ övriga länder där ni har personal)
Mark as trusted: Yes

Ny CA-policy: BLOCK-SIGNIN-OUTSIDE-ALLOWED-COUNTRIES
Users: All users (exkludera break-glass)
Conditions → Locations → Include: Any | Exclude: Allowed-Countries
Grant: Block access
```

**Verifiera:**
```powershell
Connect-MgGraph -Scopes "Policy.Read.All"

$namedLocations = Get-MgIdentityConditionalAccessNamedLocation
$countryLocations = $namedLocations | Where-Object { 
    $_.AdditionalProperties.'@odata.type' -eq "#microsoft.graph.countryNamedLocation" 
}

foreach ($loc in $countryLocations) {
    Write-Host "Named Location: $($loc.DisplayName)"
    Write-Host "Länder: $($loc.AdditionalProperties.countriesAndRegions -join ', ')"
}
```

---

## Kontroll 5 – Privileged Identity Management (PIM)

Förmodligen den kontroll som ger mest men som flest organisationer saknar. Permanenta admin-roller, där någon alltid är Global Administrator dygnet runt, är en av de vanligaste attackvektorerna.

PIM gör rollerna "eligible" istället för aktiva. Vill du använda en admin-roll aktiverar du den för en begränsad tid, med en motivering och ofta ett godkännande. Ingen aktivering, inga privilegier.

**Gör så här:**
```
Identity Governance → Privileged Identity Management → Microsoft Entra roles
→ Välj Global Administrator → Assignments → + Add assignments

Membership type: Eligible (inte Active)

Role settings:
  Max aktiveringstid: 4 timmar
  Kräv MFA vid aktivering
  Kräv motivering
  Kräv godkännande från namngiven approver
```

**Verifiera:**
```powershell
Connect-MgGraph -Scopes "RoleManagement.Read.Directory"

$roleId = (Get-MgDirectoryRole | Where-Object { $_.DisplayName -eq "Global Administrator" }).RoleTemplateId
$eligibleAssignments = Get-MgRoleManagementDirectoryRoleEligibilitySchedule -Filter "roleDefinitionId eq '$roleId'"

Write-Host "Antal Eligible Global Admins: $($eligibleAssignments.Count)"
foreach ($a in $eligibleAssignments) {
    $user = Get-MgUser -UserId $a.PrincipalId -ErrorAction SilentlyContinue
    Write-Host "  → $($user.UserPrincipalName)"
}
```

---

## Kontroll 6 – Identity Protection med riskbaserade policies

Entra Identity Protection analyserar inloggningsdata och flaggar beteenden som avviker: omöjliga resor (inloggning från Sverige och USA inom en timme), anonyma IP-adresser, läckta lösenord. Den sortens mönsterigenkänning kan du inte göra manuellt, oavsett hur ambitiös du är.

Du kan låta systemet agera automatiskt. Hög användarrisk kräver lösenordsbyte, misstänkt inloggning kräver MFA, utan att du behöver sitta och titta.

**Gör så här:**
```
Protection → Identity Protection

User risk policy:
  Users: All | User risk: High
  Access: Allow + Require password change

Sign-in risk policy:
  Users: All | Sign-in risk: Medium and above
  Access: Allow + Require MFA
```

**Verifiera:**
```powershell
Connect-MgGraph -Scopes "IdentityRiskEvent.Read.All"

$riskyUsers = Get-MgRiskyUser -Filter "riskState eq 'atRisk'"
Write-Host "Riskfyllda användare: $($riskyUsers.Count)"

if ($riskyUsers.Count -gt 0) {
    Write-Warning "Dessa konton bör granskas:"
    $riskyUsers | ForEach-Object { Write-Host "  → $($_.UserPrincipalName) | Risk: $($_.RiskLevel)" }
}
```

---

## Kontroll 7 – Passwordless

Lösenord är den svagaste länken i nästan alla säkerhetsincidenter. Passwordless via Microsoft Authenticator eller en FIDO2-nyckel tar bort risken för nätfiske och lösenordsstöld. Det finns inget lösenord att stjäla.

Dessutom är det smidigare. Ett godkännande i telefonen går snabbare än att skriva ett lösenord och sedan fumla med en MFA-kod.

**Gör så här:**
```
Protection → Authentication methods → Policies → Microsoft Authenticator

Enable: Yes
Authentication mode: Passwordless
Target: Börja med IT-personal och privilegierade konton
```

**Verifiera:**
```powershell
Connect-MgGraph -Scopes "UserAuthenticationMethod.Read.All", "User.Read.All"

$users = Get-MgUser -All -Property "id,userPrincipalName"
$passwordlessUsers = @()

foreach ($user in $users) {
    $methods = Get-MgUserAuthenticationMethod -UserId $user.Id
    $hasFido2 = $methods | Where-Object { 
        $_.AdditionalProperties.'@odata.type' -eq "#microsoft.graph.fido2AuthenticationMethod" 
    }
    if ($hasFido2) { $passwordlessUsers += $user.UserPrincipalName }
}

Write-Host "Användare med FIDO2/Passwordless: $($passwordlessUsers.Count)"
$passwordlessUsers | ForEach-Object { Write-Host "  → $_" }
```

---

## Kontroll 8 – Least Privilege

Global Administrator ska vara ovanlig. I de miljöer jag granskat brukar det vara tvärtom: rollen tilldelas när någon behöver göra en sak och tas aldrig bort.

Least Privilege innebär att varje konto har exakt de rättigheter det behöver. Helpdesk behöver inte kunna ändra faktureringsuppgifter. En avdelningschef behöver inte kunna skapa app-registreringar.

**Granska Global Admins:**
```
Entra → Roles and administrators → Global Administrator → Assignments
→ Reducera till specifika roller (User Administrator, Helpdesk Administrator, etc.)
```

**Delegera med Administrative Units:**
```
Entra → Administrative units → + Add
→ Lägg till avdelningens användare
→ Tilldela scoped admins som bara ser sin avdelning
```

**Verifiera:**
```powershell
Connect-MgGraph -Scopes "RoleManagement.Read.Directory", "User.Read.All"

$globalAdminRole = Get-MgDirectoryRole | Where-Object { $_.DisplayName -eq "Global Administrator" }
$permanentAdmins = Get-MgDirectoryRoleMember -DirectoryRoleId $globalAdminRole.Id

Write-Host "Permanenta Global Admins: $($permanentAdmins.Count)"
foreach ($admin in $permanentAdmins) {
    $user = Get-MgUser -UserId $admin.Id -ErrorAction SilentlyContinue
    if ($user) { Write-Host "  → $($user.UserPrincipalName)" }
}

if ($permanentAdmins.Count -gt 3) {
    Write-Warning "Mer än 3 permanenta Global Admins. Det är förmodligen för många."
}
```

---

## Kontroll 9 – B2B-gäster med löpande Access Reviews

Externa samarbetspartners, konsulter och leverantörer bjuds in som gäster och glöms sedan bort. Det är inte ovanligt att hitta gästkonton som tillhör folk som slutade för ett år sedan, fortfarande i systemet, med åtkomst till dokument och grupper.

Access Reviews ser till att gäster regelbundet granskas och automatiskt tas bort om ingen tar ansvar för dem.

**Gör så här:**
```
External Identities → External collaboration settings
  Guest access: Most restrictive
  Guest invite: Only users assigned to specific admin roles can invite

Identity Governance → Access reviews → + New access review
  Scope: Guest users only
  Reviewer: Group owners
  Duration: 7 dagar, Monthly
  Auto-apply: Yes | If no response: Remove access
```

**Verifiera:**
```powershell
Connect-MgGraph -Scopes "User.Read.All"

$guests = Get-MgUser -Filter "userType eq 'Guest'" -All -Property "displayName,userPrincipalName,createdDateTime"
Write-Host "Totalt antal gästanvändare: $($guests.Count)"

$threshold = (Get-Date).AddDays(-180)
$oldGuests = $guests | Where-Object { [datetime]$_.CreatedDateTime -lt $threshold }
Write-Host "Gäster skapade för >180 dagar sedan: $($oldGuests.Count)"

if ($oldGuests.Count -gt 0) {
    Write-Warning "Dessa bör granskas i en Access Review."
}
```

---

## Kontroll 10 – Centraliserad loggning och KQL

Du kan inte försvara det du inte ser. Utan centraliserad loggning vet du inte om en attack pågår, och du kan inte utreda den efteråt.

Sign-in logs och Audit logs är grunden. Skicka dem till Log Analytics så kan du börja ställa frågor mot datan. Vilka användare misslyckas med inloggning? Vem aktiverade en admin-roll klockan tre på natten?

**Sätta upp loggning:**
```
Azure Portal → Log Analytics workspaces → + Create

Entra → Monitoring → Diagnostic settings → + Add diagnostic setting
  SignInLogs ✓
  AuditLogs ✓
  NonInteractiveUserSignInLogs ✓
  Skicka till Log Analytics workspace
```

**Tre queries jag använder regelbundet:**
```kql
// Misslyckade inloggningar senaste dygnet
SignInLogs
| where TimeGenerated > ago(24h)
| where ResultType != 0
| summarize Failures = count() by UserPrincipalName, ResultDescription
| where Failures > 5
| order by Failures desc

// Lyckade inloggningar utanför Sverige
SignInLogs
| where TimeGenerated > ago(7d)
| where Location !startswith "SE"
| where ResultType == 0
| project TimeGenerated, UserPrincipalName, Location, IPAddress, AppDisplayName

// PIM-aktiveringar senaste veckan
AuditLogs
| where TimeGenerated > ago(7d)
| where OperationName == "Add member to role completed (PIM activation)"
| project TimeGenerated, InitiatedBy, TargetResources
```

**Verifiera att loggarna flödar:**
```powershell
Connect-MgGraph -Scopes "AuditLog.Read.All"

$recentSignIns = Get-MgAuditLogSignIn -Top 10 -OrderBy "createdDateTime desc"
Write-Host "Sign-in logs tillgängliga: $($recentSignIns.Count -gt 0)"

$recentAudit = Get-MgAuditLogDirectoryAudit -Top 10 -OrderBy "activityDateTime desc"
Write-Host "Audit logs tillgängliga: $($recentAudit.Count -gt 0)"
Write-Host "Senaste händelse: $($recentAudit[0].ActivityDisplayName)"
```

---

## Prioriteringsordning

Om du ska börja från noll, gör det ungefär i den här ordningen:

| # | Kontroll | Prioritet | Tid |
|---|---------|-----------|-----|
| 1 | MFA för alla | 🔴 Kritisk | 5 min |
| 2 | Break-glass-konto | 🔴 Kritisk | 10 min |
| 3 | CA-policy för MFA | 🔴 Kritisk | 30 min |
| 5 | PIM för admins | 🔴 Kritisk | 1–2h |
| 10 | Loggning | 🔴 Kritisk | 1h |
| 4 | Blockera högriskländer | 🟠 Hög | 15 min |
| 6 | Identity Protection | 🟠 Hög | 30 min |
| 9 | B2B + Access Reviews | 🟠 Hög | 1h |
| 8 | Least Privilege / RBAC | 🔴 Kritisk | Pågående |
| 7 | Passwordless | 🟡 Medel | Pågående |

---

## Secure Score

Alla kontroller i den här guiden bidrar till ett högre Secure Score, Microsofts eget mått på hur säkert din tenant är konfigurerad.

```
Entra Admin Center → Identity Secure Score
```

Sikta på minst 70 % som baseline. Rekommendationerna är sorterade efter vad som ger mest poäng per insats, bra startpunkt om du inte vet var du ska börja.

---

## Komma igång med PowerShell

```powershell
# Installera Microsoft Graph SDK (kör en gång)
Install-Module Microsoft.Graph -Scope CurrentUser

# Anslut med nödvändiga scopes
Connect-MgGraph -Scopes "Policy.Read.All","User.Read.All","RoleManagement.Read.Directory","AuditLog.Read.All","IdentityRiskEvent.Read.All","UserAuthenticationMethod.Read.All","AccessReview.Read.All"
```

---

*Bygger på praktisk erfarenhet från ett Entra ID P2-labbtenant och Microsofts Zero Trust-ramverk.*
