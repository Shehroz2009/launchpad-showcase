# Overzicht van Launchpad-functies

Deze gids is een beknopt overzicht van de Launchpad starter kit voor potentiële
kopers. Het vat samen wat erbij inbegrepen is, wat externe services of
configuratie vereist, en wat de kit niet automatisch biedt. De volledige
functie-gids en broncode van het aangekochte project blijven de
autoritatieve implementatierferenties.

## Aankoop en updates

Launchpad ontvangt regelmatig updates. Bekijk de Gumroad-productpagina voor de actuele toegangsopties en aankoopvoorwaarden. Launchpad is software die je zelf host en beheert voor je eigen product, geen gehoste SaaS-dienst; hosting en kosten van externe providers zijn apart.

## Wat Launchpad bevat

Launchpad is een modulaire SaaS-toepassing gebouwd op één gedeelde codebase.
De optionele modules dekken facturatie, AI-chat, organisaties, publicatie,
ondersteuning, community, meldingen, e-mailmarketing, analytics, verwijzingen,
adverteren, SEO, integraties en privé-bestandsopslag. De configuratiewizard
laat u kiezen welke optionele modules u wilt behouden en een
implementatiedoel. De kerncomponenten voor accounts, beveiliging, instellingen
en de gedeelde database blijven aanwezig.

 Voor de implementatie moeten de app, database en providers afzonderlijk worden geconfigureerd.

Bij een Git-checkout gebruikt de configuratie sparse-checkout, zodat
uitgesloten bestanden die in Git worden bijgehouden, in de Git-geschiedenis
blijven. Dit is een
eenmalige configuratie, geen tool voor het wisselen van modules op de
bestaande locatie.

De wizard schrijft de geselecteerde modules en implementatievoorkeuren naar
`launchpad.config.json`, en provider-secrets naar het door Git genegeerde `.env`-bestand.
Het kan de geselecteerde provider-credentials controleren wanneer u expliciet
**Verbinding
testen** kiest. Het maakt geen provider-accounts aan, implementeert
geen infrastructuur, en configureert geen externe dashboards. U moet die
services nog steeds zelf voltooien en onderhouden.

Het plugin-gebied biedt een toegestane set ingebouwde componenten; het is
geen willekeurige pakketmarkt en downloadt of voert geen geüploade
plugincode uit. De optionele event-relay-service kan apart worden
geïmplementeerd voor het momenteel ondersteunde accountprofiel-updategebeurtenis.
Het is geen algemene serviceregister, message bus, of per-consument
bevestigingssysteem.

## Accounts en beveiliging

Authenticatie ondersteunt e-mailaanmelding en optionele Google- en
GitHub-aanmelding. In de ontwikkelomgeving kan Mailpit aanmeldingsmails
ontvangen wanneer er geen e-mailprovider is geconfigureerd. Geïmplementeerde
omgevingen hebben een e-mailprovider nodig voor e-mailaanmelding en
transactiemails. Beheerderspagina's en API's handhaven rollen op de server.

Optionele TOTP-meervoudige authenticatie werkt met compatibele
authenticator-apps en bevat eenmalige herstelcodes. Beveiligingsinstellingen
omvatten sessiebeheer en auditactiviteit. De kit bevat geen
wachtwoordgebaseerde aanmelding of een wachtwoordherstelstroom.

De account-onboarding-checklist dekt profielinstellingen, accountbeveiliging
en het dashboard; deze is optioneel en kan opnieuw worden bezocht.

## Facturatie en betaalde toegang

Launchpad ondersteunt abonnementen en eenmalige Lifetime-aankopen via Stripe, Paddle, Lemon Squeezy of PayPro Global; per product kies je één betaalprovider. Voor producten met native mobiele apps is ook verificatie van aankopen via Apple App Store en Google Play beschikbaar.

De kit verwerkt geverifieerde betaal- en abonnementsgebeurtenissen, registreert
betaalgeschiedenis, en behandelt terugbetalingen en geschillen. Het
**beveiligd** niet automatisch elke functie in uw product
alleen omdat facturatie is ingeschakeld.
`paymentBlocked` is een facturatiesignaal, geen universele paywall. Elke betaalde
functie moet toegang handhaven op de server; een knop verbergen in de browser
is niet voldoende. Beslis welke functies betaald zijn en wat beperkte
klanten nog steeds kunnen doen, en implementeer en test die regels. De kit
bevat ook geen kant-en-klaar beheerdersworkflow voor het verifiëren van
betalingen die buiten Stripe zijn gedaan.

Facturatie-e-mail gebruikt de geconfigureerde e-mailprovider, maar facturatie
vereist geen e-mail om te functioneren. E-mailbezorging kan vertraagd,
mislukt of gedupliceerd worden na een onzekere netwerkreactie; het is geen
exact-eens-bezorgingssysteem.

## E-mail, wachtlijst en marketing

De gedeelde e-mailtransport ondersteunt Resend, SendGrid en SMTP. Het wordt
gebruikt voor authenticatie, facturatiemeldingen, notificaties, nieuwsbrieven
en marketingberichten. Afzenderidentiteiten en provider-credentials moeten
worden geconfigureerd. De SMTP-check van de configuratiewizard authentificeert
zonder een bericht te verzenden.

De gelokaliseerde vroege-toegangswachtlijst vereist expliciete toestemming.
De statussen worden handmatig beheerd: iemand uitnodigen of als lid markeren
maakt geen account aan, verzendt geen uitnodiging, en verleent geen toegang.
Wachtlijsttoestemming is gescheiden van nieuwsbriefabonnement.

Het nieuwsbriefondersteunt toestemminggebaseerde inschrijvingen,
afmeldlinks, en door beheerders aangemaakte platte-tekstcampagnes.
Marketingautomatisering ondersteunt korte e-mailsequenties voor nieuwe
nieuwsbriefabonnees. Deze tools zijn geen visuele e-maileditor,
publieksegmentatiesysteem, of compliance/juridische dienst. Ze schrijven
wachtlijstinvoer niet automatisch in of verzenden berichten via push
notificaties en webhooks. Geplande campagnes en automatisering vereisen ook
een externe scheduler en een geconfigureerde e-mailprovider.

## AI-assistent en organisaties

De optionele dashboardassistent verbindt met een geconfigureerde
OpenAI-compatibele tekst-API. Het slaat gesprekken op in de
applicatiedatabase, handhaaft accountverzoeklimieten, en ondersteunt
persoonlijke en organisatiegesprekken. Het bevat geen model, voert geen
tools uit, en neemt geen autonome acties. Prompts en antwoorden worden naar
de geconfigureerde provider verzonden en door de applicatie opgeslagen;
configureer de privacy- en retentie-instellingen van de provider en
onthul deze verwerking aan gebruikers.

Organisaties bieden gedeelde werkruimtes, lidrollen en uitnodigingen.
Autorisatie wordt gecontroleerd tegenover organisatielidmaatschap, maar dit
is gedeelde-database-tenantie, geen aparte database of schema voor elke
klant. Facturatie, accountinstellingen, ondersteuning, meldingen, analytics,
nieuwsbrieven en globale publicatie-inhoud worden niet automatisch
organisatiegebonden.

## Publicatie, ondersteuning en community

Aparte modules bieden een gelokaliseerde Markdown-blog, release-updatefeed en
CMS-pagina's. Beheerders beheren concepten en gepubliceerde inhoud.
Openbare pagina's bevatten gelokaliseerde metadata en sitemap-ondersteuning.
De publicatietools bevatten geen geplande publicatie, automatische
vertaling, afbeeldingsuploads, of een commentaar-moderatiesysteem.

De ondersteuningsmodule biedt klanttickets en beheerdersantwoorden. De
communitymodule biedt een discussieforum. De geselekteerde directory laat
beheerders openbare items beheren. Dit zijn startfuncties in plaats van
integraties met een gehost helpdesk of een volledig sociaal netwerk:
bijlagen, toewijzing/SLA-automatisering, forumreacties, geaccepteerde
antwoorden en openbare gebruikersprofielen zijn niet inbegrepen.

## Meldingen en klantdata

Het dashboard heeft een permanente meldingsinbox en optionele e-mail, Web
Push, en ondertekende-webhook-bezorgingskanalen. Pushmeldingen vereisen VAPID
sleutels en browserabonnement; webhook-consumenten moeten handtekeningen
verifiëren en gebeurtenissen dedupliceren. Automatische herpogingen voor
meldingen, marketing en opschoningsopruiving vereisen extern geplande
taken.

Accountinstellingen bieden een JSON-export van ondersteunde accountdata en een
aparte link naar facturatiebeheer. De export sluit credentials, sessies,
ondertekeningssleutels en andere gevoelige tokens uit. Het is een
gestreamde download, geen volledige database-back-up of een
achtergrondarchiefdienst.

## Analytics, lokalisatie en aanpassing

Het dashboard bevat account- en facturatie-weergaven. Optionele product
analytics verzamelen toestemminggepoorte gebeurtenissen en bieden
beheersrapporten; ze zijn geen vervanging voor een derde-partij
analyticsplatform. Toeschrijving, marketingtracking en verwijzingscookies
kunnen toestemmingsverplichtingen hebben in uw implementatie. Launchpad
bevat geen toestemmingsbeheersplatform.

De interface ondersteunt 18 locaties: Engels, Arabisch, Bengaals, Duits,
Spaans, Frans, Hindi, Indonesisch, Italiaans, Japans, Koreaans, Nederlands,
Pools, Portugees, Russisch, Turks, Vietnamees en Chinees. Engels is de
fallback-catalogus en de fallback-taal van de configuratiewizard. Beoordeel
vertalingen en juridische tekst voor uw publiek vóór lancering.

Met de productbuilder kun je ondersteunde pagina-inhoud, navigatie en merkinstellingen aanpassen. In de bewegingseditor kunnen gebruikers ondersteunde animaties bekijken en aanpassen.

## Opslag, verwijzingen, adverteren en SEO

Optionele privé-bestandsopslag gebruikt S3-compatibele objectopslag. De
applicatie valideert bestandstypen en -groottes, gebruikt kortlevende
upload/download-URL's, en controleert bestandeigenaarschap. Configureer een
privé-bucket en de vereiste credentials. De module scant bestanden niet op
malware, en accountverwijdering verwijdert niet automatisch externe
objecten.

Met verwijzingslinks kunnen partners klanten naar je product brengen. Beloningen houden rekening met geschikte Stripe-betalingen en worden aangepast bij terugbetalingen of geschillen; uitbetalingen vinden buiten Launchpad plaats.

Advertentietools organiseren campagne- en accountinformatie en importeren
rapporten van CSV. Ze verbinden zich niet met advertenetwerken om live
campagnes aan te maken of te beheren. SEO-functies bieden gelokaliseerde
metadata, sitemaps, robotsregels, gestructureerde data en een
applicatiedatabase-audit; ze zijn geen webcrawler, rankingtracker,
zoektermenonderzoekstool, of redirectbeheerder.

Op de integratiepagina kun je ondersteunde diensten instellen en bepaalde verbindingen controleren.

## Implementatie en operationele verantwoordelijkheden

Implementaties vereisen PostgreSQL en passende omgevingsvariabelen,
waaronder databaseverbindingen en `AUTH_SECRET`. Configureer e-mail, Stripe,
winkelfacturatie, objectopslag, AI, push en geplande taken alleen voor de
functies die u inschakelt. Productiemigraties moeten worden toegepast met
gebruik van het gedocumenteerde implementatieproces; houd previewomgevingen
op een geïsoleerde database.

De applicatie bevat server-side validatie, rolcontroles,
beveiligingscontroles en auditlogboeking, maar het is geen volledig
beveiligingsoperatieprogramma. Het biedt geen WAF, malwarescanning, externe
beveiligingsmonitoring, of een strikte Content Security Policy. Configureer
een vertrouwde ingress en hostingbeveiligingen, beoordeel het
dreigingsmodel, en test de geïmplementeerde applicatie vóór lancering.

Voor gedetailleerde configuratie, data-eigendom, limieten, migraties en
operationele procedures, zie de volledige functie-gids die met het project
wordt geleverd.
