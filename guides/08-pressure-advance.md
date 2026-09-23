# 08 – Pressure Advance

Tiskárna už umí bezpečně homovat, zná Z-offset, má změřenou podložku a správně nastavený extruder.

Teď můžeme začít ladit kvalitu tisku při změnách rychlosti.

K tomu v Klipperu slouží **Pressure Advance (PA)**.

## 1. Proč Pressure Advance existuje

Filament, hotend ani celý extruzní systém nereagují dokonale okamžitě.

Když tiskárna zrychluje, tlak roztaveného plastu v trysce se musí nejdřív vytvořit. Když naopak před rohem zpomaluje, tlak v hotendu nezmizí okamžitě.

Bez kompenzace se to může projevit například:

- příliš tlustými rohy,
- bouličkami při změně rychlosti,
- nedostatkem materiálu po zrychlení,
- méně přesnou šířkou čáry.

Pressure Advance se snaží tyto změny tlaku předvídat a upravit pohyb extruderu tak, aby průtok lépe odpovídal okamžitému pohybu tiskové hlavy.

## 2. PA není univerzální číslo

V soukromém stroji, ze kterého tento projekt původně vychází, byla používána konkrétní hodnota Pressure Advance.

Do veřejného základního configu jsme ji **záměrně nepřenesli**.

PA závisí například na:

- extruderu,
- délce a pružnosti filamentové cesty,
- materiálu,
- teplotě,
- trysce,
- rychlosti tisku.

Proto hodnotu z internetu nebo z jiné tiskárny nepovažuj za svoji kalibraci.

## 3. Nejdřív musí fungovat základ tiskárny

Pressure Advance neladíme jako řešení jiného problému.

Před kalibrací musí být rozumně nastavené:

- `rotation_distance`,
- teploty,
- průtok materiálu,
- mechanika extruderu,
- řemeny a pohyb os.

Pokud extruder prokluzuje nebo má tiskárna mechanickou vůli, PA to neopraví.

## 4. Dočasné nastavení PA

Hodnotu můžeš během testování změnit z konzole bez přepisování configu:

```text
SET_PRESSURE_ADVANCE ADVANCE=0.02
```

Tím nastavíš PA pro aktuální běh Klipperu.

Pro návrat na nulovou kompenzaci:

```text
SET_PRESSURE_ADVANCE ADVANCE=0
```

To je při kalibraci praktické, protože nemusíš při každém pokusu upravovat `printer.cfg`.

## 5. Kalibrace pomocí testovacího modelu

Pro přesnou kalibraci použij test Pressure Advance určený pro Klipper.

Smyslem testu je během jednoho tisku postupně měnit PA a sledovat, při které hodnotě vypadají změny směru a rohy nejlépe.

Klipper umožňuje použít příkaz:

```text
TUNING_TOWER COMMAND=SET_PRESSURE_ADVANCE PARAMETER=ADVANCE START=0 FACTOR=0.005
```

Tím se hodnota Pressure Advance postupně mění podle výšky modelu.

> [!IMPORTANT]
> Parametry testu nejsou univerzální pro každou kombinaci extruderu a testovacího modelu. Vždy používej postup odpovídající konkrétnímu testu, který tiskneš.

## 6. Co při testu sleduj

Nehledej pouze „nejhezčí celý model“.

Soustřeď se hlavně na místa, kde tiskárna mění směr a rychlost.

Příliš nízké PA se může projevit přebytečným materiálem v rozích.

Příliš vysoké PA může naopak způsobit nedostatek materiálu v okolí změn rychlosti nebo jiné viditelné artefakty.

Hledáme oblast, kde jsou přechody nejrovnoměrnější.

## 7. Výpočet výsledné hodnoty

U klasického `TUNING_TOWER` testu se výsledná hodnota odvozuje od výšky, ve které model vypadá nejlépe.

Při příkladu:

```text
START=0
FACTOR=0.005
```

a nejlepší oblasti například ve výšce:

```text
8 mm
```

dostaneme:

```text
0 + 8 × 0.005 = 0.040
```

tedy:

```ini
pressure_advance: 0.040
```

Toto je pouze ukázka výpočtu, **ne doporučená hodnota pro tvoji tiskárnu**.

## 8. Uložení do printer.cfg

Až máš vlastní ověřenou hodnotu, můžeš ji přidat do sekce extruderu:

```ini
[extruder]
pressure_advance: 0.040
```

Číslo `0.040` zde opět slouží pouze jako příklad.

Potom proveď:

```text
RESTART
```

## 9. pressure_advance_smooth_time

Můžeš narazit také na:

```ini
pressure_advance_smooth_time
```

To je další parametr související s tím, jak Klipper Pressure Advance aplikuje.

Pro první základní nastavení ho **není potřeba bezdůvodně měnit**.

Nejdřív správně zkalibruj samotný `pressure_advance`. Další parametr nemá smysl ladit jen proto, že jsi někde našel cizí hodnotu.

## 10. Direct Drive a Bowden

Obecně může Bowden systém kvůli dlouhé pružné cestě filamentu potřebovat výraznější kompenzaci než Direct Drive.

To ale není důvod opsat „typickou Bowden hodnotu“ nebo „typickou Direct Drive hodnotu“.

Použij vlastní měření.

## 11. Materiál může hodnotu změnit

PLA, PETG, TPU a další materiály se nechovají stejně.

U pružných materiálů může být rozdíl výrazný.

Pokud chceš tiskárnu později ladit opravdu přesně, můžeš mít Pressure Advance nastavený podle konkrétního filamentu nebo profilu.

Pro první zprovoznění ale není potřeba vytvořit dvacet kalibrací najednou.

Začni materiálem, který používáš nejčastěji.

## 12. PA není náhrada za správný průtok

Pressure Advance řeší dynamické změny tlaku při změnách rychlosti.

Neřeší celkové množství vytlačovaného materiálu.

Pokud je celý tisk přeextrudovaný nebo podextrudovaný, řeš nejdřív:

- mechaniku extruderu,
- `rotation_distance`,
- teplotu,
- průtok / flow konkrétního materiálu.

Teprve potom má smysl jemně ladit PA.

## Co máš mít po této kapitole hotové

- rozumíš, proč Pressure Advance existuje,
- víš, že cizí PA hodnotu nemáš slepě kopírovat,
- umíš PA dočasně změnit přes `SET_PRESSURE_ADVANCE`,
- chápeš princip `TUNING_TOWER`,
- umíš z testu určit vlastní hodnotu,
- výsledný `pressure_advance` máš uložený v konfiguraci.

Další krok:

➡️ **09 – Input Shaper a ADXL345**
