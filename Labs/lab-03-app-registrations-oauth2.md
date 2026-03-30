# Lab 03 – App Registrations, OAuth2 & OIDC

**Datum:** Mars 2026  
**Miljö:** Microsoft 365 Business Premium + Microsoft Entra ID P2  
**Tenant:** entrajohanlabb.onmicrosoft.com  
**Svårighetsgrad:** Medel  
**Tidsåtgång:** ~1.5 timmar

---

## Scenario

Det här labbet registrerar en testapplikation i Entra ID och kör Authorization Code Flow manuellt via webbläsare och Postman. Målet är att förstå hur tokens utfärdas och vad de faktiskt innehåller.

---

## Mål

- Registrera en applikation i Entra ID
- Genomföra Authorization Code Flow manuellt
- Dekoda och analysera en JWT access token
- Konfigurera API permissions med admin consent
- Förstå skillnaden mellan Delegated och Application permissions

---

## Förutsättningar

- Slutfört Lab 01 och Lab 02
- Postman installerat
- Alice konfigurerad med MFA

---

## OAuth2 Authorization Code Flow

Flödet sker i två steg. Först loggar användaren in och Microsoft returnerar en Authorization Code. Sedan byter appen Authorization Code mot en Access Token i en server-to-server-förfrågan.

```
1. Användaren loggar in → Microsoft returnerar Authorization Code (engångskod, giltig ~10 min)
2. Appen byter Authorization Code mot Access Token via POST till token-endpoint
```

Anledningen till att det är två steg: Authorization Code skickas via webbläsaren och kan potentiellt exponeras. Access Token hämtas server-to-server med client secret, så den aldrig syns i webbläsarens URL eller historik.

---

## Implementation

### 1. Registrera applikationen

```
Entra admin center → Applications → App registrations → + New registration
```

| Inställning | Värde |
|-------------|-------|
| Name | MyTestApp |
| Supported account types | Single tenant only |
| Redirect URI (platform) | Web |
| Redirect URI | https://jwt.ms |

`jwt.ms` är Microsofts token-debugger. Tokens skickas dit och dekodas direkt i webbläsaren.

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

> ⚠️ Client secret visas bara en gång direkt efter skapandet. Navigerar man bort utan att kopiera värdet måste ett nytt skapas.

### 3. Hämta Authorization Code

Bygger Authorization Code Flow-URL och öppnar i webbläsaren:

```
https://login.microsoftonline.com/<TENANT_ID>/oauth2/v2.0/authorize
  ?client_id=<CLIENT_ID>
  &response_type=code
  &redirect_uri=https://jwt.ms
  &scope=openid profile
  &response_mode=fragment
```

| Parameter | Värde | Förklaring |
|-----------|-------|------------|
| response_type | code | Begär authorization code |
| scope | openid profile | OIDC-scopes för användaridentitet |
| response_mode | fragment | Code returneras i URL-fragmentet (#) |

Efter inloggning som Alice returnerade Microsoft en authorization code i redirect URL:en:
```
https://jwt.ms/#code=1.Aa8A9cw2V-I13Uuq3WRp...
```

### 4. Byta Authorization Code mot Access Token i Postman

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
| `amr` | pwd | Autentiserad med lösenord, inte MFA |
| `oid` | `<REDACTED>` | Alices objekt-ID i Entra |
| `ipaddr` | `<REDACTED>` | IP-adress vid autentisering |

`amr: pwd` är värt att notera. Alice loggade in utan MFA, även om MFA är aktiverat. Det beror på att Authentication methods-policyn är aktiverad men ingen CA-policy tvingar MFA för just det här flödet. CA-policies från Lab 04 löser det.

### 6. Konfigurera API Permissions

```
MyTestApp → API permissions → + Add a permission → Microsoft Graph → Delegated permissions
```

| Permission | Typ | Admin Consent | Syfte |
|------------|-----|---------------|-------|
| User.Read | Delegated | Nej | Läs inloggad användares profil |
| User.ReadAll | Delegated | Ja | Läs alla användares profiler |

Admin consent beviljades för User.ReadAll via "Grant admin consent for Entra Lab".

---

## Delegated vs Application Permissions

| | Delegated | Application |
|--|-----------|-------------|
| Agerar som | Inloggad användare | Applikationen själv |
| Kräver inloggad användare | Ja | Nej |
| Typiskt användningsfall | Webbapp som läser användarens data | Bakgrundstjänst |
| Risk | Begränsad av användarens behörigheter | Kan ha vida behörigheter utan mänsklig kontroll |

Application permissions bör granskas noga vid säkerhetsreview. De agerar utan en inloggad användare och kan komma åt stora datamängder utan att någon märker det.

---

## Troubleshooting

### `AADSTS700051 – response_type 'token' is not supported`
**Orsak:** Implicit flow är inaktiverat som standard i nya app registrations. Det är korrekt beteende eftersom implicit flow exponerar tokens i webbläsarens URL.  
**Lösning:** Använd `response_type=code` (Authorization Code Flow) istället.

### `AADSTS70008 – authorization code expired`
**Orsak:** Authorization codes är giltiga i ~10 minuter och kan bara användas en gång.  
**Lösning:** Hämta ny code och byt mot token direkt utan fördröjning.

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

## Reflektioner

Det som var mest lärorikt var att se `amr: pwd` i token-claimsen. Det visar konkret att en aktiverad MFA-policy i Authentication methods inte automatiskt tvingar MFA på alla tokens. CA-policies i Lab 04 gör det jobbet.

Client secrets är i princip lösenord för applikationer. I produktion ska de bytas mot certifikat eller Managed Identities, annars är de en säkerhetsrisk om de läcker via källkod eller loggar.

---

## Referenser

- [Microsoft identity platform och OAuth 2.0](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-auth-code-flow)
- [JWT.ms – Token debugger](https://jwt.ms)
- [Microsoft Graph permissions](https://learn.microsoft.com/en-us/graph/permissions-reference)
- [App registration i Entra ID](https://learn.microsoft.com/en-us/entra/identity-platform/quickstart-register-app)
