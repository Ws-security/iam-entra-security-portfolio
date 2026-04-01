# Lab 09 – SAML-baserad SSO i Microsoft Entra

**Datum:** Mars 2026  
**Miljö:** Microsoft 365 Business Premium + Microsoft Entra ID P2  
**Tenant:** \<tenant\>.onmicrosoft.com  
**Svårighetsgrad:** Medel  
**Tidsåtgång:** ~1 timme

---

## Scenario

Det här labbet konfigurerar SAML-baserad SSO mellan Microsoft Entra ID och en extern applikation. Som testapplikation användes Microsoft Entra SAML Toolkit från Entra App Gallery. Målet var att förstå hur en Enterprise Application kopplas till Entra, hur SAML-federering fungerar i praktiken och hur man verifierar att en användare faktiskt kan logga in via SSO.

---

## Mål

- Lägga till Microsoft Entra SAML Toolkit som Enterprise Application
- Konfigurera SAML-baserad Single Sign-On
- Tilldela en testanvändare till applikationen
- Konfigurera applikationen med rätt Entra-värden
- Verifiera att användaren kan logga in via SAML SSO

---

## Förutsättningar

- Befintlig Microsoft Entra-labbmiljö
- Behörighet att hantera Enterprise Applications
- Alice Andersson konfigurerad som testanvändare

---

## Implementation

### Steg 1: Lägg till applikationen

```
Enterprise applications → New application
Sök efter: Microsoft Entra SAML Toolkit
```

Lägg till applikationen från Entra App Gallery.

### Steg 2: Tilldela Alice till applikationen

```
Enterprise applications → Microsoft Entra SAML Toolkit → Users and groups → Add user/group
```

Alice Andersson tilldelades applikationen. Adminkontot och break-glass-kontot användes inte eftersom en vanlig användare ger ett mer realistiskt test av slutanvändarflödet.

### Steg 3: Konfigurera SAML i Microsoft Entra

```
Enterprise applications → Microsoft Entra SAML Toolkit → Single sign-on → SAML
```

**Basic SAML Configuration:**

| Inställning | Värde |
|-------------|-------|
| Identifier (Entity ID) | https://samltoolkit.azurewebsites.net |
| Reply URL (ACS URL) | https://samltoolkit.azurewebsites.net/SAML/Consume |
| Sign on URL | https://samltoolkit.azurewebsites.net/ |
| Relay State | Tom |
| Logout URL | Tom |

### Steg 4: Hämta Entra-värden till applikationen

Från Set up Microsoft Entra SAML Toolkit och SAML Certificates hämtades:

- Login URL
- Microsoft Entra Identifier
- Logout URL
- Certificate (Raw)

### Steg 5: Registrera användaren i toolkit-applikationen

I SAML Toolkit registrerades en användare med samma identitet som Alice i Entra:

```
alice@<tenant>.onmicrosoft.com
```

### Steg 6: Konfigurera SAML i toolkit-applikationen

| Inställning | Källa |
|-------------|-------|
| Azure AD Login URL | Från Entra |
| Azure AD Identifier | Från Entra |
| Logout URL | Från Entra |
| RAW Certificate | Från Entra |

### Steg 7: Identifiera rätt inloggningsflöde

Applikationen hade två olika inloggningsvägar:

- Lokal login-sida (vanlig Log in-knapp i toppmeny)
- SAML-baserad login-endpoint (`/SAML/Login/...`)

Den vanliga Log in-knappen triggar lokal autentisering, inte SAML. Det riktiga testet krävde att SP-initiated SAML login-URL:en användes direkt.

### Steg 8: Testa SSO-flödet

Testet genomfördes i InPrivate-fönster.

1. Öppna nytt InPrivate-fönster
2. Gå till `https://myapps.microsoft.com`
3. Logga in som Alice Andersson
4. Öppna Microsoft Entra SAML Toolkit
5. Navigera till SAML login-endpoint
6. Klicka Log in

**Resultat:** Alice skickades genom det federerade SAML-flödet och applikationen visade:

```
Hello alice@<tenant>.onmicrosoft.com!
```

SP-initiated SAML SSO fungerade korrekt. ✅

---

## Verifiering

| Kontroll | Status |
|----------|--------|
| Enterprise Application tillagd | ✅ |
| Alice Andersson tilldelad applikationen | ✅ |
| Basic SAML Configuration konfigurerad | ✅ |
| SAML Toolkit konfigurerad med Entra-värden | ✅ |
| Rätt SAML login-endpoint identifierad | ✅ |
| SP-initiated SAML SSO verifierad | ✅ |

---

## Troubleshooting

### SSO verkade inte fungera trots korrekt konfiguration
**Orsak:** Den vanliga Log in-knappen i applikationens toppmeny går till lokal inloggning, inte SAML-federering.  
**Lösning:** Använd applikationens SP-initiated SAML login-URL direkt. Det är via den endpoint som det federerade flödet triggas.

---

## Reflektioner

Det mest lärorika var skillnaden mellan lokal applikationsinloggning och federerad SAML-inloggning. Till en början såg det ut som att SSO inte fungerade eftersom applikationen öppnades men Alice hamnade på en vanlig startsida. Efter vidare testning blev det tydligt att applikationen hade en separat SAML login-endpoint.

Det visar något som är lätt att missa vid implementation: det räcker inte att fylla i rätt värden. Man måste också förstå vilket autentiseringsflöde som faktiskt används och vilken endpoint som triggar federeringen. Vid ett kunduppdrag är det värt att dokumentera exakt vilken URL som ska användas för SSO-inloggning.

---

## Referenser

- [Microsoft Entra Enterprise Applications](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/what-is-application-management)
- [Microsoft Entra SAML Toolkit](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/tutorial-debug-saml-sso-issues)
- [SAML-baserad SSO i Microsoft Entra](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/configure-saml-single-sign-on)
- [Microsoft My Apps](https://myapps.microsoft.com)
