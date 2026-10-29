# Analýza skrývání bloků v CRM144.pdf

Soubor [CRM144.pdf](./CRM144.pdf) je dynamický XFA formulář vytvořený v Adobe LiveCycle Designer 11. Skrývání bloků řeší JavaScript, který se spouští jednou při události `initialize` (`runAt="both"`). Podle hodnot ze vstupních dat nastaví vlastnost `presence` na `visible` nebo `hidden`. Všechny řízené bloky jsou v šabloně předem nastavené jako `presence="hidden"`, takže viditelný blok se objeví jen tehdy, když podmínka platí.

## 1. Řídicí vstupy

Všechna pole jsou svázaná s daty přes `bind match="global"`.

| Pole | Umístění | Co řídí |
|---|---|---|
| `TEMPL_CODE` | `form.Page1.Varianta` (invisible) | výběr šablony těla dokumentu: 314, 315 nebo 316 |
| `TEXT_VAR` | `form.Page1.Varianta` (invisible) | výběr textového bloku uvnitř šablony |
| `MIDD_INVISIBLE` | patička, 3. nepojmenovaný subform (invisible) | zobrazení řádku o zprostředkovateli |

Formulářové proměnné (`form/variables`) jsou: `templ314=314`, `templ315=315`, `templ316=316`, `sf39=039`, `sf41=041`, `sf44=044`, `sf46=046`, `sf47=047`, `sf50=050`, `sf54=054` a `TRUE=X`.

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

Každý blok varianty obsahuje pole `BODY1`, které nese rich text z dat. V `TEMP_314` jsou navíc pole `PREMEXT` a `PREMLABEL`, která jsou ale staticky skrytá a nikdy se nezobrazí.

### Patička – zprostředkovatel

Subform s textem „Zpracováno zprostředkovatelem {MIDDLEMAN}, jednajícím pro innogy Energie…“ se skryje, když platí:

```javascript
xfa.resolveNode("#subform[2].MIDD_INVISIBLE").rawValue == TRUE.value
```

Tedy když `MIDD_INVISIBLE = "X"`.

## 3. Vyhodnocení pro testovací data [xmlIN.xml](./xmlIN.xml)

Data obsahují `TEMPL_CODE=315`, `TEXT_VAR=041` a `MIDD_INVISIBLE=X`. Výsledek:

- Zobrazí se `TEMP_315`, v něm `sf_BODY`, `sf_41` a `sf_BODY2`.
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
