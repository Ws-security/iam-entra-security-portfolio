# Lab 02 – SAML-baserad SSO i Microsoft Entra

**Datum:** Mars 2026  
**Miljö:** Microsoft 365 Business Premium + Microsoft Entra ID P2  
**Tenant:** `<tenant>.onmicrosoft.com`  
**Svårighetsgrad:** Medel  
**Tidsåtgång:** ~1 timme  

## Scenario

Det här labbet fokuserar på att konfigurera och testa **SAML-baserad Single Sign-On (SSO)** mellan **Microsoft Entra ID** och en extern applikation. Som testapplikation användes **Microsoft Entra SAML Toolkit** från Entra App Gallery.

Målet var att förstå hur en **Enterprise Application** kopplas till Entra, hur **SAML-federering** fungerar i praktiken och hur man verifierar att en användare faktiskt kan logga in via **SSO**.

Under labben blev det också tydligt att applikationen hade både en **lokal login-sida** och en separat **SAML-login-endpoint**. Det gjorde att det först såg ut som att SSO inte fungerade, trots att konfigurationen i princip var korrekt.

## Mål

- Lägga till **Microsoft Entra SAML Toolkit** som Enterprise Application
- Konfigurera **SAML-baserad Single Sign-On**
- Tilldela en testanvändare till applikationen
- Konfigurera applikationen med rätt Entra-värden
- Verifiera att användaren kan logga in via **SAML SSO**

## Förutsättningar

- Befintlig Microsoft Entra-labbmiljö
- Behörighet att hantera Enterprise Applications
- Minst en vanlig intern testanvändare
- Webbläsare med möjlighet att testa i **InPrivate/Incognito**

I detta labb användes:

- **Alice Andersson** som testanvändare
- befintlig tenant i Microsoft Entra
- **Microsoft Entra SAML Toolkit** som testapplikation

## Implementation

### Steg 1: Lägg till applikationen

**Enterprise applications → New application**

Sök efter:

**Microsoft Entra SAML Toolkit**

Lägg till applikationen från Entra App Gallery.

### Steg 2: Välj testanvändare

I stället för att skapa ett nytt konto användes en befintlig intern användare:

- **Alice Andersson**

Adminkontot och break-glass-kontot användes inte, eftersom en vanlig användare ger ett mer realistiskt test av slutanvändarflödet.

### Steg 3: Tilldela användaren till applikationen

**Enterprise applications → Microsoft Entra SAML Toolkit → Users and groups → Add user/group**

Tilldela:

- **Alice Andersson**

Detta behövdes för att användaren skulle kunna använda applikationen via Entra.

### Steg 4: Konfigurera SAML i Microsoft Entra

**Enterprise applications → Microsoft Entra SAML Toolkit → Single sign-on → SAML**

#### Basic SAML Configuration

| Inställning | Värde |
|---|---|
| Identifier (Entity ID) | `https://samltoolkit.azurewebsites.net` |
| Reply URL (Assertion Consumer Service URL) | `https://samltoolkit.azurewebsites.net/SAML/Consume` |
| Sign on URL | `https://samltoolkit.azurewebsites.net/` |
| Relay State | Tom |
| Logout URL | Tom |

När dessa värden sparades accepterades konfigurationen utan fel.

### Steg 5: Hämta Entra-värden till applikationen

Från sektionerna **Set up Microsoft Entra SAML Toolkit** och **SAML Certificates** hämtades följande värden för att användas i applikationen:

- **Login URL**
- **Microsoft Entra Identifier**
- **Logout URL**
- **Certificate (Raw)**

Det här behövdes eftersom SAML SSO måste konfigureras både i **Microsoft Entra** och i **målapplikationen**.

### Steg 6: Registrera användaren i toolkit-applikationen

I själva **SAML Toolkit** registrerades en användare med samma identitet som testanvändaren i Entra.

Användare:

- **alice@<tenant>.onmicrosoft.com**

Det krävdes för att användarmatchningen mellan Entra och applikationen skulle fungera korrekt.

### Steg 7: Konfigurera SAML i toolkit-applikationen

Efter inloggning i toolkit-applikationen skapades en **SAML-konfiguration** där följande värden fylldes i:

| Inställning | Källa |
|---|---|
| Azure AD Login URL | Från Entra |
| Azure AD Identifier | Från Entra |
| Logout URL | Från Entra |
| RAW Certificate | Från Entra |

När detta sparades visade applikationen detaljerna för den skapade **single sign-on configuration**.

### Steg 8: Identifiera rätt inloggningsflöde

Under testningen blev det tydligt att applikationen hade två olika inloggningsvägar:

1. **Lokal login-sida**
2. **SAML-baserad login-sida**

Den vanliga **Log in**-knappen i applikationens toppmeny gick till lokal inloggning och användes därför inte för att verifiera federerad SSO.

Det riktiga testet behövde i stället göras via applikationens **SP-initiated SAML login URL**, som låg under en adress i stil med:

`/SAML/Login/...`

Det här var en viktig lärdom, eftersom det först såg ut som att SSO inte fungerade när fel login-väg användes.

### Steg 9: Testa SSO-flödet

Testet genomfördes i **InPrivate-fönster**.

#### Testflöde

1. Öppna nytt **InPrivate-fönster**
2. Gå till **https://myapps.microsoft.com**
3. Logga in som **Alice Andersson**
4. Öppna **Microsoft Entra SAML Toolkit**
5. Gå till den riktiga **SAML login-sidan**
6. Klicka på **Log in**

#### Resultat

Efter detta skickades Alice genom det federerade SAML-flödet och tillbaka till applikationen som inloggad användare.

Det bekräftades genom att applikationen visade:

- **Hello alice@<tenant>.onmicrosoft.com!**

Detta verifierade att **SP-initiated SAML SSO** fungerade korrekt.

## Verifiering

| Kontroll | Status |
|---|---|
| Enterprise Application tillagd | ✅ |
| Alice Andersson tilldelad applikationen | ✅ |
| Basic SAML Configuration konfigurerad | ✅ |
| SAML Toolkit konfigurerad med Entra-värden | ✅ |
| Rätt SAML login-endpoint identifierad | ✅ |
| SP-initiated SAML SSO verifierad | ✅ |

## Reflektioner

Det mest lärorika i labbet var att förstå skillnaden mellan **lokal applikationsinloggning** och **federerad SAML-inloggning**.

Till en början såg det ut som att SSO inte fungerade, eftersom applikationen öppnades men användaren hamnade på en vanlig startsida eller lokal login-sida. Efter vidare testning blev det tydligt att applikationen hade en separat **SAML login-endpoint**, och det var först via den som det riktiga SSO-flödet kunde verifieras.

Det här visar en viktig IAM-princip i praktiken: det räcker inte att bara fylla i rätt värden. Man måste också förstå vilket autentiseringsflöde som faktiskt används och vilken endpoint som triggar federeringen.

## Säkerhetsvärde

Ur ett säkerhetsperspektiv är SSO värdefullt eftersom autentisering centraliseras till Microsoft Entra. Det gör det enklare att kombinera applikationsåtkomst med:

- **MFA**
- **Conditional Access**
- central användarstyrning
- bättre spårbarhet
- enklare livscykelhantering

I stället för att varje applikation hanterar egna lokala autentiseringsflöden kan organisationen använda en gemensam identitetsplattform med bättre kontroll.

## Referenser

- Microsoft Entra Enterprise Applications
- Microsoft Entra SAML Toolkit
- SAML-baserad Single Sign-On i Microsoft Entra
- Microsoft My Apps
