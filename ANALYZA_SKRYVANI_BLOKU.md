# Analýza skrývání bloků v CRM144.pdf

Soubor [CRM144.pdf](./CRM144.pdf) je dynamický XFA formulář vytvořený v Adobe LiveCycle Designer 11. Skrývání bloků řeší JavaScript, který se spouští jednou při události `initialize` (`runAt="both"`). Podle hodnot ze vstupních dat nastaví vlastnost `presence` na `visible` nebo `hidden`. Všechny řízené bloky jsou v šabloně předem nastavené jako `presence="hidden"`, takže viditelný blok se objeví jen tehdy, když podmínka platí.

## 1. Řídicí vstupy

Všechna pole jsou svázaná s daty přes `bind match="global"`.

| Pole | Umístění | Co řídí |
|---|---|---|
| `TEMPL_CODE` | `form.Page1.Varianta` (invisible) | výběr šablony těla dokumentu: 314, 315 nebo 316 |
| `TEXT_VAR` | `form.Page1.Varianta` (invisible) | výběr textového bloku uvnitř šablony |
| `MIDD_INVISIBLE` | patička, 3. nepojmenovaný subform (invisible) | zobrazení řádku o zprostředkovateli |

### Formulářové proměnné

Proměnné najdete v Designeru přes File → Form Properties → Variables (v XML šablony jako `form/variables`). Jsou to textové konstanty a jejich obsah je tento:

| Proměnná | Hodnota | Použití |
|---|---|---|
| `templ314`, `templ315`, `templ316` | `314`, `315`, `316` | výběr šablony podle `TEMPL_CODE` |
| `sf39`, `sf41`, `sf44`, `sf46`, `sf47`, `sf50`, `sf54` | `039`, `041`, `044`, `046`, `047`, `050`, `054` | výběr bloku varianty podle `TEXT_VAR` |
| `TRUE` | `X` | skrytí řádku o zprostředkovateli podle `MIDD_INVISIBLE` |

Proměnné platí pro celý formulář, takže na ně může odkazovat skript kteréhokoli bloku. Text proměnné se čte přes `.value`. Příklad skriptu bloku `sf_39`:

```javascript
if (Varianta.TEXT_VAR.rawValue == sf39.value)   // např. "041" == "039" → false
    {this.presence = "visible";}
else
    {this.presence = "hidden";}
```

Na co si dát pozor:

- **Rozhoduje hodnota proměnné, ne její název.** Kdyby proměnná `sf39` obsahovala `041`, blok `sf_39` by se zobrazil pro kód 041.
- **Ne všechny kódy mají proměnnou.** Proměnnou používají jen kódy 039, 041, 044, 046, 047, 050 a 054. Kódy 040, 042, 043, 045, 048, 049, 055 a 056 jsou zapsané přímo ve skriptu jako text, například `== '040'`. Výsledek je stejný, jen se tyto kódy přes proměnné změnit nedají.
- **Jedna proměnná platí ve všech šablonách.** Například změna `sf41` se projeví u bloku `sf_41` v `TEMP_314`, `TEMP_315` i `TEMP_316` současně.
- **Porovnává se text.** Hodnota v datech musí přesně odpovídat hodnotě proměnné, včetně úvodní nuly.
- **Pořadí proměnných nemá vliv na zobrazení.** Pořadí v seznamu proměnných (`templ314`, `sf46`, `sf44`, …) s pořadím bloků v dokumentu nesouvisí. Pořadí bloků určuje jen jejich pořadí v šabloně (viz kapitola 2).

### Výchozí hodnoty polí v šabloně

Výchozí hodnota se použije jen tehdy, když ji data nedodají:

| Pole | Výchozí hodnota | Poznámka |
|---|---|---|
| `TEMPL_CODE` v záhlaví | `CRM144` | popisek „innogy formulář“, netiskne se (`relevant="-print"`) |
| `ID_form` v patičce | `CRM113` | netiskne se; neodpovídá názvu formuláře CRM144 |
| `Varianta.TEMPL_CODE`, `TEXT_VAR`, `MIDD_INVISIBLE` | žádná | |
| `BODY`, `BODY1`, `BODY2` | žádná | |

Co z toho vyplývá:

- **Dvě pole `TEMPL_CODE`:** pole v záhlaví i pole ve `Varianta` jsou napojená globálně, takže sdílejí stejnou hodnotu z dat. V záhlaví se proto na obrazovce místo „CRM144“ ukáže například „315“ (netiskne se).
- **Bez dat je tělo prázdné:** žádná ze šablon se nezobrazí. Výchozí hodnota „CRM144“ se neshoduje s 314, 315 ani 316, i kdyby se k `Varianta.TEMPL_CODE` dostala přes globální vazbu (v Designeru neověřeno).
- **Bez `MIDD_INVISIBLE` se řádek o zprostředkovateli zobrazí:** prázdná hodnota se nerovná „X“.
## 2. Hierarchie skrývání (2 úrovně + patička)

### Úroveň 1 – šablona

Podmínka: `Varianta.TEMPL_CODE.rawValue == templNNN.value`

- `TEMP_314`, `TEMP_315` a `TEMP_316` se navzájem vylučují.
- Každá šablona má vždy viditelné bloky `sf_BODY` (pole `BODY`) a `sf_BODY2` (pole `BODY2`). Mezi nimi jsou bloky variant.

### Úroveň 2 – blok varianty

Podmínka: `Varianta.TEXT_VAR.rawValue == '0NN'`

| Kód TEXT_VAR | TEMP_314 | TEMP_315 | TEMP_316 |
|---|:-:|:-:|:-:|
| 039 | ✅ | ✅ | ✅ |
| 040 | ✅ | ✅ | – |
| 041 | ✅ | ✅ | ✅ |
| 042 | ✅ | ✅ | ✅ |
| 043 | ✅ | ✅ | – |
| 044 | ✅ | ✅ | – |
| 045 | ✅ | ✅ | – |
| 046 | ✅ | – | – |
| 047 | – | ✅ | – |
| 048 | – | – | ✅ |
| 049 | – | – | ✅ |
| 050 | – | – | ✅ |
| 054 | ✅ | ✅ | – |
| 055 | – | – | ✅ |
| 056 | – | ✅ | – |

### Pořadí bloků v šablonách

Všechny tři šablony (`TEMP_314`, `TEMP_315`, `TEMP_316`) mají `layout="tb"`. Bloky se proto skládají shora dolů v tom pořadí, v jakém jsou definované v šabloně, a skryté bloky (`presence="hidden"`) nezabírají místo. Skript každého bloku `sf_NN` testuje kód `0NN`, takže název bloku odpovídá kódu `TEXT_VAR`.

| Pozice | TEMP_314 | TEMP_315 | TEMP_316 |
|:-:|---|---|---|
| 1 | `sf_BODY` | `sf_BODY` | `sf_BODY` |
| 2 | `sf_39` (039) | `sf_39` (039) | `sf_48` (048) |
| 3 | `sf_40` (040) | `sf_40` (040) | `sf_41` (041) |
| 4 | `sf_54` (054) | `sf_54` (054) | `sf_42` (042) |
| 5 | `sf_41` (041) | `sf_41` (041) | `sf_39` (039) |
| 6 | `sf_42` (042) | `sf_42` (042) | `sf_49` (049) |
| 7 | `sf_43` (043) | `sf_43` (043) | `sf_50` (050) |
| 8 | `sf_44` (044) | `sf_44` (044) | `sf_55` (055) |
| 9 | `sf_45` (045) | `sf_56` (056) | `sf_BODY2` |
| 10 | `sf_46` (046) | `sf_45` (045) | |
| 11 | `sf_BODY2` | `sf_47` (047) | |
| 12 | | `sf_BODY2` | |

Pozorování:

- Pořadí neodpovídá číselnému řazení kódů. Blok `sf_54` je ve `TEMP_314` a `TEMP_315` hned za `sf_40`, `sf_56` je v `TEMP_315` před `sf_45` a `TEMP_316` začíná blokem `sf_48`, za kterým teprve následují `sf_41`, `sf_42` a `sf_39`.
- `TEMP_314` a `TEMP_315` mají společné pořadí 039, 040, 054, 041, 042, 043, 044. Pak se liší: v `TEMP_314` následují 045 a 046, v `TEMP_315` následují 056, 045 a 047.
- `TEXT_VAR` má vždy jen jednu hodnotu, proto je viditelný nejvýše jeden blok varianty. Výsledný dokument má tedy vždy pořadí `sf_BODY` → `sf_NN` → `sf_BODY2` a pořadí uvnitř šablony se projeví až tehdy, kdyby se v budoucnu zobrazovalo více bloků najednou.

V `TEMP_314` jsou navíc pole `PREMEXT` a `PREMLABEL`, která jsou ale staticky skrytá a nikdy se nezobrazí.

### Pole `BODY1` – umístění a zobrazení

Pole `BODY1` je v každém bloku varianty `sf_NN` ve všech třech šablonách, celkem 30×:

```
TEMP_31x
├─ sf_BODY   → BODY
├─ sf_NN     → BODY1   (v každém bloku varianty)
└─ sf_BODY2  → BODY2
```

Všechna pole `BODY`, `BODY1` a `BODY2` jsou na data napojená přes `bind match="global"`. Všech 30 polí `BODY1` proto čte jeden a tentýž uzel `<BODY1>` z dat. Totéž platí pro `BODY` a `BODY2`. Formulář tedy neobsahuje 15 různých textů, ale jeden text `BODY1`, který přijde z dat. Kód `TEXT_VAR` určuje jen to, ve kterém bloku a s jakou výškou se zobrazí.

Jak se `BODY1` zobrazuje:

- **Formát:** pole je víceřádkové (`textEdit multiLine="1"`) a hodnotu bere jako HTML (`exData contentType="text/html"`). Data posílají XHTML (`<b>`, `<br/>`, `<p>`), takže se tučné písmo i zalomení řádků vykreslí.
- **Písmo a šířka:** Arial 8 pt, šířka 165 mm. Popisek obsahuje jen mezery, takže se nevykreslí.
- **Výška se liší podle bloku:**

| Šablona | Pole roste s textem (`minH` 10 mm) | Pevná výška 20 mm | Pevná výška 10 mm |
|---|---|---|---|
| TEMP_314 | sf_40, sf_42, sf_43, sf_45 | sf_39, sf_41, sf_46, sf_54 | sf_44 |
| TEMP_315 | – | sf_39, sf_41, sf_47, sf_54 | sf_40, sf_42, sf_43, sf_44, sf_45, sf_56 |
| TEMP_316 | – | sf_39, sf_41 | sf_42, sf_48, sf_49, sf_50, sf_55 |

Kde je výška pevná, delší text se ořízne.

### Patička – zprostředkovatel

Subform s textem „Zpracováno zprostředkovatelem {MIDDLEMAN}, jednajícím pro innogy Energie…“ se skryje, když platí:

```javascript
xfa.resolveNode("#subform[2].MIDD_INVISIBLE").rawValue == TRUE.value
```

Tedy když `MIDD_INVISIBLE = "X"`.

## 3. Vyhodnocení pro testovací data [xmlIN.xml](./xmlIN.xml)

Data obsahují `TEMPL_CODE=315`, `TEXT_VAR=041` a `MIDD_INVISIBLE=X`. Výsledek:

- Zobrazí se `TEMP_315`, v něm `sf_BODY` (25 mm), `sf_41` (20 mm) a `sf_BODY2` (40 mm). V `sf_41.BODY1` je text „Dovolujeme si Vás však informovat, že podmínky pro ukončení Smlouvy… nebyly splněny z důvodu chybné plné moci.“
- Všechny ostatní bloky se skryjí, včetně celých šablon `TEMP_314` a `TEMP_316`.
- Řádek o zprostředkovateli v patičce se skryje.

## 4. Zjištěná rizika a nedostatky

1. **Neplatná kombinace kódů nic nenahlásí.** Pokud kód `TEXT_VAR` v dané šabloně neexistuje (například 048 v šabloně 315), nezobrazí se žádný blok varianty a dokument vyjde bez varovné hlášky. Totéž platí pro `TEMPL_CODE` mimo 314/315/316: tělo dokumentu zůstane prázdné.
2. **Porovnání je přesná shoda řetězců.** Hodnota `"41"` nebo `" 041"` blok nezobrazí. Data musí kód poslat vždy jako tři číslice s úvodní nulou.
3. **Odkaz na pole přes pořadí subformu.** Výraz `#subform[2].MIDD_INVISIBLE` hledá pole podle pozice nepojmenovaného subformu v patičce. Když se subformy v patičce přeuspořádají nebo se nějaký přidá, podmínka přestane fungovat. Doporučuji subform pojmenovat a odkazovat se na něj jménem.
4. **Pevná výška může oříznout text.**
   - Bloky `sf_39`, `sf_41`, `sf_54`, `sf_46` a `sf_47` mají pevnou výšku 20 mm, `sf_44` a `sf_50` 10 mm.
   - V `TEMP_315` a `TEMP_316` mají pevnou výšku 10 mm i pole `BODY1` uvnitř řazených (flowed) bloků, zatímco `TEMP_314` tam používá `minH`, takže se pole může zvětšovat.
   - Pole `BODY` (25 mm) a `BODY2` (40 mm) mají pevnou výšku ve všech šablonách.
   - Delší text z dat se proto může oříznout. Pro testovací data se to týká bloku `TEMP_315.sf_41` (20 mm).
5. **Nejednotný zápis podmínek.** Část skriptů porovnává s proměnnou (`sf39.value`) a část s literálem (`'040'`). Funkčně to vychází nastejno, ale údržba je kvůli tomu náchylná k chybám.
6. **Duplicitní kód.** Logika je zkopírovaná do 30 skriptů. Šla by nahradit jedním skriptem na úrovni šablony, který by porovnal přípony názvů `sf_NN` s hodnotou `TEXT_VAR`.
7. **Nevyužitá pole.** Pole `PREMEXT` a `PREMLABEL` v `TEMP_314` jsou staticky skrytá, bez jakékoli logiky a nejsou vložená ani jako floating field. Ověřte, jestli je formulář opravdu potřebuje. Pole `SALESREGISTER1/2` jsou sice skrytá, ale používají se jako floating fields v textu „Zápis v OR“.
8. **Nesprávné ID formuláře.** Výchozí hodnota pole `ID_form` je `CRM113`, ale formulář je CRM144. Pravděpodobně jde o pozůstatek kopie z jiného formuláře.
