# Dex-projektet

Vi bygger vores egen app, der samler flere apps i én. Første app er en
personlig relations-CRM inspireret af [Dex](https://getdex.com), bygget på
Agent-Native-frameworket i dette repo. Mappen her er projektets
planlægnings- og vidensbase. Selve app-koden kommer først, når
beslutningerne i [decisions.md](decisions.md) er truffet.

## Start her (for agenter og mennesker)

1. Læs repoets `CLAUDE.md` / `AGENTS.md` (identiske) og den skill, der
   matcher opgaven, under `.agents/skills/`. Reglerne dér vinder over alt i
   denne mappe.
2. Læs [knowledge/agent-native-basics.md](knowledge/agent-native-basics.md)
   for frameworkets byggesten og hvilke skills der ejer hvad.
3. Læs [dex-context.md](dex-context.md) for, hvad Dex er og kan.
4. Læs [dex-api-to-actions.md](dex-api-to-actions.md) for, hvordan Dex' domæne
   oversættes til Agent-Native actions, data og UI.
5. Tjek [decisions.md](decisions.md) for trufne og åbne beslutninger.

## Struktur

| Fil / mappe                                    | Indhold                                                                              | Opdateres                                     |
| ---------------------------------------------- | ------------------------------------------------------------------------------------ | --------------------------------------------- |
| [initial-prompt.md](initial-prompt.md)         | Nikolajs første prompt, ordret                                                       | Aldrig                                        |
| [dex-context.md](dex-context.md)               | Levende viden om Dex: produkt, features, datamodel, API                              | Hver gang vi lærer noget nyt om Dex           |
| [dex-api-to-actions.md](dex-api-to-actions.md) | Mapping fra Dex API til Agent-Native actions, tabeller, app-state og agent-øjeblikke | Når mappingen ændres                          |
| [decisions.md](decisions.md)                   | Beslutningslog og åbne spørgsmål                                                     | Ved hver beslutning                           |
| [knowledge/](knowledge/)                       | Fast agent-viden om Agent-Native                                                     | Når frameworket eller vores forståelse ændres |
| [diagrams/](diagrams/)                         | Diagrammer lavet med `/diagram-design` i standardtemaet (HTML/SVG/PNG)               | Pr. opgave                                    |

## Arbejdsregler i projektet

- Kommunikation går mellem tråde og koordinatoren. Koordinatoren godkender på
  Nikolajs vegne.
- Hver tråd indlæser selv den skill, der passer til opgaven.
- Åbn aldrig Chrome på Nikolajs lokale maskine. Har en tråd ingen indbygget
  browser, sendes URL'en i chatten.
- Diagrammer laves med `/diagram-design` i standardtemaet og lægges i
  [diagrams/](diagrams/) med en kort note om, hvad de viser. Overvej et
  diagram eller skema for hver større opgave.
- Planlægning sker løbende med `/visual-plan` (Plan MCP).
- Planer følger `turn-into-app`-skillens kildebrief og App design-afsnit, så
  de kan oversættes direkte til Agent-Native: actions først, så UI, skills/
  instruktioner og application state (de fire områder i `adding-a-feature`).
- Hold dokumenterne levende: ret det, der er forkert, i stedet for at tilføje
  en ny fil ved siden af. Datér nye fakta om Dex og angiv kilden.
