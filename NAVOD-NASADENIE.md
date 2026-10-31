# WoodPlast 3D konfigurátor – nasadenie na woodplast.sk

Verzia konfigurátora: **123** (31. 10. 2026). Balík obsahuje všetko potrebné; nasadenie zvládne správca webu za hodinu.

## Aktuálne nasadenie: GitHub Pages (od 24. 10. 2026)

Konfigurátor beží na **https://vecilazo155-hash.github.io/Konfigurator/** z repozitára `vecilazo155-hash/Konfigurator`
(súbory `index.html`, `sw.js`, `embed-priklad.html`; nasadenie robí automaticky workflow `.github/workflows/pages.yml`
pri každom push do `main`). Aktualizácie robí WoodPlast cez Claude priamo v repozitári – správca webu nič nenahráva.
V hostovanej kópii je `forceMake: true`, takže dopyty idú cez Make aj z domény github.io.

Správca webu urobí jednorazovo: vloží iframe na stránku (krok 2 nižšie, adresa iframe = github.io) a pridá DNS
`CNAME konfigurator → vecilazo155-hash.github.io`. Po DNS sa do repozitára pridá súbor `CNAME` s obsahom
`konfigurator.woodplast.sk` a v nastaveniach GitHub Pages sa zapne „Enforce HTTPS“ – konfigurátor potom beží na
https://konfigurator.woodplast.sk/. Kroky 1 (FTP) nižšie sú alternatíva, ak by mal konfigurátor bežať priamo na hostingu webu.

## Čo je v balíku

| Súbor / priečinok | Na čo slúži |
|---|---|
| `konfigurator/index.html` | celý konfigurátor (jeden súbor, ~4,7 MB – obsahuje 3D, textúry dekorov, cenník, plánovač) |
| `konfigurator/sw.js` | offline kópia a inštalácia na plochu telefónu (PWA) – nahrať vedľa index.html |
| `konfigurator/embed-priklad.html` | príklad stránky, ktorá konfigurátor vloží cez iframe |
| `poradca-worker/` | služba AI poradcu pre web (Cloudflare Worker) – samostatný návod v `README.md` |
| `nastroje/nahrad_dekor.py` | výmena fotky dekoru v konfigurátore |
| `SPRAVCA.md` | dokumentácia pre správcu: ako meniť ceny, dekory, videá, texty |

## Krok 1 – nahrať konfigurátor

1. Na webe vytvorte priečinok `/konfigurator/` (napr. cez FTP alebo správcu súborov hostingu).
2. Nahrajte doň `index.html` a `sw.js` z priečinka `konfigurator/`.
3. Otvorte `https://www.woodplast.sk/konfigurator/` – musí sa zobraziť výber produktu a 3D terasa.

Podmienky: stránka musí ísť cez **https** (kamera, AR, zdieľanie a PWA inak nefungujú) a server musí posielať `index.html` ako `text/html; charset=utf-8` (bežný hosting to robí sám).

Dopyty sa na doméne woodplast.sk posielajú automaticky na existujúce webhooky Make (predaj@woodplast.sk) – rovnaké ako pôvodný konfigurátor. Na inej doméne (skúšobná) by sa otvoril e-mailový program; ak chcete Make aj na skúšobnej doméne, pozrite `SPRAVCA.md` → Dopyty.

## Krok 2 – vložiť na stránku webu (voliteľné)

Konfigurátor môže bežať na vlastnej adrese `/konfigurator/`, alebo ho vložíte do existujúcej stránky webu:

1. Otvorte `konfigurator/embed-priklad.html` – je v ňom hotový kód `<iframe>` + skript. Skopírujte ho do stránky (WordPress: blok „Vlastné HTML“).
2. Skript sám prispôsobuje výšku iframe obsahu (žiadne dvojité posúvanie) a odovzdáva odkazy `#k=…` na konfiguráciu do iframe.
3. V `index.html` hore v bloku **NASTAVENIA PRE SPRÁVCU WEBU** nastavte `embedLink` na adresu tejto stránky, napr. `"https://www.woodplast.sk/konfigurator-terasy"` – odkazy na zdieľanie a QR kódy potom vedú na stránku webu.

Parameter `?embed=1` v adrese iframe skryje hornú lištu konfigurátora (logo, telefón) – stránka webu má vlastné.

Ten istý kód (aj s vysvetlením správ, ktoré konfigurátor posiela stránke) je aj priamo v konfigurátore: `…/konfigurator/?obchod` → panel „Pravidlá výpočtu“ → „Vložiť na web (embed)“.

## Krok 3 – AI poradca na webe (voliteľné, 5 minút)

V ukážke na claude.ai odpovedá poradca cez Claude priamo. Na webe potrebuje malú službu s API kľúčom – je pripravená v `poradca-worker/` (Cloudflare Worker, bezplatný plán stačí). Postup je v `poradca-worker/README.md`; po nasadení vložte adresu služby do `advisorEndpoint` v nastaveniach správcu. Bez služby odpovedá na webe vstavaný poradca s pripravenými odpoveďami na časté otázky.

## Krok 4 – nastavenia správcu

Hore v `index.html` je blok `window.WP3D_CONFIG = { … }`. Pri nasadení skontrolujte:

- `advisorEndpoint` – adresa služby poradcu (krok 3) alebo prázdne,
- `embedLink` – adresa stránky s iframe (krok 2) alebo prázdne,
- `videos` – videonávody k montáži (YouTube), ak ich máte,
- `rules` – prípadné úpravy cenníka (inak ostávajú ceny z kódu).

Podrobnosti v `SPRAVCA.md`.

## Krok 5 – Make: nové polia v dopyte

Konfigurátor posiela dopyt v rovnakom tvare ako pôvodný (`wizard-final-submit` / `atypical-shape-contact`), scenáre teda fungujú bez zmeny. Navyše posiela nové polia, ktoré sa oplatí dať do e-mailu obchodníkovi:

| Pole | Obsah |
|---|---|
| `samples` | vzorky dekorov zdarma: `[{id, name, mat}]` |
| `sitePhotos` / `attachments` | fotky miesta realizácie (JPEG, dataUrl) – pri `atypical` ako prílohy e-mailu |
| `installEstimate` | odhad montáže `{h, crew, days, what}` |
| `vatMode`, `vatRate` | `material-incl-vat` / `material-ex-vat`, sadzba DPH |
| `followUp` | názov, suma, odkaz, platnosť ponuky 30 dní (pre pripomienku) |
| `callback` | „Zavolajte mi“ – deň a čas |
| `offerPdf`, `offerEmail` | PDF ponuky (base64) a hotový e-mail pre zákazníka – pozri SPRAVCA.md 6.1 |

Text dopytu (`summaryText`, `form.note`) tieto údaje obsahuje aj bez úprav scenára (vzorky, fotky, DPH, odhad montáže sú v ňom vypísané).

## Krok 5b – Make: e-mail s ponukou zákazníkovi a štatistiky (voliteľné)

Dva malé scenáre navyše (postup s mapovaním polí je v `SPRAVCA.md` → 6.1 a 6.2):

- **Ponuka zákazníkovi e-mailom** – Webhook → Email: pošle zákazníkovi PDF ponuku a odkaz hneď po dopyte. Adresu webhooku do `offerEmailWebhook`.
- **Štatistiky** – Webhook → Google Sheets (riadok na udalosť) + tabuľka publikovaná ako CSV do `analytics.sheetCsv`; prehľad je v paneli `?obchod` → Štatistiky.
  Alebo Google Analytics 4 cez `analytics.ga4`.

## Krok 6 – kontrola pred spustením

- [ ] `https://www.woodplast.sk/konfigurator/` sa načíta na počítači aj na telefóne (iPhone Safari, Android Chrome)
- [ ] 3D sa otáča, pôdorys funguje, menu „Viac ▾“ sa otvára
- [ ] terasa 4 × 3 m, betón, výška 5 cm, Dub → cena 1 361,07 € s montážou (bez dopravy)
- [ ] zadanie obce doplní dopravu
- [ ] „Poslať dopyt“ s testovacím e-mailom príde na predaj@woodplast.sk (cez Make) – aj so vzorkami a fotkou miesta
- [ ] „Uložiť a zdieľať“ → odkaz otvorí tú istú konfiguráciu (aj z iného telefónu); WhatsApp / e-mail sa otvorí
- [ ] „Cenová ponuka (PDF)“ → „Stiahnuť PDF“ uloží A4 ponuku; tlač / uložiť ako PDF funguje tiež
- [ ] „Plán montáže“ → nákres, DXF a SVG sa stiahnu
- [ ] poradca odpovedá (AI alebo vstavaný)
- [ ] prepnutie jazyka EN / DE / HU
- [ ] na telefóne „Viac ▾ → Pridať na plochu telefónu“ (Android ponúkne inštaláciu; iPhone: Zdieľať → Pridať na plochu)
- [ ] `?obchod` – panel pravidiel pre obchodníka (nie je verejný, len cez parameter v adrese)

## Aktualizácie

Nová verzia = nahradiť `index.html` (nastavenia správcu v hlavičke si pred tým skopírujte a vložte do novej verzie). Ak používate `sw.js`, zmeňte v ňom `CACHE='wp3d-v1'` na `v2`, `v3`… – telefóny s nainštalovanou aplikáciou si potom stiahnu novú verziu.

Predchádzajúcu verziu si vždy nechajte (napr. `index-v119.html`), aby sa dalo vrátiť späť.
