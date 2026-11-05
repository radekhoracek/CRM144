# Gramatické chyby ve formulářích CRM113 a CRM144

Kontrolované zdroje:

- **CRM113:** statické texty v XFA šabloně [CRM113.pdf](./CRM113/CRM113.pdf) a testovací data [CRM113_T304.xml](./CRM113/CRM113_T304.xml)
- **CRM144:** statické texty v XFA šabloně [CRM144.pdf](./CRM144/CRM144.pdf) a testovací data [xmlIN.xml](./CRM144/xmlIN.xml). Šablona CRM144 nemá vlastní text dopisu, tělo (`BODY`, `BODY1`, `BODY2`) přichází z dat.

Chyby jsou rozdělené podle závažnosti:

- **Chyba:** porušení pravidel pravopisu nebo mluvnice, doporučuji opravit.
- **Typografie:** chybná mezera, uvozovky apod.
- **Stylistika:** text je správně, ale zní neobratně nebo nepřesně.

`{POLE}` označuje místo, kam formulář vkládá hodnotu z dat (floating field).

---

## 1. CRM113 – šablona

### 1.1 Chyby

| # | Umístění | Původní text | Oprava | Vysvětlení |
|---|---|---|---|---|
| 1 | `TEMP_304/sf_nelze_apl_odst/BODY` | …nenaplnilo žádný z důvodů **stanovenými** právními předpisy… | …nenaplnilo žádný z důvodů **stanovených** právními předpisy… | Přívlastek se shoduje s „důvodů“ (2. pád mn. č.), ne se 7. pádem. |
| 2 | `TEMP_308/sf_chyba_zakaznika/BODY_1` | Výše uvedená společnost nás však informovala, že **evidují** Vaše osobní údaje… | …že **eviduje** Vaše osobní údaje… | Podmětem je „společnost“ (jednotné číslo), přísudek musí být v jednotném čísle. |
| 3 | `TEMP_308/sf_chyba_zakaznika/BODY_1` | …než které jste nám **nahlásil** při uzavření smlouvy. | …než které jste nám **nahlásil(a)**… nebo **sdělil(a)** | Text je jen v mužském rodě, ale dopis dostávají i ženy. Obdobně řeší oslovení pole `TITLE` („Vážená paní / Vážený pane“). |
| 4 | `TEMP_308/sf_sml_v_prostor/BODY_1` | …smlouva byla uzavřena Vámi v **pobočce, nebo** v prostorách společnosti… | …v **pobočce nebo** v prostorách společnosti… | Před „nebo“, které jen vyjmenovává možnosti, se čárka nepíše. |
| 5 | `TEMP_301/sf_jiny_zak/BODY` | …evidujeme **smlouvu, uzavřenou** s jiným zákazníkem. | …evidujeme **smlouvu uzavřenou** s jiným zákazníkem. | Přívlastek „uzavřenou s jiným zákazníkem“ určuje, o jakou smlouvu jde (těsný přívlastek), proto bez čárky. |
| 6 | `TEMP_308/sf_sml_v_prostor`, `sf_neni_pl_sml`, `sf_chyba_zakaznika` (`BODY_1`) | V případě, že zjistíte nové **skutečnosti, týkající** se Vaší smlouvy… | …nové **skutečnosti týkající** se Vaší smlouvy… | Těsný přívlastek, čárka se nepíše. Chyba je ve 3 blocích. |
| 7 | `TEMP_301/sf_fin_zar/BODY` | …v ročním vyúčtování (které bude **vystavené** po uplynutí… | …(které bude **vystaveno** po uplynutí… | V přísudku trpného rodu se ve spisovném textu používá jmenný tvar příčestí („vystaveno“). |

### 1.2 Stylistika (doporučení)

| # | Umístění | Původní text | Návrh |
|---|---|---|---|
| 8 | `TEMP_301` (všechny 4 bloky) | Při **zpracovávání** smlouvy jsme však zjistili… | Při **zpracování** smlouvy… (jde o jednorázový úkon) |
| 9 | `TEMP_301/sf_exist_sml/BODY` | …že **na** stejné odběrné místo máte s námi **již z minulosti** sjednanou platnou smlouvu. | …že **pro** stejné odběrné místo s námi již máte sjednanou platnou smlouvu. |
| 10 | `TEMP_308/sf_sml_v_prostor/BODY_1` | …že smlouva byla uzavřena **Vámi** v pobočce… | …že **jste** smlouvu **uzavřel(a)** v pobočce… (činný rod místo neobratného „uzavřena Vámi“) |
| 11 | `TEMP_308/sf_chyba_zakaznika/BODY_1` | …že evidují **Vaše osobní údaje jiné, než které** jste nám nahlásil… | …že eviduje **jiné osobní údaje, než jaké** jste nám nahlásil(a)… |
| 12 | `TEMP_308` (všechny 3 bloky) | …máte platnou smlouvu na dodávku {COMMODITY} od innogy, **která** je závazná… | „která“ se gramaticky může vztahovat k „innogy“. Návrh: …máte s innogy platnou smlouvu na dodávku {COMMODITY}, která je závazná… |
| 13 | `TEMP_303` (všechny 4 bloky) | Vaši **výpověď ze smlouvy** na {COMMODITY} | Vaši **výpověď smlouvy** na {COMMODITY} (vypovídá se smlouva; energetický zákon používá „výpověď smlouvy“) |
| 14 | `TEMP_705` (oba bloky) | …zastavte se … **v kterékoliv naší pobočce**. | …zastavte se … **na kterékoli z našich poboček**. |
| 15 | Patička `Prostrednik` | Zpracováno zprostředkovatelem {MIDDLEMAN}, jednajícím pro… | Čárka je přípustná (volný přívlastek). Když `MIDDLEMAN` chybí, vznikne „zprostředkovatelem , jednajícím“ s mezerou před čárkou. |

## 2. CRM113 – testovací data `CRM113_T304.xml`

| # | Řádek | Původní text | Oprava | Typ |
|---|---|---|---|---|
| 16 | 412 | (v dokumentu jako „innogy**”**) | (v dokumentu jako „innogy**“**) | Typografie: česká zavírací uvozovka je `“` (U+201C), ne `”` (U+201D). |
| 17 | 391, 410 | zápis v OR: MěS v Praze, **sp.zn.:** C 220583 | …, **sp. zn.:** C 220583 | Typografie: mezi zkratkami se píše mezera. |
| 18 | 28 | k 15. dni **v měsíci** | k 15. dni **měsíce** | Stylistika |

---

## 3. CRM144 – šablona

Šablona obsahuje jen záhlaví, patičku a podpisy. Gramatickou chybu v nich nevidím.

| # | Umístění | Poznámka |
|---|---|---|
| 19 | Patička `Prostrednik` | Stejné jako bod 15: když chybí `MIDDLEMAN`, zobrazí se „zprostředkovatelem , jednajícím“. |

## 4. CRM144 – testovací data `xmlIN.xml` (tělo dopisu)

| # | Řádek / pole | Původní text | Oprava | Typ |
|---|---|---|---|---|
| 20 | 328 `HEADER1` | NEAKCEPTACE **ŽÁDOSTI NA** UKONČENÍ SMLOUVY | NEAKCEPTACE **ŽÁDOSTI O** UKONČENÍ SMLOUVY | **Chyba:** správná vazba je „žádost **o** něco“. CRM113 v obdobném nadpisu používá „POŽADAVKU NA UKONČENÍ“, což je správně. |
| 21 | 38 `BODY` | …o sdružených službách **dodávky  plynu**… | …o sdružených službách **dodávky plynu**… | Typografie: dvojitá mezera. |
| 22 | 38 `BODY` | …pro uvedené odběrné místo **EIC kód:** 27ZG700Z03707779. | …pro odběrné místo **s EIC kódem** 27ZG700Z03707779. | Stylistika: dvojtečka uprostřed věty. Navíc „uvedené“ odkazuje na místo, které ještě nebylo uvedeno. |
| 23 | 38 `BODY` | obdrželi jsme od Vás **výpověď Smlouvy** o sdružených službách… | …výpověď **smlouvy** o sdružených službách… | Stylistika: velké „S“ je správně jen tehdy, když jde o definovaný pojem. Ve stejném dopise je ale „smlouva“ jinde psaná malým písmenem, proto zápis sjednoťte. |
| 24 | 48 `BODY2` | …využijte chat na našich webových **stránkách. Nebo nás** navštivte v nejbližší pobočce. | …využijte chat na našich webových **stránkách, nebo nás** navštivte **na** nejbližší pobočce. | Stylistika: věta nemá začínat spojkou „Nebo“, je lepší ji připojit k předchozí. |
| 25 | 480 `SALES_POM` | (v dokumentu jako „innogy**”**) | (v dokumentu jako „innogy**“**) | Typografie, viz bod 16. |
| 26 | 458 a další `SALESALL`, `SALES_PART_II`, `SALESREGISTER3` | **sp.zn.:** | **sp. zn.:** | Typografie, viz bod 17. |

---

## 5. Shrnutí – co opravit přednostně

1. **CRM113** `TEMP_304/sf_nelze_apl_odst`: „stanovenými“ → „stanovených“.
2. **CRM113** `TEMP_308/sf_chyba_zakaznika`: „evidují“ → „eviduje“; „nahlásil“ → „nahlásil(a)“.
3. **CRM144** `HEADER1`: „ŽÁDOSTI NA UKONČENÍ“ → „ŽÁDOSTI O UKONČENÍ“. Tento text pochází z generujícího systému (data), ne z šablony.
4. **CRM113** čárky: `TEMP_301/sf_jiny_zak` („smlouvu uzavřenou“), `TEMP_308` ×3 („skutečnosti týkající“) a `TEMP_308/sf_sml_v_prostor` („v pobočce nebo“).
5. **CRM113** `TEMP_301/sf_fin_zar`: „vystavené“ → „vystaveno“.
6. Typografie v datech obou formulářů: uvozovky v `SALES_POM`, „sp. zn.“ a dvojitá mezera v `BODY` (CRM144).
