# Ender 3 + Klipper + BTT SKR Mini E3 V3.0

Praktický návod pro zprovoznění a nastavení **Creality Ender 3** s deskou **BTT SKR Mini E3 V3.0** a firmwarem **Klipper**.

Tento projekt vychází z reálně provozované a upravené tiskárny. Cílem není nabídnout jeden „zázračný“ `printer.cfg`, který stačí slepě zkopírovat, ale ukázat postup tak, aby i začátečník věděl **co nastavuje, proč to nastavuje a co musí přizpůsobit své tiskárně**.

> [!WARNING]
> Konfigurace není univerzální pro každý Ender 3. Hodnoty jako MCU ID, Z-offset, PID, rotation distance, sensorless homing citlivost, Pressure Advance nebo Input Shaper musíš nastavit pro svůj konkrétní stroj.

## 🎥 Jak může tiskárna fungovat po úpravách

Sem bude doplněno video z mého YouTube s ukázkou reálného provozu upraveného Enderu 3.

**▶️ TODO: vložit odkaz na ukázkové video**

> Klipper sám o sobě automaticky neudělá z Enderu rychlou tiskárnu. Výsledná rychlost a kvalita závisí také na mechanickém stavu tiskárny, použitých komponentech a správné kalibraci.

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

## 🧭 Doporučený postup

1. Připravit Raspberry Pi.
2. Nainstalovat Klipper, Moonraker a Mainsail.
3. Vytvořit firmware pro SKR Mini E3 V3.0.
4. Flashnout řídicí desku.
5. Zjistit správné MCU ID.
6. Naklonovat tento repozitář.
7. Připravit základní `printer.cfg`.
8. Ověřit endstopy, směry motorů a homing.
9. Ověřit BLTouch.
10. Ověřit teplotní senzory a topení.
11. Kalibrovat extruder.
12. PID tuning.
13. Z-offset a bed mesh.
14. Pressure Advance.
15. Input Shaper.
16. Teprve potom přidat další funkce, například KAMP.

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

# 4. Stažení tohoto projektu

Nemusíš stahovat ZIP, rozbalovat ho na počítači a ručně kopírovat soubory.

```bash
cd ~
git clone https://github.com/Gulag211/Ender3-Klipper-Guide.git
cd Ender3-Klipper-Guide
ls
```

## Co dělá git clone?

`git clone` vytvoří v aktuálním adresáři kopii projektu včetně jeho Git historie. Později tak lze změny z GitHubu jednoduše stáhnout pomocí Gitu místo opakovaného stahování ZIP archivů.

> [!IMPORTANT]
> Po naklonování projektu zatím slepě nekopíruj konfiguraci do `printer_data/config`. Nejprve projdi návod a uprav hodnoty pro svoji tiskárnu.

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

# 6. Konfigurace projektu

V adresáři `config/` bude základní konfigurace potřebná pro první zprovoznění tiskárny.

Pokročilé části budou oddělené:

```text
optional/
├── adxl345.cfg
├── input_shaper.cfg
└── kamp.cfg
```

Začátečník tedy nebude potřebovat ADXL345 ani KAMP jen proto, aby mohl poprvé připojit tiskárnu ke Klipperu.

---

# 7. Doporučené pořadí kontroly a kalibrace

```text
MCU
 ↓
endstopy
 ↓
motory
 ↓
sensorless homing
 ↓
BLTouch
 ↓
teplotní senzory
 ↓
hotend + bed
 ↓
extruder
 ↓
PID tuning
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
```

Nesnaž se řešit deset problémů současně. Pokud ještě nefunguje správně homing, nemá smysl řešit KAMP.

---

# ⚠️ Bezpečnost

První testy dělej vždy pod dohledem.

Před zapnutím topení ověř, že Klipper zobrazuje rozumnou teplotu hotendu i bedu.

Před prvním homingem ověř směry pohybu, funkci endstopů nebo sensorless homingu, funkci BLTouch a že mechanika může bezpečně projet požadovaný rozsah.

---

# 📚 Připravované návody

Postupně budou doplněny návody pro první spuštění, sensorless homing, BLTouch, kalibraci extruderu, PID tuning, Z-offset, Pressure Advance, Input Shaper, ADXL345 a KAMP.

---

# 👤 O projektu

Tento projekt vznikl z mojí vlastní konfigurace upraveného Enderu 3.

Nejde o oficiální konfiguraci Creality, BigTreeTech ani projektu Klipper. Je to praktický návod a výchozí bod pro lidi, kteří si chtějí Ender 3 upravit a zároveň pochopit, **co jednotlivá nastavení dělají**.

Pokud najdeš chybu nebo máš užitečné vylepšení, můžeš otevřít Issue nebo Pull Request.
