# Lab 01 – Entra Basics

## Syfte
Syftet med labben var att bygga praktisk grundförståelse för **Microsoft Entra ID** och centrala delar inom **Identity and Access Management (IAM)**. Fokus låg på att arbeta med användare, grupper, administrativa enheter, autentiseringsrelaterade inställningar och inbyggda administrativa roller för att förstå hur identiteter organiseras, hur åtkomst kan struktureras och hur privilegier bör hanteras i en modern molnbaserad identitetsplattform.

## Miljö
Labben genomfördes i en egen **testtenant i Microsoft Entra**, vilket gjorde det möjligt att arbeta i en säker och kontrollerad labbmiljö utan påverkan på produktion.

## Steg jag gjorde
Jag började med att skapa fem testanvändare i **Identity → Users → All users** för att simulera olika funktioner i en organisation:

- **Anna HR**
- **Erik IT**
- **Sara Sales**
- **Admin Test**
- **Temp Consultant**

Därefter skapade jag fyra **Security Groups** i **Identity → Groups → All groups** och lade till användare utifrån funktion och användningsområde:

- **HR-Users** → Anna HR
- **IT-Admins** → Erik IT, Admin Test
- **MFA-Pilot** → Temp Consultant
- **Sales-Users** → Sara Sales

Denna struktur användes för att efterlikna hur en organisation kan gruppera identiteter baserat på avdelning, administrativ funktion och säkerhetsinitiativ, exempelvis införande av MFA i en pilotgrupp.

Som nästa steg gick jag till **Identity → Administrative units** och skapade en **Administrative Unit** med namnet **Students**. Jag lade därefter till två testanvändare i denna administrativa enhet för att förstå hur administration kan avgränsas till en viss del av katalogen.

Jag gick sedan till **Identity → Roles & administrators**, där jag granskade följande inbyggda roller:

- **Global Administrator**
- **User Administrator**
- **Security Administrator**
- **Privileged Role Administrator**

I rollgranskningen fokuserade jag på vilket ansvar rollen verkar ha, varför rollen är säkerhetsmässigt viktig eller känslig, och om rollen framstår som bred eller mer avgränsad.

**Global Administrator** framstod som den bredaste och mest privilegierade rollen, eftersom den kan hantera i princip alla delar av Microsoft Entra ID och andra Microsoft-tjänster som använder Entra-identiteter. Rollen är därför mycket känslig ur ett säkerhetsperspektiv.

**User Administrator** framstod som en bred men mer avgränsad roll, med fokus på att hantera användare och grupper samt vissa lösenordsåterställningar. Rollen är viktig eftersom den påverkar identiteter direkt och därmed också organisationens åtkomsthantering.

**Security Administrator** framstod som en specialiserad men kraftfull roll, eftersom den ger åtkomst till säkerhetsinformation, rapporter och vissa säkerhetsrelaterade konfigurationer i Entra ID och Microsoft 365. Rollen är viktig eftersom den påverkar miljöns säkerhetsstyrning.

**Privileged Role Administrator** framstod som en smalare men mycket känslig roll, eftersom den hanterar rolltilldelningar i Microsoft Entra ID samt funktioner kopplade till **Privileged Identity Management (PIM)**. Rollen är särskilt viktig eftersom den påverkar vem som får administrativa privilegier.

Avslutningsvis gick jag till **Password reset → Properties** och verifierade att **Self-Service Password Reset** var satt till **None** för vanliga slutanvändare. Jag noterade samtidigt att administratörer ändå är aktiverade för self-service password reset och att de måste använda två autentiseringsmetoder vid lösenordsåterställning.

Jag gick därefter till **Authentication methods → Policies** och granskade vilka autentiseringsmetoder som var aktiverade respektive inaktiverade i tenantet. Jag noterade att **Microsoft Authenticator**, **Temporary Access Pass**, **Software OATH tokens** och **Email OTP** var aktiverade för **all users**.

## Resultat
Labben resulterade i en fungerande testmiljö där användare, grupper, gruppmedlemskap och en administrativ enhet kunde verifieras korrekt i Microsoft Entra-portalen. Genom att skapa en enkel identitetsstruktur med tydliga användartyper och gruppindelningar fick jag praktisk erfarenhet av hur Entra kan användas för att organisera identiteter på ett strukturerat sätt.

Jag fick också en tydligare förståelse för skillnaden mellan **grupper**, **roller** och **Administrative Units**. Grupper används för att samla användare och förenkla åtkomsthantering, roller används för att ge administrativa privilegier och Administrative Units används för att avgränsa administration till en viss del av katalogen.

Granskningen av de inbyggda rollerna gav dessutom en bättre bild av hur olika administrativa roller innebär olika nivåer av ansvar, risk och privilegier. Genom att granska **Password reset** och **Authentication methods** kunde jag även se hur autentiseringsmetoder och återställningsinställningar påverkar säkerheten i identitetsmiljön.

## Säkerhetsreflektion
Labben visade tydligt att **grupper, administrativa roller och administrativa enheter inte är samma sak**, även om de alla påverkar hur identiteter hanteras i Microsoft Entra. Grupper används främst för att organisera användare och förenkla åtkomsthantering, roller innebär förhöjda privilegier och Administrative Units används för att avgränsa administration.

En viktig insikt var varför **Global Administrator** bör begränsas så mycket som möjligt. Rollen har mycket bred åtkomst och kan påverka stora delar av miljön. Om ett sådant konto komprometteras, eller används felaktigt, kan konsekvenserna bli omfattande. Därför bör den här typen av roll endast tilldelas till ett fåtal konton med tydligt behov.

Labben visade även varför **grupper ofta är bättre än direkt tilldelning av rättigheter till enskilda användare**. När åtkomst styrs via grupper blir administrationen mer strukturerad, enklare att följa upp och mer skalbar över tid. Det blir också lättare att lägga till eller ta bort användare utan att behöva hantera varje behörighet manuellt.

Granskningen av **Password reset** och **Authentication methods** visade dessutom att säker identitetshantering inte bara handlar om att skapa användare och grupper, utan också om hur användare får verifiera sig och återställa sina lösenord. Det blev tydligt att olika autentiseringsmetoder innebär olika säkerhetsnivåer och att administrativa konton kräver starkare skydd än vanliga användare.

## Rekommendation till kund
Jag rekommenderar att organisationer bygger sin åtkomstmodell med **grupper som grund**, där användare organiseras utifrån funktion, avdelning eller säkerhetsbehov. Det är normalt bättre än att tilldela rättigheter direkt till enskilda användare, eftersom det ger bättre struktur, enklare administration och bättre kontroll över vem som har åtkomst till vad.

Jag rekommenderar också att administrativa roller hanteras separat och mycket restriktivt. Särskilt roller som **Global Administrator** bör hållas få och endast tilldelas när det finns ett tydligt verksamhetsbehov. Högt privilegierade roller bör följas upp regelbundet och omfattas av tydliga säkerhetsrutiner för att minska risken för överprivilegierade konton och felaktig åtkomst.

För större eller mer komplexa miljöer kan det även vara lämpligt att använda **Administrative Units** för att avgränsa administration mellan olika delar av organisationen.

## Vad jag lärde mig
- hur användare skapas och administreras i Microsoft Entra
- hur grupper kan användas för att strukturera identiteter utifrån funktion och behov
- hur gruppmedlemskap kan stödja en mer kontrollerad och skalbar åtkomstmodell
- skillnaden mellan en grupp, en administrativ roll och en Administrative Unit
- varför **Global Administrator** är en mycket känslig roll som bör begränsas
- varför gruppbaserad åtkomst ofta är bättre än direkt tilldelning till enskilda användare
- hur inbyggda roller i Entra skiljer sig åt i ansvar, känslighet och privilegienivå
- varför **least privilege** är en grundprincip inom säker IAM-styrning
- att **Self-Service Password Reset** kan vara olika konfigurerat för vanliga användare och administratörer
- att administratörer ofta har striktare säkerhetskrav vid lösenordsåterställning
- att **Authentication methods** styr vilka sätt användare får använda för att autentisera sig
- att **Microsoft Authenticator** och **Temporary Access Pass** verkar vara viktiga metoder i moderna Entra-miljöer