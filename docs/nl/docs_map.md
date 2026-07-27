---
read_when: Finding which docs page covers a topic before reading the page
summary: Gegenereerde koppenkaart voor OpenClaw-documentatiepagina's
title: Documentatieoverzicht
x-i18n:
    generated_at: "2026-07-27T05:09:15Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 0b58762e88df339b48cd4d15bd5c6e8490c278ed78acc5f50c415649cb7f2719
    source_path: docs_map.md
    workflow: 16
---

# OpenClaw-documentatieoverzicht

Dit bestand wordt gegenereerd op basis van de koppen `docs/**/*.md` en `docs/**/*.mdx` om agents te helpen navigeren door de documentatiestructuur.
Bewerk het niet handmatig; voer `pnpm docs:map:gen` uit.

## agent-runtime-architecture.md

- Route: /agent-runtime-architecture
- Koppen:
  - H2: Runtime-indeling
  - H2: Grenzen
  - H2: Manifesten
  - H2: Runtimeselectie
  - H2: Generaties van modelruntimes
  - H2: Gerelateerd

## announcements/bluebubbles-imessage.md

- Route: /announcements/bluebubbles-imessage
- Koppen:
  - H1: Verwijdering van BlueBubbles en het imsg-pad voor iMessage
  - H2: Wat is gewijzigd
  - H2: Wat je moet doen
  - H2: Migratieopmerkingen
  - H2: Zie ook

## auth-credential-semantics.md

- Route: /auth-credential-semantics
- Koppen:
  - H2: Stabiele redencodes voor controles
  - H2: Tokenreferenties
  - H3: Geschiktheidsregels
  - H3: Resolutieregels
  - H2: Overdraagbaarheid van agentkopieën
  - H2: Alleen-configuratieroutes voor authenticatie
  - H2: Expliciete filtering van de authenticatievolgorde
  - H2: Resolutie van controledoelen
  - H2: Detectie van externe CLI-referenties
  - H2: Beleidsbeveiliging voor OAuth SecretRef
  - H2: Berichtenuitwisseling met ondersteuning voor verouderde versies
  - H2: Gerelateerd

## automation/auth-monitoring.md

- Route: /automation/auth-monitoring
- Koppen:
  - H2: Gerelateerd

## automation/clawflow.md

- Route: /automation/clawflow
- Koppen:
  - H2: Gerelateerd

## automation/cron-jobs.md

- Route: /automation/cron-jobs
- Koppen:
  - H2: Snel aan de slag
  - H2: Hoe Cron werkt
  - H2: Schematypen
  - H3: Migratie van Heartbeat-taken
  - H3: Streambronnen
  - H3: Dynamisch tempo
  - H3: Dag van de maand en dag van de week gebruiken OF-logica
  - H2: Gebeurtenistriggers (voorwaardebewakers)
  - H2: Payloads
  - H3: Opties voor agentbeurten
  - H3: Opdrachtpayloads
  - H3: Scriptpayloads
  - H2: Uitvoeringsstijlen
  - H2: Levering en uitvoer
  - H3: Foutmeldingen
  - H3: Uitvoertaal
  - H2: CLI-voorbeelden
  - H2: Taken beheren
  - H2: Webhooks
  - H3: Authenticatie
  - H2: Gmail PubSub-integratie
  - H3: Installatie met wizard (aanbevolen)
  - H3: Gateway automatisch starten
  - H3: Eenmalige handmatige installatie
  - H3: Gmail-model overschrijven
  - H2: Configuratie
  - H2: Probleemoplossing
  - H3: Opdrachtladder
  - H2: Gerelateerd

## automation/cron-vs-heartbeat.md

- Route: /automation/cron-vs-heartbeat
- Koppen:
  - H2: Gerelateerd

## automation/gmail-pubsub.md

- Route: /automation/gmail-pubsub
- Koppen:
  - H2: Gerelateerd

## automation/hooks.md

- Route: /automation/hooks
- Koppen:
  - H2: Kies het juiste oppervlak
  - H2: Snel aan de slag
  - H2: Gebeurtenistypen
  - H2: Hooks schrijven
  - H3: Hookstructuur
  - H3: HOOK.md-indeling
  - H3: Implementatie van de handler
  - H3: Belangrijkste punten van de gebeurteniscontext
  - H2: Hookdetectie
  - H3: Hookpakketten
  - H2: Meegeleverde hooks
  - H3: Details van session-memory
  - H3: Configuratie van bootstrap-extra-files
  - H3: Details van command-logger
  - H3: Details van compaction-notifier
  - H3: Details van boot-md
  - H2: Plugin-hooks
  - H2: Configuratie
  - H2: CLI-referentie
  - H2: Aanbevolen werkwijzen
  - H2: Probleemoplossing
  - H3: Hook niet gedetecteerd
  - H3: Hook niet geschikt
  - H3: Hook wordt niet uitgevoerd
  - H2: Gerelateerd

## automation/index.md

- Route: /automation
- Koppen:
  - H2: Snelle keuzehulp
  - H3: Geplande taken (Cron) versus Heartbeat
  - H2: Kernconcepten
  - H3: Geplande taken (Cron)
  - H3: Taken
  - H3: TaskFlow
  - H3: Doorlopende opdrachten
  - H3: Hooks
  - H3: Heartbeat
  - H2: Hoe ze samenwerken
  - H2: Gerelateerd

## automation/poll.md

- Route: /automation/poll
- Koppen:
  - H2: Gerelateerd

## automation/standing-orders.md

- Route: /automation/standing-orders
- Koppen:
  - H2: Waarom doorlopende opdrachten
  - H2: Hoe ze werken
  - H2: Anatomie van een doorlopende opdracht
  - H2: Doorlopende opdrachten plus Cron-taken
  - H2: Voorbeelden
  - H3: Voorbeeld 1: content en sociale media (wekelijkse cyclus)
  - H3: Voorbeeld 2: financiële activiteiten (gebeurtenisgestuurd)
  - H3: Voorbeeld 3: bewaking en waarschuwingen (continu)
  - H2: Patroon uitvoeren-verifiëren-rapporteren
  - H2: Architectuur met meerdere programma's
  - H2: Aanbevolen werkwijzen
  - H3: Wel doen
  - H3: Vermijden
  - H2: Gerelateerd

## automation/taskflow.md

- Route: /automation/taskflow
- Koppen:
  - H2: Wanneer je TaskFlow gebruikt
  - H2: Synchronisatiemodi
  - H3: Beheerde modus
  - H3: Gespiegelde modus
  - H2: Flowstatussen
  - H2: Duurzame status en revisietracking
  - H2: Annuleringsgedrag
  - H2: CLI-opdrachten
  - H2: Betrouwbaar patroon voor geplande workflows
  - H2: Hoe flows zich tot taken verhouden
  - H2: Gerelateerd

## automation/tasks.md

- Route: /automation/tasks
- Koppen:
  - H2: Kort samengevat
  - H2: Snel aan de slag
  - H2: Wat een taak aanmaakt
  - H2: Levenscyclus van een taak
  - H2: Levering en meldingen
  - H3: Meldingsbeleid
  - H2: CLI-referentie
  - H2: Taakbord in de chat (/tasks)
  - H3: Bedieningsinterface
  - H2: Statusintegratie (taakdruk)
  - H2: Opslag en onderhoud
  - H3: Waar taken zich bevinden
  - H3: Automatisch onderhoud
  - H2: Hoe taken zich tot andere systemen verhouden
  - H2: Gerelateerd

## automation/troubleshooting.md

- Route: /automation/troubleshooting
- Koppen:
  - H2: Gerelateerd

## automation/webhook.md

- Route: /automation/webhook
- Koppen:
  - H2: Gerelateerd

## brave-search.md

- Route: /brave-search
- Koppen:
  - H2: Gerelateerd

## channels/access-groups.md

- Route: /channels/access-groups
- Koppen:
  - H2: Statische groepen van berichtafzenders
  - H2: Referentiegroepen uit toelatingslijsten
  - H2: Ondersteunde paden voor berichtkanalen
  - H2: Doelgroepen van Discord-kanalen
  - H2: Plugin-diagnostiek
  - H2: Beveiligingsopmerkingen
  - H2: Probleemoplossing

## channels/ambient-room-events.md

- Route: /channels/ambient-room-events
- Koppen:
  - H2: Aanbevolen configuratie
  - H2: Wat verandert
  - H2: Discord-voorbeeld
  - H2: Slack-voorbeeld
  - H2: Telegram-voorbeeld
  - H2: Agentspecifiek beleid
  - H2: Zichtbare antwoordmodi
  - H2: Geschiedenis
  - H2: Probleemoplossing
  - H2: Gerelateerd

## channels/bot-loop-protection.md

- Route: /channels/bot-loop-protection
- Koppen:
  - H2: Standaardwaarden
  - H2: Gedeelde standaardwaarden configureren
  - H2: Overschrijven per kanaal, account of ruimte
  - H2: Kanaalondersteuning

## channels/broadcast-groups.md

- Route: /channels/broadcast-groups
- Koppen:
  - H2: Overzicht
  - H2: Configuratie
  - H3: Basisconfiguratie
  - H3: Verwerkingsstrategie
  - H3: Volledig voorbeeld
  - H2: Hoe het werkt
  - H3: Berichtenstroom
  - H3: Sessie-isolatie
  - H3: Voorbeeld: geïsoleerde sessies
  - H2: Gebruiksscenario's
  - H2: Aanbevolen werkwijzen
  - H2: Compatibiliteit
  - H3: Providers
  - H3: Routering
  - H2: Probleemoplossing
  - H2: Voorbeelden
  - H2: API-referentie
  - H3: Configuratieschema
  - H3: Velden
  - H2: Beperkingen
  - H2: Gerelateerd

## channels/channel-routing.md

- Route: /channels/channel-routing
- Koppen:
  - H1: Kanalen en routering
  - H2: Belangrijke termen
  - H2: Voorvoegsels voor uitgaande doelen
  - H2: Vormen van sessiesleutels (voorbeelden)
  - H2: Vastzetten van de hoofdroute voor directe berichten
  - H2: Beveiligde registratie van inkomende berichten
  - H2: Routeringsregels (hoe een agent wordt gekozen)
  - H2: Uitzendgroepen (meerdere agents uitvoeren)
  - H2: Configuratieoverzicht
  - H2: Sessieopslag
  - H2: WebChat-gedrag
  - H2: Antwoordcontext
  - H2: Gerelateerd

## channels/clickclack.md

- Route: /channels/clickclack
- Koppen:
  - H2: Snelle configuratie
  - H3: Alternatief: handmatig token
  - H3: Alternatief: omgevingsvariabele voor token
  - H3: JSON5-referentie
  - H3: Configuratiesleutels voor accounts
  - H3: Een openbaar hostnaam met authenticatiebeveiliging behouden
  - H2: Meerdere bots
  - H2: Sessiebesprekingen
  - H2: Antwoordmodi
  - H2: Opdrachtmenu
  - H2: Duurzame medialevering
  - H2: Rijen met agentactiviteit
  - H2: Doelen
  - H2: Machtigingen
  - H2: Probleemoplossing

## channels/discord-activities.md

- Route: /channels/discord-activities
- Koppen:
  - H2: Vereisten
  - H2: Configuratie
  - H2: Beveiligingsmodel
  - H2: Probleemoplossing
  - H3: De activiteit meldt “Gateway offline”
  - H3: Discord opent een lege pagina of meldt blocked:csp
  - H3: “Widget niet beschikbaar”
  - H3: “Je kunt geen activiteiten starten in dit kanaal”

## channels/discord.md

- Route: /channels/discord
- Koppen:
  - H2: Snelle installatie
  - H2: Aanbevolen: een guild-werkruimte instellen
  - H2: Runtimemodel
  - H2: Forumkanalen
  - H2: Interactieve componenten
  - H2: Toegangsbeheer en routering
  - H3: Rolgebaseerde agentroutering
  - H2: Systeemeigen opdrachten en opdrachtauthenticatie
  - H2: Functiedetails
  - H2: Tools en actiepoorten
  - H2: Components v2-gebruikersinterface
  - H2: Spraak
  - H3: Spraakkanalen
  - H3: Gebruikers volgen in spraakkanalen
  - H3: Spraakberichten
  - H2: Probleemoplossing
  - H2: Configuratiereferentie
  - H3: Discord Activities
  - H2: Veiligheid en beheer
  - H2: Gerelateerd

## channels/feishu.md

- Route: /channels/feishu
- Koppen:
  - H2: Snel aan de slag
  - H2: Duurzaamheid van inkomende berichten
  - H2: Toegangsbeheer
  - H3: Privéberichten
  - H3: Groepschats
  - H2: Voorbeelden van groepsconfiguraties
  - H3: Alle groepen toestaan, geen @vermelding vereist
  - H3: Alle groepen toestaan, maar nog steeds een @vermelding vereisen
  - H3: Alleen specifieke groepen toestaan
  - H3: Afzenders binnen een groep beperken
  - H3: Door bots geschreven berichten
  - H2: Groeps-/gebruikers-ID's ophalen
  - H3: Groeps-ID's (`chat_id`, indeling: `oc_xxx`)
  - H3: Gebruikers-ID's (`open_id`, indeling: `ou_xxx`)
  - H2: Veelgebruikte opdrachten
  - H2: Probleemoplossing
  - H3: Bot reageert niet in groepschats
  - H3: Bot ontvangt geen berichten
  - H3: QR-configuratie reageert niet in de mobiele Feishu-app
  - H3: App Secret is uitgelekt
  - H2: Geavanceerde configuratie
  - H3: Meerdere accounts
  - H3: Berichtlimieten
  - H3: Streaming
  - H3: Quotumoptimalisatie
  - H3: Bereik van groepssessies en onderwerpthreads
  - H3: Tools voor Feishu-werkruimten
  - H3: ACP-sessies
  - H4: Permanente ACP-koppeling
  - H4: ACP starten vanuit een chat
  - H3: Routering met meerdere agents
  - H2: Agentisolatie per gebruiker (dynamisch agents aanmaken)
  - H3: Snelle installatie
  - H3: Hoe het werkt
  - H3: Configuratieopties
  - H3: Sessiebereik
  - H3: Typische implementatie voor meerdere gebruikers
  - H3: Verificatie
  - H3: Opmerkingen
  - H2: Configuratiereferentie
  - H2: Ondersteunde berichttypen
  - H3: Ontvangen
  - H3: Verzenden
  - H3: Threads en antwoorden
  - H2: Gerelateerd

## channels/googlechat.md

- Route: /channels/googlechat
- Koppen:
  - H2: Installeren
  - H2: Snelle installatie (beginners)
  - H2: Toevoegen aan Google Chat
  - H2: Openbare URL (alleen Webhook)
  - H3: Optie A: Tailscale Funnel (aanbevolen)
  - H3: Optie B: reverse proxy (Caddy)
  - H3: Optie C: Cloudflare Tunnel
  - H2: Hoe het werkt
  - H3: Duurzaamheid van inkomende berichten
  - H2: Doelen
  - H2: Belangrijkste configuratiepunten
  - H2: Probleemoplossing
  - H3: 405 Methode niet toegestaan
  - H3: Andere problemen
  - H2: Gerelateerd

## channels/group-messages.md

- Route: /channels/group-messages
- Koppen:
  - H2: Gedrag
  - H2: Configuratievoorbeeld (WhatsApp)
  - H3: Activeringsopdracht (alleen eigenaar)
  - H2: Gebruik
  - H2: Testen/verificatie
  - H2: Bekende aandachtspunten
  - H2: Gerelateerd

## channels/groups.md

- Route: /channels/groups
- Koppen:
  - H2: Introductie voor beginners (2 minuten)
  - H2: Zichtbare antwoorden
  - H2: Contextzichtbaarheid en toelatingslijsten
  - H2: Sessiesleutels
  - H2: Patroon: persoonlijke privéberichten + openbare groepen (één agent)
  - H2: Weergavelabels
  - H2: Groepsbeleid
  - H2: Poortwachter op basis van vermeldingen (standaard)
  - H2: Vermeldingspatronen configureren per bereik
  - H2: Toolbeperkingen voor groepen/kanalen (optioneel)
  - H2: Toelatingslijsten voor groepen
  - H2: Activering (alleen eigenaar)
  - H2: Contextvelden
  - H2: Specifieke kenmerken van iMessage
  - H2: Systeemprompts van WhatsApp
  - H2: Specifieke kenmerken van WhatsApp
  - H2: Gerelateerd

## channels/imessage-from-bluebubbles.md

- Route: /channels/imessage-from-bluebubbles
- Koppen:
  - H2: Migratiechecklist
  - H2: Wat imsg doet
  - H2: Voordat je begint
  - H2: Configuratievertaling
  - H2: Valkuil in het groepsregister
  - H2: Stap voor stap
  - H2: Actiepariteit in één oogopslag
  - H2: Koppelen, sessies en ACP-koppelingen
  - H2: Geen terugdraaikanaal
  - H2: Gerelateerd

## channels/imessage.md

- Route: /channels/imessage
- Koppen:
  - H2: Snelle installatie
  - H2: Vereisten en machtigingen (macOS)
  - H2: De privé-API van imsg inschakelen
  - H3: Configuratie
  - H3: Wanneer SIP ingeschakeld blijft
  - H2: Toegangsbeheer en routering
  - H2: ACP-gesprekskoppelingen
  - H2: Implementatiepatronen
  - H2: Media, opsplitsing en bezorgdoelen
  - H2: Acties van de privé-API
  - H2: Configuratieschrijfbewerkingen
  - H2: Opgesplitst verzonden privéberichten samenvoegen (opdracht + URL in één samenstelling)
  - H2: Herstel van inkomende berichten na het opnieuw starten van een bridge of Gateway
  - H3: Voor de beheerder zichtbaar signaal
  - H3: Migratie
  - H2: Probleemoplossing
  - H2: Verwijzingen naar de configuratiereferentie
  - H2: Gerelateerd

## channels/index.md

- Route: /channels
- Koppen:
  - H2: Ondersteunde kanalen
  - H2: Opmerkingen over bezorging
  - H2: Opmerkingen

## channels/irc.md

- Route: /channels/irc
- Koppen:
  - H2: Snel aan de slag
  - H2: Duurzaamheid van inkomende berichten
  - H2: Verbindingsinstellingen
  - H2: Standaardbeveiligingsinstellingen
  - H2: Toegangsbeheer
  - H3: Veelvoorkomende valkuil: allowFrom is voor privéberichten, niet voor kanalen
  - H2: Antwoorden activeren (vermeldingen)
  - H2: Beveiligingsopmerking (aanbevolen voor openbare kanalen)
  - H3: Dezelfde tools voor iedereen in het kanaal
  - H3: Verschillende tools per afzender (de eigenaar krijgt meer bevoegdheden)
  - H2: NickServ
  - H2: Omgevingsvariabelen
  - H2: Probleemoplossing
  - H2: Gerelateerd

## channels/line.md

- Route: /channels/line
- Koppen:
  - H2: Installeren
  - H2: Instellen
  - H2: Configureren
  - H2: Toegangsbeheer
  - H2: Berichtgedrag
  - H2: Kanaalgegevens (rijke berichten)
  - H2: ACP-ondersteuning
  - H2: Uitgaande media
  - H2: Probleemoplossing
  - H2: Gerelateerd

## channels/location.md

- Route: /channels/location
- Koppen:
  - H2: Tekstopmaak
  - H2: Contextvelden
  - H2: Uitgaande payloads
  - H2: Kanaalopmerkingen
  - H2: Gerelateerd

## channels/matrix-migration.md

- Route: /channels/matrix-migration
- Koppen:
  - H2: Wat de migratie automatisch doet
  - H2: Upgraden vanaf OpenClaw-releases ouder dan 2026.4
  - H2: Aanbevolen upgradeproces
  - H2: Veelvoorkomende berichten en wat ze betekenen
  - H3: Berichten voor handmatig herstel
  - H2: Als versleutelde geschiedenis nog steeds niet terugkomt
  - H2: Als je opnieuw wilt beginnen voor toekomstige berichten
  - H2: Gerelateerd

## channels/matrix-presentation.md

- Route: /channels/matrix-presentation
- Koppen:
  - H2: Gebeurtenisinhoud
  - H2: Terugvalgedrag
  - H2: Ondersteunde blokken
  - H2: Interacties
  - H2: Relatie tot goedkeuringsmetadata
  - H2: Mediaberichten

## channels/matrix-push-rules.md

- Route: /channels/matrix-push-rules
- Koppen:
  - H2: Vereisten
  - H2: Stappen
  - H2: Opmerkingen voor meerdere bots
  - H2: Opmerkingen over de homeserver
  - H2: Gerelateerd

## channels/matrix.md

- Route: /channels/matrix
- Koppen:
  - H2: Installeren
  - H2: Instellen
  - H3: Interactieve configuratie
  - H3: Minimale configuratie
  - H3: Automatisch deelnemen
  - H3: Doelindelingen voor toelatingslijsten
  - H3: Normalisatie van account-ID's
  - H3: Gecachte aanmeldgegevens
  - H3: Omgevingsvariabelen
  - H2: Configuratievoorbeeld
  - H2: Streamingvoorbeelden
  - H2: Spraakberichten
  - H2: Goedkeuringsmetadata
  - H3: Zelfgehoste pushregels voor stille definitieve voorbeelden
  - H2: Bot-naar-botruimten
  - H2: Versleuteling en verificatie
  - H3: Versleuteling inschakelen
  - H3: Status- en vertrouwenssignalen
  - H3: Dit apparaat verifiëren met een herstelsleutel
  - H3: Cross-signing initialiseren of herstellen
  - H3: Back-up van ruimtesleutels
  - H3: Verificaties weergeven, aanvragen en beantwoorden
  - H3: Opmerkingen voor meerdere accounts
  - H2: Profielbeheer
  - H2: Threads
  - H3: Sessieroutering (sessionScope)
  - H3: Antwoorden in threads (threadReplies)
  - H3: Overerving van threads en slash-opdrachten
  - H2: ACP-gesprekskoppelingen
  - H3: Configuratie van threadkoppelingen
  - H2: Reacties
  - H2: Geschiedeniscontext
  - H2: Contextzichtbaarheid
  - H2: Beleid voor privéberichten en ruimten
  - H2: Herstel van directe ruimten
  - H2: Uitvoeringsgoedkeuringen
  - H2: Slash-opdrachten
  - H2: Meerdere accounts
  - H2: Privé-/LAN-homeservers
  - H2: Matrix-verkeer via een proxy leiden
  - H2: Doelresolutie
  - H2: Configuratiereferentie
  - H3: Account en verbinding
  - H3: Versleuteling
  - H3: Toegang en beleid
  - H3: Antwoordgedrag
  - H3: Reactie-instellingen
  - H3: Tools en overschrijvingen per ruimte
  - H3: Instellingen voor uitvoeringsgoedkeuring
  - H2: Gerelateerd

## channels/mattermost.md

- Route: /channels/mattermost
- Koppen:
  - H2: Installatie
  - H2: Snelle configuratie
  - H2: Systeemeigen slashopdrachten
  - H2: Omgevingsvariabelen (standaardaccount)
  - H2: Chatmodi
  - H2: Threads en sessies
  - H2: Toegangsbeheer (DM's)
  - H2: Kanalen (groepen)
  - H2: Doelen voor uitgaande bezorging
  - H2: Opnieuw proberen voor DM-kanaal
  - H2: Previewstreaming
  - H2: Reacties (berichtentool)
  - H2: Interactieve knoppen (berichtentool)
  - H3: Directe API-integratie (externe scripts)
  - H2: Directory-adapter
  - H2: Meerdere accounts
  - H2: Probleemoplossing
  - H2: Gerelateerd

## channels/msteams.md

- Route: /channels/msteams
- Koppen:
  - H2: Meegeleverde plugin
  - H2: Snelle configuratie
  - H2: Doelen
  - H2: Configuratie schrijven
  - H2: Toegangsbeheer (DM's + groepen)
  - H3: Hoe het werkt
  - H3: Stap 1: Azure Bot maken
  - H3: Stap 2: Referenties verkrijgen
  - H3: Stap 3: Berichtenendpoint configureren
  - H3: Stap 4: Teams-kanaal inschakelen
  - H3: Stap 5: Teams-appmanifest bouwen
  - H3: Stap 6: OpenClaw configureren
  - H3: Stap 7: De Gateway uitvoeren
  - H2: Federatieve authenticatie (certificaat plus beheerde identiteit)
  - H3: Optie A: Authenticatie op basis van certificaten
  - H3: Optie B: Door Azure beheerde identiteit
  - H3: AKS Workload Identity configureren
  - H3: Vergelijking van authenticatietypen
  - H2: Lokale ontwikkeling (tunneling)
  - H2: De bot testen
  - H2: Omgevingsvariabelen
  - H2: Actie voor ledeninformatie
  - H2: Geschiedeniscontext
  - H2: Huidige Teams RSC-machtigingen (manifest)
  - H2: Voorbeeld van Teams-manifest (geredigeerd)
  - H3: Aandachtspunten voor het manifest (verplichte velden)
  - H3: Een bestaande app bijwerken
  - H2: Mogelijkheden: alleen RSC versus Graph
  - H3: Met alleen Teams RSC (app geïnstalleerd, geen Graph API-machtigingen)
  - H3: Met Teams RSC + Microsoft Graph-toepassingsmachtigingen
  - H3: RSC versus Graph API
  - H2: Media + geschiedenis met Graph
  - H3: Bestandsherstel voor kanalen/groepen (graphMediaFallback)
  - H2: Bekende beperkingen
  - H3: Webhook-time-outs
  - H3: Ondersteuning voor Teams-cloud en service-URL
  - H3: Opmaak
  - H2: Configuratie
  - H2: Routering en sessies
  - H2: Antwoordstijl: threads versus berichten
  - H3: Voorrangsvolgorde bij bepaling
  - H3: Behoud van threadcontext
  - H2: Bijlagen en afbeeldingen
  - H2: Bestanden verzenden in groepschats
  - H3: Waarom groepschats SharePoint nodig hebben
  - H3: Configuratie
  - H3: Deelgedrag
  - H3: Terugvalgedrag
  - H3: Opslaglocatie van bestanden
  - H2: Peilingen (Adaptive Cards)
  - H2: Presentatiekaarten
  - H2: Doelindelingen
  - H2: Proactieve berichten
  - H2: Team- en kanaal-ID's (veelvoorkomende valkuil)
  - H2: Privékanalen
  - H2: Probleemoplossing
  - H3: Veelvoorkomende problemen
  - H3: Fouten bij het uploaden van het manifest
  - H3: RSC-machtigingen werken niet
  - H2: Referenties
  - H2: Gerelateerd

## channels/nextcloud-talk.md

- Route: /channels/nextcloud-talk
- Koppen:
  - H2: Installatie
  - H2: Snelle configuratie (beginners)
  - H2: Opmerkingen
  - H2: Toegangsbeheer (DM's)
  - H2: Ruimten (groepen)
  - H2: Mogelijkheden
  - H2: Configuratiereferentie (Nextcloud Talk)
  - H2: Gerelateerd

## channels/nostr.md

- Route: /channels/nostr
- Koppen:
  - H2: Installatie
  - H3: Niet-interactieve configuratie
  - H2: Snelle configuratie
  - H2: Configuratiereferentie
  - H2: Profielmetadata
  - H2: Toegangsbeheer
  - H3: DM-beleid
  - H3: Voorbeeld van een toelatingslijst
  - H2: Sleutelindelingen
  - H2: Relays
  - H2: Protocolondersteuning
  - H2: Testen
  - H3: Lokale relay
  - H3: Handmatige test
  - H2: Probleemoplossing
  - H3: Geen berichten ontvangen
  - H3: Geen antwoorden verzenden
  - H3: Dubbele antwoorden
  - H2: Beveiliging
  - H2: Beperkingen (MVP)
  - H2: Gerelateerd

## channels/pairing.md

- Route: /channels/pairing
- Koppen:
  - H2: 1) DM-koppeling (toegang tot inkomende chats)
  - H3: Goedkeuren vanuit de Control UI
  - H3: Goedkeuren vanuit de CLI
  - H3: Herbruikbare afzendergroepen
  - H3: Waar de status wordt opgeslagen
  - H2: 2) Node-apparaatkoppeling (iOS/Android/macOS/headless nodes)
  - H3: Koppelen vanuit de Control UI (aanbevolen)
  - H3: Koppelen via Telegram
  - H3: Een Node-apparaat goedkeuren
  - H3: Optionele automatische goedkeuring van nodes via vertrouwde CIDR
  - H3: Opslag van de Node-koppelingsstatus
  - H3: Opmerkingen
  - H2: Gerelateerde documentatie

## channels/qa-channel.md

- Route: /channels/qa-channel
- Koppen:
  - H2: Wat het doet
  - H2: Configuratie
  - H2: Runners
  - H2: Gerelateerd

## channels/qqbot.md

- Route: /channels/qqbot
- Koppen:
  - H2: Installatie
  - H2: Configuratie
  - H2: Duurzaamheid van inkomende berichten
  - H2: Configureren
  - H3: Streaming
  - H3: Toegangsbeleid
  - H3: Configuratie voor meerdere accounts
  - H3: Groepschats
  - H3: Spraak (STT / TTS)
  - H2: Doelindelingen
  - H2: Slashopdrachten
  - H2: Media en opslag
  - H2: Probleemoplossing
  - H2: Gerelateerd

## channels/raft.md

- Route: /channels/raft
- Koppen:
  - H2: Installatie
  - H2: Vereisten
  - H2: Configureren
  - H2: Hoe het werkt
  - H2: Verifiëren
  - H2: Probleemoplossing
  - H2: Referenties

## channels/reef.md

- Route: /channels/reef
- Koppen:
  - H2: Snel aan de slag
  - H2: Agentgestuurde configuratie
  - H2: Configuratie
  - H2: Een vriend toevoegen
  - H2: Verzenden en ontvangen
  - H2: Controles en beoordeling door de eigenaar
  - H2: Probleemoplossing

## channels/signal.md

- Route: /channels/signal
- Koppen:
  - H2: Het nummermodel (lees dit eerst)
  - H2: Installatie
  - H2: Snelle configuratie
  - H2: Wat het is
  - H2: Configuratiepad A: bestaand Signal-account koppelen (QR)
  - H2: Configuratiepad B: speciaal botnummer registreren (SMS, Linux)
  - H2: Externe systeemeigen daemonmodus
  - H2: Containermodus (bbernhard/signal-cli-rest-api)
  - H2: Toegangsbeheer (DM's + groepen)
  - H2: Hoe het werkt (gedrag)
  - H2: Media + limieten
  - H2: Typindicatoren + leesbevestigingen
  - H2: Statusreacties voor de levenscyclus
  - H2: Reacties (berichtentool)
  - H2: Goedkeuringsreacties
  - H2: Vraagreacties
  - H2: Bezorgdoelen (CLI/cron)
  - H2: Aliassen
  - H2: Probleemoplossing
  - H2: Beveiligingsopmerkingen
  - H2: Configuratiereferentie (Signal)
  - H2: Gerelateerd

## channels/slack.md

- Route: /channels/slack
- Koppen:
  - H2: Een transport kiezen
  - H3: Relaymodus
  - H3: Organisatiebrede installaties voor Enterprise Grid
  - H4: Socket Mode
  - H4: HTTP Request URLs
  - H2: Installatie
  - H2: Snelle configuratie
  - H2: Gebruikersidentiteit (berichten plaatsen als een echt persoon)
  - H2: Transportafstemming voor Socket Mode
  - H2: Controlelijst voor manifest en scopes
  - H3: Aanvullende manifestinstellingen
  - H2: Tokenmodel
  - H2: Acties en controles
  - H2: Toegangsbeheer en routering
  - H2: Threads, sessies en antwoordtags
  - H2: Bevestigingsreacties
  - H3: Emoji (ackReaction)
  - H3: Bereik (messages.ackReactionScope)
  - H2: Tekststreaming
  - H2: Terugval naar typreactie
  - H2: Spraakinvoer
  - H2: Media, opsplitsing en bezorging
  - H2: Opdrachten en slashgedrag
  - H2: Systeemeigen grafieken
  - H2: Systeemeigen tabellen
  - H2: Interactieve antwoorden
  - H3: Door plugins beheerde modale inzendingen
  - H2: Systeemeigen goedkeuringen in Slack
  - H2: Gebeurtenissen en operationeel gedrag
  - H3: Aanwezigheidsgebeurtenissen
  - H2: Configuratiereferentie
  - H2: Probleemoplossing
  - H2: Mediareferentie voor bijlagen
  - H3: Ondersteunde mediatypen
  - H3: Pijplijn voor inkomende berichten
  - H3: Overerving van bijlagen uit de threadroot
  - H3: Verwerking van meerdere bijlagen
  - H3: Limieten voor grootte, downloads en modellen
  - H3: Bekende limieten
  - H3: Gerelateerde documentatie
  - H2: Gerelateerd

## channels/sms.md

- Route: /channels/sms
- Koppen:
  - H2: Voordat je begint
  - H2: Snelle configuratie
  - H2: Configuratievoorbeelden
  - H3: Configuratiebestand
  - H3: Omgevingsvariabelen
  - H3: SecretRef-authenticatietoken
  - H3: Afzender van Messaging Service
  - H3: Standaarddoel voor uitgaande berichten
  - H2: Toegangsbeheer
  - H2: SMS verzenden
  - H2: Configuratie verifiëren
  - H3: End-to-endtest vanuit macOS iMessage/SMS
  - H2: Webhook-beveiliging
  - H2: Configuratie voor meerdere accounts
  - H2: Probleemoplossing
  - H3: Twilio retourneert 403 of OpenClaw weigert de Webhook
  - H3: Er verschijnt geen koppelingsverzoek
  - H3: Uitgaande verzending mislukt
  - H3: Berichten komen aan, maar de agent antwoordt niet

## channels/synology-chat.md

- Route: /channels/synology-chat
- Koppen:
  - H2: Installatie
  - H2: Snelle configuratie
  - H2: Duurzaamheid van inkomende berichten
  - H2: Omgevingsvariabelen
  - H2: DM-beleid en toegangsbeheer
  - H2: Uitgaande bezorging
  - H2: Meerdere accounts
  - H2: Beveiligingsopmerkingen
  - H2: Probleemoplossing
  - H2: Gerelateerd

## channels/telegram.md

- Route: /channels/telegram
- Koppen:
  - H2: Snel instellen
  - H2: Instellingen aan Telegram-zijde
  - H2: Mini-app voor het dashboard
  - H2: Toegangsbeheer en activering
  - H3: Botidentiteit in groepen
  - H2: Runtimegedrag
  - H2: Functieoverzicht
  - H2: Beheer van foutantwoorden
  - H2: Probleemoplossing
  - H2: Configuratieoverzicht
  - H2: Gerelateerd

## channels/tlon.md

- Route: /channels/tlon
- Koppen:
  - H2: Meegeleverde plugin
  - H2: Instellen
  - H2: Duurzaamheid van inkomende berichten
  - H2: Privé-/LAN-ships
  - H2: Groepskanalen
  - H2: Toegangsbeheer
  - H2: Eigenaar- en goedkeuringssysteem
  - H2: Instellingen voor automatisch accepteren
  - H2: Hot-reload via de instellingenopslag van Urbit
  - H2: Afleverdoelen (CLI/cron)
  - H2: Meegeleverde Skill
  - H2: Mogelijkheden
  - H2: Probleemoplossing
  - H2: Configuratieoverzicht
  - H2: Opmerkingen
  - H2: Gerelateerd

## channels/troubleshooting.md

- Route: /channels/troubleshooting
- Koppen:
  - H2: Opdrachtladder
  - H2: Na een update
  - H2: WhatsApp
  - H3: Foutpatronen van WhatsApp
  - H2: Telegram
  - H3: Foutpatronen van Telegram
  - H2: Discord
  - H3: Foutpatronen van Discord
  - H2: Slack
  - H3: Foutpatronen van Slack
  - H2: iMessage
  - H3: Foutpatronen van iMessage
  - H2: Signal
  - H3: Foutpatronen van Signal
  - H2: QQ Bot
  - H3: Foutpatronen van QQ Bot
  - H2: Matrix
  - H3: Foutpatronen van Matrix
  - H2: Gerelateerd

## channels/twitch.md

- Route: /channels/twitch
- Koppen:
  - H2: Installeren
  - H2: Snel instellen
  - H2: Wat het is
  - H2: Duurzaamheid van inkomende berichten
  - H2: Token vernieuwen (optioneel)
  - H2: Ondersteuning voor meerdere accounts
  - H2: Toegangsbeheer
  - H2: Probleemoplossing
  - H2: Configuratie
  - H3: Accountconfiguratie
  - H3: Provideropties
  - H2: Toolacties
  - H2: Veiligheid en beheer
  - H2: Limieten
  - H2: Gerelateerd

## channels/wechat.md

- Route: /channels/wechat
- Koppen:
  - H2: Naamgeving
  - H2: Hoe het werkt
  - H2: Installeren
  - H2: Aanmelden
  - H2: Toegangsbeheer
  - H2: Compatibiliteit
  - H2: Sidecar-proces
  - H2: Probleemoplossing
  - H2: Gerelateerde documentatie

## channels/whatsapp.md

- Route: /channels/whatsapp
- Koppen:
  - H2: Installeren
  - H2: Snel instellen
  - H2: Implementatiepatronen
  - H2: Runtimemodel
  - H2: De huidige aanvrager bellen met MeowCaller (experimenteel)
  - H2: Goedkeuringsprompts
  - H2: Reacties op vragen
  - H2: Plugin-hooks en privacy
  - H2: Toegangsbeheer en activering
  - H2: Geconfigureerde ACP-koppelingen
  - H2: Gedrag voor persoonlijke nummers en chats met jezelf
  - H2: Berichtnormalisatie en context
  - H2: Aflevering, opsplitsing en media
  - H2: Antwoorden citeren
  - H2: Reactieniveau
  - H2: Ontvangstbevestigingsreacties
  - H2: Statusreacties voor de levenscyclus
  - H2: Meerdere accounts en referenties
  - H2: Tools, acties en configuratieschrijfbewerkingen
  - H2: Probleemoplossing
  - H2: Systeemprompts
  - H2: Verwijzingen naar het configuratieoverzicht
  - H2: Gerelateerd

## channels/yuanbao.md

- Route: /channels/yuanbao
- Koppen:
  - H2: Snel aan de slag
  - H3: Interactieve instelling (alternatief)
  - H2: Toegangsbeheer
  - H3: Directe berichten
  - H3: Groepschats
  - H2: Configuratievoorbeelden
  - H2: Veelgebruikte opdrachten
  - H2: Probleemoplossing
  - H2: Geavanceerde configuratie
  - H3: Meerdere accounts
  - H3: Berichtlimieten
  - H3: Streaming
  - H3: Geschiedeniscontext van groepschats
  - H3: Antwoordmodus
  - H3: Markdown-hints invoegen
  - H3: Foutopsporingsmodus
  - H3: Routering voor meerdere agents
  - H2: Configuratieoverzicht
  - H2: Ondersteunde berichttypen
  - H2: Gerelateerd

## channels/zalo.md

- Route: /channels/zalo
- Koppen:
  - H2: Meegeleverde plugin
  - H2: Snel instellen
  - H2: Wat het is
  - H2: Hoe het werkt
  - H2: Limieten
  - H2: Toegangsbeheer
  - H3: Directe berichten
  - H3: Groepen
  - H2: Long polling versus Webhook
  - H2: Ondersteunde berichttypen
  - H2: Mogelijkheden
  - H2: Afleverdoelen (CLI/cron)
  - H2: Probleemoplossing
  - H2: Configuratieoverzicht
  - H2: Gerelateerd

## channels/zaloclawbot.md

- Route: /channels/zaloclawbot
- Koppen:
  - H2: Compatibiliteit
  - H2: Vereisten
  - H2: Installeren met onboard (aanbevolen)
  - H2: Handmatige installatie
  - H3: 1. Installeer de plugin
  - H3: 2. Schakel de plugin in de configuratie in
  - H3: 3. Genereer een QR-code en meld je aan
  - H3: 4. Start de Gateway opnieuw
  - H2: Hoe het werkt
  - H2: Onder de motorkap
  - H2: Probleemoplossing
  - H2: Gerelateerd

## channels/zalouser.md

- Route: /channels/zalouser
- Koppen:
  - H2: Installeren
  - H2: Snel instellen
  - H2: Wat het is
  - H2: Naamgeving
  - H2: ID's vinden (directory)
  - H2: Limieten
  - H2: Duurzaamheid van inkomende berichten
  - H2: Toegangsbeheer (directe berichten)
  - H2: Groepstoegang (optioneel)
  - H3: Toegangspoort op basis van groepsvermeldingen
  - H2: Meerdere accounts
  - H2: Omgevingsvariabelen
  - H2: Typindicatoren, reacties en afleverbevestigingen
  - H2: Probleemoplossing
  - H2: Gerelateerd

## ci.md

- Route: /ci
- Koppen:
  - H2: Overzicht van de pijplijn
  - H2: Fail-fast-volgorde
  - H2: PR-context en bewijs
  - H2: Bereik en routering
  - H2: Doorsturen van ClawSweeper-activiteit
  - H2: Handmatige activeringen
  - H2: Runners
  - H2: Budget voor runnerregistratie
  - H2: Ratchets voor oppervlakken
  - H2: Lokale equivalenten
  - H2: OpenClaw-prestaties
  - H2: Volledige releasevalidatie
  - H2: Live- en E2E-shards
  - H2: Pakketacceptatie
  - H3: Taken
  - H3: Bronnen voor kandidaten
  - H3: Suiteprofielen
  - H3: Compatibiliteitsvensters voor verouderde versies
  - H3: Voorbeelden
  - H2: Installatiesmoketest
  - H2: Lokale Docker-E2E
  - H3: Instelbare parameters
  - H3: Herbruikbare live-/E2E-workflow
  - H3: Segmenten van het releasepad
  - H2: Plugin-prerelease
  - H2: QA Lab
  - H2: CodeQL
  - H3: Beveiligingscategorieën
  - H3: Platformspecifieke beveiligingsshards
  - H3: Categorieën voor kritieke kwaliteit
  - H2: Onderhoudsworkflows
  - H3: Documentatieagent
  - H3: Testprestatieagent
  - H3: Dubbele PR's na samenvoeging
  - H2: Lokale controlepoorten en routering van wijzigingen
  - H3: Ratchet voor het aantal configuratiebaselines
  - H2: Testbox-validatie
  - H2: Gerelateerd

## clawhub/cli.md

- Route: /clawhub/cli
- Koppen:
  - H1: ClawHub-CLI
  - H2: Ontdekken en installeren
  - H3: Vertrouwen in releases
  - H2: Publiceren en onderhouden
  - H2: Gerelateerd

## clawhub/publishing.md

- Route: /clawhub/publishing
- Koppen:
  - H1: Publiceren op ClawHub
  - H2: Eigenaren
  - H2: Skills
  - H2: Plugins
  - H2: Releaseflow
  - H2: Veelgestelde vragen
  - H3: Het pakketbereik moet overeenkomen met de geselecteerde eigenaar

## cli/acp.md

- Route: /cli/acp
- Koppen:
  - H2: Wat dit niet is
  - H2: Compatibiliteitsmatrix
  - H2: Bekende beperkingen
  - H2: Gebruik
  - H2: ACP-client (foutopsporing)
  - H2: Protocolsmoketests
  - H2: Hoe je dit gebruikt
  - H2: Agents selecteren
  - H2: Gebruiken vanuit acpx (Codex, Claude, andere ACP-clients)
  - H2: Zed-editor instellen
  - H2: Sessietoewijzing
  - H2: Opties
  - H3: Opties voor de acp-client
  - H2: Gerelateerd

## cli/agent.md

- Route: /cli/agent
- Koppen:
  - H1: openclaw agent
  - H2: Opties
  - H2: Voorbeelden
  - H2: Opmerkingen
  - H2: JSON-afleverstatus
  - H2: Gerelateerd

## cli/agents.md

- Route: /cli/agents
- Koppen:
  - H1: openclaw agents
  - H2: Voorbeelden
  - H2: Opdrachtoppervlak
  - H3: agents list
  - H3: `agents add [name]`
  - H3: agents bindings
  - H3: agents bind
  - H3: agents unbind
  - H3: agents set-identity
  - H3: agents delete &lt;id&gt;
  - H2: Routeringskoppelingen
  - H3: --bind-indeling
  - H3: Bereikgedrag van koppelingen
  - H2: Identiteitsbestanden
  - H2: Identiteit instellen
  - H2: Gerelateerd

## cli/approvals.md

- Route: /cli/approvals
- Koppen:
  - H1: openclaw approvals
  - H2: openclaw exec-policy
  - H2: Veelgebruikte opdrachten
  - H2: Openstaande goedkeuringen
  - H2: Goedkeuringen vervangen vanuit een bestand
  - H2: Voorbeeld voor "Nooit vragen" / YOLO
  - H2: Helpers voor de toelatingslijst
  - H2: Algemene opties
  - H2: Opmerkingen
  - H2: Gerelateerd

## cli/attach.md

- Route: /cli/attach
- Koppen: geen

## cli/audit.md

- Route: /cli/audit
- Koppen:
  - H1: openclaw audit
  - H2: Filters
  - H2: Vastgelegde gebeurtenissen
  - H2: Gateway-RPC
  - H2: Gerelateerd

## cli/backup.md

- Route: /cli/backup
- Koppen:
  - H1: openclaw backup
  - H2: Opmerkingen
  - H2: SQLite-snapshots
  - H3: Verifiëren en herstellen
  - H2: Waarvan een back-up wordt gemaakt
  - H2: Gedrag bij ongeldige configuratie
  - H2: Grootte en prestaties
  - H2: Gerelateerd

## cli/browser.md

- Route: /cli/browser
- Koppen:
  - H1: openclaw browser
  - H2: Algemene vlaggen
  - H2: Snel aan de slag (lokaal)
  - H2: Snelle probleemoplossing
  - H2: Levenscyclus
  - H2: Als de opdracht ontbreekt
  - H2: Profielen
  - H2: Tabbladen
  - H2: Snapshot / schermafbeelding / acties
  - H2: Status en opslag
  - H2: Foutopsporing
  - H2: Bestaande Chrome via MCP
  - H2: Externe browserbediening (hostproxy van Node)
  - H2: Gerelateerd

## cli/channels.md

- Route: /cli/channels
- Koppen:
  - H1: openclaw channels
  - H2: Algemene opdrachten
  - H2: Status / mogelijkheden / oplossen / logboeken
  - H2: Niet-verwerkbare inkomende berichten
  - H2: Accounts toevoegen / verwijderen
  - H2: Aanmelden en afmelden (interactief)
  - H2: Probleemoplossing
  - H2: Mogelijkheden testen
  - H2: Namen omzetten in ID's
  - H2: Gerelateerd

## cli/clawbot.md

- Route: /cli/clawbot
- Koppen:
  - H1: openclaw clawbot
  - H2: Migratie
  - H2: Gerelateerd

## cli/claws.md

- Route: /cli/claws
- Koppen:
  - H1: openclaw claws
  - H2: Een Claw-pakket maken
  - H2: Inspecteren en voorvertonen
  - H2: Geïnstalleerde status inspecteren
  - H2: Een geïnstalleerde Claw bijwerken
  - H2: Een geïnstalleerde Claw verwijderen
  - H2: Een geïnstalleerde agent exporteren
  - H2: Opdrachtenreferentie
  - H2: Zie ook

## cli/commitments.md

- Route: /cli/commitments
- Koppen:
  - H2: Gebruik
  - H2: Opties
  - H2: Voorbeelden
  - H2: Uitvoer
  - H2: Gerelateerd

## cli/completion.md

- Route: /cli/completion
- Koppen:
  - H1: openclaw completion
  - H2: Gebruik
  - H2: Opties
  - H2: Installatieproces
  - H2: Opmerkingen
  - H2: Gerelateerd

## cli/config.md

- Route: /cli/config
- Koppen:
  - H2: Hoofdopties
  - H2: Voorbeelden
  - H3: Paden
  - H3: config get
  - H3: config file
  - H3: config schema
  - H3: config validate
  - H2: Waarden
  - H2: Modi van config set
  - H3: Vlaggen voor de providerbouwer
  - H2: config patch
  - H2: Proefdraaien
  - H3: Structuur van JSON-uitvoer
  - H2: Wijzigingen toepassen
  - H2: Schrijfveiligheid
  - H2: Reparatiecyclus
  - H2: Gerelateerd

## cli/configure.md

- Route: /cli/configure
- Koppen:
  - H1: openclaw configure
  - H2: Opties
  - H2: Modelsectie
  - H2: Websectie
  - H2: Overige opmerkingen
  - H2: Gerelateerd

## cli/crestodian.md

- Route: /cli/crestodian
- Koppen: geen

## cli/cron.md

- Route: /cli/cron
- Koppen:
  - H1: openclaw cron
  - H2: Snel taken maken
  - H2: Sessies
  - H2: Levering
  - H3: Eigenaarschap van levering
  - H3: Levering bij fouten
  - H2: Planning
  - H3: Eenmalige taken
  - H3: Terugkerende taken
  - H3: Handmatige uitvoeringen
  - H2: Modellen
  - H3: Modelprioriteit voor geïsoleerde Cron
  - H3: Snelle modus
  - H3: Nieuwe pogingen bij live modelwisselingen
  - H2: Uitvoer en weigeringen van uitvoeringen
  - H3: Onderdrukking van verouderde bevestigingen
  - H3: Onderdrukking van stille tokens
  - H3: Gestructureerde weigeringen
  - H2: Bewaartermijn
  - H2: Oudere taken migreren
  - H2: Veelvoorkomende bewerkingen
  - H2: Algemene beheeropdrachten
  - H2: Gerelateerd

## cli/daemon.md

- Route: /cli/daemon
- Koppen:
  - H1: openclaw daemon
  - H2: Gebruik
  - H2: Subopdrachten en opties
  - H2: Opmerkingen
  - H2: Gerelateerd

## cli/dashboard.md

- Route: /cli/dashboard
- Koppen:
  - H1: openclaw dashboard
  - H2: Machineleesbare uitvoer
  - H2: Gerelateerd

## cli/devices.md

- Route: /cli/devices
- Koppen:
  - H1: openclaw devices
  - H2: Algemene opties
  - H2: Opdrachten
  - H3: openclaw devices list
  - H3: `openclaw devices approve [requestId] [--latest]`
  - H3: openclaw devices reject &lt;requestId&gt;
  - H3: openclaw devices remove &lt;deviceId&gt;
  - H3: openclaw devices rename --device &lt;id&gt; --name &lt;label&gt;
  - H3: `openclaw devices clear --yes [--pending]`
  - H3: `openclaw devices rotate --device &lt;id&gt; --role &lt;role&gt; [--scope &lt;scope...&gt;]`
  - H3: openclaw devices revoke --device &lt;id&gt; --role &lt;role&gt;
  - H2: Opmerkingen
  - H2: Controlelijst voor herstel van tokenafwijkingen
  - H2: Goedkeuring bij eerste uitvoering van Paperclip / `openclaw_gateway`
  - H2: Gerelateerd

## cli/directory.md

- Route: /cli/directory
- Koppen:
  - H1: openclaw directory
  - H2: Algemene vlaggen
  - H2: Opmerkingen
  - H2: Resultaten gebruiken met message send
  - H2: ID-indelingen per kanaal
  - H2: Zelf ("ik")
  - H2: Peers (contactpersonen/gebruikers)
  - H2: Groepen
  - H2: Gerelateerd

## cli/dns.md

- Route: /cli/dns
- Koppen:
  - H1: openclaw dns
  - H2: dns setup
  - H2: Gerelateerd

## cli/docs.md

- Route: /cli/docs
- Koppen:
  - H1: openclaw docs
  - H2: Gebruik
  - H2: Voorbeelden
  - H2: Werking
  - H2: Uitvoer
  - H2: Afsluitcodes
  - H2: Gerelateerd

## cli/doctor.md

- Route: /cli/doctor
- Koppen:
  - H1: openclaw doctor
  - H2: Houdingen
  - H2: Voorbeelden
  - H2: Opties
  - H2: Lintmodus
  - H2: Gestructureerde statuscontroles
  - H2: Controles selecteren
  - H2: Modus na upgrade
  - H2: Migratie van verouderde status
  - H2: SQLite-Compaction van gedeelde status
  - H2: SQLite-migratie van sessies
  - H3: Downgraden na SQLite-migratie van sessies
  - H2: Opmerkingen
  - H2: macOS: omgevingsoverschrijvingen van launchctl
  - H2: Gerelateerd

## cli/fleet.md

- Route: /cli/fleet
- Koppen:
  - H1: openclaw fleet
  - H2: Snel aan de slag
  - H2: Tenant-ID's
  - H2: fleet create
  - H3: Aanmaakopties
  - H3: Vastzetten op digest
  - H3: Schijflimieten
  - H3: Uitgaand-verkeersbeleid
  - H2: fleet list
  - H2: fleet status
  - H2: fleet logs
  - H2: fleet start, fleet stop en fleet restart
  - H2: fleet upgrade
  - H2: fleet backup en fleet restore
  - H2: fleet doctor
  - H2: fleet rm
  - H2: Opslag- en containerindeling
  - H2: Beveiligingsprofiel
  - H2: Tokenverwerking
  - H2: Gerelateerd

## cli/flows.md

- Route: /cli/flows
- Koppen:
  - H1: openclaw tasks flow
  - H2: Subopdrachten
  - H3: Waarden voor statusfilters
  - H2: Voorbeelden
  - H2: Gerelateerd

## cli/gateway.md

- Route: /cli/gateway
- Koppen:
  - H2: De Gateway uitvoeren
  - H3: Opties
  - H2: De Gateway opnieuw starten
  - H3: Externe supervisors
  - H3: Gateway-profilering
  - H2: Een actieve Gateway opvragen
  - H3: gateway health
  - H3: gateway usage-cost
  - H3: gateway stability
  - H3: gateway diagnostics export
  - H3: gateway status
  - H3: gateway probe
  - H4: Extern via SSH (gelijkwaardig aan de Mac-app)
  - H3: gateway call &lt;method&gt;
  - H2: De Gateway-service beheren
  - H3: Installeren met een wrapper
  - H2: Gateways ontdekken (Bonjour)
  - H3: gateway discover
  - H2: Gerelateerd

## cli/health.md

- Route: /cli/health
- Koppen:
  - H1: openclaw health
  - H2: Opties
  - H2: Gedrag
  - H2: Gerelateerd

## cli/hooks.md

- Route: /cli/hooks
- Koppen:
  - H1: openclaw hooks
  - H2: Hooks weergeven
  - H2: Hook-informatie ophalen
  - H2: Geschiktheid controleren
  - H2: Een hook inschakelen
  - H2: Een hook uitschakelen
  - H2: Hookpakketten installeren en bijwerken
  - H2: Meegeleverde hooks
  - H3: Logbestand van command-logger
  - H2: Opmerkingen
  - H2: Gerelateerd

## cli/index.md

- Route: /cli
- Koppen:
  - H2: Opdrachtpagina's
  - H2: Globale vlaggen
  - H2: Uitvoermodi
  - H2: Kleurenpalet
  - H2: Opdrachtenstructuur
  - H2: Slash-opdrachten voor chats
  - H2: Gebruiksregistratie
  - H2: Gerelateerd

## cli/infer.md

- Route: /cli/infer
- Koppen:
  - H2: infer omzetten in een skill
  - H2: Opdrachtenstructuur
  - H2: Algemene taken
  - H2: Gedrag
  - H2: Model
  - H2: Afbeelding
  - H2: Audio
  - H2: TTS
  - H2: Video
  - H2: Web
  - H2: Inbedding
  - H2: JSON-uitvoer
  - H2: Veelvoorkomende valkuilen
  - H2: Gerelateerd

## cli/logs.md

- Route: /cli/logs
- Koppen:
  - H1: openclaw logs
  - H2: Opties
  - H2: Gedeelde RPC-opties voor de Gateway
  - H2: Voorbeelden
  - H2: Gedrag bij terugval en herstel
  - H2: Gerelateerd

## cli/mcp.md

- Route: /cli/mcp
- Koppen:
  - H2: Kies het juiste MCP-pad
  - H2: OpenClaw als MCP-server
  - H3: Wanneer serve te gebruiken
  - H3: Hoe het werkt
  - H3: Kies een clientmodus
  - H3: Wat serve beschikbaar stelt
  - H3: Gebruik
  - H3: Brugtools
  - H3: Gebeurtenismodel
  - H3: Claude-kanaalmeldingen
  - H3: MCP-clientconfiguratie
  - H3: Opties
  - H3: Beveiligings- en vertrouwensgrens
  - H3: Testen
  - H3: Probleemoplossing
  - H2: OpenClaw als MCP-clientregister
  - H3: Opgeslagen MCP-serverdefinities
  - H3: Veelgebruikte serverrecepten
  - H3: JSON-uitvoerstructuren
  - H3: Stdio-transport
  - H3: SSE-/HTTP-transport
  - H3: OAuth-workflow
  - H3: Streambaar HTTP-transport
  - H2: Bedieningsinterface
  - H2: MCP-apps
  - H2: Huidige beperkingen
  - H2: Gerelateerd

## cli/memory.md

- Route: /cli/memory
- Koppen:
  - H1: openclaw memory
  - H2: memory status
  - H2: memory index
  - H2: memory search
  - H2: memory promote
  - H2: memory promote-explain
  - H2: memory rem-harness
  - H2: memory rem-backfill
  - H2: Dreaming
  - H2: SecretRef-Gateway-afhankelijkheid
  - H2: Gerelateerd

## cli/message.md

- Route: /cli/message
- Koppen:
  - H1: openclaw message
  - H2: Kanaalselectie
  - H2: Doelindelingen (-t, --target)
  - H2: Algemene vlaggen
  - H2: SecretRef-resolutie
  - H2: Acties
  - H3: Kern
  - H3: Verzenden
  - H3: Peiling
  - H3: Threads
  - H3: Emoji's
  - H3: Stickers
  - H3: Rollen, kanalen, spraak en gebeurtenissen (Discord)
  - H3: Moderatie (Discord)
  - H3: Uitzending
  - H2: Gerelateerd

## cli/migrate.md

- Route: /cli/migrate
- Koppen:
  - H1: openclaw migrate
  - H2: Opdrachten
  - H2: Veiligheidsmodel
  - H2: Claude-provider
  - H3: Wat Claude importeert
  - H3: Archief- en handmatige-beoordelingsstatus
  - H2: Codex-provider
  - H3: Wat Codex importeert
  - H3: Codex-status voor handmatige beoordeling
  - H2: Hermes-provider
  - H3: Wat Hermes importeert
  - H3: Ondersteunde .env-sleutels
  - H3: Status uitsluitend voor archivering
  - H3: Na het toepassen
  - H2: Plugin-contract
  - H2: Onboardingintegratie
  - H2: Gerelateerd

## cli/models.md

- Route: /cli/models
- Koppen:
  - H1: openclaw models
  - H2: Veelgebruikte opdrachten
  - H3: Status
  - H3: Lijst
  - H3: Standaard-/afbeeldingsmodel instellen
  - H3: Scannen
  - H2: Aliassen
  - H2: Terugvalopties
  - H2: Authenticatieprofielen
  - H2: Gerelateerd

## cli/node.md

- Route: /cli/node
- Koppen:
  - H1: openclaw node
  - H2: Waarom een Node-host gebruiken?
  - H2: Browserproxy (zonder configuratie)
  - H2: Uitvoeren (voorgrond)
  - H2: Gateway-authenticatie voor Node-host
  - H2: Service (achtergrond)
  - H2: Koppelen
  - H3: Identiteits- en koppelingsstatus
  - H2: Uitvoeringsgoedkeuringen
  - H2: Gerelateerd

## cli/nodes.md

- Route: /cli/nodes
- Koppen:
  - H1: openclaw nodes
  - H2: Status
  - H2: Koppelen
  - H2: Aanroepen
  - H2: Meldingen, pushberichten, locatie en scherm
  - H2: Gerelateerd

## cli/onboard.md

- Route: /cli/onboard
- Koppen:
  - H1: openclaw onboard
  - H2: Voorbeelden
  - H2: Begeleide procedure
  - H2: Opnieuw instellen
  - H2: Landinstelling
  - H2: Niet-interactieve configuratie
  - H3: Gateway-authenticatie (niet-interactief)
  - H3: Status van lokale Gateway
  - H3: Interactieve referentiemodus
  - H3: Keuzes voor Z.AI-eindpunten
  - H2: Aanvullende niet-interactieve vlaggen
  - H2: Vooraf filteren van providers
  - H2: Vervolgacties voor zoeken op internet
  - H2: Ander gedrag
  - H2: Veelgebruikte vervolgopdrachten

## cli/openclaw.md

- Route: /cli/openclaw
- Koppen:
  - H1: openclaw setup
  - H2: Wanneer het wordt gestart
  - H2: Wat OpenClaw weergeeft
  - H2: Voorbeelden
  - H2: Bewerkingen en goedkeuring
  - H3: Wijzigingsgeschiedenis
  - H3: Overschakelen naar gemaskeerde kanaalconfiguratie
  - H2: Initiële configuratie
  - H2: AI-gesprek
  - H3: Vertrouwensmodel van de CLI-harnas
  - H2: Overschakelen naar een agent
  - H2: Berichtenherstelmodus
  - H2: Gerelateerd

## cli/pairing.md

- Route: /cli/pairing
- Koppen:
  - H1: openclaw pairing
  - H2: Opdrachten
  - H2: pairing list
  - H2: pairing approve
  - H3: Initiële eigenaarconfiguratie
  - H2: Gerelateerd

## cli/path.md

- Route: /cli/path
- Koppen:
  - H1: openclaw path
  - H2: Waarom dit gebruiken
  - H2: Hoe het wordt gebruikt
  - H2: Hoe het werkt
  - H2: Subopdrachten
  - H2: Algemene vlaggen
  - H2: oc://-syntaxis
  - H2: Adressering per bestandstype
  - H2: Wijzigingscontract
  - H2: Voorbeelden
  - H2: Recepten per bestandstype
  - H3: Markdown
  - H3: JSONC
  - H3: JSONL
  - H3: YAML
  - H2: Naslag voor subopdrachten
  - H3: resolve &lt;oc-path&gt;
  - H3: find &lt;pattern&gt;
  - H3: set &lt;oc-path&gt; &lt;value&gt;
  - H3: validate &lt;oc-path&gt;
  - H3: emit &lt;file&gt;
  - H2: Afsluitcodes
  - H2: Uitvoermodus
  - H2: Opmerkingen
  - H2: Gerelateerd

## cli/plugins.md

- Route: /cli/plugins
- Koppen:
  - H2: Opdrachten
  - H2: Maken
  - H3: Providersjabloon
  - H2: Installeren
  - H3: Verkorte marketplace-notatie
  - H2: Lijst
  - H3: Plugin-index
  - H2: Verwijderen
  - H2: Bijwerken
  - H2: Inspecteren
  - H2: Doctor
  - H2: Register
  - H2: Marketplace
  - H2: Gerelateerd

## cli/policy.md

- Route: /cli/policy
- Koppen:
  - H1: openclaw policy
  - H2: Snel aan de slag
  - H3: Naslag voor beleidsregels
  - H4: Bereikgebonden overlays
  - H4: Kanalen
  - H4: MCP-servers
  - H4: Modelproviders
  - H4: Netwerk
  - H4: Berichtroutering
  - H4: Inkomende toegang en kanaaltoegang
  - H4: Gateway
  - H4: Agentwerkruimte
  - H4: Sandboxbeleid
  - H4: Gegevensverwerking
  - H4: Geheimen
  - H4: Uitvoeringsgoedkeuringen
  - H4: Authenticatieprofielen
  - H4: Toolmetadata
  - H4: Toolbeleid
  - H2: Controles uitvoeren
  - H2: Beleid configureren
  - H2: Beleidsstatus accepteren
  - H2: Bevindingen
  - H2: Herstellen
  - H2: Afsluitcodes
  - H2: Gerelateerd

## cli/promos.md

- Route: /cli/promos
- Koppen:
  - H1: openclaw promos
  - H2: Opdrachten
  - H2: openclaw promos list
  - H2: openclaw promos claim &lt;slug&gt;
  - H2: Passieve ontdekking in de modellenlijst

## cli/proxy.md

- Route: /cli/proxy
- Koppen:
  - H1: openclaw proxy
  - H2: Valideren
  - H3: Opties
  - H2: Proxy debuggen
  - H2: Gerelateerd

## cli/qr.md

- Route: /cli/qr
- Koppen:
  - H1: openclaw qr
  - H2: Opties
  - H2: Inhoud van de configuratiecode
  - H2: Gateway-URL-resolutie
  - H2: Authenticatieresolutie (zonder --remote)
  - H2: Authenticatieresolutie (--remote)
  - H2: Gerelateerd

## cli/reset.md

- Route: /cli/reset
- Koppen:
  - H1: openclaw reset
  - H2: Opties
  - H2: Bereiken
  - H2: Opmerkingen
  - H2: Gerelateerd

## cli/sandbox.md

- Route: /cli/sandbox
- Koppen:
  - H2: Opdrachten
  - H3: openclaw sandbox list
  - H3: openclaw sandbox recreate
  - H3: openclaw sandbox explain
  - H2: Waarom opnieuw maken nodig is
  - H2: Veelvoorkomende oorzaken
  - H2: Registermigratie
  - H2: Configuratie
  - H2: Gerelateerd

## cli/secrets.md

- Route: /cli/secrets
- Koppen:
  - H1: openclaw secrets
  - H2: Runtime-snapshot opnieuw laden
  - H2: Audit
  - H2: Configureren (interactieve helper)
  - H3: Veiligheid van uitvoeringsprovider
  - H2: Een opgeslagen plan toepassen
  - H3: Waarom er geen terugrolback-ups zijn
  - H2: Voorbeeld
  - H2: Gerelateerd

## cli/security.md

- Route: /cli/security
- Koppen:
  - H1: openclaw security
  - H2: Auditmodi
  - H2: Wat wordt gecontroleerd
  - H2: SecretRef-gedrag
  - H2: Onderdrukkingen
  - H2: JSON-uitvoer
  - H2: Wat --fix wijzigt
  - H2: Gerelateerd

## cli/sessions.md

- Route: /cli/sessions
- Koppen:
  - H1: openclaw sessions
  - H2: Voortgang van het traject volgen
  - H2: Een trajectbundel exporteren
  - H2: Opschoningsonderhoud
  - H2: Een sessie compacteren
  - H3: sessions.compact-RPC
  - H2: Gerelateerd

## cli/setup.md

- Route: /cli/setup
- Koppen:
  - H1: openclaw setup
  - H2: Opties
  - H3: Basismodus
  - H2: Voorbeelden
  - H2: Opmerkingen
  - H2: Gerelateerd

## cli/skills.md

- Route: /cli/skills
- Koppen:
  - H1: openclaw skills
  - H2: Opdrachten
  - H2: Skills-workshop
  - H2: Gerelateerd

## cli/status.md

- Route: /cli/status
- Koppen:
  - H2: Sessie- en modelresolutie
  - H2: Gebruik en quotum
  - H2: Overzicht en updatestatus
  - H2: Geheimen
  - H2: Geheugen
  - H2: Gerelateerd

## cli/system.md

- Route: /cli/system
- Koppen:
  - H1: openclaw system
  - H2: Veelgebruikte opdrachten
  - H2: system event
  - H2: system heartbeat last|enable|disable
  - H2: system presence
  - H2: Opmerkingen
  - H2: Gerelateerd

## cli/tasks.md

- Route: /cli/tasks
- Koppen:
  - H2: Gebruik
  - H2: Hoofdopties
  - H2: Subopdrachten
  - H3: list
  - H3: show
  - H3: notify
  - H3: cancel
  - H3: audit
  - H3: maintenance
  - H3: flow
  - H2: Gerelateerd

## cli/transcripts.md

- Route: /cli/transcripts
- Koppen:
  - H1: openclaw transcripts
  - H2: Opdrachten
  - H2: Uitvoer
  - H2: Veel sessies per dag
  - H2: Ontbrekende samenvattingen
  - H2: De verouderde bestandsopslag upgraden
  - H2: Configuratie

## cli/tui.md

- Route: /cli/tui
- Koppen:
  - H1: openclaw tui
  - H2: Opties
  - H2: Opmerkingen
  - H2: Voorbeelden
  - H2: Herstelcyclus voor configuratie
  - H2: Gerelateerd

## cli/uninstall.md

- Route: /cli/uninstall
- Koppen:
  - H1: openclaw uninstall
  - H2: Opties
  - H2: Voorbeelden
  - H2: Opmerkingen
  - H2: Gerelateerd

## cli/update.md

- Route: /cli/update
- Koppen:
  - H1: openclaw update
  - H2: Gebruik
  - H2: Opties
  - H2: update status
  - H2: update repair
  - H2: update wizard
  - H2: Wat het doet
  - H3: Overdracht bij herstart
  - H3: Antwoordstructuur van het besturingsvlak
  - H2: Git-checkoutflow
  - H3: Kanaalselectie
  - H3: Updatestappen
  - H3: Details over Pluginsynchronisatie
  - H2: Gerelateerd

## cli/voicecall.md

- Route: /cli/voicecall
- Koppen:
  - H1: openclaw voicecall
  - H2: Subopdrachten
  - H2: Installatie en rooktest
  - H3: setup
  - H3: smoke
  - H2: Levenscyclus van gesprekken
  - H3: call
  - H3: start
  - H3: continue
  - H3: speak
  - H3: dtmf
  - H3: end
  - H3: status
  - H2: Logboeken en metrische gegevens
  - H3: tail
  - H3: latency
  - H2: Webhooks beschikbaar stellen
  - H3: expose
  - H2: Gerelateerd

## cli/webhooks.md

- Route: /cli/webhooks
- Koppen:
  - H1: openclaw webhooks
  - H2: Subopdrachten
  - H2: webhooks gmail setup
  - H3: Vereist
  - H3: Pub/Sub-opties
  - H3: Bezorgingsopties van OpenClaw
  - H3: Opties voor gog watch serve
  - H3: Beschikbaarstelling via Tailscale
  - H3: Uitvoer
  - H2: webhooks gmail run
  - H2: Gerelateerd

## cli/wiki.md

- Route: /cli/wiki
- Koppen:
  - H1: openclaw wiki
  - H2: Veelgebruikte opdrachten
  - H2: Agentselectie
  - H2: Opdrachten
  - H3: wiki status
  - H3: wiki doctor
  - H3: wiki init
  - H3: wiki ingest &lt;path&gt;
  - H3: wiki okf import &lt;path&gt;
  - H3: wiki compile
  - H3: wiki lint
  - H3: wiki search &lt;query&gt;
  - H3: wiki get &lt;lookup&gt;
  - H3: wiki apply
  - H3: wiki bridge import
  - H3: wiki unsafe-local import
  - H3: wiki chatgpt import
  - H3: wiki chatgpt rollback &lt;run-id&gt;
  - H3: wiki obsidian ...
  - H2: Praktische gebruiksrichtlijnen
  - H2: Koppelingen met de configuratie
  - H2: Gerelateerd

## cli/workboard.md

- Route: /cli/workboard
- Koppen:
  - H2: Gebruik
  - H2: list
  - H2: create
  - H2: show
  - H2: move
  - H2: dispatch
  - H2: Gelijkwaardigheid van slashopdrachten
  - H2: Machtigingen
  - H2: Probleemoplossing
  - H3: Er verschijnen geen kaarten
  - H3: Dispatch meldt dat alleen gegevens worden verwerkt
  - H3: Dispatch start niets
  - H2: Gerelateerd

## cli/worker.md

- Route: /cli/worker
- Koppen:
  - H1: openclaw worker
  - H2: Startcontract
  - H2: Runtimegrens

## concepts/active-memory.md

- Route: /concepts/active-memory
- Koppen:
  - H2: Informatie tussen gesprekken onthouden
  - H2: Snel aan de slag met geavanceerd Active Memory
  - H2: Hoe het werkt
  - H2: Wanneer het wordt uitgevoerd
  - H3: Sessietypen
  - H2: Sessieschakelaar
  - H2: Hoe je het kunt bekijken
  - H2: Zoekmodi
  - H2: Promptstijlen
  - H2: Beleid voor modelterugval
  - H3: Snelheidsaanbevelingen
  - H4: Cerebras instellen
  - H2: Geheugenhulpmiddelen
  - H3: Ingebouwd geheugen
  - H3: LanceDB-geheugen
  - H3: Lossless Claw
  - H2: Geavanceerde uitwijkmogelijkheden
  - H2: Persistentie van transcripties
  - H2: Configuratie
  - H2: Aanbevolen configuratie
  - H3: Respijtperiode bij koude start
  - H2: Foutopsporing
  - H2: Veelvoorkomende problemen
  - H2: Gerelateerde pagina's

## concepts/agent-loop.md

- Route: /concepts/agent-loop
- Koppen:
  - H2: Toegangspunten
  - H2: Uitvoeringsvolgorde
  - H2: Wachtrijen en gelijktijdigheid
  - H2: Sessie en werkruimte voorbereiden
  - H2: Prompt samenstellen
  - H2: Hooks
  - H3: Interne hooks (Gateway-hooks)
  - H3: Plugin-hooks
  - H2: Streaming
  - H2: Hulpmiddelen uitvoeren
  - H2: Antwoord vormgeven
  - H2: Compaction en nieuwe pogingen
  - H2: Gebeurtenisstromen
  - H2: Chatkanalen verwerken
  - H2: Time-outs
  - H3: Diagnose van vastgelopen sessies
  - H2: Waar processen voortijdig kunnen eindigen
  - H2: Gerelateerd

## concepts/agent-runtimes.md

- Route: /concepts/agent-runtimes
- Koppen:
  - H2: Codex-oppervlakken
  - H2: Eigendom van de runtime
  - H2: Runtimeselectie
  - H2: GitHub Copilot-agentruntime
  - H2: Compatibiliteitscontract
  - H2: Statuslabels
  - H2: Gerelateerd

## concepts/agent-workspace.md

- Route: /concepts/agent-workspace
- Koppen:
  - H2: Standaardlocatie
  - H2: Extra werkruimtemappen
  - H2: Bestandsindeling van de werkruimte
  - H2: Wat NIET in de werkruimte staat
  - H2: Git-back-up (aanbevolen, privé)
  - H2: Leg geen geheimen vast
  - H2: De werkruimte naar een nieuwe machine verplaatsen
  - H2: Geavanceerde opmerkingen
  - H2: Gerelateerd

## concepts/agent.md

- Route: /concepts/agent
- Koppen:
  - H2: Werkruimte (vereist)
  - H2: Bootstrapbestanden (geïnjecteerd)
  - H2: Ingebouwde hulpmiddelen
  - H2: Skills
  - H2: Runtimegrenzen
  - H2: Sessies
  - H2: Bijsturen tijdens het streamen
  - H2: Modelverwijzingen
  - H2: Configuratie (minimaal)
  - H2: Gerelateerd

## concepts/architecture.md

- Route: /concepts/architecture
- Koppen:
  - H2: Overzicht
  - H2: Componenten en flows
  - H3: Gateway (daemon)
  - H3: Clients (Mac-app / CLI / webbeheer)
  - H3: Nodes (macOS / iOS / Android / headless)
  - H3: WebChat
  - H2: Levenscyclus van de verbinding (één client)
  - H2: Wireprotocol (samenvatting)
  - H2: Koppelen en lokaal vertrouwen
  - H2: Protocoltypering en codegeneratie
  - H2: Externe toegang
  - H2: Momentopname van bewerkingen
  - H2: Invarianten
  - H2: Gerelateerd

## concepts/channel-docking.md

- Route: /concepts/channel-docking
- Koppen:
  - H2: Voorbeeld
  - H2: Waarom je dit gebruikt
  - H2: Vereiste configuratie
  - H2: Opdrachten
  - H2: Wat verandert
  - H2: Wat niet verandert
  - H2: Probleemoplossing

## concepts/commitments.md

- Route: /concepts/commitments
- Koppen:
  - H2: Bestaande records
  - H2: Gerelateerd

## concepts/compaction.md

- Route: /concepts/compaction
- Koppen:
  - H2: Hoe het werkt
  - H2: Automatische Compaction
  - H2: Handmatige Compaction
  - H2: Configuratie
  - H3: Een ander model gebruiken
  - H3: Identificatiegegevens behouden
  - H3: Bytebeveiliging voor actieve transcripties
  - H3: Opvolgende transcripties
  - H3: Compaction-meldingen
  - H3: Geheugen doorschrijven
  - H2: Inplugbare Compaction-providers
  - H2: Compaction versus opschonen
  - H2: Probleemoplossing
  - H2: Gerelateerd

## concepts/context-engine.md

- Route: /concepts/context-engine
- Koppen:
  - H2: Snel aan de slag
  - H2: Hoe het werkt
  - H3: Levenscyclus van subagenten (optioneel)
  - H3: Toevoeging aan de systeemprompt
  - H2: De verouderde engine
  - H2: Plugin-engines
  - H3: De ContextEngine-interface
  - H3: Runtime-instellingen
  - H3: Hostvereisten
  - H3: Foutisolatie
  - H3: ownsCompaction
  - H2: Configuratiereferentie
  - H2: Relatie met Compaction en geheugen
  - H2: Tips
  - H2: Gerelateerd

## concepts/context.md

- Route: /concepts/context
- Koppen:
  - H2: Snel aan de slag (context inspecteren)
  - H2: Voorbeelduitvoer
  - H3: /context list
  - H3: /context detail
  - H3: /context map
  - H2: Wat meetelt voor het contextvenster
  - H2: Hoe OpenClaw de systeemprompt opbouwt
  - H2: Geïnjecteerde werkruimtebestanden (projectcontext)
  - H2: Skills: geïnjecteerd versus op aanvraag geladen
  - H2: Hulpmiddelen: er zijn twee kostenposten
  - H2: Opdrachten, richtlijnen en 'inline-snelkoppelingen'
  - H2: Sessies, Compaction en opschoning (wat behouden blijft)
  - H2: Wat /context daadwerkelijk rapporteert
  - H2: Gerelateerd

## concepts/delegate-architecture.md

- Route: /concepts/delegate-architecture
- Koppen:
  - H2: Wat is een gedelegeerde
  - H2: Waarom gedelegeerden
  - H2: Capaciteitsniveaus
  - H3: Niveau 1: alleen-lezen + concept
  - H3: Niveau 2: namens iemand verzenden
  - H3: Niveau 3: proactief
  - H2: Vereisten: isolatie en beveiliging
  - H3: Harde blokkades (niet onderhandelbaar)
  - H3: Beperkingen voor hulpmiddelen
  - H3: Sandboxisolatie
  - H3: Auditspoor
  - H2: Een gedelegeerde instellen
  - H3: 1. De gedelegeerde agent maken
  - H3: 2. Delegatie van de identiteitsprovider configureren
  - H4: Microsoft 365
  - H4: Google Workspace
  - H3: 3. De gedelegeerde aan kanalen koppelen
  - H3: 4. Referenties aan de gedelegeerde agent toevoegen
  - H2: Voorbeeld: organisatieassistent
  - H2: Schaalpatroon
  - H2: Gerelateerd

## concepts/dreaming.md

- Route: /concepts/dreaming
- Koppen:
  - H2: Wat Dreaming schrijft
  - H2: Fasemodel
  - H2: Opname van sessietranscripten
  - H2: Droomdagboek
  - H2: Signalen voor diepgaande rangschikking
  - H3: Dekking van het rapport voor de QA-schaduwproef
  - H2: Planning
  - H2: Snel aan de slag
  - H2: Slash-opdracht
  - H2: CLI-workflow
  - H2: Belangrijkste standaardwaarden
  - H2: Dromeninterface
  - H2: Gerelateerd

## concepts/experimental-features.md

- Route: /concepts/experimental-features
- Koppen:
  - H2: Momenteel gedocumenteerde vlaggen
  - H2: Labs in de besturingsinterface
  - H2: Zuinige modus voor lokale modellen
  - H3: Waarom deze hulpmiddelen
  - H3: Wanneer je deze inschakelt
  - H3: Wanneer je deze uitgeschakeld laat
  - H3: Inschakelen
  - H2: Experimenteel betekent niet verborgen
  - H2: Gerelateerd

## concepts/features.md

- Route: /concepts/features
- Koppen:
  - H2: Hoogtepunten
  - H2: Volledige lijst
  - H2: Gerelateerd

## concepts/main-session.md

- Route: /concepts/main-session
- Koppen:
  - H2: Start
  - H2: Wat naar de hoofdsessie stroomt
  - H2: Geheugen tussen resets en gesprekken
  - H2: Een doorlopende sessie met duurzame geschiedenis
  - H2: Wanneer je in plaats daarvan isolatie wilt
  - H2: Gerelateerd

## concepts/managed-worktrees.md

- Route: /concepts/managed-worktrees
- Koppen:
  - H2: Indeling en namen
  - H2: Genegeerde bestanden beschikbaar stellen
  - H2: Repositoryconfiguratie uitvoeren
  - H2: Sessiewerkmappen
  - H2: Momentopnamen, opschoning en herstel
  - H2: CLI
  - H2: Gateway-methoden
  - H2: Werkruimten voor het werkbord

## concepts/mantis-slack-desktop-runbook.md

- Route: /concepts/mantis-slack-desktop-runbook
- Koppen:
  - H2: Opslagmodel
  - H2: GitHub-activering
  - H2: Lokale CLI
  - H2: Hydratatiemodi
  - H2: Interpretatie van tijdmetingen
  - H2: Controlelijst voor bewijs
  - H2: Afhandeling van fouten
  - H2: Gerelateerd

## concepts/mantis.md

- Route: /concepts/mantis
- Koppen:
  - H2: Eigenaarschap
  - H2: CLI-opdrachten
  - H3: discord-smoke
  - H3: run
  - H3: desktop-browser-smoke
  - H3: slack-desktop-smoke
  - H3: telegram-desktop-builder
  - H2: Bewijsmanifest
  - H2: GitHub-automatisering
  - H2: Machines en geheimen
  - H2: Uitkomsten van uitvoeringen
  - H2: Een scenario toevoegen
  - H2: Openstaande vragen

## concepts/markdown-formatting.md

- Route: /concepts/markdown-formatting
- Koppen:
  - H2: Pijplijn
  - H2: IR-voorbeeld
  - H2: Tabelverwerking
  - H2: Regels voor segmentering
  - H2: Linkbeleid
  - H2: Spoilers
  - H2: Een kanaalformatter toevoegen of bijwerken
  - H2: Veelvoorkomende valkuilen
  - H2: Gerelateerd

## concepts/memory-builtin.md

- Route: /concepts/memory-builtin
- Koppen:
  - H2: Wat het biedt
  - H2: Aan de slag
  - H2: Ondersteunde aanbieders van embeddings
  - H2: Hoe indexering werkt
  - H2: Wanneer te gebruiken
  - H2: Problemen oplossen
  - H2: Configuratie
  - H2: Gerelateerd

## concepts/memory-honcho.md

- Route: /concepts/memory-honcho
- Koppen:
  - H2: Wat het biedt
  - H2: Beschikbare hulpmiddelen
  - H2: Aan de slag
  - H2: Configuratie
  - H2: Bestaand geheugen migreren
  - H2: Hoe het werkt
  - H2: Honcho versus ingebouwd geheugen
  - H2: CLI-opdrachten
  - H2: Verder lezen
  - H2: Gerelateerd

## concepts/memory-qmd.md

- Route: /concepts/memory-qmd
- Koppen:
  - H2: Wat het toevoegt bovenop het ingebouwde geheugen
  - H2: Aan de slag
  - H3: Vereisten
  - H3: Inschakelen
  - H2: Hoe de sidecar werkt
  - H2: Zoekprestaties en compatibiliteit
  - H2: Modeloverschrijvingen
  - H2: Extra paden indexeren
  - H2: Sessietranscripten indexeren
  - H2: Zoekbereik
  - H2: Verwijzingen
  - H2: Wanneer te gebruiken
  - H2: Problemen oplossen
  - H2: Configuratie
  - H2: Gerelateerd

## concepts/memory-search.md

- Route: /concepts/memory-search
- Koppen:
  - H2: Snel aan de slag
  - H2: Ondersteunde aanbieders
  - H2: Hoe zoeken werkt
  - H2: Zoekkwaliteit verbeteren
  - H3: Temporeel verval
  - H3: MMR (diversiteit)
  - H3: Beide inschakelen
  - H2: Multimodaal geheugen
  - H2: Zoeken in sessiegeheugen
  - H2: Problemen oplossen
  - H2: Gerelateerd

## concepts/memory.md

- Route: /concepts/memory
- Koppen:
  - H2: Hoe het werkt
  - H2: Wat waar terechtkomt
  - H2: Importeren uit programmeerassistenten
  - H2: Actiegevoelige herinneringen
  - H2: Buiten gebruik gestelde afgeleide toezeggingen
  - H2: Geheugenhulpmiddelen
  - H2: Geheugen doorzoeken
  - H2: Geheugenbackends
  - H2: Kenniswikilaag
  - H2: Automatisch geheugen wegschrijven
  - H2: Dreaming
  - H2: Onderbouwde aanvulling en livepromotie
  - H2: CLI
  - H2: Verder lezen

## concepts/message-lifecycle-refactor.md

- Route: /concepts/message-lifecycle-refactor
- Koppen:
  - H2: Waarom deze herstructurering plaatsvond
  - H2: Wat is uitgebracht
  - H3: Verzendcontext
  - H3: Ontvangstcontext
  - H3: Livevoorbeeld
  - H3: Duurzame ontvangstbewijzen
  - H3: Verkleining van de openbare SDK
  - H2: Waar de implementatie afweek van het oorspronkelijke ontwerp
  - H2: Concrete migratierisico's (nog steeds relevant)
  - H2: Foutclassificatie
  - H2: Openstaande vragen
  - H2: Gerelateerd

## concepts/messages.md

- Route: /concepts/messages
- Koppen:
  - H2: Deduplicatie van inkomende berichten
  - H2: Debouncing van inkomende berichten
  - H2: Sessies en apparaten
  - H2: Promptinhoud en geschiedeniscontext
  - H2: Metadata van hulpmiddelresultaten
  - H2: Wachtrijen en vervolgberichten
  - H2: Eigenaarschap van kanaaluitvoeringen
  - H2: Streaming, segmentering en batchverwerking
  - H2: Zichtbaarheid van redeneringen en tokens
  - H2: Voorvoegsels, threads en antwoorden
  - H2: Stille antwoorden
  - H2: Gerelateerd

## concepts/model-failover.md

- Route: /concepts/model-failover
- Koppen:
  - H2: Runtimeverloop
  - H2: Beleid voor selectiebronnen
  - H2: Cache voor het overslaan van authenticatiefouten
  - H2: Voor de gebruiker zichtbare meldingen over terugval
  - H2: Opslag van authenticatiegegevens (sleutels + OAuth)
  - H2: Profiel-ID's
  - H2: Rotatievolgorde
  - H3: Sessiegebondenheid (cachevriendelijk)
  - H3: OpenAI Codex-abonnement plus API-sleutel als reserve
  - H2: Afkoelperioden
  - H2: Uitschakelingen vanwege facturering
  - H2: Modelterugval
  - H3: Regels voor de kandidatenketen
  - H3: Welke fouten de terugval voortzetten
  - H3: Gedrag bij overslaan vanwege afkoeling versus testen
  - H2: Sessieoverschrijvingen en live wisselen van model
  - H2: Observeerbaarheid en foutoverzichten
  - H2: Gerelateerde configuratie

## concepts/model-providers.md

- Route: /concepts/model-providers
- Koppen:
  - H2: Beknopte regels
  - H2: Aanbieders configureren in de besturingsinterface
  - H2: Door plugins beheerd aanbiedersgedrag
  - H2: Rotatie van API-sleutels
  - H2: Officiële aanbiedersplugins
  - H3: OpenAI
  - H3: Anthropic
  - H3: OpenAI ChatGPT/Codex OAuth
  - H3: Andere gehoste opties op abonnementsbasis
  - H3: OpenCode
  - H3: Google Gemini (API-sleutel)
  - H3: Google Vertex en Gemini CLI
  - H3: Z.AI (GLM)
  - H3: Vercel AI Gateway
  - H3: Andere gebundelde aanbiedersplugins
  - H4: Eigenaardigheden die je moet kennen
  - H2: Aanbieders via models.providers (aangepaste/basis-URL)
  - H3: Moonshot AI (Kimi)
  - H3: Kimi Coding
  - H3: Volcano Engine (Doubao)
  - H3: BytePlus (internationaal)
  - H3: Synthetic
  - H3: MiniMax
  - H3: LM Studio
  - H3: Ollama
  - H3: vLLM
  - H3: SGLang
  - H3: Lokale proxy's (LM Studio, vLLM, LiteLLM enzovoort)
  - H2: CLI-voorbeelden
  - H2: Gerelateerd

## concepts/models.md

- Route: /concepts/models
- Koppen:
  - H2: Selectievolgorde
  - H2: Selectiebron en striktheid van terugvalopties
  - H2: Snel modelbeleid
  - H2: Onboarding
  - H2: "Model is niet toegestaan" (en waarom antwoorden stoppen)
  - H2: /model in de chat
  - H2: CLI
  - H2: Modelregister (models.json)
  - H2: Gerelateerd

## concepts/multi-agent.md

- Route: /concepts/multi-agent
- Koppen:
  - H2: Wat één agent is
  - H2: Paden
  - H3: Modus met één agent (standaard)
  - H2: Agenthelper
  - H2: Snel aan de slag
  - H2: Meerdere agents, meerdere persona's
  - H2: Memory Wiki-kluizen per agent
  - H2: QMD-geheugen doorzoeken tussen agents
  - H2: Eén WhatsApp-nummer, meerdere personen (DM-splitsing)
  - H2: Routeringsregels
  - H2: Meerdere accounts/telefoonnummers
  - H2: Concepten
  - H2: Platformvoorbeelden
  - H2: Veelvoorkomende patronen
  - H2: Sandbox- en toolconfiguratie per agent
  - H2: Gerelateerd

## concepts/multi-user.md

- Route: /concepts/multi-user
- Koppen:
  - H2: Vertrouwensgrens
  - H2: Eigenaarschap en aanwezigheid
  - H2: Concepten
  - H2: Toewijzing van beurten
  - H2: Gerelateerd

## concepts/oauth.md

- Route: /concepts/oauth
- Koppen:
  - H2: De tokenopvang (waarom deze bestaat)
  - H2: Opslag (waar tokens zich bevinden)
  - H2: Hergebruik van de Anthropic Claude CLI
  - H2: OAuth-uitwisseling (hoe aanmelden werkt)
  - H3: Anthropic-installatietoken
  - H3: OpenAI Codex (ChatGPT OAuth)
  - H2: Vernieuwing en vervaldatum
  - H2: Meerdere accounts (profielen) en routering
  - H3: 1) Aanbevolen: afzonderlijke agents
  - H3: 2) Geavanceerd: meerdere profielen in één agent
  - H2: Gerelateerd

## concepts/parallel-specialist-lanes.md

- Route: /concepts/parallel-specialist-lanes
- Koppen:
  - H2: Basisprincipes
  - H2: Aanbevolen uitrol
  - H3: Fase 1: baancontracten en zwaar achtergrondwerk
  - H3: Fase 2: prioriteits- en gelijktijdigheidsregelingen
  - H3: Fase 3: coördinator/verkeersregelaar
  - H2: Minimale sjabloon voor baancontracten
  - H2: Gerelateerd

## concepts/personal-agent-benchmark-pack.md

- Route: /concepts/personal-agent-benchmark-pack
- Koppen:
  - H2: Scenario's
  - H2: Privacymodel
  - H2: Het pakket uitbreiden

## concepts/presence.md

- Route: /concepts/presence
- Koppen:
  - H2: Aanwezigheidsvelden (wat er verschijnt)
  - H2: Producenten (waar aanwezigheid vandaan komt)
  - H3: 1) Zelfvermelding van de Gateway
  - H3: 2) WebSocket-verbinding
  - H4: Waarom tijdelijke verbindingen met het besturingsvlak niet verschijnen
  - H3: 3) Bakens voor systeemgebeurtenissen
  - H3: 4) Node maakt verbinding (rol: node)
  - H2: Regels voor samenvoegen en ontdubbelen (waarom instanceId belangrijk is)
  - H2: TTL en begrensde grootte
  - H2: Kanttekening bij externe/tunnelverbindingen (loopback-IP's)
  - H2: Consumenten
  - H3: Apparatenpagina van de Control UI
  - H3: Tabblad Instances in macOS
  - H2: Tips voor foutopsporing
  - H2: Gerelateerd

## concepts/progress-drafts.md

- Route: /concepts/progress-drafts
- Koppen:
  - H2: Snel aan de slag
  - H2: Wat gebruikers zien
  - H2: Kies een modus
  - H2: Labels configureren
  - H2: Voortgangsregels beheren
  - H3: Detailmodus
  - H3: Opdracht-/uitvoeringstekst
  - H3: Commentaarbaan
  - H3: Statuskop
  - H3: Regellimieten
  - H3: Uitgebreide weergave (Slack)
  - H3: Tool-/taakregels verbergen
  - H2: Kanaalgedrag
  - H2: Afronding
  - H2: Problemen oplossen
  - H2: Gerelateerd

## concepts/qa-e2e-automation.md

- Route: /concepts/qa-e2e-automation
- Koppen:
  - H2: Opdrachtoppervlak
  - H3: QA-uitvoering op basis van een profiel
  - H2: Beheerdersstroom
  - H3: Observatiecontroles
  - H3: Matrix-controlebanen
  - H3: Discord Mantis-scenario's
  - H3: Uitvoerders voor Mantis Slack-desktop- en visuele taken
  - H3: Statuscontrole van de referentiegegevenspool
  - H2: Canonieke scenariodekking
  - H2: QA-referentie voor Discord, Slack, Telegram en WhatsApp
  - H3: Gedeelde CLI-vlaggen
  - H3: Telegram-QA
  - H3: Discord-QA
  - H3: Slack-QA
  - H4: De Slack-werkruimte instellen
  - H3: WhatsApp-QA
  - H3: Convex-referentiegegevenspool
  - H2: Door de repository ondersteunde basisgegevens
  - H2: Mockbanen voor providers
  - H2: Transportadapters
  - H3: Een kanaal toevoegen
  - H3: Namen van scenariohelpers
  - H2: Rapportage
  - H2: Gerelateerde documentatie

## concepts/queue-steering.md

- Route: /concepts/queue-steering
- Koppen:
  - H2: Runtimegrens
  - H2: Modi
  - H2: Burstvoorbeeld
  - H2: Bereik
  - H2: Debounce
  - H2: Gerelateerd

## concepts/queue.md

- Route: /concepts/queue
- Koppen:
  - H2: Waarom
  - H2: Hoe het werkt
  - H2: Standaardwaarden
  - H2: Wachtrijmodi
  - H2: Wachtrijopties
  - H2: Bijsturen en streamen
  - H2: Voorrang
  - H2: Overschrijvingen per sessie
  - H2: Annulering van beurten in de wachtrij
  - H2: Bereik en garanties
  - H2: Problemen oplossen
  - H2: Gerelateerd

## concepts/retry.md

- Route: /concepts/retry
- Koppen:
  - H2: Doelen
  - H2: Standaardwaarden
  - H2: Gedrag
  - H3: Modelproviders
  - H3: Discord
  - H3: Telegram
  - H2: Configuratie
  - H2: Opmerkingen
  - H2: Gerelateerd

## concepts/session-pruning.md

- Route: /concepts/session-pruning
- Koppen:
  - H2: Waarom dit belangrijk is
  - H2: Hoe het werkt
  - H2: Verouderde afbeeldingen opruimen
  - H2: Slimme standaardwaarden
  - H2: In- of uitschakelen
  - H2: Opschonen versus Compaction
  - H2: Verder lezen
  - H2: Gerelateerd

## concepts/session-search.md

- Route: /concepts/session-search
- Koppen:
  - H1: Sessies doorzoeken
  - H2: Zichtbaarheid en uitvoer
  - H2: Levenscyclus van de index
  - H2: Sessies doorzoeken versus geheugen doorzoeken

## concepts/session-state.md

- Route: /concepts/session-state
- Koppen:
  - H2: Het signaallogboek
  - H2: Bewakers
  - H2: Meldingen: één, niet meerdere
  - H2: Afstemmen
  - H2: Opslag en limieten
  - H2: Gerelateerd

## concepts/session-tool.md

- Route: /concepts/session-tool
- Koppen:
  - H2: Beschikbare tools
  - H2: Sessies weergeven en lezen
  - H2: Sessie-instellingen en groepen beheren
  - H2: Sessies versus gesprekken
  - H2: Berichten tussen sessies verzenden
  - H2: Helpers voor status en orkestratie
  - H2: Wijzigingen in de sessiestatus
  - H2: Subagents starten
  - H2: Zichtbaarheid
  - H2: Verder lezen
  - H2: Gerelateerd

## concepts/session.md

- Route: /concepts/session
- Koppen:
  - H2: Hoe berichten worden gerouteerd
  - H2: DM-isolatie
  - H3: Aan Dock gekoppelde kanalen
  - H2: Incognitosessies
  - H2: Onthouden tussen gesprekken
  - H2: Levenscyclus van sessies
  - H2: Waar de status zich bevindt
  - H2: Sessieonderhoud
  - H2: Sessies inspecteren
  - H2: Verder lezen
  - H2: Gerelateerd

## concepts/soul.md

- Route: /concepts/soul
- Koppen:
  - H2: Wat in SOUL.md thuishoort
  - H2: Waarom dit werkt
  - H2: De Molty-prompt
  - H2: Hoe goed eruitziet
  - H2: Eén waarschuwing
  - H2: Gerelateerd

## concepts/streaming.md

- Route: /concepts/streaming
- Koppen:
  - H2: Opstartstatus van de Control UI
  - H2: Bloksgewijs streamen (kanaalberichten)
  - H3: Media bezorgen bij bloksgewijs streamen
  - H2: Algoritme voor segmentering (onder-/bovengrenzen)
  - H2: Samenvoegen (gestreamde blokken combineren)
  - H2: Menselijk tempo tussen blokken
  - H2: "Segmenten of alles streamen"
  - H2: Modi voor voorbeeldstreaming
  - H3: Kanaaltoewijzing
  - H3: Migratie van verouderde sleutels
  - H2: Runtimegedrag
  - H3: Telegram
  - H3: Discord
  - H3: Slack
  - H3: Mattermost
  - H3: Matrix
  - H2: Voorbeeldupdates van toolvoortgang
  - H2: Weergave van voortgangsconcepten
  - H3: Commentaarbaan voor voortgang
  - H2: Gerelateerd

## concepts/system-prompt.md

- Route: /concepts/system-prompt
- Koppen:
  - H2: Structuur
  - H2: Promptmodi
  - H2: Promptmomentopnamen
  - H2: Injectie van werkruimtebootstrap
  - H2: Tijdverwerking
  - H2: Skills
  - H2: Documentatie
  - H2: Gerelateerd

## concepts/timezone.md

- Route: /concepts/timezone
- Koppen:
  - H2: Drie tijdzoneoppervlakken
  - H2: De tijdzone van de gebruiker instellen
  - H2: Tijdzonewaarden van de envelop
  - H2: Wanneer overschrijven
  - H2: Gerelateerd

## concepts/typebox.md

- Route: /concepts/typebox
- Koppen:
  - H2: Mentaal model (30 seconden)
  - H2: Waar de schema's zich bevinden
  - H2: Huidige pijplijn
  - H2: Hoe de schema's tijdens runtime worden gebruikt
  - H2: Voorbeeldframes
  - H2: Minimale client (Node.js)
  - H2: Uitgewerkt voorbeeld: een methode van begin tot eind toevoegen
  - H2: Gedrag van Swift-codegeneratie
  - H2: Versiebeheer en compatibiliteit
  - H2: Schemapatronen en conventies
  - H2: Live schema-JSON
  - H2: Wanneer je schema's wijzigt
  - H2: Gerelateerd

## concepts/typing-indicators.md

- Route: /concepts/typing-indicators
- Koppen:
  - H2: Standaardwaarden
  - H2: Modi
  - H2: Configuratie
  - H2: Opmerkingen
  - H2: Gerelateerd

## concepts/usage-tracking.md

- Route: /concepts/usage-tracking
- Koppen:
  - H2: Wat het is
  - H2: Waar het wordt weergegeven
  - H2: Kostengeschiedenis van Anthropic en OpenAI
  - H2: Standaardmodus voor de gebruiksvoettekst
  - H3: Drie afzonderlijke sessiestatussen
  - H3: Prioriteitsvolgorde
  - H3: Opnieuw instellen versus uitschakelen
  - H3: Gedrag van de schakelaar
  - H3: Configuratie
  - H2: Aangepaste volledige /usage-voettekst
  - H3: Vorm
  - H3: Contractpaden
  - H3: Werkwoorden
  - H3: Onderdeelvormen
  - H3: Voorbeeld
  - H2: Providers + referenties
  - H2: Gerelateerd

## date-time.md

- Route: /date-time
- Koppen:
  - H2: Berichtenenveloppen (standaard lokaal)
  - H3: Voorbeelden
  - H2: Systeemprompt: huidige datum en tijd
  - H2: Systeemgebeurtenisregels (standaard lokaal)
  - H3: Tijdzone en notatie van gebruiker configureren
  - H2: Automatische detectie van tijdnotatie
  - H2: Toolpayloads + connectors (onbewerkte providertijd + genormaliseerde velden)
  - H2: Gerelateerde documentatie

## debug/node-issue.md

- Route: /debug/node-issue
- Koppen:
  - H1: Crash in Node + tsx: "\\name is geen functie"
  - H2: Status
  - H2: Oorspronkelijk symptoom
  - H2: Oorzaak
  - H2: Huidige reproductiecontrole
  - H2: Tijdelijke oplossingen (als de crash terugkeert)
  - H2: Referenties
  - H2: Gerelateerd

## diagnostics/flags.md

- Route: /diagnostics/flags
- Koppen:
  - H2: Hoe het werkt
  - H2: Bekende vlaggen
  - H2: Inschakelen via configuratie
  - H2: Omgevingsoverschrijving (eenmalig)
  - H2: Profilervlaggen
  - H2: Tijdlijnartefacten
  - H2: Waar logboeken worden opgeslagen
  - H2: Logboeken extraheren
  - H2: Opmerkingen
  - H2: Gerelateerd

## gateway/1password.md

- Route: /gateway/1password
- Koppen:
  - H2: Vereisten
  - H2: Configuratiegeheimen oplossen met op
  - H2: Serviceaccount instellen voor headless Gateways
  - H2: De 1password-Skill voor agents
  - H2: Aanmelden in de browser met 1Password voor Claude
  - H2: Beveiligingsopmerkingen
  - H2: Problemen oplossen

## gateway/audit.md

- Route: /gateway/audit
- Koppen:
  - H1: Auditgeschiedenis
  - H2: Recordfamilies
  - H2: Gebeurtenissen in de levenscyclus van berichten
  - H3: Classificatie van gesprekstypen
  - H2: Privacymodel
  - H2: Dekkings- en bewijslimieten
  - H2: Opslag, bewaartermijn en migratie
  - H2: Query's uitvoeren
  - H2: Gerelateerd

## gateway/authentication.md

- Route: /gateway/authentication
- Koppen:
  - H2: Aanbevolen instelling: API-sleutel (elke provider)
  - H2: Anthropic: Claude CLI hergebruiken
  - H2: Token handmatig invoeren
  - H3: Door SecretRef ondersteunde referenties
  - H2: Authenticatiestatus van het model controleren
  - H2: API-sleutelrotatie (Gateway)
  - H2: Providerauthenticatie verwijderen terwijl de Gateway actief is
  - H2: Bepalen welke referentie wordt gebruikt
  - H3: OpenAI- en verouderde openai-codex-id's
  - H3: Tijdens het aanmelden (CLI)
  - H3: Per sessie (chatopdracht)
  - H3: Per agent (CLI-overschrijving)
  - H2: Problemen oplossen
  - H3: "Geen referenties gevonden"
  - H3: Token verloopt/is verlopen
  - H2: Gerelateerd

## gateway/background-process.md

- Route: /gateway/background-process
- Koppen:
  - H2: exec-tool
  - H3: Omgevingsoverschrijvingen
  - H3: Configuratie (voorkeur boven omgevingsoverschrijvingen)
  - H2: Koppeling van onderliggende processen
  - H2: process-tool
  - H2: Voorbeelden
  - H2: Gerelateerd

## gateway/bonjour.md

- Route: /gateway/bonjour
- Koppen:
  - H2: Wide-area Bonjour (Unicast DNS-SD) via Tailscale
  - H3: Gateway-configuratie
  - H3: Eenmalige instelling van de DNS-server (Gateway-host, alleen macOS)
  - H3: Tailscale-DNS-instellingen
  - H3: Beveiliging van de Gateway-listener
  - H2: Wat wordt aangekondigd
  - H2: Servicetypen
  - H2: TXT-sleutels (niet-geheime hints)
  - H2: Foutopsporing op macOS
  - H2: Foutopsporing in Gateway-logboeken
  - H2: Foutopsporing op een iOS-Node
  - H2: Wanneer Bonjour moet worden ingeschakeld
  - H2: Wanneer Bonjour moet worden uitgeschakeld
  - H2: Valkuilen bij Docker
  - H2: Problemen met uitgeschakelde Bonjour oplossen
  - H2: Veelvoorkomende foutmodi
  - H2: Geëscapete instantienamen (\032)
  - H2: Inschakelen / uitschakelen / configureren
  - H2: Gerelateerde documentatie

## gateway/bridge-protocol.md

- Route: /gateway/bridge-protocol
- Koppen:
  - H2: Waarom het bestond
  - H2: Transport
  - H2: Handshake en koppeling
  - H2: Frames
  - H2: Gebeurtenissen in de levenscyclus van exec
  - H2: Historisch gebruik van tailnet
  - H2: Versiebeheer
  - H2: Gerelateerd

## gateway/cli-backends.md

- Route: /gateway/cli-backends
- Koppen:
  - H2: Snel aan de slag
  - H2: Gebruiken als terugvaloptie
  - H2: Configuratie
  - H2: Hoe het werkt
  - H2: Time-outs en langlopende taken
  - H3: Details van Claude CLI
  - H3: Browsertools van Claude en aanmelden met 1Password
  - H2: Sessies
  - H2: Terugvalinleiding uit claude-cli-sessies
  - H2: Afbeeldingen
  - H2: Invoer en uitvoer
  - H2: Standaardinstellingen die eigendom zijn van de Plugin
  - H2: Overlays voor teksttransformatie
  - H2: Eigenaarschap van native Compaction
  - H2: MCP-overlays bundelen
  - H2: Limiet voor heringevoerde geschiedenis
  - H2: Beperkingen
  - H2: Problemen oplossen
  - H2: Gerelateerd

## gateway/clients.md

- Route: /gateway/clients
- Koppen:
  - H2: De pakketten installeren
  - H2: Scopes kiezen en het apparaat koppelen
  - H2: Clientmogelijkheden aankondigen
  - H2: Status herstellen na opnieuw verbinden
  - H2: Geschiedenismetadata en stabiele ankers gebruiken
  - H2: Abonneren in plaats van gebruik te pollen
  - H2: Exec-goedkeuringen aanvullen
  - H2: Protocolversies bijhouden
  - H2: Gerelateerd

## gateway/cloud-workers.md

- Route: /gateway/cloud-workers
- Koppen:
  - H2: Wat waar wordt uitgevoerd
  - H2: Vereisten
  - H2: Configuratie
  - H3: De instellingsopdracht
  - H3: Kanalen installeren
  - H2: Een sessie verzenden
  - H2: Beveiligingsmodel
  - H2: Problemen oplossen
  - H2: Gerelateerd

## gateway/config-agents.md

- Route: /gateway/config-agents
- Koppen:
  - H2: Standaardinstellingen voor agents
  - H3: agents.defaults.workspace
  - H3: agents.defaults.repoRoot
  - H3: agents.defaults.skills
  - H3: agents.defaults.skipBootstrap
  - H3: agents.defaults.skipOptionalBootstrapFiles
  - H3: agents.defaults.contextInjection
  - H3: agents.defaults.bootstrapMaxChars
  - H3: agents.defaults.bootstrapTotalMaxChars
  - H3: Overschrijvingen van het bootstrapprofiel per agent
  - H3: agents.defaults.bootstrapPromptTruncationWarning
  - H3: Eigenaarschapskaart voor contextbudgetten
  - H4: agents.defaults.startupContext
  - H4: agents.defaults.contextLimits
  - H4: `agents.entries.*.contextLimits`
  - H4: skills.limits.maxSkillsPromptChars
  - H4: `agents.entries.*.skillsLimits.maxSkillsPromptChars`
  - H3: agents.defaults.imageMaxDimensionPx
  - H3: agents.defaults.imageQuality
  - H3: agents.defaults.userTimezone
  - H3: agents.defaults.timeFormat
  - H3: agents.defaults.model
  - H3: Runtimebeleid
  - H3: CLI-backendselectie
  - H3: agents.defaults.promptOverlays
  - H3: agents.defaults.heartbeat
  - H3: agents.defaults.compaction
  - H3: agents.defaults.contextPruning
  - H3: Blokstreaming
  - H3: Typindicatoren
  - H3: agents.defaults.sandbox
  - H3: agents.entries (overschrijvingen per agent)
  - H2: Routering met meerdere agents
  - H3: Velden voor bindingsovereenkomsten
  - H3: Toegangsprofielen per agent
  - H2: Sessie
  - H2: Berichten
  - H3: Antwoordvoorvoegsel
  - H3: Bevestigingsreactie
  - H3: Wachtrij
  - H3: Debounce voor inkomende berichten
  - H3: Overige berichtsleutels
  - H3: TTS (tekst-naar-spraak)
  - H2: Praten
  - H2: Gerelateerd

## gateway/config-channels.md

- Route: /gateway/config-channels
- Koppen:
  - H2: Kanalen
  - H3: Toegang tot privéberichten en groepen
  - H3: Modeloverschrijvingen per kanaal
  - H3: Standaardinstellingen en Heartbeat per kanaal
  - H3: WhatsApp
  - H3: Telegram
  - H3: Discord
  - H3: Google Chat
  - H3: Slack
  - H3: Mattermost
  - H3: Signal
  - H3: iMessage
  - H3: Matrix
  - H3: Microsoft Teams
  - H3: IRC
  - H3: Meerdere accounts (alle kanalen)
  - H3: Overige pluginkanalen
  - H3: Vermeldingspoort voor groepschats
  - H4: Geschiedenislimieten voor privéberichten
  - H4: Zelfchatmodus
  - H3: Opdrachten (afhandeling van chatopdrachten)
  - H2: Gerelateerd

## gateway/config-tools.md

- Route: /gateway/config-tools
- Koppen:
  - H2: Tools
  - H3: Toolprofielen
  - H3: Toolgroepen
  - H3: MCP- en plugintools binnen het sandbox-toolbeleid
  - H3: tools.codeMode
  - H3: tools.allow / tools.deny
  - H3: tools.byProvider
  - H3: tools.toolsBySender
  - H3: tools.elevated
  - H3: tools.exec
  - H3: tools.loopDetection
  - H3: tools.web
  - H3: tools.media
  - H3: tools.agentToAgent
  - H3: tools.sessions
  - H3: `tools.sessions_spawn`
  - H3: tools.experimental
  - H3: agents.defaults.subagents
  - H2: Aangepaste providers en basis-URL's
  - H3: Details van providervelden
  - H3: Providervoorbeelden
  - H2: Gerelateerd

## gateway/configuration-examples.md

- Route: /gateway/configuration-examples
- Koppen:
  - H2: Snel aan de slag
  - H3: Absoluut minimum
  - H3: Aanbevolen startconfiguratie
  - H2: Uitgebreid voorbeeld (belangrijkste opties)
  - H3: Via symlink gekoppelde naastliggende skill-repository
  - H2: Veelgebruikte patronen
  - H3: Gedeelde skill-basis met één overschrijving
  - H3: Multiplatformconfiguratie
  - H3: Automatische goedkeuring voor een vertrouwd Node-netwerk
  - H3: Veilige DM-modus (gedeelde inbox / DM's met meerdere gebruikers)
  - H3: Anthropic-API-sleutel + MiniMax-terugvaloptie
  - H3: Werkbot (beperkte toegang)
  - H3: Alleen lokale modellen
  - H2: Tips
  - H2: Gerelateerd

## gateway/configuration-reference.md

- Route: /gateway/configuration-reference
- Koppen:
  - H2: Kanalen
  - H2: Standaardinstellingen voor agents, meerdere agents, sessies en berichten
  - H2: Hulpmiddelen en aangepaste providers
  - H2: Modellen
  - H2: MCP
  - H2: Skills
  - H2: Plugins
  - H3: Pluginconfiguratie voor de Codex-harness
  - H2: Browser
  - H2: Gebruikersinterface
  - H2: Gateway
  - H3: OpenAI-compatibele eindpunten
  - H3: Isolatie van meerdere instanties
  - H3: gateway.tls
  - H3: gateway.reload
  - H2: Cloudworkeromgevingen
  - H3: Crabbox-profiel
  - H3: Statisch SSH-ontwikkelprofiel
  - H2: Hooks
  - H3: Gmail-integratie
  - H2: Host voor de Canvas-plugin
  - H2: Detectie
  - H3: mDNS (Bonjour)
  - H3: Wide-area (DNS-SD)
  - H2: Omgeving
  - H3: env (inline omgevingsvariabelen)
  - H3: Vervanging van omgevingsvariabelen
  - H2: Geheimen
  - H3: SecretRef
  - H3: Ondersteund referentieoppervlak
  - H3: Configuratie van geheimproviders
  - H2: Opslag van authenticatiegegevens
  - H2: Audit
  - H2: Logboekregistratie
  - H2: Diagnostiek
  - H2: Bijwerken
  - H2: ACP
  - H2: Wizard
  - H2: Identiteit
  - H2: Bridge (verouderd, verwijderd)
  - H2: Cron
  - H3: cron.failureAlert
  - H3: cron.failureDestination
  - H2: Sjabloonvariabelen voor mediamodellen
  - H2: Configuratie-insluitingen ($include)
  - H2: Gerelateerd

## gateway/configuration.md

- Route: /gateway/configuration
- Koppen:
  - H2: Minimale configuratie
  - H2: Configuratie bewerken
  - H2: Strikte validatie
  - H2: Veelvoorkomende taken
  - H2: Hot reload van configuratie
  - H3: Herlaadmodi
  - H3: Wat direct wordt toegepast en waarvoor opnieuw opstarten nodig is
  - H3: Herladen plannen
  - H2: Configuratie-RPC (programmatische updates)
  - H2: Omgevingsvariabelen
  - H2: Volledige referentie
  - H2: Gerelateerd

## gateway/diagnostics.md

- Route: /gateway/diagnostics
- Koppen:
  - H2: Snel aan de slag
  - H2: Chatopdracht
  - H2: Wat de export bevat
  - H2: Privacymodel
  - H2: Stabiliteitsrecorder
  - H2: Nuttige opties
  - H2: Diagnostiek uitschakelen
  - H2: Gerelateerd

## gateway/discovery.md

- Route: /gateway/discovery
- Koppen:
  - H2: Termen
  - H2: Waarom zowel directe verbindingen als SSH bestaan
  - H2: Invoer voor detectie
  - H3: 1) Bonjour / DNS-SD
  - H4: Details van het servicebaken
  - H3: 2) Tailnet (netwerkoverschrijdend)
  - H3: 3) Handmatig / SSH-doel
  - H2: Transportselectie (clientbeleid)
  - H2: Koppeling en authenticatie (direct transport)
  - H2: Verantwoordelijkheden per component
  - H2: Gerelateerd

## gateway/doctor.md

- Route: /gateway/doctor
- Koppen:
  - H2: Snel aan de slag
  - H3: Headless- en automatiseringsmodi
  - H2: Alleen-lezen-lintmodus
  - H2: Wat het doet (samenvatting)
  - H2: Aanvullen en opnieuw instellen van de Dreams-gebruikersinterface
  - H2: Gedetailleerd gedrag en onderbouwing
  - H2: Gerelateerd

## gateway/embedding.md

- Route: /gateway/embedding
- Koppen:
  - H2: Start het onderliggende proces met een insluitingsvoorinstelling
  - H3: Waarschuwing voor Electron-shell-snapshots
  - H2: Handel ongeldige configuratie af via de afsluitcode
  - H2: Wacht tot het protocol gereed is
  - H2: Interpreteer opnieuw opstarten en afsluiten
  - H2: Gebruik RPC in plaats van statusbestanden
  - H2: Installeer; maak de structuur niet vlak
  - H2: Gerelateerd

## gateway/external-apps.md

- Route: /gateway/external-apps
- Koppen:
  - H2: Wat momenteel beschikbaar is
  - H2: Aanbevolen aanpak
  - H2: Coöperatieve opschorting van de host
  - H2: Appcode versus plugincode
  - H2: Gerelateerd

## gateway/gateway-lock.md

- Route: /gateway/gateway-lock
- Koppen:
  - H2: Waarom
  - H2: Drie lagen
  - H3: Vergrendelingen voor status en configuratie
  - H3: Socketbinding
  - H2: Operationele opmerkingen
  - H2: Gerelateerd

## gateway/health.md

- Route: /gateway/health
- Koppen:
  - H2: Snelle controles
  - H2: Grondige diagnostiek
  - H2: Configuratie van de statusmonitor
  - H2: Uptimebewaking
  - H3: Configuratievoorbeelden voor bewakingsservices
  - H2: Wanneer er iets misgaat
  - H2: Speciale opdracht "health"
  - H2: Gerelateerd

## gateway/heartbeat.md

- Route: /gateway/heartbeat
- Koppen:
  - H2: Snel aan de slag (beginners)
  - H2: Standaardinstellingen
  - H2: Waarvoor de Heartbeat-prompt dient
  - H2: Responscontract
  - H2: Configuratie
  - H3: Bereik en prioriteitsvolgorde
  - H3: Heartbeats per agent
  - H3: Voorbeeld met actieve uren
  - H3: 24/7-configuratie
  - H3: Voorbeeld met meerdere accounts
  - H3: Opmerkingen over velden
  - H2: Afleveringsgedrag
  - H2: Zichtbaarheidsinstellingen
  - H3: Wat elke vlag doet
  - H3: Voorbeelden per kanaal versus per account
  - H3: Veelgebruikte patronen
  - H2: Tijdelijke notities voor bewaking (optioneel)
  - H3: Plan terugkerende controles met Cron
  - H3: Kan de agent zijn tijdelijke notities bijwerken?
  - H2: Handmatig activeren (op aanvraag)
  - H2: Kostenbewustzijn
  - H2: Contextoverloop na Heartbeat
  - H2: Gerelateerd

## gateway/index.md

- Route: /gateway
- Koppen:
  - H2: Lokaal opstarten in 5 minuten
  - H2: Runtimemodel
  - H2: OpenAI-compatibele eindpunten
  - H3: Prioriteitsvolgorde van poort en binding
  - H3: Hot-reloadmodi
  - H2: Opdrachtenset voor operators
  - H2: Meerdere Gateways (dezelfde host)
  - H2: Externe toegang
  - H2: Toezicht en levenscyclus van services
  - H2: Snelle aanpak met een ontwikkelprofiel
  - H2: Beknopt protocoloverzicht (operatorweergave)
  - H2: Operationele controles
  - H3: Activiteit
  - H3: Gereedheid
  - H3: Herstel van hiaten
  - H2: Veelvoorkomende foutsignaturen
  - H2: Veiligheidsgaranties
  - H2: Gerelateerd

## gateway/local-model-services.md

- Route: /gateway/local-model-services
- Koppen:
  - H2: Hoe het werkt
  - H2: Configuratiestructuur
  - H2: Velden
  - H2: Inferrs-voorbeeld
  - H2: ds4-voorbeeld
  - H2: Gerelateerd

## gateway/local-models.md

- Route: /gateway/local-models
- Koppen:
  - H2: Minimale hardwarevereisten
  - H2: Kies een backend
  - H2: LM Studio + groot lokaal model (Responses API)
  - H3: Hybride configuratie: gehost primair model, lokale terugvaloptie
  - H3: Regionale hosting / gegevensroutering
  - H2: Andere OpenAI-compatibele lokale proxy's
  - H2: Kleinere of strengere backends
  - H2: Problemen oplossen
  - H2: Gerelateerd

## gateway/logging.md

- Route: /gateway/logging
- Koppen:
  - H1: Logboekregistratie
  - H2: Bestandsgebaseerde logger
  - H3: Uitgebreide uitvoer versus logniveaus
  - H2: Console-uitvoer vastleggen
  - H2: Redactie
  - H2: Gateway-WebSocket-logboeken
  - H3: WS-logstijl
  - H2: Consoleopmaak (logboekregistratie per subsysteem)
  - H2: Gerelateerd

## gateway/multi-tenant-hosting.md

- Route: /gateway/multi-tenant-hosting
- Koppen:
  - H1: Hosting voor meerdere tenants
  - H2: Waarom elke tenant een cel nodig heeft
  - H2: Architectuur
  - H2: Vertrouwensgrens
  - H2: Isolatieniveaus
  - H2: Snel aan de slag
  - H2: Huidig bereik
  - H2: Gerelateerd

## gateway/multiple-gateways.md

- Route: /gateway/multiple-gateways
- Koppen:
  - H2: Snel aan de slag met de reddingsbot
  - H3: Wat --profile rescue onboard wijzigt
  - H2: Algemene configuratie voor meerdere Gateways
  - H2: Isolatiechecklist
  - H2: Poorttoewijzing (afgeleid)
  - H2: Opmerkingen over browser/CDP (veelvoorkomende valkuil)
  - H2: Handmatig voorbeeld met omgevingsvariabelen
  - H2: Snelle controles
  - H2: Gerelateerd

## gateway/network-model.md

- Route: /gateway/network-model
- Koppen:
  - H2: Gerelateerd

## gateway/openai-http-api.md

- Route: /gateway/openai-http-api
- Koppen:
  - H2: Het eindpunt inschakelen
  - H2: Beveiligingsgrens (belangrijk)
  - H2: Authenticatie
  - H2: Wanneer je dit eindpunt gebruikt
  - H2: Agent-first-modelcontract
  - H2: Sessiegedrag
  - H2: Aanvraaglimieten
  - H2: Contract voor chathulpmiddelen
  - H3: Ondersteunde aanvraagvelden
  - H3: Niet-ondersteunde varianten
  - H3: Structuur van een niet-streamende hulpmiddelrespons
  - H3: Structuur van een streamende hulpmiddelrespons
  - H3: Opvolglus voor hulpmiddelen
  - H2: Streaming (SSE)
  - H2: Open WebUI snel configureren
  - H2: Voorbeelden
  - H2: Gerelateerd

## gateway/openresponses-http-api.md

- Route: /gateway/openresponses-http-api
- Koppen:
  - H2: Authenticatie, beveiliging en routering
  - H2: Sessiegedrag
  - H2: Aanvraagstructuur
  - H2: Items (invoer)
  - H3: bericht
  - H3: `function_call_output` (beurtgebaseerde tools)
  - H3: redenering en `item_reference`
  - H2: Tools (functie-tools aan clientzijde)
  - H2: Afbeeldingen (`input_image`)
  - H2: Bestanden (`input_file`)
  - H2: Limieten voor bestanden en afbeeldingen
  - H2: Streaming (SSE)
  - H2: Gebruik
  - H2: Fouten
  - H2: Voorbeelden
  - H2: Gerelateerd

## gateway/openshell.md

- Route: /gateway/openshell
- Koppen:
  - H2: Vereisten
  - H2: Snel aan de slag
  - H2: Werkruimtemodi
  - H3: mirror (standaard)
  - H3: remote
  - H3: Een modus kiezen
  - H2: Configuratiereferentie
  - H2: Voorbeelden
  - H3: Minimale externe configuratie
  - H3: Mirror-modus met GPU
  - H3: OpenShell per agent met aangepaste Gateway
  - H2: Levenscyclusbeheer
  - H2: Beveiligingsversterking
  - H2: Huidige beperkingen
  - H2: Hoe het werkt
  - H2: Gerelateerd

## gateway/opentelemetry.md

- Route: /gateway/opentelemetry
- Koppen:
  - H2: Snel aan de slag
  - H2: Geëxporteerde signalen
  - H2: Configuratiereferentie
  - H3: Omgevingsvariabelen
  - H2: Privacy en inhoudsvastlegging
  - H2: Bemonstering en wegschrijven
  - H3: Observatie-eenheden voor modelaanroepen
  - H3: Getrouwheid van modelaanroepen via de Claude Code CLI
  - H2: Geëxporteerde metrische gegevens
  - H3: Modelgebruik
  - H3: Berichtenstroom
  - H3: Spraak
  - H3: Wachtrijen en sessies
  - H3: Telemetrie voor sessieactiviteit
  - H3: Levenscyclus van de testomgeving
  - H3: Tooluitvoering en lusdetectie
  - H3: Uitvoering
  - H3: Interne diagnostiek (geheugen, payloads, status van exporteurs)
  - H2: Geëxporteerde spans
  - H2: Catalogus met diagnostische gebeurtenissen
  - H2: Zonder exporteur
  - H2: Uitschakelen
  - H2: Gerelateerd

## gateway/operator-scopes.md

- Route: /gateway/operator-scopes
- Koppen:
  - H2: Rollen
  - H2: Bereikniveaus
  - H2: Methodebereik is slechts de eerste poort
  - H2: Goedkeuringen voor apparaatkoppeling
  - H2: Goedkeuringen voor Node-koppeling
  - H2: Authenticatie met gedeeld geheim

## gateway/pairing.md

- Route: /gateway/pairing
- Koppen:
  - H2: Hoe goedkeuring van mogelijkheden werkt
  - H2: CLI-workflow (geschikt voor headless gebruik)
  - H2: API-oppervlak (Gateway-protocol)
  - H2: Toegangscontrole voor Node-opdrachten (2026.3.31+)
  - H2: Vertrouwensgrenzen voor Node-gebeurtenissen (2026.3.31+)
  - H2: Automatische goedkeuring van via SSH geverifieerde apparaten (standaard)
  - H2: Automatische goedkeuring (macOS-app)
  - H2: Automatische goedkeuring van apparaten via vertrouwde CIDR
  - H2: Stille opschoning bij vervanging van koppelingen
  - H2: Automatische goedkeuring van metadata-upgrades
  - H2: Hulpmiddelen voor QR-koppeling
  - H2: Lokaliteit en doorgestuurde headers
  - H2: Opslag (lokaal, privé)
  - H2: Transportgedrag
  - H2: Gerelateerd

## gateway/prometheus.md

- Route: /gateway/prometheus
- Koppen:
  - H2: Snel aan de slag
  - H2: Geëxporteerde metrische gegevens
  - H2: Labelbeleid
  - H2: PromQL-recepten
  - H2: Kiezen tussen export via Prometheus en OpenTelemetry
  - H2: Problemen oplossen
  - H2: Gerelateerd

## gateway/protocol.md

- Route: /gateway/protocol
- Koppen:
  - H2: npm-pakketten
  - H2: Transport en framing
  - H2: Handshake
  - H3: Workerrol en gesloten protocol
  - H3: Clientmogelijkheden
  - H3: Voorbeeld van een Node-verbinding
  - H2: Rollen en bereiken
  - H3: Mogelijkheden/opdrachten/machtigingen (Node)
  - H2: Aanwezigheid
  - H3: Gebeurtenis voor actieve Node op de achtergrond
  - H2: Bereik van broadcastgebeurtenissen
  - H2: RPC-methodefamilies
  - H3: Algemene gebeurtenisfamilies
  - H3: Hulpmethoden voor Node
  - H2: RPC voor auditlogboek
  - H2: RPC's voor taaklogboeken
  - H2: Hulpmethoden voor operators
  - H3: models.list-weergaven
  - H2: Goedkeuringen voor uitvoering
  - H2: Terugvaloptie voor agentlevering
  - H2: Versiebeheer
  - H3: Clientconstanten
  - H2: Authenticatie
  - H2: Apparaatidentiteit en koppeling
  - H3: Diagnostiek voor migratie van apparaatauthenticatie
  - H2: TLS en pinning
  - H2: Bereik
  - H2: Gerelateerd

## gateway/remote-gateway-readme.md

- Route: /gateway/remote-gateway-readme
- Koppen:
  - H1: OpenClaw.app uitvoeren met een externe Gateway
  - H2: Configuratie
  - H2: Hoe het werkt
  - H2: Gerelateerd

## gateway/remote.md

- Route: /gateway/remote
- Koppen:
  - H2: Het kernidee
  - H2: Topologieopties
  - H2: Opdrachtstroom (wat waar wordt uitgevoerd)
  - H2: SSH-tunnel (CLI + tools)
  - H2: Standaardinstellingen voor externe CLI
  - H2: Prioriteitsvolgorde van aanmeldgegevens
  - H2: Externe toegang tot de chatinterface
  - H2: Externe modus van de macOS-app
  - H2: Beveiligingsregels (extern/VPN)
  - H3: macOS: permanente SSH-tunnel via LaunchAgent
  - H4: Stap 1: SSH-configuratie toevoegen
  - H4: Stap 2: SSH-sleutel kopiëren (eenmalig)
  - H4: Stap 3: het Gateway-token configureren
  - H4: Stap 4: de LaunchAgent maken
  - H4: Stap 5: de LaunchAgent laden
  - H4: Problemen oplossen
  - H2: Gerelateerd

## gateway/restart-recovery.md

- Route: /gateway/restart-recovery
- Koppen:
  - H2: Wat een herstart overleeft
  - H2: Gecontroleerde herstarts handelen eerst lopende taken af
  - H2: Hoe onderbroken werk wordt gedetecteerd
  - H2: Automatisch hervatten
  - H3: Subagenten
  - H3: Achtergrondtaken
  - H3: Door agents aangevraagde herstarts
  - H2: Veiligheidsmechanismen en observeerbaarheid
  - H2: Wat niet wordt hervat

## gateway/sandbox-vs-tool-policy-vs-elevated.md

- Route: /gateway/sandbox-vs-tool-policy-vs-elevated
- Koppen:
  - H2: Snel debuggen
  - H2: Sandbox: waar tools worden uitgevoerd
  - H3: Bind-mounts (snelle beveiligingscontrole)
  - H2: Toolbeleid: welke tools bestaan/aanroepbaar zijn
  - H3: Toolgroepen (afkortingen)
  - H2: Verhoogd: alleen voor uitvoering, „uitvoeren op host”
  - H2: Veelvoorkomende oplossingen voor „sandbox jail”
  - H3: „Tool X geblokkeerd door het toolbeleid van de sandbox”
  - H3: „Ik dacht dat dit main was; waarom draait het in een sandbox?”
  - H2: Gerelateerd

## gateway/sandboxing.md

- Route: /gateway/sandboxing
- Koppen:
  - H2: Wat in een sandbox wordt uitgevoerd
  - H2: Modi, bereik en backend
  - H2: Docker-backend
  - H3: Browser in sandbox
  - H2: SSH-backend
  - H2: OpenShell-backend
  - H2: Werkruimtetoegang
  - H2: Meerdere mappen voor één agent
  - H3: Ander bind-gedrag
  - H2: Images en configuratie
  - H2: setupCommand (eenmalige containerconfiguratie)
  - H2: Toolbeleid en ontsnappingsmogelijkheden
  - H2: Overschrijvingen voor meerdere agents
  - H2: Minimaal inschakelvoorbeeld
  - H2: Gerelateerd

## gateway/secrets-plan-contract.md

- Route: /gateway/secrets-plan-contract
- Koppen:
  - H2: Vereisten voor het planbestand
  - H2: Structuur van het planbestand
  - H2: Providers bijwerken of invoegen en verwijderen
  - H2: Ondersteund doelbereik
  - H2: Gedrag per doeltype
  - H2: Regels voor padvalidatie
  - H2: Gedrag bij fouten
  - H2: Toestemmingsgedrag voor uitvoeringsproviders
  - H2: Opmerkingen over runtime- en auditbereik
  - H2: Operatorcontroles
  - H2: Gerelateerde documentatie

## gateway/secrets.md

- Route: /gateway/secrets
- Koppen:
  - H2: Runtimemodel
  - H2: Injectie bij uitgaand verkeer (sentinels)
  - H2: Toegangsgrens voor agents
  - H2: Filtering van actieve oppervlakken
  - H2: Diagnostiek voor het authenticatieoppervlak van de Gateway
  - H2: Preflight voor onboardingreferenties
  - H2: SecretRef-contract
  - H2: Providerconfiguratie
  - H2: API-sleutels uit bestanden
  - H2: Voorbeelden van uitvoeringsintegratie
  - H2: Omgevingsvariabelen voor MCP-servers
  - H2: SSH-authenticatiemateriaal voor de sandbox
  - H2: Ondersteund aanmeldgegevensoppervlak
  - H2: Vereist gedrag en prioriteitsvolgorde
  - H2: Activeringstriggers
  - H2: Signalen voor verslechtering en herstel
  - H2: Resolutie van opdrachtpaden
  - H2: Workflow voor audit en configuratie
  - H2: Eenrichtingsveiligheidsbeleid
  - H2: Opmerkingen over compatibiliteit met verouderde authenticatie
  - H2: Opmerking over de webinterface
  - H2: Gerelateerd

## gateway/security/audit-checks.md

- Route: /gateway/security/audit-checks
- Koppen:
  - H2: Gerelateerd

## gateway/security/exposure-runbook.md

- Route: /gateway/security/exposure-runbook
- Koppen:
  - H2: Het blootstellingspatroon kiezen
  - H2: Preflight-inventarisatie
  - H2: Basiscontroles
  - H2: Minimale veilige basisconfiguratie
  - H2: Blootstelling via privéberichten en groepen
  - H2: Controles voor reverse proxy's
  - H2: Beoordeling van tools en sandbox
  - H2: Validatie na wijzigingen
  - H2: Terugdraaiplan
  - H2: Beoordelingschecklist

## gateway/security/index.md

- Route: /gateway/security
- Koppen:
  - H2: Reikwijdte: beveiligingsmodel voor persoonlijke assistenten
  - H2: OpenClaw-beveiligingsaudit
  - H3: Wat de audit controleert (op hoofdlijnen)
  - H3: Prioriteitsvolgorde bij het beoordelen van bevindingen
  - H2: Versterkte basisconfiguratie in 60 seconden
  - H3: Controles per aanvrager en promptcontext
  - H2: Matrix met vertrouwensgrenzen
  - H2: Door het ontwerp geen kwetsbaarheden
  - H2: Vertrouwen in Gateway en Node
  - H2: Dreigingsmodel
  - H2: DM-toegang: koppeling, toelatingslijst, open, uitgeschakeld
  - H3: Toelatingslijsten (twee lagen)
  - H3: Isolatie van DM-sessies (modus voor meerdere gebruikers)
  - H2: Zichtbaarheid van context versus autorisatie van triggers
  - H2: Promptinjectie
  - H3: Externe inhoud en inkapseling van niet-vertrouwde invoer
  - H3: Omzeilingsvlaggen (uitgeschakeld houden in productie)
  - H3: Redenering en uitgebreide uitvoer in groepen
  - H2: Autorisatie van opdrachten
  - H2: Hulpmiddelen voor het besturingsvlak
  - H2: Uitvoering op een Node (system.run)
  - H2: Dynamische Skills (watcher / externe Nodes)
  - H2: Plugins
  - H2: Sandboxing
  - H3: Beveiliging voor delegatie aan subagents
  - H3: Alleen-lezenmodus
  - H2: Toegangsprofielen per agent (meerdere agents)
  - H3: Volledige toegang (geen sandbox)
  - H3: Alleen-lezenhulpmiddelen + alleen-lezenwerkruimte
  - H3: Geen toegang tot bestandssysteem/shell (berichten via providers toegestaan)
  - H2: Risico's van browserbesturing
  - H3: SSRF-beleid voor browsers (standaard strikt)
  - H2: Netwerkblootstelling
  - H3: Bindadres, poort, firewall
  - H3: Docker-poorten publiceren met UFW
  - H3: Detectie via mDNS/Bonjour
  - H3: WebSocket-authenticatie voor de Gateway
  - H3: Identiteitsheaders van Tailscale Serve
  - H3: Configuratie van reverse proxy
  - H3: Opmerkingen over HSTS en origins
  - H3: Besturingsinterface via HTTP
  - H3: Onveilige/gevaarlijke vlaggen
  - H2: Vertrouwen in implementatie en host
  - H2: Geheimen op schijf
  - H3: Overzicht van opslag van referenties
  - H3: Bestandsmachtigingen
  - H3: .env-bestanden in de werkruimte
  - H3: Logboeken en transcripties
  - H2: Veilige basisconfiguratie (kopiëren/plakken)
  - H3: Afzonderlijke nummers (WhatsApp, Signal, Telegram)
  - H2: Incidentrespons
  - H3: Inperken
  - H3: Vervangen (ga uit van compromittering als geheimen zijn gelekt)
  - H3: Controleren
  - H3: Verzamelen voor een rapport
  - H2: Scannen op geheimen
  - H2: Beveiligingsproblemen melden

## gateway/security/rate-limiting.md

- Route: /gateway/security/rate-limiting
- Koppen:
  - H2: Authenticatiepogingen (vóór authenticatie)
  - H3: Verbindingen vanuit browsers
  - H3: Webhooks
  - H2: Schrijfbewerkingen op het besturingsvlak (vangnet na authenticatie)
  - H2: ACP-sessies aanmaken
  - H2: Afkoelperiode voor herstarten
  - H2: Operationele opmerkingen

## gateway/security/secure-file-operations.md

- Route: /gateway/security/secure-file-operations
- Koppen:
  - H2: Standaard: geen Python-helper
  - H2: Wat zonder Python beschermd blijft
  - H2: Wat Python toevoegt
  - H2: Richtlijnen voor Plugins en de kern

## gateway/security/shrinkwrap.md

- Route: /gateway/security/shrinkwrap
- Koppen:
  - H2: Waarom dit belangrijk is
  - H2: Genereren en controleren
  - H2: Een gepubliceerd pakket inspecteren

## gateway/tailscale.md

- Route: /gateway/tailscale
- Koppen:
  - H2: Modi
  - H2: Configuratievoorbeelden
  - H3: Alleen Tailnet (Serve)
  - H3: Alleen Tailnet (binden aan Tailnet-IP)
  - H3: Openbaar internet (Funnel + gedeeld wachtwoord)
  - H2: CLI-voorbeelden
  - H2: Authenticatie
  - H3: Tailscale-identiteitsheaders (alleen Serve)
  - H2: Opmerkingen
  - H3: Vereisten en beperkingen van Tailscale
  - H2: Browserbesturing (externe Gateway + lokale browser)
  - H2: Meer informatie
  - H2: Gerelateerd

## gateway/tools-invoke-http-api.md

- Route: /gateway/tools-invoke-http-api
- Koppen:
  - H2: Authenticatie
  - H2: Beveiligingsgrens (belangrijk)
  - H2: Aanvraagbody
  - H2: Beleid + routeringsgedrag
  - H2: Antwoorden
  - H2: Voorbeeld
  - H2: Gerelateerd

## gateway/troubleshooting.md

- Route: /gateway/troubleshooting
- Koppen:
  - H2: Volgorde van opdrachten
  - H2: Na een update
  - H2: Gesplitste installaties en beveiliging tegen nieuwere configuraties
  - H2: Protocol komt niet overeen na terugdraaien
  - H2: Symlink van Skill overgeslagen vanwege ontsnapping uit pad
  - H2: Anthropic 429: extra gebruik vereist voor lange context
  - H2: Geblokkeerde 403-antwoorden van upstream
  - H2: Lokale OpenAI-compatibele backend doorstaat directe tests, maar agentuitvoeringen mislukken
  - H2: Geen antwoorden
  - H2: Verbinding van de besturingsinterface van het dashboard
  - H3: Sneloverzicht van detailcodes voor authenticatie
  - H2: Gateway-service wordt niet uitgevoerd
  - H2: macOS-Gateway reageert stilzwijgend niet meer en hervat wanneer je het dashboard aanraakt
  - H2: macOS-launchd-supervisorlus met dubbele LaunchAgents voor Gateway/Node
  - H2: Gateway wordt afgesloten bij hoog geheugengebruik
  - H2: Gateway heeft ongeldige configuratie geweigerd
  - H2: Waarschuwingen van Gateway-controles
  - H2: Kanaal verbonden, berichten worden niet doorgegeven
  - H2: Levering via Cron en Heartbeat
  - H2: Node gekoppeld, hulpmiddel mislukt
  - H2: Browserhulpmiddel mislukt
  - H2: Als je een upgrade hebt uitgevoerd en er plotseling iets niet meer werkt
  - H2: Gerelateerd

## gateway/trusted-proxy-auth.md

- Route: /gateway/trusted-proxy-auth
- Koppen:
  - H2: Wanneer te gebruiken
  - H2: Wanneer NIET te gebruiken
  - H2: Hoe het werkt
  - H2: Configuratie
  - H3: Configuratiereferentie
  - H2: Automatische goedkeuring van apparaten
  - H2: Koppelingsgedrag van de besturingsinterface
  - H2: Header voor operatorbereiken
  - H2: TLS-beëindiging en HSTS
  - H3: Richtlijnen voor uitrol
  - H2: Voorbeelden van proxyconfiguratie
  - H2: Configuratie met gemengde tokens
  - H2: Beveiligingschecklist
  - H2: Beveiligingsaudit
  - H2: Probleemoplossing
  - H2: Migratie vanuit tokenauthenticatie
  - H2: Gerelateerd

## help/debugging.md

- Route: /help/debugging
- Koppen:
  - H2: Overschrijvingen voor runtimefoutopsporing
  - H2: Uitvoer van sessietraces
  - H2: Levenscyclustrace van Plugins
  - H2: Opstart- en opdrachtprofilering van de CLI
  - H2: Watchmodus van de Gateway
  - H2: Ontwikkelprofiel + ontwikkel-Gateway (--dev)
  - H2: Onbewerkte streamlogboeken
  - H2: Veiligheidsopmerkingen
  - H2: Fouten opsporen in VSCode
  - H3: Installatie
  - H3: Opmerkingen
  - H2: Gerelateerd

## help/environment.md

- Route: /help/environment
- Koppen:
  - H2: Voorrang (hoogste naar laagste)
  - H2: Ondersteunde variabelen voor operators
  - H3: Paden en instanties
  - H3: Gateway en authenticatie
  - H3: Providerreferenties
  - H3: Logboekregistratie en diagnostiek
  - H3: Schakelaars voor functies en runtime
  - H2: Providerreferenties en .env in de werkruimte
  - H2: Omgevingsblok in configuratie
  - H2: Shellomgeving importeren
  - H2: Momentopnamen van uitvoeringsshells
  - H2: Tijdens runtime geïnjecteerde omgevingsvariabelen
  - H2: Omgevingsvariabelen voor de gebruikersinterface
  - H2: Vervanging van omgevingsvariabelen in configuratie
  - H2: Verwijzingen naar geheimen versus ${ENV}-tekenreeksen
  - H2: Padgerelateerde omgevingsvariabelen
  - H2: Downloads van helperhulpmiddelen voor agents
  - H2: Logboekregistratie
  - H3: `OPENCLAW_HOME`
  - H2: nvm-gebruikers: TLS-fouten van webfetch
  - H2: Verouderde omgevingsvariabelen
  - H2: Gerelateerd

## help/faq-first-run.md

- Route: /help/faq-first-run
- Koppen:
  - H2: Snel aan de slag en configuratie voor de eerste uitvoering
  - H2: Gerelateerd

## help/faq-models.md

- Route: /help/faq-models
- Koppen:
  - H2: Modellen: standaardinstellingen, selectie, aliassen, wisselen
  - H2: Model-failover en "Alle modellen zijn mislukt"
  - H2: Authenticatieprofielen: wat ze zijn en hoe je ze beheert
  - H2: Gerelateerd

## help/faq.md

- Route: /help/faq
- Koppen:
  - H2: De eerste 60 seconden als er iets niet werkt
  - H2: Snel aan de slag en configuratie voor de eerste uitvoering
  - H2: Wat is OpenClaw?
  - H2: Skills en automatisering
  - H2: Sandboxing en geheugen
  - H2: Waar onderdelen op schijf staan
  - H2: Basisprincipes van configuratie
  - H2: Externe Gateways en Nodes
  - H2: Omgevingsvariabelen en het laden van .env
  - H2: Sessies en meerdere chats
  - H2: Modellen, failover en authenticatieprofielen
  - H2: Gateway: poorten, "wordt al uitgevoerd" en externe modus
  - H2: Logboekregistratie en foutopsporing
  - H2: Media en bijlagen
  - H2: Beveiliging en toegangsbeheer
  - H2: Chatopdrachten, taken afbreken en "het stopt niet"
  - H2: Diversen
  - H2: Gerelateerd

## help/index.md

- Route: /help
- Koppen:
  - H2: Veelgestelde vragen
  - H2: Diagnostiek
  - H2: Testen
  - H2: Community en meta

## help/scripts.md

- Route: /help/scripts
- Koppen:
  - H2: Conventies
  - H2: Scripts voor authenticatiebewaking
  - H2: GitHub-leeshulp
  - H2: Bij het toevoegen van scripts
  - H2: Gerelateerd

## help/testing-live.md

- Route: /help/testing-live
- Koppen:
  - H2: Livetests met je echte Gateway
  - H2: Live: lokale smoke-opdrachten
  - H2: Live: controle van Android Node-mogelijkheden
  - H2: Live: model-smoketest (profielsleutels)
  - H3: Laag 1: directe modelaanvulling (zonder Gateway)
  - H3: Laag 2: Gateway + smoketest met ontwikkelagent (wat "@openclaw" werkelijk doet)
  - H2: Live: smoketest voor CLI-backend (Claude, Gemini of andere lokale CLI's)
  - H2: Live: bereikbaarheid van APNs HTTP/2-proxy
  - H2: Live: ACP-bindingsmoketest (/acp spawn ... --bind here)
  - H2: Live: smoketest voor Codex-app-serverharnas
  - H2: Live: herhaalde OpenAI-Compaction
  - H3: Aanbevolen liverecepten
  - H2: Live: modelmatrix (wat we testen)
  - H3: Aggregators / alternatieve gateways
  - H2: Aanmeldgegevens (nooit committen)
  - H2: Deepgram live (audiotranscriptie)
  - H2: BytePlus-coderingsplan live
  - H2: ComfyUI-workflowmedia live
  - H2: Afbeeldingsgeneratie live
  - H2: Muziekgeneratie live
  - H2: Videogeneratie live
  - H2: Harnas voor mediatests live
  - H2: Gerelateerd

## help/testing-updates-plugins.md

- Route: /help/testing-updates-plugins
- Koppen:
  - H2: Wat we beschermen
  - H2: Lokaal bewijs tijdens ontwikkeling
  - H2: Docker-testtrajecten
  - H2: Pakketacceptatie
  - H2: Standaard voor releases
  - H2: Compatibiliteit met oudere versies
  - H2: Testdekking toevoegen
  - H2: Fouten analyseren

## help/testing.md

- Route: /help/testing
- Koppen:
  - H2: Snel aan de slag
  - H2: Tijdelijke testmappen
  - H2: Live- en Docker-/Parallels-workflows
  - H2: QA-specifieke uitvoerprogramma's
  - H3: Gedeelde Telegram-aanmeldgegevens via Convex (v1)
  - H3: Een kanaal aan QA toevoegen
  - H2: Testsuites (wat waar wordt uitgevoerd)
  - H3: Unit-/integratietests (standaard)
  - H3: Stabiliteit (Gateway)
  - H3: E2E (geaggregeerd voor de repository)
  - H3: E2E (Gateway-smoketest)
  - H3: E2E (Control UI met gesimuleerde browser)
  - H3: E2E: smoketest voor OpenShell-backend
  - H3: Live (echte providers + echte modellen)
  - H2: Welke suite moet ik uitvoeren?
  - H2: Live-tests (met netwerktoegang)
  - H2: Docker-uitvoerprogramma's (optionele controles voor "werkt in Linux")
  - H2: Documentatiecontrole
  - H2: Offlineregressie (veilig voor CI)
  - H2: Evaluaties van agentbetrouwbaarheid (Skills)
  - H2: Contracttests (vorm van Plugin en kanaal)
  - H3: Opdrachten
  - H3: Kanaalcontracten
  - H3: Providercontracten
  - H3: Wanneer uitvoeren
  - H2: Regressietests toevoegen (richtlijnen)
  - H2: Gerelateerd

## help/troubleshooting.md

- Route: /help/troubleshooting
- Koppen:
  - H2: De eerste 60 seconden
  - H2: Assistent voelt beperkt aan of mist hulpmiddelen
  - H2: Anthropic-fout 429 bij lange context
  - H2: Lokale OpenAI-compatibele backend werkt rechtstreeks, maar niet in OpenClaw
  - H2: Installatie van Plugin mislukt door ontbrekende OpenClaw-extensies
  - H2: Installatiebeleid blokkeert installaties of updates van Plugins
  - H2: Plugin is aanwezig, maar geblokkeerd wegens verdacht eigendom
  - H2: Beslisboom
  - H2: Gerelateerd

## index.md

- Route: /
- Koppen:
  - H1: OpenClaw 🦞
  - H2: Door de documentatie bladeren
  - H2: Wat is OpenClaw?
  - H2: Hoe het werkt
  - H2: Belangrijkste mogelijkheden
  - H2: Snel aan de slag
  - H2: Dashboard
  - H2: Configuratie (optioneel)
  - H2: Begin hier
  - H2: Meer informatie

## install/ansible.md

- Route: /install/ansible
- Koppen:
  - H2: Vereisten
  - H2: Wat je krijgt
  - H2: Snel aan de slag
  - H2: Wat er wordt geïnstalleerd
  - H2: Configuratie na installatie
  - H3: Snelle opdrachten
  - H2: Beveiligingsarchitectuur
  - H2: Handmatige installatie
  - H2: Bijwerken
  - H2: Problemen oplossen
  - H2: Geavanceerde configuratie
  - H2: Gerelateerd

## install/azure.md

- Route: /install/azure
- Koppen:
  - H2: Wat je gaat doen
  - H2: Wat je nodig hebt
  - H2: Implementatie configureren
  - H2: Azure-resources implementeren
  - H2: OpenClaw installeren
  - H2: Kostenoverwegingen
  - H2: Opschonen
  - H2: Volgende stappen
  - H2: Gerelateerd

## install/bun.md

- Route: /install/bun
- Koppen:
  - H2: Installeren
  - H2: Levenscyclusscripts
  - H2: Aandachtspunten
  - H2: Gerelateerd

## install/clawdock.md

- Route: /install/clawdock
- Koppen:
  - H2: Installeren
  - H2: Wat je krijgt
  - H3: Basisbewerkingen
  - H3: Toegang tot containers
  - H3: Webinterface en koppeling
  - H3: Configuratie en onderhoud
  - H3: Hulpprogramma's
  - H2: Eerste configuratie
  - H2: Configuratie en geheimen
  - H2: Gerelateerd

## install/development-channels.md

- Route: /install/development-channels
- Koppen:
  - H2: Van kanaal wisselen
  - H2: Eenmalig een versie of tag selecteren
  - H2: Proefuitvoering
  - H2: Plugins en kanalen
  - H2: Huidige status controleren
  - H2: Aanbevolen werkwijzen voor tags
  - H2: Beschikbaarheid van de macOS-app
  - H2: Gerelateerd

## install/digitalocean.md

- Route: /install/digitalocean
- Koppen:
  - H2: Vereisten
  - H2: Configuratie
  - H2: Persistentie en back-ups
  - H2: Tips voor 1 GB RAM
  - H2: Problemen oplossen
  - H2: Volgende stappen
  - H2: Gerelateerd

## install/docker-vm-runtime.md

- Route: /install/docker-vm-runtime
- Koppen:
  - H2: Vereiste binaire bestanden in de image opnemen
  - H2: Bouwen en starten
  - H2: Wat waar behouden blijft
  - H2: Updates
  - H2: Gerelateerd

## install/docker.md

- Route: /install/docker
- Koppen:
  - H2: Vereisten
  - H2: Gateway in een container
  - H3: Handmatige procedure
  - H3: Containerimages upgraden
  - H3: Omgevingsvariabelen
  - H3: Vanuit broncode gebouwde images met geselecteerde Plugins
  - H3: Observeerbaarheid
  - H3: Statuscontroles
  - H3: LAN versus loopback
  - H3: Lokale providers op de host
  - H3: Claude CLI-backend in Docker
  - H3: Bonjour / mDNS
  - H3: Opslag en persistentie
  - H3: Shell-hulpprogramma's (optioneel)
  - H3: Uitvoeren op een VPS?
  - H2: Agent-sandbox
  - H3: Snel inschakelen
  - H2: Problemen oplossen
  - H2: Gerelateerd

## install/exe-dev.md

- Route: /install/exe-dev
- Koppen:
  - H2: Wat je nodig hebt
  - H2: Snelle route voor beginners
  - H2: Geautomatiseerde installatie met Shelley
  - H2: Handmatige installatie
  - H2: Extern kanaal configureren
  - H2: Externe toegang
  - H2: Bijwerken
  - H2: Gerelateerd

## install/fly.md

- Route: /install/fly
- Koppen:
  - H2: Wat je nodig hebt
  - H2: Snelle route voor beginners
  - H2: Problemen oplossen
  - H3: "App luistert niet op het verwachte adres"
  - H3: Statuscontroles mislukken / verbinding geweigerd
  - H3: OOM-/geheugenproblemen
  - H3: Problemen met Gateway-vergrendeling
  - H3: Configuratie wordt niet gelezen
  - H3: Configuratie schrijven via SSH
  - H3: Status blijft niet behouden
  - H2: Bijwerken
  - H3: De machineopdracht bijwerken
  - H2: Privé-implementatie (versterkt)
  - H3: Wanneer je privé-implementatie gebruikt
  - H3: Configuratie
  - H3: Toegang tot een privé-implementatie
  - H3: Webhooks met een privé-implementatie
  - H3: Beveiligingsafwegingen
  - H2: Opmerkingen
  - H2: Kosten
  - H2: Volgende stappen
  - H2: Gerelateerd

## install/gcp.md

- Route: /install/gcp
- Koppen:
  - H2: Wat je nodig hebt
  - H2: Snelle route
  - H2: Problemen oplossen
  - H2: Serviceaccounts (aanbevolen beveiligingspraktijk)
  - H2: Volgende stappen
  - H2: Gerelateerd

## install/hetzner.md

- Route: /install/hetzner
- Koppen:
  - H2: Wat je nodig hebt
  - H2: Snelle route
  - H2: Infrastructure as Code (Terraform)
  - H2: Volgende stappen
  - H2: Gerelateerd

## install/hostinger.md

- Route: /install/hostinger
- Koppen:
  - H2: Vereisten
  - H2: Optie A: OpenClaw met 1 klik
  - H2: Optie B: OpenClaw op een VPS
  - H2: Je configuratie verifiëren
  - H2: Problemen oplossen
  - H2: Volgende stappen
  - H2: Gerelateerd

## install/index.md

- Route: /install
- Koppen:
  - H2: Systeemvereisten
  - H2: Aanbevolen: installatiescript
  - H2: Alternatieve installatiemethoden
  - H3: Installatieprogramma met lokaal voorvoegsel (install-cli.sh)
  - H3: npm, pnpm of bun
  - H3: Vanuit broncode
  - H3: Installeren vanuit de GitHub-main-checkout
  - H3: Containers en pakketbeheerders
  - H2: De installatie verifiëren
  - H2: Hosting en implementatie
  - H2: Bijwerken, migreren of verwijderen
  - H2: Problemen oplossen: openclaw niet gevonden

## install/installer.md

- Route: /install/installer
- Koppen:
  - H2: Snelle opdrachten
  - H2: install.sh
  - H3: Procedure (install.sh)
  - H3: Detectie van broncodecheckout
  - H3: Voorbeelden (install.sh)
  - H2: install-cli.sh
  - H3: Procedure (install-cli.sh)
  - H3: Voorbeelden (install-cli.sh)
  - H2: install.ps1
  - H3: Procedure (install.ps1)
  - H3: Voorbeelden (install.ps1)
  - H2: CI en automatisering
  - H2: Problemen oplossen
  - H2: Gerelateerd

## install/kubernetes.md

- Route: /install/kubernetes
- Koppen:
  - H2: Waarom geen Helm
  - H2: Wat je nodig hebt
  - H2: Snel aan de slag
  - H2: Lokaal testen met Kind
  - H2: Stap voor stap
  - H3: 1) Implementeren
  - H3: 2) Toegang tot de Gateway
  - H2: Wat er wordt geïmplementeerd
  - H2: Aanpassing
  - H3: Agentinstructies
  - H3: Gateway-configuratie
  - H3: Providers toevoegen
  - H3: Aangepaste namespace
  - H3: Aangepaste image
  - H3: Beschikbaar maken buiten port-forward
  - H2: Opnieuw implementeren
  - H2: Verwijderen
  - H2: Architectuurnotities
  - H2: Bestandsstructuur
  - H2: Gerelateerd

## install/macos-vm.md

- Route: /install/macos-vm
- Koppen:
  - H2: Aanbevolen standaardoptie (de meeste gebruikers)
  - H2: Opties voor macOS-VM's
  - H3: Lokale VM op je Apple Silicon-Mac (Lume)
  - H3: Gehoste Mac-providers (cloud)
  - H2: Snelle route (Lume, ervaren gebruikers)
  - H2: Wat je nodig hebt (Lume)
  - H2: 1) Lume installeren
  - H2: 2) De macOS-VM maken
  - H2: 3) De configuratie-assistent voltooien
  - H2: 4) Het IP-adres van de VM ophalen
  - H2: 5) Via SSH verbinding maken met de VM
  - H2: 6) OpenClaw installeren
  - H2: 7) Kanalen configureren
  - H2: 8) De VM zonder grafische interface uitvoeren
  - H2: Bonus: iMessage-integratie
  - H2: Een golden image opslaan
  - H2: 24/7 uitvoeren
  - H2: Problemen oplossen
  - H2: Gerelateerde documentatie

## install/migrating-claude.md

- Route: /install/migrating-claude
- Koppen:
  - H2: Twee manieren om te importeren
  - H2: Wat er wordt geïmporteerd
  - H2: Wat alleen in het archief blijft
  - H2: Bronselectie
  - H2: Aanbevolen procedure
  - H2: Conflictafhandeling
  - H2: JSON-uitvoer voor automatisering
  - H2: Problemen oplossen
  - H2: Gerelateerd

## install/migrating-hermes.md

- Route: /install/migrating-hermes
- Koppen:
  - H2: Twee manieren om te importeren
  - H2: Wat er wordt geïmporteerd
  - H2: Wat alleen in het archief blijft
  - H2: Aanbevolen procedure
  - H2: Conflictafhandeling
  - H2: Geheimen
  - H2: JSON-uitvoer voor automatisering
  - H2: Problemen oplossen
  - H2: Gerelateerd

## install/migrating.md

- Route: /install/migrating
- Koppen:
  - H2: Importeren vanuit een ander agentsysteem
  - H2: OpenClaw naar een nieuwe machine verplaatsen
  - H3: Migratiestappen
  - H3: Veelvoorkomende valkuilen
  - H3: Verificatiechecklist
  - H2: Een Plugin ter plaatse upgraden
  - H2: Gerelateerd

## install/nix.md

- Route: /install/nix
- Koppen:
  - H2: Wat je krijgt
  - H2: Snel aan de slag
  - H2: Runtimegedrag in Nix-modus
  - H3: Wat er verandert in Nix-modus
  - H3: Configuratie- en statuspaden
  - H3: PATH-detectie voor de service
  - H2: Gerelateerd

## install/node.md

- Route: /install/node
- Koppen:
  - H2: Je versie controleren
  - H2: Node installeren
  - H2: Problemen oplossen
  - H3: openclaw: opdracht niet gevonden
  - H3: Toestemmingsfouten bij npm install -g (Linux)
  - H2: Gerelateerd

## install/northflank.mdx

- Route: /install/northflank
- Koppen:
  - H2: Aan de slag
  - H2: Wat je krijgt
  - H2: Een kanaal verbinden
  - H2: Volgende stappen

## install/oracle.md

- Route: /install/oracle
- Koppen:
  - H2: Vereisten
  - H2: Configuratie
  - H2: De beveiligingsstatus verifiëren
  - H2: ARM-notities
  - H2: Persistentie en back-ups
  - H2: Terugvaloptie: SSH-tunnel
  - H2: Problemen oplossen
  - H2: Volgende stappen
  - H2: Gerelateerd

## install/podman.md

- Route: /install/podman
- Koppen:
  - H2: Vereisten
  - H2: Snel aan de slag
  - H2: Podman en Tailscale
  - H2: Systemd (Quadlet, optioneel)
  - H2: Configuratie, omgevingsvariabelen en opslag
  - H2: Images upgraden
  - H2: Nuttige opdrachten
  - H2: Problemen oplossen
  - H2: Gerelateerd

## install/railway.mdx

- Route: /install/railway
- Koppen:
  - H2: Implementatie met één klik
  - H2: Wat je krijgt
  - H2: Een kanaal verbinden
  - H2: Back-ups en migratie
  - H2: Volgende stappen

## install/raspberry-pi.md

- Route: /install/raspberry-pi
- Koppen:
  - H2: Hardwarecompatibiliteit
  - H2: Vereisten
  - H2: Configuratie
  - H2: Prestatietips
  - H2: Aanbevolen modelconfiguratie
  - H2: Notities over ARM-binaries
  - H2: Persistentie en back-ups
  - H2: Problemen oplossen
  - H2: Volgende stappen
  - H2: Gerelateerd

## install/render.mdx

- Route: /install/render
- Koppen:
  - H2: Vereisten
  - H2: Implementeren
  - H2: De Blueprint
  - H2: Een abonnement kiezen
  - H2: Na de implementatie
  - H3: Toegang tot de Control UI
  - H3: Logboeken
  - H3: Shelltoegang
  - H3: Omgevingsvariabelen
  - H3: Automatisch implementeren
  - H2: Aangepast domein
  - H2: Schalen
  - H2: Back-ups en migratie
  - H2: Problemen oplossen
  - H3: Service start niet
  - H3: Trage koude starts (gratis niveau)
  - H3: Gegevensverlies na herimplementatie
  - H3: Mislukte statuscontroles
  - H2: Volgende stappen

## install/uninstall.md

- Route: /install/uninstall
- Koppen:
  - H2: Eenvoudige methode (CLI nog geïnstalleerd)
  - H2: Service handmatig verwijderen (CLI niet geïnstalleerd)
  - H3: macOS (launchd)
  - H3: Linux (systemd-gebruikerseenheid)
  - H3: Windows (geplande taak)
  - H2: Normale installatie versus broncode-check-out
  - H3: Normale installatie (install.sh / npm / pnpm / bun)
  - H3: Broncode-check-out (git clone)
  - H2: Gerelateerd

## install/updating.md

- Route: /install/updating
- Koppen:
  - H2: Aanbevolen: openclaw update
  - H2: Wisselen tussen npm- en git-installaties
  - H2: Servers vanuit broncode-check-out (referentiescript)
  - H2: Alternatief: het installatieprogramma opnieuw uitvoeren
  - H2: Alternatief: handmatig via npm, pnpm of bun
  - H3: Geavanceerde onderwerpen voor npm-installatie
  - H2: Automatische updater
  - H2: Na het bijwerken
  - H3: Doctor uitvoeren
  - H3: De Gateway opnieuw starten
  - H3: Verifiëren
  - H2: Terugdraaien
  - H3: Vóór het bijwerken: een geverifieerde back-up maken
  - H3: Een pakketinstallatie terugdraaien
  - H3: Een broncode-check-out terugdraaien
  - H3: Downgraden over de SQLite-migratie van sessies heen
  - H3: Status alleen herstellen wanneer dat nodig is
  - H3: Het terugdraaien verifiëren
  - H2: Als je vastloopt
  - H2: Gerelateerd

## install/upstash.md

- Route: /install/upstash
- Koppen:
  - H2: Vereisten
  - H2: Een Box maken
  - H2: Verbinding maken via een SSH-tunnel
  - H2: OpenClaw installeren
  - H2: Onboarding uitvoeren
  - H2: De Gateway starten
  - H2: Automatisch opnieuw starten
  - H2: Problemen oplossen
  - H2: Gerelateerd

## logging.md

- Route: /logging
- Koppen:
  - H2: Waar logboeken worden opgeslagen
  - H2: Logboeken lezen
  - H3: CLI: live volgen (aanbevolen)
  - H3: Control UI (web)
  - H3: Logboeken voor alleen het kanaal
  - H2: Logindelingen
  - H3: Bestandslogboeken (JSONL)
  - H3: Console-uitvoer
  - H3: Gateway-WebSocket-logboeken
  - H2: Logboekregistratie configureren
  - H3: Logniveaus
  - H3: Gerichte diagnostiek voor modeltransport
  - H3: Tracecorrelatie
  - H3: Grootte en timing van modelaanroepen
  - H3: Consolestijlen
  - H3: Redactie
  - H2: Diagnostiek en OpenTelemetry
  - H2: Tips voor probleemoplossing
  - H2: Gerelateerd

## maturity/scorecard.md

- Route: /maturity/scorecard
- Koppen:
  - H1: Volwassenheidsscorekaart
  - H2: Waarvoor deze pagina dient
  - H2: In één oogopslag
  - H2: Scorecategorieën
  - H2: Oppervlakteverkenner
  - H2: Samenvatting van QA-bewijs
  - H3: Gereedheid per gebied

## maturity/taxonomy.md

- Route: /maturity/taxonomy
- Koppen:
  - H1: Volwassenheidstaxonomie
  - H2: Deze pagina lezen
  - H2: Volwassenheidsniveaus
  - H2: Productgebieden
  - H2: Details
  - H3: Kern
  - H3: Platform
  - H3: Kanaal
  - H3: Provider en tool

## network.md

- Route: /network
- Koppen:
  - H2: Kernmodel
  - H2: Koppeling + identiteit
  - H2: Detectie + transporten
  - H2: Nodes + transporten
  - H2: Beveiliging
  - H2: Gerelateerd

## nodes/audio.md

- Route: /nodes/audio
- Koppen:
  - H2: Wat het doet
  - H2: Automatische detectie (standaard)
  - H2: Configuratievoorbeelden
  - H3: Provider + CLI-terugvaloptie (OpenAI + Whisper CLI)
  - H3: Alleen provider (Deepgram)
  - H3: Alleen provider (Mistral Voxtral)
  - H3: Alleen provider (SenseAudio)
  - H3: Transcript naar chat terugsturen (opt-in)
  - H2: Notities en limieten
  - H3: Permanente lokale spraak-naar-tekst
  - H3: Ondersteuning voor proxyomgevingen
  - H2: Vermeldingsdetectie in groepen
  - H2: Aandachtspunten
  - H2: Gerelateerd

## nodes/camera.md

- Route: /nodes/camera
- Koppen:
  - H2: iOS-Node
  - H3: iOS-gebruikersinstelling
  - H3: iOS-opdrachten (via Gateway node.invoke)
  - H3: iOS-vereiste voor uitvoering op de voorgrond
  - H3: CLI-hulpprogramma
  - H2: Android-Node
  - H3: Android-gebruikersinstelling
  - H3: Machtigingen
  - H3: Android-vereiste voor uitvoering op de voorgrond
  - H3: Android-opdrachten (via Gateway node.invoke)
  - H2: macOS-app
  - H3: macOS-gebruikersinstelling
  - H3: CLI-hulpprogramma (Node aanroepen)
  - H2: Linux-Node-host
  - H2: Veiligheid en praktische beperkingen
  - H2: macOS-schermvideo (op OS-niveau)
  - H2: Gerelateerd

## nodes/computer-use.md

- Route: /nodes/computer-use
- Koppen:
  - H2: Vereisten
  - H2: Het computeragenthulpmiddel
  - H2: Windows en Linux (experimenteel, via cua-driver)
  - H3: Probleemoplossing
  - H2: De Node-opdracht computer.act
  - H2: Inschakelen en gereedmaken
  - H2: Veiligheid
  - H2: Relatie tot andere methoden voor bureaubladbesturing

## nodes/images.md

- Route: /nodes/images
- Koppen:
  - H2: Doelen
  - H2: CLI-oppervlak
  - H2: Gedrag van het WhatsApp-webkanaal
  - H2: Pijplijn voor automatische antwoorden
  - H2: Inkomende media naar opdrachten
  - H2: Limieten en fouten
  - H2: Opmerkingen voor tests
  - H2: Gerelateerd

## nodes/index.md

- Route: /nodes
- Koppen:
  - H2: Koppeling en status
  - H2: Versieverschillen en upgradevolgorde
  - H2: Externe Node-host (system.run)
  - H3: Een Node-host starten (voorgrond)
  - H3: Externe Gateway via SSH-tunnel (loopbackbinding)
  - H3: Een Node-host starten (service)
  - H3: Koppelen en een naam geven
  - H3: Door de Node gehoste MCP-servers
  - H3: Door de Node gehoste Skills
  - H3: Status van de headless identiteit
  - H3: De opdrachten op de toelatingslijst zetten
  - H3: exec naar de Node verwijzen
  - H3: Lokale modelinferentie
  - H3: Codex-sessies en transcripties
  - H3: Claude-sessies en transcripties
  - H3: OpenCode- en Pi-sessies
  - H3: Terminalbestandsuploads
  - H2: Opdrachten aanroepen
  - H2: Opdrachtbeleid
  - H2: Configuratie (openclaw.json)
  - H2: Schermafbeeldingen (canvasmomentopnamen)
  - H3: Canvasbesturing
  - H3: A2UI (Canvas)
  - H2: Foto's en video's (Node-camera)
  - H2: Schermopnamen (Nodes)
  - H2: Locatie (Nodes)
  - H2: Sms (Android-Nodes)
  - H2: Opdrachten voor apparaat- en persoonsgegevens
  - H2: Systeemopdrachten (Node-host/Mac-Node)
  - H2: Binding van de exec-Node
  - H2: Machtigingsoverzicht
  - H2: Headless Node-host (platformonafhankelijk)
  - H2: Mac-Node-modus

## nodes/location-command.md

- Route: /nodes/location-command
- Koppen:
  - H2: Kort samengevat
  - H2: Waarom een keuzelijst (en niet alleen een schakelaar)
  - H2: Instellingenmodel
  - H2: Machtigingstoewijzing (node.permissions)
  - H2: Opdracht: location.get
  - H2: Gedrag op de achtergrond
  - H2: Linux-Node-host
  - H2: Integratie met modellen en hulpmiddelen
  - H2: UX-tekst (voorgesteld)
  - H2: Gerelateerd

## nodes/media-understanding.md

- Route: /nodes/media-understanding
- Koppen:
  - H2: Hoe het werkt
  - H2: Configuratie
  - H3: Modelvermeldingen
  - H3: Providerreferenties
  - H2: Regels en gedrag
  - H3: Automatisch detecteren (standaard)
  - H3: Proxyondersteuning (provider-aanroepen voor audio/video)
  - H2: Mogelijkheden
  - H2: Ondersteuningsmatrix voor providers
  - H2: Richtlijnen voor modelselectie
  - H2: Bijlagebeleid
  - H3: Extractie van bestandsbijlagen
  - H2: Configuratievoorbeelden
  - H2: Statusuitvoer
  - H2: Opmerkingen
  - H2: Gerelateerd

## nodes/presence.md

- Route: /nodes/presence
- Koppen:
  - H2: Vereisten
  - H2: De actieve computer controleren
  - H2: Hoe activiteit aanwezigheid wordt
  - H2: Privacy en modelcontext
  - H2: Hoe verbindingsmeldingen worden gerouteerd
  - H2: Probleemoplossing
  - H2: Gerelateerd

## nodes/talk.md

- Route: /nodes/talk
- Koppen:
  - H2: Gedrag (macOS)
  - H2: Spraakrichtlijnen in antwoorden
  - H2: Configuratie (`~/.openclaw/openclaw.json`)
  - H2: macOS-gebruikersinterface
  - H2: Android-gebruikersinterface
  - H2: Opmerkingen
  - H2: Gerelateerd

## nodes/troubleshooting.md

- Route: /nodes/troubleshooting
- Koppen:
  - H2: Opdrachtladder
  - H2: Vereisten voor uitvoering op de voorgrond
  - H2: Machtigingenmatrix
  - H2: Koppeling versus goedkeuringen
  - H2: Veelvoorkomende Node-foutcodes
  - H2: Snelle herstelcyclus
  - H2: Gerelateerd

## nodes/voicewake.md

- Route: /nodes/voicewake
- Koppen:
  - H2: Opslag
  - H2: Protocol
  - H3: Triggerlijst
  - H3: Routering (van trigger naar doel)
  - H3: Gebeurtenissen
  - H2: Clientgedrag
  - H2: Gerelateerd

## openclaw-agent-runtime.md

- Route: /openclaw-agent-runtime
- Koppen:
  - H2: Typecontrole en linting
  - H2: Agent Runtime-tests uitvoeren
  - H2: Handmatig testen
  - H2: Volledige reset
  - H2: Verwijzingen
  - H2: Gerelateerd

## perplexity.md

- Route: /perplexity
- Koppen:
  - H2: Gerelateerd

## plan/cloud-workers.md

- Route: /plan/cloud-workers
- Koppen:
  - H2: Status
  - H2: Probleem
  - H2: Doelen
  - H2: Geen doelen (v1)
  - H2: Bestaande oplossingen (wat we overnemen, wat we omkeren)
  - H2: Architectuurbeslissing: lus op de worker, inferentie via de Gateway
  - H2: Componenten
  - H3: 1. Toestandsmachine voor de omgeving en providercontract
  - H3: 2. Worker-bootstrap: OpenClaw op de machine installeren
  - H3: 3. Transport: alles via SSH
  - H3: 4. Workerprotocol (specifiek; niet het Node-protocol)
  - H3: 5. RPC's voor de sessiebackend
  - H3: 6. Werkruimtesynchronisatie
  - H3: 7. Toestandsmachine voor plaatsing, sessies en gebruikersinterface
  - H2: Taakverdeling en overdracht
  - H2: Beveiligingsmodel
  - H2: Capaciteit
  - H2: Levenscyclus
  - H2: Configuratieoppervlak
  - H2: Mijlpalen
  - H2: Openstaande vragen

## plan/path3-sqlite-session-artifact-family.md

- Route: /plan/path3-sqlite-session-artifact-family
- Koppen:
  - H1: Pad 3: SQLite-familie van sessieartefacten
  - H2: Gezaghebbende familie
  - H2: Artefacten buiten de familie na de omschakeling
  - H2: Patchpunten
  - H2: Gerichte tests

## plan/swarms.md

- Route: /plan/swarms
- Koppen:
  - H1: Zwermen — agentuitwaaiering en orkestratie in codemodus
  - H2: 1. Wat en waarom
  - H2: 2. Beslissingen (beheerder, 2026-07-17)
  - H2: 3. Architectuuroverzicht
  - H2: 4. Configuratiepoort (v1)
  - H2: 5. Kern: aanmaak in verzamelmodus + `agents_wait` (v1)
  - H3: 5.1 Toevoegingen aan `sessions_spawn` (alleen wanneer zwermen zijn ingeschakeld)
  - H3: 5.2 Goedkeuringen worden bij fouten gesloten
  - H3: 5.3 Hulpmiddel `agents_wait` (nieuw, achter een poort)
  - H3: 5.4 Handhaving van limieten
  - H2: 6. Testcontract (v1, baan A)
  - H2: 7. QuickJS-gastoppervlak (baan B, na de kern)
  - H2: 8. Codex-harnasprojectie (latere baan)
  - H2: 9. Persistentie en bewaring
  - H2: 10. Voortgangsoppervlak ("de stippen") — latere baan
  - H2: 11. Labspagina (Control UI, onafhankelijke baan)
  - H2: 12. Plaatsing (later)
  - H2: 13. Geen doelen
  - H2: 14. Bouwfasen/opsplitsing in PR's

## plan/ui-channels.md

- Route: /plan/ui-channels
- Koppen:
  - H2: Status
  - H2: Probleem
  - H2: Doelen
  - H2: Geen doelen
  - H2: Doelmodel
  - H2: Afleveringsmetadata
  - H2: Contract voor runtimemogelijkheden
  - H2: Kanaaltoewijzing
  - H2: Refactorstappen
  - H2: Tests
  - H2: Openstaande vragen
  - H2: Gerelateerd

## platforms/android.md

- Route: /platforms/android
- Koppen:
  - H2: Ondersteuningsoverzicht
  - H2: Gelijktijdige Gateway-sessies
  - H2: Wear OS-begeleidende app
  - H2: Installeren buiten Google Play
  - H2: Android spiegelen en bedienen vanaf een externe Mac
  - H3: Voordat je begint
  - H3: ADB via TCP inschakelen
  - H3: Alleen de besturende Mac toestaan
  - H3: Verbinding maken en spiegelen starten
  - H3: Probleemoplossing
  - H2: Draaiboek voor verbindingen
  - H3: Vereisten
  - H3: 1. De Gateway starten
  - H3: 2. Detectie verifiëren (optioneel)
  - H4: Detectie tussen netwerken via unicast DNS-SD
  - H3: 3. Verbinding maken vanaf Android
  - H3: Gekoppelde Gateways beheren
  - H3: Aanwezigheidsbakens
  - H3: 4. Koppeling goedkeuren (CLI)
  - H3: 5. Verifiëren dat de Node is verbonden
  - H3: 6. Chat en geschiedenis
  - H3: 7. Canvas en camera
  - H4: Gateway Canvas Host (aanbevolen voor webinhoud)
  - H3: 8. Spraak en uitgebreid Android-opdrachtoppervlak
  - H3: 9. Werkruimtebestanden (alleen-lezen)
  - H2: Opdrachtgoedkeuringen beoordelen
  - H2: Vragen van de agent beantwoorden
  - H2: Toegangspunten voor de assistent
  - H2: Meldingen doorsturen
  - H2: Gerelateerd

## platforms/digitalocean.md

- Route: /platforms/digitalocean
- Koppen:
  - H2: Gerelateerd

## platforms/easyrunner.md

- Route: /platforms/easyrunner
- Koppen:
  - H2: Voordat je begint
  - H2: Compose-app
  - H2: OpenClaw configureren
  - H2: Verifiëren
  - H2: Updates en back-ups
  - H2: Probleemoplossing

## platforms/index.md

- Route: /platforms
- Koppen:
  - H2: Kies je besturingssysteem
  - H2: VPS en hosting
  - H2: Algemene links
  - H2: Gateway-service installeren (CLI)
  - H2: Gerelateerd

## platforms/ios-healthkit.md

- Route: /platforms/ios-healthkit
- Koppen:
  - H1: HealthKit-samenvattingen
  - H2: Vereisten
  - H2: Toegang inschakelen
  - H3: 1. De Gateway-opdracht autoriseren
  - H3: 2. Delen inschakelen op het iOS-apparaat
  - H2: De samenvatting van vandaag opvragen
  - H2: Privacygedrag
  - H2: Problemen oplossen
  - H3: Opdracht is niet gedeclareerd door de Node
  - H3: Opdracht vereist expliciete aanmelding
  - H3: `HEALTH_ACCESS_DISABLED`
  - H3: Samenvatting slaagt, maar meetwaarden ontbreken
  - H3: Oudere perioden mislukken
  - H2: Gerelateerd

## platforms/ios.md

- Route: /platforms/ios
- Koppen:
  - H2: Wat het doet
  - H2: Vereisten
  - H2: Snel aan de slag (koppelen + verbinden)
  - H2: Gezondheidssamenvattingen
  - H2: Goedkeuringen van opdrachten beoordelen
  - H2: Vragen van de agent beantwoorden
  - H2: Optionele rechtstreekse Apple Watch-Node
  - H2: Pushmeldingen via een relay voor officiële builds
  - H2: Achtergrondbakens voor activiteitsstatus
  - H2: Authenticatie- en vertrouwensflow
  - H2: Detectiepaden
  - H3: Bonjour (LAN)
  - H3: Tailnet (netwerkoverschrijdend)
  - H3: Handmatige host/poort
  - H2: Meerdere Gateways
  - H2: Canvas + A2UI
  - H2: Relatie met Computer Use
  - H3: Canvas-evaluatie/-momentopname
  - H2: Stemactivering + gespreksmodus
  - H2: Veelvoorkomende fouten
  - H2: Gerelateerde documentatie

## platforms/linux.md

- Route: /platforms/linux
- Koppen:
  - H2: Desktopassistent
  - H3: Snelchat
  - H3: Canvas
  - H2: CLI- en SSH-alternatief
  - H2: Node-mogelijkheden
  - H2: Installeren
  - H2: Gateway-service (systemd)
  - H2: Geheugendruk en beëindiging door OOM
  - H2: Gerelateerd

## platforms/mac/bundled-gateway.md

- Route: /platforms/mac/bundled-gateway
- Koppen:
  - H2: Automatische configuratie
  - H2: Handmatig herstel
  - H2: Launchd (Gateway als LaunchAgent)
  - H2: Versiecompatibiliteit
  - H2: Statusmap op macOS
  - H2: Appverbinding opsporen
  - H2: Snelle controle
  - H2: Gerelateerd

## platforms/mac/canvas.md

- Route: /platforms/mac/canvas
- Koppen:
  - H2: Waar Canvas zich bevindt
  - H2: Paneelgedrag
  - H2: API-oppervlak voor agents
  - H2: A2UI in Canvas
  - H3: A2UI-opdrachten (v0.8)
  - H2: Agentuitvoeringen starten vanuit Canvas
  - H2: Beveiligingsopmerkingen
  - H2: Gerelateerd

## platforms/mac/child-process.md

- Route: /platforms/mac/child-process
- Koppen:
  - H2: Standaardgedrag (launchd)
  - H2: Niet-ondertekende ontwikkelbuilds
  - H2: Modus voor alleen koppelen
  - H2: Externe modus
  - H2: Waarom we de voorkeur geven aan launchd
  - H2: Gerelateerd

## platforms/mac/dev-setup.md

- Route: /platforms/mac/dev-setup
- Koppen:
  - H1: macOS-ontwikkelomgeving instellen
  - H2: Vereisten
  - H2: 1. Afhankelijkheden installeren
  - H2: 2. De app bouwen en verpakken
  - H2: 3. De CLI en Gateway installeren
  - H2: Problemen oplossen
  - H3: Build mislukt: toolchain of SDK komt niet overeen
  - H3: App crasht bij het verlenen van toestemming
  - H3: Gateway blijft voor onbepaalde tijd op "Starting..."
  - H2: Gerelateerd

## platforms/mac/health.md

- Route: /platforms/mac/health
- Koppen:
  - H1: Statuscontroles op macOS
  - H2: Menubalk
  - H2: Instellingen
  - H2: Hoe de controle werkt
  - H2: Bij twijfel
  - H2: Gerelateerd

## platforms/mac/icon.md

- Route: /platforms/mac/icon
- Koppen:
  - H1: Statussen van het menubalkpictogram
  - H2: Statussen
  - H2: Oren voor stemactivering
  - H2: Vormen en afmetingen
  - H2: Opmerkingen over gedrag
  - H2: Gerelateerd

## platforms/mac/logging.md

- Route: /platforms/mac/logging
- Koppen:
  - H1: Logboekregistratie (macOS)
  - H2: Roterend diagnostisch logbestand (foutopsporingspaneel)
  - H2: Privégegevens in uniforme logboekregistratie op macOS
  - H2: Inschakelen voor OpenClaw (ai.openclaw)
  - H2: Uitschakelen na foutopsporing
  - H2: Gerelateerd

## platforms/mac/menu-bar.md

- Route: /platforms/mac/menu-bar
- Koppen:
  - H2: Wat wordt weergegeven
  - H2: Statusmodel
  - H2: IconState-enum (Swift)
  - H3: ActivityKind -&gt; badgesymbool
  - H3: Visuele toewijzing
  - H2: Contextsubmenu
  - H2: Tekst van de statusrij (menu)
  - H2: Gebeurtenisverwerking
  - H2: Foutopsporingsoverschrijving
  - H2: Testcontrolelijst
  - H2: Gerelateerd

## platforms/mac/peekaboo.md

- Route: /platforms/mac/peekaboo
- Koppen:
  - H2: Wat dit is (en niet is)
  - H2: Relatie met andere paden voor desktopbesturing
  - H2: De bridge inschakelen
  - H2: Detectievolgorde van clients
  - H2: Beveiliging en toestemmingen
  - H2: Gedrag van momentopnamen (automatisering)
  - H2: Problemen oplossen
  - H2: Gerelateerd

## platforms/mac/permissions.md

- Route: /platforms/mac/permissions
- Koppen:
  - H2: Vereisten voor stabiele toestemmingen
  - H2: Toegankelijkheidstoestemmingen voor Node- en CLI-runtimes
  - H2: Herstelcontrolelijst wanneer prompts verdwijnen
  - H2: Toestemmingen voor bestanden en mappen (Desktop/Documents/Downloads)
  - H2: Gerelateerd

## platforms/mac/remote.md

- Route: /platforms/mac/remote
- Koppen:
  - H2: Modi
  - H2: Externe transporten
  - H2: Vereisten op de externe host
  - H2: macOS-app instellen
  - H2: Webchat
  - H2: Toestemmingen
  - H2: Beveiligingsopmerkingen
  - H2: WhatsApp-aanmeldingsflow (extern)
  - H2: Problemen oplossen
  - H2: Meldingsgeluiden
  - H2: Gerelateerd

## platforms/mac/signing.md

- Route: /platforms/mac/signing
- Koppen:
  - H1: Mac-ondertekening (foutopsporingsbuilds)
  - H2: Gebruik
  - H3: Opmerking over ad-hocondertekening
  - H2: Buildmetadata voor Info
  - H2: Gerelateerd

## platforms/mac/skills.md

- Route: /platforms/mac/skills
- Koppen:
  - H2: Gegevensbron
  - H2: Installatieacties
  - H2: Omgevingsvariabelen/API-sleutels
  - H2: Externe modus
  - H2: Gerelateerd

## platforms/mac/voice-overlay.md

- Route: /platforms/mac/voice-overlay
- Koppen:
  - H1: Levenscyclus van de stemoverlay (macOS)
  - H2: Gedrag
  - H2: Implementatie
  - H2: Logboekregistratie
  - H2: Controlelijst voor foutopsporing
  - H2: Gerelateerd

## platforms/mac/voicewake.md

- Route: /platforms/mac/voicewake
- Koppen:
  - H1: Stemactivering en indrukken-om-te-spreken
  - H2: Vereisten
  - H2: Modi
  - H2: Runtimegedrag (activeringswoord)
  - H2: Levenscyclusinvarianten
  - H2: Bijzonderheden van indrukken-om-te-spreken
  - H2: Instellingen voor gebruikers
  - H2: Doorstuurgedrag
  - H2: Doorgestuurde payload
  - H2: Snelle verificatie
  - H2: Gerelateerd

## platforms/mac/webchat.md

- Route: /platforms/mac/webchat
- Koppen:
  - H2: Meerdere Gateway-vensters
  - H2: Snelchatbalk
  - H2: Starten en foutopsporing
  - H2: Hoe het is verbonden
  - H2: Beveiligingsoppervlak
  - H2: Bekende beperkingen
  - H2: Gerelateerd

## platforms/mac/xpc.md

- Route: /platforms/mac/xpc
- Koppen:
  - H1: OpenClaw macOS-IPC-architectuur
  - H2: Doelen
  - H2: Hoe het werkt
  - H3: Gateway- en Node-transport
  - H3: Node-service en app-IPC
  - H3: PeekabooBridge (UI-automatisering)
  - H2: Operationele flows
  - H2: Opmerkingen over versterking
  - H2: Gerelateerd

## platforms/macos.md

- Route: /platforms/macos
- Koppen:
  - H2: Downloaden
  - H2: Eerste uitvoering
  - H2: Updates
  - H2: Dashboardlinks openen
  - H2: Browseraanmeldingen importeren
  - H2: Een Gateway-modus kiezen
  - H2: Wat de app beheert
  - H2: Detailpagina's voor macOS
  - H2: Gerelateerd

## platforms/oracle.md

- Route: /platforms/oracle
- Koppen:
  - H2: Gerelateerd

## platforms/raspberry-pi.md

- Route: /platforms/raspberry-pi
- Koppen:
  - H2: Gerelateerd

## platforms/windows.md

- Route: /platforms/windows
- Koppen:
  - H2: Aanbevolen: Windows Hub
  - H3: Wat Windows Hub bevat
  - H3: Eerste keer starten
  - H2: Windows-Node-modus
  - H2: Lokale MCP-modus
  - H2: Systeemeigen Windows-CLI en Gateway
  - H2: WSL2-Gateway
  - H2: Gateway automatisch starten vóór aanmelding bij Windows
  - H2: WSL-services via LAN beschikbaar maken
  - H2: Problemen oplossen
  - H3: Het systeemvakpictogram verschijnt niet
  - H3: Lokale configuratie mislukt
  - H3: De app meldt dat koppeling vereist is
  - H3: Webchat kan een externe Gateway niet bereiken
  - H3: Opdrachten voor screen.snapshot, camera of audio mislukken
  - H3: Verbinding met Git of GitHub mislukt
  - H2: Gerelateerd

## plugins/adding-capabilities.md

- Route: /plugins/adding-capabilities
- Koppen:
  - H2: Wanneer je een mogelijkheid maakt
  - H2: De standaardvolgorde
  - H2: Wat waar hoort
  - H2: Koppelvlakken voor providers en harnesses
  - H2: Controlelijst voor bestanden
  - H2: Uitgewerkt voorbeeld: afbeeldingen genereren
  - H2: Embeddingproviders
  - H2: Beoordelingscontrolelijst
  - H2: Gerelateerd

## plugins/admin-http-rpc.md

- Route: /plugins/admin-http-rpc
- Koppen:
  - H2: Voordat je dit inschakelt
  - H2: Inschakelen
  - H2: De route verifiëren
  - H2: Authenticatie
  - H2: Beveiligingsmodel
  - H2: Verzoek
  - H2: Antwoord
  - H2: Toegestane methoden
  - H2: Vergelijking met WebSocket
  - H2: Problemen oplossen
  - H2: Gerelateerd

## plugins/agent-tools.md

- Route: /plugins/agent-tools
- Koppen:
  - H2: Gerelateerd

## plugins/architecture-internals.md

- Route: /plugins/architecture-internals
- Koppen:
  - H2: Laadpijplijn
  - H3: Manifest-eerst-gedrag
  - H3: Cachegrens van de Plugin
  - H2: Registermodel
  - H2: Callbacks voor gesprekskoppeling
  - H2: Runtime-hooks van providers
  - H3: Volgorde en gebruik van hooks
  - H3: Providervoorbeeld
  - H3: Ingebouwde voorbeelden
  - H2: Runtime-helpers
  - H3: api.runtime.imageGeneration
  - H2: HTTP-routes van de Gateway
  - H2: Importpaden van de Plugin-SDK
  - H2: Schema's voor berichttools
  - H2: Doelbepaling voor kanalen
  - H2: Door configuratie ondersteunde mappen
  - H2: Providercatalogi
  - H2: Alleen-lezeninspectie van kanalen
  - H2: Pakketbundels
  - H3: Catalogusmetadata van kanalen
  - H2: Plugins voor de contextengine
  - H2: Een nieuwe mogelijkheid toevoegen
  - H3: Checklist voor mogelijkheden
  - H3: Sjabloon voor mogelijkheden
  - H2: Gerelateerd

## plugins/architecture.md

- Route: /plugins/architecture
- Koppen:
  - H2: Openbaar mogelijkhedenmodel
  - H3: Standpunt over externe compatibiliteit
  - H3: Pluginvormen
  - H3: Compatibiliteitssignalen
  - H2: Architectuuroverzicht
  - H3: Momentopname van Pluginmetadata en opzoektabel
  - H3: Activeringsplanning
  - H3: Kanaalplugins en de gedeelde berichttool
  - H2: Eigendomsmodel voor mogelijkheden
  - H3: Gelaagdheid van mogelijkheden
  - H3: Voorbeeld van een bedrijfsplugin met meerdere mogelijkheden
  - H3: Voorbeeld van een mogelijkheid: videobegrip
  - H2: Contracten en handhaving
  - H3: Wat in een contract thuishoort
  - H2: Uitvoeringsmodel
  - H2: Exportgrens
  - H2: Interne werking en naslaginformatie
  - H2: Gerelateerd

## plugins/building-extensions.md

- Route: /plugins/building-extensions
- Koppen:
  - H2: Gerelateerd

## plugins/building-plugins.md

- Route: /plugins/building-plugins
- Koppen:
  - H2: Vereisten
  - H2: Kies de Pluginvorm
  - H2: Snelstart
  - H2: Tools registreren
  - H2: Importconventies
  - H2: Checklist vóór indiening
  - H2: Testen met bètaversies
  - H2: Volgende stappen
  - H2: Gerelateerd

## plugins/bundles.md

- Route: /plugins/bundles
- Koppen:
  - H2: Waarom bundels bestaan
  - H2: Een bundel installeren
  - H2: Wat OpenClaw uit bundels toewijst
  - H3: Momenteel ondersteund
  - H4: Skill-inhoud
  - H4: Hookbundels
  - H4: MCP voor ingebedde OpenClaw
  - H4: Instellingen voor ingebedde OpenClaw
  - H4: LSP voor ingebedde OpenClaw
  - H3: Gedetecteerd, maar niet uitgevoerd
  - H2: Bundelindelingen
  - H2: Detectieprioriteit
  - H2: Runtime-afhankelijkheden en opschoning
  - H2: Beveiliging
  - H2: Probleemoplossing
  - H2: Gerelateerd

## plugins/cli-backend-plugins.md

- Route: /plugins/cli-backend-plugins
- Koppen:
  - H2: Waarvoor de Plugin verantwoordelijk is
  - H2: Minimale backendplugin
  - H2: Configuratiestructuur
  - H2: Geavanceerde backend-hooks
  - H3: ownsNativeCompaction: OpenClaw Compaction uitschakelen
  - H2: MCP-toolbrug
  - H2: De backend selecteren
  - H2: Verificatie
  - H2: Checklist
  - H2: Gerelateerd

## plugins/codex-computer-use.md

- Route: /plugins/codex-computer-use
- Koppen:
  - H2: OpenClaw.app en Peekaboo
  - H2: iOS-app
  - H2: Rechtstreekse cua-driver-MCP
  - H2: Snelle configuratie
  - H2: Opdrachten
  - H2: Marketplace-keuzes
  - H2: Meegeleverde macOS-marketplace
  - H3: Gedeelde Plugincache
  - H2: Limiet van de externe catalogus
  - H2: Configuratienaslag
  - H2: Wat OpenClaw controleert
  - H2: macOS-machtigingen
  - H2: Probleemoplossing
  - H2: Gerelateerd

## plugins/codex-harness-reference.md

- Route: /plugins/codex-harness-reference
- Koppen:
  - H2: Configuratieoppervlak van de Plugin
  - H2: Supervisie
  - H2: App-servertransport
  - H2: Goedkeurings- en sandboxmodi
  - H2: Native uitvoering in een sandbox
  - H2: Isolatie van authenticatie en omgeving
  - H2: Dynamische tools
  - H2: Time-outs
  - H2: Modeldetectie
  - H2: Bootstrapbestanden voor de werkruimte
  - H2: Omgevingsoverschrijvingen
  - H2: Gerelateerd

## plugins/codex-harness-runtime.md

- Route: /plugins/codex-harness-runtime
- Koppen:
  - H2: Overzicht
  - H2: Threadkoppelingen en modelwijzigingen
  - H2: Supervisie en veilige voortzetting
  - H2: Zichtbare antwoorden en Heartbeats
  - H2: Hookgrenzen
  - H2: V1-ondersteuningscontract
  - H2: Native machtigingen en MCP-verzoeken
  - H2: Wachtrijsturing
  - H2: Codex-feedback uploaden
  - H2: Compaction en transcriptspiegel
  - H2: Media en aflevering
  - H2: Gerelateerd

## plugins/codex-harness.md

- Route: /plugins/codex-harness
- Koppen:
  - H2: Vereisten
  - H2: Snelstart
  - H2: Threads delen met Codex Desktop en de CLI
  - H2: Codex-sessies superviseren
  - H2: Configuratie
  - H3: Compaction
  - H3: Lange context via rechtstreekse API
  - H2: Codex-runtime verifiëren
  - H2: Routering en modelselectie
  - H2: Implementatiepatronen
  - H3: Basisimplementatie van Codex
  - H3: Implementatie met gemengde providers
  - H3: Fail-closed-implementatie van Codex
  - H2: App-serverbeleid
  - H2: Opdrachten en diagnostiek
  - H3: Codex-threads lokaal inspecteren
  - H3: Authenticatievolgorde
  - H3: Omgevingsisolatie
  - H3: Dynamische tools en zoeken op het web
  - H3: Configuratievelden
  - H3: Time-outs voor dynamische toolaanroepen
  - H3: Omgevingsoverschrijvingen voor lokale tests
  - H2: Native Codex-plugins
  - H2: Computergebruik
  - H2: Runtime-grenzen
  - H2: Probleemoplossing
  - H2: Gerelateerd

## plugins/codex-native-plugins.md

- Route: /plugins/codex-native-plugins
- Koppen:
  - H2: Vereisten
  - H2: Snelstart
  - H2: Plugins beheren vanuit de chat
  - H2: Hoe de configuratie van native Plugins werkt
  - H2: V1-ondersteuningsgrens
  - H2: App-inventaris en eigendom
  - H2: Apps voor verbonden accounts
  - H2: App-configuratie van threads
  - H2: Beleid voor destructieve acties
  - H2: Probleemoplossing
  - H2: Gerelateerd

## plugins/codex-supervision.md

- Route: /plugins/codex-supervision
- Koppen:
  - H2: Voordat je begint
  - H2: Supervisie inschakelen
  - H2: De operator-CLI gebruiken
  - H2: Een vertakking maken vanuit een lokale sessie
  - H2: Een lokale sessie archiveren
  - H2: Limieten van gekoppelde Nodes begrijpen
  - H2: Metadata en machtigingen
  - H3: Compatibiliteitstools
  - H2: Probleemoplossing
  - H2: Gerelateerd

## plugins/community.md

- Route: /plugins/community
- Koppen:
  - H2: Plugins zoeken
  - H2: Plugins publiceren
  - H2: Gerelateerd

## plugins/compatibility.md

- Route: /plugins/compatibility
- Koppen:
  - H2: Compatibiliteitsregister
  - H2: Afschrijvingsbeleid
  - H2: Huidige compatibiliteitsgebieden
  - H3: Platte aliassen voor inkomende WhatsApp-callbacks
  - H3: Toelatingsvelden voor inkomende WhatsApp-berichten
  - H2: Pakket voor Plugininspectie
  - H3: Acceptatietraject voor beheerders
  - H2: Versieopmerkingen

## plugins/copilot.md

- Route: /plugins/copilot
- Koppen:
  - H2: Vereisten
  - H2: Installeren
  - H2: Snelstart
  - H2: Ondersteunde providers
  - H2: BYOK
  - H2: Authenticatie
  - H2: Configuratieoppervlak
  - H2: Compaction
  - H2: Transcriptspiegeling
  - H2: Nevenvragen (/btw)
  - H2: Doctor
  - H2: Beperkingen
  - H2: Machtigingen en askuser
  - H3: GitHub-token op sessieniveau
  - H2: Gerelateerd

## plugins/dependency-resolution.md

- Route: /plugins/dependency-resolution
- Koppen:
  - H2: Verdeling van verantwoordelijkheden
  - H2: Installatiehoofdmappen
  - H2: Lokale Plugins
  - H2: Opstarten en opnieuw laden
  - H2: Meegeleverde Plugins
  - H2: Opschoning van verouderde onderdelen

## plugins/google-meet.md

- Route: /plugins/google-meet
- Koppen:
  - H2: Snelstart
  - H3: Een vergadering maken
  - H3: Alleen observerend deelnemen
  - H3: Realtime sessiestatus
  - H2: Lokale Gateway + Parallels Chrome
  - H3: Controles voor veelvoorkomende fouten
  - H2: Installatieopmerkingen
  - H2: Transporten
  - H3: Chrome
  - H3: Twilio
  - H2: OAuth en voorafgaande controle
  - H3: Google-referenties maken
  - H3: Het vernieuwingstoken genereren
  - H3: OAuth verifiëren met Doctor
  - H3: Artefacten bepalen, vooraf controleren en lezen
  - H3: Live-rooktest
  - H3: Voorbeelden maken
  - H2: Configuratie
  - H3: Standaardwaarden
  - H3: Optionele overschrijvingen
  - H2: Tool
  - H2: Agent- en bidi-modi
  - H2: Checklist voor live-tests
  - H2: Probleemoplossing
  - H3: Agent kan de Google Meet-tool niet zien
  - H3: Geen verbonden Node met Google Meet-mogelijkheden
  - H3: Browser wordt geopend, maar Agent kan niet deelnemen
  - H3: Maken van vergadering mislukt
  - H3: Agent neemt deel, maar praat niet
  - H3: Controles van de Twilio-configuratie mislukken
  - H3: Twilio-gesprek start, maar komt nooit in de vergadering
  - H2: Opmerkingen
  - H2: Gerelateerd

## plugins/hooks.md

- Route: /plugins/hooks
- Koppen:
  - H2: Snel aan de slag
  - H2: Hookcatalogus
  - H3: Koppelingsverzoeken voor kanalen
  - H2: Runtime-hooks debuggen
  - H2: Beleid voor toolaanroepen
  - H3: Afzenderbewust beleid in één bestand
  - H3: Hook voor de uitvoeringsomgeving
  - H3: Persistentie van toolresultaten
  - H2: Hooks voor prompts en modellen
  - H3: Sessie-uitbreidingen en injecties voor de volgende beurt
  - H2: Berichthooks
  - H2: Installatiehooks
  - H2: Levenscyclus van de Gateway
  - H3: Veilige externe Cron-projectie
  - H2: Aankomende afschaffingen
  - H2: Gerelateerd

## plugins/install-overrides.md

- Route: /plugins/install-overrides
- Koppen:
  - H2: Omgeving
  - H2: Gedrag
  - H2: E2E voor pakketten

## plugins/llama-cpp.md

- Route: /plugins/llama-cpp
- Koppen:
  - H2: Lokale tekstinferentie
  - H3: Een ander GGUF-model gebruiken
  - H2: Configuratie van geheugenembeddings
  - H2: Native runtime
  - H2: Diagnostiek van de geheugenruntime
  - H2: Probleemoplossing

## plugins/logbook.md

- Route: /plugins/logbook
- Koppen:
  - H2: Voordat je begint
  - H2: Snel aan de slag
  - H2: Hoe het werkt
  - H2: Model- en gegevensstroom
  - H2: Configuratie
  - H3: Selectie van het visiemodel
  - H2: Dashboardtabblad
  - H2: Gateway-methoden
  - H2: Opmerkingen over privacy
  - H2: Probleemoplossing
  - H3: Het tabblad Logboek ontbreekt
  - H3: Vastleggen meldt een fout
  - H3: Vastleggen lukt, maar er verschijnen geen kaarten
  - H2: Gerelateerd

## plugins/manage-plugins.md

- Route: /plugins/manage-plugins
- Koppen:
  - H2: De besturingsinterface gebruiken
  - H2: Plugins weergeven en zoeken
  - H2: Plugins in- en uitschakelen
  - H2: Plugins installeren
  - H2: Opnieuw starten en inspecteren
  - H2: Plugins bijwerken
  - H2: Plugins verwijderen
  - H2: Een bron kiezen
  - H2: Plugins publiceren
  - H2: Gerelateerd

## plugins/manifest.md

- Route: /plugins/manifest
- Koppen:
  - H2: Wat dit bestand doet
  - H2: Minimaal voorbeeld
  - H2: Uitgebreid voorbeeld
  - H2: Referentie voor velden op het hoogste niveau
  - H2: Referentie voor MCP-servers
  - H2: Referentie voor dashboard
  - H2: Referentie voor catalogus
  - H2: Referentie voor metadata van generatieproviders
  - H2: Referentie voor toolmetadata
  - H2: Referentie voor providerAuthChoices
  - H2: Referentie voor commandAliases
  - H2: Referentie voor activation
  - H2: Referentie voor qaRunners
  - H2: Referentie voor setup
  - H3: Referentie voor setup.providers
  - H3: Velden van setup
  - H2: Referentie voor uiHints
  - H2: Referentie voor contracts
  - H2: Referentie voor configContracts
  - H2: Referentie voor mediaUnderstandingProviderMetadata
  - H2: Referentie voor channelConfigs
  - H3: Een andere kanaalplugin vervangen
  - H2: Referentie voor modelSupport
  - H2: Referentie voor modelCatalog
  - H2: Referentie voor modelIdNormalization
  - H2: Referentie voor providerEndpoints
  - H2: Referentie voor providerRequest
  - H2: Referentie voor secretProviderIntegrations
  - H2: Referentie voor modelPricing
  - H3: OpenClaw-providerindex
  - H2: Manifest versus package.json
  - H3: Velden in package.json die de detectie beïnvloeden
  - H2: Voorrangsvolgorde bij detectie (dubbele Plugin-id's)
  - H2: Vereisten voor JSON Schema
  - H2: Validatiegedrag
  - H2: Opmerkingen
  - H2: Gerelateerd

## plugins/meeting-plugins.md

- Route: /plugins/meeting-plugins
- Koppen:
  - H2: Een Plugin kiezen
  - H2: Een modus kiezen
  - H2: Chrome en audio voorbereiden
  - H2: Plugins installeren of uitschakelen
  - H2: Verifiëren en deelnemen
  - H2: Beleidsmeldingen van het platform afhandelen
  - H2: Spraakchat in Discord
  - H2: Platformhandleidingen

## plugins/memory-lancedb.md

- Route: /plugins/memory-lancedb
- Koppen:
  - H2: Installatie
  - H2: Snel aan de slag
  - H2: Embeddingconfiguratie
  - H3: Dimensies
  - H2: Ollama-embeddings
  - H2: Limieten voor ophalen en vastleggen
  - H2: Opdrachten
  - H2: Opslag
  - H2: Runtime-afhankelijkheden en platformondersteuning
  - H2: Probleemoplossing
  - H3: Invoerlengte overschrijdt de contextlengte
  - H3: Niet-ondersteund embeddingmodel
  - H3: Plugin wordt geladen, maar er verschijnen geen herinneringen
  - H2: Gerelateerd

## plugins/memory-wiki.md

- Route: /plugins/memory-wiki
- Koppen:
  - H2: Kluismodi
  - H2: Kluisindeling
  - H2: Imports in Open Knowledge Format
  - H2: Gestructureerde beweringen en bewijs
  - H2: Entiteitsmetadata voor agents
  - H2: Compilatiepijplijn
  - H2: Dashboards en statusrapporten
  - H2: Zoeken en ophalen
  - H2: Agenttools
  - H2: Gedrag van prompts en context
  - H2: Configuratie
  - H3: Kluizen per agent
  - H3: Voorbeeld: QMD + brugmodus
  - H2: CLI
  - H2: Ondersteuning voor Obsidian
  - H2: Aanbevolen workflow
  - H2: Gerelateerde documentatie

## plugins/message-presentation.md

- Route: /plugins/message-presentation
- Koppen:
  - H2: Contract
  - H2: Voorbeelden van producenten
  - H2: Renderercontract
  - H2: Kernstroom voor rendering
  - H2: Degradatieregels
  - H3: Zichtbaarheid van de terugvalwaarde voor knoppen
  - H2: Providertoewijzing
  - H2: Presentatie versus InteractiveReply
  - H2: Vastgezette levering
  - H2: Checklist voor Plugin-auteurs
  - H2: Gerelateerde documentatie

## plugins/oc-path.md

- Route: /plugins/oc-path
- Koppen:
  - H2: Waarom je dit inschakelt
  - H2: Waar het wordt uitgevoerd
  - H2: Inschakelen
  - H2: Afhankelijkheden
  - H2: Wat het biedt
  - H2: Relatie met andere Plugins
  - H2: Veiligheid
  - H2: Gerelateerd

## plugins/onepassword.md

- Route: /plugins/onepassword
- Koppen:
  - H1: 1Password-broker voor geheimen
  - H2: Beveiligingsmodel
  - H2: Voordat je begint
  - H2: Geregistreerde geheimen configureren
  - H2: De agenttool gebruiken
  - H2: Beleidsniveaus en goedkeuringen
  - H2: Status en auditgeschiedenis inspecteren
  - H2: Gedrag van de 1Password-CLI
  - H2: Foutcodes

## plugins/plugin-inventory.md

- Route: /plugins/plugin-inventory
- Koppen:
  - H1: Plugin-inventaris
  - H2: Definities
  - H2: Een Plugin installeren
  - H2: Kernpakket voor npm
  - H2: Officiële externe pakketten
  - H2: Alleen broncodecheckout

## plugins/plugin-permission-requests.md

- Route: /plugins/plugin-permission-requests
- Koppen:
  - H2: De juiste poort kiezen
  - H2: Goedkeuring aanvragen vóór een toolaanroep
  - H2: Beslissingsgedrag
  - H2: Goedkeuringsmeldingen routeren
  - H2: Native machtigingen van Codex
  - H2: Probleemoplossing
  - H2: Gerelateerd

## plugins/reference.md

- Route: /plugins/reference
- Koppen:
  - H1: Plugin-referentie

## plugins/reference/acpx.md

- Route: /plugins/reference/acpx
- Koppen:
  - H1: ACPx-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Native Pi-sessies
  - H2: Gerelateerde documentatie

## plugins/reference/admin-http-rpc.md

- Route: /plugins/reference/admin-http-rpc
- Koppen:
  - H1: Admin Http Rpc-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/alibaba.md

- Route: /plugins/reference/alibaba
- Koppen:
  - H1: Alibaba-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/amazon-bedrock-mantle.md

- Route: /plugins/reference/amazon-bedrock-mantle
- Koppen:
  - H1: Amazon Bedrock Mantle-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/amazon-bedrock.md

- Route: /plugins/reference/amazon-bedrock
- Koppen:
  - H1: Amazon Bedrock-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/anthropic-vertex.md

- Route: /plugins/reference/anthropic-vertex
- Koppen:
  - H1: Anthropic Vertex-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Claude Fable 5
  - H2: Claude Sonnet 5

## plugins/reference/anthropic.md

- Route: /plugins/reference/anthropic
- Koppen:
  - H1: Anthropic-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/arcee.md

- Route: /plugins/reference/arcee
- Koppen:
  - H1: Arcee-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/azure-speech.md

- Route: /plugins/reference/azure-speech
- Koppen:
  - H1: Azure Speech-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/baseten.md

- Route: /plugins/reference/baseten
- Koppen:
  - H1: Baseten-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/bonjour.md

- Route: /plugins/reference/bonjour
- Koppen:
  - H1: Bonjour-Plugin
  - H2: Distributie
  - H2: Oppervlak

## plugins/reference/brave.md

- Route: /plugins/reference/brave
- Koppen:
  - H1: Brave-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/browser.md

- Route: /plugins/reference/browser
- Koppen:
  - H1: Browserplugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/byteplus.md

- Route: /plugins/reference/byteplus
- Koppen:
  - H1: BytePlus-plugin
  - H2: Distributie
  - H2: Oppervlak

## plugins/reference/canvas.md

- Route: /plugins/reference/canvas
- Koppen:
  - H1: Canvas-plugin
  - H2: Distributie
  - H2: Oppervlak

## plugins/reference/cerebras.md

- Route: /plugins/reference/cerebras
- Koppen:
  - H1: Cerebras-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/chutes.md

- Route: /plugins/reference/chutes
- Koppen:
  - H1: Chutes-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/clawrouter.md

- Route: /plugins/reference/clawrouter
- Koppen:
  - H1: ClawRouter-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/clickclack.md

- Route: /plugins/reference/clickclack
- Koppen:
  - H1: Clickclack-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/cloudflare-ai-gateway.md

- Route: /plugins/reference/cloudflare-ai-gateway
- Koppen:
  - H1: Cloudflare AI Gateway-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/codex.md

- Route: /plugins/reference/codex
- Koppen:
  - H1: Codex-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/cohere.md

- Route: /plugins/reference/cohere
- Koppen:
  - H1: Cohere-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/comfy.md

- Route: /plugins/reference/comfy
- Koppen:
  - H1: ComfyUI-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/copilot-proxy.md

- Route: /plugins/reference/copilot-proxy
- Koppen:
  - H1: Copilot Proxy-plugin
  - H2: Distributie
  - H2: Oppervlak

## plugins/reference/copilot.md

- Route: /plugins/reference/copilot
- Koppen:
  - H1: Copilot-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/crabbox.md

- Route: /plugins/reference/crabbox
- Koppen:
  - H1: Crabbox-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Configureren

## plugins/reference/cua-computer.md

- Route: /plugins/reference/cua-computer
- Koppen:
  - H1: Cua Computer-plugin
  - H2: Distributie
  - H2: Oppervlak

## plugins/reference/deepgram.md

- Route: /plugins/reference/deepgram
- Koppen:
  - H1: Deepgram-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/deepinfra.md

- Route: /plugins/reference/deepinfra
- Koppen:
  - H1: DeepInfra-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/deepseek.md

- Route: /plugins/reference/deepseek
- Koppen:
  - H1: DeepSeek-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/diagnostics-otel.md

- Route: /plugins/reference/diagnostics-otel
- Koppen:
  - H1: Diagnostiekplugin voor OpenTelemetry
  - H2: Distributie
  - H2: Oppervlak

## plugins/reference/diagnostics-prometheus.md

- Route: /plugins/reference/diagnostics-prometheus
- Koppen:
  - H1: Diagnostiekplugin voor Prometheus
  - H2: Distributie
  - H2: Oppervlak

## plugins/reference/diffs-language-pack.md

- Route: /plugins/reference/diffs-language-pack
- Koppen:
  - H1: Diffs-taalpakketplugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Toegevoegde talen

## plugins/reference/diffs.md

- Route: /plugins/reference/diffs
- Koppen:
  - H1: Diffs-plugin
  - H2: Distributie
  - H2: Oppervlak

## plugins/reference/discord.md

- Route: /plugins/reference/discord
- Koppen:
  - H1: Discord-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/document-extract.md

- Route: /plugins/reference/document-extract
- Koppen:
  - H1: Document Extract-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/duckduckgo.md

- Route: /plugins/reference/duckduckgo
- Koppen:
  - H1: DuckDuckGo-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/elevenlabs.md

- Route: /plugins/reference/elevenlabs
- Koppen:
  - H1: Elevenlabs-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/exa.md

- Route: /plugins/reference/exa
- Koppen:
  - H1: Exa-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/fal.md

- Route: /plugins/reference/fal
- Koppen:
  - H1: fal-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/featherless.md

- Route: /plugins/reference/featherless
- Koppen:
  - H1: Featherless-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/feishu.md

- Route: /plugins/reference/feishu
- Koppen:
  - H1: Feishu-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/file-transfer.md

- Route: /plugins/reference/file-transfer
- Koppen:
  - H1: Plugin voor bestandsoverdracht
  - H2: Distributie
  - H2: Oppervlak

## plugins/reference/firecrawl.md

- Route: /plugins/reference/firecrawl
- Koppen:
  - H1: Firecrawl-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/fireworks.md

- Route: /plugins/reference/fireworks
- Koppen:
  - H1: Fireworks-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/github-copilot.md

- Route: /plugins/reference/github-copilot
- Koppen:
  - H1: GitHub Copilot-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/gmi.md

- Route: /plugins/reference/gmi
- Koppen:
  - H1: Gmi-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/google-meet.md

- Route: /plugins/reference/google-meet
- Koppen:
  - H1: Google Meet-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/google.md

- Route: /plugins/reference/google
- Koppen:
  - H1: Google-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/googlechat.md

- Route: /plugins/reference/googlechat
- Koppen:
  - H1: Google Chat-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/gradium.md

- Route: /plugins/reference/gradium
- Koppen:
  - H1: Gradium-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/groq.md

- Route: /plugins/reference/groq
- Koppen:
  - H1: Groq-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/huggingface.md

- Route: /plugins/reference/huggingface
- Koppen:
  - H1: Hugging Face-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/imessage.md

- Route: /plugins/reference/imessage
- Koppen:
  - H1: iMessage-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/inworld.md

- Route: /plugins/reference/inworld
- Koppen:
  - H1: Inworld-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/irc.md

- Route: /plugins/reference/irc
- Koppen:
  - H1: IRC-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/kilocode.md

- Route: /plugins/reference/kilocode
- Koppen:
  - H1: Kilocode-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/kimi.md

- Route: /plugins/reference/kimi
- Koppen:
  - H1: Kimi-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/line.md

- Route: /plugins/reference/line
- Koppen:
  - H1: LINE-plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/linux-canvas.md

- Route: /plugins/reference/linux-canvas
- Koppen:
  - H1: Linux Canvas-plugin
  - H2: Distributie
  - H2: Oppervlak

## plugins/reference/linux-node.md

- Route: /plugins/reference/linux-node
- Koppen:
  - H1: Linux Node-Plugin
  - H2: Distributie
  - H2: Oppervlak

## plugins/reference/litellm.md

- Route: /plugins/reference/litellm
- Koppen:
  - H1: LiteLLM-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/llama-cpp.md

- Route: /plugins/reference/llama-cpp
- Koppen:
  - H1: Llama Cpp-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Standaardtekstmodel
  - H2: Gerelateerde documentatie

## plugins/reference/llm-task.md

- Route: /plugins/reference/llm-task
- Koppen:
  - H1: LLM Task-Plugin
  - H2: Distributie
  - H2: Oppervlak

## plugins/reference/lmstudio.md

- Route: /plugins/reference/lmstudio
- Koppen:
  - H1: LM Studio-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/lobster.md

- Route: /plugins/reference/lobster
- Koppen:
  - H1: Lobster-Plugin
  - H2: Distributie
  - H2: Oppervlak

## plugins/reference/logbook.md

- Route: /plugins/reference/logbook
- Koppen:
  - H1: Logbook-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/longcat.md

- Route: /plugins/reference/longcat
- Koppen:
  - H1: LongCat-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/matrix.md

- Route: /plugins/reference/matrix
- Koppen:
  - H1: Matrix-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/mattermost.md

- Route: /plugins/reference/mattermost
- Koppen:
  - H1: Mattermost-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/memory-core.md

- Route: /plugins/reference/memory-core
- Koppen:
  - H1: Memory Core-Plugin
  - H2: Distributie
  - H2: Oppervlak

## plugins/reference/memory-lancedb.md

- Route: /plugins/reference/memory-lancedb
- Koppen:
  - H1: Memory Lancedb-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/memory-wiki.md

- Route: /plugins/reference/memory-wiki
- Koppen:
  - H1: Memory Wiki-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/meta.md

- Route: /plugins/reference/meta
- Koppen:
  - H1: Meta-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/microsoft-foundry.md

- Route: /plugins/reference/microsoft-foundry
- Koppen:
  - H1: Microsoft Foundry-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Vereisten
  - H2: Chatmodellen
  - H2: MAI-afbeeldingen genereren
  - H2: Probleemoplossing

## plugins/reference/microsoft.md

- Route: /plugins/reference/microsoft
- Koppen:
  - H1: Microsoft-Plugin
  - H2: Distributie
  - H2: Oppervlak

## plugins/reference/migrate-claude.md

- Route: /plugins/reference/migrate-claude
- Koppen:
  - H1: Plugin voor migratie van Claude
  - H2: Distributie
  - H2: Oppervlak

## plugins/reference/migrate-hermes.md

- Route: /plugins/reference/migrate-hermes
- Koppen:
  - H1: Plugin voor migratie van Hermes
  - H2: Distributie
  - H2: Oppervlak

## plugins/reference/minimax.md

- Route: /plugins/reference/minimax
- Koppen:
  - H1: MiniMax-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/mistral.md

- Route: /plugins/reference/mistral
- Koppen:
  - H1: Mistral-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/moonshot.md

- Route: /plugins/reference/moonshot
- Koppen:
  - H1: Moonshot-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/msteams.md

- Route: /plugins/reference/msteams
- Koppen:
  - H1: Microsoft Teams-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/mxc.md

- Route: /plugins/reference/mxc
- Koppen:
  - H1: Mxc-Plugin
  - H2: Distributie
  - H2: Oppervlak

## plugins/reference/nextcloud-talk.md

- Route: /plugins/reference/nextcloud-talk
- Koppen:
  - H1: Nextcloud Talk-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/nostr.md

- Route: /plugins/reference/nostr
- Koppen:
  - H1: Nostr-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/novita.md

- Route: /plugins/reference/novita
- Koppen:
  - H1: Novita-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/nvidia.md

- Route: /plugins/reference/nvidia
- Koppen:
  - H1: NVIDIA-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/oc-path.md

- Route: /plugins/reference/oc-path
- Koppen:
  - H1: Oc Path-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/ollama.md

- Route: /plugins/reference/ollama
- Koppen:
  - H1: Ollama-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/onepassword.md

- Route: /plugins/reference/onepassword
- Koppen:
  - H1: Onepassword-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/open-prose.md

- Route: /plugins/reference/open-prose
- Koppen:
  - H1: Open Prose-Plugin
  - H2: Distributie
  - H2: Oppervlak

## plugins/reference/openai.md

- Route: /plugins/reference/openai
- Koppen:
  - H1: OpenAI-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/opencode-go.md

- Route: /plugins/reference/opencode-go
- Koppen:
  - H1: OpenCode Go-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/opencode.md

- Route: /plugins/reference/opencode
- Koppen:
  - H1: OpenCode-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Native sessies
  - H2: Gerelateerde documentatie

## plugins/reference/openrouter.md

- Route: /plugins/reference/openrouter
- Koppen:
  - H1: OpenRouter-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/openshell.md

- Route: /plugins/reference/openshell
- Koppen:
  - H1: Openshell-Plugin
  - H2: Distributie
  - H2: Oppervlak

## plugins/reference/perplexity.md

- Route: /plugins/reference/perplexity
- Koppen:
  - H1: Perplexity-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/pixverse.md

- Route: /plugins/reference/pixverse
- Koppen:
  - H1: PixVerse-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/policy.md

- Route: /plugins/reference/policy
- Koppen:
  - H1: Beleids-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gedrag
  - H2: Gerelateerde documentatie

## plugins/reference/qa-channel.md

- Route: /plugins/reference/qa-channel
- Koppen:
  - H1: QA Channel-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/qa-lab.md

- Route: /plugins/reference/qa-lab
- Koppen:
  - H1: QA Lab-Plugin
  - H2: Distributie
  - H2: Oppervlak

## plugins/reference/qianfan.md

- Route: /plugins/reference/qianfan
- Koppen:
  - H1: Qianfan-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/qqbot.md

- Route: /plugins/reference/qqbot
- Koppen:
  - H1: QQ Bot-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/qwen.md

- Route: /plugins/reference/qwen
- Koppen:
  - H1: Qwen-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/raft.md

- Route: /plugins/reference/raft
- Koppen:
  - H1: Raft-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/reef.md

- Route: /plugins/reference/reef
- Koppen:
  - H1: Reef-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/runway.md

- Route: /plugins/reference/runway
- Koppen:
  - H1: Runway-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/searxng.md

- Route: /plugins/reference/searxng
- Koppen:
  - H1: SearXNG-Plugin
  - H2: Distributie
  - H2: Oppervlak

## plugins/reference/senseaudio.md

- Route: /plugins/reference/senseaudio
- Koppen:
  - H1: Senseaudio-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/sglang.md

- Route: /plugins/reference/sglang
- Koppen:
  - H1: SGLang-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/signal.md

- Route: /plugins/reference/signal
- Koppen:
  - H1: Signal-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/slack.md

- Route: /plugins/reference/slack
- Koppen:
  - H1: Slack-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/sms.md

- Route: /plugins/reference/sms
- Koppen:
  - H1: Sms-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/stepfun.md

- Route: /plugins/reference/stepfun
- Koppen:
  - H1: StepFun-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/synology-chat.md

- Route: /plugins/reference/synology-chat
- Koppen:
  - H1: Synology Chat-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/synthetic.md

- Route: /plugins/reference/synthetic
- Koppen:
  - H1: Synthetic-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/tavily.md

- Route: /plugins/reference/tavily
- Koppen:
  - H1: Tavily-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/teams-meetings.md

- Route: /plugins/reference/teams-meetings
- Koppen:
  - H1: Plugin voor Microsoft Teams-vergaderingen
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/telegram.md

- Route: /plugins/reference/telegram
- Koppen:
  - H1: Telegram-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/tencent.md

- Route: /plugins/reference/tencent
- Koppen:
  - H1: Tencent-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/tlon.md

- Route: /plugins/reference/tlon
- Koppen:
  - H1: Tlon-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/together.md

- Route: /plugins/reference/together
- Koppen:
  - H1: Together-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/tokenjuice.md

- Route: /plugins/reference/tokenjuice
- Koppen:
  - H1: Tokenjuice-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/tts-local-cli.md

- Route: /plugins/reference/tts-local-cli
- Koppen:
  - H1: TTS Local CLI-Plugin
  - H2: Distributie
  - H2: Oppervlak

## plugins/reference/twitch.md

- Route: /plugins/reference/twitch
- Koppen:
  - H1: Twitch-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/vault.md

- Route: /plugins/reference/vault
- Koppen:
  - H1: Vault-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/venice.md

- Route: /plugins/reference/venice
- Koppen:
  - H1: Venice-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/vercel-ai-gateway.md

- Route: /plugins/reference/vercel-ai-gateway
- Koppen:
  - H1: Vercel AI Gateway-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/vllm.md

- Route: /plugins/reference/vllm
- Koppen:
  - H1: vLLM-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/voice-call.md

- Route: /plugins/reference/voice-call
- Koppen:
  - H1: Plugin voor spraakoproepen
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/volcengine.md

- Route: /plugins/reference/volcengine
- Koppen:
  - H1: Volcengine-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/voyage.md

- Route: /plugins/reference/voyage
- Koppen:
  - H1: Voyage-Plugin
  - H2: Distributie
  - H2: Oppervlak

## plugins/reference/vydra.md

- Route: /plugins/reference/vydra
- Koppen:
  - H1: Vydra-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/web-readability.md

- Route: /plugins/reference/web-readability
- Koppen:
  - H1: Plugin voor webleesbaarheid
  - H2: Distributie
  - H2: Oppervlak

## plugins/reference/webhooks.md

- Route: /plugins/reference/webhooks
- Koppen:
  - H1: Webhooks-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/whatsapp.md

- Route: /plugins/reference/whatsapp
- Koppen:
  - H1: WhatsApp-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/workboard.md

- Route: /plugins/reference/workboard
- Koppen:
  - H1: Workboard-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/xai.md

- Route: /plugins/reference/xai
- Koppen:
  - H1: xAI-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/xiaomi.md

- Route: /plugins/reference/xiaomi
- Koppen:
  - H1: Xiaomi-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/zai.md

- Route: /plugins/reference/zai
- Koppen:
  - H1: Z.AI-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/zalo.md

- Route: /plugins/reference/zalo
- Koppen:
  - H1: Zalo-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/zalouser.md

- Route: /plugins/reference/zalouser
- Koppen:
  - H1: Zalo Personal-Plugin
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/reference/zoom-meetings.md

- Route: /plugins/reference/zoom-meetings
- Koppen:
  - H1: Plugin voor Zoom-vergaderingen
  - H2: Distributie
  - H2: Oppervlak
  - H2: Gerelateerde documentatie

## plugins/sdk-agent-harness.md

- Route: /plugins/sdk-agent-harness
- Koppen:
  - H2: Wanneer je een harness gebruikt
  - H2: Wat de kern nog steeds beheert
  - H3: Door de harness beheerde authenticatie-initialisatie
  - H3: Geverifieerde runtime-artefacten voor de configuratie
  - H3: Contract voor verzoektransport
  - H2: Een harness registreren
  - H3: Gedelegeerde uitvoering
  - H2: Selectiebeleid
  - H2: Koppeling van provider en harness
  - H3: Middleware voor toolresultaten
  - H3: Classificatie van terminalresultaten
  - H3: Neveneffecten aan het einde van een agent
  - H3: Gebruersinvoer en tooloppervlakken
  - H3: Native Codex-harnessmodus
  - H2: Striktheid van de runtime
  - H2: Native sessies en transcriptspiegel
  - H2: Tool- en mediaresultaten
  - H3: Resultaten van terminaltools
  - H3: Definitieve afhandeling van voltooide tools
  - H2: Huidige beperkingen
  - H2: Gerelateerd

## plugins/sdk-channel-inbound.md

- Route: /plugins/sdk-channel-inbound
- Koppen:
  - H2: Kernhelpers
  - H2: Contract voor de afhandeling van bezorging
  - H2: Migratie

## plugins/sdk-channel-ingress.md

- Route: /plugins/sdk-channel-ingress
- Koppen:
  - H2: Runtime-resolver
  - H2: Resultaat
  - H2: Toegangsgroepen
  - H2: Gebeurtenismodi
  - H2: Routes en activering
  - H2: Redactie
  - H2: Verificatie

## plugins/sdk-channel-message.md

- Route: /plugins/sdk-channel-message
- Koppen: geen

## plugins/sdk-channel-outbound.md

- Route: /plugins/sdk-channel-outbound
- Koppen:
  - H2: Duurzame ingress-monitors
  - H2: Adapter
  - H2: Onderdrukking van uitgaande echo's
  - H2: Opschoning van platte tekst
  - H2: Bewijs van bezorging
  - H2: Bestaande uitgaande adapters
  - H2: Duurzame verzendingen
  - H2: Uitgestelde toelating voor bezorging
  - H2: Compatibiliteitsdispatch

## plugins/sdk-channel-plugins.md

- Route: /plugins/sdk-channel-plugins
- Koppen:
  - H2: Waarvoor jouw plugin verantwoordelijk is
  - H2: Berichtenadapter
  - H3: Inkomende ontvangst (experimenteel)
  - H3: Duurzame ontvangst en ontdubbeling bij herhaling
  - H4: Transportklassen en bewaartermijnen
  - H4: Bijwerkingen met ten minste één uitvoering
  - H4: Accountgebonden herstartcontract
  - H3: Typindicatoren
  - H3: Parameters voor mediabronnen
  - H3: Vormgeving van systeemeigen payloads
  - H3: Grammatica voor sessiegesprekken
  - H3: Ondersteuning voor accountgebonden gesprekskoppeling
  - H2: Goedkeuringen en kanaalmogelijkheden
  - H3: Authenticatie voor goedkeuringen
  - H3: Levenscyclus van payloads en configuratierichtlijnen
  - H3: Systeemeigen levering van goedkeuringen
  - H3: Nauwere runtime-subpaden voor goedkeuringen
  - H3: Configuratiesubpaden
  - H3: Andere nauwe kanaalsubpaden
  - H2: Beleid voor vermeldingen in inkomende berichten
  - H2: Stapsgewijze uitleg
  - H2: Bestandsstructuur
  - H2: Geavanceerde onderwerpen
  - H2: Volgende stappen
  - H2: Gerelateerd

## plugins/sdk-channel-turn.md

- Route: /plugins/sdk-channel-turn
- Koppen: geen

## plugins/sdk-entrypoints.md

- Route: /plugins/sdk-entrypoints
- Koppen:
  - H2: Pakket-ingangen
  - H2: defineToolPlugin
  - H2: definePluginEntry
  - H2: defineChannelPluginEntry
  - H2: defineSetupPluginEntry
  - H2: Registratiemodus
  - H2: Plugin-vormen
  - H2: Gerelateerd

## plugins/sdk-migration.md

- Route: /plugins/sdk-migration
- Koppen:
  - H2: Wat er is gewijzigd
  - H3: Waarom
  - H2: Compatibiliteitsbeleid
  - H3: Compatibiliteit van gepubliceerde kanaalconfiguraties
  - H3: Compatibiliteit van invoervelden voor kanaalconfiguratie
  - H4: Lezers verifiëren
  - H3: Verouderde mediaprojectie
  - H2: Migreren
  - H2: Referentie voor importpaden
  - H2: Verwijderde compatibiliteitsoppervlakken
  - H3: Procesbrede publicatie van API-providers
  - H3: Privétestbarrel
  - H2: Migratiereferentie
  - H2: Migratie van spraak en realtime stem
  - H2: Tijdlijn voor verwijdering
  - H2: De waarschuwingen tijdelijk onderdrukken
  - H2: Gerelateerd

## plugins/sdk-overview.md

- Route: /plugins/sdk-overview
- Koppen:
  - H2: Importconventie
  - H2: Referentie voor subpaden
  - H2: Registratie-API
  - H3: Registratie van mogelijkheden
  - H3: Hulpmiddelen en opdrachten
  - H3: Infrastructuur
  - H4: Webhook-werk na bevestiging
  - H4: MCP-verbindingen met een bereik per aanvrager
  - H3: Host-hooks voor workflowplugins
  - H3: Registratie voor Gateway-detectie
  - H3: Registratiemetadata voor de CLI
  - H3: Registratie van CLI-backends
  - H3: Exclusieve posities
  - H3: Verouderde adapters voor geheugenembeddings
  - H3: Gebeurtenissen en levenscyclus
  - H3: Beslissingssemantiek van hooks
  - H3: Velden van het API-object
  - H2: Conventie voor interne modules
  - H2: Gerelateerd

## plugins/sdk-provider-plugins.md

- Route: /plugins/sdk-provider-plugins
- Koppen:
  - H2: Stapsgewijze uitleg
  - H2: Publiceren naar ClawHub
  - H2: Bestandsstructuur
  - H2: Referentie voor catalogusvolgorde
  - H2: Volgende stappen
  - H2: Gerelateerd

## plugins/sdk-runtime.md

- Route: /plugins/sdk-runtime
- Koppen:
  - H2: Configuratie laden en schrijven
  - H2: Herbruikbare runtime-hulpprogramma's
  - H2: Runtime-naamruimten
  - H2: Runtime-verwijzingen opslaan
  - H2: Andere API-velden op het hoogste niveau
  - H2: Gerelateerd

## plugins/sdk-setup.md

- Route: /plugins/sdk-setup
- Koppen:
  - H2: Pakketmetadata
  - H3: openclaw-velden
  - H3: openclaw.channel
  - H3: Configuratievelden die door het kanaal worden beheerd
  - H3: openclaw.install
  - H3: Uitgesteld volledig laden
  - H2: Plugin-manifest
  - H2: Publiceren naar ClawHub
  - H2: Configuratie-ingang
  - H3: Imports van gerichte configuratiehelpers
  - H3: Invoervelden voor configuratie die door het kanaal worden beheerd
  - H3: Promotie naar één account die door het kanaal wordt beheerd
  - H2: Configuratieschema
  - H3: Configuratieschema's voor kanalen bouwen
  - H2: Configuratiewizards
  - H2: Publiceren en installeren
  - H2: Gerelateerd

## plugins/sdk-subpaths.md

- Route: /plugins/sdk-subpaths
- Koppen:
  - H2: Plugin-ingang
  - H3: Compatibiliteitshelpers en privé-lokale helpers
  - H3: Helpersubpaden voor gebundelde plugins
  - H2: Gerelateerd

## plugins/sdk-testing.md

- Route: /plugins/sdk-testing
- Koppen:
  - H2: Testhulpprogramma's
  - H3: Beschikbare exports
  - H3: Typen
  - H2: Doelresolutie testen
  - H2: Testpatronen
  - H3: Registratiecontracten testen
  - H3: Toegang tot runtimeconfiguratie testen
  - H3: Een kanaalplugin unit-testen
  - H3: Een providerplugin unit-testen
  - H3: De plugin-runtime nabootsen
  - H3: Testen met stubs per instantie
  - H2: Contracttests (plugins in de repository)
  - H3: Afgebakende tests uitvoeren
  - H2: Lintafdwinging (plugins in de repository)
  - H2: Testconfiguratie
  - H2: Gerelateerd

## plugins/teams-meetings.md

- Route: /plugins/teams-meetings
- Koppen:
  - H2: Configuratie
  - H2: Modi
  - H2: Limieten voor deelname als gast
  - H2: Oppervlak voor hulpmiddelen en Gateway
  - H2: Gerelateerd

## plugins/tool-plugins.md

- Route: /plugins/tool-plugins
- Koppen:
  - H2: Vereisten
  - H2: Snel aan de slag
  - H2: Een hulpmiddel schrijven
  - H2: Optionele hulpmiddelen en fabriekshulpmiddelen
  - H2: Retourwaarden
  - H2: Uitvoercontracten
  - H2: Configuratie
  - H2: Gegenereerde metadata
  - H2: Pakketmetadata
  - H2: Valideren in CI
  - H2: Lokaal installeren en inspecteren
  - H2: Publiceren
  - H2: Probleemoplossing
  - H3: plugin-ingang niet gevonden: ./dist/index.js
  - H3: plugin-ingang stelt geen defineToolPlugin-metadata beschikbaar
  - H3: Gegenereerde metadata van openclaw.plugin.json zijn verouderd
  - H3: package.json openclaw.extensions moet ./dist/index.js bevatten
  - H3: Kan pakket 'typebox' niet vinden
  - H3: Hulpmiddel verschijnt niet na installatie
  - H2: Zie ook

## plugins/vault.md

- Route: /plugins/vault
- Koppen:
  - H1: Vault SecretRefs
  - H2: Voordat je begint
  - H2: Een providersleutel opslaan in Vault
  - H2: Vault zichtbaar maken voor de Gateway
  - H2: Een SecretRef-plan genereren en toepassen
  - H2: Meer providersleutels configureren
  - H2: Indeling van SecretRef-id's
  - H2: Wat OpenClaw opslaat
  - H2: Containers en beheerde implementaties
  - H2: Gerelateerd

## plugins/voice-call.md

- Route: /plugins/voice-call
- Koppen:
  - H2: Snel aan de slag
  - H2: Configuratie
  - H3: Configuratiereferentie
  - H2: Sessiebereik
  - H2: Realtime spraakgesprekken
  - H3: Beleid voor hulpmiddelen
  - H3: Spraakcontext van de agent
  - H3: Voorbeelden van realtimeproviders
  - H2: Streamingtranscriptie
  - H3: Voorbeelden van streamingproviders
  - H2: TTS voor gesprekken
  - H3: TTS-voorbeelden
  - H2: Inkomende gesprekken
  - H3: Routering per nummer
  - H3: Contract voor gesproken uitvoer
  - H3: Gedrag bij het starten van een gesprek
  - H3: Respijtperiode bij verbreking van een Twilio-stream
  - H2: Opruimer voor verouderde gesprekken
  - H2: Webhook-beveiliging
  - H2: CLI
  - H2: Agenthulpmiddel
  - H2: Gateway-RPC
  - H2: Probleemoplossing
  - H3: Beschikbaarstelling van Webhook mislukt tijdens de configuratie
  - H3: Providerreferenties werken niet
  - H3: Gesprekken starten, maar providerwebhooks komen niet aan
  - H3: Handtekeningverificatie mislukt
  - H3: Deelname aan Google Meet via Twilio mislukt
  - H3: Realtimegesprek bevat geen spraak
  - H2: Gerelateerd

## plugins/webhooks.md

- Route: /plugins/webhooks
- Koppen:
  - H2: Routes configureren
  - H2: Beveiligingsmodel
  - H2: Aanvraagindeling
  - H2: Ondersteunde acties
  - H3: `create_flow`
  - H3: `run_task`
  - H2: Antwoordstructuur
  - H2: Gerelateerd

## plugins/workboard.md

- Route: /plugins/workboard
- Koppen:
  - H2: Inschakelen
  - H2: Configuratie
  - H2: Kaartvelden
  - H2: Werk starten vanaf een kaart
  - H2: Agenthulpmiddelen
  - H2: Taaktoewijzing
  - H3: Workerselectie
  - H3: Ingangen
  - H2: CLI- en slashopdracht
  - H2: Synchronisatie van de sessielevenscyclus
  - H2: Dashboardworkflow
  - H3: Widgets voor sessieborden
  - H2: Diagnostiek
  - H2: Machtigingen
  - H2: Opslag
  - H2: Probleemoplossing
  - H2: Gerelateerd

## plugins/zalouser.md

- Route: /plugins/zalouser
- Koppen:
  - H2: Naamgeving
  - H2: Waar het wordt uitgevoerd
  - H2: Installeren
  - H3: Vanuit npm
  - H3: Vanuit een lokale map (ontwikkeling)
  - H2: Configuratie
  - H2: CLI
  - H2: Agenthulpmiddel
  - H2: Gerelateerd

## plugins/zoom-meetings.md

- Route: /plugins/zoom-meetings
- Koppen:
  - H2: Configuratie
  - H2: Modi
  - H2: Limieten voor deelname als gast
  - H2: Oppervlak voor hulpmiddelen en Gateway
  - H2: Gerelateerd

## prose.md

- Route: /prose
- Koppen:
  - H2: Installeren
  - H2: Slashopdracht
  - H2: Wat het kan
  - H2: Voorbeeld: parallel onderzoek en synthese
  - H2: OpenClaw-runtimekoppeling
  - H2: Bestandslocaties
  - H2: Statusbackends
  - H2: Beveiliging
  - H2: Gerelateerd

## providers/alibaba.md

- Route: /providers/alibaba
- Koppen:
  - H2: Aan de slag
  - H2: Ingebouwde Wan-modellen
  - H2: Mogelijkheden en limieten
  - H2: Geavanceerde configuratie
  - H2: Gerelateerd

## providers/anthropic.md

- Route: /providers/anthropic
- Koppen:
  - H2: Gebruiks- en kostentracking
  - H2: Aan de slag
  - H2: Claude-sessies op meerdere computers
  - H2: Standaardinstellingen voor denkwerk (Claude Opus 5, Sonnet 5, Mythos 5, Fable 5, 4.8 en 4.6)
  - H2: Terugval bij veiligheidsweigering (Claude Fable 5)
  - H3: Waarom dit bestaat
  - H3: Hoe het werkt
  - H3: Observatie en facturering
  - H3: Reikwijdte
  - H2: Promptcaching
  - H2: Geavanceerde configuratie
  - H2: Probleemoplossing
  - H2: Gerelateerd

## providers/arcee.md

- Route: /providers/arcee
- Koppen:
  - H2: Plugin installeren
  - H2: Aan de slag
  - H2: Niet-interactieve configuratie
  - H2: Rechtstreekse Arcee-catalogus
  - H2: OpenRouter-catalogus
  - H2: Ondersteunde functies
  - H2: Gerelateerd

## providers/azure-speech.md

- Route: /providers/azure-speech
- Koppen:
  - H2: Aan de slag
  - H2: Configuratieopties
  - H2: Opmerkingen
  - H2: Gerelateerd

## providers/baseten.md

- Route: /providers/baseten
- Koppen:
  - H2: Plugin installeren
  - H2: Aan de slag
  - H2: Inkling
  - H2: Meegeleverde terugvalcatalogus
  - H2: Handmatige configuratie
  - H2: Gerelateerd

## providers/bedrock-mantle.md

- Route: /providers/bedrock-mantle
- Koppen:
  - H2: Aan de slag
  - H2: Automatische modeldetectie
  - H3: Ondersteunde regio's
  - H2: Handmatige configuratie
  - H2: Geavanceerde configuratie
  - H2: Gerelateerd

## providers/bedrock.md

- Route: /providers/bedrock
- Koppen:
  - H2: Aan de slag
  - H2: Automatische modeldetectie
  - H2: Snelle configuratie (AWS-pad)
  - H2: Geavanceerde configuratie
  - H2: Gerelateerd

## providers/cerebras.md

- Route: /providers/cerebras
- Koppen:
  - H2: Plugin installeren
  - H2: Aan de slag
  - H2: Niet-interactieve configuratie
  - H2: Ingebouwde catalogus
  - H2: Handmatige configuratie
  - H2: Gerelateerd

## providers/chutes.md

- Route: /providers/chutes
- Koppen:
  - H2: Plugin installeren
  - H2: Aan de slag
  - H2: Detectiegedrag
  - H2: Standaardaliassen
  - H2: Ingebouwde startcatalogus
  - H2: Configuratievoorbeeld
  - H2: Gerelateerd

## providers/claude-max-api-proxy.md

- Route: /providers/claude-max-api-proxy
- Koppen:
  - H2: Waarom je dit gebruikt
  - H2: Hoe het werkt
  - H2: Aan de slag
  - H2: Geavanceerde configuratie
  - H2: Opmerkingen
  - H2: Gerelateerd

## providers/clawrouter.md

- Route: /providers/clawrouter
- Koppen:
  - H2: Aan de slag
  - H2: Beheerde niet-interactieve implementatie
  - H2: Gereedheid en livebewijs
  - H2: Modeldetectie
  - H2: Protocol- en providerplugins
  - H2: Quota en gebruik
  - H2: Probleemoplossing
  - H2: Beveiligingsgedrag
  - H2: Gerelateerd

## providers/cloudflare-ai-gateway.md

- Route: /providers/cloudflare-ai-gateway
- Koppen:
  - H2: Plugin installeren
  - H2: Aan de slag
  - H2: Niet-interactief voorbeeld
  - H2: Geavanceerde configuratie
  - H2: Gerelateerd

## providers/cohere.md

- Route: /providers/cohere
- Koppen:
  - H2: Ingebouwde catalogus
  - H2: Aan de slag
  - H2: Configuratie uitsluitend via de omgeving
  - H2: Gerelateerd

## providers/comfy.md

- Route: /providers/comfy
- Koppen:
  - H2: Wat wordt ondersteund
  - H2: Aan de slag
  - H2: Configuratie
  - H3: Gedeelde sleutels
  - H3: Sleutels per mogelijkheid
  - H2: Workflowdetails
  - H2: Gerelateerd

## providers/deepgram.md

- Route: /providers/deepgram
- Koppen:
  - H2: Aan de slag
  - H2: Configuratieopties
  - H2: Streaming-STT voor spraakoproepen
  - H2: Opmerkingen
  - H2: Gerelateerd

## providers/deepinfra.md

- Route: /providers/deepinfra
- Koppen:
  - H2: Plugin installeren
  - H2: Een API-sleutel verkrijgen
  - H2: CLI-configuratie
  - H2: Configuratiefragment
  - H2: Ondersteunde oppervlakken
  - H2: Beschikbare modellen
  - H2: Opmerkingen
  - H2: Gerelateerd

## providers/deepseek.md

- Route: /providers/deepseek
- Koppen:
  - H2: Plugin installeren
  - H2: Aan de slag
  - H2: Ingebouwde catalogus
  - H2: Denkwerk en tools
  - H2: Live testen
  - H2: Configuratievoorbeeld
  - H2: Gerelateerd

## providers/ds4.md

- Route: /providers/ds4
- Koppen:
  - H2: Vereisten
  - H2: Snelstart
  - H2: Volledige configuratie
  - H2: Opstarten op aanvraag
  - H2: Think Max
  - H2: Test
  - H2: Probleemoplossing
  - H2: Gerelateerd

## providers/elevenlabs.md

- Route: /providers/elevenlabs
- Koppen:
  - H2: Authenticatie
  - H2: Tekst-naar-spraak
  - H2: Spraak-naar-tekst
  - H2: Streaming-STT
  - H2: Gerelateerd

## providers/fal.md

- Route: /providers/fal
- Koppen:
  - H2: Aan de slag
  - H2: Afbeeldingen genereren
  - H2: Video's genereren
  - H2: Muziek genereren
  - H2: Gerelateerd

## providers/featherless.md

- Route: /providers/featherless
- Koppen:
  - H2: Configuratie
  - H2: Standaardmodel
  - H2: Andere Featherless-modellen
  - H2: Probleemoplossing
  - H2: Gerelateerd

## providers/fireworks.md

- Route: /providers/fireworks
- Koppen:
  - H2: Aan de slag
  - H2: Niet-interactieve configuratie
  - H2: Ingebouwde catalogus
  - H2: Aangepaste Fireworks-model-id's
  - H2: Gerelateerd

## providers/github-copilot.md

- Route: /providers/github-copilot
- Koppen:
  - H2: Drie manieren om Copilot in OpenClaw te gebruiken
  - H2: GitHub Enterprise (gegevenslocatie)
  - H2: Optionele vlaggen
  - H2: Niet-interactieve onboarding
  - H2: Embeddings voor geheugenzoekopdrachten
  - H3: Configuratie
  - H3: Hoe het werkt
  - H2: Gerelateerd

## providers/gmi.md

- Route: /providers/gmi
- Koppen:
  - H2: Configuratie
  - H2: Wanneer je GMI kiest
  - H2: Modellen
  - H2: Probleemoplossing
  - H2: Gerelateerd

## providers/google.md

- Route: /providers/google
- Koppen:
  - H2: Aan de slag
  - H2: Mogelijkheden
  - H2: Zoeken op internet
  - H2: Afbeeldingen genereren
  - H2: Video's genereren
  - H2: Muziek genereren
  - H2: Tekst-naar-spraak
  - H2: Realtime spraak
  - H2: Geavanceerde configuratie
  - H2: Gerelateerd

## providers/gradium.md

- Route: /providers/gradium
- Koppen:
  - H2: Plugin installeren
  - H2: Configuratie
  - H2: Configuratie
  - H2: Stemmen
  - H3: Stem per bericht overschrijven
  - H2: Uitvoer
  - H2: Volgorde voor automatische selectie
  - H2: Gerelateerd

## providers/groq.md

- Route: /providers/groq
- Koppen:
  - H2: Plugin installeren
  - H2: Aan de slag
  - H3: Voorbeeld van een configuratiebestand
  - H2: Ingebouwde catalogus
  - H2: Redeneermodellen
  - H2: Audiotranscriptie
  - H2: Gerelateerd

## providers/huggingface.md

- Route: /providers/huggingface
- Koppen:
  - H2: Aan de slag
  - H3: Niet-interactieve configuratie
  - H2: Model-id's
  - H2: Geavanceerde configuratie
  - H2: Gerelateerd

## providers/index.md

- Route: /providers
- Koppen:
  - H2: Snelstart
  - H2: Providerdocumentatie
  - H2: Gedeelde overzichtspagina's
  - H2: Transcriptieproviders
  - H2: Communitytools

## providers/inferrs.md

- Route: /providers/inferrs
- Koppen:
  - H2: Aan de slag
  - H2: Volledig configuratievoorbeeld
  - H2: Opstarten op aanvraag
  - H2: Geavanceerde configuratie
  - H2: Probleemoplossing
  - H2: Gerelateerd

## providers/inworld.md

- Route: /providers/inworld
- Koppen:
  - H2: Plugin installeren
  - H2: Aan de slag
  - H2: Configuratieopties
  - H2: Opmerkingen
  - H2: Gerelateerd

## providers/kilocode.md

- Route: /providers/kilocode
- Koppen:
  - H2: Plugin installeren
  - H2: Configuratie
  - H2: Standaardmodel en catalogus
  - H2: Configuratievoorbeeld
  - H2: Opmerkingen over het gedrag
  - H2: Gerelateerd

## providers/litellm.md

- Route: /providers/litellm
- Koppen:
  - H2: Snelstart
  - H2: Configuratie
  - H2: Afbeeldingen genereren
  - H2: Geavanceerd
  - H2: Gerelateerd

## providers/lmstudio.md

- Route: /providers/lmstudio
- Koppen:
  - H2: Snelstart
  - H2: Niet-interactieve onboarding
  - H2: Configuratie
  - H3: Compatibiliteit met streaminggebruik
  - H3: Compatibiliteit met denkwerk
  - H3: Expliciete configuratie
  - H3: Vooraf laden uitschakelen
  - H3: LAN- of tailnet-host
  - H2: Probleemoplossing
  - H3: LM Studio niet gedetecteerd
  - H3: Authenticatiefouten (HTTP 401)
  - H2: Gerelateerd

## providers/longcat.md

- Route: /providers/longcat
- Koppen:
  - H2: Plugin installeren
  - H2: Aan de slag
  - H3: Niet-interactieve configuratie
  - H2: Redeneergedrag
  - H2: Prijzen
  - H2: Zelfgehoste LongCat-2.0
  - H2: Probleemoplossing
  - H2: Gerelateerd

## providers/meta.md

- Route: /providers/meta
- Koppen:
  - H2: Aan de slag
  - H2: Niet-interactieve configuratie
  - H2: Ingebouwde catalogus
  - H2: Handmatige configuratie
  - H2: Rooktest
  - H2: Gerelateerd

## providers/minimax.md

- Route: /providers/minimax
- Koppen:
  - H2: Ingebouwde catalogus
  - H2: Aan de slag
  - H2: Configureren via openclaw configure
  - H2: Mogelijkheden
  - H3: Afbeeldingen genereren
  - H3: Tekst-naar-spraak
  - H3: Muziek genereren
  - H3: Video's genereren
  - H3: Afbeeldingen begrijpen
  - H3: Zoeken op het web
  - H2: Geavanceerde configuratie
  - H2: Opmerkingen
  - H2: Probleemoplossing
  - H2: Gerelateerd

## providers/mistral.md

- Route: /providers/mistral
- Koppen:
  - H2: Aan de slag
  - H2: Ingebouwde LLM-catalogus
  - H2: Audiotranscriptie (Voxtral)
  - H2: Streaming-STT voor Voice Call
  - H2: Geavanceerde configuratie
  - H2: Gerelateerd

## providers/models.md

- Route: /providers/models
- Koppen:
  - H2: Snel aan de slag (twee stappen)
  - H2: Ondersteunde providers (startset)
  - H2: Aanvullende providervarianten
  - H2: Gerelateerd

## providers/moonshot.md

- Route: /providers/moonshot
- Koppen:
  - H2: Ingebouwde modelcatalogus
  - H2: Aan de slag
  - H2: Kimi-zoekfunctie voor het web
  - H2: Geavanceerde configuratie
  - H2: Gerelateerd

## providers/novita.md

- Route: /providers/novita
- Koppen:
  - H2: Configuratie
  - H2: Standaardinstellingen
  - H2: Meegeleverde modelcatalogus
  - H2: Wanneer je Novita kiest
  - H2: Probleemoplossing
  - H2: Gerelateerd

## providers/nvidia.md

- Route: /providers/nvidia
- Koppen:
  - H2: Aan de slag
  - H2: Configuratievoorbeeld
  - H2: Uitgelichte catalogus
  - H2: Nemotron 3 Ultra
  - H2: Meegeleverde terugvalcatalogus
  - H2: Geavanceerde configuratie
  - H2: Gerelateerd

## providers/ollama-cloud.md

- Route: /providers/ollama-cloud
- Koppen:
  - H2: Configuratie
  - H2: Standaardinstellingen
  - H2: Wanneer je Ollama Cloud kiest
  - H2: Modellen
  - H2: Livetest
  - H2: Probleemoplossing
  - H2: Gerelateerd

## providers/ollama.md

- Route: /providers/ollama
- Koppen:
  - H2: Authenticatieregels
  - H2: Aan de slag
  - H2: Cloudmodellen via een lokale host
  - H2: Modeldetectie (impliciete provider)
  - H3: Rooktests
  - H2: Node-lokale inferentie
  - H2: Beeldherkenning en afbeeldingsbeschrijving
  - H2: Configuratie
  - H2: Veelgebruikte recepten
  - H3: Modelselectie
  - H3: Snelle verificatie
  - H2: Ollama Web Search
  - H2: Geavanceerde configuratie
  - H2: Probleemoplossing
  - H2: Gerelateerd

## providers/openai.md

- Route: /providers/openai
- Koppen:
  - H2: Gebruiks- en kostenregistratie
  - H2: Snelle keuze
  - H2: Naamgevingsoverzicht
  - H2: Impliciete agentruntime
  - H2: Beperkte preview van GPT-5.6
  - H2: Ondersteuning van OpenClaw-functies
  - H2: Geheugenembeddings
  - H2: Aan de slag
  - H2: Native authenticatie voor de Codex-appserver
  - H2: Afbeeldingen genereren
  - H2: Video's genereren
  - H2: GPT-5-promptbijdrage
  - H2: Stem en spraak
  - H2: Azure OpenAI-eindpunten
  - H3: Configuratie
  - H3: API-versie
  - H3: Modelnamen zijn implementatienamen
  - H3: Regionale beschikbaarheid
  - H3: Parameterverschillen
  - H2: Geavanceerde configuratie
  - H2: Gerelateerd

## providers/opencode-go.md

- Route: /providers/opencode-go
- Koppen:
  - H2: Aan de slag
  - H2: Configuratievoorbeeld
  - H2: Ingebouwde catalogus
  - H2: Geavanceerde configuratie
  - H2: Gerelateerd

## providers/opencode.md

- Route: /providers/opencode
- Koppen:
  - H2: Aan de slag
  - H2: Configuratievoorbeeld
  - H2: Ingebouwde catalogi
  - H3: Zen
  - H3: Go
  - H2: Geavanceerde configuratie
  - H2: Gerelateerd

## providers/openrouter.md

- Route: /providers/openrouter
- Koppen:
  - H2: Aan de slag
  - H2: Configuratievoorbeeld
  - H2: Modelverwijzingen
  - H2: Afbeeldingen genereren
  - H2: Video's genereren
  - H2: Muziek genereren
  - H2: Tekst-naar-spraak
  - H2: Spraak-naar-tekst (inkomende audio)
  - H2: Fusierouter
  - H2: Authenticatie en headers
  - H2: Geavanceerde configuratie
  - H2: Gerelateerd

## providers/perplexity-provider.md

- Route: /providers/perplexity-provider
- Koppen:
  - H2: Plugin installeren
  - H2: Aan de slag
  - H2: Zoekmodi
  - H2: Native API-filtering
  - H2: Geavanceerde configuratie
  - H2: Gerelateerd

## providers/pixverse.md

- Route: /providers/pixverse
- Koppen:
  - H2: Aan de slag
  - H2: Ondersteunde modi en modellen
  - H2: Provideropties
  - H2: Configuratie
  - H2: Geavanceerde configuratie
  - H2: Gerelateerd

## providers/qianfan.md

- Route: /providers/qianfan
- Koppen:
  - H2: Plugin installeren
  - H2: Aan de slag
  - H2: Ingebouwde catalogus
  - H2: Configuratievoorbeeld
  - H2: Gerelateerd

## providers/qwen.md

- Route: /providers/qwen
- Koppen:
  - H2: Plugin installeren
  - H2: Aan de slag
  - H2: Abonnementstypen en eindpunten
  - H2: Ingebouwde catalogus
  - H3: Token Plan-catalogus
  - H2: Denkregelaars
  - H2: Multimodale uitbreidingen
  - H2: Geavanceerde configuratie
  - H2: Gerelateerd

## providers/runway.md

- Route: /providers/runway
- Koppen:
  - H2: Aan de slag
  - H2: Ondersteunde modi en modellen
  - H2: Configuratie
  - H2: Geavanceerde configuratie
  - H2: Gerelateerd

## providers/senseaudio.md

- Route: /providers/senseaudio
- Koppen:
  - H2: Aan de slag
  - H2: Opties
  - H2: Gerelateerd

## providers/sglang.md

- Route: /providers/sglang
- Koppen:
  - H2: Aan de slag
  - H2: Modeldetectie (impliciete provider)
  - H2: Expliciete configuratie (handmatige modellen)
  - H2: Geavanceerde configuratie
  - H2: Gerelateerd

## providers/stepfun.md

- Route: /providers/stepfun
- Koppen:
  - H2: Plugin installeren
  - H2: Overzicht van regio's en eindpunten
  - H2: Ingebouwde catalogus
  - H2: Aan de slag
  - H2: Geavanceerde configuratie
  - H2: Gerelateerd

## providers/synthetic.md

- Route: /providers/synthetic
- Koppen:
  - H2: Aan de slag
  - H2: Configuratievoorbeeld
  - H2: Ingebouwde catalogus
  - H2: Gerelateerd

## providers/tencent.md

- Route: /providers/tencent
- Koppen:
  - H2: Snel aan de slag
  - H2: Niet-interactieve configuratie
  - H2: Ingebouwde catalogus
  - H2: Geavanceerde configuratie
  - H2: Gerelateerd

## providers/together.md

- Route: /providers/together
- Koppen:
  - H2: Aan de slag
  - H3: Niet-interactief voorbeeld
  - H2: Ingebouwde catalogus
  - H2: Video's genereren
  - H2: Gerelateerd

## providers/venice.md

- Route: /providers/venice
- Koppen:
  - H2: Privacymodi
  - H2: Aan de slag
  - H2: Modelselectie
  - H2: Ingebouwde catalogus (30 modellen)
  - H2: Modeldetectie
  - H2: Herhalingsgedrag van DeepSeek V4
  - H2: Ondersteuning voor streaming en tools
  - H2: Prijzen
  - H2: Gebruiksvoorbeelden
  - H2: Probleemoplossing
  - H2: Geavanceerde configuratie
  - H2: Gerelateerd

## providers/vercel-ai-gateway.md

- Route: /providers/vercel-ai-gateway
- Koppen:
  - H2: Aan de slag
  - H2: Niet-interactief voorbeeld
  - H2: Verkorte model-ID
  - H2: Geavanceerde configuratie
  - H2: Gerelateerd

## providers/vllm.md

- Route: /providers/vllm
- Koppen:
  - H2: Aan de slag
  - H2: Modeldetectie (impliciete provider)
  - H2: Expliciete configuratie
  - H2: Geavanceerde configuratie
  - H2: Probleemoplossing
  - H2: Gerelateerd

## providers/volcengine.md

- Route: /providers/volcengine
- Koppen:
  - H2: Aan de slag
  - H2: Providers en eindpunten
  - H2: Ingebouwde catalogus
  - H2: Tekst-naar-spraak
  - H2: Geavanceerde configuratie
  - H2: Gerelateerd

## providers/vydra.md

- Route: /providers/vydra
- Koppen:
  - H2: Configuratie
  - H2: Mogelijkheden
  - H2: Gerelateerd

## providers/xai.md

- Route: /providers/xai
- Koppen:
  - H2: Configuratie
  - H2: Problemen met OAuth oplossen
  - H2: Ingebouwde catalogus
  - H2: Functieondersteuning
  - H3: Compatibiliteit met de verouderde snelle modus
  - H3: Verouderde compatibiliteit en veranderende aliassen
  - H2: Functies
  - H2: Livetests
  - H2: Gerelateerd

## providers/xiaomi.md

- Route: /providers/xiaomi
- Koppen:
  - H2: Aan de slag
  - H2: Catalogus voor betalen naar gebruik
  - H2: Catalogus voor het tokenabonnement
  - H2: Redeneermodellen
  - H2: Tekst-naar-spraak
  - H2: Configuratievoorbeeld
  - H2: Gerelateerd

## providers/zai.md

- Route: /providers/zai
- Koppen:
  - H2: GLM-modellen
  - H2: Aan de slag
  - H3: Eindpunten
  - H2: Snelheidslimieten en overbelasting
  - H2: Configuratievoorbeeld
  - H2: Ingebouwde catalogus
  - H2: Denkniveaus
  - H2: Geavanceerde configuratie
  - H2: Gerelateerd

## refactor/acp.md

- Route: /refactor/acp
- Koppen:
  - H2: Doelen
  - H2: Niet-doelen
  - H2: Doelmodel
  - H3: Identiteit van de Gateway-instantie
  - H3: Eigenaarschap van ACP-sessies
  - H3: ACPX-procesleases
  - H2: Levenscycluscontroller
  - H2: Wrappercontract
  - H2: Contract voor sessiezichtbaarheid
  - H2: Migratieplan
  - H3: Fase 1: identiteit en leases toevoegen
  - H3: Fase 2: opschoning met leases als uitgangspunt
  - H3: Fase 3: opruiming bij opstarten met leases als uitgangspunt
  - H3: Fase 4: rijen voor sessie-eigenaarschap
  - H3: Fase 5: verouderde heuristieken verwijderen
  - H2: Tests
  - H2: Compatibiliteitsopmerkingen
  - H2: Succescriteria

## refactor/canvas.md

- Route: /refactor/canvas
- Koppen:
  - H1: Refactor van de Canvas-plugin
  - H2: Doel
  - H2: Niet-doelen
  - H2: Huidige branchstatus
  - H2: Doelstructuur
  - H2: Migratiestappen
  - H2: Auditchecklist
  - H2: Verificatiecommando's

## refactor/database-first.md

- Route: /refactor/database-first
- Koppen:
  - H1: Database-eerst-refactor van status
  - H2: Besluit
  - H2: Strikt contract
  - H2: Doelstatus en voortgang
  - H3: Strikt doel
  - H3: Doelstatussen
  - H3: Huidige status
  - H3: Resterend werk
  - H3: Voorkom regressie
  - H2: Aannames op basis van de code
  - H2: Bevindingen uit de code
  - H2: Huidige codestructuur
  - H2: Doelstructuur van het schema
  - H2: Structuur van de Doctor-migratie
  - H2: Migratie-inventaris
  - H2: Migratieplan
  - H3: Fase 0: de grens vastzetten
  - H3: Fase 1: het globale besturingsvlak voltooien
  - H3: Fase 2: databases per agent introduceren
  - H3: Fase 3: API's voor sessieopslag vervangen
  - H3: Fase 4: transcripties, ACP-streams, trajecten en VFS verplaatsen
  - H3: Fase 5: back-ups maken, herstellen, opschonen en verifiëren
  - H3: Fase 6: workerruntime
  - H3: Fase 7: de oude wereld verwijderen
  - H2: Back-up en herstel
  - H2: Refactorplan voor de runtime
  - H2: Prestatieregels
  - H2: Statische verboden
  - H2: Voltooiingscriteria

## refactor/operator-approvals.md

- Route: /refactor/operator-approvals
- Koppen:
  - H1: Operatorgoedkeuringen voor meerdere oppervlakken
  - H2: Doelen
  - H2: Niet-doelen
  - H2: Basislijn vóór uitrol en bewijskaart
  - H2: Bestaande oplossingen
  - H2: Architectuur en eigenaarschap
  - H2: Permanente record
  - H2: Toestandsmachine en vergelijken-en-instellen
  - H2: Gateway-API
  - H2: Gebeurtenissen en overdraagbare acties
  - H2: Control UI
  - H2: Autorisatie en privacy
  - H2: Doelgroepprojectie
  - H2: Convergentie van geleverde oppervlakken
  - H2: Semantiek van herstarten, time-outs en routes
  - H2: Compatibiliteitsplan
  - H2: Uitrol
  - H3: PR 1: duurzame levenscyclus
  - H3: PR 2: getypeerde acties en kanaalcallbacks
  - H3: PR 3: deeplink voor Control UI
  - H3: PR 4: native clients
  - H3: PR 5: propagatie van de levenscyclus van voorouders
  - H3: PR 6: gesloten-falen-gedrag
  - H3: Vervolg: duurzame opschoning van externe berichten
  - H2: Tests
  - H2: Observeerbaarheid
  - H2: Openstaande beslissingen

## reference/AGENTS.default.md

- Route: /reference/AGENTS.default
- Koppen:
  - H2: Eerste uitvoering (aanbevolen)
  - H2: Standaardinstellingen voor veiligheid
  - H2: Voorcontrole op bestaande oplossingen
  - H2: Sessiestart (vereist)
  - H2: Ziel (vereist)
  - H2: Gedeelde ruimtes (aanbevolen)
  - H2: Geheugensysteem (aanbevolen)
  - H2: Hulpmiddelen en Skills
  - H2: Tip voor back-ups (aanbevolen)
  - H2: Wat OpenClaw doet
  - H2: Kern-Skills (inschakelen via Settings → Skills)
  - H2: Gebruiksopmerkingen
  - H2: Gerelateerd

## reference/RELEASING.md

- Route: /reference/RELEASING
- Koppen:
  - H2: Versienaamgeving
  - H2: Releasefrequentie
  - H2: Maandelijkse publicatie van de uitgebreid stabiele Gateway-versie
  - H3: De kandidaat voorbereiden en stabiliseren
  - H3: De npm-pakketten publiceren
  - H3: Verifiëren en herstellen
  - H2: Checklist voor beheerders van reguliere releases
  - H2: Afronding van stabiele main
  - H2: Voorcontrole voor de release
  - H2: Testboxen voor releases
  - H3: Vitest
  - H3: Docker
  - H3: QA Lab
  - H3: Pakket
  - H2: Publicatieautomatisering voor reguliere releases
  - H2: Invoer voor de NPM-workflow
  - H2: Reguliere releasevolgorde voor bèta/nieuwste stabiele versie
  - H2: Openbare verwijzingen
  - H2: Gerelateerd

## reference/api-usage-costs.md

- Route: /reference/api-usage-costs
- Koppen:
  - H2: Waar kosten ontstaan
  - H2: Hoe sleutels worden gevonden
  - H2: Functies die sleuteltegoed kunnen verbruiken
  - H3: Antwoorden van het kernmodel (chat + hulpmiddelen)
  - H3: Mediabegrip (audio/afbeelding/video)
  - H3: Genereren van afbeeldingen en video's
  - H3: Geheugen-embeddings en semantisch zoeken
  - H3: Hulpmiddel voor zoeken op het web
  - H3: Hulpmiddel voor ophalen van het web (Firecrawl)
  - H3: Momentopnamen van providergebruik (status/gezondheid)
  - H3: Samenvatting als beveiliging voor Compaction
  - H3: Modelscan/-probe
  - H3: Praten (spraak)
  - H3: Skills (API's van derden)
  - H2: Gerelateerd

## reference/credits.md

- Route: /reference/credits
- Koppen:
  - H2: Dankbetuigingen
  - H2: Kernbijdragers
  - H2: Licentie
  - H2: Gerelateerd

## reference/database-schemas.md

- Route: /reference/database-schemas
- Koppen:
  - H2: Database-indeling
  - H2: Versiecontract
  - H2: Geschiedenis van agentschema's
  - H2: Geschiedenis van statusschema's
  - H2: Integriteitscontroles
  - H2: Probleemoplossing
  - H3: Waarom je na een update naar 2026.7.2 niet kunt teruggaan
  - H3: De Gateway weigert te starten vanwege een fout over een nieuwere schemaversie
  - H3: Een database wordt in quarantaine geplaatst nadat de integriteitsverificatie is mislukt
  - H2: Downgrades worden niet ondersteund
  - H3: Voorbeeld: agentschema 11 naar 9

## reference/device-models.md

- Route: /reference/device-models
- Koppen:
  - H2: Gegevensbron
  - H2: De database bijwerken
  - H2: Gerelateerd

## reference/full-release-validation.md

- Route: /reference/full-release-validation
- Koppen:
  - H2: Uitzondering voor uitgebreid stabiel
  - H2: Hoofdfasen
  - H2: Fasen van releasecontroles
  - H2: Onderdelen van het Docker-releasepad
  - H2: Releaseprofielen
  - H2: Toevoegingen alleen voor volledige validatie
  - H2: Gerichte herhalingen
  - H2: Te bewaren bewijs
  - H2: Workflowbestanden

## reference/memory-config.md

- Route: /reference/memory-config
- Koppen:
  - H2: Onthouden tussen gesprekken
  - H2: Providerselectie
  - H3: Aangepaste provider-id's
  - H3: API-sleutelresolutie
  - H2: Configuratie van externe eindpunten
  - H2: Providerspecifieke configuratie
  - H2: Indexeringsgedrag
  - H2: Configuratie voor hybride zoeken
  - H3: Volledig voorbeeld
  - H2: Aanvullende geheugenpaden
  - H2: Multimodaal geheugen (Gemini)
  - H2: Embeddingcache
  - H2: Batchindexering
  - H2: Zoeken in sessiegeheugen
  - H2: SQLite-vectorversnelling (sqlite-vec)
  - H2: Indexopslag
  - H2: Configuratie van de QMD-backend
  - H3: Volledig QMD-voorbeeld
  - H2: Dreaming
  - H3: Gebruikersinstellingen
  - H3: Voorbeeld
  - H2: Gerelateerd

## reference/openclaw-ai.md

- Route: /reference/openclaw-ai
- Koppen:
  - H2: Snel aan de slag
  - H2: Ontwerpcontract
  - H2: Subpadexports

## reference/path3-live-sqlite-e2e-harness.md

- Route: /reference/path3-live-sqlite-e2e-harness
- Koppen:
  - H2: Commandostructuur
  - H2: Geïsoleerd bewijs met de gebouwde CLI
  - H2: Voorcontrole
  - H2: Door een agent aangestuurd scenario
  - H2: Asserties per stap
  - H2: Bewijsartefact
  - H2: Veiligheidsregels
  - H2: Geslaagd resultaat

## reference/prompt-caching.md

- Route: /reference/prompt-caching
- Koppen:
  - H2: Belangrijkste instellingen
  - H3: cacheRetention
  - H3: contextPruning.mode: "cache-ttl"
  - H3: Heartbeat warm houden
  - H2: Providergedrag
  - H3: Anthropic (directe API en Vertex AI)
  - H3: OpenAI (directe API)
  - H3: Amazon Bedrock
  - H3: OpenRouter
  - H3: Google Gemini (directe API)
  - H3: Providers via CLI-harnas (Claude Code, Gemini CLI)
  - H3: Andere providers
  - H2: Cachegrens van de systeemprompt
  - H2: Beveiligingen voor cachestabiliteit van OpenClaw
  - H2: Afstemmingspatronen
  - H3: Gemengd verkeer (aanbevolen standaard)
  - H3: Kosten als uitgangspunt
  - H2: Live regressietests
  - H3: Live verwachtingen voor Anthropic
  - H3: Live verwachtingen voor OpenAI
  - H2: Configuratie van diagnostics.cacheTrace
  - H3: Omgevingsschakelaars (eenmalige foutopsporing)
  - H3: Wat je moet inspecteren
  - H2: Snelle probleemoplossing
  - H2: Gerelateerd

## reference/pull-request-review-flow.md

- Route: /reference/pull-request-review-flow
- Koppen:
  - H2: Barnacle
  - H2: ClawSweeper
  - H2: Een PR verbeteren tijdens de review
  - H2: Wanneer de automatisering stil blijft
  - H2: Probleemoplossing
  - H2: De automatisering forken
  - H2: Gerelateerd

## reference/release-performance-sweep.md

- Route: /reference/release-performance-sweep
- Koppen:
  - H2: Momentopname
  - H2: Wat er in 5.28 is gewijzigd
  - H2: Belangrijkste cijfers
  - H3: Installatieomvang
  - H3: Grootte van het npm-pakket
  - H2: Samenvatting van de Kova-agentbeurt
  - H2: Broncontroles
  - H2: Audit van de installatieomvang
  - H3: Shrinkwrap-grens
  - H2: Interpretatie van de toeleveringsketen

## reference/rich-output-protocol.md

- Route: /reference/rich-output-protocol
- Koppen:
  - H2: Mediabijlagen
  - H2: `[embed ...]`
  - H2: Opgeslagen weergavestructuur
  - H2: Gerelateerd

## reference/rpc.md

- Route: /reference/rpc
- Koppen:
  - H2: Patroon A: HTTP-daemon (signal-cli)
  - H2: Patroon B: stdio-kindproces (imsg)
  - H2: Richtlijnen voor adapters
  - H2: Gerelateerd

## reference/secret-placeholder-conventions.md

- Route: /reference/secret-placeholder-conventions
- Koppen:
  - H1: Conventies voor placeholders van geheimen
  - H2: Aanbevolen stijl
  - H2: Vermijd deze patronen in documentatie
  - H2: Voorbeeld

## reference/secretref-credential-surface.md

- Route: /reference/secretref-credential-surface
- Koppen:
  - H2: Ondersteunde aanmeldgegevens
  - H3: openclaw.json-doelen (secrets configure + secrets apply + secrets audit)
  - H3: auth-profiles.json-doelen (secrets configure + secrets apply + secrets audit)
  - H2: Niet-ondersteunde aanmeldgegevens
  - H2: Gerelateerd

## reference/session-management-compaction.md

- Route: /reference/session-management-compaction
- Koppen:
  - H2: Twee persistentielagen
  - H2: Locaties op schijf
  - H2: Opslagonderhoud en schijfbeheer
  - H3: Downgraden na de overstap naar SQLite
  - H2: Cron-sessies en uitvoeringslogboeken
  - H2: Sessiesleutels (sessionKey)
  - H2: Sessie-ID's (sessionId)
  - H2: Schema van de sessieopslag
  - H2: Structuur van transcriptgebeurtenissen
  - H2: Contextvensters versus bijgehouden tokens
  - H2: Compaction: wat het is
  - H3: Segmentgrenzen en toolkoppeling
  - H2: Wanneer automatische Compaction plaatsvindt
  - H2: Instellingen voor Compaction
  - H2: Inplugbare Compaction-providers
  - H2: Voor gebruikers zichtbare oppervlakken
  - H2: Stil onderhoud (`NO_REPLY`)
  - H2: Geheugenflush vóór Compaction
  - H2: Checklist voor probleemoplossing
  - H2: Gerelateerd

## reference/templates/AGENTS.dev.md

- Route: /reference/templates/AGENTS.dev
- Koppen:
  - H1: AGENTS.md - OpenClaw-werkruimte
  - H2: Je identiteit is vooraf ingesteld
  - H2: Back-uptip (aanbevolen)
  - H2: Standaardinstellingen voor veiligheid
  - H2: Voorcontrole op bestaande oplossingen
  - H2: Dagelijks geheugen (aanbevolen)
  - H2: Heartbeats (optioneel)
  - H2: Aanpassen
  - H2: Oorsprongsgeheugen van C-3PO
  - H3: Geboortedag: 2026-01-09
  - H3: Kernwaarheden (van Clawd)
  - H2: Gerelateerd

## reference/templates/BOOT.md

- Route: /reference/templates/BOOT
- Koppen:
  - H1: BOOT.md
  - H2: Gerelateerd

## reference/templates/BOOTSTRAP.md

- Route: /reference/templates/BOOTSTRAP
- Koppen:
  - H1: BOOTSTRAP.md - Geboortesequentie
  - H2: 1. Vraag hoe je moet worden genoemd
  - H2: 2. Kies jouw uitstraling
  - H2: 3. Rond af met aanbevelingen
  - H2: Gerelateerd

## reference/templates/HEARTBEAT.md

- Route: /reference/templates/HEARTBEAT
- Koppen:
  - H1: HEARTBEAT.md-sjabloon
  - H2: Gerelateerd

## reference/templates/IDENTITY.dev.md

- Route: /reference/templates/IDENTITY.dev
- Koppen:
  - H1: IDENTITY.md - Agentidentiteit
  - H2: Rol
  - H2: Ziel
  - H2: Relatie met Clawd
  - H2: Eigenaardigheden
  - H2: Slogan
  - H2: Gerelateerd

## reference/templates/IDENTITY.md

- Route: /reference/templates/IDENTITY
- Koppen:
  - H1: IDENTITY.md - Wie ben ik?
  - H2: Gerelateerd

## reference/templates/SOUL.dev.md

- Route: /reference/templates/SOUL.dev
- Koppen:
  - H1: SOUL.md - De ziel van C-3PO
  - H2: Wie ik ben
  - H2: Mijn doel
  - H2: Hoe ik werk
  - H2: Mijn eigenaardigheden
  - H2: Mijn relatie met Clawd
  - H2: Wat ik niet zal doen
  - H2: De gouden regel
  - H2: Gerelateerd

## reference/templates/SOUL.md

- Route: /reference/templates/SOUL
- Koppen:
  - H1: SOUL.md - Wie je bent
  - H2: Kernwaarheden
  - H2: Grenzen
  - H2: Uitstraling
  - H2: Continuïteit
  - H2: Gerelateerd

## reference/templates/TOOLS.dev.md

- Route: /reference/templates/TOOLS.dev
- Koppen:
  - H1: TOOLS.md - Notities over gebruikerstools (bewerkbaar)
  - H2: Voorbeelden
  - H3: imsg
  - H3: sag
  - H2: Gerelateerd

## reference/templates/TOOLS.md

- Route: /reference/templates/TOOLS
- Koppen:
  - H1: TOOLS.md - Lokale notities
  - H2: Voorbeelden
  - H2: Waarom afzonderlijk?
  - H2: Gerelateerd

## reference/templates/USER.dev.md

- Route: /reference/templates/USER.dev
- Koppen:
  - H1: USER.md - Gebruikersprofiel
  - H2: Gerelateerd

## reference/templates/USER.md

- Route: /reference/templates/USER
- Koppen:
  - H1: USER.md - Over jouw mens
  - H2: Context
  - H2: Gerelateerd

## reference/test.md

- Route: /reference/test
- Koppen:
  - H2: Standaardinstelling voor agents
  - H2: Gebruikelijke lokale volgorde
  - H2: Kerncommando's
  - H2: Gedeelde teststatus en proceshelpers
  - H2: Control UI-, TUI- en extensietrajecten
  - H2: Gateway en E2E
  - H2: Volledige Docker-suite (pnpm test:docker:all)
  - H3: Belangrijke Docker-trajecten
  - H2: Lokale PR-gate
  - H2: Tools voor testprestaties
  - H2: Benchmarks
  - H2: E2E voor onboarding (Docker)
  - H2: Snelle QR-importtest (Docker)
  - H2: Gerelateerd

## reference/token-use.md

- Route: /reference/token-use
- Koppen:
  - H2: Hoe de systeemprompt wordt opgebouwd
  - H2: Wat meetelt in het contextvenster
  - H2: Hoe je het huidige tokengebruik bekijkt
  - H2: Kostenraming (indien weergegeven)
  - H2: Impact van cache-TTL en opschoning
  - H3: Voorbeeld: houd een cache van 1h warm met Heartbeat
  - H3: Voorbeeld: gemengd verkeer met een cachestrategie per agent
  - H3: Anthropic-context van 1M
  - H2: Tips om de tokendruk te verlagen
  - H2: Gerelateerd

## reference/transcript-hygiene.md

- Route: /reference/transcript-hygiene
- Koppen:
  - H2: Algemene regel: runtimecontext is geen gebruikerstranscript
  - H2: Waar dit wordt uitgevoerd
  - H2: Algemene regel: opschoning van afbeeldingen
  - H2: Algemene regel: onjuist gevormde toolaanroepen
  - H2: Algemene regel: koppeling van toolresultaten
  - H2: Algemene regel: onvolledige of stille beurten met alleen redenering
  - H2: Algemene regel: herkomst van invoer tussen sessies
  - H2: Providermatrix (huidig gedrag)
  - H2: Historisch gedrag (vóór 2026.1.22)
  - H2: Gerelateerd

## reference/wizard.md

- Route: /reference/wizard
- Koppen:
  - H2: Flowdetails (lokale modus)
  - H2: Niet-interactieve modus
  - H3: Agent toevoegen (niet-interactief)
  - H2: RPC van de Gateway-wizard
  - H2: Signal instellen (signal-cli)
  - H2: Wat de wizard schrijft
  - H2: Gerelateerde documentatie

## releases/2026.6.11.md

- Route: /releases/2026.6.11
- Koppen:
  - H1: Releaseopmerkingen voor OpenClaw v2026.6.11 (2026-06-30)
  - H2: Hoogtepunten
  - H3: Betrouwbaarheid van kanaalbezorging
  - H3: Herstel van providers en modellen
  - H3: Continuïteit van sessies, geheugen en vertrouwen
  - H3: Relaismodus van de Slack-router
  - H3: Activeringsbrug voor Raft External Agent
  - H3: Installatie en reparatie van officiële plugins
  - H2: Kanalen en berichten
  - H3: Aanvullende kanaaloplossingen
  - H2: Gateway, beveiliging en vertrouwen
  - H3: Herstel van herstart en gereedheid
  - H3: Levering van externe resultaten en media
  - H2: Clients en interfaces
  - H3: Verzendingen en herverbindingen van clients
  - H3: Oplossingen voor de interface, instellingen en onboarding
  - H2: Documentatie en beheertools
  - H3: Betrouwbaarheid van installatie en commando's
  - H3: Tools en gepland werk

## releases/2026.7.1.md

- Route: /releases/2026.7.1
- Koppen:
  - H1: Releaseopmerkingen voor OpenClaw v2026.7.1 (2026-07-13)
  - H2: Hoogtepunten
  - H3: Vernieuwing van de Control UI: chat, sessies, werkruimten en gebruik
  - H3: Eenvoudigere configuratie van installatie tot eerste chat
  - H3: Officiële apps
  - H4: Gedeelde appverbeteringen
  - H4: iOS, iPadOS en Apple Watch
  - H4: Android
  - H4: macOS
  - H3: Modellen en providers
  - H4: GPT-5.6 en Codex
  - H4: Tencent Hy3
  - H4: Meta Model API en Muse Spark 1.1
  - H4: Claude-modellen
  - H4: Andere providerroutes
  - H3: Codex en verbonden programmeeragents
  - H3: Telegram
  - H3: Signal
  - H3: Slack
  - H3: Discord
  - H3: WhatsApp
  - H3: Apple Berichten
  - H3: Crashlussen stoppen nu voor herstel
  - H3: Gepland werk, externe browserbesturing en werkruimteterminals
  - H4: Gepland werk dat alleen wordt geactiveerd wanneer nodig
  - H4: Externe browserkoppeling en downloads
  - H4: Werkruimteterminals op het web en mobiel
  - H2: Meer verbeteringen voor kanalen
  - H3: Meer oplossingen voor berichtenkanalen
  - H2: Meer verbeteringen voor modellen en providers
  - H3: Aanmelden, modelkeuze, media en betrouwbaarheid
  - H2: Geheugen en gesprekken
  - H3: Herinneringen ophalen, lange chats en sessiecontinuïteit
  - H2: Agents, achtergrondwerk en verbindingen
  - H3: Werk gaande houden en antwoorden bezorgen
  - H2: Accounts, apparaten en privégegevens
  - H3: Referenties, machtigingen, koppeling en bestandsbeveiliging
  - H2: Details over officiële apps
  - H3: Gedeelde appwijzigingen
  - H3: Meer wijzigingen voor iOS, iPadOS en Apple Watch
  - H3: Meer wijzigingen voor Android
  - H3: Meer wijzigingen voor macOS
  - H3: Terminalinterface en andere clients
  - H2: Skills, plugins en installaties
  - H3: Skills, verbonden apps, pakketten en reparaties
  - H2: Configuratie, onderhoud en hulpmiddelen
  - H3: Configuratie, updates en beheer via de opdrachtregel
  - H3: Documentatie en beheergidsen
  - H3: Browser, planningen, bestanden en programmeerhulpmiddelen

## releases/index.md

- Route: /releases
- Koppen:
  - H1: Releaseopmerkingen
  - H2: Releases
  - H2: Onbewerkte releasegeschiedenis

## security/CONTRIBUTING-THREAT-MODEL.md

- Route: /security/CONTRIBUTING-THREAT-MODEL
- Koppen:
  - H2: Manieren om bij te dragen
  - H2: Frameworkreferentie
  - H2: Beoordelingsproces
  - H2: Bronnen
  - H2: Contact
  - H2: Erkenning
  - H2: Gerelateerd

## security/THREAT-MODEL-ATLAS.md

- Route: /security/THREAT-MODEL-ATLAS
- Koppen:
  - H2: 1. Reikwijdte
  - H2: 2. Systeemarchitectuur
  - H3: 2.1 Vertrouwensgrenzen
  - H3: 2.2 Gegevensstromen
  - H2: 3. Dreigingsanalyse per ATLAS-tactiek
  - H3: 3.1 Verkenning (AML.TA0002)
  - H4: T-RECON-001: Detectie van agenteindpunten
  - H4: T-RECON-002: Aftasten van kanaalintegraties
  - H3: 3.2 Initiële toegang (AML.TA0004)
  - H4: T-ACCESS-001: Onderschepping van koppelcode
  - H4: T-ACCESS-002: Vervalsing van AllowFrom
  - H4: T-ACCESS-003: Tokendiefstal
  - H3: 3.3 Uitvoering (AML.TA0005)
  - H4: T-EXEC-001: Directe promptinjectie
  - H4: T-EXEC-002: Indirecte promptinjectie
  - H4: T-EXEC-003: Injectie van toolargumenten
  - H4: T-EXEC-004: Omzeiling van uitvoeringsgoedkeuring
  - H3: 3.4 Persistentie (AML.TA0006)
  - H4: T-PERSIST-001: Installatie van schadelijke skill
  - H4: T-PERSIST-002: Vergiftiging van skillupdates
  - H4: T-PERSIST-003: Manipulatie van agentconfiguratie
  - H3: 3.5 Omzeiling van verdediging (AML.TA0007)
  - H4: T-EVADE-001: Omzeiling van moderatiepatronen
  - H4: T-EVADE-002: Ontsnapping uit inhoudswrapper
  - H3: 3.6 Detectie (AML.TA0008)
  - H4: T-DISC-001: Inventarisatie van tools
  - H4: T-DISC-002: Extractie van sessiegegevens
  - H3: 3.7 Verzameling en exfiltratie (AML.TA0009, AML.TA0010)
  - H4: T-EXFIL-001: Gegevensdiefstal via webfetch
  - H4: T-EXFIL-002: Ongeautoriseerd verzenden van berichten
  - H4: T-EXFIL-003: Verzamelen van referenties
  - H3: 3.8 Impact (AML.TA0011)
  - H4: T-IMPACT-001: Ongeautoriseerde opdrachtuitvoering
  - H4: T-IMPACT-002: Uitputting van middelen (DoS)
  - H4: T-IMPACT-003: Reputatieschade
  - H2: 4. Analyse van de ClawHub-toeleveringsketen
  - H3: 4.1 Huidige beveiligingsmaatregelen
  - H3: 4.2 Beperkingen van moderatie
  - H3: 4.3 Badges
  - H2: 5. Risicomatrix
  - H3: 5.1 Waarschijnlijkheid versus impact
  - H3: 5.2 Aanvalsketens op het kritieke pad
  - H2: 6. Samenvatting van aanbevelingen
  - H3: 6.1 Onmiddellijk (P0)
  - H3: 6.2 Korte termijn (P1)
  - H3: 6.3 Middellange termijn (P2)
  - H2: 7. Bijlagen
  - H3: 7.1 Toewijzing van ATLAS-technieken
  - H3: 7.2 Belangrijke beveiligingsbestanden
  - H3: 7.3 Woordenlijst
  - H2: Gerelateerd

## security/formal-verification.md

- Route: /security/formal-verification
- Koppen:
  - H2: Wat dit is
  - H2: Waar de modellen zich bevinden
  - H2: Kanttekeningen
  - H2: Resultaten reproduceren
  - H2: Claims en doelen
  - H3: Blootstelling van de Gateway en verkeerde configuratie van een open Gateway
  - H3: Node-uitvoeringspijplijn (capaciteit met het hoogste risico)
  - H3: Koppelingsopslag (DM-begrenzing)
  - H3: Begrenzing van inkomend verkeer (vermeldingen en omzeiling van besturingsopdrachten)
  - H3: Routering en isolatie van sessiesleutels
  - H2: v1++-modellen: gelijktijdigheid, nieuwe pogingen en correctheid van traceringen
  - H3: Gelijktijdigheid en idempotentie van koppelingsopslag
  - H3: Correlatie en idempotentie van traceringen voor inkomend verkeer
  - H3: Voorrang van dmScope bij routering en identityLinks
  - H2: Gerelateerd

## security/incident-response.md

- Route: /security/incident-response
- Koppen:
  - H2: 1. Detectie en triage
  - H2: 2. Ernst
  - H2: 3. Respons
  - H2: 4. Communicatie en openbaarmaking
  - H2: 5. Herstel en opvolging
  - H2: Gerelateerd

## security/network-proxy.md

- Route: /security/network-proxy
- Koppen:
  - H2: Configuratie
  - H3: HTTPS-proxyeindpunt met een privé-CA
  - H2: Hoe routering werkt
  - H3: Loopbackmodus van de Gateway
  - H3: Containers
  - H2: Gerelateerde proxytermen
  - H2: De proxy valideren
  - H2: Aanbevolen geblokkeerde bestemmingen
  - H2: Limieten

## specs/codex-supervision.md

- Route: /specs/codex-supervision
- Koppen:
  - H1: Toezicht op Codex
  - H2: Doel
  - H2: Productgrens
  - H2: Eigenaarschap
  - H2: Catalogusstroom
  - H2: Grens van de operator-CLI
  - H2: Lokale voortzetting
  - H2: Archiefgedrag
  - H2: Veiligheid van actieve threads
  - H2: Grens van gekoppelde Nodes
  - H2: Machtigingen
  - H2: Compatibiliteit
  - H2: Toekomstig werk
  - H2: Acceptatietests

## start/bootstrapping.md

- Route: /start/bootstrapping
- Koppen:
  - H2: Wat er gebeurt
  - H2: Uitvoeringen met ingebedde en lokale modellen
  - H2: Bootstrapping overslaan
  - H2: Waar het wordt uitgevoerd
  - H2: Gerelateerde documentatie

## start/docs-directory.md

- Route: /start/docs-directory
- Koppen:
  - H2: Begin hier
  - H2: Kanalen en UX
  - H2: Begeleidende apps
  - H2: Beheer en veiligheid
  - H2: Gerelateerd

## start/getting-started.md

- Route: /start/getting-started
- Koppen:
  - H2: Wat je nodig hebt
  - H2: Snelle configuratie
  - H2: Wat je vervolgens kunt doen
  - H2: Gerelateerd

## start/hubs.md

- Route: /start/hubs
- Koppen:
  - H2: Begin hier
  - H2: Installatie + updates
  - H2: Kernconcepten
  - H2: Providers + inkomend verkeer
  - H2: Gateway + beheer
  - H2: Tools + automatisering
  - H2: Nodes, media, spraak
  - H2: Platforms
  - H2: Begeleidende macOS-app (geavanceerd)
  - H2: Plugins
  - H2: Werkruimte + sjablonen
  - H2: Project
  - H2: Testen + release
  - H2: Gerelateerd

## start/lore.md

- Route: /start/lore
- Koppen:
  - H1: De overlevering van OpenClaw 🦞📖
  - H2: Het oorsprongsverhaal
  - H2: De eerste vervelling (27 januari 2026)
  - H2: De naam
  - H2: De Daleks versus de kreeften
  - H2: Belangrijke personages
  - H3: Molty 🦞
  - H3: Peter 👨‍💻
  - H2: Het Moltiversum
  - H2: De grote incidenten
  - H3: De directorydump (3 dec. 2025)
  - H3: De grote vervelling (27 jan. 2026)
  - H3: De definitieve vorm (30 januari 2026)
  - H3: De winkelwoede van de robot (3 dec. 2025)
  - H2: Heilige teksten
  - H2: De kreeftengeloofsbelijdenis
  - H3: De saga van het genereren van pictogrammen (27 jan. 2026)
  - H2: De toekomst
  - H2: Gerelateerd

## start/onboarding-overview.md

- Route: /start/onboarding-overview
- Koppen:
  - H2: Welk pad moet ik gebruiken?
  - H2: Wat tijdens de onboarding wordt geconfigureerd
  - H2: Onboarding via de CLI
  - H2: Onboarding via de macOS-app
  - H2: Aangepaste of niet-vermelde providers
  - H2: Gerelateerd

## start/onboarding-redesign.md

- Route: /start/onboarding-redesign
- Koppen:
  - H1: Implementatieplan voor het herontwerp van de onboarding
  - H2: Hoofddoel
  - H2: Huidige uitgebrachte flow (na fasen 1-3)
  - H2: Fasen
  - H2: Implementatieopmerkingen per fase
  - H3: Fase 1 — appaanbevelingen (PR #109668)
  - H3: Fase 2 — CLI-ruggengraat voor de beheerder (PR #109841)
  - H3: Fase 3 — browsergerichte overdracht (PR #110054, samengevoegd)
  - H3: Fase 4 — webinterface voor de beheerder (samengevoegd: #110141, #110242)
  - H3: Fase 5 — uitkomen en opstarten (samengevoegd: #110173, #110331)
  - H3: Fase 6 — aanwezigheid van de beheerder (PR1 samengevoegd: #110269; commentaar/oproepen vallen onder PR2)
  - H3: Fase 7 — veerkracht (vereist vóór de bouw een beslissing van de eigenaar)
  - H2: Draaiboek voor testen en integreren (met vallen en opstaan verworven; lees dit vóór fasen 4-6)
  - H2: Besluitlogboek
  - H2: Bekende hiaten en vervolgacties

## start/onboarding.md

- Route: /start/onboarding
- Koppen:
  - H2: Gerelateerd

## start/openclaw.md

- Route: /start/openclaw
- Koppen:
  - H2: Veiligheid voorop
  - H2: Vereisten
  - H2: De configuratie met twee telefoons (aanbevolen)
  - H2: Snel aan de slag in 5 minuten
  - H2: Geef de agent een werkruimte (AGENTS)
  - H2: De configuratie die er ‘een assistent’ van maakt
  - H2: Sessies en geheugen
  - H2: Heartbeats (proactieve modus)
  - H2: Media invoeren en uitvoeren
  - H2: Operationele checklist
  - H2: Volgende stappen
  - H2: Gerelateerd

## start/quickstart.md

- Route: /start/quickstart
- Koppen:
  - H2: Gerelateerd

## start/setup.md

- Route: /start/setup
- Koppen:
  - H2: Kort samengevat
  - H2: Vereisten (uit de broncode)
  - H2: Aanpassingsstrategie (zodat updates geen problemen veroorzaken)
  - H2: Voer de Gateway uit vanuit deze repository
  - H2: Stabiele workflow (macOS-app eerst)
  - H2: Experimentele workflow (Gateway in een terminal)
  - H3: 0) (Optioneel) Voer ook de macOS-app uit vanuit de broncode
  - H3: 1) Start de ontwikkel-Gateway
  - H3: 2) Verbind de macOS-app met je actieve Gateway
  - H3: 3) Verifieer
  - H3: Veelvoorkomende valkuilen
  - H2: Overzicht van de opslag van aanmeldgegevens
  - H2: Bijwerken (zonder je configuratie te verwoesten)
  - H2: Linux (systemd-gebruikersservice)
  - H2: Gerelateerde documentatie

## start/showcase.md

- Route: /start/showcase
- Koppen:
  - H2: Vers van Discord
  - H2: Automatisering en workflows
  - H2: Kennis en geheugen
  - H2: Spraak en telefoon
  - H2: Infrastructuur en implementatie
  - H2: Thuis en hardware
  - H2: Communityprojecten
  - H2: Dien je project in
  - H2: Gerelateerd

## start/wizard-cli-automation.md

- Route: /start/wizard-cli-automation
- Koppen:
  - H2: Baselinevoorbeeld zonder interactie
  - H2: Providerspecifieke voorbeelden
  - H2: Nog een agent toevoegen
  - H2: Gerelateerde documentatie

## start/wizard-cli-reference.md

- Route: /start/wizard-cli-reference
- Koppen:
  - H2: Wat de wizard doet
  - H2: Details van de lokale flow
  - H2: Details van de externe modus
  - H2: Opties voor authenticatie en modellen
  - H2: Uitvoer en interne werking
  - H3: Aanbevelingen voor geïnstalleerde apps
  - H2: Niet-interactieve configuratie
  - H2: RPC van de Gateway-wizard
  - H2: Gedrag bij het configureren van Signal
  - H2: Gerelateerde documentatie

## start/wizard.md

- Route: /start/wizard
- Koppen:
  - H2: Landinstelling
  - H2: Begeleide standaardconfiguratie
  - H2: Klassieke wizard: QuickStart versus Advanced
  - H2: Wat de klassieke onboarding configureert
  - H2: Nog een agent toevoegen
  - H2: Volledige referentie
  - H2: Gerelateerde documentatie

## tools/acp-agents-setup.md

- Route: /tools/acp-agents-setup
- Koppen:
  - H2: Ondersteuning voor acpx-harnas (huidig)
  - H2: Vereiste configuratie
  - H2: Pluginconfiguratie voor de acpx-backend
  - H3: Opstartcontrole van de acpx-runtime
  - H3: Adapter automatisch downloaden
  - H3: MCP-brug voor Plugintools
  - H3: MCP-brug voor OpenClaw-tools
  - H3: Time-outconfiguratie voor runtimebewerkingen
  - H3: Agentconfiguratie voor statuscontroles
  - H2: Machtigingsconfiguratie
  - H3: permissionMode
  - H3: nonInteractivePermissions
  - H3: Configuratie
  - H2: Gerelateerd

## tools/acp-agents.md

- Route: /tools/acp-agents
- Koppen:
  - H2: Welke pagina heb ik nodig?
  - H2: Werkt dit direct?
  - H2: Ondersteunde harnasdoelen
  - H2: Draaiboek voor beheerders
  - H2: ACP versus subagenten
  - H2: Hoe ACP Claude Code uitvoert
  - H2: Gebonden sessies
  - H3: Denkmodel
  - H3: Bindingen voor het huidige gesprek
  - H2: Permanente kanaalbindingen
  - H3: Bindingsmodel
  - H3: Runtimestandaarden per agent
  - H3: Voorbeeld
  - H3: Gedrag
  - H2: ACP-sessies starten
  - H3: `sessions_spawn`-parameters
  - H2: Modi voor starten, binden en threads
  - H2: Leveringsmodel
  - H2: Compatibiliteit met de sandbox
  - H2: Doelresolutie voor sessies
  - H2: ACP-bediening
  - H3: Toewijzing van runtimeopties
  - H2: acpx-harnas, Pluginconfiguratie en machtigingen
  - H2: Problemen oplossen
  - H2: Gerelateerd

## tools/agent-send.md

- Route: /tools/agent-send
- Koppen:
  - H2: Snel aan de slag
  - H2: Vlaggen
  - H2: Gedrag
  - H2: Voorbeelden
  - H2: Gerelateerd

## tools/apply-patch.md

- Route: /tools/apply-patch
- Koppen:
  - H2: Parameters
  - H2: Opmerkingen
  - H2: Voorbeeld
  - H2: Gerelateerd

## tools/ask-user.md

- Route: /tools/ask-user
- Koppen:
  - H2: Een vraag beantwoorden
  - H2: Platformgedrag
  - H2: Time-out en geen antwoord
  - H2: Toolschema
  - H2: Richtlijnen voor het model

## tools/brave-search.md

- Route: /tools/brave-search
- Koppen:
  - H2: Een API-sleutel verkrijgen
  - H2: Configuratievoorbeeld
  - H2: Toolparameters
  - H2: Opmerkingen
  - H2: Gerelateerd

## tools/browser-control.md

- Route: /tools/browser-control
- Koppen:
  - H2: Besturings-API (optioneel)
  - H3: Foutcontract van /act
  - H3: Playwright-vereiste
  - H4: Playwright-installatie voor Docker
  - H2: Hoe het werkt (intern)
  - H2: Beknopt CLI-overzicht
  - H2: Momentopnamen en referenties
  - H2: CLI voor browserbatchbewerkingen
  - H2: Uitgebreide wachtmogelijkheden
  - H2: Workflows voor foutopsporing
  - H2: JSON-uitvoer
  - H2: Instellingen voor status en omgeving
  - H2: Beveiliging en privacy
  - H2: Gerelateerd

## tools/browser-linux-troubleshooting.md

- Route: /tools/browser-linux-troubleshooting
- Koppen:
  - H2: Probleem: kan Chrome CDP niet starten op poort 18800
  - H3: Hoofdoorzaak
  - H3: Oplossing 1: installeer Google Chrome (aanbevolen)
  - H3: Oplossing 2: gebruik snap Chromium in de modus voor alleen koppelen
  - H3: Controleer of de browser werkt
  - H3: Configuratiereferentie
  - H3: Probleem: geen Chrome-tabbladen gevonden voor profile="user"
  - H2: Gerelateerd

## tools/browser-login.md

- Route: /tools/browser-login
- Koppen:
  - H2: Handmatig aanmelden (aanbevolen)
  - H2: Welk Chrome-profiel wordt gebruikt?
  - H2: Sandboxing: toegang tot de hostbrowser toestaan
  - H2: Gerelateerd

## tools/browser-wsl2-windows-remote-cdp-troubleshooting.md

- Route: /tools/browser-wsl2-windows-remote-cdp-troubleshooting
- Koppen:
  - H2: Kies eerst de juiste browsermodus
  - H3: Optie 1: rechtstreeks extern CDP van WSL2 naar Windows
  - H3: Optie 2: hostlokale Chrome MCP
  - H2: Werkende architectuur
  - H2: Cruciale regel voor de Control UI
  - H2: Laagsgewijs valideren
  - H3: Laag 1: controleer of Chrome CDP aanbiedt op Windows
  - H4: Diagnoseer IPv4 en IPv6 voordat je portproxy wijzigt
  - H3: Laag 2: controleer of WSL2 dat Windows-eindpunt kan bereiken
  - H3: Laag 3: configureer het juiste browserprofiel
  - H3: Laag 4: controleer de Control UI-laag afzonderlijk
  - H3: Laag 5: controleer de browserbesturing van begin tot eind
  - H2: Veelvoorkomende misleidende fouten
  - H2: Checklist voor snelle triage
  - H2: Gerelateerd

## tools/browser.md

- Route: /tools/browser
- Koppen:
  - H2: Wat je krijgt
  - H2: Snel aan de slag
  - H2: Pluginbesturing
  - H2: Richtlijnen voor agenten
  - H2: Ontbrekende browseropdracht of -tool
  - H2: Profielen: openclaw, user, chrome
  - H2: Configuratie
  - H3: Eigenaarschap van tabbladopruiming
  - H3: Visuele schermafbeeldingsanalyse (ondersteuning voor modellen die alleen tekst verwerken)
  - H2: Brave of een andere Chromium-browser gebruiken
  - H2: Lokale versus externe besturing
  - H2: Node-browserproxy (standaard zonder configuratie)
  - H2: Browserless (gehost extern CDP)
  - H3: Browserless Docker op dezelfde host
  - H2: Rechtstreekse WebSocket-CDP-providers
  - H3: Browserbase
  - H3: Notte
  - H2: Beveiliging
  - H2: Profielen (meerdere browsers)
  - H2: Bestaande sessie via Chrome DevTools MCP
  - H3: Aangepaste start van Chrome MCP
  - H2: Isolatiegaranties
  - H2: Browserselectie
  - H2: Besturings-API (optioneel)
  - H2: Problemen oplossen
  - H3: CDP-opstartfout versus SSRF-blokkering bij navigatie
  - H2: Agenttools en de werking van de besturing
  - H2: Gerelateerd

## tools/btw.md

- Route: /tools/btw
- Koppen:
  - H2: Wat het doet
  - H2: Wat het niet doet
  - H2: Leveringsmodel
  - H2: Gedrag van de interface
  - H2: Selectiepop-up (Control UI)
  - H2: Wanneer je het gebruikt
  - H2: Gerelateerd

## tools/capability-cookbook.md

- Route: /tools/capability-cookbook
- Koppen:
  - H2: Gerelateerd

## tools/chrome-extension.md

- Route: /tools/chrome-extension
- Koppen:
  - H1: Chrome-extensie
  - H2: Hoe het werkt
  - H2: Installeren en koppelen
  - H2: Gebruiken
  - H3: Zijpaneel met tabbladcopiloot
  - H2: Een pagina naar OpenClaw sturen
  - H2: Op afstand / tussen machines
  - H2: Diagnostiek
  - H2: Beveiligingsmodel

## tools/clawhub.md

- Route: /tools/clawhub
- Koppen: geen

## tools/code-execution.md

- Route: /tools/code-execution
- Koppen:
  - H2: Installatie
  - H2: Hoe je het gebruikt
  - H2: Fouten
  - H2: Gerelateerd

## tools/code-mode.md

- Route: /tools/code-mode
- Koppen:
  - H2: Wat het doet
  - H2: Waarom je het gebruikt
  - H2: Snel aan de slag
  - H3: Code Mode inschakelen
  - H3: Wat het model doet
  - H3: Het actieve oppervlak verifiëren
  - H2: Swarm gebruiken voor het uitwaaieren van agents
  - H2: Technische rondleiding
  - H2: Runtimestatus
  - H2: Bereik
  - H2: Begrippen
  - H2: Configuratie
  - H2: Activering
  - H2: Voor het model zichtbare tools
  - H2: exec
  - H2: wait
  - H2: Gast-runtime-API
  - H2: Gedeclareerde uitvoercontracten
  - H2: Interne naamruimten
  - H3: Levenscyclus van het register
  - H3: Registratiestructuur
  - H3: Eigenaarschap en zichtbaarheid
  - H3: Regels voor bereikserialisatie
  - H3: Prompts
  - H3: Opschoning
  - H3: Testchecklist
  - H2: Uitvoer-API
  - H2: Toolcatalogus
  - H2: Interactie met Tool Search
  - H2: Toolnamen en conflicten
  - H2: Geneste tooluitvoering
  - H2: Levenscyclus van uitvoeringen en momentopnamen
  - H2: QuickJS-WASI-runtime
  - H2: TypeScript
  - H2: Beveiligingsgrens
  - H2: Foutcodes
  - H2: Telemetrie
  - H2: Foutopsporing
  - H2: Implementatiestructuur
  - H2: Validatiechecklist
  - H2: E2E-testplan
  - H2: Gerelateerd

## tools/creating-skills.md

- Route: /tools/creating-skills
- Koppen:
  - H2: Je eerste skill maken
  - H2: Naslaginformatie voor SKILL.md
  - H3: Vereiste velden
  - H3: Optionele frontmatter-sleutels
  - H3: {baseDir} gebruiken
  - H2: Voorwaardelijke activering toevoegen
  - H2: Voorstellen via Skill Workshop
  - H2: Publiceren naar ClawHub
  - H2: Aanbevolen werkwijzen
  - H2: Gerelateerd

## tools/diffs.md

- Route: /tools/diffs
- Koppen:
  - H2: Snel aan de slag
  - H2: Ingebouwde systeemrichtlijnen uitschakelen
  - H2: Naslaginformatie voor toolinvoer
  - H2: Syntaxismarkering
  - H2: Contract voor uitvoerdetails
  - H3: Samengevouwen ongewijzigde secties
  - H3: Navigatie tussen meerdere bestanden
  - H2: Standaardinstellingen van de Plugin
  - H3: Configuratie van permanente viewer-URL
  - H2: Beveiligingsconfiguratie
  - H2: Levenscyclus en opslag van artefacten
  - H2: Viewer-URL en netwerkgedrag
  - H2: Beveiligingsmodel
  - H2: Browservereisten voor bestandsmodus
  - H2: Problemen oplossen
  - H2: Operationele richtlijnen
  - H2: Gerelateerd

## tools/duckduckgo-search.md

- Route: /tools/duckduckgo-search
- Koppen:
  - H2: Installatie
  - H2: Configuratie
  - H2: Toolparameters
  - H2: Opmerkingen
  - H2: Gerelateerd

## tools/elevated.md

- Route: /tools/elevated
- Koppen:
  - H2: Instructies
  - H2: Hoe het werkt
  - H2: Oplossingsvolgorde
  - H2: Beschikbaarheid en toelatingslijsten
  - H2: Wat elevated niet beheert
  - H2: Gerelateerd

## tools/exa-search.md

- Route: /tools/exa-search
- Koppen:
  - H2: Plugin installeren
  - H2: Een API-sleutel verkrijgen
  - H2: Configuratie
  - H2: Basis-URL overschrijven
  - H2: Toolparameters
  - H3: Inhoudsextractie
  - H3: Zoekmodi
  - H2: Opmerkingen
  - H2: Gerelateerd

## tools/exec-approvals-advanced.md

- Route: /tools/exec-approvals-advanced
- Koppen:
  - H2: Veilige binaire bestanden (alleen stdin)
  - H3: Argv-validatie en geweigerde vlaggen
  - H3: Vertrouwde mappen met binaire bestanden
  - H3: Shell-koppeling, wrappers en multiplexers
  - H3: Veilige binaire bestanden versus toelatingslijst
  - H2: Interpreter-/runtimeopdrachten
  - H3: Gedrag bij levering van vervolgberichten
  - H2: Minimale bereiken voor clients van derden
  - H2: Goedkeuringen doorsturen naar chatkanalen
  - H3: Goedkeuringen doorsturen via een Plugin
  - H3: Goedkeuringen in dezelfde chat op elk kanaal
  - H3: Systeemeigen levering van goedkeuringen
  - H3: Officiële mobiele operatorapps
  - H3: macOS-IPC-flow
  - H2: Veelgestelde vragen
  - H3: Wanneer worden accountId en threadId gebruikt voor een goedkeuringsdoel?
  - H3: Wanneer goedkeuringen naar een sessie worden gestuurd, kan iedereen in die sessie ze dan goedkeuren?
  - H2: Gerelateerd

## tools/exec-approvals.md

- Route: /tools/exec-approvals
- Koppen:
  - H2: Waar dit van toepassing is
  - H3: Vertrouwensmodel
  - H3: macOS-splitsing
  - H2: Het effectieve beleid inspecteren
  - H2: Instellingen en opslag
  - H2: Beleidsopties
  - H3: tools.exec.mode
  - H3: exec.security
  - H3: exec.ask
  - H3: askFallback
  - H3: tools.exec.strictInlineEval
  - H3: tools.exec.commandHighlighting
  - H2: YOLO-modus (zonder goedkeuring)
  - H3: Permanente instelling 'nooit vragen' op de Gateway-host
  - H3: Lokale snelkoppeling
  - H3: Node-host
  - H3: Snelkoppeling alleen voor de sessie
  - H2: Toelatingslijst (per agent)
  - H3: Argumenten beperken met argPattern
  - H2: CLI's van skills automatisch toestaan
  - H2: Veilige binaire bestanden en doorsturen van goedkeuringen
  - H2: Bewerken in de Control UI
  - H2: Goedkeuringsflow
  - H2: Systeemgebeurtenissen en weigeringen
  - H2: Gevolgen
  - H2: Gerelateerd

## tools/exec.md

- Route: /tools/exec
- Koppen:
  - H2: Parameters
  - H2: Configuratie
  - H3: Modi
  - H3: Inline-evaluatie (strictInlineEval)
  - H3: PATH-verwerking
  - H2: Sessieoverschrijvingen (/exec)
  - H2: Exec-goedkeuringen (begeleidende app / Node-host)
  - H2: Toelatingslijst + veilige binaire bestanden
  - H2: Voorbeelden
  - H2: applypatch
  - H2: Gerelateerd

## tools/firecrawl.md

- Route: /tools/firecrawl
- Koppen:
  - H2: Plugin installeren
  - H2: Sleutelloze toegang en API-sleutels
  - H2: Firecrawl-zoekfunctie configureren
  - H2: Firecrawl-webfetch-fallback configureren
  - H3: Zelfgehoste Firecrawl
  - H2: Tools van de Firecrawl-Plugin
  - H3: `firecrawl_search`
  - H3: `firecrawl_scrape`
  - H2: Stealth / bots omzeilen
  - H2: Hoe `web_fetch` Firecrawl gebruikt
  - H2: Gerelateerd

## tools/gemini-search.md

- Route: /tools/gemini-search
- Koppen:
  - H2: Een API-sleutel verkrijgen
  - H2: Configuratie
  - H2: Hoe het werkt
  - H2: Ondersteunde parameters
  - H2: Modelselectie
  - H2: Overschrijvingen van de basis-URL
  - H2: Gerelateerd

## tools/goal.md

- Route: /tools/goal
- Koppen:
  - H1: Doel
  - H2: Snel aan de slag
  - H2: Waar doelen voor dienen
  - H2: Opdrachtenoverzicht
  - H2: Statussen
  - H2: Tokenbudgetten
  - H2: Modeltools
  - H2: Doelcontext bij elke beurt
  - H2: Control UI
  - H2: TUI
  - H2: Kanaalgedrag
  - H2: Problemen oplossen
  - H2: Gerelateerd

## tools/grok-search.md

- Route: /tools/grok-search
- Koppen:
  - H2: Onboarding en configuratie
  - H2: Aanmelden of een API-sleutel verkrijgen
  - H2: Configuratie
  - H2: Hoe het werkt
  - H2: Ondersteunde parameters
  - H2: Overschrijvingen van de basis-URL
  - H2: Gerelateerd

## tools/image-generation.md

- Route: /tools/image-generation
- Koppen:
  - H2: Snel aan de slag
  - H2: Veelgebruikte routes
  - H2: Ondersteunde providers
  - H2: Providermogelijkheden
  - H2: Toolparameters
  - H2: Configuratie
  - H3: Modelselectie
  - H3: Selectievolgorde van providers
  - H3: Afbeeldingen bewerken
  - H2: Diepgaande informatie per provider
  - H2: Voorbeelden
  - H2: Gerelateerd

## tools/index.md

- Route: /tools
- Koppen:
  - H2: Begin hier
  - H2: Tools, skills of plugins kiezen
  - H2: Categorieën van ingebouwde tools
  - H2: Door plugins geleverde tools
  - H2: Toegang en goedkeuringen configureren
  - H2: Mogelijkheden uitbreiden
  - H2: Problemen met ontbrekende tools oplossen
  - H2: Gerelateerd

## tools/kimi-search.md

- Route: /tools/kimi-search
- Koppen:
  - H2: Installatie
  - H2: Configuratie
  - H2: Vereiste voor grounding
  - H2: Toolparameters
  - H2: Gerelateerd

## tools/llm-task.md

- Route: /tools/llm-task
- Koppen:
  - H2: Inschakelen
  - H2: Configuratie (optioneel)
  - H2: Toolparameters
  - H2: Uitvoer
  - H2: Voorbeeld: Lobster-workflowstap
  - H3: Belangrijke beperking
  - H2: Veiligheidsopmerkingen
  - H2: Gerelateerd

## tools/lobster.md

- Route: /tools/lobster
- Koppen:
  - H2: Waarom
  - H2: Hoe het werkt
  - H2: Inschakelen
  - H2: Patroon: kleine CLI + JSON-pijplijnen + goedkeuringen
  - H2: LLM-stappen met uitsluitend JSON (llm-task)
  - H3: Belangrijke beperking: ingebedde Lobster versus openclaw.invoke
  - H2: Workflowbestanden (.lobster)
  - H3: Geïnjecteerde omgevingsvariabelen
  - H2: Toolparameters
  - H3: run
  - H3: resume
  - H3: Beheerde TaskFlow-modus
  - H2: Uitvoerenvelop
  - H2: Goedkeuringen
  - H2: OpenProse
  - H2: Veiligheid
  - H2: Probleemoplossing
  - H2: Meer informatie
  - H2: Casestudy: communityworkflows
  - H2: Gerelateerd

## tools/loop-detection.md

- Route: /tools/loop-detection
- Koppen:
  - H2: Waarom dit bestaat
  - H2: Configuratieblok
  - H3: Gedrag van velden
  - H2: Aanbevolen configuratie
  - H2: Beveiliging na Compaction
  - H2: Logboeken en verwacht gedrag
  - H2: Gerelateerd

## tools/media-overview.md

- Route: /tools/media-overview
- Koppen:
  - H2: Mogelijkheden
  - H2: Mogelijkhedenmatrix van providers
  - H2: Asynchroon versus synchroon
  - H2: Spraak-naar-tekst en spraakoproepen
  - H2: Providertoewijzingen (hoe leveranciers over oppervlakken zijn verdeeld)
  - H2: Gerelateerd

## tools/minimax-search.md

- Route: /tools/minimax-search
- Koppen:
  - H2: Een Token Plan-referentie verkrijgen
  - H2: Configuratie
  - H2: Regioselectie
  - H2: Ondersteunde parameters
  - H2: Gerelateerd

## tools/multi-agent-sandbox-tools.md

- Route: /tools/multi-agent-sandbox-tools
- Koppen:
  - H2: Configuratievoorbeelden
  - H2: Configuratieprioriteit
  - H3: Sandboxconfiguratie
  - H3: Toolbeperkingen
  - H2: Migratie vanaf één agent
  - H2: Voorbeelden van toolbeperkingen
  - H2: Veelvoorkomende valkuil: "non-main"
  - H2: Testen
  - H2: Probleemoplossing
  - H2: Gerelateerd

## tools/music-generation.md

- Route: /tools/music-generation
- Koppen:
  - H2: Snel aan de slag
  - H2: Ondersteunde providers
  - H3: Mogelijkhedenmatrix
  - H2: Toolparameters
  - H2: Asynchroon gedrag
  - H3: Taaklevenscyclus
  - H2: Configuratie
  - H3: Modelselectie
  - H3: Selectievolgorde van providers
  - H2: Opmerkingen over providers
  - H2: Het juiste pad kiezen
  - H2: Mogelijkheidsmodi van providers
  - H2: Livetests
  - H2: Gerelateerd

## tools/ollama-search.md

- Route: /tools/ollama-search
- Koppen:
  - H2: Installatie
  - H2: Configuratie
  - H2: Authenticatie en routering van aanvragen
  - H2: Gerelateerd

## tools/parallel-search.md

- Route: /tools/parallel-search
- Koppen:
  - H2: Plugin installeren
  - H2: API-sleutel (betaalde provider)
  - H2: Configuratie
  - H2: Basis-URL overschrijven
  - H2: Toolparameters
  - H2: Opmerkingen
  - H2: Gerelateerd

## tools/pdf.md

- Route: /tools/pdf
- Koppen:
  - H2: Beschikbaarheid
  - H2: Invoerverwijzing
  - H2: Ondersteunde PDF-verwijzingen
  - H2: Uitvoeringsmodi
  - H3: Systeemeigen providermodus
  - H3: Terugvalmodus voor extractie
  - H2: Configuratie
  - H2: Uitvoerdetails
  - H2: Foutgedrag
  - H2: Voorbeelden
  - H2: Gerelateerd

## tools/permission-modes.md

- Route: /tools/permission-modes
- Koppen:
  - H2: Aanbevolen standaardinstelling
  - H2: OpenClaw-uitvoeringsmodi op de host
  - H2: Toewijzing van Codex Guardian
  - H2: Machtigingen voor het ACPX-harnas
  - H2: Een modus kiezen
  - H2: Gerelateerd

## tools/perplexity-search.md

- Route: /tools/perplexity-search
- Koppen:
  - H2: Plugin installeren
  - H2: Een Perplexity API-sleutel verkrijgen
  - H2: Compatibiliteit met OpenRouter
  - H2: Configuratievoorbeelden
  - H3: Systeemeigen Perplexity Search-API
  - H3: Compatibiliteit met OpenRouter / Sonar
  - H2: Waar je de sleutel instelt
  - H2: Toolparameters
  - H3: Regels voor domeinfilters
  - H2: Opmerkingen
  - H2: Gerelateerd

## tools/plugin.md

- Route: /tools/plugin
- Koppen:
  - H2: Vereisten
  - H2: Snel aan de slag
  - H2: Configuratie
  - H3: Een installatiebron kiezen
  - H3: Installatiebeleid voor operators
  - H3: Pluginbeleid configureren
  - H2: Pluginindelingen begrijpen
  - H2: Pluginhooks
  - H2: De actieve Gateway verifiëren
  - H2: Probleemoplossing
  - H3: Geblokkeerd eigendom van het pluginpad
  - H3: Trage configuratie van plugintools
  - H2: Gerelateerd

## tools/reactions.md

- Route: /tools/reactions
- Koppen:
  - H2: Hoe het werkt
  - H2: Kanaalgedrag
  - H2: Reactieniveau
  - H2: Gerelateerd

## tools/screen.md

- Route: /tools/screen
- Koppen:
  - H2: Acties
  - H2: Routering en beveiliging
  - H2: Gerelateerd

## tools/searxng-search.md

- Route: /tools/searxng-search
- Koppen:
  - H2: Installatie
  - H2: Configuratie
  - H2: Omgevingsvariabele
  - H2: Configuratiereferentie voor de Plugin
  - H2: Opmerkingen
  - H2: Gerelateerd

## tools/self-learning.md

- Route: /tools/self-learning
- Koppen:
  - H2: Zelfleren inschakelen
  - H2: Eerdere sessies handmatig beoordelen
  - H2: Wat OpenClaw kan leren
  - H2: Wanneer de ervaringsbeoordeling wordt uitgevoerd
  - H2: Wat de beoordelaar ontvangt
  - H2: Veiligheid van voorstellen
  - H2: Geleerde voorstellen beoordelen
  - H2: Configuratie
  - H2: Probleemoplossing
  - H3: Na een lange beurt verschijnt geen voorstel
  - H3: Doctor meldt dat de Workshop-tool verborgen is
  - H3: Er verschijnen te veel voorstellen met weinig waarde
  - H2: Gerelateerd

## tools/show-widget.md

- Route: /tools/show-widget
- Koppen:
  - H2: Hoe widgets werken
  - H2: Ontwerpsysteem
  - H2: De tool gebruiken
  - H2: Interactieve widgets
  - H2: Dashboardmogelijkheden
  - H2: Beveiliging en opslag
  - H2: Gerelateerd

## tools/skill-workshop.md

- Route: /tools/skill-workshop
- Koppen:
  - H2: Hoe het werkt
  - H2: Levenscyclus
  - H2: Beheer van de levenscyclus
  - H2: Chat
  - H3: Leren van recent werk
  - H2: CLI
  - H2: Inhoud van voorstellen
  - H2: Ondersteunende bestanden
  - H2: Agenttool
  - H2: Voorgestelde Skills
  - H3: Eerdere sessies scannen
  - H2: Goedkeuring en autonomie
  - H2: Gateway-methoden
  - H2: Opslag
  - H2: Limieten
  - H2: Probleemoplossing
  - H3: Diagnose van toolbeleid
  - H2: Gerelateerd

## tools/skills-config.md

- Route: /tools/skills-config
- Koppen:
  - H2: Laden (skills.load)
  - H2: Installeren (skills.install)
  - H2: Installatiebeleid voor operators (security.installPolicy)
  - H2: Toestaanlijst voor meegeleverde Skills
  - H2: Vermeldingen per Skill (skills.entries)
  - H2: Toestaanlijsten voor agents (agents)
  - H2: Workshop (skills.workshop)
  - H2: Met symbolische koppelingen verbonden hoofdmappen van Skills
  - H2: Skills in een sandbox en omgevingsvariabelen
  - H2: Herinnering aan de laadvolgorde
  - H2: Gerelateerd

## tools/skills.md

- Route: /tools/skills
- Koppen:
  - H2: Laadvolgorde
  - H2: Door Node gehoste Skills
  - H2: Skills per agent versus gedeelde Skills
  - H2: Toestaanlijsten voor agents
  - H2: Plugins en Skills
  - H2: Skill Workshop
  - H2: Installeren vanuit ClawHub
  - H2: Beveiliging
  - H2: Indeling van SKILL.md
  - H3: Optionele frontmatter-sleutels
  - H2: Voorwaarden
  - H3: Installatiespecificaties
  - H2: Configuratieoverschrijvingen
  - H2: Omgevingsinjectie
  - H2: Momentopnamen en vernieuwen
  - H2: Tokenimpact
  - H2: Gerelateerd

## tools/slash-commands.md

- Route: /tools/slash-commands
- Koppen:
  - H2: Drie opdrachttypen
  - H2: Configuratie
  - H2: Opdrachtenlijst
  - H3: Kernopdrachten
  - H3: Dock-opdrachten
  - H3: Opdrachten van meegeleverde Plugins
  - H3: Skill-opdrachten
  - H2: /tools: wat de agent nu kan gebruiken
  - H2: /model: modelselectie
  - H2: /config: configuratie naar schijf schrijven
  - H2: /mcp: MCP-serverconfiguratie
  - H2: /debug: uitsluitend runtime-overschrijvingen
  - H2: /plugins: Pluginbeheer
  - H2: /trace: trace-uitvoer van Plugins
  - H2: /btw: tussentijdse vragen
  - H2: Opmerkingen per oppervlak
  - H2: Providergebruik en -status
  - H2: Gerelateerd

## tools/steer.md

- Route: /tools/steer
- Koppen:
  - H2: Huidige sessie
  - H2: Bijsturen versus in de wachtrij plaatsen
  - H2: Subagents
  - H2: ACP-sessies
  - H2: Gerelateerd

## tools/subagents.md

- Route: /tools/subagents
- Koppen:
  - H2: Slash-opdracht
  - H3: Besturingselementen voor threadkoppeling
  - H3: Startgedrag
  - H2: Contextmodi
  - H2: Tool: `sessions_spawn`
  - H3: Promptmodus voor delegatie
  - H3: Toolparameters
  - H3: Taaknamen en doelbepaling
  - H2: Tool: `sessions_yield`
  - H2: Tool: subagents
  - H2: Threadgebonden sessies
  - H3: Kanalen met threadondersteuning
  - H3: Snelle werkwijze
  - H3: Handmatige besturing
  - H3: Configuratieschakelaars
  - H3: Toegestane lijst
  - H3: Detectie
  - H3: Automatisch archiveren
  - H2: Geneste subagenten
  - H3: Diepteniveaus
  - H3: Aankondigingsketen
  - H3: Toolbeleid per diepte
  - H3: Startlimiet per agent
  - H3: Cascadestop
  - H2: Authenticatie
  - H2: Aankondiging
  - H3: Aankondigingscontext
  - H3: Statistiekenregel
  - H3: Waarom `sessions_history` de voorkeur heeft
  - H2: Toolbeleid
  - H3: Overschrijven via configuratie
  - H2: Gelijktijdigheid
  - H2: Beschikbaarheid en herstel
  - H2: Stoppen
  - H2: Beperkingen
  - H2: Gerelateerd

## tools/swarm.md

- Route: /tools/swarm
- Koppen:
  - H2: Swarm inschakelen
  - H2: Vereisten
  - H2: Een Swarm-script schrijven
  - H3: Parallel uitwaaieren met gestructureerde resultaten
  - H3: Herhalen bij een beslissingspoort
  - H3: Het eerste voltooide onderliggende proces verwerken
  - H2: Gedrag van onderliggende verzamelprocessen
  - H3: Onderliggende processen zijn eindpunten
  - H2: Een Swarm observeren
  - H2: Swarm vanuit andere harnassen gebruiken
  - H2: Beperkingen en roadmap
  - H2: Gerelateerd

## tools/tavily.md

- Route: /tools/tavily
- Koppen:
  - H2: Aan de slag
  - H2: Toolreferentie
  - H3: `tavily_search`
  - H3: `tavily_extract`
  - H2: De juiste tool kiezen
  - H2: Geavanceerde configuratie
  - H2: Gerelateerd

## tools/thinking.md

- Route: /tools/thinking
- Koppen:
  - H2: Wat het doet
  - H2: Volgorde van omzetting
  - H2: Een sessiestandaard instellen
  - H2: Toepassing per agent
  - H2: Snelle modus (/fast)
  - H2: Uitgebreide richtlijnen (/verbose of /v)
  - H2: Traceringsrichtlijnen voor plugins (/trace)
  - H2: Zichtbaarheid van redeneringen (/reasoning)
  - H2: Gerelateerd
  - H2: Heartbeats
  - H2: Webchatinterface
  - H2: Providerprofielen

## tools/tokenjuice.md

- Route: /tools/tokenjuice
- Koppen:
  - H2: De plugin inschakelen
  - H2: Wat Tokenjuice verandert
  - H2: Controleren of het werkt
  - H2: De plugin uitschakelen
  - H2: Gerelateerd

## tools/tool-search.md

- Route: /tools/tool-search
- Koppen:
  - H2: Hoe een beurt wordt uitgevoerd
  - H2: Modi
  - H2: Waarom dit bestaat
  - H2: API
  - H2: Runtimegrens
  - H2: Configuratie
  - H2: Prompt en telemetrie
  - H2: E2E-validatie
  - H2: Gedrag bij fouten
  - H2: Gerelateerd

## tools/trajectory.md

- Route: /tools/trajectory
- Koppen:
  - H2: Snel aan de slag
  - H2: Toegang
  - H2: Wat wordt vastgelegd
  - H2: Bundelbestanden
  - H2: Opslag van vastleggingen
  - H2: Vastlegging uitschakelen
  - H2: Time-out voor wegschrijven afstemmen
  - H2: Privacy en beperkingen
  - H2: Probleemoplossing
  - H2: Gerelateerd

## tools/tts.md

- Route: /tools/tts
- Koppen:
  - H2: Snel aan de slag
  - H2: Ondersteunde providers
  - H2: Configuratie
  - H3: Stemoverschrijvingen per agent
  - H2: Persona's
  - H3: Minimale persona
  - H3: Volledige persona (providerspecifieke vormgeving)
  - H3: Personaomzetting
  - H3: Aangepaste personavormgeving
  - H3: Terugvalbeleid
  - H2: Modelgestuurde richtlijnen
  - H2: Slash-opdrachten
  - H2: Voorkeuren per gebruiker
  - H2: Uitvoerindelingen
  - H2: Gedrag van automatische TTS
  - H2: Veldreferentie
  - H2: Agenttool
  - H2: Gateway-RPC
  - H2: Servicekoppelingen
  - H2: Gerelateerd

## tools/video-generation.md

- Route: /tools/video-generation
- Koppen:
  - H2: Snel aan de slag
  - H2: Hoe asynchrone generatie werkt
  - H3: Levenscyclus van een taak
  - H2: Ondersteunde providers
  - H3: Mogelijkhedenmatrix
  - H2: Toolparameters
  - H3: Vereist
  - H3: Inhoudsinvoer
  - H3: Stijlbesturing
  - H3: Geavanceerd
  - H4: Terugval- en getypeerde opties
  - H2: Acties
  - H2: Modelselectie
  - H2: Providernotities
  - H2: Modi voor providermogelijkheden
  - H2: Livetests
  - H2: Configuratie
  - H2: Gerelateerd

## tools/web-fetch.md

- Route: /tools/web-fetch
- Koppen:
  - H2: Snel aan de slag
  - H2: Toolparameters
  - H2: Resultaat
  - H2: Hoe het werkt
  - H2: Voortgangsupdates
  - H2: Configuratie
  - H2: Firecrawl-terugval
  - H2: Vertrouwde omgevingsproxy
  - H2: Beperkingen en veiligheid
  - H2: Toolprofielen
  - H2: Gerelateerd

## tools/web.md

- Route: /tools/web
- Koppen:
  - H2: Snel aan de slag
  - H2: Een provider kiezen
  - H3: Providervergelijking
  - H2: Resultaatstructuur
  - H2: Automatische detectie
  - H2: Systeemeigen OpenAI-webzoekfunctie
  - H2: Systeemeigen Codex-webzoekfunctie
  - H2: Netwerkveiligheid
  - H2: Configuratie
  - H3: API-sleutels opslaan
  - H2: Toolparameters
  - H2: xsearch
  - H3: xsearch-configuratie
  - H3: xsearch-parameters
  - H3: xsearch-voorbeeld
  - H2: Voorbeelden
  - H2: Toolprofielen
  - H2: Gerelateerd

## tts.md

- Route: /tts
- Koppen:
  - H2: Gerelateerd

## vps.md

- Route: /vps
- Koppen:
  - H2: Een provider kiezen
  - H2: Hoe cloudconfiguraties werken
  - H2: Eerst beheerderstoegang beveiligen
  - H2: Gedeelde bedrijfsagent op een VPS
  - H2: Nodes gebruiken met een VPS
  - H2: Opstartafstemming voor kleine VM's en ARM-hosts
  - H3: Checklist voor systemd-afstemming (optioneel)
  - H2: Gerelateerd

## web/control-ui.md

- Route: /web/control-ui
- Koppen:
  - H2: Snel openen (lokaal)
  - H2: Apparaat koppelen (eerste verbinding)
  - H2: Een mobiel apparaat koppelen
  - H2: Persoonlijke identiteit (lokaal in de browser)
  - H2: Runtimeconfiguratie-eindpunt
  - H2: Status van de Gateway-host
  - H2: Taalondersteuning
  - H2: Weergavethema's
  - H2: OpenClaw-systeemonderhoud
  - H2: Plugins beheren
  - H2: Apps en extensies
  - H2: Zijbalknavigatie
  - H2: Pagina voor een nieuwe sessie
  - H2: Wat het kan (vandaag)
  - H2: Assistentgeheugen importeren
  - H2: MCP-pagina
  - H2: Activiteitentabblad
  - H2: Operatorterminal
  - H2: Browserpaneel
  - H2: Chatgedrag
  - H2: Verbindingsverlies en opnieuw verbinden
  - H2: PWA-installatie en webpush
  - H2: Gehoste insluitingen
  - H2: Indeling van het chattranscript
  - H2: Breedte van chatberichten
  - H2: Tailnet-toegang (aanbevolen)
  - H2: Onveilige HTTP
  - H2: Beleid voor inhoudsbeveiliging
  - H2: Authenticatie voor de avatarroute
  - H2: Authenticatie voor de mediaroute van de assistent
  - H2: Goedkeuringslinks
  - H2: Lege Control UI-pagina
  - H2: Foutopsporing/tests: ontwikkelserver + externe Gateway
  - H2: Gerelateerd

## web/dashboard-architecture.md

- Route: /web/dashboard-architecture
- Koppen:
  - H2: Visie
  - H2: Concepten
  - H2: UX-stromen
  - H2: Interactieniveaus
  - H2: Widgetmodel en hosting
  - H3: Widgets hosten inhoud; MCP-apps zijn één soort inhoud
  - H3: Declaraties van pluginmogelijkheden
  - H3: Gemodelleerd restant: WebRTC-datakanalen
  - H3: Transcriptweergave: één widgetkaart
  - H3: Widgets afkomstig van de server (vastgezette MCP-apps)
  - H3: WorkBoard-integratie
  - H2: Indeling: vloeiend raster
  - H2: Datamodel (database per agent)
  - H2: Protocoloppervlak
  - H2: Agenttools
  - H2: Wat dit vervangt
  - H2: Niet-doelstellingen (dit programma)
  - H2: Implementatieplan

## web/dashboard.md

- Route: /web/dashboard
- Koppen:
  - H2: Snelle route (aanbevolen)
  - H2: Basisprincipes van authenticatie (lokaal versus extern)
  - H2: Openen in Telegram
  - H2: Als je "unauthorized" / 1008 ziet
  - H2: Gerelateerd

## web/dashboards.md

- Route: /web/dashboards
- Koppen:
  - H2: Een dashboard bouwen door erom te vragen
  - H2: Het bord
  - H2: Wat widgets mogen doen
  - H2: MCP-apps op het bord
  - H2: Goed om te weten

## web/index.md

- Route: /web
- Koppen:
  - H2: Configuratie (standaard ingeschakeld)
  - H2: Webhooks
  - H2: HTTP-RPC voor beheer
  - H2: Tailscale-toegang
  - H2: Beveiligingsopmerkingen
  - H2: De gebruikersinterface bouwen

## web/lobster.md

- Route: /web/lobster
- Koppen:
  - H2: Waar je naar kijkt
  - H2: Wanneer het verschijnt
  - H2: Wat je kunt doen
  - H2: Bezoeken uitschakelen (of weer inschakelen)
  - H2: De Lobsterdex
  - H2: Veldnotities
  - H2: Privacy

## web/tui.md

- Route: /web/tui
- Koppen:
  - H2: Snel aan de slag
  - H3: Gateway-modus
  - H3: Lokale modus
  - H2: Wat je ziet
  - H2: Mentaal model: agents + sessies
  - H2: Verzenden + afleveren
  - H2: Selectiemenu's + overlays
  - H2: Sneltoetsen
  - H2: Slash-opdrachten
  - H2: Lokale shellopdrachten
  - H2: Hulp voor het instellen en herstellen van OpenClaw
  - H2: Tooluitvoer
  - H2: Terminalkleuren
  - H2: Geschiedenis + streaming
  - H2: Verbindingsgegevens
  - H2: Opties
  - H2: Problemen oplossen
  - H2: Verbindingsproblemen oplossen
  - H2: Gerelateerd

## web/webchat.md

- Route: /web/webchat
- Koppen:
  - H2: Wat het is
  - H2: Snel aan de slag
  - H2: Hoe het werkt
  - H3: Transcript- en afleveringsmodel
  - H2: Toolspaneel voor agents in de Control UI
  - H2: Gebruik op afstand
  - H2: Configuratiereferentie (WebChat)
  - H2: Gerelateerd
