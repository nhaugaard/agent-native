# Planlægning med /visual-plan i AI App

Hvordan threads i AI App-projektet (vores Dex-inspirerede app på Agent-Native)
bruger den hostede Plan-MCP til planer, recaps og review.

Levende version med diagram: [Planlægning med /visual-plan i AI App](https://plan.agent-native.com/plans/plan-5362de37705a4d83)
(privat plan i Nikolajs Plan-konto).

## Status

Verificeret 1. november 2026 fra en cloud-thread:

- Connectoren hedder `Plan`, og dens tools er deferred (`mcp__Plan__*`).
  Indlæs dem med ToolSearch, fx `select:mcp__Plan__create-visual-plan`.
- Connectoren er logget ind som Nikolaj. Nye planer lander i hans konto som
  private planer.
- `list-visual-plans`, `get-visual-plan`, `get-plan-blocks`,
  `create-visual-plan` og `update-visual-plan` virker uden ekstra login.

## Flow

1. Indlæs skill'et `visual-plan` (eller `visual-recap`), og læs `CLAUDE.md`,
   `AGENTS.md` og den relevante skill i `.agents/skills/`.
2. Research repoet. Planlægning er read-only.
3. Kald `get-plan-blocks`, og opret derefter præcis én plan med det
   værktøj, der passer til opgaven (tabellen nedenfor).
4. Post `openLink.webUrl` fra svaret i threaden. Feltet `url` fra
   `create-visual-plan` er kun en relativ sti (`/plans/<id>`) og virker ikke
   som link.
5. Nikolaj kommenterer i planen. Hent kommentarerne med `get-plan-feedback`,
   ret med `update-visual-plan` og `contentPatches`, og verificér med
   `get-visual-plan`.

| Opgave                                       | Værktøj                              |
| -------------------------------------------- | ------------------------------------ |
| Arkitektur, data, actions, backend           | `create-visual-plan` (ingen canvas)  |
| UI-skærme og tilstande                       | `create-ui-plan`                     |
| Klikbart flow                                | `create-prototype-plan`              |
| Fuld branded design (kun på eksplicit ønske) | `create-plan-design`                 |
| Opsummering af en PR eller branch            | skill `visual-recap`                 |
| Diagrammer til rapporter og artifacts        | skill `diagram-design`, standardtema |

## Browser

Globe-ikonet i thread-vinduet åbner Claude-appens indbyggede browser ("Browse
with Claude"). Agenten kan kun styre den, hvis sessionen har fået
browser-tools: `mcp__remote-devices__Claude_Browser__*` eller et enkelt
`enable__mcp__remote-devices__Claude_Browser`, som slår dem til.

Tjek ved opgavestart med ToolSearch (`+Claude_Browser`):

- Tools findes: indlæs skill'et `built-in-browser`, indlæs alle tools i ét
  kald, og åbn planen med `preview_start` og `url`.
- Ingen tools: post planens URL i threaden, og Nikolaj åbner den i
  browser-ruden.

Pr. 1. november 2026 fik en cloud-thread ingen browser-tools, heller ikke
efter at ruden var åben og sessionen var linket til Nikolajs Mac.

Åbn aldrig Chrome på Nikolajs lokale system, heller ikke via computer use
eller `open`-kommandoer.

## Fejl

- Tools ikke synlige: søg med ToolSearch efter `create-visual-plan`.
  Connectoren kan også hedde `plan` eller `agent-native-plans`.
- `needs auth`, `Unauthorized` eller `Session terminated`: stop med at prøve
  igen, og bed koordinatoren få Nikolaj til at genforbinde Plan-connectoren i
  Claude-appens connector-indstillinger. Lever aldrig planen som Markdown i
  chatten i stedet.
