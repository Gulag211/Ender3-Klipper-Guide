# 03 – BLTouch a první bezpečný Z-home

Tento návod navazuje na nastavení a otestování sensorless homingu X/Y.

Teď ověříme BLTouch a teprve potom dovolíme trysce přiblížit se k podložce.

> [!CAUTION]
> **Nedělej první Z-home rovnou proti podložce.** Nejdřív otestujeme sondu vysoko nad bedem a její pin při homingu ručně sepneme. Pokud je něco špatně zapojené nebo nastavené, získáš čas tiskárnu zastavit dřív, než tryska narazí do podložky.

## 1. BLTouch není vždy stejný BLTouch

Existuje několik hardwarových revizí originálního BLTouch a také CR Touch a různé kompatibilní sondy/klony.

Podle konkrétní sondy, její revize a způsobu zapojení může být potřeba trochu jiné nastavení.

Proto **nekopíruj celý `[bltouch]` blok jen proto, že někomu jinému funguje na Enderu 3**.

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

Toto nastavení vychází z konkrétní tiskárny, ze které vznikl tento projekt.

Parametry jako:

```ini
probe_with_touch_mode
pin_up_touch_mode_reports_triggered
stow_on_each_sample
```

nemusí být vhodné pro každou sondu.

Také `x_offset` a `y_offset` závisejí na držáku sondy a musí odpovídat skutečné poloze BLTouch vůči trysce.

## 2. Nejdřív otestuj samotný pin

Tiskárna zatím nemusí nikam jezdit.

Do konzole Mainsailu napiš:

```text
BLTOUCH_DEBUG COMMAND=pin_down
```

Pin sondy se musí vysunout.

Potom:

```text
BLTOUCH_DEBUG COMMAND=pin_up
```

Pin se musí zasunout.

Zopakuj tento test několikrát.

Pokud pin nereaguje správně, **nepokračuj k homingu Z**.

## 3. Ověř, že Klipper pozná sepnutí sondy

Vysuň pin:

```text
BLTOUCH_DEBUG COMMAND=pin_down
```

Potom:

```text
QUERY_PROBE
```

Klipper musí hlásit stav odpovídající nesepnuté sondě.

Teď pin BLTouch **jemně ručně zatlač prstem nahoru** a znovu použij:

```text
QUERY_PROBE
```

Stav se musí změnit.

Nakonec:

```text
BLTOUCH_DEBUG COMMAND=pin_up
```

> [!IMPORTANT]
> Samotné vysouvání a zasouvání pinu ještě nestačí. Musíme ověřit i to, že Klipper elektricky pozná jeho sepnutí.

## 4. Pro první Z-home používáme pouze 2 mm/s

V základním `printer.cfg` je záměrně:

```ini
[stepper_z]
homing_speed: 2
```

Tohle **není doporučená finální rychlost**.

Je to bezpečnostní nastavení pro první testy.

Až bude BLTouch spolehlivě fungovat, můžeš rychlost postupně zvýšit například do rozsahu:

```text
5–16 mm/s
```

Pro první test nám rychlost nic nepřináší. Chceme hlavně dostatek času reagovat.

## 5. Připrav tiskárnu pro test ve vzduchu

Nejdřív musí být správně ověřený homing X a Y podle předchozí kapitoly.

Proveď:

```text
G28 X
G28 Y
```

Potom chceme dostat osu Z přibližně **do poloviny její výšky**, aby mezi tryskou a podložkou zůstala velká bezpečnostní vzdálenost.

Pokud Z ještě není zahomované, Klipper běžný absolutní pohyb Z nemusí dovolit. Pro tento bezpečnostní test proto můžeš tiskárnu vypnout a **ručně pootočit Z šroubem**, případně bezpečně mechanicky nastavit výšku tak, aby byla hlava přibližně v polovině osy.

Po opětovném zapnutí zůstává skutečná poloha Z pro Klipper neznámá – a to je v pořádku. Právě ji budeme homovat.

> [!CAUTION]
> Neposouvej násilím zapnutý krokový motor. Pokud potřebuješ mechanicky změnit výšku před prvním homingem, udělej to s vypnutými motory/tiskárnou.

## 6. Nejdůležitější test – zastav Z prstem

Teď přichází test, který může zachránit podložku.

Měj jednu ruku připravenou u vypínače.

Spusť pouze Z-home:

```text
G28 Z
```

BLTouch vysune pin a osa Z začne **velmi pomalu** sjíždět směrem k podložce.

Ale nenech sondu dojet až k bedu.

Dokud je tryska stále bezpečně vysoko nad podložkou, **jemně prstem zatlač pin BLTouch nahoru**, jako kdyby se dotkl podložky.

### Správný výsledek

V okamžiku sepnutí BLTouch musí Klipper detekovat sondu a homing Z ukončit.

### Špatný výsledek

Pokud Z pokračuje směrem dolů i po sepnutí sondy:

**OKAMŽITĚ TISKÁRNU ZASTAV NEBO VYPNI.**

Nenechávej ji pokračovat až k podložce „jestli se třeba chytne později“.

V takovém případě zkontroluj:

- zapojení BLTouch,
- `sensor_pin`,
- konfiguraci konkrétní verze sondy,
- výsledek `QUERY_PROBE`.

> [!NOTE]
> Prstem testujeme **pin sondy**. Nesnaž se rukou zastavovat osu Z nebo držet celý toolhead.

## 7. Test zopakuj

Jeden úspěšný pokus ještě není důvod pustit trysku k bedu.

Znovu dostaň hlavu do bezpečné výšky a test zopakuj.

Chceme vidět, že:

1. BLTouch spolehlivě vysune pin,
2. Z začne pomalu sjíždět,
3. ruční sepnutí pinu pokaždé zastaví Z-home.

Teprve potom pokračuj.

## 8. První skutečný Z-home

Nyní můžeš poprvé nechat BLTouch skutečně dojet k podložce.

Stále měj ruku připravenou u vypínače a sleduj sondu i trysku.

Spusť:

```text
G28 Z
```

Pin BLTouch se musí dotknout podložky **dříve než tryska**.

Pokud se tryska nebezpečně přibližuje k bedu a sonda stále nemůže sepnout, test okamžitě zastav. Pravděpodobně je problém v mechanické výšce sondy nebo jejím držáku.

## 9. Teprve teď kompletní G28

Pokud už máme samostatně ověřeno:

- X sensorless homing,
- Y sensorless homing,
- BLTouch,
- bezpečný Z-home,

můžeme poprvé použít:

```text
G28
```

Tiskárna by nyní měla bezpečně zahomovat všechny tři osy.

## 10. Z-offset ještě není hotový

To, že Z-home funguje, **neznamená, že je správně nastavená vzdálenost trysky od podložky**.

Nekopíruj Z-offset z jiné tiskárny.

Například hodnota z tiskárny, ze které tento projekt vychází, není použitelná jako univerzální hodnota pro tvoji sondu, držák, hotend a trysku.

Z-offset zkalibruj samostatně podle dalšího návodu.

## Co máš mít po této kapitole hotové

- znáš typ/verzi sondy, kterou používáš,
- pin BLTouch lze vysunout a zasunout,
- `QUERY_PROBE` reaguje na sepnutí,
- první test Z-home proběhl vysoko nad podložkou,
- ruční sepnutí pinu spolehlivě zastaví Z,
- sonda se při skutečném homingu dotkne podložky dříve než tryska,
- kompletní `G28` funguje bezpečně.

Další krok:

➡️ **04 – Kontrola topení a PID tuning**
