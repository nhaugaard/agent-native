# Beslutninger

Nyeste øverst. En beslutning har dato, hvad der blev valgt, og hvorfor. Åbne
spørgsmål står nederst med en anbefaling.

## Truffet

### 2026-11-01: Agent-Native Workspace er udelukket

Nikolaj har valgt, at appen ikke bygges som et multi-app workspace. Appen
bygges som én selvstændig app.

### 2026-11-01: Projektets dokumenter ligger i `docs/projects/dex/`

Planlægning, Dex-viden og agent-viden samles ét sted i repoet, så alle tråde
læser den samme kilde, og ændringer kan reviewes i PR'er. Selve app-koden
placeres efter beslutning 1 nedenfor.

### 2026-11-01: Actions følger Agent-Native, ikke Dex' endpoints

Dex' API bruges som kort over produktets evner. Overfladen holdes lille og
ortogonal (~40 actions mod Dex' ~250 endpoints), jf.
[dex-api-to-actions.md](dex-api-to-actions.md).

### 2026-11-01: Diagrammer laves med `/diagram-design` i standardtemaet

De gemmes i [diagrams/](diagrams/) og indgår i artifacts og rapporter.

## Åbne spørgsmål

1. **Hvor bor app-koden?** Anbefaling: en selvstændig app oprettet med
   `create <slug> --standalone` i sit eget repo. At lægge den i `templates/`
   i denne fork blander vores app med frameworkets offentlige skabeloner og
   gør upstream-opdateringer sværere. Et Agent-Native-workspace er udelukket
   (se ovenfor).
2. **Chat- eller CRM-skabelon som udgangspunkt?** Åbent; Nikolaj afgør det
   undervejs. `turn-into-app` foreskriver `chat`, mens gennemgangen af
   `templates/crm` viser, at den allerede dækker det meste af kernen
   (kontakter, lister, saved views, dubletter og fletning).
3. **Skal vi kunne importere fra en rigtig Dex-konto?** Anbefaling: ja, via
   provider-API-mønsteret med nøglen i en workspace-forbindelse.
4. **Navn på appen** (slug).
