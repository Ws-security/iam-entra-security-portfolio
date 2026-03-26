# Lab 03 – App Registrations, OAuth2 & OIDC

**Datum:** Mars 2026
**Miljö:** Microsoft 365 Business Premium + Microsoft Entra ID P2
**Tenant:** entrajohanlabb.onmicrosoft.com
**Svårighetsgrad:** Medel
**Tidsåtgång:** ~1.5 timmar

---

## Scenario

När en applikation behöver logga in användare eller komma åt Microsoft-resurser (som Graph API) måste den registreras i Entra ID. App Registrations är grunden för hur moderna applikationer integreras med Microsoft-identitetsplattformen via OAuth2 och OIDC.

I enterprise-miljöer hanterar IAM-konsulter regelbundet app registrations – vid onboarding av nya SaaS-applikationer, vid integration av interna system med Microsoft Graph, och vid granskning av vilka permissions applikationer har beviljats. Felaktigt konfigurerade API permissions är en vanlig säkerhetsrisk i organisationer.

---

## Mål

- Registrera en applikation i Entra ID
- Genomföra Authorization Code Flow manuellt via webbläsare och Postman
- Dekoda och analysera en JWT access token
- Konfigurera API permissions med admin consent
- Förstå skillnaden mellan Delegated och Application permissions

---

## Förutsättningar

- Slutfört Lab 01 och Lab 02
- Postman installerat
- Testanvändare Alice konfigurerad med MFA

---

## Teoriöversikt: OAuth2 Authorization Code Flow

OAuth2 Authorization Code Flow är den säkraste och mest använda autentiseringsmetoden för webbapplikationer. Flödet fungerar i två steg:

```
1. Användaren loggar in → Microsoft returnerar en Authorization Code (engångskod)
2. Appen byter Authorization Code mot en Access Token via en server-to-server-förfrågan
```

**Authorization Code** är en tillfällig engångskod som representerar användarens godkännande. Den är giltig i ca 10 minuter och kan bara användas en gång – som en kassakvittens som byts mot varan.

**Access Token** är den faktiska nyckeln som ger applikationen tillgång till resurser. Den är en JWT (JSON Web Token) som innehåller claims om användaren, applikationen och vilka behörigheter som beviljats.

**Varför två steg?** Authorization Code skickas via webbläsaren (osäkert). Access Token hämtas server-to-server med client secret (säkert). Det gör att access token aldrig exponeras i webbläsarens URL eller historik.

---

## Implementation

### 1. Registrera applikationen

```
Entra admin center → Applications → App registrations → + New registration
```

| Inställning | Värde |
|-------------|-------|
| Name | MyTestApp |
| Supported account types | Single tenant only – Entra Lab |
| Redirect URI (platform) | Web |
| Redirect URI | https://jwt.ms |

`jwt.ms` är Microsofts egna token-debugger och ett utmärkt verktyg för labbmiljöer – tokens skickas dit och dekodas direkt i webbläsaren utan att lämna klienten.

**Applikationsdetaljer efter registrering:**

| Värde | ID |
|-------|-----|
| Application (client) ID | `<CLIENT_ID>` |
| Directory (tenant) ID | `<TENANT_ID>` |

### 2. Skapa Client Secret

```
MyTestApp → Certificates & secrets → + New client secret
```

| Inställning | Värde |
|-------------|-------|
| Description | lab-secret |
| Expires | 180 days |

> ⚠️ **Kritiskt:** Client secret visas endast en gång direkt efter skapandet. Om du navigerar bort utan att kopiera värdet måste du skapa ett nytt secret. I produktionsmiljöer rekommenderas certifikat framför client secrets för ökad säkerhet.

### 3. Hämta Authorization Code

Byggde Authorization Code Flow-URL manuellt och öppnade i webbläsaren:

```
https://login.microsoftonline.com/<TENANT_ID>/oauth2/v2.0/authorize
  ?client_id=<CLIENT_ID>
  &response_type=code
  &redirect_uri=https://jwt.ms
  &scope=openid profile
  &response_mode=fragment
```

**Parametrar förklarade:**

| Parameter | Värde | Förklaring |
|-----------|-------|------------|
| response_type | code | Begär authorization code (inte token direkt) |
| scope | openid profile | OIDC-scopes för användaridentitet |
| response_mode | fragment | Code returneras i URL-fragmentet (#) |

Efter inloggning som Alice returnerade Microsoft en authorization code i redirect URL:en:
```
https://jwt.ms/#code=1.Aa8A9cw2V-I13Uuq3WRp...
```

> **Notering:** `response_type=token` (implicit flow) returnerade felet `AADSTS700051`. Implicit flow är inaktiverat som standard i moderna Entra-appar – detta är korrekt säkerhetsbeteende. Implicit flow är deprecated och bör aldrig användas i nya applikationer.

### 4. Byta Authorization Code mot Access Token i Postman

Konfigurerade en POST-förfrågan i Postman:

**URL:**
```
https://login.microsoftonline.com/<TENANT_ID>/oauth2/v2.0/token
```

**Body (x-www-form-urlencoded):**

| Key | Value |
|-----|-------|
| grant_type | authorization_code |
| client_id | `<CLIENT_ID>` |
| client_secret | [client secret value] |
| redirect_uri | https://jwt.ms |
| code | [authorization code] |
| scope | openid profile |

**Svar från Microsoft:**
```json
{
    "token_type": "Bearer",
    "scope": "openid profile email",
    "expires_in": 3958,
    "access_token": "eyJ0eXAiOiJKV1Qi...",
    "id_token": "eyJ0eXAiOiJKV1Qi..."
}
```

> **Notering:** Authorization codes är engångskoder med ~10 minuters giltighetstid. Vid första försöket returnerades `AADSTS70008: authorization code expired` eftersom för lång tid förflöt mellan kod-hämtning och token-byte. Lösning: hämta ny kod och byt omedelbart.

### 5. Dekoda och analysera Access Token

Kopierade access token och dekodade på jwt.ms. Viktiga claims:

| Claim | Värde | Betydelse |
|-------|-------|-----------|
| `name` | Alice Andersson | Inloggad användare |
| `upn` | alice@entrajohanlabb.onmicrosoft.com | Username |
| `tid` | `<TENANT_ID>` | Tenant ID |
| `appid` | `<CLIENT_ID>` | MyTestApp |
| `scp` | openid profile email | Beviljade scopes |
| `exp` | 2026-03-26 14:38 | Token utgår om ~1h |
| `amr` | pwd | Autentiserad med lösenord (ej MFA) |
| `oid` | `<REDACTED>` | Alices unika objekt-ID i Entra |
| `ipaddr` | `<REDACTED>` | IP-adress vid autentisering |

**Säkerhetsnotering om `amr: pwd`:**
Access token visar att Alice autentiserade med enbart lösenord, utan MFA. Detta illustrerar varför Conditional Access är kritiskt – utan CA-policies kan applikationer erhålla tokens utan MFA-krav, även i miljöer där MFA är "aktiverat". CA-policies enforcar MFA på policy-nivå, inte bara på användar-nivå.

### 6. Konfigurera API Permissions

```
MyTestApp → API permissions → + Add a permission → Microsoft Graph → Delegated permissions
```

| Permission | Typ | Admin Consent | Syfte |
|------------|-----|---------------|-------|
| User.Read | Delegated | Nej | Läs inloggad användares profil |
| User.ReadAll | Delegated | **Ja** | Läs alla användares profiler |

**Admin consent beviljades** för User.ReadAll via "Grant admin consent for Entra Lab".

---

## Delegated vs Application Permissions

| | Delegated | Application |
|--|-----------|-------------|
| Agerar som | Inloggad användare | Applikationen själv |
| Kräver inloggad användare | Ja | Nej |
| Typiskt användningsfall | Webbapp som läser användarens data | Bakgrundstjänst, daemon |
| Exempel | User.Read (läs min profil) | User.ReadAll (läs alla profiler utan inloggning) |
| Risk | Begränsad av användarens behörigheter | Kan ha vida behörigheter utan mänsklig kontroll |

> **Säkerhetsprincip:** Application permissions bör granskas extra noga vid säkerhetsreview – de agerar utan mänsklig kontext och kan ha tillgång till stora datamängder utan att någon användare behöver vara inloggad.

---

## Troubleshooting

### Problem 1: `AADSTS700051 – response_type 'token' is not supported`
**Orsak:** Implicit flow är inaktiverat som standard.
**Förklaring:** Implicit flow (response_type=token) är deprecated och osäkert – tokens exponeras i webbläsarens URL. Microsoft inaktiverar det som standard i nya app registrations.
**Lösning:** Använd `response_type=code` (Authorization Code Flow) istället.

### Problem 2: `AADSTS70008 – authorization code expired`
**Orsak:** Authorization codes är giltiga i ~10 minuter och kan bara användas en gång.
**Lösning:** Hämta ny authorization code och byt mot token omedelbart utan fördröjning.

---

## Verifiering

| Kontroll | Status |
|----------|--------|
| App registrerad med korrekt redirect URI | ✅ |
| Client secret skapat | ✅ |
| Authorization Code Flow genomfört | ✅ |
| Access token dekodad och claims analyserade | ✅ |
| API permissions konfigurerade med admin consent | ✅ |

---

## Säkerhetsreflektioner

**Principle of Least Privilege för appar:** En applikation bör bara beviljas de permissions den faktiskt behöver. User.ReadAll är en kraftfull permission som ger tillgång till alla användares profiler – i en produktionsmiljö bör detta motiveras och dokumenteras.

**Client secrets vs certifikat:** Client secrets är lösenord för applikationer. De kan läcka via källkod, loggar eller konfigurationsfiler. I produktionsmiljöer rekommenderas certifikat eller Managed Identities istället.

**Token-livslängd:** Access tokens är giltiga i ~1 timme. Om ett token stjäls har angriparen en timmes obegränsad tillgång. Continuous Access Evaluation (CAE) kan minska denna risk genom att omedelbart revokera tokens vid säkerhetshändelser.

---

## Nästa steg

➡️ **Lab 04** – Conditional Access (Zero Trust policies)
➡️ **Lab 05** – B2B Gäståtkomst och External Identities
➡️ **Lab 06** – Privileged Identity Management (PIM)

---

## Referenser

- [Microsoft identity platform och OAuth 2.0](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-auth-code-flow)
- [JWT.ms – Token debugger](https://jwt.ms)
- [Microsoft Graph permissions](https://learn.microsoft.com/en-us/graph/permissions-reference)
- [App registration i Entra ID](https://learn.microsoft.com/en-us/entra/identity-platform/quickstart-register-app)
