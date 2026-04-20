# Case – Microsoft Secure Score: Säkerhetsgenomgång och åtgärdsplan

**Datum:** April 2026  
**Miljö:** Microsoft 365 Business Premium + Microsoft Entra ID P2  
**Tenant:** \<tenant\>.onmicrosoft.com  
**Typ:** Säkerhetsgenomgång, IAM-fokus

---

## Bakgrund

Det här caset dokumenterar en säkerhetsgenomgång baserad på Microsoft Secure Score. Utgångspunkten är en exporterad Secure Score-rapport från Microsoft Defender-portalen. Rapporten rangordnar rekommendationer efter potentiell poängökning och visar nuläget per kontroll.

Secure Score är ett bra verktyg för att identifiera lågt hängande frukt i en kunds miljö. Det ger en snabb överblick och ett konkret underlag för att prioritera åtgärder.

---

## Nuläge vid genomgång

Secure Score-rapporten exporterades den 16 april 2026. Totalt 16 rekommendationer, varav 6 redan slutförda och 10 kvarstående.

**Slutförda kontroller:**
- Lösenord satt till att aldrig löpa ut
- User consent till appar begränsad
- Mer än en Global Administrator utsedd
- Least privilege-roller i bruk
- Teams-inställningar konfigurerade (lobby, anonyma användare)

**Kvarstående med högst prioritet:**

| Rank | Rekommendation | Poängökning | Status |
|------|---------------|-------------|--------|
| 1 | MFA för alla adminroller | +15.63% | To address |
| 2 | Identity Protection sign-in risk policy | +10.94% | To address |
| 3 | Identity Protection user risk policy | +10.94% | To address |
| 4 | MFA för alla användare | +14.06% | To address |
| 5 | Blockera legacy authentication | +12.50% | To address |
| 9 | SSPR för alla användare | +1.56% | To address |

---

## Analys

**Rank 1 och 4 – MFA**

PowerShell-körning via Microsoft Graph API visade att Charlie Carlsson (Helpdesk Administrator, Authentication Administrator) saknade MFA-registrering. Det förklarade varför rank 1 var markerad som "Regressed" – kontot hade tidigare uppfyllt kravet men tappat det.

```powershell
# Query som identifierade problemet
$adminMFA | Format-Table -AutoSize

Roll                   Namn                  MFA
----                   ----                  ---
User Administrator     Alice Andersson       Ja
Global Administrator   Johan Lab             Ja
Global Administrator   Break Glass Emergency Nej
Helpdesk Administrator Charlie Carlsson      Nej
```

Break Glass saknar MFA avsiktligt och ska undantas från alla MFA-krav. Charlie registrerade MFA via `aka.ms/mysecurityinfo` och kontrollerades med ny Graph-körning.

**Rank 5 – Legacy authentication**

Ingen CA-policy för att blockera legacy authentication fanns i miljön. Legacy-protokoll som Exchange ActiveSync och Other clients stöder inte MFA och är en känd attackvektor. Policyn skapades och aktiverades direkt.

**Rank 2 och 3 – Identity Protection**

User Risk Policy och Sign-in Risk Policy är konfigurerade i Report-only via Conditional Access (Lab 07). De visas som "To address" i Secure Score eftersom de inte är fullt aktiverade. Beslutet att hålla dem i Report-only är medvetet – de ska aktiveras efter att FIDO2-registrering slutförts för adminanvändare.

---

## Genomförda åtgärder

### Åtgärd 1: MFA-registrering för Charlie

Charlie Carlsson registrerade Microsoft Authenticator via `aka.ms/mysecurityinfo`. Verifierades med Graph API-query efteråt.

**Före:**
```
Charlie Carlsson – MFA: Nej
```

**Efter:**
```
Charlie Carlsson – MFA: Ja
```

### Åtgärd 2: Skapa BLOCK-LEGACY-AUTH

```
Conditional Access → + New policy

Name: BLOCK-LEGACY-AUTH
Users – Include: All users
Users – Exclude: breakglass@<tenant>.onmicrosoft.com
Target resources: All cloud apps
Conditions → Client apps:
  Exchange ActiveSync clients: Ja
  Other clients: Ja
  Browser: Nej
  Mobile apps and desktop clients: Nej
Grant: Block access
Enable policy: On
```

Verifierades med:

```powershell
Get-MgIdentityConditionalAccessPolicy | 
    Select-Object DisplayName, State | 
    Format-Table -AutoSize

DisplayName          State
-----------          -----
BLOCK-LEGACY-AUTH    enabled
```

---

## Status efter åtgärder

| Rank | Rekommendation | Status |
|------|---------------|--------|
| 1 | MFA för alla adminroller | ✅ Åtgärdad |
| 2 | Identity Protection sign-in risk policy | ⏳ Report-only, medvetet val |
| 3 | Identity Protection user risk policy | ⏳ Report-only, medvetet val |
| 4 | MFA för alla användare | ✅ Åtgärdad |
| 5 | Blockera legacy authentication | ✅ Åtgärdad |
| 9 | SSPR för alla användare | Ej prioriterad i detta uppdrag |

---

## Lärdomar

Secure Score är ett bra startpunkt för en säkerhetsgenomgång men behöver tolkas i kontext. "Regressed" på rank 1 visade att miljön tidigare uppfyllt kravet men tappat det – det signalerar att något förändrats som borde fångas av en löpande övervakningsprocess.

Att kombinera Secure Score med Graph API-queries gör det möjligt att snabbt identifiera exakt vilka konton eller konfigurationer som orsakar ett underkänt krav, istället för att leta manuellt i portalen.

Legacy authentication bör alltid vara en av de första kontrollerna vid en kundgenomgång. Det är en av de enklaste åtgärderna att genomföra och ger stor säkerhetseffekt.

---

## Verktyg som användes

- Microsoft Defender-portalen – Secure Score-export
- Microsoft Graph PowerShell SDK – MFA-verifiering per användare och adminroll
- Microsoft Entra admin center – CA-policy och MFA-registrering

---

## Referenser

- [Microsoft Secure Score](https://learn.microsoft.com/en-us/defender-xdr/microsoft-secure-score)
- [Blockera legacy authentication med CA](https://learn.microsoft.com/en-us/entra/identity/conditional-access/block-legacy-authentication)
- [Microsoft Graph PowerShell SDK](https://learn.microsoft.com/en-us/powershell/microsoftgraph/overview)
