# 01 – První spuštění a bezpečná kontrola

V této chvíli předpokládáme, že máš nainstalovaný Klipper, Moonraker a Mainsail, desku **BTT SKR Mini E3 V3.0** flashnutou Klipperem a připravený základní `printer.cfg`.

> [!CAUTION]
> **Ještě nemačkej HOME ALL.** Nejdřív ověříme komunikaci, teploty, endstopy, BLTouch a směry motorů. První kontrolu dělej u tiskárny a buď připravený ji vypnout.

## 1. Zkopíruj konfiguraci

Soubor z tohoto projektu:

```text
config/printer.cfg
```

zkopíruj do:

```text
~/printer_data/config/printer.cfg
```

Před restartem Klipperu musíš upravit MCU ID.

Na Raspberry Pi spusť:

```bash
ls /dev/serial/by-id/*
```

Výsledek bude podobný:

```text
/dev/serial/by-id/usb-Klipper_stm32g0b1xx_XXXXXXXXXXXXXXXX-if00
```

V `printer.cfg` nahraď:

```ini
[mcu]
serial: /dev/serial/by-id/CHANGE_ME
```

skutečnou cestou k tvojí desce.

Ulož soubor a v Mainsailu použij **Firmware Restart**.

## 2. Klipper se musí připojit

Pokud je vše správně, Mainsail nesmí hlásit chybu připojení k MCU ani chybu při načítání konfigurace.

Pokud vidíš například chybu typu:

```text
mcu 'mcu': Unable to connect
```

**nepokračuj dál.** Nejprve zkontroluj MCU ID, USB kabel, napájení desky a zda byl firmware správně flashnut.

## 3. Zkontroluj teploty – ještě nic nezahřívej

Po připojení se podívej na teplotu hotendu a podložky.

Při pokojové teplotě by oba senzory měly ukazovat přibližně teplotu okolí. Nemusí ukazovat naprosto stejnou hodnotu.

Pokud jeden senzor ukazuje nesmyslnou hodnotu nebo Mainsail hlásí chybu ADC/teploty, **nezapínej topení**.

V našem ukázkovém configu jsou:

```ini
[extruder]
sensor_type: EPCOS 100K B57560G104F
```

a:

```ini
[heater_bed]
sensor_type: ATC Semitec 104GT-2
```

Tyto typy musí odpovídat skutečným termistorům na tvojí tiskárně.

## 4. Ověř endstopy

Do konzole Mainsailu napiš:

```text
QUERY_ENDSTOPS
```

Protože tento projekt používá **sensorless homing X/Y**, jejich stav souvisí s TMC2209 a DIAG piny. Zatím nic nehomuj.

Osa Z používá BLTouch jako:

```ini
endstop_pin: probe:z_virtual_endstop
```

Sensorless homing budeme nastavovat samostatně v dalším návodu.

## 5. Otestuj BLTouch

Nejdřív zkontroluj, že sonda může volně vysunout pin a do ničeho nenarazí.

Do konzole napiš:

```text
BLTOUCH_DEBUG COMMAND=pin_down
```

Pin by se měl vysunout.

Potom:

```text
BLTOUCH_DEBUG COMMAND=pin_up
```

Pin by se měl zasunout.

Následně:

```text
BLTOUCH_DEBUG COMMAND=pin_down
QUERY_PROBE
```

Bez stisknutí pinu očekávej stav odpovídající otevřené sondě. Potom pin BLTouch **jemně ručně stiskni** a znovu spusť:

```text
QUERY_PROBE
```

Stav se musí změnit.

Nakonec pin zasuň:

```text
BLTOUCH_DEBUG COMMAND=pin_up
```

> [!CAUTION]
> Pokud BLTouch nereaguje správně, **nehomuj osu Z**.

## 6. Ověř směry motorů bez homingu

Klipper obsahuje příkaz:

```text
STEPPER_BUZZ STEPPER=stepper_x
```

Motor se krátce pohne jedním směrem a vrátí se zpět.

Stejně otestuj:

```text
STEPPER_BUZZ STEPPER=stepper_y
STEPPER_BUZZ STEPPER=stepper_z
STEPPER_BUZZ STEPPER=extruder
```

U extruderu není v tuto chvíli cílem vytlačovat filament. Jde pouze o ověření, že Klipper ovládá správný motor.

Pokud se při testu X rozjede Y nebo se ozve jiný motor, zkontroluj zapojení ještě před dalším krokem.

> [!NOTE]
> `STEPPER_BUZZ` je bezpečnější první test než náhodné ruční posouvání os, protože ještě nemáme ověřený homing a Klipper nezná skutečnou polohu tiskové hlavy.

## 7. Teprve potom budeme řešit homing

V tuto chvíli bys měl mít ověřeno:

- Klipper komunikuje s SKR Mini E3 V3.0,
- teplotní senzory dávají smysluplné hodnoty,
- BLTouch se vysune, zasune a reaguje na dotyk,
- jednotlivé motory odpovídají správným osám.

**HOME ALL stále ještě není další krok.**

Nejdřív nastavíme a bezpečně otestujeme sensorless homing os X a Y.

Pokračuj:

➡️ **02 – Sensorless homing**
