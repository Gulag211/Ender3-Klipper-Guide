# 03 – BLTouch a první bezpečný Z-home

Tento návod navazuje na nastavení a otestování sensorless homingu X/Y.

Teď ověříme BLTouch a teprve potom dovolíme trysce přiblížit se k podložce.

> [!CAUTION]
> **Nedělej první Z-home rovnou proti podložce.** Nejdřív otestujeme sondu vysoko nad bedem a její pin při homingu ručně sepneme. Pokud je něco špatně zapojené nebo nastavené, získáš čas tiskárnu zastavit dřív, než tryska narazí do podložky.

## 1. BLTouch není vždy stejný BLTouch

Existuje několik hardwarových revizí originálního BLTouch a také CR Touch a různé kompatibilní sondy/klony.

Podle konkrétní sondy, její revize a způsobu zapojení může být potřeba trochu jiné nastavení. Proto **nekopíruj celý `[bltouch]` blok jen proto, že někomu jinému funguje na Enderu 3**.

V našem základním configu používáme:

```ini
[bltouch]
sensor_pin: ^PC2
control_pin: PA1
x_offset: 30
y_offset: 2
pin_move_time: 0.1
speed: 25
probe_with_touch_mode: True
pin_up_touch_mode_reports_triggered: False
stow_on_each_sample: False
```

Parametry `probe_with_touch_mode`, `pin_up_touch_mode_reports_triggered` a `stow_on_each_sample` nemusí být vhodné pro každou sondu.

## 2. Nejdřív správně namontuj sondu

Než začneš řešit config, ověř mechanickou výšku BLTouch.

Při **zasunutém pinu** má být špička pinu bezpečně výš než špička trysky, aby při tisku nezachytávala o výtisk. Oficiální dokumentace Klipperu uvádí jako první kontrolu přibližně **2 mm nad tryskou**.

Při **vysunutém pinu** naopak musí být pin níž než tryska, aby při Z-home sepnul dříve, než tryska narazí do podložky.

> [!IMPORTANT]
> Pokud je sonda mechanicky příliš vysoko, tryska může narazit do bedu dříve, než BLTouch sepne. Pokud je příliš nízko, zasunutý pin může zachytávat o výtisk.

## 3. Nastav správný X/Y offset sondy vůči trysce

`x_offset` a `y_offset` říkají Klipperu, **kde se sonda fyzicky nachází vůči trysce**. Změříš je pravítkem nebo lépe posuvným měřítkem od středu trysky ke středu pinu sondy.

Při běžné orientaci tiskárny, když stojíš před ní:

```text
                 ZADNÍ ČÁST TISKÁRNY
                         +Y
                          ↑

             -X   ←   TRYSKA   →   +X

                          ↓
                         -Y
                 PŘEDNÍ ČÁST TISKÁRNY
```

Tedy:

- sonda **vpravo od trysky** → kladné `x_offset`,
- sonda **vlevo od trysky** → záporné `x_offset`,
- sonda **za tryskou** → kladné `y_offset`,
- sonda **před tryskou** → záporné `y_offset`.

Příklad:

```ini
[bltouch]
x_offset: 30
y_offset: 2
```

znamená, že sonda je přibližně **30 mm vpravo a 2 mm za tryskou**.

Tohle je stejný geometrický princip, který Marlin popisuje jako `NOZZLE_TO_PROBE_OFFSET`. Hodnoty ale vždy změř na své vlastní tiskárně a zapiš je syntaxí Klipperu.

### Přesnější metoda podle Klipperu

Pokud nechceš spoléhat jen na měření posuvkou, můžeš offset ověřit přímo na bedu:

1. Zahomuj tiskárnu a přesuň hlavu přibližně doprostřed.
2. Dej na bed kousek papírové pásky.
3. Proveď `PROBE` a označ místo přímo pod středem pinu sondy.
4. Pomocí `GET_POSITION` si poznamenej XY polohu.
5. Přesuň hlavu tak, aby byla **tryska přesně nad stejnou značkou**.
6. Znovu použij `GET_POSITION`.
7. Rozdíl poloh použij jako X/Y offset podle postupu v oficiální dokumentaci Klipperu.

Po změně `x_offset` nebo `y_offset` znovu zkontroluj také `safe_z_home` a hranice `bed_mesh`, protože Klipper musí při měření udržet **sondu**, ne pouze trysku, nad podložkou.

> [!NOTE]
> X/Y offset není totéž co Z-offset. X/Y popisuje polohu sondy vedle trysky. Z-offset se kalibruje později pomocí `PROBE_CALIBRATE`.

## 4. Otestuj samotný pin

Do konzole Mainsailu napiš:

```text
BLTOUCH_DEBUG COMMAND=pin_down
```

Pin sondy se musí vysunout. Potom:

```text
BLTOUCH_DEBUG COMMAND=pin_up
```

Pin se musí zasunout. Zopakuj test několikrát. Pokud pin nereaguje správně, **nepokračuj k homingu Z**.

## 5. Ověř, že Klipper pozná sepnutí sondy

Vysuň pin:

```text
BLTOUCH_DEBUG COMMAND=pin_down
```

Potom:

```text
QUERY_PROBE
```

Klipper musí hlásit stav odpovídající nesepnuté sondě. Teď pin BLTouch **velmi jemně** zatlač nahoru a znovu použij `QUERY_PROBE`. Stav se musí změnit.

Nakonec:

```text
BLTOUCH_DEBUG COMMAND=pin_up
```

> [!IMPORTANT]
> Samotné vysouvání a zasouvání pinu ještě nestačí. Musíme ověřit i to, že Klipper elektricky pozná jeho sepnutí.

## 6. Pro první Z-home používáme pouze 2 mm/s

V základním `printer.cfg` je záměrně:

```ini
[stepper_z]
homing_speed: 2
```

Tohle **není doporučená finální rychlost**. Je to bezpečnostní nastavení pro první testy. Až bude BLTouch spolehlivě fungovat, můžeš rychlost postupně zvýšit například do rozsahu 5–16 mm/s.

## 7. Připrav tiskárnu pro test ve vzduchu

Nejdřív musí být správně ověřený homing X a Y:

```text
G28 X
G28 Y
```

Potom dostaň osu Z přibližně do poloviny výšky, aby mezi tryskou a podložkou zůstala velká bezpečnostní vzdálenost.

Pokud Z ještě není zahomované, můžeš tiskárnu vypnout a **ručně pootočit Z šroubem**. Po opětovném zapnutí zůstává skutečná poloha Z pro Klipper neznámá – a to je v pořádku.

> [!CAUTION]
> Neposouvej násilím zapnutý krokový motor.

## 8. Nejdůležitější test – zastav Z sepnutím sondy

Měj ruku připravenou u vypínače a spusť:

```text
G28 Z
```

BLTouch vysune pin a Z začne pomalu sjíždět. Dokud je tryska bezpečně vysoko nad podložkou, **jemně prstem sepni pin BLTouch**.

Správně musí Klipper v okamžiku sepnutí homing Z ukončit.

Pokud Z pokračuje dolů:

**OKAMŽITĚ TISKÁRNU ZASTAV NEBO VYPNI.**

Zkontroluj zapojení, `sensor_pin`, konfiguraci sondy a `QUERY_PROBE`.

## 9. Test několikrát zopakuj

Chceme vidět, že BLTouch pokaždé vysune pin, Z začne pomalu sjíždět a ruční sepnutí pokaždé zastaví Z-home.

Teprve potom pokračuj.

## 10. První skutečný Z-home

Spusť:

```text
G28 Z
```

Pin BLTouch se musí dotknout podložky **dříve než tryska**. Pokud se tryska nebezpečně přibližuje k bedu a sonda stále nemůže sepnout, test okamžitě zastav a oprav mechanickou výšku sondy.

## 11. Teprve teď kompletní G28

Pokud jsou samostatně ověřeny X, Y a BLTouch/Z:

```text
G28
```

## 12. Z-offset ještě není hotový

Funkční Z-home **neznamená správnou vzdálenost trysky od podložky**. Nekopíruj Z-offset z jiné tiskárny.

Z-offset zkalibruj samostatně podle kapitoly 06 pomocí `PROBE_CALIBRATE`.

## Co máš mít po této kapitole hotové

- sonda je mechanicky ve správné výšce,
- znáš a máš změřené X/Y offsety,
- pin lze vysunout a zasunout,
- `QUERY_PROBE` reaguje na sepnutí,
- první test Z-home proběhl vysoko nad podložkou,
- ruční sepnutí spolehlivě zastaví Z,
- při skutečném homingu se sonda dotkne podložky dříve než tryska,
- kompletní `G28` funguje bezpečně.

Další krok:

➡️ **04 – Kontrola topení a PID tuning**

### Oficiální dokumentace

- [Klipper – BLTouch](https://www.klipper3d.org/BLTouch.html)
- [Klipper – Probe calibration](https://www.klipper3d.org/Probe_Calibrate.html)
- [Marlin – XYZ Probe Offset / M851](https://marlinfw.org/docs/gcode/M851.html)
