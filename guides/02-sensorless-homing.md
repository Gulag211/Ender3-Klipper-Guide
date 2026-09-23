# 02 – Sensorless homing X/Y

Tento návod navazuje na **01 – První spuštění a bezpečná kontrola**.

Na této sestavě nejsou pro osy X a Y použité klasické mechanické endstopy. Klipper využívá drivery **TMC2209** a jejich funkci StallGuard. Driver při nárazu vozíku do mechanického konce osy rozpozná zvýšené zatížení motoru a Klipper tento okamžik použije jako virtuální endstop.

> [!CAUTION]
> Sensorless homing nastavuj vždy u tiskárny. Měj ruku připravenou na vypínači. Špatně nastavená citlivost může způsobit tvrdý náraz vozíku do rámu nebo naopak falešné ukončení homingu.

## 1. Jak je sensorless homing zapojený v configu

Pro osu X používáme:

```ini
[stepper_x]
endstop_pin: tmc2209_stepper_x:virtual_endstop
homing_retract_dist: 0

[tmc2209 stepper_x]
diag_pin: ^PC0
driver_sgthrs: 100
```

Pro Y:

```ini
[stepper_y]
endstop_pin: tmc2209_stepper_y:virtual_endstop
homing_retract_dist: 0

[tmc2209 stepper_y]
diag_pin: ^PC1
driver_sgthrs: 100
```

Hodnota `100` je pouze **výchozí příklad z mojí tiskárny**. Není to univerzální hodnota pro každý Ender 3.

## 2. Co znamená driver_sgthrs

U TMC2209 je `driver_sgthrs` citlivost StallGuardu v rozsahu:

```text
0 až 255
```

Obecně:

- vyšší hodnota = vyšší citlivost,
- nižší hodnota = nižší citlivost.

Příliš vysoká citlivost může způsobit falešné sepnutí během pohybu.

Příliš nízká citlivost může způsobit, že driver náraz na konec osy nerozpozná dostatečně rychle.

Proto hodnotu **kalibruj na své tiskárně**.

## 3. Před prvním pokusem

Nejdřív znovu ověř, že X opravdu ovládá X a Y opravdu ovládá Y:

```text
STEPPER_BUZZ STEPPER=stepper_x
STEPPER_BUZZ STEPPER=stepper_y
```

Zkontroluj také, že se vozík může mechanicky volně pohybovat po celé ose.

Řemen nesmí být extrémně volný ani přetažený a kolečka/lineární vedení se nesmí zasekávat.

Sensorless homing totiž nerozezná rozdíl mezi „jsem na konci osy“ a „něco mě mechanicky zablokovalo“.

## 4. Směr homingu

V našem základním configu je:

```ini
position_endstop: 0
```

Tím počítáme s homingem směrem k minimálnímu konci osy.

Před testem se podívej, kde se tisková hlava a bed právě nachází. Pokud jsou už natlačené přímo na mechanickém dorazu, ručně je při vypnutých motorech posuň několik centimetrů od něj.

## 5. Začni pouze osou X

Do konzole neposílej `G28` bez parametrů.

Nejdřív testujeme pouze X:

```text
G28 X
```

Sleduj pohyb.

Správné chování:

1. vozík jede směrem k dorazu X,
2. dotkne se mechanického konce,
3. StallGuard náraz detekuje,
4. homing skončí.

Pokud vozík jede **opačným směrem**, tiskárnu zastav a neopakuj test. Je potřeba zkontrolovat směr motoru a konfiguraci osy.

Pokud motor dál tlačí do rámu, okamžitě test ukonči a uprav citlivost.

Pokud homing skončí ještě před dosažením dorazu, citlivost je pravděpodobně příliš vysoká nebo je na ose neobvykle velký mechanický odpor.

## 6. Ladění citlivosti X

Pro první pokusy není nutné pokaždé upravovat soubor a restartovat Klipper.

Citlivost můžeš dočasně změnit příkazem:

```text
SET_TMC_FIELD STEPPER=stepper_x FIELD=SGTHRS VALUE=100
```

Změň `100` na testovanou hodnotu.

Například:

```text
SET_TMC_FIELD STEPPER=stepper_x FIELD=SGTHRS VALUE=80
```

Potom znovu:

```text
G28 X
```

Jakmile najdeš spolehlivou hodnotu, zapiš ji do:

```ini
[tmc2209 stepper_x]
driver_sgthrs: TVOJE_HODNOTA
```

a proveď `SAVE & RESTART`.

> [!TIP]
> Nehledej hodnotu, která fungovala jednou. Vyzkoušej homing několikrát z různých míst osy. Cílem je spolehlivé chování bez zbytečně tvrdých nárazů a bez falešného sepnutí.

## 7. Stejným způsobem nastav Y

Až X funguje spolehlivě, pokračuj osou Y:

```text
G28 Y
```

Dočasná změna citlivosti:

```text
SET_TMC_FIELD STEPPER=stepper_y FIELD=SGTHRS VALUE=100
```

Výslednou hodnotu potom zapiš do:

```ini
[tmc2209 stepper_y]
driver_sgthrs: TVOJE_HODNOTA
```

X a Y **nemusí mít stejnou hodnotu**. Osy mají jinou hmotnost, mechaniku a zatížení.

## 8. Proč máme homing_retract_dist: 0

U klasického endstopu může tiskárna po prvním sepnutí kousek odjet a endstop znovu najít.

U sensorless homingu používáme:

```ini
homing_retract_dist: 0
```

Tím tento klasický druhý dotyk vypneme.

## 9. Ověř X a Y několikrát

Samostatně několikrát vyzkoušej:

```text
G28 X
```

a:

```text
G28 Y
```

Teprve když jsou **obě osy spolehlivé**, můžeme pokračovat k bezpečnému homingu Z přes BLTouch.

> [!IMPORTANT]
> Ještě není důvod testovat kompletní `G28`, pokud jsme neověřili Z-home a BLTouch nad podložkou.

## Co máš mít po této kapitole hotové

- X jede při homingu správným směrem.
- Y jede při homingu správným směrem.
- X spolehlivě detekuje mechanický konec.
- Y spolehlivě detekuje mechanický konec.
- Znáš vlastní `driver_sgthrs` pro X.
- Znáš vlastní `driver_sgthrs` pro Y.
- Homing není zbytečně agresivní ani se nespouští falešně.

Další krok:

➡️ **03 – BLTouch a první bezpečný Z-home**
