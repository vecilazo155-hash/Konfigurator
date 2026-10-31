# WoodPlast 3D konfigurátor – dokumentácia pre správcu webu

Ako meniť ceny, dekory, videá a texty bez programátora. Konfigurátor je jeden súbor `index.html`; upravuje sa v textovom editore
(Visual Studio Code, Notepad++, na Macu TextEdit v režime obyčajného textu). Súbor má ~4,7 MB – väčšinu tvoria fotky dekorov a 3D knižnica,
texty a ceny sú v prehľadných blokoch, ktoré nájdete vyhľadaním (Ctrl+F).

**Zlaté pravidlo:** pred každou zmenou si odložte kópiu súboru (napr. `index-v120.html`) a po zmene otvorte konfigurátor v prehliadači – ak by sa
nenačítal (biela stránka), je v úprave preklep (najčastejšie chýbajúca čiarka alebo úvodzovka) a vrátite kópiu.

---

## 1. Nastavenia správcu (hlavička súboru)

Hore v súbore je blok označený **NASTAVENIA PRE SPRÁVCU WEBU**:

```js
window.WP3D_CONFIG = {
  advisorEndpoint: "",   // adresa služby AI poradcu (viď poradca-worker/README.md)
  videos: "",            // videonávody pri krokoch montáže
  embedLink: "",         // adresa stránky webu, kde je konfigurátor vložený cez iframe
  rules: {},             // úpravy cenníka bez zásahu do kódu
  makeWebhooks: null,    // iné webhooky Make pre dopyty
  inquiryEndpoint: "",   // vlastná služba pre dopyty
  photoEndpoint: "",     // služba na rozpoznanie okien z fotky
  forceMake: false,      // true = dopyty cez Make aj mimo domény woodplast.sk (hosting na GitHub Pages)
  offerEmailWebhook: "", // webhook Make „Ponuka zákazníkovi“ – e-mail s PDF hneď po dopyte (kap. 6.1)
  analytics: { ga4: "", consent: "", webhook: "", sheetCsv: "" } // štatistiky (kap. 6.2)
};
```

Toto je jediné miesto, ktoré sa pri nasadení musí upraviť. Hodnoty sú v úvodzovkách, položky oddelené čiarkou.

---

## 2. Ceny (cenník)

### 2.1 Rýchle skúšanie – panel obchodníka
Otvorte konfigurátor s parametrom **`?obchod`** (napr. `https://www.woodplast.sk/konfigurator/?obchod`). V paneli vpravo úplne dole je
**Pravidlá výpočtu** – každá cena z cenníka ako pole. Zmena sa hneď prejaví v cene, ukladá sa len v tomto prehliadači (zákazníci ju nevidia)
a rozpis aj ponuka sa označia ako „podľa upravených pravidiel“. Slúži na skúšanie a na výpočet individuálnej ponuky obchodníkom.
„Obnoviť pravidlá z cenníka“ vráti pôvodné hodnoty; „Ukončiť režim obchodníka“ panel schová.

### 2.2 Trvalá zmena pre všetkých zákazníkov
Dve možnosti:

**a) Cez nastavenia správcu** (odporúčané – bez zásahu do kódu): do `rules` vpíšte kľúč a novú hodnotu, napr.
```js
rules: { boardBm: 10.5, boardBmSelf: 12.2, montazBeton: 24, dph: 23 },
```
Kľúče sú v tabuľke nižšie (alebo v paneli ?obchod: podržte myš nad poľom). Desatinná bodka, nie čiarka.

**b) Priamo v kóde**: vyhľadajte `var RULE_DEFS` – zoznam riadkov `{k:'boardBm', label:'Terasová doska s montážou (€/bm)', v:9.9, p:'terrace'}`;
zmeňte číslo `v:`.

| Kľúč | Význam | Predvolené |
|---|---|---|
| `boardBm` / `boardBmSelf` | WPC doska s montážou / rezaná bez montáže (€/bm) | 9,9 / 11,5 |
| `montazBeton`, `montazStrk`, `montazDlazba`, `montazPasy` | montáž terasy podľa podkladu (€/m²) | 23 / 25 / 23 / 23 |
| `montazCustom`, `montazCustomStrk` | montáž na vlastnú konštrukciu (€/m²) | 21 / 23 |
| `montazSchod` | montáž schodu (€/m² stupňov) | 35 |
| `montazMin` | minimálna cena montáže (€) | 250 |
| `dph` | DPH na materiál (%) – montáž a doprava sú bez DPH | 23 |
| `listaPrice`, `listaWidePrice`, `listaScrewPrice` | lišta 2,9 m, široká lišta 5,8 m, skrutka k lište | 16 / 55 / 0,33 |
| `terceScrew` | skrutka k terču | 0,05 |
| `joistMax`, `joistMaxPiece` | rozostup hranolov WPC / Renolit a Natura (cm) | 38 / 35 |
| `strkMinH`, `obrubnikyPerM2` | štrk: min. výška (mm), obrubníky (ks/m²) | 100 / 1,5 |
| `doprava50`, `doprava100`, `dopravaViac` | doprava €/km podľa plochy (tam aj späť) | 0,6 / 1,1 / 1,65 |
| `montHodTer`, `montHodFas`, `montHodPlot`, `montHodSln`, `montHodInt`, `montDen` | odhad času montáže (h na m², bm, ks; dĺžka dňa) | 1 / 1,3 / 1,6 / 0,5 / 0,7 / 8 |
| `drevoM2`, `drevoM2Mat`, `drevoUdrzbaM2`, `drevoHodM2`, `drevoOprava`, `wpcUdrzbaM2` | porovnanie s drevom na 10 rokov | 85 / 55 / 6 / 0,25 / 15 / 0,2 |
| `inMontAk`, `inMontFx`, `inMontFxBath`, `inMontSd`, `inGlue`, `inGluePer*` | interiér: montáž panelov, lepidlo | 27 / 53 / 93 / 53 / 10,2 / 1 |
| `lnFull`, `lnHalf`, `lnFoot`, `lnRail`, `lnMont`, `lnTrip`, `lnDeduct` | slnolamy | 85 / 43 / 13,4 / 156 / 25 / 50 / 6 |
| `feB58`, `feB29`, `feB19`, `feB144`, `fePostS/M/L`, `feU2525`, `feU`, `feUCoat`, `feBottom`, `feSeal`, `feDeckBm`, `feMontM2` | plot | viď panel ?obchod |

### 2.3 Ceny, ktoré nie sú v pravidlách
Niektoré ceny sú pri materiáli – vyhľadajte v kóde:
- `var TER_MATS` – Renolit a Natura: `piecePrice` (cena dosky 5,8 m), `listaPrice` (lišta z dosky).
- `var HRANOLY` – hranoly 15×30, 20×30, 60×20: `price`.
- `var TERCE` – terče po pásmach: `price`.
- `var SPOJKY` – klipy (plastový, nerezový, ClipFix): `price`, `gap` (medzera v mm).
- `var FA_MAT` – fasádne materiály (lamely Solo, ProDuo, WPC, Renolit, Natura): ceny za bm / ks, rozmery.
- `var IN_MAT` – interiérové panely (akustické `ak`, Flexy `fx`, kamenná dyha `sd`): cena za panel, rozmery, dekory.

Po zmene v kóde sa cena zmení pre všetkých a v rozpise sa nič neoznačuje (je to „cenník“).

---

## 3. Dekory a fotky

### 3.1 Zoznam dekorov
- Terasa WPC: `var DECORS = [ {id:'dub', name:'Dub', c1:'#8E6A47', c2:'#7…'}, … ]` – `id` (nemení sa, viaže fotku a odkazy), `name` (zobrazený názov),
  `c1`/`c2` (farby pre náčrt a 3D bez fotky). Renolit a Natura majú dekory v `TER_MATS` → `colors`.
- Široká lišta 14 cm je len pre dekory v `var WIDE_DECORS={breza:1, dub:1, dubsedy:1, smrek:1}` – nový dekor so širokou lištou sem pridajte.
- Fasáda: `var FA_COL` (názov, farba) a v `FA_MAT` pri každom materiáli `colors` (ktoré dekory má).
- Plot: `var FE_DECORS`; slnolamy: `var LN_DECORS`; interiér: v `IN_MAT` pri každom materiáli `decors`.

Pridanie nového dekoru = nový riadok do zoznamu (skopírujte existujúci, zmeňte `id`, `name`, farby) + fotka (3.2). Odstránenie = riadok zmazať
(staré odkazy s týmto dekorom sa otvoria s prvým dekorom v zozname).

### 3.2 Fotky dekorov
Fotky sú uložené priamo v súbore (blok `<script type="application/json" id="wpTex">`), každá ako pás 1024 × 192 px. Vymeníte ich nástrojom:
```bash
python3 nastroje/nahrad_dekor.py index.html --zoznam           # zoznam dekorov a ich fotiek
python3 nastroje/nahrad_dekor.py index.html dub nova_fotka.jpg  # vymení fotku dekoru „dub“
```
Nástroj fotku oreže a zmenší na správny rozmer a urobí zálohu `index.html.zaloha`. Pre fasádne lamely je kľúč `lam|lamel-solo|oak`,
pre plot `fe-dub`, pre interiér `ak-dub`, `fx-blanco`, `sd-…`. Fotka má byť záber dosky pozdĺžne (vlákno vodorovne), dobre osvetlená, bez lesku.

### 3.3 Fotky realizácií na úvodnej obrazovke
Dlaždice produktov (`<button class="ptile" data-pick="terrace">` …) majú `<img src="data:image/jpeg;base64,…">`. Novú fotku prevediete na
base64 (napr. `base64 -w0 fotka.jpg` alebo online nástroj) a vložíte namiesto pôvodného reťazca. Odporúčaný rozmer 1200 × 800 px, JPEG do 150 kB.

---

## 4. Videonávody pri krokoch montáže
V nastaveniach správcu do `videos` vpíšte riadky „názov kroku = odkaz YouTube“; `*` platí pre všetky kroky:
```js
videos: "Pripraviť podklad = https://youtu.be/XXXXXXXXXXX\nKlásť dosky a klipy = https://youtu.be/YYYYYYYYYYY\n* = https://youtu.be/ZZZZZZZZZZZ",
```
`\n` oddeľuje riadky. Názvy krokov sú v pláne montáže (stačí začiatok názvu, bez ohľadu na veľkosť písmen). Video sa načíta až po kliknutí
na „▶ Pozrieť video k tomuto kroku“ (youtube-nocookie). Obchodník si môže videá nastaviť aj dočasne v paneli ?obchod → Videonávody.

---

## 5. Texty

- **Texty v paneli a na obrazovkách** sú priamo v HTML (`<h3>Tvar terasy</h3>`, `<p class="note">…`) – vyhľadajte presnú vetu a prepíšte ju.
- **Texty, ktoré sa skladajú z častí** (voľby, popisy materiálov, hlášky, rozpis ceny) sú v JavaScripte v úvodzovkách `'…'` – hľadajte tiež presnú vetu.
  Pozor na apostrof vnútri textu: v `'…'` ho zapíšte ako `\'`.
- **Montážny návod** (kroky „Pripraviť podklad“ …) – vyhľadajte `function planSteps` – každý krok je `{h:'nadpis', p:'text', k:'poznámka', m:[…]}`.
- **Texty poradcu**: pravidlá pre AI sú v `var ADVISOR_SYS`, vstavané odpovede na časté otázky v `var ADVISOR_FAQ` (kľúčové slová + odpoveď –
  stačí doplniť riadok v rovnakom tvare).
- **Preklady EN / DE / HU**: slovník `var I18N={ "Slovenský text":["English","Deutsch","Magyar"], … }` (dva bloky – `I18N` a doplnok v `i18n2`).
  Keď zmeníte slovenský text v rozhraní, zmeňte rovnako aj kľúč v slovníku, inak sa prestane prekladať. Dlhé texty návodu sa neprekladajú
  (prehliadač ich preloží sám).
- **Kontakt**: e-mail `predaj@woodplast.sk` je v `var INQUIRY_EMAIL`, telefón `+421 949 606 767` a adresa `Lieskovská cesta 4379` sú v HTML a
  v niekoľkých textoch – vyhľadajte ich.

---

## 6. Dopyty (Make / e-mail)
- Na doméne woodplast.sk idú dopyty na webhooky Make (`var MAKE_WEBHOOKS` – terasa, fasáda, ostatné „atypical“). Iné adresy nastavíte
  v `makeWebhooks` v nastaveniach správcu. Na inej doméne sa otvorí e-mailový program zákazníka.
- Tvar dopytu je zhodný s pôvodným konfigurátorom (`wizard-final-submit`, `atypical-shape-contact`); nové polia sú v `NAVOD-NASADENIE.md` → Krok 5.
- Čas hovoru („Zavolajte mi“): pracovné dni 8:30 – 15:30, najbližších `CALL_DAYS` dní (vyhľadajte `CALL_FROM`, `CALL_TO`, `CALL_DAYS`).
- Vzorky zdarma: najviac `SAMPLES_MAX` (3) dekory; fotky miesta: najviac `SITE_MAX` (4), zmenšené na 1600 px.
- **PDF ponuky v dopyte (od v122):** pri dopyte s cenou konfigurátor zostaví cenovú ponuku ako PDF priamo v prehliadači (logo, parametre,
  obrázok 3D, rozpis, DPH, QR kód) a pošle ju v dopyte ako `offerPdf` (`{name, mimeType, base64}`) aj v `attachments`. Rovnaké PDF si zákazník
  stiahne tlačidlom „Stiahnuť PDF“ v cenovej ponuke. Spolu s ním ide `offerEmail` – hotový e-mail pre zákazníka (`to, subject, html, text`).

### 6.1 Automatická ponuka zákazníkovi e-mailom (Make, 5 minút)

Zákazník dostane PDF ponuku a odkaz na konfiguráciu hneď po odoslaní dopytu (kým je „horúci“). Treba jeden nový scenár Make:

1. **Nový scenár → Webhooks → Custom webhook** (napr. „WP3D ponuka zákazníkovi“). Skopírujte adresu webhooku.
2. Pridajte modul **Email → Send an email** (alebo Gmail → Send an email) a namapujte:

| Pole | Hodnota |
|---|---|
| To | `{{1.email.to}}` |
| Subject | `{{1.email.subject}}` |
| Content type | HTML |
| Content | `{{1.email.html}}` |
| Attachments → File name | `{{1.pdf.name}}` |
| Attachments → Data | `{{toBinary(1.pdf.base64; "base64")}}` |
| BCC (voliteľné) | predaj@woodplast.sk – kópia pre obchodníka |

3. Scenár zapnite (Scheduling: Immediately) a adresu webhooku vložte do `offerEmailWebhook` v nastaveniach správcu.
4. Skúška: pošlite testovací dopyt s vlastným e-mailom – do minúty príde e-mail „Vaša cenová ponuka WoodPlast – …“ s PDF.

Konfigurátor na webhook posiela `{kind:"offer-email", email:{to,name,subject,html,text,link,code,total,totalText,product,title}, pdf:{name,mimeType,base64}, form:{…}, url}`.
Pri dopyte bez ceny (iný tvar, chyba zadania) sa e-mail neposiela – ponuku pripraví obchodník. Ak `offerEmailWebhook` ostane prázdny,
PDF je aj tak v hlavnom dopyte a môže ho poslať existujúci scenár.

### 6.2 Štatistiky – čo ľudia konfigurujú, odkiaľ sú, koľko pošle dopyt

Dve nezávislé cesty, obe bez cookies a bez osobných údajov (obec je tá, ktorú zákazník zadal pre dopravu):

**A) Vlastný prehľad cez Make → Google tabuľka → panel `?obchod` → Štatistiky**

1. Vytvorte Google tabuľku s prvým riadkom (názvy stĺpcov presne takto):
   `ts, event, session, product, shape, decor, mat, area, total, priced, city, km, lang, device, ref, utm, embed, reason, samples, photos, kind`
2. Make: **Webhooks → Custom webhook** → **Google Sheets → Add a row**; každý stĺpec namapujte na rovnomenné pole z webhooku (`{{1.ts}}`, `{{1.event}}`, …).
3. Adresu webhooku vložte do `analytics.webhook`. Konfigurátor posiela 1–4 riadky na návštevu: `visit` (otvorenie), `product_open` (produkt),
   `config` (snímka konfigurácie pri odchode zo stránky alebo zmene produktu – tvar, dekor, plocha, cena, obec), `inquiry` (dopyt),
   plus `offer_open`, `offer_pdf`, `offer_print`, `share`, `advisor`, `ar`, `guide`, `pwa`, `lang`. Pri bezplatnom pláne Make (1 000 operácií/mesiac)
   to stačí na ~250–300 návštev mesačne; pri väčšej návštevnosti použite GA4 (B) alebo platený plán.
4. Tabuľku publikujte: **Súbor → Zdieľať → Publikovať na webe → Celý dokument → CSV** a adresu vložte do `analytics.sheetCsv`
   (alebo ju vložte priamo v paneli `?obchod` → **Štatistiky** – uloží sa v prehliadači).
5. Panel ukáže: návštevy, otvorené konfigurácie, dopyty a konverziu, návštevy po dňoch, produkty, dekory, tvary, priemernú cenu a plochu,
   obce zákazníkov, zariadenia, jazyky a odkazujúce stránky; obdobie 7 / 30 / 90 dní / všetko.

**B) Google Analytics 4** – do `analytics.ga4` vložte meracie ID („G-…“). Konfigurátor posiela tie isté udalosti ako vlastné udalosti GA
(v GA → Správa → Udalosti ich označte ako konverzie, napr. `inquiry`). GA doplní mapu podľa miest, zariadenia a zdroje návštev.
Súhlas s cookies: bez súhlasu beží GA v režime consent mode (bez cookies, modelované údaje). Ak má web vlastnú lištu súhlasu,
po súhlase pošle stránka do iframe `postMessage({type:"wp3d", cmd:"consent", analytics:true}, "*")`, alebo nastavte `analytics.consent: "granted"`,
ak sa konfigurátor zobrazuje až po súhlase. V ukážke na claude.ai sa GA nenačítava.

---

## 7. Doprava a mapa
Vzdialenosti zo Zvolena sú podľa okresov v `var MAP_OKRES` (súradnice) a `kmOkres` (km po ceste). Nová obec = pridať okres alebo opraviť
vzdialenosť. Sadzby za km sú v pravidlách (`doprava50`, `doprava100`, `dopravaViac`).

---

## 8. Poradca, PWA, jazyky, embed
- **AI poradca**: služba v `poradca-worker/` + `advisorEndpoint`. Bez služby odpovedá vstavaný poradca (`ADVISOR_FAQ`).
- **Pridať na plochu / offline**: potrebuje `sw.js` vedľa `index.html`. Pri každej novej verzii zmeňte v `sw.js` `CACHE='wp3d-v1'` na vyššie číslo.
  Ikona aplikácie je v `var PWA_ICONS` (PNG v base64) – vymeníte rovnakým postupom ako fotky realizácií.
- **Jazyky**: prepínač v hlavičke; predvolený jazyk sa dá vnútiť parametrom `?lang=en`.
- **Vloženie na stránku**: kód v `konfigurator/embed-priklad.html` alebo v konfigurátore `?obchod` → „Vložiť na web (embed)“.
  `?embed=1` skryje hornú lištu.

---

## 9. Parametre v adrese
| Parameter | Význam |
|---|---|
| `?obchod` | panel obchodníka (pravidlá výpočtu, videonávody, embed, import z pôvodného konfigurátora `?cw=`) |
| `?embed=1` | bez hornej lišty (pre iframe) |
| `?lang=en` / `de` / `hu` | jazyk rozhrania |
| `#k=WP2…` | otvorí uloženú konfiguráciu (odkaz na zdieľanie, QR) |
| `#terasa`, `#fasada`, `#plot`, `#slnolamy`, `#interier` | priamo otvorí produkt |

---

## 10. Časté otázky správcu
- **Po úprave je biela stránka** → preklep v JavaScripte; otvorte konzolu prehliadača (F12 → Console), ukáže riadok s chybou. Vráťte zálohu.
- **Cena na webe sa nezmenila** → prehliadač má starú verziu v pamäti; obnovte s Ctrl+F5, pri PWA zmeňte `CACHE` v `sw.js`.
- **Dopyt neprišiel** → skontrolujte doménu (Make len na woodplast.sk) a scenár v Make (webhook aktívny). Text dopytu si zákazník vie aj
  skopírovať – v okne „Dopyt“ je po odoslaní.
- **Obchodník vidí „upravené pravidlá“** → v paneli ?obchod klikol do cien; „Obnoviť pravidlá z cenníka“ to vráti (ukladá sa len v jeho prehliadači).
- **Chcem inú predvolenú terasu** → vyhľadajte `var CFG_BASE` (základná konfigurácia: tvar, rozmery, dekor, podklad…) – pozor, mení aj
  kódovanie odkazov; staré odkazy ostanú platné (ukladajú rozdiely oproti základni).
