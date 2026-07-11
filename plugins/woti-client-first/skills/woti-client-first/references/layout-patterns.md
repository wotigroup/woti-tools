# Layout binnen de container: grid & kolommen

Bron: Client-First *Spacing strategy* (CSS Grid-strategie), *Core structure strategy* (container-onafhankelijkheid).

Binnen `Container.astro` (zie `core-structure-and-components.md`) volgt meestal nog een extra layout-laag: een grid met kaarten, of twee kolommen naast elkaar. CF noemt dit kort de "CSS Grid strategy" — één `gap`-eigenschap op een gedeelde parent in plaats van losse spacing-blocks/wrappers per kind-element (zie `tokens-and-sizing.md` §4). Dit bestand werkt dat uit tot een herbruikbaar patroon.

## 1. Eerst de vraag: utility of custom?

Dezelfde beslisboom als in `naming-and-file-structure.md` §1 geldt hier direct:

| Situatie | Type | Voorbeeld |
|---|---|---|
| Standaard grid-patroon, overal in het project herbruikt (2 kolommen, 3 kolommen, auto-fit kaarten) | **Utility** → `Grid.astro`-component | `grid-2-col`, `grid-3-col`, `grid-auto-col` |
| Eenmalige, unieke verhouding specifiek voor één sectie (bijv. 60/40 i.p.v. 50/50) | **Custom class**, in het scoped `<style>`-block van dat sectie-component | `.about_grid { grid-template-columns: 3fr 2fr; }` |

Vuistregel (CF classes-strategy-1): zodra een grid-verhouding op een tweede plek hergebruikt wordt, promoveer je 'm van custom naar een utility-variant op `Grid.astro` — niet andersom.

## 2. `Grid.astro` — de utility-component

```astro
---
// src/components/ui/Grid.astro
interface Props {
  columns?: 2 | 3 | 4 | 'auto';
  gap?: 'small' | 'medium' | 'large';
}
const { columns = 2, gap = 'medium' }: Props = Astro.props;
---
<div class:list={['grid', `grid-${columns}-col`, `grid-gap-${gap}`]}>
  <slot />
</div>
```

```css
.grid { display: grid; }

.grid-gap-small  { gap: var(--space-3); }
.grid-gap-medium { gap: var(--space-4); }
.grid-gap-large  { gap: var(--space-5); }

.grid-2-col    { grid-template-columns: repeat(2, 1fr); }
.grid-3-col    { grid-template-columns: repeat(3, 1fr); }
.grid-4-col    { grid-template-columns: repeat(4, 1fr); }
.grid-auto-col { grid-template-columns: repeat(auto-fit, minmax(16rem, 1fr)); }

/* Responsive: alle vaste kolom-aantallen vallen terug naar 1 kolom op mobiel.
   grid-auto-col hoeft dit niet — auto-fit lost dat al zelf op. */
@media (max-width: 47.99rem) {
  .grid-2-col, .grid-3-col, .grid-4-col { grid-template-columns: 1fr; }
}
```

- `gap` gebruikt dezelfde `--space-*`-tokens als de rest van het project — geen aparte grid-specifieke schaal.
- `grid-auto-col` (auto-fit) is de aanbevolen default voor kaarten-achtige content (services, reviews, team) waarvan het aantal items kan wisselen — dit voorkomt lege cellen of handmatige breakpoint-aanpassingen per project.

## 3. Voorbeeld A: 50/50-kolommen (tekst + afbeelding)

Dit is een **utility**-geval: een gelijke 2-koloms-verdeling komt in vrijwel elk project op meerdere plekken terug.

```astro
---
// binnen AboutContent.astro, gebruikt binnen <Container> uit het core-structure-voorbeeld
import Grid from '../ui/Grid.astro';
---
<Grid columns={2} gap="large">
  <div class="about_content">
    <h2>Over Omit Design</h2>
    <p class="text-color-neutral">Meppel-based studio, gespecialiseerd in...</p>
  </div>
  <div class="about_image-wrapper">
    <img src="/team.jpg" alt="Het team van Omit Design" />
  </div>
</Grid>
```

Gerenderde HTML — dit vervangt de inhoud van de `about`-sectie uit het eerdere voorbeeld:

```html
<section id="about" class="section">
  <div class="padding-global padding-section-medium">
    <div class="container container-medium">
      <div class="grid grid-2-col grid-gap-large">
        <div class="about_content">
          <h2>Over Omit Design</h2>
          <p class="text-color-neutral">Meppel-based studio, gespecialiseerd in...</p>
        </div>
        <div class="about_image-wrapper">
          <img src="/team.jpg" alt="Het team van Omit Design" />
        </div>
      </div>
    </div>
  </div>
</section>
```

| Element | Class(es) | Doel |
|---|---|---|
| `<div class="grid grid-2-col grid-gap-large">` | `grid` (display:grid), `grid-2-col` (2 gelijke kolommen), `grid-gap-large` (ruimte tussen de kolommen) | Utility-grid, herbruikbaar overal waar een gelijke 2-koloms-split nodig is |
| `<div class="about_content">` | **custom class** | Specifiek voor deze sectie, bevat mogelijk unieke interne spacing tussen `h2`/`p` die geen aparte utility rechtvaardigt |
| `<div class="about_image-wrapper">` | **custom class** | Wrapper om de afbeelding, bijv. voor `border-radius`/`aspect-ratio` specifiek aan deze sectie |

## 4. Voorbeeld B: grid met kaarten (auto-fit, wisselend aantal items)

```astro
---
import Grid from '../ui/Grid.astro';
import ServiceCard from './ServiceCard.astro';
---
<Grid columns="auto" gap="medium">
  {services.map((service) => <ServiceCard {...service} />)}
</Grid>
```

```html
<div class="grid grid-auto-col grid-gap-medium">
  <div class="service_card">…</div>
  <div class="service_card">…</div>
  <div class="service_card">…</div>
</div>
```

`ServiceCard.astro` is hier zelf een herbruikbaar component (`/components/ui/ServiceCard.astro` als het generiek is, of `/components/content-blocks/ServiceCard.astro` als het uit Keystatic-content komt) — de kaart zelf regelt zijn eigen interne padding/typografie, de `Grid` regelt alleen de buitenste rangschikking. Dit is dezelfde scheiding als tussen `Container` (buitenkant) en de content erin (zie `core-structure-and-components.md`).

## 5. Eenmalige/afwijkende verhouding: custom class in plaats van `Grid`

Als een sectie een unieke, niet-herbruikte verhouding nodig heeft (bijv. 60/40 in plaats van 50/50), voeg je dat **niet** toe als nieuwe `Grid`-variant, maar als custom class in het component zelf:

```astro
---
// Hero.astro
---
<div class="hero_grid">
  <div class="hero_content">…</div>
  <div class="hero_visual">…</div>
</div>

<style>
  .hero_grid {
    display: grid;
    grid-template-columns: 3fr 2fr;
    gap: var(--space-5);
  }
  @media (max-width: 47.99rem) {
    .hero_grid { grid-template-columns: 1fr; }
  }
</style>
```

Dit gebruikt nog steeds de gedeelde `--space-*`-tokens (geen magic numbers), maar de grid-verhouding zelf blijft lokaal — precies CF's regel dat een eenmalige combinatie geen eigen globale utility-class verdient.

## 6. Checklist

- [ ] Terugkerende grid-verhoudingen (2/3/4 kolommen, auto-fit) lopen via `Grid.astro`, niet via losse CSS per sectie.
- [ ] Eenmalige/unieke verhoudingen zijn een custom class in het scoped `<style>`-block van dat specifieke component.
- [ ] `gap` gebruikt altijd een `--space-*`-token, nooit een losse waarde.
- [ ] Vaste kolom-aantallen (`grid-2/3/4-col`) hebben een mobiele fallback naar 1 kolom; `grid-auto-col` lost dit zelf op via `auto-fit`.
- [ ] De content ín een grid-cel (kaart, tekstblok, afbeelding) is een eigen component — `Grid` regelt uitsluitend de rangschikking, niet de inhoud.
