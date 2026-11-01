# Agent-Native: grundviden for projektets agenter

Et kort over frameworket, så en ny tråd ved, hvad tingene hedder, og hvilken
skill der er autoriteten. Filen gentager ikke reglerne; den peger på dem.
Er der uenighed, vinder `CLAUDE.md` og skillen.

## Kerneideen

Agent og UI er ligeværdige: alt UI'et kan, kan agenten gennem de samme
SQL-data og den samme action-overflade. En feature, der kun findes i UI'et, er
usynlig for agenten, og omvendt.

## Byggestenene

| Begreb                             | Hvad det er                                                                                                                                                                                                                          | Autoritet                                             |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------- |
| **Actions**                        | `defineAction` i `actions/<navn>.ts`, én pr. fil. Agenten kalder dem som værktøjer; UI'et via `useActionQuery`/`useActionMutation` fra `@agent-native/core/client/hooks`. HTTP-transport på `/_agent-native/actions/:name` er gratis | `.agents/skills/actions`                              |
| **Data**                           | PostgreSQL via Drizzle. Additive migrationer, `ownableColumns()` + `accessFilter`/`assertAccess`. Store filer i blob-storage, kun referencer i SQL                                                                                   | `storing-data`, `security`, `framework-stores`        |
| **Application state**              | SQL-tabellen `application_state`: navigation, valg og fokuseret objekt, så agenten ved, hvad brugeren ser. Actions `navigate` og `view-screen`                                                                                       | `context-awareness`                                   |
| **Agent-chat**                     | Al AI går gennem agent-chatten. UI kalder `sendToAgentChat` med `openSidebar: true`; ingen LLM-kald fra browser eller actions                                                                                                        | `delegate-to-agent`                                   |
| **Skills og instruktioner**        | App-ens `AGENTS.md` (action-tabel), skills i `.agents/skills/`, system-prompt                                                                                                                                                        | `writing-agent-instructions`, `create-skill`          |
| **UI**                             | React Router, shadcn/ui, Tabler-ikoner, skeletter, optimistiske opdateringer, ingen unødig "chrome"                                                                                                                                  | `frontend-design`, `shadcn-ui`, `client-side-routing` |
| **Real-time**                      | Agentens skrivninger opdaterer UI'et af sig selv; `useDbSync` kun når eksterne ændringer skal ses live                                                                                                                               | `real-time-sync`                                      |
| **Konfiguration og hemmeligheder** | Ét zod-skema, `getAppConfig()`. Udbyder-nøgler i vault via workspace-forbindelser, aldrig `process.env`                                                                                                                              | `configuration`, `secrets`                            |
| **Udbyder-API'er**                 | `provider-api-catalog`/`-docs`/`-request` i stedet for en action pr. endpoint                                                                                                                                                        | `actions/references/provider-apis.md`                 |
| **Baggrundsarbejde**               | `recurring-jobs`, automations; aldrig tungt arbejde ved cold start                                                                                                                                                                   | `recurring-jobs`, `automations`, `performance`        |
| **Mini-apps**                      | Mange fokuserede apps, der kalder hinanden over A2A, frem for én kæmpe-app                                                                                                                                                           | `composable-mini-apps`, `a2a-protocol`                |

## Feature-tjeklisten (fire områder)

Hver feature med nye data eller en ny evne berører: **1) UI**, **2) actions**,
**3) skills/instruktioner**, **4) application state**. Se
`adding-a-feature`. En ren restyling berører kun de områder, den rører.

## Hvordan en app bliver til

- **`turn-into-app`**: fra en arbejdsgang (tråd, skill, regneark, projekt) til
  en app. Trin: kildebrief → App design (arketype, retning, shell, første
  viewport med sample-data, agent-øjeblikke) → scaffold → actions og UI →
  screenshot-review → typecheck, doctor, build. Kildebriefens skabelon:
  `.agents/skills/turn-into-app/references/source-brief.md`.
- **`build-an-app`**: den konkrete byggeopskrift (data, actions, UI,
  instruktioner, verifikation).
- **Scaffold:** `npx --yes @agent-native/core@latest create <slug> --standalone
--template chat` (selvstændig app) eller `pnpm exec agent-native add-app
<slug> --template chat` inde i et workspace. Tjek seneste version først.
- **Eksisterende CRM-skabelon:** `templates/crm` (skjult i det offentlige
  katalog) har typede attributter, lister, saved views, dublet-detektion og
  fletning. Læs den som reference, før vi designer egne løsninger på samme
  problemer.

## Ufravigelige regler, vi oftest kan komme til at bryde

- Ingen `/api/*`-ruter til app-data; undtagelser er upload, streaming,
  webhooks, OAuth-callbacks, offentlige URL'er og ikke-JSON svar.
- En `catch` eller default, der ligner succes, er en fejl. Tom, mangler og
  ulæselig skal kunne skelnes.
- Ingen hardcodede nøgler eller kundedata, heller ikke i eksempler og
  fixtures.
- Brugervendt tekst er lokalisering: kør `pnpm guard:i18n-catalogs` og
  `pnpm guard:i18n-changed-copy`.
- Ingen `Co-Authored-By` eller agent-mærkater i commits og GitHub-metadata.
- Afslut svar med statusblokken fra `CLAUDE.md`.

## Slå op i frameworket

`agent-native-docs` beskriver opslag: `framework-search`, derefter
`docs-search`/`source-search`. Frameworkets egne docs ligger i
`packages/core/docs/content`.
