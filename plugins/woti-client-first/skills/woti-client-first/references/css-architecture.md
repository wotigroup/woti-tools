# CSS-architectuur & specificiteit

Bron: Client-First *CSS Specificity*.

## Het probleem in CF

CF's specificity-probleem is Webflow-specifiek: class-volgorde in Webflow bepaalt CSS-specificiteit (een class die *later* is aangemaakt in het project wint van een class die *eerder* is aangemaakt, bij gelijke specificiteit), waardoor het kopiëren van classes tussen projecten in de verkeerde volgorde stijlen kan breken (bijv. `margin-large` die `margin-bottom`'s richting overschrijft in plaats van aanvult).

## Waarom dit in Astro grotendeels niet optreedt — en wat er wél overblijft

In Astro schrijf je CSS zelf (geen tool die classes in willekeurige volgorde genereert), dus het exacte CF-probleem (bron-volgorde per ongeluk verkeerd door copy-paste) speelt niet op dezelfde manier. Wat wél overblijft: **de onderliggende les — specificiteit/volgorde is iets waar je bewust mee moet omgaan, niet iets dat "vanzelf goed gaat".**

De vertaling naar moderne CSS is `@layer`, waarmee je *expliciet* de override-volgorde vastlegt, onafhankelijk van de bron-volgorde waarin selectors toevallig geschreven zijn — dit is een preciezere oplossing dan CF's "let op de aanmaak-volgorde"-workaround.

```css
/* global.css — layer-volgorde bepaalt override-prioriteit, laatste laag wint */
@layer tokens, base, components, utilities;

@layer tokens {
  :root { --color-primary: #2563eb; /* ... */ }
}

@layer base {
  body { font-family: var(--font-body); }
  h1, h2, h3 { font-family: var(--font-display); }
}

@layer components {
  .button { padding: var(--space-2) var(--space-3); }
}

@layer utilities {
  .text-color-brand { color: var(--color-primary); }
}
```

Met deze layer-volgorde wint een utility-token altijd van component-CSS, en component-CSS altijd van base-styles — ongeacht in welke volgorde de bestanden toevallig geladen worden. Dit is het directe, robuustere equivalent van CF's `margin-[size]`-vóór-`margin-[direction]`-regel: **de gewenste override-volgorde staat expliciet vast, in plaats van impliciet af te hangen van aanmaak-volgorde.**

## Scoped component-styles (Astro-specifiek voordeel)

Astro's `<style>`-block per `.astro`-component is standaard scoped (component-specifieke class-namen worden automatisch uniek gemaakt). Dit elimineert een hele categorie CF-problemen (naam-botsingen tussen componenten, onbedoelde cross-component overrides) die in Webflow's platte, globale class-namespace juist wél kunnen optreden. Gebruik dit voordeel bewust:

- **Tokens** (`tokens.css`): globaal, in `:root`, buiten componenten.
- **Component-specifieke styling**: in het scoped `<style>`-block van het component zelf, verwijzend naar tokens via `var(--token-naam)`.
- **Utility-classes** (zeldzaam nodig dankzij props/variants, zie `naming-and-file-structure.md` §3): globaal gedefinieerd in een `@layer utilities`, spaarzaam gebruikt.

## Checklist

- [ ] Nieuwe globale CSS staat in de juiste `@layer` (tokens → base → components → utilities).
- [ ] Component-specifieke styling staat in het scoped `<style>`-block van het component, niet los in een globaal bestand.
- [ ] Geen impliciete afhankelijkheid van bestands-laadvolgorde voor override-gedrag — altijd expliciet via `@layer` of specificiteit.
