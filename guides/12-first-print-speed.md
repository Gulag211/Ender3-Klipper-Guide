# 12 – První testovací tisk a bezpečné zvyšování rychlosti

Tiskárna je nakonfigurovaná, zkalibrovaná a makra fungují. Teď přichází chvíle, kdy je lákavé napsat do sliceru 300 mm/s a zjistit, co se stane.

Nedělej to. 🙂

Klipper umožní z Enderu dostat výrazně víc než původní firmware, ale firmware sám o sobě neodstraní fyzikální limity tiskárny.

## 1. Nejdřív vytiskni něco pomalu

První testovací tisk není speed bench.

Použij známý jednoduchý model a rozumné rychlosti, například v řádu:

```text
40–60 mm/s
```

Cílem prvního tisku je ověřit celý proces:

- startovací makro,
- první vrstvu,
- extruzi,
- chlazení,
- pohyb os,
- Bed Mesh,
- ukončení a parkování.

Pokud něco nefunguje při 50 mm/s, rychlost 200 mm/s to neopraví.

## 2. Sleduj první vrstvu

U prvního tisku zůstaň u tiskárny.

Sleduj, zda:

- tryska není příliš vysoko,
- tryska není příliš nízko,
- filament se spolehlivě přichytává,
- jednotlivé čáry první vrstvy správně navazují,
- BLTouch/mesh nedělá něco podezřelého.

Pokud je potřeba drobné doladění Z, udělej ho opatrně.

Velká korekce znamená, že je lepší vrátit se ke kalibraci Z-offsetu.

## 3. Rychlost a akcelerace nejsou totéž

Rychlost říká, jak rychle se má osa pohybovat.

Akcelerace říká, jak rychle se na tuto rychlost dostane.

Můžeš tedy nastavit:

```text
300 mm/s
```

ale u krátké stěny se tiskárna na tuto rychlost vůbec nemusí dostat.

Naopak vysoká akcelerace může výrazně zvýšit mechanické namáhání i při nižší maximální rychlosti.

Proto při ladění nesleduj pouze jedno číslo ze sliceru.

## 4. Základní limity v printer.cfg

V našem veřejném configu začínáme konzervativně:

```ini
[printer]
kinematics: cartesian
max_velocity: 200
max_accel: 2000
max_z_velocity: 15
max_z_accel: 100
```

To nejsou hodnoty, které definují maximum každého Enderu 3.

Jsou to rozumné výchozí limity pro zprovoznění.

Jakmile je tiskárna mechanicky i elektronicky ověřená, můžeš je zvyšovat postupně.

## 5. Zvyšuj vždy jednu věc

Pokud současně změníš:

- rychlost,
- akceleraci,
- teplotu,
- flow,
- Pressure Advance,
- Input Shaper,

a tisk se zhorší, nebudeš vědět proč.

Měň jednu hlavní věc, vytiskni test a porovnej výsledek.

To je pomalejší než náhodné přepisování deseti hodnot, ale ve výsledku se k dobrému nastavení dostaneš rychleji.

## 6. Mechanické limity

Při zvyšování akcelerace a rychlosti sleduj tiskárnu.

Varovné příznaky mohou být například:

- ztracené kroky,
- posunuté vrstvy,
- silné vibrace,
- nezvyklé rány,
- přeskakování řemenu,
- přehřívání motorů nebo driverů,
- zhoršující se kvalita povrchu.

Když se něco takového objeví, neřeš to automaticky ještě vyšším proudem motoru.

Najdi příčinu.

## 7. Hotend má limit průtoku

Jedna z nejdůležitějších věcí při rychlém tisku:

**rychlost pohybu není totéž jako množství plastu, které hotend zvládne roztavit.**

Přibližný objemový průtok je:

```text
průtok = šířka čáry × výška vrstvy × rychlost
```

Například:

```text
0.45 mm × 0.20 mm × 100 mm/s = 9 mm³/s
```

Při 200 mm/s:

```text
0.45 × 0.20 × 200 = 18 mm³/s
```

Hotend tedy musí za stejný čas roztavit dvojnásobné množství plastu.

Pokud to nezvládne, může se objevit podextruze i přesto, že mechanika tiskárny zvládá danou rychlost bez problému.

## 8. Maximum volumetric speed ve sliceru

Moderní slicery umožňují nastavit maximální objemový průtok filamentu.

To je velmi užitečný limit.

Místo toho, aby tiskárna slepě jela požadovaných 250 mm/s i v místě s velkým průtokem, slicer může rychlost snížit tak, aby nepřekročil schopnosti hotendu a materiálu.

Tuto hodnotu ale opět nemá smysl kopírovat z internetu.

Záleží na:

- hotendu,
- trysce,
- materiálu,
- teplotě,
- konkrétním filamentu.

## 9. Chlazení může být další limit

Hotend může zvládnout materiál roztavit, ale výtisk ho ještě musí zvládnout ochladit.

Při vysoké rychlosti může být problém například:

- převis,
- bridge,
- malá vrstva,
- ostrý detail.

Proto rychlejší tisk často vyžaduje také lepší chlazení výtisku.

To ale neznamená, že má být ventilátor vždy na 100 %. Záleží na materiálu a modelu.

## 10. Input Shaper není turbo tlačítko

Input Shaper pomáhá omezovat projevy rezonancí.

Neznamená to:

```text
Input Shaper hotový = tiskárna může libovolnou rychlost
```

Stále existují limity:

- motorů,
- řemenů,
- rámu,
- extruderu,
- hotendu,
- chlazení,
- konkrétního modelu.

## 11. Pressure Advance není turbo tlačítko

Stejně tak PA pomáhá řídit tlak v trysce při změnách rychlosti.

Nezvýší maximální průtok hotendu a neopraví prokluzující extruder.

PA a Input Shaper pomáhají tiskárně pracovat přesněji při dynamickém pohybu. Neodstraňují ostatní limity.

## 12. Doporučený postup zvyšování výkonu

Začni například kolem:

```text
50 mm/s
2000 mm/s²
```

Ověř kvalitu a mechaniku.

Potom postupně testuj vyšší rychlost a akceleraci.

Například nejdřív zvyšuj po rozumných krocích, ne skokem na několikanásobek.

Při každé změně sleduj:

- kvalitu,
- zvuk,
- vibrace,
- spolehlivost,
- teploty motorů,
- extruzi.

Nejde o to najít číslo, při kterém tiskárna ještě jednou přežije test.

Hledáme nastavení, při kterém může **spolehlivě tisknout opakovaně**.

## 13. Travel může být rychlejší než tisk

Pohyb bez extruze neomezuje průtok hotendu.

Proto může být travel výrazně rychlejší než samotný tisk.

Pořád ho ale omezuje mechanika a akcelerace tiskárny.

Vysoký travel není užitečný, pokud kvůli němu tiskárna ztrácí kroky nebo se celý rám rozkmitá.

## 14. Číslo ze sliceru není skutečná průměrná rychlost

Když nastavíš ve sliceru například:

```text
200 mm/s
```

neznamená to, že celý model bude tisknutý 200 mm/s.

Slicer může zpomalovat kvůli:

- vnějším stěnám,
- malým perimetrům,
- převisům,
- minimálním časům vrstvy,
- maximálnímu objemovému průtoku,
- akceleračním limitům.

Proto neposuzuj výkon tiskárny jen podle jednoho čísla v profilu.

## 15. Co je vlastně dobrý výsledek

Cílem není nejvyšší číslo, které můžeš napsat do sliceru.

Dobrý profil je takový, který:

- tiskne kvalitně,
- je opakovatelný,
- neztrácí kroky,
- netrápí mechaniku,
- nepřekračuje průtok hotendu,
- funguje bez neustálého dozoru.

Rychlost je až jeden z výsledků správně naladěného systému.

## 16. Klipper sám z Enderu rychlou tiskárnu neudělá

Tohle je důležité i při sledování videí rychlých upravených Enderů.

Výsledek může ovlivňovat například:

- jiný extruder,
- výkonnější hotend,
- lehčí toolhead,
- lepší chlazení,
- úpravy mechaniky,
- jiné motory,
- správně naladěný Input Shaper a PA.

Proto video rychlého Enderu ber jako ukázku toho, čeho lze s úpravami dosáhnout – ne jako důkaz, že stejné hodnoty jsou bezpečné pro každý Ender 3.

## Co máš mít po této kapitole hotové

- první pomalý testovací tisk proběhl správně,
- první vrstva je použitelná,
- rozumíš rozdílu mezi rychlostí a akcelerací,
- víš, že hotend omezuje objemový průtok,
- zvyšuješ parametry postupně,
- nesnažíš se mechanický problém řešit firmwarem,
- hledáš spolehlivé nastavení, ne rekordní číslo.

---

Tím máme základní cestu od čisté instalace až po zkalibrovanou a připravenou tiskárnu.

Další rozšíření můžeme přidávat samostatně, aniž bychom rozbili jednoduchý základ.
