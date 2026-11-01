# Dex API → Agent-Native

Hvordan Dex' domæne bliver til en Agent-Native-app. Dex' API er ikke en
skabelon, vi kopierer endpoint for endpoint: det er en kort over, hvad
produktet kan. Agent-Native-reglerne bestemmer formen. Læs
[knowledge/agent-native-basics.md](knowledge/agent-native-basics.md) først.

## Oversættelsesregler

Disse kommer fra `.agents/skills/actions`, `build-an-app`, `adding-a-feature`
og `turn-into-app`, og de afgør hvert valg nedenfor.

1. **Én action pr. fil, default-export, filnavn = action-navn**
   (`actions/list-contacts.ts`). Actions er den eneste vej til data for både UI
   (`useActionQuery`/`useActionMutation`) og agent. Ingen `/api/*`-ruter til
   app-data.
2. **Lille, ortogonal overflade.** Hver action er et værktøj i modellens
   kontekst. Dex har ~250 endpoints; vi sigter efter ~40 actions. Én
   `update-<ting>` med et valgfrit patch, ikke en action pr. felt.
3. **Batch-formede ønsker får én action med et array**, skrevet i én
   transaktion (Dex' `bulk-*` endpoints).
4. **Læsninger** er `http: { method: "GET" }` og `readOnly: true`, altid
   afgrænsede (`limit`/cursor), og returnerer struktureret data med `id`.
5. **Fejl** kastes med `fail(message, { errorCode, statusCode })`; aldrig et
   returneret `{ error }`. Ukendt, tom og ulæselig er forskellige værdier.
6. **AI-formet arbejde er ikke en action.** Research, udkast og opsummering
   kører i agent-sidebaren via `sendToAgentChat`, som orkestrerer de
   deterministiske actions.
7. **Ens id-navngivning** i en familie: `get-/update-/delete-contact` tager
   alle `id`.
8. **Data i PostgreSQL via Drizzle**, additive migrationer,
   `ownableColumns()` med `accessFilter`/`assertAccess`.

## Actions pr. domæne

`R` = læs (GET, readOnly), `W` = skriv (POST). Kolonnen "Dex" peger på de
v2-endpoint-grupper, actionen dækker.

### Kontakter

| Action                    |     | Dækker i Dex                                                              | Noter                                                                                                             |
| ------------------------- | --- | ------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `list-contacts`           | R   | list, count, filter, search-by-name, search-full                          | Ét filter-skema (vores udgave af `where`), `query`, `sort`, `cursor`, `limit`; returnerer `nextCursor` og `total` |
| `get-contact`             | R   | get-a-contact                                                             | Inkl. e-mails, telefoner, adresser, tags, grupper, custom fields, relationer, seneste note                        |
| `create-contacts`         | W   | create, bulk-create                                                       | Tager et array; ét element er det almindelige tilfælde                                                            |
| `update-contact`          | W   | update, contact-emails, phone-numbers, addresses, set-custom-field-values | Patch inkl. nested lister (erstat-semantik pr. liste)                                                             |
| `update-contacts`         | W   | bulk-update, archive                                                      | Samme patch på mange id'er; kræver preview ved store sæt                                                          |
| `delete-contacts`         | W   | delete, bulk-delete                                                       | Destruktiv: bekræftelse i UI, `needsApproval` for agenten                                                         |
| `find-duplicate-contacts` | R   | (implicit i merge)                                                        | Viser hvorfor kandidater matcher; fletter ikke                                                                    |
| `merge-contacts`          | W   | merge-duplicate-contacts                                                  | Eksplicit overlever; taberen tombstones og linkes                                                                 |
| `set-contact-relations`   | W   | contact-relations add/remove                                              | `add`/`remove` arrays                                                                                             |
| `list-birthdays`          | R   | list-birthdays                                                            | Kan foldes ind i `get-agenda`, hvis UI ikke behøver den alene                                                     |

### Organisering

| Action                                                                |     | Dækker i Dex                          | Noter                                                                |
| --------------------------------------------------------------------- | --- | ------------------------------------- | -------------------------------------------------------------------- |
| `list-groups`                                                         | R   | list, count, contact-counts-per-group | Antal pr. gruppe med i svaret                                        |
| `create-group` / `update-group` / `delete-group`                      | W   | create, update, delete, bulk-delete   |                                                                      |
| `set-group-members`                                                   | W   | add/remove-contacts                   | `add`/`remove` arrays af kontakt-id'er                               |
| `merge-groups`                                                        | W   | merge-groups                          |                                                                      |
| `list-tags`                                                           | R   | list, count, get                      |                                                                      |
| `create-tag` / `update-tag` / `delete-tag`                            | W   | create, update, delete, bulk          | Opret tager array                                                    |
| `set-tags`                                                            | W   | assign, remove, create-assign         | Virker på kontakter og firmaer: `targetType`, `ids`, `add`, `remove` |
| `list-custom-fields`                                                  | R   | list                                  | Kald før skrivning; gæt aldrig et felt-id                            |
| `create-custom-field` / `update-custom-field` / `delete-custom-field` | W   | create, update, delete, reorder       | Rækkefølge er et felt i `update`                                     |
| `list-favorites` / `save-favorite` / `delete-favorite`                | R/W | favorites                             | Gemte visninger; afklares (se dex-context)                           |

### Firmaer

| Action                                                                          |     | Dækker i Dex                                            | Noter                                                              |
| ------------------------------------------------------------------------------- | --- | ------------------------------------------------------- | ------------------------------------------------------------------ |
| `list-companies`                                                                | R   | list, search, filter, count, stats                      | Samme filtermønster som kontakter                                  |
| `get-company`                                                                   | R   | get, contacts, team, activity, title-changes, org-chart | `include` styrer hvilke dele der hentes                            |
| `create-companies` / `update-company` / `update-companies` / `delete-companies` | W   | create, bulk, archive, restore, delete                  | Arkiv/gendan er et felt i patch                                    |
| `merge-companies`                                                               | W   | merge-companies                                         |                                                                    |
| `set-company-role`                                                              | W   | company-roles (alle 8)                                  | Én action: tilføj, afslut, genåbn, ret, gør primær via `operation` |
| `update-org-chart`                                                              | W   | org-chart add/remove/update                             |                                                                    |

### Tidslinje og noter

| Action                                                                                       |     | Dækker i Dex                               | Noter                                                        |
| -------------------------------------------------------------------------------------------- | --- | ------------------------------------------ | ------------------------------------------------------------ |
| `list-timeline`                                                                              | R   | list notes, search-notes, unified timeline | Samlet tidslinje for kontakt eller firma; `query` søger      |
| `create-notes` / `update-note` / `delete-note`                                               | W   | create, create-many, update, delete        |                                                              |
| `list-meeting-types` / `create-meeting-type` / `update-meeting-type` / `delete-meeting-type` | R/W | meeting-types                              | Overvej én `manage-meeting-type` hvis listen forbliver lille |

### Hold kontakten og påmindelser

| Action                                                                        |     | Dækker i Dex                                                | Noter                                               |
| ----------------------------------------------------------------------------- | --- | ----------------------------------------------------------- | --------------------------------------------------- |
| `get-agenda`                                                                  | R   | agenda, birthdays, due cadences, reminders, attention items | Landingssidens data: i dag og de næste dage         |
| `set-keep-in-touch`                                                           | W   | frequencies, cadences (create/change/delete/bulk)           | Frekvens på kontakter eller grupper; `null` fjerner |
| `update-cadence`                                                              | W   | mark-met, snooze                                            | `operation: "met" \| "snooze"`, `until` ved snooze  |
| `list-reminders` / `create-reminders` / `update-reminder` / `delete-reminder` | R/W | reminders inkl. recurring og add-to-many                    | Fuldført er et felt i `update-reminder`             |

### Kalender, mail og opmærksomhed

| Action                                           |     | Dækker i Dex                    | Noter                                                                     |
| ------------------------------------------------ | --- | ------------------------------- | ------------------------------------------------------------------------- |
| `list-calendar-events` / `get-calendar-event`    | R   | agenda calendar-events          | Via workspace-forbindelse til Google/Microsoft, ikke egne tokens          |
| `list-attention-items` / `update-attention-item` | R/W | attention-items list, mark-done | Udkast til svar er et agent-øjeblik, ikke en action                       |
| `send-reply`                                     | W   | send-a-reply                    | Ekstern skrivning: kræver review af den præcise besked og `needsApproval` |

### Import, eksport og integration

| Action / mekanisme                            | Dækker i Dex                             | Noter                                                                                                                                      |
| --------------------------------------------- | ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `plan-import` → `apply-import`                | companies CSV plan/apply, contact import | To-faset: plan viser præcis diff, apply tager kun godkendte rækker. Selve filen uploades via upload-undtagelsen, ikke base64 i SQL         |
| `export-contacts`                             | export CSV                               | Binært/CSV svar er en tilladt rute-undtagelse                                                                                              |
| `provider-api-catalog` / `-docs` / `-request` | Hele Dex API'et                          | Til at importere fra en rigtig Dex-konto og til ad hoc-læsninger. Nøglen ligger i en workspace-forbindelse i vault, aldrig i `process.env` |
| `list-title-changes`                          | title-changes                            | Afhænger af LinkedIn-kilde; senere fase                                                                                                    |

Framework-actions vi får gratis: `navigate`, `view-screen`, `tool-search`,
`db-query` (dev), `upload-image`.

## Datamodel (PostgreSQL / Drizzle)

Første udkast. Alle tabeller med brugerdata får `ownableColumns()`; id'er er
tekst fra `crypto.randomUUID()`; tidsstempler er UTC ISO.

```text
contacts            id, owner, first_name, last_name, job_title, company_id?,
                    description, birthday?, birthday_year?, first_met_at?,
                    starred, archived_at?, priority_tier?, source,
                    last_interaction_at?, created_at, updated_at
contact_channels    id, contact_id, kind (email|phone|address|social),
                    label, value, position
contact_relations   id, source_contact_id, target_contact_id, kind
companies           id, owner, name, domain?, archived_at?, ...
company_roles       id, contact_id, company_id, title, started_at?, ended_at?,
                    is_primary
groups / group_members          id, name, color · group_id, contact_id
tags / taggings                 id, name · tag_id, target_type, target_id
custom_fields / custom_field_values
                    id, name, type, position · field_id, target_type,
                    target_id, value
notes               id, contact_id?, company_id?, body, occurred_at,
                    meeting_type_id?, emoji?
meeting_types       id, name, emoji
keep_in_touch       id, target_type (contact|group), target_id, every_days,
                    next_due_at, snoozed_until?, last_met_at?
reminders           id, contact_ids[], body, due_at, recurrence?, done_at?
interactions        id, contact_id, channel, occurred_at, snippet?, link?
                    (spejl af mail/kalender/beskeder: kun metadata og et kort
                    uddrag, aldrig rå beskeder)
favorites           id, name, filter (json), position
```

`interactions` erstatter Dex' mange `*_last_message_*`-felter på kontakten:
én række pr. interaktion, og "seneste pr. kanal" er en forespørgsel. Ingen rå
mail- eller beskedindhold i SQL (`storing-data`).

## Application state

Agenten skal vide, hvad brugeren ser (`context-awareness`):

- `navigation`: `{ view: "agenda" | "contacts" | "contact" | "companies" |
"company" | "groups" | "settings", id?, filter? }`
- `selection`: valgte kontakt-id'er i listen (til bulk-handlinger og
  "gør dette for de valgte").
- `navigate` og `view-screen` bruges af agenten til at flytte og læse visningen.

## Agent-øjeblikke (højst tre)

Hvert øjeblik har en deterministisk del (actions) og en vurderingsdel
(agenten). Knapperne kalder `sendToAgentChat({ openSidebar: true,
chatTarget: "local", context: { ids, kort resumé }, submit: true })`.

1. **Research kontakt** (på kontaktsiden): agenten læser kontakt, noter og
   offentlige kilder og skriver et resumé tilbage med `update-contact`
   (svarer til Dex Research / `web_search_summary`).
2. **Udkast til opfølgning** (på en forfalden kadence eller et attention
   item): agenten læser tidslinjen og skriver et udkast. Afsendelse sker kun
   via `send-reply` efter brugerens review.
3. **Forbered møde** (på en kalenderbegivenhed): agenten samler deltagernes
   kontekst til en kort brief på begivenheden.

## UI (forslag, afklares i App design)

- **Landingsside (`app.homePath`): Agenda** som en triage-kø (arketype A1):
  forfaldne kadencer, påmindelser, fødselsdage og attention items for i dag.
- **Kontakter** som directory (arketype A10) med gemte visninger, filtre og
  bulk-handlinger; rækkeklik åbner profilen med tidslinje.
- **Firmaer** som directory med firmaside (team, roller, org-diagram).
- shadcn/ui-primitiver, Tabler-ikoner, optimistiske opdateringer med rollback,
  skeletter i layoutets form, ingen gentagne sidetitler eller stat-strips.

## Bevidst udskudt

LinkedIn-synk og titelskift, beskedkanaler (WhatsApp, iMessage, Instagram,
Facebook), browserudvidelse, mobil-app, teams/workspaces og MCP-eksponering.
De står i feature-træet i [dex-context.md](dex-context.md) og planlægges, når
kernen virker.
