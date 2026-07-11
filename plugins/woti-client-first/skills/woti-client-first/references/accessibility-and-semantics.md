# Semantische HTML & toegankelijkheid

Bron: Client-First *Semantic HTML tags*, *Accessibility*.

CF besteedt hier relatief veel aandacht aan omdat Webflow Designer het makkelijk maakt om alles een `<div>` te laten zijn. In Astro schrijf je HTML direct, dus dit risico is kleiner — maar de regels blijven het naslagwerk voor *welke* tag je waar gebruikt.

## 1. Semantische tags — verplichte basisset

Gebruik altijd de juiste tag in plaats van een generieke `<div>`, ongeacht of er styling op zit:

| Tag | Gebruik |
|---|---|
| `<header>` | Introductie-/kopblok voor de content die erop volgt. Meerdere `<header>`-tags per pagina mag (site-header, maar ook binnen een los `<article>`/`<section>`). Zie §2 voor het volledige onderscheid met `<nav>`. |
| `<main>` | Eén per pagina, omvat de hoofdinhoud, sluit nav/footer uit (zie `core-structure-and-components.md`). |
| `<nav>` | Navigatieblokken — mag meerdere keren voorkomen, ook zonder `<header>` (sidebar-TOC, sectie-specifieke doorklik-links). Zie §2. |
| `<footer>` | Site- of sectie-footer. |
| `<section>` | Een afgebakend inhoudsblok — dit is exact CF's `section_[naam]`. |
| `<article>` | Zelfstandig herbruikbare content (blogpost, kennisbank-item, review). |
| `<figure>` + `<figcaption>` | Zelfstandige media (afbeelding, diagram, codeblok) met bijschrift. Minder kritiek voor a11y/SEO dan correcte `alt`-tekst, maar wel de juiste structuur. |
| `<h1>`–`<h6>` | Altijd hiërarchisch correct — nooit een heading-niveau overslaan omwille van optiek. Gebruik i.p.v. daarvan een utility-token (`text-style-h2` op een `<h1>`, zie `tokens-and-sizing.md` §3) om de visuele stijl aan te passen zonder de semantische hiërarchie te breken. |
| `<button>` | Elk element dat een actie op de pagina triggert (toggle, open/close, submit) — nooit een `<div>` met een click-handler. |
| `<a>` | Elk element dat navigeert naar een andere pagina/URL/anchor. |

## 2. `<header>` vs `<nav>` — vaak samen, maar geen synoniemen

Deze twee worden regelmatig door elkaar gehaald omdat ze in de praktijk vaak samen voorkomen (logo + menu in één blok), maar ze markeren iets fundamenteel anders:

- **`<header>`** = een introductie-/kopblok voor de content die erop volgt. Mag **meerdere keren** per pagina voorkomen — elke keer als kop van het blok waarin hij staat (site-header, maar ook de header van een los `<article>` of `<section>`).
- **`<nav>`** = specifiek een blok met navigatielinks. Schermlezers bieden een sneltoets om direct naar elk `<nav>`-landmark te springen — dat werkt alléén als de links daadwerkelijk in een `<nav>` staan, niet los in een `<header>` of `<div>`. Een pagina mag meerdere `<nav>`-blokken hebben; geef ze bij meer dan één een eigen `aria-label` zodat duidelijk is welke welke is.

**Ze zijn geen alternatieven voor elkaar** — meestal zit een `<nav>` gewoon *in* een `<header>`, maar niet altijd. Drie situaties die je in een Astro-project regelmatig tegenkomt:

**1. Site-brede navigatie: `<nav>` binnen `<header>`**

```html
<header class="nav">
  <a href="/" class="nav_logo-link">Omit Design</a>
  <nav aria-label="Hoofdmenu">
    <ul class="nav_menu">
      <li><a href="/werk" class="nav_link">Werk</a></li>
      <li><a href="/contact" class="nav_link">Contact</a></li>
    </ul>
  </nav>
</header>
```

De logo-link is geen navigatie-item in de screenreader-zin — die hoort dus buiten de `<nav>`, terwijl de menu-links er wél in horen. `<header>` is hier de container voor het hele bovenste blok, `<nav>` is specifiek het linkjes-gedeelte daarbinnen.

**2. Sidebar-navigatie in een artikel: `<nav>` zónder `<header>`**

Een inhoudsopgave/sidebar-menu naast een blogpost of kennisbank-artikel is óók navigatie, maar heeft geen introductie-functie — dus geen `<header>` nodig, alleen `<nav>`:

```html
<article>
  <h1>Hoe werkt server-side rendering?</h1>
  <div class="article-layout">
    <nav aria-label="Inhoudsopgave" class="article_toc">
      <ul>
        <li><a href="#wat-is-ssr">Wat is SSR?</a></li>
        <li><a href="#voordelen">Voordelen</a></li>
        <li><a href="#nadelen">Nadelen</a></li>
      </ul>
    </nav>
    <div class="article_content">
      <!-- artikel-inhoud -->
    </div>
  </div>
</article>
```

Hier is `<nav>` een landmark op zichzelf, los van enige header — de schermlezer-gebruiker springt direct naar "Inhoudsopgave" zonder eerst door een introductieblok te hoeven.

**3. `<nav>` binnen een `<section>` die doorverwijst naar subpagina's**

Een sectie die zelf géén losse pagina-navigatie is, maar wél een blok met doorklik-links bevat (bijv. "Onze diensten" met links naar elke dienstpagina), krijgt zijn eigen `<nav>` binnen de sectie — niet de site-brede `<nav>` hergebruiken:

```html
<section id="services" class="section">
  <div class="padding-global padding-section-medium">
    <div class="container container-large">
      <h2>Onze diensten</h2>
      <nav aria-label="Diensten">
        <ul class="services_grid">
          <li><a href="/diensten/webdesign" class="service_link">Webdesign</a></li>
          <li><a href="/diensten/development" class="service_link">Development</a></li>
          <li><a href="/diensten/seo" class="service_link">SEO</a></li>
        </ul>
      </nav>
    </div>
  </div>
</section>
```

De `<h2>` boven de `<nav>` heeft hier zelf géén `<header>` nodig — een los kopje boven een blok hoeft niet per se in een `<header>`-tag; `<header>` is pas relevant zodra er *meerdere* elementen (icoon + titel + intro-tekst, bijv.) samen de "kop" van het blok vormen. Vuistregel: één simpele heading → geen `<header>` nodig; heading + extra introductie-elementen samen → wel.

**Kort samengevat:** `<nav>` gaat over *wat erin staat* (links om te navigeren) — waar dat blok dan weer staat (in een `<header>`, in een sidebar, in een `<section>`) is een aparte, onafhankelijke keuze.

## 3. ARIA: alleen aanvullen wat semantische HTML niet al dekt

CF's kernregel: **native semantische elementen hebben al een impliciete rol** — voeg daar geen `role`-attribuut aan toe, dat leidt tot dubbele aankondiging door screen readers (bijv. `<form role="form">` wordt voorgelezen als "Form, Form").

Voeg ARIA-attributen alleen toe:
- Wanneer je een non-semantisch element (zoals een gestyled `<div>` dat zich als knop gedraagt — vermijd dit waar mogelijk, gebruik liever `<button>`) een rol moet geven.
- Voor dynamische states: `aria-expanded`, `aria-hidden`, `aria-current`, `aria-live` bij content die via JS/client-side script verandert (accordions, dropdowns, modals, tabs).

```astro
<button aria-expanded={isOpen} aria-controls="faq-panel-1">
  {question}
</button>
<div id="faq-panel-1" hidden={!isOpen}>
  {answer}
</div>
```

## 4. Toetsenbordnavigatie

- Elk interactief element moet met `Tab` bereikbaar zijn en een zichtbare `:focus`-stijl hebben (verplicht per STACK.md §3.2 — states: default/hover/active/focus/disabled).
- Gebruik `tabindex="-1"` alleen om een *native focusbaar* element tijdelijk uit de tab-volgorde te halen — niet om custom elementen focusbaar te maken (gebruik daarvoor een echte `<button>`/`<a>`).
- Bij modals/overlays: verplaats focus programmatisch naar het eerste interactieve element bij openen, en terug naar de trigger bij sluiten.
- Val niet automatisch focus-verplaatsing toe te passen bij content die niet in scherm-focus hoort te grijpen (bijv. tab-wissels die niet direct om input vragen) — CF's waarschuwing hierover blijft gelden: denk aan de UX, niet alleen de techniek.

## 5. `prefers-reduced-motion`

Niet expliciet in CF (Webflow-specifiek probleem was minder relevant), maar wél verplicht in deze stack gezien de fluid/interactie-tokens uit `responsive-and-motion.md`:

```css
@media (prefers-reduced-motion: reduce) {
  * { animation-duration: 0.01ms !important; transition-duration: 0.01ms !important; }
}
```

## 6. Checklist per component/pagina

- [ ] Juiste semantische tag gebruikt (geen `<div>` waar `<button>`, `<a>`, `<nav>`, `<section>` etc. hoort).
- [ ] `<nav>` alleen gebruikt voor daadwerkelijke navigatielinks, niet automatisch samen met `<header>` — meerdere `<nav>`'s op een pagina hebben elk een eigen `aria-label`.
- [ ] Heading-hiërarchie klopt (geen overgeslagen niveaus), visuele afwijkingen via utility-tokens, niet via verkeerd heading-niveau.
- [ ] Geen dubbele ARIA-rollen op elementen die al semantisch correct zijn.
- [ ] Alle interactieve elementen: toetsenbord-bereikbaar + zichtbare focus-stijl.
- [ ] Dynamische content (accordion, modal, dropdown) heeft de juiste `aria-*`-attributen die meebewegen met de state.
- [ ] Alt-tekst op alle content-afbeeldingen; decoratieve afbeeldingen krijgen `alt=""`.
