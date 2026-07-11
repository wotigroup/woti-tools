# Project Stack & Werkwijze

> Dit bestand beschrijft de vaste technische stack en werkwijze voor dit project.
> Claude (en Claude Code) moet dit bestand als leidend uitgangspunt gebruiken bij het genereren, aanpassen en beoordelen van code. Wijk hier niet vanaf zonder expliciete toestemming van de projecteigenaar.

---

## 1. Kernstack

| Laag | Keuze | Waarom |
|---|---|---|
| Frontend framework | **Astro** (static output, `output: 'static'`) | Geen server-runtime nodig, snel, content-first |
| Content/CMS | **Keystatic** (Git-based, MDX) | Geen database, geen server-onderhoud, content = bestanden in de repo |
| Content-formaat | **MDX** (niet `.md`) | Nodig voor custom content-componenten binnen artikelen |
| Design-tool | **Paper.design** | Code-native canvas (HTML/CSS), MCP-gekoppeld, output = herbruikbare Astro-componenten |
| UI-feedback / debug tool | **Agentation** | Visuele UI-annotatie: klik een element, voeg een notitie toe, output bevat class names/selectors/component-hiërarchie/computed styles — direct bruikbaar als context voor Claude Code |
| Styling | **CSS custom properties** (design tokens) + Tailwind waar zinvol | Consistent design system, leesbaar voor mens én AI |
| Hosting | **Bestaande shared hosting (Xynta/DirectAdmin), statische export** | Geen Node.js-runtime nodig op de server, alleen HTML/CSS/JS |
| Deploy | **GitHub Actions → FTP/SFTP naar hosting** | Build gebeurt in CI, alleen `dist/` wordt geüpload |
| Versiebeheer | **Git (GitHub)** | Enige bron van waarheid voor content én code |
| AI-koppeling | **MCP**: GitHub MCP (content/commits) + Paper MCP (design-naar-code) | Claude kan direct content en componenten aanmaken/aanpassen |

**Bewuste keuzes / wat dit NIET is:**
- Geen database (geen MySQL/MariaDB/Postgres) — content leeft als bestanden in Git
- Geen aparte CMS-backend om te onderhouden (geen WordPress, Payload, Strapi, etc.)
- Geen Node.js-runtime nodig in productie — alles wordt statisch gebouwd
- Als een project toch een volwaardige database-backed CMS nodig heeft (bijv. complexe workflows, veel non-technische redacteuren), is dat een bewuste uitzondering — overleg dan eerst, dit is niet de default

---

## 2. TypeScript

TypeScript is verplicht voor alle logica in dit project: `.astro`-frontmatter, losse `.ts`-bestanden, Keystatic-config en Content Collection-schema's. Geen `.js`-bestanden voor nieuwe code, behalve losse third-party snippets die geen TS-variant hebben.

### 2.1 Strictness

`tsconfig.json` gebruikt Astro's `strict`-preset (niet `strictest`, niet `base`):

```json
{
  "extends": "astro/tsconfigs/strict"
}
```

- Goede balans tussen veiligheid en schrijfsnelheid — vangt de meeste fouten (impliciete `any`, null-safety, ongebruikte locals) zonder de overhead van `noUncheckedIndexedAccess` die `strictest` toevoegt.
- Bij een project dat later meer typesafety-garanties nodig heeft (bijv. veel array/object-indexering in complexe dataverwerking), kan per project bewust naar `strictest` overgestapt worden — dit is geen automatisch upgrade-pad, altijd expliciet afstemmen.
- Geen `any` in nieuwe code. Als een externe library geen types levert, schrijf een minimaal `.d.ts`-declaratiebestand in `/src/types/` in plaats van `any` te gebruiken.

### 2.2 Linting & formatting: Biome

Biome vervangt ESLint + Prettier volledig — één tool, één config, snellere checks.

```json
// biome.json
{
  "$schema": "https://biomejs.dev/schemas/1.9.0/schema.json",
  "formatter": { "enabled": true, "indentStyle": "space", "indentWidth": 2 },
  "linter": { "enabled": true, "rules": { "recommended": true } },
  "javascript": { "formatter": { "quoteStyle": "single", "semicolons": "always" } }
}
```

- `.astro`-bestanden: Biome formatteert de frontmatter (TypeScript-blok); de template-markup zelf blijft vooralsnog buiten Biome's bereik (Astro-specifieke formatting) — gebruik hiervoor de Astro VS Code-extensie's ingebouwde formatter als aanvulling, niet Prettier.
- Package-scripts:
  ```json
  {
    "scripts": {
      "lint": "biome check .",
      "lint:fix": "biome check --write .",
      "typecheck": "astro check"
    }
  }
  ```
- **Geen ESLint/Prettier-config toevoegen** aan nieuwe projecten — als een bestaand project deze al heeft, overleg eerst voor migratie naar Biome, dit is geen stilzwijgende vervanging.

### 2.3 Type-checking in CI

De GitHub Action (zie §7 Deployment) draait vóór de build een expliciete typecheck-stap, zodat een type-fout de deploy blokkeert in plaats van pas zichtbaar te worden na een mislukte runtime:

```yaml
- run: npm run typecheck   # astro check — faalt de build bij TS-fouten
- run: npm run lint        # biome check
- run: npm run build       # astro build
```

### 2.4 Content type-safety: Keystatic's afgeleide types

We vertrouwen op Keystatic's automatisch afgeleide types (via het gegenereerde `reader`-object), **geen** aparte Zod-schema's bovenop de Keystatic-config voor runtime-validatie.

- Reden: Keystatic's eigen schema (`keystatic.config.ts`) is al de bron van waarheid voor zowel de Studio-UI als de types — een tweede schema-laag (Zod) zou dubbel onderhoud betekenen zonder veel extra garantie, aangezien content altijd via Keystatic zelf wordt aangemaakt/bewerkt (geen externe/onvertrouwde databron die runtime-validatie rechtvaardigt).
- Gebruik het `reader`-object consistent, nooit los MDX-bestanden handmatig parsen:
  ```ts
  import { createReader } from '@keystatic/core/reader';
  import keystaticConfig from '../../keystatic.config';

  const reader = createReader(process.cwd(), keystaticConfig);
  const posts = await reader.collections.blog.all();
  // posts is volledig getypeerd op basis van keystaticConfig — geen handmatige types nodig
  ```
- Als een collectie-schema wijzigt, wijzigen de types automatisch mee — dit is de belangrijkste reden om **niet** een parallelle Zod-laag te onderhouden die uit sync kan raken.
- Mocht een collectie later een externe/onvertrouwde bron krijgen (bijv. een import-script vanuit een CSV of API), dan is dat een bewuste uitzondering waarbij alsnog Zod-validatie aan de rand (bij de import, niet bij het lezen) toegevoegd wordt — overleg dit eerst, net als bij andere architectuurkeuzes (zie §7).

### 2.5 Component props

Elk Astro-component met props gebruikt een expliciete `Props`-interface, geen inline destructuring zonder type:

```astro
---
interface Props {
  variant?: 'default' | 'brand' | 'outline';
  size?: 'small' | 'medium' | 'large';
}
const { variant = 'default', size = 'medium' }: Props = Astro.props;
---
```

- Varianten/states altijd als union types (`'default' | 'brand' | 'outline'`), nooit als losse `boolean`-props (`isBrand`, `isLarge`) — zie ook de `woti-client-first`-skill (`references/naming-and-file-structure.md`) voor de volledige onderbouwing.
- Geen `any` of `unknown` in een `Props`-interface; als een prop generiek moet zijn, gebruik een specifiek generic type-parameter.

---

## 3. Projectstructuur (uitgangspunt)

```
/src
  /content          → Keystatic content collections (blog, faq, kennisbank, projecten)
    /blog/*.mdx
    /faq/*.mdx
    /kennisbank/*.mdx
    /projecten/*.mdx
  /components        → Herbruikbare Astro-componenten (uit Paper.design of handmatig)
    /ui               → Basis UI-elementen (Button, Card, Badge)
    /sections         → Herbruikbare paginasecties (Hero, USPRow, CTA)
    /content-blocks    → MDX content-componenten (ImageTextSplit, ReviewBlock)
  /layouts
  /pages
  /styles
    tokens.css        → Design tokens (kleur, typografie, spacing, radii, shadows)
keystatic.config.ts
design-system.md       → Component-manifest (zie sectie 4)
astro.config.mjs
```

---

## 4. Design system

### 3.1 Tokens (`/src/styles/tokens.css`)
Alle visuele basiswaarden als CSS custom properties. Geen magic numbers in componenten.

- **Kleur**: basis + variant + state (bijv. `--color-primary`, `--color-primary-hover`, `--color-primary-active`)
- **Typografie**: schaal (`--font-size-sm/md/lg/xl`), font-families, line-heights
- **Spacing**: consistente schaal (`--space-1` t/m `--space-8`)
- **Radii, shadows, borders**: als tokens, niet los per component

### 3.2 States (verplicht per interactief component)
Elk interactief element moet expliciet ontworpen en benoemd zijn voor:
`default`, `hover`, `active/pressed`, `focus` (toetsenbord-navigatie), `disabled`, en waar relevant `error`/`success`.

### 3.3 Naamgeving (Paper ↔ code, 1-op-1)
- Paper-laagnaam = Astro-componentnaam. Structuur: `ComponentNaam/Variant/State`
  (bijv. `Card/Featured/Hover` → `Card.astro` met `variant="featured"`)
- Geen generieke namen als `Rectangle 47` of `btn-1` — dit breekt de AI-conversie

### 3.4 Component-manifest (`design-system.md`, los bestand in de repo)
Per component vastleggen:
- Naam + bestandslocatie
- Beschikbare props/varianten
- Wanneer wel/niet gebruiken
- Toegankelijkheidseisen (contrast, focus-ring verplicht)

**Claude/Claude Code moet dit manifest lezen vóór het genereren van nieuwe componenten**, om te voorkomen dat er inconsistente duplicaten ontstaan naast bestaande componenten.

---

## 5. Content-componenten (MDX in Keystatic)

Blogposts/kennisbank-artikelen mogen afwisselen tussen platte tekst en custom layout-blokken:

```mdx
Normale tekst...

<ImageTextSplit image="../foto.jpg" text="Beschrijving..." richting="links" />

<ReviewBlock rating={4} author="Naam" quote="Korte quote" />
```

- Componenten worden gedefinieerd in `/src/components/content-blocks/`
- Geregistreerd in Keystatic als **content components** met eigen schema (velden zoals `image`, `text`, `richting`)
- **Geen `import`-statements in de MDX-bestanden zelf** — componenten worden gekoppeld via een `components`-object bij het renderen (in de layout, niet per artikel)

---

## 6. MCP-koppelingen

| MCP-server | Doel |
|---|---|
| **GitHub MCP** | Content aanmaken/wijzigen (Markdown/MDX-bestanden committen), rechtstreeks nieuwe blogposts/FAQ/kennisbank-items toevoegen |
| **Paper MCP** | Designs uitlezen (`get_jsx`, `get_computed_styles`) en omzetten naar Astro-componenten; tokens synchroniseren tussen Paper en `tokens.css` |

**Workflow content toevoegen:** gebruiker vraagt om nieuwe blogpost → Claude schrijft MDX-bestand met juiste frontmatter en (optioneel) content-componenten → commit via GitHub MCP → GitHub Action bouwt en deployt automatisch.

**Workflow nieuwe componenten:** ontwerp in Paper.design → Claude leest via Paper MCP de structuur/styling uit → controleert `design-system.md` op bestaande componenten → genereert of hergebruikt Astro-component → werkt manifest bij.

**Workflow UI-feedback:** gebruiker annoteert een element in de browser met Agentation → plakt de gestructureerde output (selectors, component-hiërarchie, computed styles) in Claude Code → Claude gebruikt dit als exacte context om het juiste component/bestand te vinden en aan te passen, zonder zelf te hoeven zoeken naar welk element bedoeld wordt.

---

## 7. Deployment

1. Push naar `main` triggert GitHub Action
2. Action draait `npm run typecheck` (`astro check`) en `npm run lint` (`biome check`) — build stopt bij fouten
3. Action bouwt Astro (`astro build` → statische `dist/`)
4. Action deployt `dist/` via FTP/SFTP naar de hostingomgeving (`public_html` of subdirectory)
5. Geen server-side stappen nodig — hosting hoeft alleen statische bestanden te serveren

---

## 8. Instructies voor Claude Code

- **Lees altijd `design-system.md` en `tokens.css`** voordat je nieuwe UI-componenten maakt of aanpast
- **Gebruik bestaande componenten** uit `/src/components` waar mogelijk in plaats van nieuwe te bouwen
- **Volg de naamgevingsconventie** (`ComponentNaam/Variant/State`) strikt, ook in bestandsnamen en props
- **Nieuwe interactieve componenten**: implementeer altijd alle states (default/hover/active/focus/disabled), ook als het ontwerp er niet expliciet in voorziet — vraag anders na
- **Content wijzigingen**: schrijf naar `/src/content/*.mdx`, nooit direct naar gegenereerde HTML
- **Geen database of CMS-backend toevoegen** zonder expliciete instructie — dit project is bewust database-vrij
- **Geen `any`, geen impliciete types, geen ESLint/Prettier** — volg §2 (TypeScript) strikt: `strict`-preset, Biome, Keystatic's afgeleide types, typed `Props`-interfaces
- **Bij een Agentation-annotatie in de prompt**: gebruik de meegegeven selectors/component-hiërarchie om direct het juiste bestand te vinden, in plaats van eerst zelf te gaan zoeken
- **Bij twijfel over een architectuurkeuze**: vraag het na in plaats van aan te nemen dat een uitzondering oké is

---

## 9. Open vragen per nieuw project (invullen bij start)

- [ ] Projectnaam / domein
- [ ] Hostingdetails (FTP-host, pad, credentials via secrets)
- [ ] Content-collecties nodig (blog / faq / kennisbank / projecten / anders): ___
- [ ] Bestaat er al een Paper.design-bestand met tokens, of starten we vanaf nul?
- [ ] Talen (NL-only / meertalig)?
