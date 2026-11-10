# Progress 03 – Implementace webu CineLumen (cvičení 4)

**Výstup:** složka `web/` – funkční web festivalu v čistém HTML, CSS a JavaScriptu.
**Spuštění:** otevřít `web/index.html` v prohlížeči (dvojklik). Nic se neinstaluje, není potřeba server ani build. Bez internetu se jen použije záložní písmo.
**Východisko:** prototyp 02 a wireframy 02 (závazné výstupy). Web na ně přímo navazuje. Změny proti prototypu jsou malé a každá je zdůvodněná (viz kap. 3), odhadem do 10 % rozsahu.

```
web/
├── index.html        jediný HTML dokument: hlavička, navigace, <main>, patička
├── css/style.css     mobile-first styly, tmavý/světlý režim
├── js/data.js        data festivalu (převod cinelumen_festival_data.json)
├── js/app.js         router, stránky, interaktivní funkce, stav v localStorage
└── progress-03.md
```

## 1. Co je implementováno

### Minimální stav podle zadání

| Požadavek | Jak je splněn |
| --- | --- |
| Úvodní stránka | `#/` – hero s datem a místem, CTA „Koupit CinePass / lístek“, banner Open-Air (vstup do scénáře 2), rozcestník, „Dnes promítáme“ s kapacitním semaforem, zahájení a finále. |
| Obsahově odlišné stránky | Program (`#/program`), Katalog filmů (`#/filmy`), Detail filmu (`#/udalost/…`), Hosté a profil hosta, Kina & mapa a profil sálu, Vstupenky, Nákup (3 kroky), Potvrzení, Můj diář, Praktické info & FAQ, Vyhledávání, **nově Kontakt & press**, stránka 404. |
| Funkční hlavní navigace | Desktop: horní menu Program · Filmy · Hosté · Kina & mapa · Vstupenky · Praktické, CTA, přihlášení CinePass, diář s počtem položek. Mobil: spodní fixní lišta Domů · Program · Filmy · Můj diář · Více (podle specifikace). Aktivní položka je označena `aria-current="page"`. Drobečková navigace na všech podstránkách, nově i patička s mapou webu. |
| Responzivní mobile-first | Základní CSS je pro mobil (jeden sloupec, spodní lišta), od 640 px dvousloupcové mřížky, od 900 px desktopové menu, filtry v levém sloupci a „sticky“ rezervační panel. |
| Funkční interaktivní funkce | Viz tabulka níže – všechny fungují, ne jen naznačeně. |
| Sémantické HTML | `header`, `nav` (s `aria-label`), `main`, `footer`, `section`, `article`, `aside`, `dl` pro fakta, `table` s `caption` a `scope`, `form` + `label` + `fieldset/legend`, `details/summary` (FAQ, filtry), nativní `dialog`, `output` u počtu vstupenek, `ol` pro drobečky a kroky nákupu. Jeden `h1` na stránku, nadpisy bez přeskakování úrovní. |
| CSS | Vlastní CSS bez frameworku, design tokeny v custom properties, `color-mix()`, `:has()`, `clamp()`, `text-wrap: balance`. |
| JavaScript podle potřeby | Vanilla JS bez knihoven. JS řeší jen to, co HTML/CSS neumí: filtrování, diář, kolize, rezervace, nákup, export. |

### Interaktivní funkce

| Funkce | Kde | Stav |
| --- | --- | --- |
| Filtr programu (den, sál, typ, Q&A, skrýt vyprodané, fulltext) + přepínač Seznam / Časová osa; stav filtru v URL (lze sdílet); seznam po 10 položkách s tlačítkem „Další“ | Program | hotovo |
| Můj diář s kontrolou časových kolizí a pěšího přesunu mezi sály, nabídka reprízy jedním klikem, trasa na mapě; **nově filtr podle dne** (s počtem položek) a stránkování po 10 | Detail, Diář | hotovo |
| Rezervace na CinePass (přihlášení číslem akreditace, 0 Kč, QR, upozornění „potvrďte příchodem 10 min předem“) | Detail, Diář | hotovo (simulace) |
| Nákup jednotlivých vstupenek bez registrace (počet, e-mail, platba, souhrn, potvrzení s QR a mapou) | Nákup, Potvrzení | hotovo (simulace platby) |
| Kapacitní semafor Volno / Dochází / Vyprodáno, mění se po rezervaci a nákupu | všude u projekcí | hotovo (simulovaná obsazenost) |
| Globální vyhledávání (bez ohledu na diakritiku) | hlavička | hotovo |
| Vyhledávání a řazení hostů, katalog filmů s řazením a stránkováním po 10 („Další“) | Hosté, Filmy | hotovo |
| Hledání ve FAQ | Praktické info | hotovo |
| **Nové:** export Mého diáře do kalendáře (.ics) | Diář | hotovo |
| **Nové:** kontaktní formulář s validací | Kontakt & press | hotovo (nic se neodesílá) |
| Tmavý / světlý režim (výchozí podle systému, přepínač si pamatuje volbu) | hlavička | hotovo |

### Implementace uživatelských scénářů

Data festivalu (`cinelumen_festival_data.json`) popisují Kutnou Horu a sály Alfa, Beta, Open-Air, Studio, proto jsou scénáře ze specifikace převedeny na odpovídající tituly z dat (stejně jako v prototypu 02).

**Scénář 1 – výběr filmu s debatou a rezervace na CinePass (mobil):**
spodní lišta *Program* → den *Pá 16. 10.* → *Další filtry* → *Pouze s debatou tvůrců (Q&A)* (13 bloků) → *Ztraceni v čase* (20:45, Velký sál Alfa) → *Rezervovat na CinePass · 0 Kč* → přihlášení (*Vyplnit ukázkové údaje*: CL-2026-0417) → potvrzení s QR kódem a hláškou „Potvrďte příchodem do 20:35“ → *Otevřít Můj diář* → případně *Přidat diář do kalendáře (.ics)*.

**Scénář 2 – nákup 2 lístků do letního kina bez registrace:**
úvodní stránka → banner *Speciální projekce pod hvězdami* → program Open-Air na sobotu → *Klasický český noir pod hradbami* → *Koupit jednotlivé lístky* → počet 2 → e-mail + Apple Pay / karta → souhrn → *Zaplatit* → potvrzení se 2 vstupenkami (QR, „Stáhnout PDF“) a mapou cesty.

**Scénář 3 – kolize a přesun mezi sály:**
*Můj diář* → *Načíst ukázkový diář* → diář hlásí „Přesun Sál Beta → Sál Alfa: máte jen 15 min, běžná pěší vzdálenost je 18 min“ → *Ukázat trasu*. Poté detail *Monzunové echo* (Pá) → *Uložit do Mého diáře* → dialog s kolizí, trasou a nabídkou reprízy → uložení reprízy jedním klikem.

## 2. Aktuální best practices a trendy v UX a designu a jejich implementace

| Trend / best practice | Proč | Jak je to v CineLumen |
| --- | --- | --- |
| **Mobile-first a navigace v dosahu palce** | Většina diváků web používá na mobilu přímo na festivalu. | Spodní lišta s 5 položkami (vzor iOS/Android), cílové plochy min. 44 × 44 px, „Více“ jako spodní panel. |
| **Tmavý režim a respektování preferencí systému** | Cílová skupina 1 chce web do potemnělého sálu; `prefers-color-scheme`. | Tmavý vzhled je výchozí, světlý podle systému nebo přepínačem, volba se pamatuje. Obě varianty prošly kontrolou kontrastu. |
| **Přístupnost podle WCAG 2.2 AA** | Od června 2025 platí European Accessibility Act, WCAG 2.2 je aktuální standard. | Kontrast min. 4,5 : 1, viditelný fokus, odkaz „Přeskočit na obsah“, `scroll-padding` tak, aby fokus nezakryla fixní lišta (kritérium 2.4.11), stav semaforu nese i text (ne jen barva), formuláře s popisky a srozumitelnými chybami (`aria-invalid`). |
| **Nativní prvky místo vlastních komponent** | Prohlížeč sám řeší fokus, Esc, čtečky obrazovky; méně JS. | Dialogy jsou nově `<dialog>` + `showModal()` (předtím vlastní `div`), FAQ a filtry přes `<details>`. |
| **Progressive disclosure** | Na mobilu nezahltit obrazovku. | Den je ve filtru programu vždy vidět, ostatní filtry ve sbalitelném bloku s počtem aktivních filtrů. Dlouhé seznamy (program, filmy, diář) ukazují 10 položek a tlačítko „Další“ s informací „Zobrazeno 10 z 60“. |
| **„Načíst další“ místo nekonečného scrollu** | Nekonečný scroll na mobilu znemožňuje dostat se k patičce a ztrácí pozici; tlačítko dává uživateli kontrolu (doporučení NN/g, Baymard). | Tlačítko „Další (10)“, po načtení se fokus přesune na první nově zobrazenou položku a čtečka ohlásí „Načteno dalších 10 položek“. Počet zobrazených se nezmění po uložení do diáře, při změně filtru se vrátí na 10. |
| **Stav v URL** | Sdílení, tlačítko Zpět, záložky. | Filtry programu, katalogu a hostů se propisují do adresy. |
| **Guest checkout a krátký nákup** | Nucená registrace je jedna z hlavních příčin opuštění košíku (Baymard Institute). | Nákup jednotlivých vstupenek bez účtu ve 3 krocích s indikátorem kroků, cena přímo v tlačítku, Apple / Google Pay. |
| **Inline validace a lidské chybové hlášky** | Uživatel ví, co opravit. | „Zadejte e-mail ve tvaru jmeno@domena.cz, pošleme na něj vstupenky.“, fokus se přesune na chybné pole. |
| **Plynulé přechody (View Transitions API)** | Moderní „app-like“ pocit bez frameworku. | Přechod mezi stránkami jemně prolne obsah. Při `prefers-reduced-motion` se animace vypne. |
| **Integrace do nástrojů uživatele** | Diváci plánují v kalendáři v telefonu. | Export Mého diáře do `.ics` (Google / Apple / Outlook kalendář), tlačítko Wallet (simulace). |
| **Moderní CSS** | Méně kódu, lepší údržba. | Custom properties jako design tokeny, `color-mix()`, `:has()` (zvýraznění zvolené platby), `clamp()` pro plynulou typografii, `text-wrap: balance`, `env(safe-area-inset-*)` pro telefony s výřezem. |
| **Výkon a jednoduchost** | Rychlé načtení i na slabém festivalovém Wi-Fi. | Žádný framework ani knihovna, cca 110 kB včetně dat, funguje offline. |

## 3. Co se proti prototypu změnilo a proč

Struktura, navigace, obrazovky i průchod scénáři zůstaly stejné jako v prototypu 02 a wireframech 02. Změny:

| # | Změna | Důvod (validace / review) |
| --- | --- | --- |
| 1 | Vlastní modální okna nahrazena nativním `<dialog>`. | Kontrola přístupnosti: vlastní dialog nedržel fokus uvnitř a čtečka viděla i stránku pod ním. Při testu se navíc našla chyba, kdy po přihlášení zůstal otevřený prázdný dialog a blokoval stránku – opraveno. |
| 2 | Každá stránka má vlastní titulek (`<title>`) a změna stránky se ohlásí čtečce obrazovky. | Review: v prototypu měly všechny stránky stejný titulek, v historii prohlížeče a záložkách nešly rozlišit. |
| 3 | Přidána stránka **Kontakt & press** (kontakty, press kit, Industry, formulář). | Soulad se specifikací: sitemapa obsahuje *Press & Industry* a *Kontakt*, prototyp je neměl (cílová skupina 3 – novináři a profesionálové). |
| 4 | Nová **patička** s mapou webu a kontaktem místo poznámky „prototyp“. | Běžný vzor webu, druhá cesta k obsahu, který na mobilu je jen pod „Více“. |
| 5 | Export Mého diáře do kalendáře (`.ics`). | Specifikace mluví o „uložení do osobního kalendáře“; prototyp ukládal jen do prohlížeče. |
| 6 | Úprava barev ve světlém režimu (tmavší jantarová a tyrkysová) a tmavší pozadí iniciál hostů. | Automatická kontrola kontrastu (axe) hlásila poměr 3,2–4,0 : 1, požadováno 4,5 : 1. |
| 7 | Opravená hierarchie nadpisů (dny v programu, karty filmů, sálů a pasů jsou `h2`), stránka 404 má `h1`, tabulka vzdáleností má `caption` a `scope`. | Kontrola axe a html-validate. |
| 8 | Časová osa na mobilu se posouvá jen uvnitř svého rámečku, ne celá stránka. | Test responzivity: na šířce 390 px stránka scrollovala do strany o 728 px. |
| 9 | Menu „Více“ se zavírá klávesou Esc a klepnutím mimo; desktopové hledání má odesílací tlačítko pro čtečky. | Test ovládání klávesnicí a validace HTML (formulář bez submit tlačítka). |
| 10 | Plynulé přechody mezi stránkami, meta popis, ikona, `theme-color`, hláška `noscript`. | Doplnění k „reálnému“ webu, nemění strukturu ani obsah. |
| 11 | **Program a katalog filmů zobrazují max. 10 položek, další se načtou tlačítkem „Další“.** | Review (oponentura) po odevzdání: seznam 60 programových bloků a 42 filmů byl na mobilu příliš dlouhý a nepřehledný. |
| 12 | **Můj diář: filtr podle dne (Vše / Čt / Pá / So s počtem položek) a stránkování po 10.** | Tatáž review: náročný divák má v diáři desítky položek, potřebuje rychle vidět jen jeden den. Kolize a přesuny se dál počítají v rámci celého dne. Zvolený den je v URL (`#/diar?den=…`). |

Beze změny zůstává rozpor specifikace a dat (Litoměřice 2027 vs. Kutná Hora 2026). Web dál používá data, jak bylo zdůvodněno v progress-02.

## 4. Jaké testy a kontroly jsme provedli

| Test / kontrola | Nástroj a postup | Výsledek |
| --- | --- | --- |
| Průchod scénáři 1, 2, 3 | Automatizovaný průchod v Chromiu (Playwright) na mobilu 390 × 844: klikání přesně podle kroků scénáře, kontrola výsledku na každém kroku (počet výsledků filtru, text potvrzení, 2 vstupenky, hláška o přesunu, nabídka reprízy). | Všechny 3 scénáře projdou. Při testu nalezena chyba „visícího“ dialogu po přihlášení → opraveno. |
| Interaktivní funkce | Stejný test: validace e-mailu při nákupu, Esc zavře dialog, export `.ics` (obsah souboru zkontrolován, rezervace Pá 20:45), kontaktní formulář (chyba i úspěch), menu „Více“. | Všechny kontroly prošly. |
| Stránkování a filtr diáře (po review) | Test: program 10 → „Další“ → 20, fokus na 11. položce, po uložení do diáře zůstane 20, změna filtru vrátí 10; filmy 10 → 20; diář s 25 položkami 10 → 20, filtr Pá ukáže jen páteční položky (8 z 8). | 36 / 36 kontrol prošlo, axe stále 0 chyb. Na mobilu se tlačítka dnů v diáři nevešla na řádek → zalamují se. |
| Přístupnost | axe-core (pravidla WCAG 2.0/2.1/2.2 A + AA a best practices) na 17 adresách × mobil/desktop × tmavý/světlý režim. | Na začátku: kontrast, pořadí nadpisů, vnořené interaktivní prvky v mapě, chybějící `h1` na 404. Po opravách **0 chyb** ve všech 68 kombinacích. |
| Validita HTML | html-validate (doporučená pravidla) na zdrojovém `index.html` i na vykresleném DOM všech stránek. | Nalezeno: formulář bez odesílacího tlačítka, `th` bez `scope`, mezery v telefonním čísle → opraveno. |
| Responzivita | Na všech stránkách kontrola, že šířka obsahu nepřesahuje okno (390 px a 1280 px), screenshoty hlavních stránek. | Přetékání časové osy na mobilu opraveno, jinak bez vodorovného scrollu. |
| Chyby v konzoli | Zachytávání `console.error` a výjimek při všech testech. | Bez chyb. |
| Ovládání klávesnicí | Tab / Shift+Tab / Enter / Esc přes hlavičku, filtry, dialogy. | Fokus viditelný, dialog drží fokus a vrací ho na tlačítko, které ho otevřelo. |
| Kontrola obsahu proti datům | Tituly, časy, sály a hosté se v UI jen zobrazují z `data.js` (převod 1 : 1 z JSON). | Bez rozdílů. |

**Plánováno do další konzultace:** test se 3–5 uživateli (spolužáci), úkoly = scénáře 1 a 2, metoda think-aloud, sledujeme dokončení úkolu, čas a místa zaváhání. Výsledky zapracujeme v další iteraci.

## 5. Použití AI a ověření výstupů AI

- **Role autora:** zadání, cílové skupiny, scénáře, informační architektura a rozhodnutí o rozsahu změn vycházejí z naší specifikace (výstup 01) a z prototypu a wireframů (výstup 02). Autor rozhodl, že pro cvičení 4 se staví na prototypu 02, určil, co se má doplnit (kontakt/press podle sitemapy, export do kalendáře, opravy přístupnosti), a výstupy AI procházel a schvaloval.
- **S čím pomáhala AI:** Claude (Anthropic) pomáhal s úpravou kódu z prototypu (nativní dialog, titulky stránek, patička, stránka Kontakt & press, export `.ics`, přechody, opravy CSS), napsal automatické testy (Playwright, axe-core, html-validate) a připravil kostru tohoto dokumentu.
- **Jak jsme výstupy AI ověřovali:**
  - kód není přijat bez spuštění: každý scénář a každá funkce mají automatický test a byly proklikány v prohlížeči;
  - AI nástroje samy chyby nehledají spolehlivě, proto kontrola přístupnosti a validity proběhla nezávislými nástroji (axe-core, html-validate), ne „odhadem“ AI;
  - testy odhalily chyby i v kódu od AI (visící dialog po přihlášení, přetékání časové osy, nízký kontrast ve světlém režimu) – opraveny a znovu otestovány;
  - obsah (tituly, časy, sály, hosté) je jen načítán z dat, AI ho nepřepisovala; nový text (kontakty, press kit) je smyšlený a označený jako simulace.
- **Známá omezení:** přihlášení, platba, obsazenost sálů, QR kódy, PDF, Wallet a odeslání formuláře jsou simulované; diář a rezervace se ukládají jen do prohlížeče (localStorage). „Dnes“ na úvodní stránce je pevně nastaveno na pátek 16. 10. 15:30 kvůli scénáři 1. Anglická verze zatím není.
