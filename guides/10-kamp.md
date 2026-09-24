# 10 – KAMP: adaptivní Bed Mesh a purge line

V základním nastavení už máme funkční Bed Mesh. Před každým tiskem ale měříme celou nastavenou oblast podložky.

To je jednoduché a spolehlivé, ale když tiskneš malý model uprostřed bedu, není vždy nutné měřit místa, kam se tryska během tisku vůbec nepodívá.

Tady přichází na řadu **KAMP – Klipper Adaptive Meshing & Purging**.

> [!IMPORTANT]
> KAMP je volitelné rozšíření. Nejdřív musí spolehlivě fungovat obyčejné `G28` a `BED_MESH_CALIBRATE`. KAMP nepoužívej k opravování nefunkčního základního Bed Meshe.

## 1. Co KAMP dělá

KAMP může podle oblasti skutečně používané konkrétním výtiskem vytvořit adaptivní mesh pouze tam, kde je potřeba.

Místo měření velké části podložky kvůli malému modelu tak může změřit jen jeho okolí.

Další užitečnou funkcí je adaptivní purge line – čisticí/naplňovací čára může být umístěna s ohledem na konkrétní výtisk.

Výhody mohou být:

- kratší příprava před tiskem,
- více měřicích bodů soustředěných do relevantní oblasti,
- purge line umístěná s ohledem na model.

## 2. KAMP potřebuje znát objekty v G-code

Aby Klipper věděl, kde se budou tisknout objekty, potřebuje informace o jednotlivých objektech z G-code.

Proto se používá:

```ini
[exclude_object]
```

Tato funkce je užitečná i sama o sobě – u více objektů může Klipper umožnit zrušit jeden problémový objekt a pokračovat v tisku ostatních.

Pro KAMP jsou ale informace o objektech důležité hlavně proto, aby dokázal určit skutečnou tiskovou oblast.

> [!IMPORTANT]
> Nestačí pouze přidat `[exclude_object]` do configu. Slicer/Moonraker musí Klipperu předat potřebné informace o objektech.

## 3. Nejdřív zálohuj funkční konfiguraci

Než začneš KAMP instalovat, měj uložený funkční stav tiskárny.

Tohle je přesně chvíle, kdy je Git velmi užitečný.

Pokud po instalaci něco přestane fungovat, musíš být schopný rozlišit:

- problém základní tiskárny,
- problém instalace KAMP,
- problém maker,
- problém dat ze sliceru.

Nepřidávej několik nových funkcí současně.

## 4. Instalace KAMP

KAMP se instaluje jako samostatný projekt.

Aktuální instalační postup se může časem změnit, proto před instalací vždy zkontroluj README projektu KAMP a nepoužívej několik let starý příkaz z náhodného videa.

Obecný princip je:

1. stáhnout/klonovat KAMP do Raspberry Pi,
2. zpřístupnit jeho konfigurační soubory Klipperu,
3. zahrnout potřebný KAMP config do konfigurace tiskárny,
4. povolit požadované funkce,
5. restartovat Klipper a zkontrolovat chyby.

> [!NOTE]
> Do základního `printer.cfg` tohoto projektu KAMP záměrně nedáváme. Základní konfigurace musí fungovat i bez externího projektu.

## 5. Instalace souboru nestačí – Klipper ho musí také načíst

Tohle je důležitý princip celé konfigurace Klipperu.

`printer.cfg` je hlavní vstupní konfigurační soubor. Nemusíš do něj nacpat všechny stovky řádků maker a doplňků, ale musí z něj vést cesta ke všemu, co má Klipper používat.

Samostatný soubor můžeš načíst například:

```ini
[include macros.cfg]
[include KAMP_Settings.cfg]
```

A některé funkce se povolují vlastní sekcí:

```ini
[exclude_object]
```

Typický config s doplňky tedy může obsahovat například:

```ini
[include mainsail.cfg]
[include macros.cfg]
[include KAMP_Settings.cfg]

[exclude_object]
```

> [!IMPORTANT]
> Řádky musí být **aktivní, tedy bez `#` na začátku**. `#` je komentář:
>
> ```ini
> # [include KAMP_Settings.cfg]
> ```
>
> a Klipper takový řádek vůbec nenačte.

Po instalaci každého doplňku proto vždy zkontroluj dvě věci:

1. že jeho soubory opravdu existují na správném místě,
2. že jsou potřebné soubory/sekce aktivně zahrnuté do konfigurace.

Stejný princip platí pro `macros.cfg`, ADXL/Input Shaper config nebo jiné vlastní rozdělené `.cfg` soubory.

### Pozor na `[gcode_macro BED_MESH_CALIBRATE]`

Na konkrétní starší nebo vlastní KAMP konfiguraci můžeš narazit i na přepsání standardního příkazu pomocí makra `[gcode_macro BED_MESH_CALIBRATE]`.

**Nepřidávej tento blok automaticky jen proto, že byl v jiném configu.** Záleží na verzi a způsobu instalace KAMP. Použij pouze sekce a include soubory, které vyžaduje právě instalovaná verze projektu.

Po každé změně použij **Firmware Restart**. Pokud include odkazuje na neexistující soubor nebo je konfigurace chybná, Klipper to při načítání oznámí.

## 6. Proč nepřepisovat celý printer.cfg

Na internetu můžeš najít hotový config obsahující například:

```ini
[include KAMP_Settings.cfg]
[exclude_object]
```

a několik maker.

Nekopíruj kvůli tomu celý cizí `printer.cfg`.

Přidej pouze části, kterým rozumíš a které potřebuješ.

Když KAMP později odstraníš, základní konfigurace tiskárny musí zůstat přehledná.

## 7. Základní PRINT_START bez KAMP

V našem základním configu používáme princip:

```text
G28
BED_MESH_CLEAR
BED_MESH_CALIBRATE
```

To je naše bezpečná výchozí varianta.

Nejdřív musí fungovat právě ona.

Teprve potom má smysl nahradit klasické měření adaptivním postupem.

## 8. Adaptivní mesh

Při správné instalaci a nastavení KAMP se oblast meshe přizpůsobí hranicím objektu nebo objektů v aktuálním G-code.

Příklad:

Tiskneš malou součástku přibližně uprostřed podložky.

Klasický mesh může změřit například celou oblast:

```text
35,10 → 220,220
```

Adaptivní mesh se může soustředit pouze na oblast kolem konkrétního modelu.

Konkrétní hranice ale neurčujeme ručně – vznikají z dat aktuálního tisku.

## 9. Bezpečnostní pravidla z Bed Meshe stále platí

KAMP nemění fyziku tiskárny.

Stále musí platit:

- BLTouch se může dostat na všechny požadované měřicí body,
- sonda nesmí vyjet mimo bed,
- toolhead nesmí narazit do limitů,
- kabeláž musí mít dostatečný rozsah,
- Z-offset musí být správně nastavený.

Pokud základní `mesh_min`, `mesh_max` nebo probe offsety nejsou správně, KAMP není řešení.

## 10. Purge line

Před samotným tiskem chceme mít trysku naplněnou materiálem.

Klasická startovací čára bývá napevno umístěná například u kraje podložky.

KAMP umí vytvořit purge line s ohledem na polohu objektu.

Výhoda je hlavně v tom, že purge může být součástí adaptivní přípravy konkrétního tisku místo pevně zakódované čáry na jednom místě.

## 11. Pozor na dvojitou purge line

Pokud používáš purge z KAMP, zkontroluj start G-code sliceru.

Nechceme:

1. purge line ze sliceru,
2. a potom další purge line z KAMP.

Výsledkem je jen zbytečný materiál a čas.

Měj jedno jasné místo, které purge řídí.

## 12. KAMP a max_extrude_cross_section

U některých purge maker může Klipper odmítnout extruzi, pokud požadovaný průřez překročí bezpečnostní limit.

Můžeš proto narazit na parametr:

```ini
max_extrude_cross_section
```

> [!CAUTION]
> Nezvyšuj bezpečnostní limit naslepo jen proto, aby zmizela chybová hláška.

Nejdřív zjisti, proč makro požaduje tak velkou extruzi a zda odpovídá konfiguraci purge, průměru trysky a konkrétní instalaci KAMP.

Bezpečnostní kontrola Klipperu má důvod.

## 13. Co když KAMP nefunguje

Vrať se k jednoduchému řetězci:

```text
G28
BED_MESH_CLEAR
BED_MESH_CALIBRATE
```

Pokud klasický mesh funguje a KAMP ne, základní tiskárna pravděpodobně není první místo, kde hledat problém.

Zkontroluj:

- instalaci KAMP,
- include soubory,
- `[exclude_object]`,
- slicer,
- zpracování objektů v G-code,
- startovací makro.

Tím si výrazně zjednodušíš hledání chyby.

## 14. KAMP není povinný

Pokud ti nevadí několik sekund nebo minut navíc před tiskem, klasický Bed Mesh je naprosto použitelný.

KAMP přidává pohodlí a efektivnější přípravu tisku.

Není to podmínka kvalitního výtisku.

Právě proto ho v tomto projektu držíme jako **volitelnou vrstvu nad už funkční tiskárnou**.

## Co máš mít po této kapitole hotové

- základní Bed Mesh stále funguje bez KAMP,
- rozumíš, proč KAMP potřebuje informace o objektech,
- máš povolené `[exclude_object]`,
- adaptivní mesh měří oblast aktuálního tisku,
- purge line se nespouští dvakrát,
- víš, jak se vrátit ke klasickému `BED_MESH_CALIBRATE`,
- KAMP není pevnou závislostí základního `printer.cfg`.

Další krok:

➡️ **11 – PRINT_START, PRINT_END a komunikace se slicerem**
