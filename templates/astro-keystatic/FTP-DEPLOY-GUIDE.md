# FTP-deploy via GitHub Actions — hoe dit werkt (en hoe je dit meeneemt naar nieuwe projecten)

> Herbruikbare kennis uit het opzetten van de deploy-workflow (Astro static build → shared hosting via FTP/FTPS). Bedoeld om te kopiëren/aanpassen voor projecten met vergelijkbare hosting (DirectAdmin/Xynta-achtige shared hosting zonder Node.js-runtime). Zie ook STACK.md §7 (Deployment) voor het grotere plaatje.

## De werkende opzet

Bestand: `.github/workflows/deploy.yml`

```yaml
name: Build and deploy (FTP)

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Build (static output)
        run: npm run build

      - name: Deploy dist/ via FTP
        uses: SamKirkland/FTP-Deploy-Action@v4.3.5
        with:
          server: ${{ secrets.FTP_SERVER }}
          username: ${{ secrets.FTP_USERNAME }}
          password: ${{ secrets.FTP_PASSWORD }}
          protocol: ${{ vars.FTP_PROTOCOL || 'ftps' }}
          server-dir: ${{ vars.FTP_SERVER_DIR || './' }}
          local-dir: dist/
```

**Waarom deze actie en niet SSH/rsync/SFTP:** we hebben dat allemaal geprobeerd (zie hieronder) voordat we teruggingen naar deze simpele, populaire FTP-actie. Voor gewone shared hosting zonder SSH-toegang op accountniveau is dit de kortste weg naar een werkende deploy.

### Wat je nodig hebt van de hostingpartij

| Secret/variabele | Type | Waar te vinden |
|---|---|---|
| `FTP_SERVER` | GitHub **secret** | Meestal het domein of het IP-adres uit het hostingpaneel (bijv. DirectAdmin → FTP-accounts) |
| `FTP_USERNAME` | GitHub **secret** | FTP-accountnaam |
| `FTP_PASSWORD` | GitHub **secret** | FTP-wachtwoord |
| `FTP_PROTOCOL` | GitHub **variable** (optioneel) | `ftps` (default, aanbevolen) of `ftp` als de host geen FTPS ondersteunt |
| `FTP_SERVER_DIR` | GitHub **variable** (optioneel) | Het pad ná inloggen waar de site moet komen, bijv. `domains/preview.verder.studio/public_html/`. Zonder deze variabele wordt de root van het FTP-account gebruikt (`./`) |

Zet secrets/variables in: **Repo → Settings → Secrets and variables → Actions** (secrets en variables staan op aparte tabbladen).

**Veelgemaakte fout:** als het FTP-account al is ingesteld met een home-directory die *al* op `public_html` wijst, laat `FTP_SERVER_DIR` dan leeg (of `./`) — anders upload je naar `public_html/public_html/` en zie je een lege of dubbel-geneste site.

## Troubleshooting-logboek (wat we tegenkwamen, en wat het betekende)

Deze foutmeldingen komen terug bij vrijwel elke shared-hosting-FTP-deploy. Bewaar dit rijtje.

### `ECONNRESET` op de data-socket (zowel bij `ftp` als `ftps`)
**Betekenis:** de control-connectie lukt, maar de host breekt de data-connectie af zodra er bestanden overgedragen worden. Bijna altijd een **passive-mode firewall-probleem** aan de kant van de hosting (de host geeft een passive-poort terug die van buitenaf niet bereikbaar is, of de firewall blokkeert het poortbereik).
**Fix:** dit is zelden op te lossen vanuit de GitHub Actions-config. Vraag de hosting-support om het passive-poortbereik vrij te geven, of probeer FTPS i.p.v. FTP (of andersom).

### `ETIMEDOUT` bij het opzetten van de control-connectie (poort 21)
**Betekenis:** anders dan `ECONNRESET` — hier komt er helemaal geen verbinding tot stand, de pakketten worden stilzwijgend genegeerd (`DROP`, geen `REJECT`). Typisch gedrag van een firewall met brute-force-bescherming (CSF/LFD, fail2ban) die het IP-adres van de GitHub-runner blokkeert of rate-limit.
**Belangrijk:** GitHub Actions-runners krijgen **elke run een ander IP-adres**. Als een deploy hierop faalt terwijl exact dezelfde config eerder wél werkte, is de eerste stap gewoon **de workflow opnieuw draaien** (Actions-tab → mislukte run → "Re-run jobs") — vaak lost dat het al op. Blijft het terugkomen, dan is een structureel IP-allowlist-probleem bij de host waarschijnlijker.

### `Network is unreachable` / `ENETUNREACH`
**Betekenis:** de GitHub-runner probeert eerst IPv6, en de host heeft geen (werkende) IPv6 — de runner valt daarna terug op IPv4, wat een paar seconden vertraging geeft maar op zich geen fatale fout hoeft te zijn (zie hierboven, vaak in combinatie met `ETIMEDOUT` op het IPv4-adres). Bij zelfgebouwde SSH/rsync-commando's kun je dit hard fixen door `-4` toe te voegen om IPv4 te forceren.

### SSH-alternatief (`web-deploy`/`easingthemes/ssh-deploy`) — waarom we dit hebben laten vallen
We hebben geprobeerd via SSH + rsync te deployen (had voordelen: geen losse FTP-poort-gedoe, incrementele sync). Problemen die we tegenkwamen:
- `error in libcrypto` met een ed25519-sleutel + de `web-deploy`-action — een bekende, onopgeloste bug in die specifieke actie-library-combinatie. Oplossing: RSA/PEM-sleutel gebruiken i.p.v. ed25519, of een andere action.
- De FTP-sub-account had geen SSH-toegang — dat vereiste het **hoofdaccount** van de hosting (DirectAdmin-login), niet het losse FTP-account. Niet elke hostingpartij staat SSH sowieso toe op shared hosting.
- Per saldo: als de hosting geen volwaardige SSH-toegang met een stabiele gebruiker biedt, is FTP/FTPS pragmatischer dan er tegenaan blijven vechten.

### SFTP-alternatief
- SFTP via `sshpass` + `sftp` bleek onbetrouwbaar (verbindingen die zomaar afbraken); `sshpass` + `ssh "mkdir -p ..."` (los commando, geen sftp-batch) werkte wel betrouwbaarder voor het vooraf aanmaken van directories.
- `sftp put -r` maakt **geen** remote directories automatisch aan — die moeten van tevoren bestaan.

### Shell-injectie bij het los interpoleren van secrets
Interpoleer secrets **nooit** direct in een `run:`-scriptregel (`ssh ${{ secrets.PASSWORD }} ...`) — geef ze door via een `env:`-blok en gebruik `$VAR` in het script. Anders loop je risico op shell-injectie als een secret toevallig speciale tekens bevat, en het is sowieso makkelijker te debuggen.

## Checklist voor een nieuw project

1. **Vraag bij de start van het project** naar: FTP-hostnaam, gebruikersnaam, wachtwoord, en of het account al op `public_html` staat of dat je zelf het pad moet opgeven.
2. Kopieer `.github/workflows/deploy.yml` (zie hierboven) als startpunt.
3. Zet de 3 secrets (`FTP_SERVER`, `FTP_USERNAME`, `FTP_PASSWORD`) en desgewenst de 2 variables (`FTP_PROTOCOL`, `FTP_SERVER_DIR`) klaar in GitHub.
4. Push naar `main` en check de Actions-tab. Faalt de eerste run met `ETIMEDOUT`? Probeer eerst gewoon **opnieuw** voordat je support gaat bellen.
5. Faalt het structureel met `ECONNRESET` of blijvende timeouts? Neem contact op met de hosting-support en vraag specifiek naar: (a) passive-FTP-poortbereik vrijgeven voor externe verbindingen, (b) of het IP-bereik van GitHub Actions-runners geblokkeerd wordt door hun firewall.
6. Overweeg SSH/rsync alleen als de hosting een stabiel hoofdaccount met SSH-toegang biedt — voor gewone shared hosting is FTPS via deze actie de pragmatische default.

## Bonus: preview-only deploys niet laten indexeren

Als je eerst op een preview-subdomein deployt voordat de site live gaat: koppel indexering aan een env-var (`PUBLIC_SITE_INDEXABLE`) i.p.v. hem los te regelen. Zie `src/layouts/Layout.astro` en `src/pages/robots.txt.ts` in een project dat dit patroon toepast — standaard `noindex` + `robots.txt: Disallow: /`, pas aan te zetten door de variabele in de workflow op `true` te zetten voor de echte productie-deploy.
