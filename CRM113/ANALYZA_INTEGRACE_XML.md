# Analýza integrace formuláře CRM113.pdf s daty CRM113_T304.xml

## 1. Princip integrace

[CRM113.pdf](./CRM113.pdf) je dynamický XFA formulář z Adobe LiveCycle Designer 11. Šablona a data jsou v něm oddělené:

| XFA paket | Obsah v PDF |
|---|---|
| `template` | layout, pole, vazby, skripty a **pevné texty všech variant dopisu** |
| `datasets` | prázdný (`<xfa:data/>`), data se dodávají až při generování |
| `config` | `runScripts=server`, `dynamicRender=required`, výstup do PDF; poslední uložení do `…\NV_20251113\CRM_šablony\CRM113_new_p.pdf` |
| `connectionSet` / XSD | chybí |

Formulář nemá datové připojení ani XSD schéma. Data z [CRM113_T304.xml](./CRM113_T304.xml) se při generování (AEM Forms Output) spojují se šablonou **jen podle shody názvů XML elementů s názvy polí**. Pak se spustí skripty `initialize` (`runAt="both"`), které podle `TEMPL_CODE` a `TEXT_VAR` skryjí nebo zobrazí bloky. Výsledek je v [out_PDF.pdf](./out_PDF.pdf) (1 strana, Producer „Adobe Experience Manager forms output“).

```text
CRM systém ──> CRM113_T304.xml ──> merge (XFA data binding) ──> initialize skripty ──> layout ──> PDF
```

**Hlavní rozdíl proti CRM144:** text dopisu není v datech (`BODY`, `BODY1`, `BODY2`), ale je napsaný přímo v šabloně jako statický text (`draw BODY`). Z dat se do něj doplňují jen jednotlivé hodnoty přes floating fields. Výjimkou je šablona `TEMP_308`, která úvod dopisu čeká v datovém elementu `BODY`.

## 2. Struktura vstupních dat

```xml
<?xml version="1.0" encoding="utf-8"?>
<form>
  <header>       <!-- 8 elementů: form_id=CRM113, template=304, pdf_box=X, ... -->
  <parameters>   <!-- 427 plochých elementů (žádné vnořené skupiny) -->
</form>
```

- Kořen `form` se jmenuje stejně jako kořenový subform šablony `form`.
- Šablona **nepoužívá žádný element z `header`**. Šablonu dokumentu řídí `parameters/TEMPL_CODE` (zde `304`, shodně s `header/template`).
- Všechna užitečná data jsou ploše v `parameters`.

## 3. Typy vazeb (binding) v šabloně

| Vazba | Pole | Jak funguje |
|---|---|---|
| `global` | řídicí pole (`TEMPL_CODE`, `TEXT_VAR`, `MIDD_INVISIBLE`), `barcode`, `DOC_ID_BARCODE`, `ID_form`, `BODY` (TEMP_308) a všechna skrytá pole pro floating fields (`CUSTTITLELET`, `PREM*`, `ZCONTPLN2`, `COMMODITY_DESCRIPTION*`) | hodnota se najde podle názvu kdekoli v datech; **všechna pole se stejným názvem dostanou stejnou hodnotu** |
| `once` (výchozí, bez `<bind>`) | 29 polí v hlavičce a adresním bloku, `MIDDLEMAN` | párování podle názvu v aktuálním datovém kontextu |
| `none` | subform `Paticka` a jeho nepojmenované subformy | subform se na data neváže |

**Ověřeno na [out_PDF.pdf](./out_PDF.pdf):** pole s vazbou `once` v hlavičce a adresním bloku se na serveru AEM naplnila správně (obchodník, zákazník, odběrné místo, korespondenční adresa, místo a datum). Riziko popsané u CRM144 se tím v praxi nepotvrdilo.

## 4. Mapování polí na data (pro CRM113_T304.xml)

### 4.1 Řízení obsahu (global)

| Pole šablony | Element dat | Hodnota v datech | Účel |
|---|---|---|---|
| `Varianta.TEMPL_CODE` | `TEMPL_CODE` | `304` | výběr šablony `TEMP_301/303/304/308/705` |
| `Varianta.TEXT_VAR` | `TEXT_VAR` | `009` | výběr bloku varianty uvnitř šablony |
| `Paticka.MIDD_INVISIBLE` | `MIDD_INVISIBLE` | `X` | skrytí řádku o zprostředkovateli |
| `DOC_ID_BARCODE`, `barcode` | stejný název | `331010000000052428` | ID dokumentu, čárový kód Code 39 (`dataLength=20`) |
| `Varianta.PREMEXT` | `PREMEXT` | `859182400508740618` | neviditelné, nikde nepoužité |

Logika skrývání je popsaná v [ANALYZA_SKRYVANI_BLOKU.md](./ANALYZA_SKRYVANI_BLOKU.md).

### 4.2 Text dopisu (statický text + floating fields)

Každý blok varianty obsahuje statický text `BODY` (u `TEMP_308` `BODY_1`), do kterého se vkládají hodnoty přes `xfa:embed="#floatingFieldNNNNNN"`. Vložená pole jsou skrytá (`presence="hidden"`) a mají vazbu `global`.

| Vložené pole | Hodnota v testovacích datech | Kde se v textu objeví |
|---|---|---|
| `CUSTTITLELET` | `Vážená paní` | oslovení (TEMP_301, 303, 304, 705) |
| `COMMODITY_DESCRIPTION_XI` | `elektřinu` | „…odstoupení od smlouvy na {…}“ (TEMP_301, 303, 304) |
| `COMMODITY_DESCRIPTION` | `elektřiny` | „…dodávku {…} od innogy“ (TEMP_308, 705) |
| `ZCONTPLN2` | `04.04.2018` | Smlouva uzavřená dne |
| `PREMSTREET`, `PREMSTRNUMBER` | `Okružní`, `1087` | Ulice, Číslo pop./orient. |
| `PREMCITY`, `PREMCITYPART`, `PREMPOSTALCODE` | `Bohumín`, `Nový Bohumín`, `735 81` | Obec, Část obce, PSČ |
| `PREMLABEL`, `PREMEXT` | `EAN`, `859182400508740618` | „Číslo {PREMLABEL}: {PREMEXT}“ (TEMP_705: „{PREMLABEL} kód: …“) |

**Klíčové zjištění – sdílené floating fields napříč šablonami.** Floating field se odkazuje na konkrétní pole podle `id`. Většina bloků nepoužívá vlastní skrytá pole, ale pole z **jiných** (i skrytých) šablon:

| Cílové pole (vlastník) | Použito v |
|---|---|
| `TEMP_301.sf_exist_sml`: `ZCONTPLN2`, `PREMSTREET`, `PREMSTRNUMBER`, `PREMCITY`, `PREMCITYPART`, `PREMPOSTALCODE` | TEMP_301, TEMP_303, TEMP_304, TEMP_308 |
| `TEMP_301.sf_jiny_zak`: `PREMLABEL`, `PREMEXT` | sf_jiny_zak, TEMP_303, TEMP_304, TEMP_308 |
| `TEMP_304.sf_po_lhute`: `COMMODITY_DESCRIPTION_XI` | TEMP_301, TEMP_303, TEMP_304 |
| `TEMP_308.sf_sml_v_prostor`: `COMMODITY_DESCRIPTION` | všechny bloky TEMP_308 |
| `TEMP_705.zam_vyp_obch`: `ZCONTPLN2`, `PREM*` | oba bloky TEMP_705 |

Funguje to jen díky vazbě `global` (všechna stejnojmenná pole mají stejnou hodnotu). Výstup [out_PDF.pdf](./out_PDF.pdf) potvrzuje, že se hodnoty doplní i z polí ve skryté `TEMP_301`.

### 4.3 Hlavička a adresní blok (once)

| Oblast | Pole = elementy dat |
|---|---|
| Nadpis | `HEADER1`, `HEADER2` |
| Obchodník | `SALESNAME`, `SALESADDRESS1/2`, `SALESIC`, `SALESDIC`, `SALESREGISTER1/2` (floating v „Zápis v OR“) |
| Zákazník | `CUSTNUMB`, `CUSTNAME`, `CUSTADDRESS1/2`, `CUSTIC`, `CUSTDIC`, `CUSTPROXY` |
| Odběrné místo | `PREMNUMBER`, `PREMADDRESS1/2` |
| Korespondenční adresa | `CORNAME`, `CORADDRESS1/2`, `CORDATECOUR`, `CUSTTELCOUR` |
| Místo a datum | `WHERE`, `DATE` |
| Patička | `MIDDLEMAN` (floating v textu „Zpracováno zprostředkovatelem…“) |

### 4.4 Obrázky a podpis

| Prvek | Typ | Poznámka |
|---|---|---|
| `Image1` (logo) | statický JPEG v šabloně | |
| `Paticka.Image2` | statický JPEG v šabloně | |
| `SIGNATURES.Podpis_new` | statický JPEG v šabloně (89 × 28 mm) | **podpis je pevně v šabloně**, elementy `PODPIS*` z dat se nepoužívají |

### 4.5 Patička

| Pole | Výchozí hodnota | Výsledek |
|---|---|---|
| `TEMPL_CODE` (titulek „innogy formulář“) | žádná | na obrazovce „innogy formulář 304“, netiskne se (`relevant="-print"`) |
| `ID_form` | `CRM113` | odpovídá názvu formuláře, netiskne se; v datech protějšek nemá |

## 5. Statistika využití dat

| Ukazatel | Hodnota |
|---|---|
| elementů v `parameters` | 427 |
| využitých šablonou | 43 |
| nevyužitých | 384, z toho 173 neprázdných |
| pole šablony bez dat | `BODY` (TEMP_308), `ID_form` |
| využité, ale v datech prázdné | `CORDATECOUR`, `CUSTDIC`, `CUSTIC`, `CUSTPROXY`, `CUSTTELCOUR`, `MIDDLEMAN` |

Nevyužité jsou hlavně checkboxy `CB_*` (56), `C_FIELD_*`, `COMMODITY_DESCRIPTION_*`, `BANK*`, `BBP_*` a `PODPIS*`. Element `BODY1` (prázdné XHTML) se také nepoužívá. Jde o generický datový export CRM.

## 6. Zjištěná rizika a nedostatky

1. **Křížové odkazy floating fields.** Texty šablon 303, 304 a 308 čerpají hodnoty ze skrytých polí v `TEMP_301` a `TEMP_304.sf_po_lhute`. Kdo smaže nebo přejmenuje šablonu `TEMP_301` (např. jako „nepoužívanou“), rozbije adresy a datum ve všech ostatních šablonách. V Designeru tato závislost není vidět.
2. **`TEMP_308` potřebuje `BODY` z dat.** Pole `TEMP_308.BODY` (rich text, pevná výška 20 mm, `global`) je jediný úvod dopisu. Texty `BODY_1` začínají až „Výše uvedená společnost nás však informovala…“, bez oslovení. Testovací data `BODY` neobsahují. Při `TEMPL_CODE=308` bez `BODY` vyjde dopis bez oslovení a s 20 mm prázdného místa; delší `BODY` se ořízne.
3. **Prázdné popisky se tisknou.** U prázdných `CUSTIC`, `CUSTDIC` a `CUSTPROXY` se v PDF vytiskne „IČ: DIČ:“ a „Zastoupen:“ bez hodnoty (ověřeno v out_PDF.pdf).
4. **Formát data.** `DATE`, `CORDATECOUR` jsou `dateTimeEdit` bez picture clause; data přicházejí jako `10.07.2026`. Zobrazí se tak, jak přišla, bez kontroly platnosti. Floating field `ZCONTPLN2` je obyčejný text.
5. **Element `header` se ignoruje.** Pokud se `header/template` a `parameters/TEMPL_CODE` rozejdou, rozhodne `TEMPL_CODE` bez varování.
6. **Podpis je natvrdo v šabloně.** Změna podepisující osoby vyžaduje úpravu a nové nasazení šablony.
7. **Chybí validace vstupu.** Neexistuje XSD ani kontrola povinných polí. Chybějící element vede k prázdnému místu v textu (např. „Ulice:  Číslo pop./orient.:“).
8. **Jazykové chyby ve statickém textu.**
   - `TEMP_304.sf_nelze_apl_odst`: „nenaplnilo žádný z důvodů **stanovenými** právními předpisy“ → „stanovených“ (je v testovacím výstupu).
   - `TEMP_308.sf_chyba_zakaznika`: „jste nám nahlásil“ – jen mužský rod.
   - Ruční zalomení řádků uvnitř vět (např. „…právními předpisy ⏎ ani smlouvou samotnou“) – při změně písma nebo šířky vznikne nepěkné zalomení.

## 7. Doporučení

- Každému bloku dát vlastní skrytá pole pro floating fields, nebo je přesunout do jednoho společného (nikdy neskrývaného) subformu, např. `Varianta`, a odkazovat se jen na něj.
- Ověřit s CRM, že pro `TEMPL_CODE=308` posílá `BODY`; pole změnit na `minH` místo pevné `h=20mm`.
- Opravit texty (bod 6.8).
- Popisky `IČ`, `DIČ`, `Zastoupen` skrývat při prázdné hodnotě.
- Posílat datum v ISO formátu a v šabloně nastavit display picture `date{DD.MM.YYYY}`.
- Zvážit XSD nebo kontrolu povinných elementů (`TEMPL_CODE`, `TEXT_VAR`, `CUSTTITLELET`, `ZCONTPLN2`, `PREM*`) na straně CRM.
