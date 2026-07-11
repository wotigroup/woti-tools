# Fluid responsive & interactie-naamgeving

Bron: Client-First *Fluid responsive*, *Interactions naming*.

## 1. Fluid responsive: root-font scaling → CSS `clamp()`

CF lost fluid scaling op door de root `<html>` font-size te laten meebewegen met de viewport via een gegenereerde CSS-snippet (omdat Webflow geen `clamp()`-support had ten tijde van ontwerp). Omdat elke rem-waarde relatief is aan de root font-size, schaalt zo het hele project mee.

In Astro/moderne CSS is dit **niet meer nodig** — `clamp(min, preferred, max)` doet exact hetzelfde, preciezer, zonder root-font-size-trucs, en blijft net als CF's aanpak volledig rem-gebaseerd (dus nog steeds a11y-vriendelijk t.o.v. browser-zoom/fontsize).

```css
:root {
  /* CF-equivalent van root-font scaling, maar per token i.p.v. globaal op html */
  --font-size-h1: clamp(2rem, 1.5rem + 2vw, 3.5rem);
  --space-section-lg: clamp(3rem, 2rem + 4vw, 8rem);
}
```

Vertaalregel: elk token dat in CF een vaste rem-waarde had die "op grote schermen groter mag" wordt hier een `clamp()` met dezelfde min/max-grenzen die je in de Client-First fluid-generator zou hebben ingesteld — behoud altijd `rem`-eenheden binnen de `clamp()` (niet puur `vw`), zodat browser-zoom blijft werken (CF's kernargument tegen pure `vw`/`vh`-scaling blijft hier net zo geldig).

**Belangrijk verschil met CF:** gebruik `clamp()` per individueel token (typografie, sectie-spacing, container-breedte), niet als één globale root-font-scaling-regel — dit geeft fijnmaziger controle en is de huidige best practice, en voorkomt dat een wijziging in root font-size onbedoeld *alle* rem-waarden in het project verschuift (inclusief waarden die je niet fluid wilde maken).

## 2. Interactie-naamgeving

CF's regel voor Webflow Interactions: naam = **Element + Actie + [Trigger] + [optioneel: breakpoint/device]**, bijv. *"Nav Sidebar Slide [Show] [Mobile]"*.

Vertaling naar Astro/TS: dezelfde structuur, toegepast op **functienamen, data-attributen en CSS-animatienamen**:

| CF-patroon | Astro/TS-equivalent |
|---|---|
| `Nav Sidebar Slide [Show] [Mobile]` | functie: `showNavSidebarMobile()`, of `data-animate="nav-sidebar-show-mobile"` |
| `Hero Scroll Trigger [In] [Tablet Mobile]` | `data-animate="hero-in"` + `data-breakpoint="tablet-mobile"`, of een `@media`-scoped CSS-animatie |
| `Background Textures [Hover In] [Desktop]` | `.background-texture:hover { }` binnen een `@media (hover: hover)`-query (i.p.v. expliciet "Desktop" te labelen, gebruik feature-detection waar mogelijk) |

Praktisch patroon voor client-side interactiviteit in Astro:

```astro
<div class="hero" data-animate="hero-in">
  <slot />
</div>

<script>
  // src/scripts/animations.ts of inline
  const el = document.querySelector('[data-animate="hero-in"]');
  // ... IntersectionObserver of View Transitions API
</script>
```

- Gebruik `data-animate="[element]-[actie]"` consistent als hook tussen HTML en JS/TS — dit is het Astro-equivalent van CF's Interactions-naamconventie, en houdt animatie-logica ontkoppeld van styling-classes (zelfde scheiding van concerns als CF nastreeft).
- CF's advies "overthink de naam niet, kies iets en ga door" blijft gelden: consistentie is belangrijker dan perfectie in de naamgeving.

## 3. Checklist

- [ ] Fluid tokens gebruiken `clamp()` met `rem`-grenzen, nooit pure `vw`/`vh`.
- [ ] Interactie-hooks volgen `[element]-[actie]`-patroon in `data-*`-attributen of functienamen.
- [ ] Breakpoint-specifieke interacties zijn expliciet benoemd of via feature-queries (`prefers-reduced-motion`, `hover: hover`) afgehandeld — zie ook `accessibility-and-semantics.md` voor `prefers-reduced-motion`.
