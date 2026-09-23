# Ender 3 + Klipper + BTT SKR Mini E3 V3.0

Praktický český návod pro zprovoznění a nastavení **Creality Ender 3** s deskou **BTT SKR Mini E3 V3.0** a firmwarem **Klipper**.

Projekt vychází z reálně provozované a upravené tiskárny. Cílem není nabídnout jeden „zázračný“ `printer.cfg`, který stačí slepě zkopírovat, ale ukázat postup tak, aby i začátečník věděl **co nastavuje, proč to nastavuje a co musí přizpůsobit své tiskárně**.

> [!WARNING]
> Konfigurace není univerzální pro každý Ender 3. MCU ID, Z-offset, PID, rotation distance, sensorless homing, Pressure Advance, Input Shaper a další hodnoty musí odpovídat konkrétnímu stroji.

## 🚀 Kde začít

Pokud s Klipperem začínáš, **nezačínej kopírováním celého configu a náhodným zkoušením příkazů**.

Doporučená cesta je:

1. projít instalaci níže v tomto README,
2. otevřít `config/printer.cfg`,
3. pokračovat návody v `guides/` **od 01 postupně dál**,
4. volitelné funkce z `optional/` řešit až ve chvíli, kdy základ tiskárny spolehlivě funguje.

### 📚 Podrobné návody a vysvětlivky

README je hlavně **rozcestník a instalační základ**. Podrobnější vysvětlení, bezpečné testovací postupy, příkazy a důvody jednotlivých nastavení jsou v samostatných kapitolách:

| Krok | Návod | Co řeší |
|---|---|---|
| 01 | [První spuštění](guides/01-first-start.md) | MCU, teploty, endstopy, BLTouch, motory a první bezpečné kontroly |
| 02 | [Sensorless homing](guides/02-sensorless-homing.md) | StallGuard, `driver_sgthrs`, bezpečné ladění X/Y |
| 03 | [BLTouch](guides/03-bltouch.md) | kontrola sondy a bezpečný první Z-home |
| 04 | [PID tuning](guides/04-pid-tuning.md) | kontrola topení a PID hotendu i bedu |
| 05 | [Kalibrace extruderu](guides/05-extruder-calibration.md) | `rotation_distance`, `gear_ratio`, význam `!`, `^` a komentářů `#` |
| 06 | [Z-offset](guides/06-z-offset.md) | `PROBE_CALIBRATE`, papírek, `TESTZ`, `SAVE_CONFIG` |
| 07 | [Bed Mesh](guides/07-bed-mesh.md) | měřicí oblast, offset sondy, bezpečné hranice meshe |
| 08 | [Pressure Advance](guides/08-pressure-advance.md) | princip PA a vlastní kalibrace |
| 09 | [Input Shaper + ADXL345](guides/09-input-shaper-adxl345.md) | rezonance, X na toolheadu, Y na bedu |
| 10 | [KAMP](guides/10-kamp.md) | adaptivní mesh a purge jako volitelné rozšíření |
| 11 | [PRINT_START / PRINT_END](guides/11-print-start-end.md) | startovací a ukončovací makra krok za krokem |
| 12 | [První tisk a rychlost](guides/12-first-print-speed.md) | první testovací tisk, rychlost, akcelerace a objemový průtok |

> [!TIP]
> Když v `printer.cfg` narazíš na parametr, kterému nerozumíš, nejdřív se podívej do odpovídající kapitoly v `guides/`. Config obsahuje stručné české komentáře, zatímco návody vysvětlují věci podrobněji a v souvislostech.

## 🗂️ Co je kde

```text
Ender3-Klipper-Guide/
├── README.md
├── config/
│   ├── printer.cfg
│   └── macros.cfg
├── guides/
│   ├── 01-first-start.md
│   ├── 02-sensorless-homing.md
│   ├── 03-bltouch.md
│   ├── 04-pid-tuning.md
│   ├── 05-extruder-calibration.md
│   ├── 06-z-offset.md
│   ├── 07-bed-mesh.md
│   ├── 08-pressure-advance.md
│   ├── 09-input-shaper-adxl345.md
│   ├── 10-kamp.md
│   ├── 11-print-start-end.md
│   └── 12-first-print-speed.md
└── optional/
    ├── adxl345.cfg
    ├── input_shaper.cfg
    └── kamp.cfg
```

### `config/printer.cfg`

Základní konfigurace tiskárny. Je opatřená českými komentáři přímo u důležitých funkcí a hodnot.

### `config/macros.cfg`

Volitelná jednoduchá makra, například zavedení/vytažení filamentu, parkování hlavy a předehřev. Uvnitř jsou české vysvětlivky.

### `guides/`

**Tady hledej podrobnější návod.** Kapitoly nejsou jen seznam příkazů – vysvětlují také proč se daný test dělá, co máš očekávat a kdy raději nepokračovat.

### `optional/`

Doplňky, které pro první zprovoznění nepotřebuješ: ADXL345, Input Shaper a KAMP. Přidávej je až na funkční a zkalibrovaný základ.

## 🎥 Jak může tiskárna fungovat po úpravách

Sem bude doplněno video z mého YouTube s ukázkou reálného provozu upraveného Enderu 3.

**▶️ TODO: vložit odkaz na ukázkové video**

> Klipper sám automaticky neudělá z Enderu rychlou tiskárnu. Výsledná rychlost a kvalita závisí také na mechanickém stavu, použitých komponentech, hotendu, chlazení a správné kalibraci.

## 🔧 Hardware, ze kterého projekt vychází

- Creality Ender 3
- BTT SKR Mini E3 V3.0
- TMC2209
- sensorless homing X/Y
- BLTouch
- převodovaný extruder 3:1
- 12864 LCD
- Raspberry Pi
- Klipper + Moonraker + Mainsail
- volitelně ADXL345 / Input Shaper
- volitelně KAMP

Pokud máš jiný extruder, sondu, termistor nebo jinou mechanickou úpravu, **nekopíruj odpovídající hodnoty bez kontroly**.

---

# 1. Raspberry Pi

Nejjednodušší cesta pro začátečníka je použít **MainsailOS**.

Pomocí Raspberry Pi Imager připrav SD kartu a při instalaci nastav hostname, Wi-Fi, uživatelské jméno, heslo a SSH.

Po spuštění Raspberry Pi by měl být Mainsail dostupný přes jeho IP adresu nebo například:

```text
http://mainsailos.local
```

## Připojení přes SSH

```bash
ssh pi@mainsailos.local
```

`pi` nahraď uživatelským jménem, které jsi nastavil při instalaci.

---

# 2. Firmware pro BTT SKR Mini E3 V3.0

Připoj se přes SSH:

```bash
cd ~/klipper
make menuconfig
```

Pro SKR Mini E3 V3.0 použij:

```text
Micro-controller Architecture: STMicroelectronics STM32
Processor model: STM32G0B1
Bootloader offset: 8KiB bootloader
Communication interface: USB
```

Potom:

```bash
make
```

Výsledný soubor:

```text
~/klipper/out/klipper.bin
```

## Flash desky

1. Zkopíruj `klipper.bin` na microSD kartu.
2. Přejmenuj jej na `firmware.bin`.
3. Vlož kartu do SKR Mini E3 V3.0.
4. Vypni a znovu zapni tiskárnu.

Po úspěšném flashnutí deska obvykle přejmenuje soubor na `FIRMWARE.CUR`.

---

# 3. Zjištění MCU ID

**Nekopíruj MCU ID z cizího printer.cfg.**

```bash
ls /dev/serial/by-id/*
```

Výsledek může vypadat například:

```text
/dev/serial/by-id/usb-Klipper_stm32g0b1xx_XXXXXXXXXXXXXXXX-if00
```

Použiješ jej v:

```ini
[mcu]
serial: /dev/serial/by-id/usb-Klipper_stm32g0b1xx_XXXXXXXXXXXXXXXX-if00
```

---

# 4. Stažení projektu

Nemusíš stahovat ZIP, rozbalovat ho na počítači a potom ručně přenášet soubory.

```bash
cd ~
git clone https://github.com/Gulag211/Ender3-Klipper-Guide.git
cd Ender3-Klipper-Guide
ls
```

### Co dělá `git clone`?

`git clone` vytvoří v aktuálním adresáři místní kopii projektu včetně Git historie. Pozdější změny z GitHubu lze díky tomu stahovat Gitem místo opakovaného stahování ZIP archivů.

> [!IMPORTANT]
> Po naklonování projektu zatím slepě nekopíruj konfiguraci do `printer_data/config`. Nejdřív projdi návod a uprav hodnoty pro svoji tiskárnu.

Podrobný postup prvního spuštění pokračuje zde:

**➡️ [01 – První spuštění](guides/01-first-start.md)**

---

# 5. Co rozhodně nekopírovat naslepo

- MCU serial
- Z-offset
- PID hotendu a bedu
- `rotation_distance`
- `gear_ratio`
- Pressure Advance
- Input Shaper
- BLTouch offset
- sensorless homing citlivost
- rozměry a limity os
- proudy motorů
- typy termistorů

Špatná hodnota nemusí znamenat pouze horší tisk. U některých nastavení může vést i k nesprávnému pohybu nebo řízení teplot.

---

# 6. Doporučené pořadí kontroly

```text
MCU
 ↓
endstopy a motory
 ↓
sensorless homing
 ↓
BLTouch
 ↓
teplotní senzory a topení
 ↓
PID
 ↓
extruder
 ↓
Z-offset
 ↓
bed mesh
 ↓
Pressure Advance
 ↓
Input Shaper
 ↓
KAMP
 ↓
první bezpečné zvyšování rychlosti
```

Nesnaž se řešit deset problémů současně. Pokud ještě nefunguje správně homing, nemá smysl řešit KAMP.

---

# ⚠️ Bezpečnost

První testy dělej vždy pod dohledem.

Před zapnutím topení ověř, že Klipper zobrazuje rozumnou teplotu hotendu i bedu.

Před prvním homingem ověř směry pohybu, funkci endstopů nebo sensorless homingu, funkci BLTouch a že mechanika může bezpečně projet požadovaný rozsah.

Pokud se při prvním testu osa, sonda nebo topení chová jinak, než očekáváš, test přeruš a nejdřív zjisti proč.

---

# 👤 O projektu

Tento projekt vznikl z mojí vlastní konfigurace upraveného Enderu 3.

Nejde o oficiální konfiguraci Creality, BigTreeTech ani projektu Klipper. Je to praktický návod a výchozí bod pro lidi, kteří si chtějí Ender 3 upravit a zároveň pochopit, **co jednotlivá nastavení dělají**.

Pokud najdeš chybu nebo máš užitečné vylepšení, můžeš otevřít Issue nebo Pull Request.
