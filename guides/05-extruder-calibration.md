# 05 – Kalibrace extruderu a rotation_distance

Teď nastavíme, aby příkaz „vytlač 100 mm filamentu“ znamenal skutečně přibližně 100 mm posunu filamentu.

V Klipperu se pro tuto kalibraci používá hlavně parametr:

```ini
rotation_distance
```

> [!IMPORTANT]
> Nekopíruj `rotation_distance` z jiné tiskárny jen proto, že používá podobný extruder. Převody, podávací kolečka a konkrétní mechanika mohou výslednou hodnotu změnit.

## 1. Nejdřív zkontroluj typ extruderu

V našem základním configu je:

```ini
gear_ratio: 3:1
rotation_distance: 22.28
```

Tento příklad vychází z převodovaného extruderu 3:1.

Pokud máš jiný extruder, `gear_ratio` může být jiný nebo se vůbec nemusí používat.

Nejdřív tedy zjisti, jaký extruder na tiskárně skutečně máš.

## 2. Zahřej hotend

Klipper z bezpečnostních důvodů nedovolí běžnou extruzi studeným hotendem.

Pro PLA můžeš hotend zahřát například na:

```text
200 °C
```

Počkej, až je hotend připravený.

## 3. Připrav si značku na filamentu

Od místa, kde filament vstupuje do extruderu, odměř například:

```text
120 mm
```

a udělej na filamentu tenkou značku fixem.

Proč 120 mm?

Budeme požadovat posun 100 mm a zbývajících 20 mm nám nechá prostor pro změření případné odchylky.

## 4. Přikáž extruderu 100 mm

Do konzole zadej:

```text
M83
G92 E0
G1 E100 F60
```

`M83` nastaví relativní extruzi.

`G92 E0` vynuluje aktuální pozici extruderu.

`G1 E100 F60` požádá o vytlačení 100 mm filamentu rychlostí 60 mm/min, tedy pomalu, aby měření zbytečně neovlivňoval prokluz.

## 5. Změř skutečný posun

Po dokončení znovu změř vzdálenost značky od vstupu extruderu.

Příklad:

Před testem byla značka:

```text
120 mm
```

Po testu zbývá:

```text
24 mm
```

Extruder tedy skutečně posunul:

```text
120 - 24 = 96 mm
```

Požadovali jsme ale 100 mm.

## 6. Výpočet nové rotation_distance

Použij:

```text
nová rotation_distance =
stará rotation_distance × skutečně vytlačená délka / požadovaná délka
```

V našem příkladu:

```text
22.28 × 96 / 100 = 21.3888
```

Nová hodnota by tedy byla přibližně:

```ini
rotation_distance: 21.389
```

Potom konfiguraci ulož a proveď restart Klipperu.

## 7. Změř znovu

Kalibraci zopakuj.

Cílem není honit setiny milimetru za každou cenu. Důležité je, aby měření bylo opakovatelné a extruder během testu neprokluzoval.

Pokud dostáváš pokaždé výrazně jiný výsledek, nejdřív hledej mechanický problém:

- přítlak podávacího kolečka,
- prokluz filamentu,
- ucpanou trysku,
- příliš nízkou teplotu,
- problém s převody extruderu.

## 8. Směr extruderu – co znamená !

U pinů v Klipperu můžeš narazit například na:

```ini
dir_pin: !PB4
```

Znak:

```text
!
```

před pinem znamená **invertování logické úrovně pinu**.

U `dir_pin` se tím prakticky obrátí směr otáčení motoru.

Pokud extruder při příkazu k extruzi tahá filament ven místo dovnitř, jednou z možností je změnit:

```ini
dir_pin: !PB4
```

na:

```ini
dir_pin: PB4
```

nebo naopak.

> [!CAUTION]
> Neměň současně směr v konfiguraci a zapojení motoru. Udělej jednu změnu a znovu ověř chování, jinak si snadno vytvoříš další problém.

## 9. Co znamená ^ před pinem

V konfiguraci uvidíš také například:

```ini
diag_pin: ^PC0
```

Znak:

```text
^
```

zapíná interní **pull-up rezistor** daného vstupu mikrokontroléru.

Velmi zjednodušeně: vstup díky tomu nezůstává elektricky „viset ve vzduchu“, když ho připojené zařízení aktivně nestahuje do opačného stavu.

Proto `^` není dekorace a nemaž ho z konfigurace jen proto, že samotný název pinu `PC0` vypadá správně.

## 10. ! a ^ lze potkat i společně

Klipper dovoluje u některých vstupních pinů použít modifikátory společně.

Například:

```ini
click_pin: ^!EXP1_2
```

Zjednodušeně:

- `^` = zapnout pull-up,
- `!` = invertovat logiku,
- `EXP1_2` = samotný pin/alias.

Tyto znaky tedy popisují **jak má Klipper s pinem elektricky/logicky pracovat**, nejsou součástí jeho fyzického názvu.

> [!NOTE]
> V Klipper configu používáme pro komentáře `#`. Zápis `//`, známý například z C/C++ a konfigurace/zdrojáků Marlinu, sem nepatří.

## Co máš mít po této kapitole hotové

- víš, zda extruder používá převod a jaký,
- extruder se otáčí správným směrem,
- rozumíš základnímu významu `!` a `^` u pinů,
- změřil jsi skutečný posun filamentu,
- vypočítal jsi vlastní `rotation_distance`,
- opakovaný test přibližně odpovídá požadované délce.

Další krok:

➡️ **06 – Z-offset a první správná výška trysky**
