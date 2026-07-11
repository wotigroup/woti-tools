# Naamgeving & bestandsstructuur

Bron: Client-First *Classes strategy 1*, *Classes strategy 2*, *Folders*, *Folders strategy*.

## 1. De vier CF class-types → Astro-equivalent

CF onderscheidt vier soorten classes. Elk heeft een direct Astro-equivalent:

| CF-type | Webflow-voorbeeld | Astro-equivalent | Waar |
|---|---|---|---|
| **Utility class** (geen `_`, wel `-`, globaal) | `text-color-primary`, `padding-large` | CSS custom property / design token | `tokens.css` |
| **Custom class** (met `_`, specifiek voor 1 component) | `header_background-layer` | Scoped `<style>` binnen het component, of een sub-element in dezelfde `.astro`-file | component-file zelf |
| **Global custom class** (met `_`, hergebruikt over het hele project) | `faq_item` | Herbruikbaar Astro-component | `/src/components/...` |
| **Combo class** (`is-*`, variant op een base class) | `button is-brand` | Typed prop (`variant`, `size`, `state`) op het component | Props-interface van het component |

**Kernregel die overeind blijft:** een naam moet zonder voorkennis van het project duidelijk maken wat het element doet. Geen afkortingen, geen `Rectangle47`-achtige namen, geen generieke `Card1`/`Card2`.

## 2. Component-naamgeving (i.p.v. Folders met `_`)

CF gebruikt `_` om een virtuele folder-hiërarchie te simuleren in Webflow's Styles panel (bijv. `nav_primary_logo-wrapper` = map `nav_` → `primary_` → element `logo-wrapper`). Astro heeft dit niet nodig — de **daadwerkelijke bestandsstructuur** vervult deze rol al (zie STACK.md §2):

```
/components
  /ui               → basis, contextloze elementen (Button, Card, Badge)
  /sections         → herbruikbare pagina-secties (Hero, USPRow, CTA)
  /content-blocks   → MDX content-componenten (ImageTextSplit, ReviewBlock)
```

Vertaalregel: waar CF een `_`-folder-naam zou gebruiken (`nav_primary_logo-wrapper`), gebruik je in Astro een submap + component-naam:
- CF: `nav_primary_logo-wrapper` (1 class, virtuele folder)
- Astro: `/components/sections/Nav/NavLogo.astro`, of als het geen eigen component verdient: een element binnen `Nav.astro` met een duidelijke `aria-label`/comment, niet een aparte class

**Component-naam = wat het is, niet waar het staat.** `Hero.astro`, niet `HomeHero.astro` — paginacontext hoort in de prop (`<Hero variant="home" />`), niet in de bestandsnaam, tenzij het component nooit ergens anders gebruikt wordt.

## 3. Combo classes → variant-props (géén deep stacking)

Dit is de belangrijkste vertaalregel. CF's `is-` combo class:

```html
<!-- Webflow/CF -->
<div class="button is-brand is-large">
```

wordt in Astro/TS **nooit** losse gestapelde classes, maar een typed Props-interface:

```astro
---
// Button.astro
interface Props {
  variant?: 'default' | 'brand' | 'outline';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
}
const { variant = 'default', size = 'medium', disabled = false }: Props = Astro.props;
---
<button class:list={['button', `button--${variant}`, `button--${size}`]} disabled={disabled}>
  <slot />
</button>
```

CF's regel "**don't deep stack**" (geen lange kettingen van gestapelde classes op één element) vertaalt hier direct: als een component meer dan 2-3 variant-dimensies nodig heeft (variant + size + state + ...), is dat een signaal dat het component opgesplitst moet worden — precies zoals CF adviseert om een nested Div Block te maken in plaats van door te stapelen.

**CF's regel tegen "instance-specific combo's van globale utility classes"** (bijv. nooit `text-size-large text-color-neutral` samen een eigen betekenis geven) vertaalt naar: geef nooit een eigen semantische naam aan een toevallige combinatie van twee ongerelateerde tokens. Twee losse props (`size`, `color`) blijven twee losse props — voeg ze niet samen tot een enkele `variant="large-neutral"`.

## 4. Global vs. lokaal — wanneer maak je een component vs. een lokaal element?

CF: "Is de class overal op de site herbruikt, of maar op 1 plek?" → dezelfde vraag geldt voor Astro-componenten:
- **Overal herbruikt** (button, card, badge, form-input) → eigen bestand in `/components/ui`
- **Herhaald binnen 1 contenttype** (FAQ-item, review-blok) → eigen bestand in `/components/content-blocks`, geregistreerd in Keystatic
- **Eenmalig, sectie-specifiek** → geen apart bestand nodig; blijft inline binnen het sectie-component (vergelijkbaar met CF's "custom class die niet global is")

## 5. `design-system.md` als vervanging voor CF's Style Guide-pagina

CF's cloneable bevat een Style Guide-pagina die alle utility classes visueel documenteert. Dat is exact de rol van `design-system.md` uit STACK.md §3.4. Vul daarin per component:
- Naam + bestandslocatie
- Props/varianten (het Astro-equivalent van CF's combo classes)
- Wanneer wel/niet gebruiken
- A11y-eisen (contrast, focus-ring)

**Altijd `design-system.md` raadplegen vóór het aanmaken van een nieuw component** — zelfde reden als in CF: voorkomen van inconsistente duplicaten (bijv. twee verschillende Card-componenten die hetzelfde doen).

## 6. Naamgevingschecklist (samengevat)

- [ ] Kan een niet-technisch teamlid uit de naam afleiden wat het component doet?
- [ ] Is het een `ui`, `sections`, of `content-blocks` component? Staat het in de juiste map?
- [ ] Zijn varianten typed props (union types), geen losse booleans of gestapelde classes?
- [ ] Is de naam contextloos genoeg om elders herbruikt te worden (tenzij bewust lokaal)?
- [ ] Staat het (indien herbruikbaar) in `design-system.md`?
