# 04 – Kontrola topení a PID tuning

Po bezpečném ověření homingu můžeme začít pracovat s hotendem a vyhřívanou podložkou.

> [!CAUTION]
> První test topení dělej vždy přímo u tiskárny. Nejdřív ověř správné teplotní senzory a až potom zapínej výkon do topných těles.

## 1. Nejdřív teploty za studena

Po zapnutí tiskárny nech hotend i bed několik minut bez topení.

V Mainsailu zkontroluj jejich teploty. Při pokojové teplotě by měly ukazovat přibližně teplotu okolí.

Nemusí být stejné na desetinu stupně, ale například 20–30 °C v běžné místnosti dává smysl. Hodnota hluboko pod nulou, extrémně vysoká hodnota nebo chyba ADC znamená problém, který musíš vyřešit před zapnutím topení.

V našem příkladu jsou použity:

```ini
[extruder]
sensor_type: EPCOS 100K B57560G104F

[heater_bed]
sensor_type: ATC Semitec 104GT-2
```

**Ověř, že tyto typy odpovídají skutečným termistorům na tvé tiskárně.**

## 2. Ověř, že se zahřívá správná část

Než nastavíš 200 °C, udělej krátký test s nízkou cílovou teplotou.

### Hotend

V Mainsailu nastav hotend například na:

```text
50 °C
```

Sleduj graf.

Teplota hotendu musí začít plynule růst a údaj bedu by měl zůstat přibližně na původní teplotě.

Potom hotend vypni.

### Bed

Stejně otestuj podložku například na:

```text
40 °C
```

Teď musí růst teplota bedu.

> [!CAUTION]
> Pokud zapneš hotend a začne růst údaj bedu – nebo obráceně – nepokračuj. Zkontroluj zapojení a konfiguraci.

## 3. Zkontroluj ventilátor hotendu

V našem configu je:

```ini
[heater_fan hotend_fan]
pin: PC6
heater: extruder
heater_temp: 50.0
```

Po překročení nastavené teploty hotendu se musí spustit ventilátor chlazení hotendu.

Nepleť si ho s ventilátorem ofuku výtisku:

```ini
[fan]
pin: PC7
```

Ten se ovládá samostatně.

## 4. Co je PID tuning

Klipper musí umět řídit výkon topení tak, aby se teplota rychle dostala na požadovanou hodnotu, ale zbytečně ji nepřestřelovala a následně nekolísala.

K tomu slouží PID regulace.

PID hodnoty jsou závislé na konkrétní sestavě – například heateru, hotendu, termistoru, silikonové ponožce, napájení a chlazení.

Proto **nekopíruj PID hodnoty z mojí ani jiné tiskárny**.

## 5. PID tuning hotendu

Pro PLA můžeš jako praktický kalibrační bod použít například 200 °C:

```text
PID_CALIBRATE HEATER=extruder TARGET=200
```

Během kalibrace bude Klipper hotend opakovaně zahřívat a regulovat.

Do tiskárny během testu nezasahuj a sleduj průběh.

Po úspěšném dokončení spusť:

```text
SAVE_CONFIG
```

Klipper vypočtené hodnoty uloží do automaticky generované části konfigurace.

> [!NOTE]
> Pokud běžně tiskneš hlavně při výrazně jiné teplotě, může dávat smysl kalibrovat PID blíže teplotě, kterou skutečně používáš.

## 6. PID tuning bedu

Pro běžné PLA můžeš bed kalibrovat například na 60 °C:

```text
PID_CALIBRATE HEATER=heater_bed TARGET=60
```

Po dokončení:

```text
SAVE_CONFIG
```

Bed se zahřívá pomaleji než hotend, takže tato kalibrace může trvat déle.

## 7. SAVE_CONFIG není kouzelné tlačítko

Příkaz:

```text
SAVE_CONFIG
```

uloží hodnoty, které Klipper podporuje tímto způsobem, na konec `printer.cfg` do automaticky generované části.

Typicky uvidíš něco jako:

```text
#*# <---------------------- SAVE_CONFIG ---------------------->
#*# DO NOT EDIT THIS BLOCK OR BELOW.
```

Tuto část **ručně neupravuj**.

Klipper ji spravuje sám.

## 8. Ověř výsledek

Po restartu nastav například:

Hotend:

```text
200 °C
```

Bed:

```text
60 °C
```

Sleduj graf v Mainsailu.

Teplota by se měla dostat k cíli a regulace by měla být rozumně stabilní.

PID tuning ale neopraví:

- špatný termistor,
- povolený heater cartridge,
- špatný kontakt kabelu,
- nedostatečný zdroj,
- mechanický problém hotendu.

Pokud je chování teploty podezřelé, nejdřív hledej skutečnou příčinu.

## Co máš mít po této kapitole hotové

- hotend a bed ukazují za studena smysluplné teploty,
- při zapnutí hotendu roste správný senzor,
- při zapnutí bedu roste správný senzor,
- ventilátor hotendu funguje,
- hotend má vlastní PID kalibraci,
- bed má vlastní PID kalibraci,
- hodnoty jsou uložené pomocí `SAVE_CONFIG`.

Další krok:

➡️ **05 – Kalibrace extruderu / rotation_distance**
