---
name: woti-client-first
description: Vertaalt Finsweet's Client-First naamgevings- en structuurmethodologie (oorspronkelijk voor Webflow) naar een component-based Astro + TypeScript workflow met CSS custom properties als design tokens. Gebruik deze skill bij het bouwen, structureren of beoordelen van nieuwe Astro-websites en -componenten: paginastructuur, design tokens, component-architectuur, props/varianten, responsive/fluid sizing, interactie-naamgeving, semantische HTML en toegankelijkheid. Trigger dit ALTIJD wanneer een nieuwe Astro-website of -component wordt opgezet, wanneer klassen/props/bestanden genoemd of hernoemd moeten worden, of wanneer de gebruiker verwijst naar "Client-First", "CF", "de methodologie", "de style guide", of naar dit project se conventies voor structuur/naamgeving. Gebruik ook wanneer bestaande Client-First/Webflow-projecten of -content omgezet moeten worden naar de Astro-stack.
---

# Client-First → Astro + TypeScript

Dit is een 1-op-1 vertaling van [Finsweet's Client-First](https://finsweet.com/client-first/docs/) methodologie — origineel geschreven voor het class-based, canvas-only bouwen in Webflow — naar een **component-based Astro + TypeScript** stack met CSS custom properties als design tokens.

## Waarom deze vertaling nodig is

Client-First lost een probleem op dat in Webflow bestaat maar in Astro grotendeels al is opgelost: Webflow heeft geen echte componenten, dus CF gebruikt classnaming (`_` voor custom classes, `-` voor utility classes, `is-` voor combo/variant classes) om orde, herbruikbaarheid en leesbaarheid te simuleren op wat in de kern losse `<div>`'s zijn.

Astro heeft **wél** echte componenten met **props, TypeScript-types en scoped styles**. We hoeven dus niet Webflow's classname-trucs te kopiëren — we vertalen de *onderliggende intentie* van elk CF-principe naar het Astro/TS-equivalent dat hetzelfde doel dient:

| Client-First doel | Webflow-mechanisme | Astro + TS-equivalent |
|---|---|---|
| Voorspelbare, leesbare naamgeving | `_` en `-` in classnamen | Bestandsnamen + component-namen + prop-namen (zie `references/naming-and-file-structure.md`) |
| Herbruikbare stijl-bouwstenen | Utility classes | CSS custom properties (design tokens) in `tokens.css` (zie `references/tokens-and-sizing.md`) |
| Consistente basisstructuur per pagina | Core Structure classes (`page-wrapper`, `main-wrapper`, `section_`, `container-*`, `padding-global`) | `Layout.astro`, `Section.astro`, `Container.astro` componenten (zie `references/core-structure-and-components.md`) |
| Variaties op een basis-stijl | Combo classes (`is-*`) | Typed props met union types (`variant`, `size`, `state`) i.p.v. gestapelde classes |
| Voorkomen van "deep stacking" | Losse nested divs per stijl-laag | Component-samenstelling (composition) + scoped CSS per component |
| Non-technisch beheerbaar | Beschrijvende classnamen | Beschrijvende component- en prop-namen + `design-system.md` manifest (al onderdeel van STACK.md) |
| Schaalbare, zoekbare organisatie | Virtuele Folders (`_`) | Mappenstructuur (`/components/ui`, `/sections`, `/content-blocks`) — al aanwezig in STACK.md |
| Toegankelijke, schaalbare maten | rem + 4pt-systeem | CSS custom properties in rem, zelfde 4pt-schaal |
| Responsive scaling | "Root-font scaling" via gegenereerde CSS | `clamp()`-gebaseerde fluid tokens (zie `references/responsive-and-motion.md`) |
| Consistente interactie-namen | Webflow Interactions naamgeving | `data-*`-attributen + TS-functienamen (zie `references/responsive-and-motion.md`) |
| Correcte class-volgorde i.v.m. CSS-specificity | Volgorde van class-aanmaak in Webflow | CSS `@layer`-strategie (zie `references/css-architecture.md`) |
| Semantiek en toegankelijkheid | Correcte HTML-tags in Designer | Semantische Astro-templates + a11y-checklist (zie `references/accessibility-and-semantics.md`) |

**Belangrijk:** we kopiëren geen Webflow-specifieke syntax (geen `_` in class-namen, geen `is-` combo classes) — Astro's eigen taalconstructies (bestanden, props, TypeScript types) vervangen die rol volledig. Wat we wél 1-op-1 overnemen: de *denkwijze* (heldere namen, geen deep stacking, globale tokens, voorspelbare core structure, rem-gebaseerde schaal, a11y-first).

## Verhouding tot STACK.md

`STACK.md` (project-root) blijft leidend voor de kernstack, mappenstructuur en deploy-workflow. Deze skill is het **naslagwerk voor naamgeving en structuur** dat je raadpleegt zodra je:
- een nieuwe pagina, sectie of component opzet
- design tokens (`tokens.css`) opzet of uitbreidt
- twijfelt over een component-, prop-, of bestandsnaam
- een bestaand Client-First/Webflow-ontwerp (bijv. uit Paper.design of een oude Webflow-export) omzet naar Astro

STACK.md's naamgevingsregel (`ComponentNaam/Variant/State`, bijv. `Card/Featured/Hover` → `Card.astro` met `variant="featured"`) is al vrijwel identiek aan Client-First's combo-class-principe (`card is-featured`). Deze skill werkt die regel verder uit voor alle CF-domeinen (structuur, spacing, typografie, interacties, a11y).

## Workflow

**Nieuwe pagina/sectie bouwen:**
1. Lees `references/core-structure-and-components.md` voor de basis page/section/container-opbouw.
2. Check `tokens.css` en `design-system.md` (STACK.md §3) op bestaande tokens/componenten vóór je nieuwe aanmaakt.
3. Gebruik `references/naming-and-file-structure.md` om component- en propnamen te bepalen.
4. Pas `references/tokens-and-sizing.md` toe voor elke maat/kleur/spacing-waarde — nooit magic numbers.
5. **Nieuwe HTML/content komt altijd tussen `<Container>...</Container>`, nooit ervoor, erna, of los in de sectie-/padding-laag** — zie `references/core-structure-and-components.md` §4.

**Nieuw component bouwen:**
1. Bepaal of het een `ui`, `sections`, of `content-blocks` component is (STACK.md §2).
2. Ontwerp de Props-interface volgens `references/naming-and-file-structure.md` (variant/size/state als typed unions, geen booleans zoals `isLarge`).
3. Implementeer alle states (default/hover/active/focus/disabled) — zie `references/accessibility-and-semantics.md`.
4. Gebruik `references/responsive-and-motion.md` voor fluid sizing en interactie-naamgeving.

**Bestaand Webflow/Client-First-project omzetten naar Astro:**
1. Inventariseer de bestaande CF-classes (core structure, utility classes, custom classes, combo classes).
2. Vertaal utility classes → tokens in `tokens.css` (`references/tokens-and-sizing.md`).
3. Vertaal custom classes/componenten → Astro-componenten met Props (`references/naming-and-file-structure.md`).
4. Vertaal combo classes (`is-*`) → variant-props, niet naar losse CSS-classes.
5. Niets overslaan: loop alle secties van deze skill langs (structure, typography, spacing, folders/naming, variables, interactions, fluid responsive, semantic HTML, accessibility, CSS-specificity) en vink af wat vertaald is.

## Reference-bestanden

| Bestand | Client-First bronsecties | Wanneer lezen |
|---|---|---|
| `references/naming-and-file-structure.md` | Classes strategy 1 & 2, Folders, Folders strategy | Bij het benoemen van componenten, props, bestanden, mappen |
| `references/tokens-and-sizing.md` | Sizes and rem, Typography strategy, Spacing strategy, Variables (color) | Bij het opzetten/uitbreiden van `tokens.css` |
| `references/core-structure-and-components.md` | Core structure strategy, Utility class systems | Bij het bouwen van `Layout.astro`, `Section.astro`, `Container.astro` |
| `references/layout-patterns.md` | Spacing strategy (CSS Grid-strategie) | Bij content-layout ín de container: grid, kolommen, kaarten |
| `references/responsive-and-motion.md` | Fluid responsive, Interactions naming | Bij fluid sizing, animaties, view transitions |
| `references/accessibility-and-semantics.md` | Semantic HTML tags, Accessibility | Bij elk interactief component en elke pagina-structuur (CF's eigen a11y-hoofdstuk vertaald) |
| `references/accessibility-standards.md` | Geen CF-bron — WCAG 2.2 AA + EAA + A11y Project Checklist | Bij projecten die volledig wettelijk/WCAG-conform moeten zijn, niet alleen "CF-stijl toegankelijk". Lees dit vóór oplevering van elk project. |
| `references/css-architecture.md` | CSS specificity | Bij twijfel over style-volgorde/overrides tussen tokens, componenten en variant-styles |
| `references/chapter-summaries.md` | Alle inhoudelijke hoofdstukken (volledige samenvattingen) | Bij twijfel of een CF-principe gemist is, of om de originele redenering achter een regel terug te vinden |

Lees het relevante referentiebestand vóórdat je code genereert — niet alles in één keer laden, alleen wat de taak vereist.
