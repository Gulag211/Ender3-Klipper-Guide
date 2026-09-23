# 11 – PRINT_START, PRINT_END a komunikace se slicerem

Tiskárna už umí jednotlivé kroky samostatně. Teď je spojíme tak, aby slicer předal Klipperu požadované teploty a tiskárna si bezpečně provedla přípravu sama.

Cílem je mít logiku startu a konce tisku hlavně v Klipperu, ne pokaždé jinou v každém profilu sliceru.

## 1. Proč používat makra

Místo dlouhého start G-code ve sliceru můžeme vytvořit makro:

```text
PRINT_START
```

Slicer mu pouze předá potřebné hodnoty, například teplotu bedu a hotendu.

Výhoda je jednoduchá:

Když později změníš homing, mesh nebo jinou část přípravy, upravíš jedno makro v Klipperu místo všech profilů ve sliceru.

Stejný princip použijeme na konci:

```text
PRINT_END
```

## 2. Základní PRINT_START v tomto projektu

V našem `printer.cfg` používáme jednoduchou variantu bez závislosti na KAMP:

```ini
[gcode_macro PRINT_START]
gcode:
    {% set BED_TEMP = params.BED|default(60)|float %}
    {% set EXTRUDER_TEMP = params.EXTRUDER|default(200)|float %}

    M140 S{BED_TEMP}
    M104 S150
    M190 S{BED_TEMP}
    M109 S150

    G90
    M83
    G28

    BED_MESH_CLEAR
    BED_MESH_CALIBRATE

    M104 S{EXTRUDER_TEMP}
    TEMPERATURE_WAIT SENSOR=extruder MINIMUM={EXTRUDER_TEMP}

    G92 E0
```

Teď si ho rozebereme.

## 3. Parametry BED a EXTRUDER

První dva řádky:

```ini
{% set BED_TEMP = params.BED|default(60)|float %}
{% set EXTRUDER_TEMP = params.EXTRUDER|default(200)|float %}
```

vezmou hodnoty předané slicerem.

Pokud slicer parametr nepředá, použijí se výchozí hodnoty:

- bed 60 °C,
- hotend 200 °C.

Výchozí hodnoty jsou pojistka, ne náhrada správně nastaveného sliceru.

## 4. M140 a M190 nejsou totéž

```text
M140 S60
```

nastaví cílovou teplotu bedu, ale program pokračuje dál.

```text
M190 S60
```

naopak čeká, dokud bed nedosáhne požadované teploty.

Stejný princip existuje u hotendu:

```text
M104
```

nastaví teplotu bez čekání.

```text
M109
```

nastaví teplotu a čeká.

Rozdíl mezi „nastav teplotu“ a „nastav a čekej“ je při návrhu startovacího makra velmi důležitý.

## 5. Proč hotend nejdřív jen na 150 °C

Na začátku používáme:

```text
M104 S150
```

a později:

```text
M109 S150
```

Hotend tedy během přípravy držíme na 150 °C místo okamžitého nahřátí například na 200–230 °C.

Důvod je praktický:

Při homingu a měření podložky nechceme, aby z plně nahřáté trysky dlouho vytékal filament a dělal kapku na trysce nebo na bedu.

Teprve po změření podložky nastavíme skutečnou tiskovou teplotu.

## 6. Proč nejdřív zahříváme bed

Máme:

```text
M140 S{BED_TEMP}
M104 S150
M190 S{BED_TEMP}
M109 S150
```

Bed se tedy začne zahřívat a hotend současně míří na bezpečnou přípravnou teplotu.

Potom počkáme na bed.

Je to užitečné i proto, že zahřátá podložka může mít trochu jiný tvar než studená. Mesh tedy měříme ve stavu bližším samotnému tisku.

## 7. G90 a M83

Před pohybem používáme:

```text
G90
M83
```

`G90` nastaví absolutní souřadnice pohybu os.

Například:

```text
G1 X100
```

potom znamená „jeď na X=100“, nikoliv „přidej dalších 100 mm“.

`M83` nastaví **relativní extruzi**.

Například:

```text
G1 E5
```

potom znamená „posuň extruder o dalších 5 mm“.

Je důležité rozlišovat režim souřadnic os a režim extruderu.

## 8. Homing a mesh

Pak následuje:

```text
G28
BED_MESH_CLEAR
BED_MESH_CALIBRATE
```

Nejdřív zahomujeme tiskárnu.

Potom odstraníme případný starý mesh a vytvoříme nový.

Tohle je základní varianta, která funguje bez KAMP.

Až máš spolehlivě funkční KAMP, můžeš tuto část nahradit adaptivním postupem.

## 9. Finální teplota hotendu

Po změření bedu nastavíme:

```text
M104 S{EXTRUDER_TEMP}
```

a čekáme:

```text
TEMPERATURE_WAIT SENSOR=extruder MINIMUM={EXTRUDER_TEMP}
```

Tím se hotend dostane na skutečnou teplotu požadovanou slicerem.

Teprve potom má začít extruze první vrstvy.

## 10. Proč je G92 E0

Na konci přípravy máme:

```text
G92 E0
```

Tím řekneme Klipperu, že aktuální pozici extruderu má považovat za E=0.

Je to užitečný čistý výchozí bod před začátkem tiskového G-code.

## 11. Co má posílat slicer

Start G-code ve sliceru chceme co nejjednodušší.

Princip je:

```text
PRINT_START BED=<teplota bedu ze sliceru> EXTRUDER=<teplota hotendu ze sliceru>
```

Konkrétní názvy proměnných se mezi OrcaSlicerem, PrusaSlicerem a dalšími slicery mohou lišit.

Proto nekopíruj syntaxi proměnné z jiného sliceru bez kontroly.

Výsledkem ale musí být například něco ve stylu:

```text
PRINT_START BED=60 EXTRUDER=210
```

Klipper pak dostane dvě obyčejná čísla a zbytek přípravy provede makro.

## 12. Nedělej stejnou věc dvakrát

Pokud `PRINT_START` provádí:

```text
G28
```

nemusíš mít další `G28` ve sliceru.

Pokud KAMP provádí purge line, nepřidávej další purge line ve sliceru.

Pokud Klipper čeká na finální teplotu hotendu, dávej pozor, aby slicer před makrem nevložil vlastní dlouhé čekání na stejnou teplotu.

Start tisku má mít jednoho jasného „šéfa“.

V našem případě je to Klipper makro.

## 13. PRINT_END

Základní konec tisku v projektu vypadá přibližně takto:

```ini
[gcode_macro PRINT_END]
gcode:
    M400
    G92 E0
    G1 E-1 F1800

    TURN_OFF_HEATERS
    M107

    {% if printer.toolhead.position.z < (printer.toolhead.axis_maximum.z - 10) %}
        G91
        G1 Z10 F1200
        G90
    {% endif %}

    G1 X0 Y200 F6000
    M84
    BED_MESH_CLEAR
```

## 14. M400 – počkej na dokončení pohybů

```text
M400
```

počká, až tiskárna dokončí naplánované pohyby.

Teprve potom začneme provádět konečné operace.

## 15. Malá retrakce

```text
G92 E0
G1 E-1 F1800
```

vynuluje pozici extruderu a provede malou retrakci 1 mm.

Cílem je po dokončení tisku trochu uvolnit tlak v trysce.

## 16. Vypnutí topení a ventilátoru

```text
TURN_OFF_HEATERS
M107
```

vypne topení a ventilátor ofuku výtisku.

Ventilátor hotendu řízený přes `[heater_fan]` se nemusí vypínat ručně – Klipper ho může nechat běžet, dokud hotend nevychladne pod nastavenou teplotu.

To je přesně to, co chceme.

## 17. Proč nezvedáme Z slepě o 10 mm

Na internetu často uvidíš:

```text
G91
G1 Z10
```

To funguje, dokud tisk nekončí velmi blízko maximální výšky tiskárny.

Pokud je ale Z například jen 2 mm pod maximem, požadavek na dalších 10 mm by se pokusil překročit povolený rozsah.

Proto máme podmínku:

```ini
{% if printer.toolhead.position.z < (printer.toolhead.axis_maximum.z - 10) %}
```

Zvednutí provedeme pouze tehdy, když na něj máme prostor.

## 18. Zaparkování hlavy

Potom:

```text
G1 X0 Y200 F6000
```

odjede s hlavou do zvolené parkovací polohy.

I tato souřadnice musí odpovídat skutečnému rozsahu konkrétní tiskárny.

Pokud máš jiné limity os, uprav ji.

## 19. M84

```text
M84
```

vypne krokové motory.

Po tomto příkazu už tiskárna nedrží mechaniku v přesně známé poloze.

Před dalším tiskem proto stejně provádíme nový homing.

## 20. BED_MESH_CLEAR

Na konci:

```text
BED_MESH_CLEAR
```

odstraní aktivní mesh.

Další tisk si vytvoří nový.

Tím je jasné, že nový tisk nezačne omylem s mapou z předchozího stavu podložky.

## 21. PRINT_END ve sliceru

End G-code sliceru pak může být velmi jednoduchý:

```text
PRINT_END
```

Opět nechceme mít polovinu ukončovací logiky ve sliceru a druhou polovinu v Klipperu.

## 22. Makra testuj po částech

Po vytvoření nového `PRINT_START` není nejlepší první test pustit dvacetihodinový tisk a odejít.

Nejdřív ověř:

- předání teplot,
- zahřívání,
- homing,
- mesh,
- finální nahřátí,
- začátek extruze,
- ukončení tisku,
- bezpečný park.

Každý krok už znáš z předchozích kapitol.

Makro je pouze skládá dohromady.

## Co máš mít po této kapitole hotové

- rozumíš parametrům `BED` a `EXTRUDER`,
- chápeš rozdíl mezi `M104/M109` a `M140/M190`,
- víš, proč během přípravy držíme hotend na 150 °C,
- slicer předává teploty do `PRINT_START`,
- nemáš duplicitní homing, mesh ani purge,
- `PRINT_END` bezpečně vypne tisk a zaparkuje hlavu,
- hlavní logika startu a konce tisku je na jednom místě v Klipperu.

Další krok:

➡️ **12 – První testovací tisk a bezpečné zvyšování rychlosti**
