# 00 – Než začneš upravovat printer.cfg

Tuhle kapitolu si přečti **ještě před prvním nastavováním motorů, homingu a sondy**.

Klipper config není univerzální seznam hodnot. Některé znaky a parametry přímo mění elektrickou logiku nebo směr pohybu a musí se posuzovat **u každého pinu, motoru a senzoru zvlášť**.

## 1. Co znamenají znaky před piny

### `!` – invertovaná logika

Příklad:

```ini
dir_pin: !PB12
```

`!` obrátí logickou úroveň pinu. U `dir_pin` tím prakticky obrátíš směr motoru.

To ale neznamená, že když potřebuje `!` osa X, musí ho mít i Y, Z nebo extruder. **Každý motor kontroluj samostatně.**

Stejný znak může být použit i u jiných pinů, například `enable_pin` nebo tlačítka/endstopu, kde neznamená „otoč motor“, ale obecně **invertuj logiku tohoto konkrétního signálu**.

### `^` – interní pull-up

Příklad:

```ini
diag_pin: ^PC0
```

`^` zapne interní pull-up rezistor vstupu MCU. Často ho uvidíš u endstopů, DIAG pinů TMC nebo sond.

### `~` – interní pull-down

Klipper podporuje u vstupních pinů také `~`, které zapíná interní pull-down rezistor, pokud ho daný MCU podporuje.

### Kombinace znaků

Můžeš narazit například na:

```ini
click_pin: ^!EXP1_2
```

Tady se současně použije pull-up (`^`) a invertovaná logika (`!`).

### `#` – komentář

```ini
# Toto Klipper neprovede.
```

`#` označuje komentář. V našem výukovém `printer.cfg` je komentářů schválně hodně, aby bylo vidět **co která část dělá a proč tam je**.

> [!IMPORTANT]
> Nekopíruj `!`, `^` ani jiné modifikátory automaticky z vedlejší osy nebo z cizího configu. Správnost vždy ověř pro konkrétní pin a hardware.

## 2. Po mechanické úpravě znovu změř skutečný rozsah os

Výměna hardwaru může změnit geometrii tiskárny, i když jsi nezměnil rám.

Typický příklad je přestavba původního Enderu z Bowdenu na **Direct Drive**, jiný držák hotendu, větší chlazení nebo jiný toolhead. Nová sestava může být hlubší než původní a při pohybu Y může dříve narazit do konstrukce.

Výsledkem může být, že původních například 235 mm fyzicky už není bezpečně použitelných.

Po podobné úpravě proto vždy znovu ověř:

- skutečné minimum a maximum X,
- skutečné minimum a maximum Y,
- skutečné maximum Z,
- polohu trysky vůči bedu,
- polohu BLTouch/probe vůči bedu,
- `position_min`, `position_max` a případně `position_endstop`,
- hranice `bed_mesh`,
- `safe_z_home`,
- parkovací polohy maker.

> [!CAUTION]
> Do `position_max` nezapisuj rozměr bedu jen proto, že má podložka 235 × 235 mm. Klipper potřebuje znát **bezpečný mechanický rozsah konkrétní tiskárny**.

## 3. Původní mechanické endstopy nemusíš odmontovat

Při přechodu na sensorless homing může být elektrický konektor původního endstopu odpojený, ale samotný spínač/držák nemusíš automaticky fyzicky odstranit.

U osy Y může původní držák endstopu dál fungovat jako **mechanický doraz**, proti kterému osa při sensorless homingu dojede. Pokud jeho poloha odpovídá novému bezpečnému HOME Y, může být užitečné ho na místě nechat.

Po každé mechanické přestavbě ale ověř, že tento doraz stále zastavuje osu na bezpečném místě a že před ním nenarazí jiná část nového toolheadu nebo bedu.

## 4. U Enderu lze někdy získat jinou polohu Y otočením vozíku bedu

U některých Ender 3 sestav není plechový vozík pod vyhřívanou podložkou vůči pojezdovým kolečkům a upevnění geometricky úplně souměrný.

Při větší přestavbě je proto možné zkontrolovat, zda **otočení Y vozíku / desky pod hotbedem o 180°** neposune využitelnou polohu podložky vůči trysce a nepomůže získat vhodnější rozsah Y.

Tohle ale není univerzální „povinná úprava“. Záleží na konkrétní verzi Enderu, vozíku, kabeláži bedu a použitém toolheadu.

Před takovou změnou zkontroluj hlavně:

- vedení a odlehčení kabelu vyhřívané podložky,
- kolize s rámem a elektronikou,
- polohu koleček/excentrických matic,
- šrouby a pružiny/silikonové sloupky,
- skutečný nový rozsah Y po složení.

Po otočení vozíku znovu změř celý rozsah Y a podle něj uprav config. **Nehádej hodnotu podle původního Enderu.**

## 5. Výukový config vs. krátký provozní config

Soubor `config/printer.cfg` má schválně hodně českých komentářů. Je určený k tomu, aby se z něj začátečník učil.

Jakmile víš, co jednotlivé části znamenají, můžeš většinu komentářů odstranit. Funkčně se nic nezmění – vznikne jen výrazně kratší a přehlednější provozní config.

V kořeni tohoto repozitáře najdeš také `printer-example.cfg`, který ukazuje právě takovou zkrácenou podobu.

> [!WARNING]
> Kratší neznamená univerzální. Ani ukázkový krátký config nekopíruj bez kontroly vlastních pinů, rozměrů, termistorů, motorů, sondy a kalibračních hodnot.

Další krok:

➡️ **01 – První spuštění a bezpečná kontrola**
