# Design tokens: sizing, typografie, spacing, kleur

Bron: Client-First *Sizes and rem*, *Typography strategy*, *Spacing strategy*, *Variables (color)*.

## 1. Rem + 4pt-systeem blijft ongewijzigd

CF's kernregel blijft 1-op-1 geldig in Astro: **gebruik altijd `rem`, nooit `px`, `vw` of `vh`** voor typografie, spacing en de meeste maten. Reden blijft identiek: rem respecteert browser-fontsize-instellingen en zoom, `px`/`vw`/`vh` niet — dit is een a11y-vereiste, geen stijlvoorkeur.

- Basis: `16px = 1rem`.
- Gebruik "ronde" rem-waarden: `0.5, 1, 1.5, 2, 2.5, 3, 4, 5, 6, 8` — vermijd waarden als `8.4375rem`.
- Conversie: `px / 16 = rem`.

```css
/* tokens.css */
:root {
  --space-1: 0.5rem;   /* 8px */
  --space-2: 1rem;     /* 16px */
  --space-3: 1.5rem;   /* 24px */
  --space-4: 2rem;     /* 32px */
  --space-5: 3rem;     /* 48px */
  --space-6: 4rem;     /* 64px */
  --space-7: 6rem;     /* 96px */
  --space-8: 8rem;     /* 128px */
}
```

## 2. Utility classes → CSS custom properties (1-op-1 mapping)

CF's utility classes zijn stuk voor stuk niets anders dan een benoemde CSS-waarde. In Astro slaan we die waarde rechtstreeks op als token, zonder de class-laag ertussen:

| CF utility class | `tokens.css` token |
|---|---|
| `text-size-small/medium/large/xl` | `--font-size-sm/md/lg/xl` |
| `text-color-primary/secondary/brand/neutral` | `--color-text-primary/secondary/brand/neutral` |
| `text-weight-regular/semibold/bold` | `--font-weight-regular/semibold/bold` |
| `heading-style-h1..h6` | (zie §3, gebruik base HTML-tag styling) |
| `margin-[direction]` + `margin-[size]` | `--space-1` t/m `--space-8` (toegepast via component-CSS, niet als losse class) |
| `padding-global` | `--space-page-inline` (horizontal page padding) |
| `container-small/medium/large` | `--container-sm/md/lg` (max-width) |
| `padding-section-small/medium/large` | `--space-section-sm/md/lg` (vertical section padding, op eigen div ónder de sectie — zie `core-structure-and-components.md`) |
| `shadow-small/medium/large` | `--shadow-sm/md/lg` |
| `radius-*` | `--radius-sm/md/lg/full` |

## 3. Typografie: default HTML-tags eerst, tokens pas bij afwijking

CF's belangrijkste typografie-regel: **idealiter staat er geen enkele class op een tekstelement.** De default styling hoort op de HTML-tag zelf (`body`, `p`, `h1`–`h6`). Een token/prop wordt pas toegevoegd bij een *afwijking* van de default.

Vertaling naar Astro:

```css
/* tokens.css of global.css — basisstijl direct op de tags, geen component nodig */
body { font-family: var(--font-body); font-size: var(--font-size-md); line-height: 1.5; }
h1 { font-size: var(--font-size-4xl); font-family: var(--font-display); }
h2 { font-size: var(--font-size-3xl); font-family: var(--font-display); }
/* ... */
```

Alleen wanneer een tekstinstantie *afwijkt* van de default (bijv. een SEO-verplichte `<h1>` die er als `<h2>` uit moet zien) grijp je naar een prop/utility, nooit naar een nieuw custom-tekstcomponent:

```astro
<h1 class="text-style-h2">{title}</h1>
```

waarbij `text-style-h2` een utility-class is die dezelfde token-set als `h2` hergebruikt — exact CF's `heading-style-h2`-patroon, nu als losse utility-class i.p.v. Webflow-class, herbruikbaar op elk tekstelement.

## 4. Spacing: tokens toepassen op het component, geen "spacing blocks/wrappers" nodig

CF gebruikt lege `spacing-block`/`spacing-wrapper` divs omdat Webflow geen component-composition heeft — elke visuele laag moet een eigen element zijn. In Astro is dat overbodig: spacing hoort in de **CSS van het component zelf** (bijv. via `gap` op een flex/grid-parent, of `margin` in scoped `<style>`), nooit als losse lege div.

- CF "spacing via margin/padding utility classes direct op custom class" → blijft identiek: pas `--space-*` tokens toe in de component-CSS.
- CF "CSS Grid strategy" (gap i.p.v. losse spacing-elementen per kind) → **dit is de aanbevolen default in Astro**: gebruik `gap` met een `--space-*` token op de parent, in plaats van marges op kinderen.

```astro
<style>
  .list { display: flex; flex-direction: column; gap: var(--space-3); }
</style>
```

- Vermijd CF's "deep stacking"-probleem (typografie- + spacing-classes samen op een tekstelement) door spacing op de **container/wrapper** te zetten via `gap`, en typografie-tokens puur op het tekstelement zelf te houden.

## 5. Kleur: primitive tokens + semantische tokens (CF *Variables*)

CF onderscheidt **primitive tokens** (ruwe kleurwaarden, meest granulaire laag) en **semantische tokens** (betekenisvolle namen die naar een primitive verwijzen). Deze twee-laags aanpak vertaalt direct naar CSS custom properties:

```css
:root {
  /* Primitive tokens — ruwe paletwaarden, nooit direct in componenten gebruiken */
  --color-blue-500: #2563eb;
  --color-gray-100: #f5f5f4;
  --color-gray-900: #1c1917;

  /* Semantische tokens — dit gebruik je in componenten */
  --color-primary: var(--color-blue-500);
  --color-text-primary: var(--color-gray-900);
  --color-bg-page: var(--color-gray-100);
}
```

Regel (identiek aan CF): **componenten refereren altijd naar semantische tokens, nooit naar primitives.** Zo blijft een globale herkleuring (bijv. rebranding) een wijziging op één plek.

### Type-safe tokens met TypeScript

Waar CF stopt bij naamgeving, voegen we in deze stack een TS-laag toe zodat token-namen autocomplete/type-checking krijgen in `.astro`-bestanden:

```ts
// src/lib/tokens.ts
export const colorToken = {
  primary: 'var(--color-primary)',
  textPrimary: 'var(--color-text-primary)',
  bgPage: 'var(--color-bg-page)',
} as const;

export type ColorToken = keyof typeof colorToken;
```

Gebruik dit vooral in componenten die kleur als prop doorgeven (bijv. een `<Icon color="primary" />`) zodat een typo in een tokennaam een build-error geeft in plaats van een stille CSS-fout.

## 6. Checklist

- [ ] Geen magic numbers — elke maat verwijst naar een `--space-*`, `--font-size-*`, of ander token.
- [ ] Geen `px`/`vw`/`vh` voor typografie/spacing — alleen `rem` (of `clamp()` met rem, zie `responsive-and-motion.md`).
- [ ] Kleur altijd via semantisch token, nooit via primitive of hex direct in een component.
- [ ] Tekstelementen hebben bij voorkeur géén eigen class — alleen bij afwijking van de HTML-tag default.
