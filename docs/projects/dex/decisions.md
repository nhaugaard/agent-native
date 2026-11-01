# Beslutninger

Nyeste øverst. En beslutning har dato, hvad der blev valgt, og hvorfor. Åbne
spørgsmål står nederst med en anbefaling.

## Truffet

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
   `create <slug> --standalone --template chat` i sit eget repo, eller som en
   app i et Agent-Native-workspace, når flere apps skal kombineres. At lægge
   den i `templates/` i denne fork blander vores app med frameworkets
   offentlige skabeloner og gør upstream-opdateringer sværere.
2. **Chat- eller CRM-skabelon som udgangspunkt?** Anbefaling: `chat` (som
   `turn-into-app` foreskriver), med `templates/crm` som reference for
   dubletter, fletning og saved views.
3. **Skal vi kunne importere fra en rigtig Dex-konto?** Anbefaling: ja, via
   provider-API-mønsteret med nøglen i en workspace-forbindelse.
4. **Navn på appen** (slug).
