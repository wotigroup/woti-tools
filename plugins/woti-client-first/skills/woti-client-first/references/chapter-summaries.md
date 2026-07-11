# Client-First documentatie — volledige hoofdstuk-samenvattingen

Dit bestand vat elk inhoudelijk hoofdstuk van de originele [Client-First-documentatie](https://finsweet.com/client-first/docs/) samen — los van de vertaalslag naar Astro (die staat in de andere reference-bestanden). Gebruik dit als naslagwerk om te controleren of een CF-principe niet over het hoofd is gezien, of om snel de *originele* redenering achter een regel terug te vinden.

Meta-hoofdstukken (Learning path, Quick guide, Big docs, Beginners, V1-to-V2, Convert past projects) zijn hier weggelaten — die gaan over hoe je CF zelf leert/verkoopt/migreert, niet over de methodologie zelf.

---

## Intro

Client-First is geen stijlgids maar een **naamgevings- en organisatieprincipe**. Het doel is tweeledig: (1) een Webflow-project overdraagbaar maken aan een andere developer/bureau zonder reverse-engineering, en (2) een niet-technisch persoon (de klant) in staat stellen het project in Designer te begrijpen en soms zelf te beheren. Beide doelen worden bereikt via één regel: **een class-naam moet zonder voorkennis van het project duidelijk maken wat het element doet** — geen afkortingen, geen generieke namen, maximale context in de naam zelf.

CF onderscheidt vanaf hier vier fundamentele class-types (custom, utility, global, combo) en een vaste core structure van zes lagen. Het hoofdstuk introduceert ook rem als verplichte meeteenheid (a11y: browserzoom/fontsize blijft werken) en geeft een voorproefje van de spacing-strategie (blocks/wrappers/direct-op-element) en het Folders-systeem (virtuele mappen via `_`).

## Classes strategy [Part 1]

Definieert de twee basis class-categorieën scherp:
- **Utility class**: geen `_`, wél `-`, een specifieke combinatie van CSS-eigenschappen die overal in het project herbruikt wordt (bijv. `text-color-primary`). Altijd globaal van aard.
- **Custom class**: wél `_`, gemaakt voor één specifiek component/pagina/element waar utility classes niet volstaan (bijv. `header_background-layer`).

Een class kan daarnaast **global** zijn (bedoeld voor projectbreed hergebruik) — dit staat los van utility/custom: een utility class is per definitie altijd global, maar een custom class kán ook global zijn (bijv. `faq_item`, gebruikt in elke FAQ-sectie).

Motivatie voor custom classes: snelle, geïsoleerde aanpassingen zonder de rest van het project te raken, en het vermijden van *deep stacking* (te veel utility classes op elkaar gestapeld) — als een combinatie van 3+ utility classes een terugkerend patroon wordt, verdient het een eigen custom class.

## Classes strategy [Part 2]

Werkt de **combo class** uit: een variant bovenop een base class, herkenbaar aan het `is-`-prefix (`button is-brand`). Een combo class werkt nooit los — hij erft van de base class ervoor.

Introduceert het universele principe **"don't deep stack"**: vermijd lange kettingen gestapelde classes op één element. Oplossing #1: splits stijllagen op in geneste elementen (zoals CF's eigen core structure doet — 6 losse divs i.p.v. 1 overladen div). Oplossing #2: als een combinatie van globale utility classes (bijv. `text-size-large text-color-neutral`) op één specifieke plek een extra unieke stijl nodig heeft, voeg dan een instance-specifieke combo class toe (`is-home-header`) die de extra stijl draagt, terwijl de globale utility classes hun globale betekenis behouden. Waarschuwt expliciet: maak nooit een nieuwe aparte class van toevallig samen voorkomende utility classes (`text-size-large text-color-neutral` mag geen eigen betekenis krijgen) — dat ondermijnt de globale herbruikbaarheid.

Sluit af met de vuistregel: **gebruik bij voorkeur één enkele, specifiek genoemde class (`container-large`) boven een combo (`container is-large`)** als een combo niet strikt nodig is — minder classes = betere scanbaarheid in het Styles panel.

## Core structure strategy

Definieert de 6 vaste structuurlagen die elk Client-First-project identiek maakt: `page-wrapper` (outerste parent, optioneel gestyled, geen unieke stijlen — voorkomt styling direct op `<body>`), `main-wrapper` (`<main>`-tag, a11y-verplicht, sluit nav/footer uit), `section_[naam]` (elk een eigen, beschrijvend benoemde Div Block, primair voor Designer Navigator-leesbaarheid en anchor-scrolling, bij voorkeur als `<section>`-tag), `padding-global` (uitsluitend `padding-left`/`padding-right`, nooit voor spacing van individuele content-items), `container-[size]` (max-width, bewust ontkoppeld van padding zodat beide onafhankelijk te gebruiken zijn — 2 tot 4 maten: small/medium/large), en `padding-section-[size]` (verticaal ritme tussen secties, 2-4 maten, staat op een eigen div samen met `padding-global`, ónder de sectie-tag).

Kernwaarde: iedereen die CF kent herkent de structuur van een onbekend project **onmiddellijk**, en iedereen die CF niet kent begrijpt de structuur alsnog door de beschrijvende namen.

## Sizes and rem

Legt uit waarom Client-First uitsluitend `rem` gebruikt (nooit `px`, `vw`, `vh`) voor vrijwel alle maten. Rem is relatief aan de font-size van het `<html>`-element (standaard 16px browser-default), dus `1rem = 16px` bij default instellingen. Kernargument is puur accessibility: rem-gebaseerde layouts reageren correct op zowel **browser font-size-instellingen** als **browser zoom** — vw/vh en px negeren beide. Adviseert "ronde", makkelijk te onthouden rem-waarden (0.5, 1, 1.5, 2, 2.5, 3...) i.p.v. onhandige exacte conversies (8.4375rem). Noemt Webflow's eigen rem-rekenfunctie in de unit-inputs (`100/16rem` typen berekent automatisch).

## Typography strategy

Kernprincipe: **idealiter staat er geen enkele class op een tekstelement** — de default typografie hoort direct op de HTML-tags (`body`, `p`, `h1`-`h6`) te staan. Een class/token wordt pas toegevoegd bij een *afwijking* van die default, en dan bij voorkeur een herbruikbare globale utility class (`text-color-brand`, `text-size-medium`) in plaats van een nieuwe custom class per instantie — dit voorkomt duplicaat-classes voor hetzelfde herhaalde stijlpatroon.

Behandelt het SEO/a11y-dilemma: een `<h1>` moet om SEO-redenen de H1-tag behouden, maar moet er soms visueel als H2 uitzien. Oplossing: `heading-style-h2` als utility class op de `<h1>` toepassen — de HTML-hiërarchie blijft correct, alleen de visuele stijl wijzigt. Adviseert typografie-utility classes te doorzoeken via het `text-`/`heading-`-prefix in het Styles panel, en om nieuwe typografie-varianten (bijv. een opacity-schaal) als nieuwe utility-folder toe te voegen zodra een project dat nodig heeft.

## Spacing strategy

Beschrijft twee complementaire spacing-implementaties, beide gebaseerd op dezelfde `margin-`/`padding-`-utility classes (richting + maat als twee losse classes, bijv. `margin-bottom` + `margin-large`):
- **Spacing block**: een lege Div Block tussen twee sibling-elementen die ruimte creëert.
- **Spacing wrapper**: een Div Block die één kind-element omwikkelt en zo ruimte creëert t.o.v. een sibling.
- Als alternatief: **CSS Grid-strategie** — in plaats van losse blocks/wrappers per kind-element, regel je alle spacing tussen kinderen met één `gap`-eigenschap op de gedeelde parent.

Waarschuwt expliciet tegen margin/padding-classes direct op tekstelementen (leidt snel tot deep stacking van typografie- + spacing-classes op hetzelfde element) — gebruik in plaats daarvan een spacing block/wrapper eromheen. Erkent dat directe margin/padding op een **custom class** (niet een tekstelement) legitiem is wanneer dat efficiënter is dan een extra wrapper-element. Waarschuwt ook tegen horizontale spacing via margin op herbruikbare elementen als buttons (`margin-right` op een button-class beperkt de herbruikbaarheid) — gebruik daarvoor flex/grid `gap` op de parent.

## Utility class systems

Overzicht van de drie kernutility-categorieën die met de officiële CF-cloneable meekomen: **core structure** (padding-global, container-*, padding-section-*), **typografie** (text-/heading--prefixed classes) en **spacing** (margin-/padding--classes). Elke categorie heeft z'n eigen strategiepagina (zie hierboven). Benadrukt dat custom CSS het beste via het native HTML-embed in Designer wordt toegevoegd (niet via Page/Site Settings' custom code) zodat de stijl zichtbaar blijft tijdens het bouwen. Alle meegeleverde classes zijn optioneel — geen enkele utility class is verplicht te gebruiken, het is een startpunt, geen keurslijf.

## Folders

Introduceert het virtuele foldersysteem (via de Finsweet Extension) dat classes in Designer's Styles panel visueel organiseert op basis van de `_`-naamgeving. Custom classes (met `_`) worden genest volgens elk `_`-gescheiden keyword (`nav_primary_logo-wrapper` → map `nav_` → submap `primary_` → element `logo-wrapper`). Utility classes (zonder `_`, alleen `-`) worden automatisch in een aparte "Utility"-hoofdmap georganiseerd, genest op keyword-index (`text-color-primary` → map `text` → submap `color` → element `primary`). Een groot voordeel: een hele folder in bulk hernoemen hernoemt automatisch alle classes erin — handig bij het herstructureren van een project of het bulk-aanpassen van library-componenten (bijv. Relume).

## Folders strategy

Vervolg op Folders: gaat dieper in op *hoe* je folder-namen strategisch kiest, afhankelijk van projectgrootte en type. Introduceert de precisering rond componenten (sinds V2): een `_`-folder is niet automatisch een component — pas het element met de expliciete `component`-identifier (`[folder-naam]_component`) telt als copy-paste-bare, complete structuur. Toont patronen voor component-libraries met veel variaties: een extra genest foldernummer (`slider_1_component`, `slider_2_component`) i.p.v. platte, moeilijk te doorzoeken lijsten. Kernboodschap: er is niet één juiste folderstrategie — kies een consistente aanpak per project (algemeen benoemd voor globale/herbruikbare elementen, pagina-specifiek benoemd voor context-gebonden elementen) en meng niet willekeurig door elkaar binnen hetzelfde project.

## Variables (kleur)

Beschrijft Webflow's (CSS custom properties) variabelen-systeem toegepast op kleur, in twee lagen: **primitieve tokens** (ruwe, meest granulaire paletwaarden — de bouwstenen) en **semantische tokens** (betekenisvolle namen die naar een primitief verwijzen, gebruikt in de daadwerkelijke styling). Voordeel: een variabele als `--primary-color` is leesbaarder en makkelijker te onthouden dan een hex-code, en een wijziging in de primitieve laag update automatisch elk semantisch token dat ernaar verwijst — één centrale plek voor een globale kleurwijziging.

## Interactions naming

Naamconventie voor Webflow Interactions (animaties/triggers): **Element + Actie + [Trigger] + [optioneel: breakpoint/device]**, bijv. *"Nav Sidebar Slide [Show] [Mobile]"*. Het device/breakpoint-suffix wordt alleen toegevoegd wanneer een interactie *niet* voor alle breakpoints hetzelfde is — standaard (alle devices) hoeft niet expliciet benoemd. Sluit af met een pragmatische regel: **overthink de naamgeving niet** — een consistente, redelijke naam kiezen en doorwerken weegt zwaarder dan de perfecte naam vinden.

## Fluid responsive in Webflow

Legt uit waarom Webflow van nature geen goede fluid-scaling-oplossing had (`clamp()` was destijds niet beschikbaar): puur `vw`/`vh`-gebaseerde scaling breekt browser-zoom volledig, omdat die eenheden enkel reageren op viewport-afmetingen, niet op de gebruikersvoorkeur voor fontgrootte. CF's oplossing: **root-font scaling** — een gegenereerde CSS-snippet die de `<html>`-font-size laat meebewegen met de viewport binnen ingestelde grenzen. Omdat elke rem-waarde in het project relatief is aan die root font-size, schaalt het hele project automatisch mee zodra de root-waarde verandert, terwijl rem's a11y-voordelen (zoom/fontsize-respect) behouden blijven.

## Semantic HTML tags in Webflow

Legt de "waarom" van semantische tags uit: ze helpen zowel mensen (screenreader-gebruikers) als machines (zoekmachine-crawlers) de structuur van de pagina te interpreteren, waar een sighted user het verschil tussen een `<div>` en een `<main>` visueel niet eens opmerkt. Behandelt de belangrijkste tags stuk voor stuk: `<header>` (mag meerdere keren voorkomen — site-nav én sectie/artikel-headers), `<main>` (a11y-verplicht, één per pagina), `<footer>`, `<section>`, `<figure>` (minder kritiek dan correcte `alt`-tekst, maar wel juiste structuur voor zelfstandige media), en `H1`-`H6` als hiërarchie-dragers. Wijst erop dat Webflow het makkelijk maakt om semantische tags toe te passen via het Settings-panel — er is geen technische drempel, alleen bewustzijn nodig.

## Accessibility

Het meest uitgebreide hoofdstuk. Kernonderscheid: elementen met **native semantiek** (`<button>`, `<nav>`, `<a>`, `<input>`) hebben al een impliciete rol voor screenreaders — voeg daar **nooit** een extra `role`-attribuut aan toe, want dat leidt tot dubbele voorlezing (`role="form"` op een `<form>` wordt voorgelezen als "Form, Form"). ARIA-rollen zijn alleen nodig voor elementen zónder native semantiek die wél een interactieve rol vervullen.

Behandelt toetsenbordnavigatie uitgebreid: de `Tab`-toets moet elk interactief element logisch kunnen bereiken; `tabindex="-1"` is bedoeld om een normaal focusbaar element *tijdelijk* uit de tab-volgorde te halen, niet om willekeurige elementen focusbaar te máken. Bespreekt programmatische focus-verplaatsing (bijv. focus naar de sluitknop bij het openen van een modal) met een expliciete UX-waarschuwing: niet elke contentwijziging rechtvaardigt automatische focus-verplaatsing (bijv. bij tabwissels is dat vaak ongewenst).

Onderscheidt bewust drie soorten disability-context: permanent, tijdelijk, en **situationeel/conditioneel** (bijv. een trage internetverbinding, of eten tijdens het browsen) — a11y is dus breder dan alleen permanente beperkingen. Verwijst door naar de A11y Project Checklist en Webflow's eigen Accessibility Checklist voor de volledige basis, en richt zich zelf specifiek op de uitdagingen die Webflow als platform met zich meebrengt.

## CSS Specificity

Legt uit waarom copy-pasten van CF's margin/padding-utility classes tussen projecten kan resulteren in kapotte spacing: in Webflow bepaalt de **aanmaakvolgorde** van classes (niet de bron-volgorde in code, zoals in reguliere CSS) de specificiteit bij gelijke specificiteits-score — een class die later is aangemaakt wint van een eerder aangemaakte class bij een conflicterende eigenschap. Als je bijvoorbeeld eerst `margin-bottom` (richting) en dan pas `margin-large` (maat, alle zijden) aanmaakt, overschrijft `margin-large` per ongeluk de gerichte `margin-bottom`-waarde in plaats van 'm aan te vullen.

Illustreert het probleem ook los van spacing, met een `display-none`-utility class die door aanmaakvolgorde niet meer "wint" van een eerder aangemaakte class met dezelfde eigenschap. Oplossing: bij het overzetten van CF's spacing-systeem tussen projecten via de Finsweet Extension, wordt de juiste aanmaakvolgorde (eerst `margin-[size]`, dan pas `margin-[direction]`) automatisch gegarandeerd — het probleem treedt alleen op bij handmatige copy-paste in de verkeerde volgorde.
