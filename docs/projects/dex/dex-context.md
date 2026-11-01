# Dex: levende kontekst

Det vi ved om Dex, og hvor vi ved det fra. Opdatér filen, når vi lærer noget
nyt; datér og kildeangiv nye fakta. Sidst opdateret 2026-11-01.

## Produktet i én sætning

Dex er en personlig CRM ("Built for people, not sales pipeline"): ét sted til
alle dine relationer, hvor du husker detaljerne om folk, bliver mindet om at
holde kontakten og får dine kanaler (LinkedIn, mail, kalender, beskeder) samlet
på hver person.

Positionering fra projektets instruktioner og forsiden:

- "The easiest way to consolidate your relationships"
- "One place for your relationships - impress with thoughtfulness"
- "Remember everything about everyone. Use Dex to store the important details
  you don't want to forget"

Forretningsmodel: abonnement, ingen reklamer. Gratis start, Professional-plan
kræves for API-nøgler. Studierabat. Eksport og sletning af data når som helst.

## Feature-træ

Grundlaget for feature-diagrammet ("company-diagram") i
[diagrams/](diagrams/). Hver gren er et produktområde; bladene er funktioner.

```text
Dex
├── Kontakter
│   ├── Profil: navn, titel, firma, uddannelse, beskrivelse, billede, visitkort
│   ├── Kontaktinfo: flere e-mails, telefonnumre, adresser (rækkefølge kan ændres)
│   ├── Sociale profiler: LinkedIn, X, Facebook, Instagram, Telegram, TikTok, YouTube
│   ├── Vigtige datoer: fødselsdag, "first met", første interaktion
│   ├── Relationer mellem kontakter (familie m.m.)
│   ├── Stjernemarkering, arkivering, prioritetsniveau
│   ├── Dubletter: find og flet
│   ├── Søgning (navn og fuldtekst) og avanceret filter (where-DSL)
│   ├── Kort / geografi (afstandsfilter)
│   └── Dex Research: AI-opsummering af en kontakt fra nettet
├── Organisering
│   ├── Grupper (med antal pr. gruppe, flet grupper)
│   ├── Tags (bulk tildel / fjern)
│   ├── Custom fields på kontakter og firmaer
│   ├── Favoritter (gemte visninger / genveje)
│   └── Views og saved searches
├── Firmaer
│   ├── Firmaprofil, kontakter i firmaet, team
│   ├── Firmaroller pr. kontakt (nuværende, tidligere, primær)
│   ├── Org-diagram pr. firma
│   ├── Aktivitet og titelskift pr. firma
│   ├── Arkiv, flet, tags, statistik
│   └── CSV-import (plan → anvend) og CSV-eksport
├── Tidslinje og noter
│   ├── Noter pr. kontakt med tidspunkt, emoji og mødetype
│   ├── Mødetyper (brugerdefinerede)
│   ├── Samlet tidslinje (noter + mails + møder + beskeder)
│   └── Søgning i noter
├── Hold kontakten (Keep in Touch)
│   ├── Frekvenser (fx hver 30. dag) pr. kontakt eller gruppe
│   ├── Kadencer: forfalden, markér som opfyldt, snooze
│   └── Bulk-tildeling af kadencer
├── Påmindelser
│   ├── Enkelte og gentagne påmindelser
│   ├── Påmindelse på mange kontakter på én gang
│   └── Fødselsdagsliste
├── Agenda og opmærksomhed
│   ├── Daglig agenda: kalenderbegivenheder, påmindelser, fødselsdage
│   ├── Attention items: tråde der venter på svar
│   ├── Udkast til svar og send svar
│   └── Opfølgningsforslag efter møder
├── LinkedIn
│   ├── Synk netværket og hold kontakter opdateret
│   ├── Jobskifte- / titelskifte-alarmer
│   └── Seneste LinkedIn-besked pr. kontakt
├── Integrationer
│   ├── Mail: Gmail, Outlook, Superhuman (seneste interaktion pr. kontakt)
│   ├── Kalender: Google Calendar, Office 365
│   ├── Beskeder: WhatsApp, iMessage, Instagram, Facebook
│   ├── Noter fra møder: Granola
│   ├── Tovejs kontaktsynk (fx Google Contacts)
│   └── Zapier
├── AI
│   ├── MCP-server: Claude, ChatGPT, Gemini kan læse kontakter, noter, opfølgning
│   ├── AI Skills
│   └── Dex Research på kontakter
├── Platforme
│   ├── Web-app, desktop-app
│   ├── iOS og Android
│   └── Browserudvidelse (LinkedIn, Gmail, X, Facebook, Instagram)
└── Teams og workspaces
    ├── Dex for Teams: delte kontakter og interaktioner i et workspace
    ├── Team keep-in-touch og team-"last interaction"
    └── Medlemmer og delte mails/møder pr. kontakt
```

## Datamodel (udledt af API'et)

Kontakten er centrum. Hovedfelterne på `v2` contact-objektet:

- **Identitet:** `id` (uuid), `first_name`, `last_name`, `full_name`,
  `job_title`, `company`, `education`, `description`, `website`, `image_url`.
- **Datoer:** `birthday`, `birthday_year`, `first_met_at`,
  `first_interaction_at`, `last_seen_at`, `created_at`, `updated_at`.
- **Flag:** `starred`, `is_archived`, `ignore_merge`, `never_keep_in_touch`,
  `source`, `priority_tier`.
- **Hold kontakten:** `frequency`, `frequency_text`, `next_reminder_at`,
  `last_reminder_at`, `team_keep_in_touch { cadence_id, frequency_id, days,
next_due_at, snoozed_until }`.
- **Seneste interaktion pr. kanal:** dato, snippet og link for LinkedIn,
  WhatsApp, iMessage, Instagram, Gmail/Outlook, kalender og telefonopkald.
- **Relaterede lister:** e-mails, telefonnumre, lokationer, adresser, custom
  field-værdier, grupper, tags, tidslinjeposter, påmindelser,
  kontaktrelationer (begge retninger), feed items.
- **Øvrigt:** `last_note` (preview), `linkedin_data`, `web_search_summary`
  (Dex Research), `latitude`/`longitude`, `user_edited_fields`.

Andre entiteter: firma, firmarolle, gruppe, tag, note (timeline item),
mødetype, påmindelse, keep-in-touch-frekvens og -kadence, custom field,
favorit, titelskift, attention item, kalenderbegivenhed, workspace, bruger.

## API-fakta

Kilde: <https://getdex.com/docs/api-reference> og underside
`authentication`, `filtering-contacts`, `reference/v2/contacts/list-contacts`
(læst 2026-11-01).

- **Base-URL:** `https://api.prod.getdex.com`, versioner `v1` og `v2`. Byg på
  `v2`; `v1` er ældre og overlapper.
- **Auth:** `Authorization: Bearer dex_<64 tegn>`. Nøgle oprettes under
  Settings → Workflow → Dex API & MCP; vises kun én gang; kræver Professional.
- **Rate limit:** 100 kald/min pr. nøgle pr. endpoint. `429 rate_limited` med
  `Retry-After`; headers `x-ratelimit-limit`, `-remaining`, `-reset`.
- **Paginering:** cursor-baseret. Svar har `data.pagination { nextCursor,
count }` og `asOf`; send `nextCursor` uændret tilbage som `cursor`. `take`
  og `skip` findes også. Ikke dokumenteret: om `nextCursor: null` betyder
  sidste side. Behandl en manglende cursor som "ukendt", ikke som "færdig",
  indtil det er bekræftet.
- **Listeparametre:** `take`, `skip`, `cursor`, `where`, `orderBy`,
  `include`, `select` (Prisma-agtig form).
- **Filter-DSL (`where`):** topniveau-nøgler AND'es, `_or` er ét niveau dybt.
  Tekst er case-insensitiv contains; `{eq}` eksakt; `{not}` negerer;
  `{gte, lte}` datointerval; `has_*` med boolean tester tilstedeværelse;
  `has_tags_all`/`has_groups_all` er AND af OR-rækker; `has_distance {lat, lon,
meters}`; `custom_fields: [{ field_id, operator, value? }]` med operatorerne
  `contains, eq, in, not_in, present, absent, gte, lte`. Brug
  `POST /v2/contacts/filter` til ikke-trivielle filtre.

### Endpoint-grupper i v2

Reminders, Custom Fields, Meeting Types, Title Changes, Contact Emails,
Contact Phone Numbers, Contact Legacy Addresses, Contact Relations,
Keep In Touch, Companies, Company Roles, Attention Items, Workspaces, Agenda,
Contacts, Favorites, Groups, Timeline, Users, Tags. Den fulde liste og vores
mapping til actions står i [dex-api-to-actions.md](dex-api-to-actions.md).

## Kilder

- Forside og feature-sider: <https://getdex.com>, `/product-overview`,
  `/teams`, `/ai-agents`, `/skills`, `/browser-extension`, `/mobile`,
  `/desktop`, `/security`, `/integrations/*`, `/pricing`.
- Docs: <https://getdex.com/docs>, changelog
  <https://getdex.com/docs/changelog/changelog>, roadmap
  <https://feedback.getdex.com/roadmap>.

## Åbne spørgsmål om Dex

- Præcise pristrin og hvad der er gratis vs. Professional (pricing-siden er
  ikke læst endnu).
- Hvordan "Favorites" og "Views" fungerer i UI'et.
- Hvordan Attention Items opstår (hvilke kanaler, hvilken logik).
- Om `nextCursor: null` markerer sidste side.

## Ændringslog

- 2026-11-01: Første version: produkt, feature-træ, datamodel og API-fakta fra
  forsiden og API-referencen.
