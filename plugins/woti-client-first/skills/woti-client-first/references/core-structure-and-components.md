# Core structure → Layout / Section / Container componenten

Bron: Client-First *Core structure strategy*, *Utility class systems*.

CF definieert 6 vaste "lagen" rond de pagina-inhoud: `page-wrapper` → `main-wrapper` → `section_[naam]` → `padding-global` + `padding-section-[size]` → `container-[size]`. Elke laag krijgt hier een direct Astro-component-equivalent, één keer gebouwd, overal hergebruikt.

**Belangrijk om vooraf te begrijpen:** de *componenten/props/bestandsnamen* volgen Astro/TypeScript-conventies (PascalCase-bestanden, camelCase-props). De *CSS-classnamen die deze componenten renderen in de uiteindelijke HTML* zijn waar de CF-naamgeving grotendeels behouden blijft — zie de toelichting onderaan dit bestand ("Wat is 'hetzelfde als CF' en wat niet").

## 1. Laag-voor-laag: component ↔ CF-class

| Laag | CF-class | Astro-component | Verantwoordelijkheid |
|---|---|---|---|
| 1 | `page-wrapper` | `Layout.astro` → `<div class="page-wrapper">` | Outerste wrapper, optioneel gestyled |
| 2 | `main-wrapper` | `<main class="main-wrapper">` in `Layout.astro` | Hoofdinhoud afbakenen (a11y) |
| 3 | `section_[naam]` | `Section.astro` → buitenste `<section>`-tag | Eén afgebakend inhoudsblok, navigeerbaar via `id`. Draagt zelf **geen** padding. |
| 4 | `padding-global` + `padding-section-[size]` | eigen `<div>`, staat binnen `Section.astro` | Horizontaal + verticaal ritme, **samen op één losse div, ónder** de sectie-tag (dit is Finsweet's eigen documentatie-patroon, zie voorbeeld hieronder) |
| 5 | `container-[size]` | `Container.astro`, genest ín de padding-div | Uitsluitend `max-width` + centreren — **geen** padding (ontkoppeld van laag 4) |

De scheiding tussen laag 4 en 5 is bewust exact zoals CF het voorschrijft: `padding-global` en `container-*` zijn onafhankelijk van elkaar te gebruiken. Als we ze zouden combineren (padding op de container zetten), zou je nooit een edge-to-edge element binnen een sectie kunnen tonen zonder de container-padding te breken.

**Correctie t.o.v. een eerdere versie van dit bestand:** Finsweet's eigen documentatie-voorbeelden zetten `padding-global` + `padding-section-[size]` niet ditect op de `section`-tag zelf, maar op een **aparte div daaronder**. De sectie-tag draagt alleen de `section`-class (puur structureel/navigatie), de padding-laag krijgt een eigen div. Dat patroon volgen we hier nu 1-op-1. Ook de class-naam is gecorrigeerd: Finsweet noemt dit `padding-section-[size]` (in de `padding-`-folder, net als `padding-global`) — **niet** `section-padding-[size]`.

## 2. De componenten

### `Layout.astro` — page-wrapper + main-wrapper

```astro
---
// src/layouts/Layout.astro
interface Props {
  title: string;
  description?: string;
}
const { title, description }: Props = Astro.props;
---
<html lang="nl">
  <head>
    <title>{title}</title>
    {description && <meta name="description" content={description} />}
  </head>
  <body>
    <div class="page-wrapper">
      <slot name="nav" />
      <main class="main-wrapper">
        <slot />
      </main>
      <slot name="footer" />
    </div>
  </body>
</html>
```

- `<body>` krijgt bewust **geen** class/opmaak buiten typografie/`background-color` (CF-regel) — alle paginabrede styling (bijv. `overflow-x: hidden`) staat op `.page-wrapper`.
- `nav`/`footer` zijn named slots: ze zitten binnen `.page-wrapper`, maar bewust **buiten** `<main>` — dat is de a11y-regel achter `main-wrapper`.

### `Section.astro` — section + aparte padding-div (padding-global + padding-section)

```astro
---
// src/components/ui/Section.astro
interface Props {
  id: string;
  edge?: 'small' | 'medium' | 'large' | 'none';
  as?: 'section' | 'div';
}
const { id, edge = 'medium', as: Tag = 'section' }: Props = Astro.props;
---
<Tag id={id} class="section">
  <div class:list={['padding-global', edge !== 'none' && `padding-section-${edge}`]}>
    <slot />
  </div>
</Tag>
```

```css
.padding-global { padding-inline: var(--space-page-inline); }
.padding-section-small  { padding-block: var(--space-section-sm); }
.padding-section-medium { padding-block: var(--space-section-md); }
.padding-section-large  { padding-block: var(--space-section-lg); }
```

- `edge` is de prop-naam (niet `padding`) om verwarring met CSS `padding` te voorkomen — de waarde stuurt zowel de vaste horizontale `padding-global` als de gekozen verticale `padding-section-*` aan, **samen op de aparte div onder de sectie-tag** — dit is exact het patroon uit Finsweet's eigen documentatievoorbeelden, inclusief de class-naam `padding-section-[size]` (niet `section-padding-[size]`).
- De `section`-tag zelf blijft "kaal": alleen de `section`-class, puur voor structuur/navigatie/anchor-links. Geen padding, geen andere stijl.
- Verplicht `id` (geen optionele prop): elke sectie in CF heeft een beschrijvende naam voor Navigator-achtige leesbaarheid — hier voor anchor-links én voor leesbaarheid in de browser devtools.

### `Container.astro` — uitsluitend container-[size]

```astro
---
// src/components/ui/Container.astro
interface Props {
  size?: 'small' | 'medium' | 'large';
}
const { size = 'large' }: Props = Astro.props;
---
<div class:list={['container', `container-${size}`]}>
  <slot />
</div>
```

```css
.container { margin-inline: auto; width: 100%; }
.container-small  { max-width: var(--container-sm); }
.container-medium { max-width: var(--container-md); }
.container-large  { max-width: var(--container-lg); }
```

Geen padding hier — dat is bewust weggehaald uit deze laag (zie tabel hierboven).

## 3. Volledig uitgewerkt voorbeeld: een homepage

```astro
---
// src/pages/index.astro
import Layout from '../layouts/Layout.astro';
import Nav from '../components/sections/Nav.astro';
import Footer from '../components/sections/Footer.astro';
import Section from '../components/ui/Section.astro';
import Container from '../components/ui/Container.astro';
import Hero from '../components/sections/Hero.astro';
import AboutContent from '../components/sections/AboutContent.astro';
---
<Layout title="Omit Design — Home">
  <Nav slot="nav" />

  <Section id="hero" edge="large">
    <Container size="large">
      <Hero />
    </Container>
  </Section>

  <Section id="about" edge="medium">
    <Container size="medium">
      <AboutContent />
    </Container>
  </Section>

  <Footer slot="footer" />
</Layout>
```

### Wat dit rendert (de daadwerkelijke HTML-output)

```html
<body>
  <div class="page-wrapper">

    <nav class="nav">…</nav>

    <main class="main-wrapper">

      <section id="hero" class="section">
        <div class="padding-global padding-section-large">
          <div class="container container-large">
            <!-- Hero-component -->
          </div>
        </div>
      </section>

      <section id="about" class="section">
        <div class="padding-global padding-section-medium">
          <div class="container container-medium">
            <!-- AboutContent-component -->
          </div>
        </div>
      </section>

    </main>

    <footer class="footer">…</footer>

  </div>
</body>
```

### Wat elk element doet — regel voor regel

| Element | Class(es) | Doel |
|---|---|---|
| `<div class="page-wrapper">` | `page-wrapper` | Outerste container van de hele pagina. Bewust géén styling buiten evt. `overflow-x`/achtergrondkleur — voorkomt dat we `<body>` zelf moeten stylen. |
| `<nav class="nav">` | `nav` | Losstaand van `main-wrapper`, zodat schermlezers/zoekmachines de navigatie niet als "hoofdinhoud" interpreteren. |
| `<main class="main-wrapper">` | `main-wrapper` | Bakent de hoofdinhoud af (a11y-verplichting). Bevat alléén de secties, geen nav/footer. |
| `<section id="hero" class="section">` | `section` | Puur structuur: één afgebakend inhoudsblok. `id="hero"` maakt hem anchor-linkbaar en herkenbaar in devtools. Draagt zelf geen padding. |
| `<div class="padding-global padding-section-large">` | `padding-global` (horizontaal ritme), `padding-section-large` (verticaal ritme) | Aparte div, één niveau onder de sectie-tag — precies zoals Finsweet's eigen documentatievoorbeelden. Beide padding-richtingen samen op dit ene element. |
| `<div class="container container-large">` | `container` (centreren/breedte-gedrag), `container-large` (max-width-waarde) | Begrenst de leesbreedte van de content, **onafhankelijk** van de padding eromheen. Geen eigen padding. |
| `<section id="about" class="section">` | `section` | Tweede sectie — zelfde patroon. |
| `<div class="padding-global padding-section-medium">` | idem, andere maat | `edge="medium"` levert een kleinere `padding-section-*`-waarde, `padding-global` blijft overal identiek. |
| `<footer class="footer">` | `footer` | Losstaand van `main-wrapper`, zelfde reden als `nav`. |

## 4. Waar komt nieuwe content in de structuur?

**Regel: alle inhoud van een sectie komt ná de openende `<div class="container container-[size]">`-tag, als kind ervan — nooit ervoor, nooit ernaast, nooit direct in de sectie- of padding-div.**

```html
<section id="about" class="section">
  <div class="padding-global padding-section-medium">
    <div class="container container-medium">
      <!-- ✅ HIER — alles wat bij deze sectie hoort komt hierbinnen -->
    </div>
  </div>
</section>
```

Fout (drie veelvoorkomende afwijkingen):

```html
<!-- ❌ FOUT: content als sibling van container, i.p.v. erin -->
<div class="padding-global padding-section-medium">
  <div class="container container-medium"></div>
  <h2>Over Omit Design</h2>
</div>

<!-- ❌ FOUT: content direct in de padding-div, container overgeslagen -->
<div class="padding-global padding-section-medium">
  <h2>Over Omit Design</h2>
</div>

<!-- ❌ FOUT: content direct op de sectie-tag, padding-div overgeslagen -->
<section id="about" class="section">
  <h2>Over Omit Design</h2>
</section>
```

Waarom dit strikt gehandhaafd wordt:
- **`container`** is de enige laag die de leesbreedte begrenst (`max-width` + centreren). Content die eromheen of ervoor staat, mist die begrenzing en loopt edge-to-edge — meestal niet de bedoeling.
- **`padding-global`/`padding-section-*`** regelen alléén het ritme rond de container, niet de inhoud zelf. Content die daar direct in staat, mist de horizontale marge die `container` toevoegt.
- Consistente insertion point = voorspelbaar voor iedereen (en voor Claude Code) die later in dezelfde sectie verder bouwt.

**Praktisch in Astro:** dit is exact waarom `Container.astro`'s `<slot />` bestaat — alles wat je tussen `<Container>...</Container>` schrijft komt automatisch op de juiste plek terecht. Bij het uitbreiden van een bestaande sectie: zoek de `<Container>`-tag op en voeg nieuwe content tussen die tags toe, nooit ervoor of erna binnen `<Section>`.

```astro
<Section id="about" edge="medium">
  <Container size="medium">
    <!-- nieuwe content hier, altijd -->
  </Container>
</Section>
```

## 5. Wat is "hetzelfde als CF" en wat niet

**1. Bestanden/componenten/props (Astro/TypeScript-laag) — dit is ánders dan CF:**
- Bestanden: PascalCase, `Section.astro`, `Container.astro` — geen `_`-naamgeving, de mappenstructuur vervult die rol (zie `naming-and-file-structure.md`).
- Props: camelCase, typed unions (`edge: 'small' | 'medium' | 'large'`) — geen `is-*`-classes.

**2. Gerenderde CSS-classnamen in de HTML-output — dit blijft bewust dicht bij CF:**
- `page-wrapper`, `main-wrapper`, `section`, `padding-global`, `padding-section-large`, `container-large` zijn (bijna) letterlijk CF's utility/structuurnamen.
- Reden: deze classnamen zijn wat je in de browser devtools/inspector daadwerkelijk ziet. Door ze herkenbaar te houden blijft de site "leesbaar" voor iedereen die de CF-conventie kent (of voor jezelf, jaren later) — exact CF's oorspronkelijke doel ("een niet-technisch persoon moet het kunnen volgen").
- De nesting (sectie-tag → aparte padding-div → container-div) en de naam `padding-section-[size]` (i.p.v. `section-padding-[size]`) volgen hier bewust Finsweet's eigen documentatievoorbeelden 1-op-1, inclusief de extra nesting-laag — dit is dus geen Astro-specifieke afwijking, maar de daadwerkelijke CF-structuur.

Kortom: **de skill verandert niet hoe de site er in de browser uitziet qua class-namen** (die blijven CF-achtig leesbaar) — hij verandert **hoe jij als developer die classes produceert** (via typed component-props in plaats van handmatig classes stapelen in een Designer-canvas).

## 6. Tokenwaarden — wat zit er achter elke class?

Elke class hierboven verwijst naar een CSS custom property uit `tokens.css` (zie ook `tokens-and-sizing.md`). Concrete startwaarden, gebaseerd op het rem/4pt-systeem en (waar bekend) Finsweet's eigen cloneable-defaults:

```css
/* tokens.css */
:root {
  /* Ruimte-schaal — basis voor alles hieronder */
  --space-1: 0.5rem;   /* 8px */
  --space-2: 1rem;     /* 16px */
  --space-3: 1.5rem;   /* 24px */
  --space-4: 2rem;     /* 32px */
  --space-5: 3rem;     /* 48px */
  --space-6: 4rem;     /* 64px */
  --space-7: 6rem;     /* 96px */
  --space-8: 8rem;     /* 128px */

  /* padding-global — horizontaal ritme, fluid tussen mobile en desktop */
  --space-page-inline: clamp(1.5rem, 4vw + 0.5rem, var(--space-4)); /* 24px → 32px */

  /* padding-section-[size] — verticaal ritme tussen secties */
  --space-section-sm: var(--space-6); /* 64px */
  --space-section-md: var(--space-7); /* 96px */
  --space-section-lg: var(--space-8); /* 128px */

  /* container-[size] — max-width, onafhankelijk van padding */
  --container-sm: 48rem;  /* 768px */
  --container-md: 64rem;  /* 1024px */
  --container-lg: 80rem;  /* 1280px — dit is ook Finsweet's eigen cloneable-default voor container-large */
}
```

| Class | Token | Waarde | Bron |
|---|---|---|---|
| `padding-global` | `--space-page-inline` | `clamp(1.5rem, 4vw + 0.5rem, 2rem)` (24px–32px) | Fluid i.p.v. vaste waarde — zie `responsive-and-motion.md` voor de `clamp()`-aanpak. Pas de min/max aan op je eigen breakpoints. |
| `padding-section-small` | `--space-section-sm` | `4rem` (64px) | Eigen keuze, volgt de `--space-*`-schaal |
| `padding-section-medium` | `--space-section-md` | `6rem` (96px) | Eigen keuze, volgt de `--space-*`-schaal |
| `padding-section-large` | `--space-section-lg` | `8rem` (128px) | Eigen keuze, volgt de `--space-*`-schaal |
| `container-small` | `--container-sm` | `48rem` (768px) | Eigen keuze |
| `container-medium` | `--container-md` | `64rem` (1024px) | Eigen keuze |
| `container-large` | `--container-lg` | `80rem` (1280px) | **Bevestigd uit Finsweet's documentatie**: "container-large has a max-width value of 80rem (1280px)" |

**Belangrijk:** alleen `container-large = 80rem` is een letterlijk gedocumenteerde Finsweet-waarde die ik heb kunnen bevestigen. De overige waarden (`padding-section-*`, `container-sm/md`, `padding-global`) zijn een redelijk startpunt volgens dezelfde rem/4pt-logica — geen officieel gegarandeerde CF-cloneable-cijfers. Pas ze aan op het daadwerkelijke ontwerp (Paper.design-tokens) van elk project; de structuur en class-namen blijven hetzelfde, alleen de getallen achter de tokens wijzigen per project.

## 7. Checklist

- [ ] Elke pagina gebruikt `Layout.astro` met een `<main class="main-wrapper">` die nav/footer uitsluit.
- [ ] Elke sectie heeft een verplicht, beschrijvend `id` en gebruikt `Section.astro`.
- [ ] De sectie-tag zelf draagt alleen `class="section"` — geen padding.
- [ ] `padding-global` en `padding-section-*` zitten samen op de aparte div ónder de sectie-tag, nooit direct op `section` en nooit op de container.
- [ ] `Container.astro` bevat nooit padding — alleen `max-width` + centreren.
- [ ] **Nieuwe content komt altijd tussen `<Container>...</Container>`, nooit ervoor/erna of in de padding-div** (zie §4).
- [ ] Elke tokenwaarde is een CSS custom property uit `tokens.css`, geen magic number in de component-CSS.
- [ ] De gerenderde class-namen zijn herkenbaar CF-achtig (devtools-leesbaarheid), ook al is de manier waarop ze geproduceerd worden (props) volledig anders dan in Webflow.
