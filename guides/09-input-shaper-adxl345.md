# 09 – ADXL345 a Input Shaper

Teď se dostáváme k jedné z velkých výhod Klipperu – měření rezonancí tiskárny a jejich omezení pomocí **Input Shaperu**.

Tato část je už pokročilejší a pro základní zprovoznění tiskárny není povinná.

Tiskárna může normálně fungovat i bez ADXL345.

## 1. Co Input Shaper řeší

Při rychlých změnách směru se mechanika tiskárny rozkmitá.

Na výtisku se to může projevit například jako opakující se vlnky za ostrou hranou – často označované jako ringing nebo ghosting.

Input Shaper se snaží řízení pohybu upravit tak, aby byly známé rezonance tiskárny méně buzené.

Neopraví ale mechanický problém.

Nejdřív zkontroluj například:

- napnutí řemenů,
- vůle vedení a koleček,
- dotažení řemenic,
- pevnost rámu,
- uchycení toolheadu,
- uchycení podložky.

Pokud se na tiskárně něco mechanicky viklá, Input Shaper není náhrada za šroubovák.

## 2. Co je ADXL345

ADXL345 je tříosý akcelerometr.

Při kalibraci Klipper řízeně rozkmitá osu tiskárny a ADXL měří její skutečné vibrace.

Z naměřených dat potom Klipper může navrhnout:

- typ Input Shaperu,
- jeho frekvenci.

Tyto hodnoty jsou specifické pro konkrétní tiskárnu.

Proto nekopíruj hodnoty Input Shaperu z jiné tiskárny – ani když je to stejný model Enderu.

## 3. Bed slinger – X a Y nejsou stejná pohybující se část

U Enderu 3 máme klasickou konstrukci **bed slinger**.

To znamená:

- v ose **X** se pohybuje toolhead/hotend,
- v ose **Y** se pohybuje celá vyhřívaná podložka.

A právě proto chceme rezonance obou pohybujících se částí měřit samostatně.

### Měření osy X

Pro měření X upevni ADXL345 pevně na toolhead/hotend.

Ideálně co nejblíž reprezentativnímu středu pohybující se sestavy a tak, aby senzor skutečně kopíroval její vibrace.

### Měření osy Y

Potom ADXL345 přesuň na vyhřívanou podložku.

Pro Y měření ho upevni pokud možno přibližně **do středu bedu**.

Tím měříš přímo vibrace hmoty, která se při pohybu osy Y skutečně rozkmitává.

> [!IMPORTANT]
> U bed slingeru tedy nedělej obě měření pouze se senzorem na toolheadu. Pro X měříme toolhead a pro Y přesuneme senzor na bed.

## 4. Uchycení senzoru je důležité

ADXL musí být při měření **pevně spojený** s měřenou částí tiskárny.

Nevhodné je například:

- nechat senzor volně ležet,
- držet ho rukou,
- zavěsit ho pouze za kabel,
- použít velmi měkké uchycení, které samo pruží.

Takové uchycení může přidat vlastní pohyb a zkreslit měření.

Nejlepší je pevný držák odpovídající konkrétní tiskárně/toolheadu.

U bedu musí držák současně zajistit, že senzor při testu neuletí a kabel se nemůže zachytit do pohybující se mechaniky.

## 5. Pozor na kabel

Při resonance testu se osy rychle rozkmitávají.

Před spuštěním vždy zkontroluj:

- že má kabel ADXL dostatečnou délku,
- že se nemůže zachytit o rám,
- že netahá za senzor,
- že nemůže spadnout do ventilátoru nebo řemenu.

Kabel zároveň nesmí senzor mechanicky „držet“ tak silně, že ovlivní jeho pohyb.

## 6. ADXL přes Raspberry Pi jako druhé MCU

Jednou z možností je připojit ADXL345 k Raspberry Pi.

Aby s jeho GPIO mohl Klipper přímo pracovat, Raspberry Pi se nastaví jako další Klipper MCU.

Typická konfigurace potom může obsahovat například:

```ini
[mcu rpi]
serial: /tmp/klipper_host_mcu
```

a následně konfiguraci akcelerometru.

> [!IMPORTANT]
> Toto **není součást základního printer.cfg** v tomto projektu. Tiskárna má nejdřív spolehlivě fungovat bez ADXL a teprve potom přidáváme tuto volitelnou část.

Konkrétní konfiguraci a instalaci host MCU dáme do samostatného souboru v adresáři `optional/`.

## 7. Nejdřív ověř komunikaci se senzorem

Po správném zapojení a konfiguraci použij:

```text
ACCELEROMETER_QUERY
```

Klipper musí vrátit naměřené hodnoty akcelerometru.

Pokud dostaneš chybu komunikace, **nespouštěj resonance test**.

Nejdřív oprav zapojení nebo konfiguraci.

## 8. Jednoduchá kontrola, že senzor opravdu reaguje

Hodnoty z `ACCELEROMETER_QUERY` nejsou při změně orientace senzoru stále stejné.

Můžeš proto ještě před kalibrací ověřit, že Klipper dostává smysluplná data.

Neřešíme tím přesnou kalibraci – jen kontrolujeme, že senzor není mrtvý a komunikace funguje.

## 9. Měření rezonancí osy X

Pevně upevni ADXL na toolhead.

Zkontroluj kabel a volný pohyb celé osy.

Potom spusť:

```text
TEST_RESONANCES AXIS=X
```

Tiskárna začne osu řízeně rozkmitávat v různých frekvencích.

> [!CAUTION]
> Test může být hlučný a pohyb tiskárny může působit nezvykle agresivně. Během prvních měření zůstaň u tiskárny a buď připravený test zastavit, pokud se mechanicky děje něco špatného.

## 10. Přesuň ADXL na bed

Po dokončení X testu senzor sundej z toolheadu a **pevně ho připevni přibližně doprostřed vyhřívané podložky**.

Znovu zkontroluj kabel.

Teprve potom měř Y:

```text
TEST_RESONANCES AXIS=Y
```

Tím získáme data přímo z pohybujícího se bedu.

## 11. Automatická kalibrace Input Shaperu

Klipper umí provést automatickou kalibraci pomocí:

```text
SHAPER_CALIBRATE AXIS=X
```

a:

```text
SHAPER_CALIBRATE AXIS=Y
```

U bed slingeru při tom opět použij správné umístění senzoru:

- X → toolhead,
- Y → bed.

Klipper z naměřených rezonancí vyhodnotí vhodný shaper a frekvenci.

Po dokončení obou měření můžeš výsledky uložit:

```text
SAVE_CONFIG
```

## 12. Co se uloží

Výsledkem mohou být hodnoty podobného typu:

```ini
[input_shaper]
shaper_type_x: ...
shaper_freq_x: ...
shaper_type_y: ...
shaper_freq_y: ...
```

Tečky zde nejsou hodnoty k opsání.

Klipper musí určit hodnoty **pro tvoji konkrétní tiskárnu**.

Dvě na pohled stejné tiskárny mohou mít jiné výsledky kvůli rozdílům v:

- napnutí řemenů,
- hmotnosti toolheadu,
- držáku hotendu,
- bedu,
- kolečkách nebo lineárním vedení,
- rámu,
- dalších úpravách.

## 13. Po změně mechaniky měř znovu

Input Shaper popisuje mechanické chování tiskárny v okamžiku měření.

Pokud výrazně změníš například:

- toolhead,
- extruder,
- hotend,
- ventilátory,
- držáky,
- bed,
- vedení,
- napnutí řemenů,

je rozumné měření zopakovat.

Staré hodnoty nemusí nový stav tiskárny dobře reprezentovat.

## 14. Vyšší rychlost není automaticky lepší

Input Shaper může umožnit tisknout rychleji při menším ringingu, ale neodstraňuje ostatní fyzikální limity tiskárny.

Pořád tě může omezovat například:

- maximální průtok hotendu,
- extruder,
- chlazení výtisku,
- motory,
- mechanika,
- kvalita konkrétního materiálu.

Kalibrace Input Shaperu tedy není povolení nastavit náhodně 500 mm/s a obrovskou akceleraci.

## Co máš mít po této kapitole hotové

- rozumíš, co Input Shaper kompenzuje,
- ADXL je pevně uchycený,
- `ACCELEROMETER_QUERY` funguje,
- X jsi změřil se senzorem na toolheadu,
- Y jsi změřil se senzorem na bedu,
- Klipper určil vlastní hodnoty Input Shaperu,
- výsledky jsou uložené pomocí `SAVE_CONFIG`,
- víš, že po významné změně mechaniky je vhodné měření zopakovat.

Další krok:

➡️ **10 – KAMP: adaptivní Bed Mesh a purge line**
