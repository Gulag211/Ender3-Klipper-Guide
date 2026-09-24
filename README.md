# Ender 3 + Klipper + BTT SKR Mini E3 V3.0

Praktický český návod pro zprovoznění a nastavení **Creality Ender 3** s deskou **BTT SKR Mini E3 V3.0** a firmwarem **Klipper**.

Projekt vychází z reálně provozované a upravené tiskárny. Cílem není nabídnout jeden „zázračný“ `printer.cfg`, který stačí slepě zkopírovat, ale ukázat postup tak, aby i začátečník věděl **co nastavuje, proč to nastavuje a co musí přizpůsobit své tiskárně**.

> [!WARNING]
> Konfigurace není univerzální pro každý Ender 3. MCU ID, Z-offset, PID, rotation distance, sensorless homing, Pressure Advance, Input Shaper a další hodnoty musí odpovídat konkrétnímu stroji.

## ⚡ Než začneš: přečti si nejdřív značky v configu

Ještě před prvním nastavováním os si přečti **[00 – Základy printer.cfg a mechanické změny](guides/00-config-basics.md)**.

Hned na začátku tam vysvětlujeme `!`, `^`, `~`, `#` a jejich kombinace. Tyto znaky se neposuzují „pro celou tiskárnu“, ale **pro každý konkrétní pin, motor, endstop nebo sondu zvlášť**. Například `!` u `dir_pin` může obrátit směr jednoho motoru, ale vůbec to neznamená, že ho máš přidat také k ostatním osám.

Stejná kapitola vysvětluje i důležitou mechanickou věc: po výměně Bowdenu za Direct Drive, hotendu, toolheadu nebo jiného hardwaru **znovu fyzicky změř skutečný rozsah X/Y/Z**. Nový hlubší toolhead může například zkrátit využitelný chod Y. Je tam také poznámka k ponechání původního mechanického Y endstopu jako dorazu a k možnosti kontroly orientace Y vozíku pod hotbedem.

## 🚀 Kde začít

Pokud s Klipperem začínáš, **nezačínej kopírováním celého configu a náhodným zkoušením příkazů**.

Doporučená cesta je:

1. připravit Raspberry Pi a nainstalovat Klipper/Moonraker/Mainsail,
2. vytvořit a nahrát firmware do SKR Mini E3 V3.0,
3. otevřít `config/printer.cfg`,
4. pokračovat návody v `guides/` **od 01 postupně dál**,
5. volitelné funkce z `optional/` řešit až ve chvíli, kdy základ tiskárny spolehlivě funguje.

### 📚 Podrobné návody a vysvětlivky

README je hlavně **rozcestník a instalační základ**. Podrobnější vysvětlení, bezpečné testovací postupy, příkazy a důvody jednotlivých nastavení jsou v samostatných kapitolách:

| Krok | Návod | Co řeší |
|---|---|---|
| 00 | [Základy configu a mechaniky](guides/00-config-basics.md) | `!`, `^`, `~`, `#`, směry, skutečný rozsah os a změny po přestavbě |\n| 01 | [První spuštění](guides/01-first-start.md) | MCU, teploty, endstopy, BLTouch, motory a první bezpečné kontroly |
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
│   ├── 00-config-basics.md\n│   ├── 01-first-start.md
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
Základní konfigurace tiskárny s českými komentáři přímo u důležitých funkcí a hodnot.

### `config/macros.cfg`
Volitelná jednoduchá makra, například zavedení/vytažení filamentu, parkování hlavy a předehřev.

### `guides/`
**Tady hledej podrobnější návod.** Kapitoly vysvětlují nejen příkazy, ale také proč se daný test dělá, co očekávat a kdy raději nepokračovat.

### `optional/`
ADXL345, Input Shaper a KAMP. Přidávej až na funkční a zkalibrovaný základ.

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

# 1. Raspberry Pi a instalace Klipperu

Pro úplného začátečníka doporučuji **MainsailOS**. Je to připravený systém pro Raspberry Pi, který už obsahuje základní Klipper stack.

Užitečné odkazy:

- [MainsailOS – GitHub](https://github.com/mainsail-crew/MainsailOS)
- [Mainsail dokumentace](https://docs.mainsail.xyz/)
- [Klipper dokumentace](https://www.klipper3d.org/)
- [Moonraker dokumentace](https://moonraker.readthedocs.io/)
- [KIAUH – GitHub](https://github.com/dw-0/kiauh)
- [Raspberry Pi Imager](https://www.raspberrypi.com/software/)

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

## Alternativa: instalace přes KIAUH

Pokud nepoužíváš hotový MainsailOS nebo chceš později snadno instalovat další části Klipper prostředí, velmi užitečný je **KIAUH – Klipper Installation And Update Helper**.

```bash
cd ~
sudo apt-get update
sudo apt-get install git -y
git clone https://github.com/dw-0/kiauh.git
./kiauh/kiauh.sh
```

V menu KIAUH potom můžeš instalovat a spravovat například Klipper, Moonraker a webové rozhraní Mainsail.

> [!TIP]
> Pro první tiskárnu není nutné kombinovat všechny možné instalační metody. Pokud použiješ MainsailOS a vše potřebné už funguje, není důvod Klipper znovu přeinstalovávat přes KIAUH.

---

# 2. Firmware pro BTT SKR Mini E3 V3.0

Připoj se k Raspberry Pi přes SSH:

```bash
cd ~/klipper
make menuconfig
```

Pro **SKR Mini E3 V3.0** nastav:

```text
Micro-controller Architecture: STMicroelectronics STM32
Processor model: STM32G0B1
Bootloader offset: 8KiB bootloader
Communication interface: USB
```

Po nastavení menu ukonči a konfiguraci ulož. Potom spusť:

```bash
make
```

Pokud kompilace proběhne úspěšně, Klipper vytvoří firmware zde:

```text
~/klipper/out/klipper.bin
```

tedy typicky:

```text
/home/TVUJ_UZIVATEL/klipper/out/klipper.bin
```

Můžeš si existenci souboru ověřit i přes SSH:

```bash
ls -lh ~/klipper/out/klipper.bin
```

## 🪟 Stažení klipper.bin do Windows pomocí WinSCP

Tohle je krok, na kterém může začátečník snadno tápat: příkaz `make` vytvořil firmware **na Raspberry Pi**, ne ve Windows.

Pro pohodlné nalezení a stažení souboru můžeš použít **WinSCP**:

- [WinSCP – oficiální stažení](https://winscp.net/eng/download.php)

Ve WinSCP vytvoř připojení:

```text
File protocol: SFTP
Host name: IP adresa Raspberry Pi
User name: stejné uživatelské jméno jako pro SSH
Password: tvoje heslo k Raspberry Pi
```

Po připojení otevři na Raspberry Pi:

```text
/home/TVUJ_UZIVATEL/klipper/out/
```

nebo jednoduše ve svém domovském adresáři:

```text
klipper → out
```

Uvnitř najdeš:

```text
klipper.bin
```

Přetáhni `klipper.bin` z Raspberry Pi například na plochu Windows.

> [!IMPORTANT]
> **Soubor se pro SKR Mini E3 V3.0 musí před vložením na microSD kartu jmenovat přesně `firmware.bin`.**
>
> Takže:
>
> `klipper.bin` → **`firmware.bin`**
>
> Nestačí ho pouze zkopírovat na kartu pod původním názvem.

### Pozor na skryté přípony ve Windows

Pokud Windows skrývá přípony známých souborů, dej pozor, aby výsledkem nebylo například:

```text
firmware.bin.bin
```

V Průzkumníku Windows je proto vhodné zapnout zobrazení **přípon názvů souborů**.

## Flash desky

1. Vytvoř pomocí `make` soubor `klipper.bin`.
2. Stáhni ho z `~/klipper/out/` do PC – například pomocí WinSCP.
3. Přejmenuj **`klipper.bin` na `firmware.bin`**.
4. Zkopíruj `firmware.bin` do kořenového adresáře microSD karty.
5. Vypni tiskárnu.
6. Vlož microSD kartu do SKR Mini E3 V3.0.
7. Zapni tiskárnu.
8. Po několika sekundách desku znovu vypni a kartu zkontroluj.

Po úspěšném flashnutí bootloader obvykle přejmenuje soubor na:

```text
FIRMWARE.CUR
```

To je dobrý první signál, že deska firmware zpracovala.

> [!NOTE]
> Pro tuto desku se firmware běžně nahrává přes microSD. Nespoléhej zde na `make flash`.

---

# 3. Zjištění MCU ID

Po připojení desky k Raspberry Pi přes USB spusť:

```bash
ls /dev/serial/by-id/*
```

Pokud firmware a USB komunikace fungují, měl by se objevit Klipper MCU, například:

```text
/dev/serial/by-id/usb-Klipper_stm32g0b1xx_XXXXXXXXXXXXXXXX-if00
```

Celou svoji cestu vlož do `printer.cfg`:

```ini
[mcu]
serial: /dev/serial/by-id/usb-Klipper_stm32g0b1xx_XXXXXXXXXXXXXXXX-if00
```

**Nekopíruj MCU ID z cizího printer.cfg. Každá deska má svoje.**

---

# 4. Stažení tohoto projektu

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

# 🔗 Užitečné odkazy

- [Klipper – dokumentace](https://www.klipper3d.org/)
- [Klipper – GitHub](https://github.com/Klipper3d/klipper)
- [Mainsail – dokumentace](https://docs.mainsail.xyz/)
- [MainsailOS – GitHub](https://github.com/mainsail-crew/MainsailOS)
- [Moonraker – dokumentace](https://moonraker.readthedocs.io/)
- [KIAUH – GitHub](https://github.com/dw-0/kiauh)
- [BIGTREETECH SKR Mini E3 – GitHub](https://github.com/bigtreetech/BIGTREETECH-SKR-mini-E3)
- [WinSCP – oficiální web](https://winscp.net/)
- [Raspberry Pi Imager](https://www.raspberrypi.com/software/)

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
