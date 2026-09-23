# 06 – Z-offset a správná výška trysky

BLTouch už bezpečně funguje a tiskárna umí udělat Z-home. Teď musíme Klipper naučit skutečnou vzdálenost mezi okamžikem sepnutí sondy a polohou trysky vůči podložce.

Tomu říkáme **Z-offset**.

> [!WARNING]
> Nekopíruj Z-offset z jiné tiskárny. Závisí na sondě, držáku, hotendu, trysce i mechanickém sestavení. Dokonce i po výměně trysky nebo zásahu do hotendu může být potřeba jej znovu zkontrolovat.

## 1. Připrav tiskárnu

Podložka i tryska musí být čisté.

Na špičce trysky nesmí viset zbytek filamentu, který by při kalibraci předstíral její skutečnou výšku.

Pro základní kalibraci si připrav obyčejný kancelářský papír.

## 2. Zahomuj tiskárnu

Do konzole zadej:

```text
G28
```

Po předchozích kapitolách už musí být bezpečně ověřené X, Y i BLTouch/Z-home.

## 3. Spusť kalibraci sondy

Do konzole napiš:

```text
PROBE_CALIBRATE
```

Klipper provede měření sondou a následně umožní ručně doladit skutečnou polohu trysky.

Pod trysku vlož papír.

## 4. TESTZ – pohyb po malých krocích

Polohu můžeš měnit příkazem:

```text
TESTZ Z=-1
```

Záporná hodnota posouvá trysku blíž k podložce.

Když se přiblížíš, používej menší kroky:

```text
TESTZ Z=-0.1
```

a později například:

```text
TESTZ Z=-0.05
```

Pokud jsi naopak příliš nízko:

```text
TESTZ Z=0.1
```

> [!CAUTION]
> Čím blíž jsi podložce, tím menší kroky používej. Není důvod posílat `-1 mm`, když je tryska už téměř na papíru.

## 5. Jak poznat správnou výšku

Papírem pod tryskou lehce pohybuj tam a zpět.

Cílem je bod, kdy tryska začne papír **lehce přibrzďovat**, ale papír stále dokážeš posouvat.

Nejde o to papír tryskou pevně sevřít.

Papír je praktická pomůcka pro základní nastavení, nikoliv laboratorní měřidlo. Finální první vrstvu ještě později jemně doladíme při skutečném tisku.

## 6. ACCEPT

Jakmile jsi s polohou spokojený, napiš:

```text
ACCEPT
```

Tím potvrdíš naměřenou polohu.

Kalibrace ale ještě není trvale uložená.

## 7. SAVE_CONFIG

Teď spusť:

```text
SAVE_CONFIG
```

Klipper uloží vypočítaný Z-offset do automaticky generované části konfigurace a restartuje se.

Na konci `printer.cfg` pak můžeš vidět například:

```text
#*# [bltouch]
#*# z_offset = ...
```

Konkrétní číslo bude tvoje vlastní.

**Nekopíruj ho do návodu ani z jiné tiskárny.**

## 8. Ověř výsledek

Po restartu znovu:

```text
G28
```

Potom můžeš bezpečně zkontrolovat pohyb Z.

Stále měj na paměti, že Z=0 představuje referenční výšku, kterou jsme právě nastavili. Při prvních pohybech směrem k podložce sleduj trysku.

## 9. Papír není konečná kalibrace první vrstvy

`PROBE_CALIBRATE` nám vytvoří velmi dobrý základ.

Skutečnou první vrstvu ale ovlivňuje například:

- materiál,
- teplota trysky a bedu,
- stav povrchu,
- průtok,
- mechanika tiskárny.

Proto při prvním reálném tisku sleduj první vrstvu a případně proveď malé doladění.

Velké korekce ale nejsou normální. Pokud musíš Z výrazně měnit, vrať se ke kalibraci a zkontroluj sondu a mechaniku.

## 10. Kdy Z-offset zkontrolovat znovu

Z-offset je vhodné znovu ověřit například po:

- výměně trysky,
- demontáži nebo výměně hotendu,
- změně držáku BLTouch,
- posunutí sondy,
- větším zásahu do toolheadu,
- mechanické změně, která ovlivní vzájemnou výšku sondy a trysky.

Samotný uložený Z-offset není hodnota, kterou nastavíš jednou navždy bez ohledu na změny tiskárny.

## Co máš mít po této kapitole hotové

- provedl jsi `PROBE_CALIBRATE`,
- pomocí `TESTZ` jsi bezpečně našel polohu trysky,
- použil jsi `ACCEPT`,
- hodnotu jsi uložil přes `SAVE_CONFIG`,
- rozumíš tomu, proč se cizí Z-offset nekopíruje,
- tiskárna má základ pro bezpečnou první vrstvu.

Další krok:

➡️ **07 – Bed Mesh: změření podložky**
