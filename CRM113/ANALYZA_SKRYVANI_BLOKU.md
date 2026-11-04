# Analýza skrývání bloků v CRM113.pdf

Soubor [CRM113.pdf](./CRM113.pdf) je dynamický XFA formulář vytvořený v Adobe LiveCycle Designer 11. Skrývání bloků řeší JavaScript, který se spouští jednou při události `initialize` (`runAt="both"`). Podle hodnot ze vstupních dat nastaví vlastnost `presence` na `visible` nebo `hidden`. Řízené šablony i bloky jsou předem nastavené jako `presence="hidden"` (výjimka viz kapitola 4), takže blok se objeví jen tehdy, když podmínka platí.

## 1. Řídicí vstupy

Všechna pole jsou svázaná s daty přes `bind match="global"`.

| Pole | Umístění | Co řídí |
|---|---|---|
| `TEMPL_CODE` | `form.Page1.Varianta` (invisible) | výběr šablony: 301, 303, 304, 308 nebo 705 |
| `TEXT_VAR` | `form.Page1.Varianta` (invisible) | výběr bloku varianty uvnitř šablony |
| `MIDD_INVISIBLE` | patička, 3. nepojmenovaný subform (invisible) | zobrazení řádku o zprostředkovateli |

### Formulářové proměnné

File → Form Properties → Variables (v XML `form/variables`):

| Proměnná | Hodnota | Použití ve skriptech |
|---|---|---|
| `exist_sml` | `001` | nepoužitá |
| `pohl_po_splat` | `002` | nepoužitá |
| `jiny_zak` | `003` | nepoužitá |
| `fin_zar` | `004` | nepoužitá |
| `neni_pl_sml` | `005` | nepoužitá |
| `po_lhute` | `006` | nepoužitá |
| `sml_v_prostor` | **prázdná** | nepoužitá (mělo by být `007`) |
| `neni_ZD` | `008` | nepoužitá |
| `nelze_apl_odst` | `009` | nepoužitá |
| `zam_vyp_obch` | `010` | nepoužitá |
| `zam_RDK_distr` | `011` | nepoužitá |
| `jiz_odst_innogy` | **`12`** (bez úvodní nuly) | nepoužitá (mělo by být `012`) |
| `templ308` | `308` | šablona `TEMP_308` |
| `TRUE` | `X` | patička, `MIDD_INVISIBLE` |

Na co si dát pozor:

- **Proměnné kódů variant se nepoužívají.** Všechny bloky porovnávají s textovým literálem, např. `== '009'`. Změna proměnné tedy nic neudělá.
- **Šablony se vybírají nejednotně.** Jen `TEMP_308` používá proměnnou `templ308`; ostatní šablony mají kód zapsaný ve skriptu (`== '301'`, `'303'`, `'304'`, `'705'`).
- **Dvě proměnné jsou chybné** (`sml_v_prostor` prázdná, `jiz_odst_innogy = 12`). Dnes to nevadí, ale kdyby je někdo začal ve skriptech používat, blok `012` by se nikdy nezobrazil a blok `sf_sml_v_prostor` by se zobrazil při prázdném `TEXT_VAR`.
- **Porovnává se text.** Kód musí přijít přesně jako tři číslice s úvodní nulou.

### Výchozí hodnoty polí v šabloně

| Pole | Výchozí hodnota | Poznámka |
|---|---|---|
| `TEMPL_CODE` v patičce | žádná | popisek „innogy formulář“, netiskne se |
| `ID_form` v patičce | `CRM113` | netiskne se; odpovídá názvu formuláře |
| `Varianta.TEMPL_CODE`, `TEXT_VAR`, `MIDD_INVISIBLE` | žádná | bez dat se nezobrazí žádná šablona, řádek o zprostředkovateli se zobrazí |

## 2. Hierarchie skrývání (2 úrovně + patička)

### Úroveň 1 – šablona

Podmínka: `Varianta.TEMPL_CODE.rawValue == 'NNN'` (u 308 `== templ308.value`). Šablony se navzájem vylučují.

| Šablona | Téma (podle textu) | Layout | Pevná část |
|---|---|---|---|
| `TEMP_301` | neakceptace nové smlouvy | `lr-tb`, `minH=120mm` | – |
| `TEMP_303` | neakceptace výpovědi (§ 11b / § 11o EZ) | `lr-tb` | – |
| `TEMP_304` | neakceptace odstoupení (§ 1829 OZ) | `lr-tb` | – |
| `TEMP_308` | nesouhlas s ukončením u předchozího dodavatele | `lr-tb` | pole `BODY` z dat (vždy viditelné) |
| `TEMP_705` | zamítnutí změny dodavatele, odstoupení a smluvní pokuta | `lr-tb` | – |

Vždy viditelné jsou `Page1` s hlavičkou a adresním blokem, `Varianta` (neviditelná pole) a `SIGNATURES` (statický obrázek podpisu).

### Úroveň 2 – blok varianty

Podmínka: `Varianta.TEXT_VAR.rawValue == '0NN'`

| Kód TEXT_VAR | TEMP_301 | TEMP_303 | TEMP_304 | TEMP_308 | TEMP_705 |
|---|:-:|:-:|:-:|:-:|:-:|
| 001 | ✅ `sf_exist_sml` | – | – | – | – |
| 002 | ✅ `sf_pohl_po_spl` | – | – | – | – |
| 003 | ✅ `sf_jiny_zak` | – | – | – | – |
| 004 | ✅ `sf_fin_zar` | – | – | – | – |
| 005 | – | – | – | ✅ `sf_neni_pl_sml` | – |
| 006 | – | ✅ `sf_po_lhute` | ✅ `sf_po_lhute` | – | – |
| 007 | – | ✅ `sf_sml_v_prostor` | ✅ `sf_sml_v_prostor` | ✅ `sf_sml_v_prostor` | – |
| 008 | – | ✅ `sf_neni_ZD` | – | – | – |
| 009 | – | – | ✅ `sf_nelze_apl_odst` | – | – |
| 010 | – | – | – | – | ✅ `zam_vyp_obch` |
| 011 | – | – | – | – | ✅ `zam_RDK_distr` |
| 012 | – | ✅ `sf_jiz_odst_innogy` | ✅ `sf_jiz_odst_innogy` | – | – |
| 023 | – | – | – | ✅ `sf_chyba_zakaznika` | – |

Na rozdíl od CRM144 **má každý blok vlastní, odlišný text** (statický `draw BODY`). Stejný kód ve dvou šablonách vede k jinému textu, např. `006` v `TEMP_303` mluví o výpovědi dle § 11b EZ, v `TEMP_304` o odstoupení dle § 1829 OZ.

### Pořadí bloků v šablonách

| Pozice | TEMP_301 | TEMP_303 | TEMP_304 | TEMP_308 | TEMP_705 |
|:-:|---|---|---|---|---|
| 1 | `sf_jiny_zak` (003) | `sf_jiz_odst_innogy` (012) | `sf_jiz_odst_innogy` (012) | `BODY` (data) | `zam_vyp_obch` (010) |
| 2 | `sf_exist_sml` (001) | `sf_po_lhute` (006) | `sf_po_lhute` (006) | `sf_sml_v_prostor` (007) | `zam_RDK_distr` (011) |
| 3 | `sf_fin_zar` (004) | `sf_sml_v_prostor` (007) | `sf_sml_v_prostor` (007) | `sf_neni_pl_sml` (005) | |
| 4 | `sf_pohl_po_spl` (002) | `sf_neni_ZD` (008) | `sf_nelze_apl_odst` (009) | `sf_chyba_zakaznika` (023) | |

Protože `TEXT_VAR` má jednu hodnotu, je viditelný nejvýše jeden blok a na pořadí v praxi nezáleží. Souřadnice `y` u bloků v `TEMP_304` (119.11 mm, 200.825 mm) se v plynulém layoutu `lr-tb` ignorují.

### Výšky textových bloků

| Blok | Výška textu `BODY` |
|---|---|
| většina bloků | `minH=30mm` – roste s textem |
| `TEMP_304.sf_sml_v_prostor` | `minH=119.11mm` – pravděpodobně omylem převzatá hodnota `y`; blok zabere min. 119 mm i při kratším textu |
| `TEMP_705.zam_RDK_distr` | pevná `h=98.649mm` – delší text (např. dlouhá adresa) se ořízne |
| `TEMP_308.BODY` (pole z dat) | pevná `h=20mm` |

### Patička – zprostředkovatel

Subform s textem „Zpracováno zprostředkovatelem {MIDDLEMAN}, jednajícím pro innogy Energie…“ se skryje, když:

```javascript
xfa.resolveNode("#subform[2].MIDD_INVISIBLE").rawValue == TRUE.value
```

Tedy když `MIDD_INVISIBLE = "X"`.

## 3. Vyhodnocení pro testovací data [CRM113_T304.xml](./CRM113_T304.xml)

Data obsahují `TEMPL_CODE=304`, `TEXT_VAR=009` a `MIDD_INVISIBLE=X`. Výsledek (ověřeno textem z [out_PDF.pdf](./out_PDF.pdf)):

- Zobrazí se `TEMP_304` a v něm jen `sf_nelze_apl_odst`. Ostatní šablony a bloky jsou skryté.
- Text: „Vážená paní, obdrželi jsme od Vás odstoupení od smlouvy na elektřinu, podané dle § 1829 občanského zákoníku. … Smlouva uzavřená dne: 04.04.2018 … Ulice: Okružní, Číslo pop./orient.: 1087, Obec: Bohumín, Část obce: Nový Bohumín, PSČ: 735 81, Číslo EAN: 859182400508740618 … toto odstoupení od smlouvy nenaplnilo žádný z důvodů stanovenými právními předpisy ani smlouvou samotnou. …“
- Řádek o zprostředkovateli v patičce je skrytý. Dokument má 1 stranu.

## 4. Zjištěná rizika a nedostatky

1. **Neplatná kombinace kódů nic nenahlásí.** Kód `TEXT_VAR`, který v dané šabloně neexistuje (např. `009` v `TEMP_303`), dá dopis jen s hlavičkou a podpisem. `TEMPL_CODE` mimo 301/303/304/308/705 dá prázdné tělo. U `TEMP_301` zůstane navíc 120 mm prázdného místa (`minH`).
2. **Porovnání je přesná shoda řetězců.** `"9"` nebo `" 009"` blok nezobrazí.
3. **Odkaz na pole přes pořadí subformu.** `#subform[2].MIDD_INVISIBLE` přestane fungovat po přeuspořádání subformů v patičce. Doporučuji subform pojmenovat.
4. **`TEMP_303.sf_sml_v_prostor` nemá výchozí `presence="hidden"`.** Funkčně to nevadí (skript ho skryje), ale v Designeru blok překrývá ostatní a je nekonzistentní s ostatními bloky.
5. **Neexistující cíl přetečení.** Bloky v `TEMP_301` mají `<overflow target="Page2.ContentArea2"/>`, ale šablona má jen stránku `MasterPage1`. Při delším textu se cíl ignoruje nebo hlásí varování; pozůstatek po odstraněné druhé stránce.
6. **Nevyužité a chybné proměnné.** 12 proměnných kódů se nepoužívá, dvě mají špatnou hodnotu (`sml_v_prostor`, `jiz_odst_innogy`). Buď je ve skriptech začít používat (a opravit), nebo je odstranit.
7. **Nejednotný zápis podmínek.** `TEMP_308` porovnává s proměnnou, ostatní s literálem.
8. **Duplicitní texty.** Bloky v `TEMP_303` a `TEMP_304` se liší jen úvodní větou a jednou odstavcovou větou; „Identifikace smlouvy“, výčet výhod a závěr jsou zkopírované 8×. Každá změna (např. rebranding innogy, telefonní linka) se musí udělat ve všech kopiích.
9. **Skrytá pole s prázdnými skripty.** `MIDDLEMAN` a pole v `TEMP_705.zam_vyp_obch` mají prázdný skript `initialize`; jsou nadbytečné.
10. **Nevyužité pole `Varianta.PREMEXT`.** Neviditelné a nikde neodkazované.
