# WoodPlast 3D konfigurátor – hosting

Verejná adresa: **https://vecilazo155-hash.github.io/Konfigurator/**  
(po nastavení DNS aj **https://konfigurator.woodplast.sk/**)

Na stránku woodplast.sk sa vkladá cez `<iframe>` – hotový kód je v `embed-priklad.html`.
Aktualizácie robí WoodPlast cez Claude priamo v tomto repozitári; adresa ani kód iframe sa nemenia.

| Súbor | Na čo slúži |
|---|---|
| `index.html` | celý konfigurátor (jeden súbor); hore v bloku `WP3D_CONFIG` sú nastavenia správcu |
| `sw.js` | offline kópia a inštalácia na plochu telefónu (PWA) |
| `embed-priklad.html` | príklad stránky s iframe + skript na výšku a odkazy `#k=` |

Dopyty idú cez webhooky Make na predaj@woodplast.sk (`forceMake: true`).
