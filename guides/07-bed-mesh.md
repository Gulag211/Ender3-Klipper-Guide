# 07 – Bed Mesh: změření podložky

Z-offset už máme nastavený. Teď může Klipper pomocí BLTouch změřit tvar podložky a při tisku drobné výškové nerovnosti kompenzovat pohybem osy Z.

Tomu slouží **Bed Mesh**.

> [!IMPORTANT]
> Bed Mesh nenahrazuje mechanicky správně sestavenou tiskárnu. Pokud je podložka výrazně nakřivo, něco je volné nebo je mechanika špatně sestavená, nejdřív oprav mechanický problém.

## 1. Náš základní config

V tomto projektu používáme jako jednoduchý výchozí příklad:

```ini
[bed_mesh]
speed: 150
horizontal_move_z: 7
mesh_min: 35,10
mesh_max: 220,220
probe_count: 3,3
```

Tyto hodnoty **nejsou univerzální pro každý Ender 3**.

Zvlášť `mesh_min` a `mesh_max` musí odpovídat:

- rozměrům podložky,
- rozsahu pohybu tiskárny,
- poloze sondy vůči trysce,
- konkrétnímu držáku BLTouch.

## 2. Tryska a BLTouch nejsou na stejném místě

V našem příkladu máme:

```ini
[bltouch]
x_offset: 30
y_offset: 2
```

To znamená, že měřicí bod sondy je vůči trysce posunutý.

Když tedy Klipper přesouvá **sondu** na místo měření, musí současně postavit trysku na takovou souřadnici, aby se BLTouch dostal na požadovaný bod.

Právě proto nemůžeš bez přemýšlení napsat:

```ini
mesh_min: 0,0
mesh_max: 235,235
```

jen proto, že má tvoje podložka přibližně 235 × 235 mm.

Sonda by se na některé požadované body nemusela fyzicky dostat.

## 3. Co znamená mesh_min a mesh_max

Například:

```ini
mesh_min: 35,10
mesh_max: 220,220
```

určuje oblast podložky, ve které chceme sondou měřit.

Velmi důležité:

**`mesh_min` a `mesh_max` popisují souřadnice měřicí sondy na podložce.**

Klipper při pohybu automaticky zohlední `x_offset` a `y_offset` sondy.

Proto při nastavování oblasti musíš kontrolovat dvě věci současně:

1. BLTouch musí zůstat nad podložkou.
2. Tryska/toolhead se musí stále vejít do povoleného rozsahu pohybu os.

## 4. Nejdřív zkontroluj rohy ručně

Než spustíš automatické měření celé sítě, je rozumné ověřit krajní body.

Zahomuj tiskárnu:

```text
G28
```

Potom zvedni Z do bezpečné výšky, například:

```text
G90
G1 Z10 F600
```

Teď můžeš postupně kontrolovat polohy odpovídající plánované oblasti měření.

Nesoustřeď se jen na polohu trysky. **Dívej se hlavně na BLTouch.**

Sonda musí být v každém měřicím bodě bezpečně nad podložkou.

> [!CAUTION]
> Pokud by BLTouch při měření sjel mimo podložku, Z bude pokračovat dolů a sonda nemá co sepnout. To může skončit nárazem trysky do bedu.

## 5. Co znamená probe_count

V našem příkladu:

```ini
probe_count: 3,3
```

Klipper vytvoří síť měřicích bodů:

```text
3 × 3 = 9 bodů
```

Pro první zprovoznění je to jednoduché a rychlé.

Vyšší počet bodů může vytvořit podrobnější mapu, ale zároveň:

- měření trvá déle,
- více bodů automaticky neznamená lepší mechaniku,
- nemá smysl maskovat velké mechanické problémy extrémně hustou sítí.

Nejdřív tedy tiskárnu rozumně mechanicky nastav a až potom řeš jemnější mesh.

## 6. Co znamená horizontal_move_z

Máme:

```ini
horizontal_move_z: 7
```

To určuje výšku Z, ve které Klipper přejíždí mezi jednotlivými měřicími body.

Sedm milimetrů nám dává při prvním nastavování rozumnou rezervu nad podložkou.

Pokud máš na podložce například klipy nebo jiné překážky, musíš při návrhu měřicí oblasti myslet i na ně.

BLTouch ani tryska do nich nesmí při přesunu narazit.

## 7. První BED_MESH_CALIBRATE

Jakmile máš ověřeno, že je oblast bezpečná, spusť:

```text
G28
BED_MESH_CALIBRATE
```

Tiskárna postupně změří jednotlivé body podložky.

U prvního měření zůstaň u tiskárny a sleduj hlavně:

- zda BLTouch v každém bodě zůstává nad bedem,
- zda toolhead nenaráží do limitů,
- zda sonda spolehlivě měří,
- zda kabeláž nikde netáhne nebo nezachytává.

Pokud se něco nezdá, měření zastav.

## 8. Co vlastně Bed Mesh dělá

Výsledkem není „srovnání podložky“.

Klipper si vytvoří matematickou mapu jejího povrchu.

Při tisku potom velmi jemně mění výšku Z tak, aby tryska sledovala naměřené odchylky podložky.

Proto je pořád důležité mít mechaniku tiskárny co nejlépe nastavenou.

Bed Mesh má kompenzovat menší nerovnosti, ne zachraňovat špatně sestavenou tiskárnu.

## 9. Ukládat mesh, nebo měřit před tiskem?

V našem základním `PRINT_START` používáme:

```text
BED_MESH_CLEAR
BED_MESH_CALIBRATE
```

To znamená, že před tiskem starou mapu zahodíme a vytvoříme novou.

Pro začátečnický základ je to snadno pochopitelné a nemusíme řešit, zda je uložený profil ještě aktuální.

Nevýhodou je, že se před každým tiskem měří celá síť.

Později si ukážeme **KAMP**, který může měřit pouze oblast podložky potřebnou pro konkrétní výtisk.

## 10. Teplota podložky má význam

Podložka může při zahřátí mírně změnit tvar.

Proto má náš `PRINT_START` logiku, ve které se nejdřív nastaví a vyčká teplota bedu a teprve potom se provádí homing a Bed Mesh.

Pro běžné používání tedy dává smysl měřit podložku ve stavu podobném tomu, ve kterém bude probíhat tisk.

## 11. Když mapa vypadá špatně

Pokud Bed Mesh ukazuje velké rozdíly, neřeš to automaticky změnou počtu bodů.

Zkontroluj například:

- dotažení a vůle koleček/vedení,
- uchycení podložky,
- pružiny nebo distanční sloupky,
- vůli osy Z,
- držák BLTouch,
- zda se sonda nehýbe,
- zda nejsou na podložce nečistoty,
- zda je mechanika tiskárny správně sestavená.

Podezřelá mapa může být informace o skutečném mechanickém problému.

## Co máš mít po této kapitole hotové

- rozumíš rozdílu mezi polohou trysky a sondy,
- víš, proč `mesh_min` a `mesh_max` nejsou univerzální,
- BLTouch zůstává ve všech měřicích bodech nad podložkou,
- rozumíš `probe_count` a `horizontal_move_z`,
- `BED_MESH_CALIBRATE` proběhne bezpečně,
- víš, že Bed Mesh kompenzuje menší nerovnosti a neopravuje mechaniku.

Další krok:

➡️ **08 – Pressure Advance**
