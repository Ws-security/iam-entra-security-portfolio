# NIS2 – IAM-kontroller i Microsoft Entra ID

**Författare:** Johan Wallén-Sjöö  
**Område:** Identity & Access Management | NIS2-efterlevnad  
**Uppdaterad:** April 2026

Mappning av NIS2 artikel 21-krav mot konkreta Entra ID-kontroller och genomförda labb.

---

## Bakgrund

NIS2 (EU 2022/2555) ställer krav på tekniska säkerhetsåtgärder för organisationer inom samhällsviktig och viktig verksamhet. Artikel 21 specificerar minimikraven. Det som ofta missas i NIS2-diskussioner är att många av kraven handlar om identitetshantering, inte brandväggar eller antivirus. Frågorna som NIS2 egentligen ställer är: Vem har tillgång till vad? Hur hanteras konton när någon slutar? Kan du visa en audit trail om något går fel?

De kontrollerna finns redan i Microsoft Entra ID P2. Det handlar om att använda dem.

---

## Mappningstabell

| NIS2-krav | Vad krävs | Entra ID-kontroll | Labb | Status |
|-----------|-----------|-------------------|------|--------|
| Art. 21.2(a) – Riskanalys och säkerhetspolicyer | Dokumenterade policyer för åtkomstkontroll och identitetshantering | Entra ID Baseline + Zero Trust IAM Baseline | Lab 01, IAM Baseline | Klar |
| Art. 21.2(b) – Incidenthantering | Förmåga att detektera och rapportera säkerhetsincidenter kopplat till identiteter | Identity Protection – User Risk och Sign-in Risk policies | Lab 07 | Klar |
| Art. 21.2(e) – Nätverks- och systemsäkerhet | Kontroll av vem som har åtkomst till vilka system och under vilka villkor | Conditional Access – MFA, geoblockering, phishing-resistant MFA för admins | Lab 04 | Klar |
| Art. 21.2(i) – Flerfaktorsautentisering | MFA ska vara aktiverat, passwordless rekommenderas | Microsoft Authenticator, Number Matching, Passwordless Phone Sign-in | Lab 02 | Klar |
| Art. 21.2(i) – Privilegierade konton | Privilegierade konton ska vara tidsbegränsade och kräva godkännande | Privileged Identity Management (PIM) – Eligible roles, approval workflow | Lab 06 | Klar |
| Art. 21.2(a) – Hantering av externa parter | Externa användare ska ha begränsad åtkomst och granskas regelbundet | B2B External Identities och Access Reviews för gästanvändare | Lab 05 | Klar |
| Art. 21.2(a) – Åtkomstkontroll och JML | Rätt person ska ha rätt åtkomst vid rätt tidpunkt | Entitlement Management – Access Packages, godkännandeflöde, livscykelhantering | Lab 08 | Klar |
| Art. 21.2(b) – Revisionsspårning och loggning | Audit trail för alla åtkomsthändelser och administrativa åtgärder | Entra Audit Logs, Sign-in Logs, PIM Resource Audit, Graph API-automation | Lab 07, Lab 10 | Delvis |
| Art. 21.2(d) – Leveranskedjesäkerhet | Applikationer som integreras med identitetsplattformen ska vara säkert konfigurerade | App Registrations, OAuth2, SAML SSO – Least Privilege för appar | Lab 03, Lab 09 | Klar |
| Art. 21.2(e) – Least Privilege | Ingen ska ha mer åtkomst än vad som krävs för arbetsuppgiften | RBAC, Administrative Units, Eligible vs Active roller i PIM | Lab 01, Lab 06 | Klar |

---

## Kontroller som saknas eller är delvis implementerade

**Loggning och monitoring (delvis)**
Entra Audit Logs och Sign-in Logs finns och används. Log Analytics-workspace med KQL-queries mot Sentinel kräver Azure-prenumeration och är inte fullt konfigurerat i labbmiljön. Dokumenterat som planerat Lab 11.

**Hybrid Identity**
Om organisationen har on-premise Active Directory kräver NIS2 att Entra Connect Sync är konfigurerat och att identiteter synkroniseras kontrollerat. Kräver lokal AD-miljö och ingår inte i nuvarande labbuppsättning.

---

## Att tänka på vid kunduppdrag

NIS2 pensioneras den 18 oktober 2024 som direktiv och implementeras i nationell lagstiftning per land. I Sverige är Cybersäkerhetslagen det relevanta ramverket. Kraven i artikel 21 gäller oavsett hur lagen benämns nationellt.

Många organisationer har redan Entra ID P2-licenser men använder knappt hälften av funktionerna. En NIS2-gap-analys bör därför börja med att kartlägga vad som redan finns, inte vad som saknas.

---

## Genomförda labb

| Labb | Ämne | Täcker NIS2 |
|------|------|-------------|
| [Lab 01](../Labs/lab-01-tenant-setup.md) | Tenant Setup, RBAC, Least Privilege | Art. 21.2(a), 21.2(e) |
| [Lab 02](../Labs/lab-02-mfa-sspr-passwordless.md) | MFA, SSPR, Passwordless | Art. 21.2(i) |
| [Lab 03](../Labs/lab-03-app-registrations-oauth2.md) | App Registrations, OAuth2 | Art. 21.2(d) |
| [Lab 04](../Labs/lab-04-conditional-access.md) | Conditional Access | Art. 21.2(e), 21.2(i) |
| [Lab 05](../Labs/lab-05-b2b-external-identities.md) | B2B, External Identities, Access Reviews | Art. 21.2(a) |
| [Lab 06](../Labs/lab-06-pim.md) | PIM, Zero Standing Privilege | Art. 21.2(i) |
| [Lab 07](../Labs/lab-07-identity-protection.md) | Identity Protection, riskbaserade CA-policies | Art. 21.2(b) |
| [Lab 08](../Labs/lab-08-entitlement-management.md) | Entitlement Management, Access Packages | Art. 21.2(a) |
| [Lab 09](../Labs/lab-09-SAML-based-SSO.md) | SAML-baserad SSO | Art. 21.2(d) |
| [Lab 10](../Labs/lab-10-graph-api-automation.md) | Graph API, IAM-automation via PowerShell | Art. 21.2(b) |

---

## Källor

- [NIS2-direktivet (EU) 2022/2555](https://eur-lex.europa.eu/legal-content/SV/TXT/?uri=CELEX%3A32022L2555)
- [Microsoft Entra ID dokumentation](https://learn.microsoft.com/en-us/entra/identity/)
- [Zero Trust-ramverket, Microsoft](https://learn.microsoft.com/en-us/security/zero-trust/)
