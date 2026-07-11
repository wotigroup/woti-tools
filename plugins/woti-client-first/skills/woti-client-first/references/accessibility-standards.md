# Toegankelijkheidsstandaard: WCAG 2.2 AA + A11y Project Checklist

**Verhouding tot `accessibility-and-semantics.md`:** dat bestand is een vertaling van Client-First's eigen a11y-hoofdstuk (Webflow-specifieke aandachtspunten: ARIA-dubbeling, `<header>`/`<nav>`, focus-management). Dit bestand is breder: de daadwerkelijke, wettelijk relevante standaard (WCAG 2.2, niveau AA) plus de praktische checklist van [The A11y Project](https://www.a11yproject.com/checklist/), toegepast op Astro. Gebruik dit bestand als je een project **volledig** WCAG-conform wil opleveren, niet alleen "CF-stijl toegankelijk".

## 1. Juridisch kader (waarom dit er niet voor niets is)

- **EAA (European Accessibility Act)** — sinds 28 juni 2025 actief gehandhaafd in de EU, ook in Nederland (ACM handhaaft o.a. e-commerce en elektronische communicatie). Van toepassing op een breed scala aan private-sector diensten (webshops, banking, telecom, transportticketing, en meer).
- **Technische norm:** de EAA verwijst naar **EN 301 549**, dat op dit moment (2026) **WCAG 2.1 niveau AA** volledig incorporeert als juridische ondergrens. Een nieuwe versie (EN 301 549 v4.1.1, verwacht in 2026) zal **WCAG 2.2** incorporeren — nu al bouwen volgens WCAG 2.2 AA is dus toekomstbestendig en heeft geen nadelen (2.2 is volledig achterwaarts compatibel met 2.1).
- **Uitzondering micro-ondernemingen:** bedrijven met <10 medewerkers en <€2 miljoen omzet zijn vrijgesteld van de EAA-*diensten*verplichting. Dit geldt voor de onderneming die de dienst aanbiedt — **niet automatisch voor de website die je voor een klant bouwt**: of die klant zelf onder de EAA valt, hangt af van hun sector/omvang. Behandel WCAG 2.2 AA daarom als de default, niet als iets dat je per project moet beargumenteren.
- **Praktisch advies bij twijfel of een klant verplicht is:** bouw sowieso AA-conform. De meerkosten zijn laag als het vanaf het begin in de component-architectuur zit (zoals in deze skill), en torenhoog als het achteraf moet worden gerepareerd.

*Stand van zaken op moment van schrijven — regelgeving en geharmoniseerde normen worden bijgewerkt; verifieer bij twijfel over een specifiek project via [EN 301 549 / EC-pagina](https://digital-strategy.ec.europa.eu/en/policies/web-accessibility-directive-standards-and-harmonisation) of [W3C WAI](https://www.w3.org/WAI/standards-guidelines/wcag/).*

## 2. WCAG 2.2 AA — de vier principes (POUR), toegepast op Astro

### Perceivable (waarneembaar)

| Eis | Implementatie in deze stack |
|---|---|
| Kleurcontrast tekst: **4,5:1** (normale tekst), **3:1** (grote tekst ≥24px/18,66px bold) | Test elk `--color-text-*`/`--color-bg-*`-paar bij het opzetten van `tokens.css` (zie `tokens-and-sizing.md` §5), niet pas achteraf. Tools: [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/). |
| Kleurcontrast UI-componenten/focus-indicators: **3:1** | Geldt ook voor button-randen, form-input-randen, focus-rings — niet alleen tekst. |
| Kleur niet als enige informatiedrager | Een foutmelding op een form-veld mag niet alleen rood zijn — voeg een icoon/tekst toe (zie §3 forms). |
| Alt-tekst op alle content-afbeeldingen; `alt=""` op decoratieve afbeeldingen | Verplicht `alt`-prop op elk `<Image>`/`<img>`-gebruik — geen default leeg laten. In Keystatic-schema's: maak `alt` een verplicht veld bij image-content-types, niet optioneel. |
| Tekst herschaalbaar tot 200% zonder verlies van functionaliteit | Consequent gebruik van `rem` (zie `tokens-and-sizing.md` §1) is hier de basis — geen vaste `px`-hoogtes op tekstcontainers die content kunnen afknippen. |
| Ondertiteling/transcript bij video/audio-content | Bij het inbedden van video (bijv. Hero-achtergrondvideo): voorzie een transcript of ondertiteling-track, of maak de video puur decoratief (`aria-hidden="true"`, geen essentiële informatie erin). |

### Operable (bedienbaar)

| Eis | Implementatie in deze stack |
|---|---|
| Volledig bedienbaar met toetsenbord, geen keyboard traps | Zie ook `accessibility-and-semantics.md` §4. Test elk interactief component (modal, dropdown, slider) met alleen `Tab`/`Shift+Tab`/`Enter`/`Esc`. |
| **Skip link** naar hoofdinhoud | Verplicht eerste focusbare element op elke pagina — zie code-voorbeeld §4. |
| Focus-indicator zichtbaar, **niet volledig verborgen** door andere content (WCAG 2.2, nieuw t.o.v. 2.1) | Sticky headers mogen een gefocust element niet volledig overlappen — test met `Tab` door de hele pagina, inclusief onder sticky elementen. |
| **Target size minimaal 24×24px** (WCAG 2.2 AA, nieuw) | Elke klikbare/tikbare `<button>`/`<a>` minimaal 24×24px, ook op mobiel — controleer dit specifiek bij icon-only buttons (bijv. een close-icon in een modal). |
| Geen "drag-only"-interacties zonder alternatief (WCAG 2.2, nieuw) | Als een component drag-and-drop gebruikt (bijv. een image-reorder in een custom Keystatic-veld), voorzie een klik-gebaseerd alternatief (bijv. pijltjes-knoppen). |
| Consistente navigatie tussen pagina's | Nav/footer-structuur identiek op elke pagina (dit volgt al automatisch uit `Layout.astro`, zie `core-structure-and-components.md`). |
| Timing: geen onaangekondigde tijdslimieten | Relevant bij bijv. formulier-sessies (Keystatic-preview-links, contactformulieren) — waarschuw ruim van tevoren als een sessie kan verlopen. |

### Understandable (begrijpelijk)

| Eis | Implementatie in deze stack |
|---|---|
| `lang`-attribuut correct ingesteld | `<html lang="nl">` in `Layout.astro` (al aanwezig, zie `core-structure-and-components.md`) — pas aan per taalversie bij meertalige projecten. |
| Labels en instructies bij elk formulierveld | Elk `<input>` heeft een gekoppeld `<label for="...">`, geen placeholder-only labels. Zie §3 voor een concreet patroon. |
| Foutmeldingen: identificatie + suggestie | Een form-error zegt niet alleen "ongeldig", maar ook wat er verwacht wordt ("E-mailadres moet een @ bevatten"), gekoppeld via `aria-describedby`. |
| **Accessible authentication** (WCAG 2.2, nieuw) — geen cognitive test (bijv. puzzel-CAPTCHA) zonder alternatief | Relevant bij contactformulieren met spam-bescherming — kies een niet-visuele/niet-puzzel-methode (bijv. honeypot-veld, server-side rate limiting) in plaats van een CAPTCHA die cognitief/visueel belastend is. |

### Robust (robuust)

| Eis | Implementatie in deze stack |
|---|---|
| Geldige, correct geneste HTML | `astro check` (zie STACK.md §2.3) vangt een deel hiervan; vul aan met een HTML-validator in CI indien gewenst. |
| Custom interactieve componenten hebben correcte `name`/`role`/`value` | Bij client-side componenten (accordion, tabs, custom select): gebruik de juiste ARIA-pattern (zie [ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/patterns/)) — bouw geen custom dropdown zonder de bijbehorende `role`/`aria-expanded`/`aria-activedescendant`-set. |
| Statusberichten aangekondigd zonder focus-verlies | Dynamische content (bijv. "Bericht verzonden" na een formulier, of een winkelwagen-update) gebruikt `aria-live="polite"`, niet alleen een visuele toast. |

## 3. Concrete patronen

### Skip link (verplicht, eerste focusbaar element)

```astro
<!-- Layout.astro, direct na <body> -->
<a href="#main-content" class="skip-link">Direct naar hoofdinhoud</a>
<div class="page-wrapper">
  <slot name="nav" />
  <main id="main-content" class="main-wrapper" tabindex="-1">
    <slot />
  </main>
  <slot name="footer" />
</div>
```

```css
.skip-link {
  position: absolute;
  left: -9999px;
  top: 0;
  z-index: 100;
  background: var(--color-bg-page);
  color: var(--color-text-primary);
  padding: var(--space-2) var(--space-3);
}
.skip-link:focus {
  left: var(--space-2);
  top: var(--space-2);
}
```

### Focus-indicator (zichtbaar, voldoende contrast, niet verborgen)

```css
:focus-visible {
  outline: 2px solid var(--color-focus-ring);
  outline-offset: 2px;
}
/* Nooit outline: none zonder een even zichtbaar alternatief terug te zetten */
```

`--color-focus-ring` is een eigen semantisch token (zie `tokens-and-sizing.md` §5) met minimaal 3:1 contrast t.o.v. de achtergrond eromheen.

### Formulierveld-patroon (label + foutmelding correct gekoppeld)

```astro
<div class="field">
  <label for="email">E-mailadres</label>
  <input
    type="email"
    id="email"
    name="email"
    required
    aria-describedby={hasError ? 'email-error' : undefined}
    aria-invalid={hasError}
  />
  {hasError && (
    <p id="email-error" class="field_error">
      Vul een geldig e-mailadres in, bijvoorbeeld naam@voorbeeld.nl
    </p>
  )}
</div>
```

### Statusbericht (aria-live, voor dynamische updates)

```astro
<div aria-live="polite" class="visually-hidden" id="form-status">
  {statusMessage}
</div>
```

```css
.visually-hidden {
  position: absolute;
  width: 1px; height: 1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
}
```

## 4. A11y Project Checklist — praktische aanvulling

Naast de WCAG-tabellen hierboven, dit zijn de items uit [The A11y Project Checklist](https://www.a11yproject.com/checklist/) die het vaakst gemist worden en niet expliciet in WCAG's formele tekst staan:

- [ ] Paginatitel (`<title>`) is uniek en beschrijvend per pagina — niet overal "Home | Omit Design".
- [ ] Heading-structuur is testbaar met alleen de headings-outline (browser devtools of een extensie) — leesbaar als inhoudsopgave, geen headings puur voor visuele grootte gekozen (zie ook `accessibility-and-semantics.md` §1: gebruik `text-style-h2` op een correct genest heading-niveau, nooit een verkeerd niveau voor het uiterlijk).
- [ ] Links hebben een betekenisvolle naam op zichzelf — geen "klik hier" of "lees meer" zonder context (screenreader-gebruikers browsen vaak een linklijst los van de omliggende tekst).
- [ ] Iframes (bijv. een ingesloten kaart of video) hebben een `title`-attribuut.
- [ ] Zoom tot 400% breekt de layout niet (test browser-zoom, niet alleen tekstgrootte).
- [ ] Reduced-motion wordt gerespecteerd (zie `accessibility-and-semantics.md` §5) — ook bij scroll-triggered animaties uit `responsive-and-motion.md`.
- [ ] Geen content die puur op `:hover` verschijnt zonder toetsenbord-equivalent (`:focus`).

## 5. Testen (minimaal, per project)

1. **Geautomatiseerd**: [axe DevTools](https://www.deque.com/axe/devtools/) of Lighthouse a11y-audit in CI — vangt ~30-40% van de problemen, geen vervanging voor handmatig testen.
2. **Toetsenbord-only**: navigeer de hele pagina met alleen `Tab`/`Shift+Tab`/`Enter`/`Esc`/pijltjes — elk interactief element bereikbaar, focus altijd zichtbaar, geen traps.
3. **Screenreader-steekproef**: minimaal één keer per project met VoiceOver (macOS, ingebouwd) of NVDA (Windows, gratis) door de hoofdflows lopen — homepage, een contentpagina, een formulier.
4. **Contrast-check**: elk kleurtoken-paar uit `tokens.css` langs [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/) vóórdat het project wordt opgeleverd, niet pas bij een klacht.

## 6. Checklist (samenvattend)

- [ ] Alle kleurtoken-paren voldoen aan 4,5:1 (tekst) / 3:1 (UI-componenten, grote tekst).
- [ ] Skip link aanwezig als eerste focusbaar element.
- [ ] Focus-indicator zichtbaar op elk interactief element, nooit `outline: none` zonder alternatief.
- [ ] Alle klikbare elementen minimaal 24×24px (WCAG 2.2 AA).
- [ ] Elk formulierveld heeft een gekoppeld `<label>` en, bij een fout, een gekoppelde `aria-describedby`-foutmelding.
- [ ] Dynamische content-updates gebruiken `aria-live`.
- [ ] Geen drag-only interacties zonder klik-alternatief.
- [ ] Getest: toetsenbord-only, minimaal één screenreader-steekproef, geautomatiseerde axe/Lighthouse-scan.
