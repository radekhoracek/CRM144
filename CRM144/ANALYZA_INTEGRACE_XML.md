# Analýza integrace formuláře CRM144.pdf s daty xmlIN.xml

## 1. Princip integrace

[CRM144.pdf](./CRM144.pdf) je dynamický XFA formulář z Adobe LiveCycle Designer 11. Šablona a data jsou v něm oddělené:

| XFA paket | Obsah v PDF |
|---|---|
| `template` | layout, pole, vazby a skripty |
| `datasets` | prázdný (`<xfa:data/>`), data se dodávají až při generování |
| `config` | `runScripts=server`, `dynamicRender=required`, výstup do PDF |
| `connectionSet` / XSD | chybí |

Formulář nemá datové připojení ani XSD schéma. Data z [xmlIN.xml](./xmlIN.xml) se proto při generování (merge na serveru) spojují s šablonou **jen podle shody názvů XML elementů s názvy polí**. Pak se spustí skripty `initialize` (`runAt="both"`), které podle hodnot dat skryjí nebo zobrazí bloky. Výsledkem je statické PDF dokumentu.

```text
CRM systém ──> xmlIN.xml ──> merge (XFA data binding) ──> initialize skripty ──> layout ──> PDF
```

## 2. Struktura vstupních dat

```xml
<?xml version="1.0" encoding="utf-8"?>
<form>
  <header>       <!-- 8 elementů: form_id, template, pdf_box, ... -->
  <parameters>   <!-- 484 plochých elementů (žádné vnořené skupiny) -->
</form>
```

- Kořen `form` se jmenuje stejně jako kořenový subform šablony `form`.
- Šablona **nepoužívá žádný element z `header`**, tedy ani `form_id=CRM144` nebo `template=315`. Šablona v dokumentu je ve skutečnosti řízena hodnotou `parameters/TEMPL_CODE`.
- Všechna užitečná data jsou ploše v `parameters`.

## 3. Typy vazeb (binding) v šabloně

| Vazba | Počet polí | Jak funguje |
|---|---|---|
| `global` | 13 názvů (BODY, BODY1, BODY2, TEMPL_CODE, TEXT_VAR, MIDD_INVISIBLE, PREMEXT, PREMLABEL, barcode, DOC_ID_BARCODE, PODPIS4, PODPIS5, ID_form) | hodnota se najde podle názvu kdekoli v datech; **všechna pole se stejným názvem dostanou stejnou hodnotu** |
| `once` (výchozí, bez `<bind>`) | 27 (hlavička, adresy, obchodník, zákazník, datum, MIDDLEMAN) | párování podle názvu v aktuálním datovém kontextu (scope) |
| `none` | subformy patičky `Paticka` | subform se na data neváže, jeho potomci se párují v kontextu rodiče |

Subform `SIGNATURES.sf_Podpisy` má vazbu `global`. Ostatní subformy (`Page1`, `sf_obch_adr`, `Subform1` …) mají výchozí vazbu `once`, ale v datech pro ně neexistují odpovídající skupiny.

## 4. Mapování polí na data (pro xmlIN.xml)

### 4.1 Řízení obsahu (global)

| Pole šablony | Element dat | Hodnota v xmlIN.xml | Účel |
|---|---|---|---|
| `Varianta.TEMPL_CODE` | `TEMPL_CODE` | `315` | výběr šablony `TEMP_314/315/316` |
| `Varianta.TEXT_VAR` | `TEXT_VAR` | `041` | výběr bloku `sf_NN` v šabloně |
| `Paticka.MIDD_INVISIBLE` | `MIDD_INVISIBLE` | `X` | skrytí řádku o zprostředkovateli |
| `DOC_ID_BARCODE`, `barcode` | stejný název | `331080000000082244` | ID dokumentu, čárový kód Code 39 (`dataLength=20`) |

Logika skrývání je popsaná v [ANALYZA_SKRYVANI_BLOKU.md](./ANALYZA_SKRYVANI_BLOKU.md).

### 4.2 Textový obsah dopisu (global, rich text)

| Pole | Element | Formát |
|---|---|---|
| `BODY` (v každé šabloně) | `BODY` | XHTML (`<body xmlns="http://www.w3.org/1999/xhtml">`) – oslovení a úvod |
| `BODY1` (v každém bloku `sf_NN`) | `BODY1` | XHTML – text varianty |
| `BODY2` (v každé šabloně) | `BODY2` | XHTML – závěr dopisu |

**Klíčové zjištění:** bloky `sf_NN` neobsahují žádný vlastní statický text. Všech 30 výskytů pole `BODY1` má vazbu `global`, takže dostanou **stejný** text z dat. Samotné znění varianty tedy skládá CRM systém a posílá ho v elementu `BODY1`. Kód `TEXT_VAR` jen vybírá kontejner, ve kterém se text zobrazí, tedy jeho výšku a chování (pevná/rostoucí výška, chráněné/needitovatelné pole).

Rich text se předává jako XHTML vnořené přímo v elementu. Podporované jsou `<p>`, `<b>`, `<br/>` a inline `style`. Pole mají `contentType="text/html"`.

### 4.3 Hlavička a adresní blok (once)

| Oblast | Pole = elementy dat |
|---|---|
| Nadpis | `HEADER1`, `HEADER2` |
| Obchodník | `SALESNAME`, `SALESADDRESS1/2`, `SALESIC`, `SALESDIC`, `SALESREGISTER1/2` |
| Zákazník | `CUSTNUMB`, `CUSTNAME`, `CUSTADDRESS1/2`, `CUSTIC`, `CUSTDIC`, `CUSTPROXY` |
| Odběrné místo | `PREMNUMBER`, `PREMADDRESS1/2` |
| Korespondenční adresa | `CORNAME`, `CORADDRESS1/2`, `CORDATECOUR`, `CUSTTELCOUR` |
| Místo a datum | `WHERE`, `DATE` |
| Patička | `MIDDLEMAN` |

### 4.4 Floating fields (pole vložená do textu)

| Statický text (draw) | Vložená pole | Výsledek |
|---|---|---|
| `Subform1.TEXT2` | `SALESREGISTER1`, `SALESREGISTER2` | „Zápis v OR: Městský soud v Praze, oddíl C, vložka 220583“ |
| `Paticka.Prostrednik` | `MIDDLEMAN` | „Zpracováno zprostředkovatelem {MIDDLEMAN}, jednajícím pro innogy Energie…“ |

Samotná pole jsou skrytá (`presence="hidden"`). Jejich hodnota se do textu vkládá přes `xfa:embed="#floatingFieldNNNNNN"`.

### 4.5 Podpisy (global, obrázky)

`PODPIS4` a `PODPIS5` jsou pole typu `imageEdit` (`data="embed"`). Očekávají obrázek v base64. V xmlIN.xml jsou prázdné, takže se podpisy nezobrazí.

## 5. Statistika využití dat

| Ukazatel | Hodnota |
|---|---|
| elementů v `parameters` | 484 |
| využitých šablonou | 39 |
| nevyužitých | 445, z toho 132 neprázdných |
| pole šablony bez dat | `ID_form` |

Nevyužité elementy jsou hlavně checkboxy `CB_*` (114) a další atributy `CUST*`, `PREM*`, `COR*`, `SALES*` a `CONTR*`. Jde o generický datový export CRM, ze kterého si každý formulář bere jen to, co potřebuje.

## 6. Zjištěná rizika a nedostatky

1. **Vazba `once` vs. plochá data.** 27 polí v hlavičce a adresním bloku nemá explicitní vazbu. Leží v pojmenovaných subformech (`Page1/sf_obch_adr/sf_obchodnik/Subform1` …), kterým v datech neodpovídá žádná skupina, a data jsou až o úroveň níž v `parameters`. Podle pravidel XFA scope matchingu se tak párování nemusí podařit. Plnění těchto polí je nutné **ověřit náhledem** v Designeru (Preview Data = xmlIN.xml) nebo na generujícím serveru. Spolehlivější je nastavit u nich `global` nebo explicitní `ref` (např. `$record.parameters.SALESNAME`).
2. **Patička přepisuje identifikaci formuláře.**
   - Pole `Paticka.TEMPL_CODE` (titulek „innogy formulář“) má výchozí hodnotu `CRM144`. Protože má vazbu `global`, přepíše ji hodnota `TEMPL_CODE=315` z dat, takže se zobrazí „innogy formulář 315“.
   - Pole `ID_form` nemá v datech protějšek a zobrazí výchozí hodnotu **`CRM113`**, zřejmě pozůstatek po kopírování šablony.
   - Obě pole mají `relevant="-print"`, takže se při tisku neobjeví, na obrazovce ale ano.
3. **Element `header` se ignoruje.** `header/form_id` a `header/template` se nepoužívají. Pokud se `header/template` a `parameters/TEMPL_CODE` rozejdou, rozhodne `parameters/TEMPL_CODE` a nikdo na nesoulad neupozorní.
4. **Formát data.** `DATE` a `CORDATECOUR` jsou pole typu `dateTimeEdit` bez picture clause. Data přicházejí jako `18.08.2026`, ne v kanonickém formátu `2026-08-18`. Hodnota se zobrazí tak, jak přišla, bez lokálního formátování a bez kontroly platnosti.
5. **Globální názvy jsou sdílené.** Všechna pole `BODY1`, `PREMEXT` a `PREMLABEL` ve všech blocích dostanou stejnou hodnotu. Případné rozlišení textu podle varianty musí zajistit CRM v elementu `BODY1`, šablona to neumí.
6. **Pevná výška rich-text polí.** `BODY` (25 mm), `BODY2` (40 mm) a `BODY1` (10 až 20 mm podle bloku) mají pevnou výšku. Delší XHTML z dat se ořízne. V testovacích datech má `BODY2` zhruba 11 řádků při 8 pt, takže je těsně na hranici 40 mm.
7. **Chybí validace vstupu.** Neexistuje XSD ani kontrola povinných polí. Chybějící nebo přejmenovaný element (překlep, jiná velikost písmen) vede k prázdnému poli bez jakékoli chyby.
8. **Prázdné podpisy.** `PODPIS4` a `PODPIS5` jsou v testovacích datech prázdné. Ověřte, jestli CRM podpisy posílá, nebo jestli se mají vkládat jinak.

## 7. Doporučení

- Ověřit plnění polí s vazbou `once` náhledem s xmlIN.xml. Podle výsledku sjednotit vazby na `global` nebo explicitní `ref`.
- Opravit výchozí hodnotu `ID_form` (`CRM113` → `CRM144`) a buď přejmenovat footer pole `TEMPL_CODE`, nebo mu nastavit vazbu `none`, aby zobrazovalo ID formuláře.
- Posílat datum v ISO formátu a v šabloně nastavit display picture `date{DD.MM.YYYY}`.
- U rich-text polí použít `minH` místo pevné `h`, aby se delší text neořízl.
- Zvážit XSD, nebo aspoň kontrolu povinných elementů (`TEMPL_CODE`, `TEXT_VAR`, `BODY*`) na straně CRM.
