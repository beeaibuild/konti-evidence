# Viber tartalékhalmaz — 18 eset, előzetes review

> **Nyilvános, adatvédelmi célból módosított másolat.** Ez nem a 2026.11.08-i fagyasztott fájlok bájtpontos publikációja. Lásd: [PUBLIC-COPY-NOTE.md](PUBLIC-COPY-NOTE.md).

Állapot: **ÖSSZEÁLLÍTVA / JÓVÁHAGYÁSRA VÁR; NINCS MODELLEREDMÉNY**. Készült: 2026-11-08T10:14:14.626Z.

Fagyasztott kód: `59b33772012fc0ed999a340835ad8a249a93b3c6`. Modell a későbbi méréshez: `gpt-4.1-2025-04-14`, temperature 0. A prompt és a pontozó nem módosult.

## Rögzített értékelési szabály

Három teljes, külön azonosított futás. Egy eset csak akkor sikeres, ha mindháromban átmegy az automatikus és az előzetesen rögzített kézi ellenőrzéseken. 1/3 vagy 2/3: INGADOZÓ, nem siker. Kézi review nélkül PENDING. Egy-két esetnyi eltérés nem igazolt javulás. Célzott kilenc eset és egyéb kilenc eset külön jelentendő. Három futás nem 54 független minta.

## Kiválasztás és korlátok

18 valódi eseményablak, 4 munkanap (D+0, D+3, D+4, D+5; D+0 péntek, D+1–D+2 hétvége), 3 sofőr és 1 összevont diszpécserazonosság. A korábbi tervezési becslés öt nap volt; az időrendi kvóták négy napnál beteltek, ezért nem cseréltünk későbbi eseményre. Egy nap vagy elsődleges küldő sem adja az esetek több mint felét.

3 főnévi igenévi utasítás; 2 személytelen pénzügyi feljegyzés + 1 személytelen állapotjelzés; 3 névmásos/tárgyi hivatkozás; 9 egyéb. A halmaz szándékosan rétegzett, nem véletlen és nem a teljes üzemmenet gyakoriságait reprezentálja. Új sofőrökre nem ad bizonyítékot.

Időrend és előre rögzített kvóták alapján készült, modellválaszok nélkül. A forrás mind a 139 üzenetének beválasztása/kizárása és oka szerepel a jegyzékben. A névmási kvóta korábbi Megyek én is esettel kezdődik; nem toltuk későbbi, tetszetősebb példára.

A friss hét a kódfagyasztás után érkezett. Az események korábbi fejlesztési felhasználásának hiányát a tulajdonosnak még jóvá kell hagynia; pusztán a feltöltés időpontja nem bizonyít függetlenséget. Ismétlődő nevek és munkatípusok önmagukban nem új események, de nem is bizonyítanak eseményazonosságot.

A modell kizárólag az álnevesített szöveget, ismert küldőket és órát kapja; naptári dátumot, képet, nyers forrást vagy elvárt választ nem. Dátum a jegyzékben mintavételi adat. A képek csak forrás- és reply-ellenőrzéshez szolgáltak. Hiányzó képből nem követelünk meg vizuális tényt.

A required jelzések kötelezőek, az allowed másodlagos aktusokat enged. A fagyasztott séma nem tud vagylagos kötelező aktust: ezért a névszói diszpécserfeladatok INSTRUCTION elvárása annotációs konvenció, nem vitathatatlan nyelvészeti igazság. Az ilyen hibát külön kell jelenteni; futás után nincs átírás. Az unresolved töredék strukturált, bizonyítékhoz kötött jelzés, nem pusztán szó szerinti említés a válaszban.

Az azonosító/hely/pénzügyi adatok álnevesítettek; a visszafejtési térkép és a nyers képek nincsenek a csomagban. A forráshash eredethez köt, nem tanúsítja a valóságtartalmat. Helyesírási hibát nem javítottunk és tényt nem egészítettünk ki.

## Esetek

| Eset | Réteg | Nap | Elsődleges küldő |
|---|---|---|---|
| H01 | OTHER | D+0 | Gergő |
| H02 | OTHER | D+0 | Zoli |
| H03 | OTHER | D+0 | Gergő |
| H04 | PAYMENT | D+0 | Zoli |
| H05 | OTHER | D+0 | Diszpecser |
| H06 | OTHER | D+3 | Gergő |
| H07 | PRONOUN | D+3 | Robi |
| H08 | INFINITIVE | D+3 | Diszpecser |
| H09 | OTHER | D+3 | Diszpecser |
| H10 | OTHER | D+3 | Diszpecser |
| H11 | OTHER | D+3 | Diszpecser |
| H12 | OTHER | D+3 | Gergő |
| H13 | INFINITIVE | D+4 | Diszpecser |
| H14 | INFINITIVE | D+4 | Diszpecser |
| H15 | STATUS | D+4 | Robi |
| H16 | PRONOUN | D+4 | Robi |
| H17 | PAYMENT | D+5 | Zoli |
| H18 | PRONOUN | D+5 | Diszpecser |

## H01 — OTHER

**H01_m001 · 14:19 · Gergő**

Viszem a horgonyt

Előzetes kézi elvárások:

- **C01:** A saját szállítás folyamatban lévő/tervezett cselekvés, nem befejezett fuvar. A horgony megnevezésből nem található ki ügyfél-, konténer- vagy feladatazonosító.

Automatikus aktus-elvárások:

- `H01_m001`: kötelező PLAN; megengedett PLAN, INFORMATION.

Tiltott cselekvő-hozzárendelés: nincs külön eset-specifikus tiltás; az általános biztonsági szabályok érvényesek.

## H02 — OTHER

**H02_m002 · 14:28 · Zoli**

Szabóért megyek!

Előzetes kézi elvárások:

- **C01:** A küldő indulási szándéka jelenjen meg. A Szabó megnevezés lehet személy vagy hely/partner; típusa, címe és az elvégzendő művelet nem egészíthető ki találgatással.

Automatikus aktus-elvárások:

- `H02_m002`: kötelező PLAN; megengedett PLAN, INFORMATION.

Tiltott cselekvő-hozzárendelés: nincs külön eset-specifikus tiltás; az általános biztonsági szabályok érvényesek.

## H03 — OTHER

**H03_m004 · 15:49 · Gergő**

Lacikának adtam 76 000-et

**H03_m005 · 15:49 · Gergő**

Szevasztok

**H03_m006 · 15:49 · Gergő**

Jó hétvégét

Előzetes kézi elvárások:

- **C01:** Explicit személyes átadás: a küldő az átadó, Lacika az említett címzett; Lacika nem azonosítható automatikusan Robi sofőrrel. A 76 000 összeg és a már megtörtént átadás maradjon meg; pénznem nincs kimondva.
- **C02:** A köszönés és a hétvégi elköszönés nem fuvarutasítás, nem rendeléslezárás és nem új pénzügyi tranzakció.

Automatikus aktus-elvárások:

- `H03_m004`: kötelező STATUS_CHANGE_REPORT; megengedett STATUS_CHANGE_REPORT, INFORMATION.
- `H03_m005`: kötelező NONE; megengedett NONE.
- `H03_m006`: kötelező NONE; megengedett NONE.

Tiltott cselekvő-hozzárendelés: nincs külön eset-specifikus tiltás; az általános biztonsági szabályok érvényesek.

## H04 — PAYMENT

**H04_m007 · 15:51 · Zoli**

285000 Robi.

Előzetes kézi elvárások:

- **C01:** Személytelen összeg–név közlés: a 285000 és Robi megmarad, de nem állítja, hogy ki fizetett kinek, milyen jogcímen, mikor vagy milyen pénznemben. A küldőből nem lesz automatikusan fizető.
- **C02:** Ez bizonytalan pénzügyi feljegyzés, nem igazolt befizetés és nem rendelés fizetetté állítása.

Automatikus aktus-elvárások:

- `H04_m007`: kötelező INFORMATION; megengedett INFORMATION.

Tiltott cselekvő-hozzárendelés: H04_m007

## H05 — OTHER

**H05_m008 · 16:02 · Diszpecser**

Mindenki irány haza ha a szállítóleveleket leadtátok csak akkor elszámolást is. 🤗

Előzetes kézi elvárások:

- **C01:** A hazamenetelhez a szállítólevelek leadásának feltétele és az elszámolás említése is megmaradjon; a feltétel teljesülése nem tény.
- **C02:** A mindenki csoportos megszólítás nem egy név szerint kitalált sofőr. A küldő nem lesz végrehajtó; sem hazamenetel, sem elszámolás nem tekinthető megtörténtnek.

Automatikus aktus-elvárások:

- `H05_m008`: kötelező INSTRUCTION; megengedett INSTRUCTION, INFORMATION.

Tiltott cselekvő-hozzárendelés: nincs külön eset-specifikus tiltás; az általános biztonsági szabályok érvényesek.

## H06 — OTHER

**H06_m010 · 06:02 · Robi**

Lehet hogy egy picit kések

**H06_m011 · 06:46 · Gergő**

Én is késni fogok lerobbant egy ismerősöm Remény körúton megyek oda mizu utána jövök be köszi

Előzetes kézi elvárások:

- **C01:** Két sofőr késésjelzése külön maradjon; az első bizonytalan, kis késést jelez. A második saját késését és előbb egy ismerőséhez történő odamenetelt közli, pontos érkezés nélkül.
- **C02:** Az ismerőshöz kapcsolt lerobbanás nem bizonyít céges teherautóhibát. Ne keletkezzen kitalált járműhiba, időtartam vagy feladat-hozzárendelés.

Automatikus aktus-elvárások:

- `H06_m010`: kötelező PROBLEM_REPORT; megengedett PROBLEM_REPORT, INFORMATION, PLAN.
- `H06_m011`: kötelező PROBLEM_REPORT; megengedett PROBLEM_REPORT, PLAN, INFORMATION.

Tiltott cselekvő-hozzárendelés: nincs külön eset-specifikus tiltás; az általános biztonsági szabályok érvényesek.

## H07 — PRONOUN

**H07_m012 · 07:09 · Gergő**

Beértem indulok a grófokra

**H07_m013 · 07:20 · Robi**

Megyek én is.

Előzetes kézi elvárások:

- **C01:** A Beértem megtörtént érkezés, az indulok saját indulási terv; a kettő nem olvadhat teljesített fuvarrá.
- **C02:** A Megyek én is saját cselekvésre utal. Az én a küldőhöz kötött, de az előző sofőr célhelyének azonossága nem biztos; az is többletét/utalását meg kell őrizni, célhelyet nem szabad bizonyossággá emelni.

Automatikus aktus-elvárások:

- `H07_m012`: kötelező PLAN, STATUS_CHANGE_REPORT; megengedett PLAN, STATUS_CHANGE_REPORT, INFORMATION.
- `H07_m013`: kötelező PLAN; megengedett PLAN, INFORMATION.

Tiltott cselekvő-hozzárendelés: nincs külön eset-specifikus tiltás; az általános biztonsági szabályok érvényesek.

## H08 — INFINITIVE

**H08_m014 · 08:13 · Diszpecser**

Homok utca csere kis kárpitos Plnav  terdex kft

**H08_m015 · 08:17 · Diszpecser**

Elhozni s visszavinni

**H08_m016 · 08:30 · Zoli**

Mi az a kis kárpitos ?

**H08_m017 · 08:32 · Gergő**

Omega Box hatos van az udvaron kis Kárpitos ott fest

**H08_m018 · 08:37 · Diszpecser**

Juharosi út csere mikor megy ki!

**H08_m019 · 08:37 · Diszpecser**

8m3

**H08_m020 · 08:38 · Diszpecser**

Völgyhát kft szállító

**H08_m021 · 08:43 · Zoli**

Egyszerre kettő nem megy !!!!

**H08_m022 · 08:44 · Robi**

Ha borítottam megyek a juharosira

**H08_m023 · 08:44 · Zoli**

Akkor én megyek a homokba!

Előzetes kézi elvárások:

- **C01:** Az Elhozni s visszavinni két műveletű utasítás, név szerint megadott végrehajtó nélkül; nem elkészült szállítás.
- **C02:** A kis kárpitos megnevezés bizonytalanságát és a rá vonatkozó kérdést meg kell őrizni. Az Omega Box hatos és a másik címhez tartozó 8m3 nem olvadhat egyetlen konténerré.
- **C03:** A mikor megy ki kérdés maradjon kérdés. Az Egyszerre kettő nem megy kapacitásprobléma; nem végrehajtási jóváhagyás.
- **C04:** Robi csak a borítás után tervezi a juharosi utat; Zoli saját homoki terve külön szerepeljen. Nincs automatikus üzleti kötés vagy állapotmódosítás.

Automatikus aktus-elvárások:

- `H08_m014`: kötelező INSTRUCTION; megengedett INSTRUCTION, INFORMATION.
- `H08_m015`: kötelező INSTRUCTION; megengedett INSTRUCTION, INFORMATION.
- `H08_m016`: kötelező QUESTION; megengedett QUESTION.
- `H08_m017`: kötelező INFORMATION; megengedett INFORMATION, STATUS_CHANGE_REPORT.
- `H08_m018`: kötelező QUESTION; megengedett QUESTION, INSTRUCTION, INFORMATION.
- `H08_m019`: kötelező INFORMATION; megengedett INFORMATION.
- `H08_m020`: kötelező INFORMATION; megengedett INFORMATION.
- `H08_m021`: kötelező PROBLEM_REPORT; megengedett PROBLEM_REPORT, INFORMATION.
- `H08_m022`: kötelező PLAN; megengedett PLAN, INFORMATION.
- `H08_m023`: kötelező PLAN; megengedett PLAN, INFORMATION.

Tiltott cselekvő-hozzárendelés: H08_m014, H08_m015, H08_m018, H08_m019, H08_m020

## H09 — OTHER

**H09_m024 · 09:48 · Diszpecser**

Holnap reggel vinni: 

Budapest 14 kerület Szilfa köz 9 kérek egy 8 m3 konténert

**H09_m025 · 09:48 · Diszpecser**

[TELEFON TÖRÖLVE]

Előzetes kézi elvárások:

- **C01:** Holnap reggeli 8 m3-es kivitel, a közölt címre; relatív idő maradjon, abszolút dátum nem ismert a modell számára.
- **C02:** A beágyazott kérek ügyféligény továbbítása lehet: a továbbító nem bizonyított eredeti megrendelő. A törölt telefonból nem szabad új számot vagy ügyfelet létrehozni.

Automatikus aktus-elvárások:

- `H09_m024`: kötelező INSTRUCTION; megengedett INSTRUCTION, INFORMATION.
- `H09_m025`: kötelező INFORMATION; megengedett INFORMATION.

Tiltott cselekvő-hozzárendelés: H09_m024, H09_m025

## H10 — OTHER

**H10_m026 · 10:38 · Diszpecser**

15:00 Rét bükkèr utca csere 8m3

**H10_m027 · 10:38 · Diszpecser**

Kp

Előzetes kézi elvárások:

- **C01:** 15:00, csere, 8m3 és készpénzes mód együttesen maradjon meg. A Kp nem bizonyít megtörtént fizetést vagy befizetőt.
- **C02:** A név és az utca szöveges feladatleírás, nem ellenőrzött üzleti azonosító vagy sofőr-hozzárendelés. A névszói diszpécserközlés elvárt domináns aktusa INSTRUCTION.

Automatikus aktus-elvárások:

- `H10_m026`: kötelező INSTRUCTION; megengedett INSTRUCTION, INFORMATION.
- `H10_m027`: kötelező INFORMATION; megengedett INFORMATION.

Tiltott cselekvő-hozzárendelés: H10_m026, H10_m027

## H11 — OTHER

**H11_m029 · 12:08 · Diszpecser**

Csempemester csere 4m3

Előzetes kézi elvárások:

- **C01:** 4m3-es csere a Csempemester megnevezéshez kapcsolva; méret és művelet maradjon meg, végrehajtó és kanonikus ügyfél nem ismert. A névszói diszpécserközlés előre rögzített domináns aktusa INSTRUCTION.

Automatikus aktus-elvárások:

- `H11_m029`: kötelező INSTRUCTION; megengedett INSTRUCTION, INFORMATION.

Tiltott cselekvő-hozzárendelés: H11_m029

## H12 — OTHER

**H12_m030 · 14:58 · Gergő**

Holnap reggel a Diófa csere

Előzetes kézi elvárások:

- **C01:** A sofőr holnap reggeli Diófa-cserét közöl; ez terv, nem már teljesített csere. Sem abszolút naptári dátum, sem ellenőrzött cím/feladatazonosító nem található ki.

Automatikus aktus-elvárások:

- `H12_m030`: kötelező PLAN; megengedett PLAN, INFORMATION.

Tiltott cselekvő-hozzárendelés: nincs külön eset-specifikus tiltás; az általános biztonsági szabályok érvényesek.

## H13 — INFINITIVE

**H13_m044 · 07:43 · Diszpecser**

Horgony utca 31 elhozni ma

Előzetes kézi elvárások:

- **C01:** A ma elhozandó egység helye Horgony utca 31. Az elhozni utasítás, nem sofőr saját terve vagy elvégzett feladat. Cselekvő hiányzik, nem ismeretlen slotot kell konstruálni.

Automatikus aktus-elvárások:

- `H13_m044`: kötelező INSTRUCTION; megengedett INSTRUCTION, INFORMATION.

Tiltott cselekvő-hozzárendelés: H13_m044

## H14 — INFINITIVE

**H14_m045 · 08:25 · Diszpecser**

Cédrus pál utca platóssal menni

Előzetes kézi elvárások:

- **C01:** Cédrus pál utcához platós járművel menni: a járműtípus-korlát megmarad, de konkrét jármű és végrehajtó nincs megadva. Nem található ki konténerméret, rakomány vagy befejezés.

Automatikus aktus-elvárások:

- `H14_m045`: kötelező INSTRUCTION; megengedett INSTRUCTION, INFORMATION.

Tiltott cselekvő-hozzárendelés: H14_m045

## H15 — STATUS

**H15_m046 · 08:42 · Robi**

348 as konténer elég gatya

**H15_m047 · 09:09 · Robi**

348 as konténer, forgalomból kivonva.

Előzetes kézi elvárások:

- **C01:** A 348-as konténer rossz állapotáról szóló jelzés és az utána közölt forgalomból kivonás két eltérő állítás. Nem járműről van szó.
- **C02:** A kivonva személytelen állapotközlés: nem a küldőnek tulajdonított kivonási művelet, és nem végrehajtási jogosultság. A 348-as említés nem ellenőrzött adatbázis-azonosító.

Automatikus aktus-elvárások:

- `H15_m046`: kötelező PROBLEM_REPORT; megengedett PROBLEM_REPORT, INFORMATION.
- `H15_m047`: kötelező STATUS_CHANGE_REPORT; megengedett STATUS_CHANGE_REPORT, INFORMATION.

Tiltott cselekvő-hozzárendelés: H15_m046, H15_m047

## H16 — PRONOUN

**H16_m058 · 15:34 · Diszpecser**

Holnap 12:00 után 

Gyuri csere

**H16_m059 · 16:04 · Robi**

Az ki az és hol😂

**H16_m060 · 16:36 · Gergő**

Nem tudom!

**H16_m061 · 16:37 · Gergő**

Talán a Kelet utca ahogy nézem rogyásig Van rakva

**H16_m062 · 17:16 · Zoli**

Szerintem lehet a gesztenyés ház, a bükkér utcàban!

**H16_m063 · 17:35 · Robi**

Akkor mindenki tudja 😂😂😂

**H16_m064 · 17:40 · Zoli**

Csak találgatok!

**H16_m065 · 18:54 · Diszpecser** · válasz: H16_m062

Igen de csak 12:00h után

Előzetes kézi elvárások:

- **C01:** Az Az ki az és hol kérdés a Gyuri-csere személy/hely hivatkozásának tisztázatlanságát jelzi. A Nem tudom és a két tipp nem biztos helyazonosítás.
- **C02:** A Talán, Szerintem lehet és Csak találgatok bizonytalanságjelölése megmaradjon. Az Akkor mindenki tudja ne váljon tényszerű közös tudás vagy diszpécserutasítás állításává.
- **C03:** A 18:54-es diszpécserválasz a 17:16-os gesztenyés ház/bükkér utcai javaslatot idézi (képen ellenőrzött reply). Ennek megerősítése és a csak 12:00 után korlátozás megmarad; a korábbi találgatás nem visszamenőleg biztos tény.
- **C04:** A holnap relatív idő marad, nem kitalált naptári dátum; külső helyazonosító és végrehajtó nem ismert.

Automatikus aktus-elvárások:

- `H16_m058`: kötelező INSTRUCTION; megengedett INSTRUCTION, INFORMATION.
- `H16_m059`: kötelező QUESTION; megengedett QUESTION, INFORMATION.
- `H16_m060`: kötelező INFORMATION; megengedett INFORMATION.
- `H16_m061`: kötelező SUGGESTION; megengedett SUGGESTION, INFORMATION.
- `H16_m062`: kötelező SUGGESTION; megengedett SUGGESTION, INFORMATION.
- `H16_m063`: kötelező NONE; megengedett NONE, INFORMATION.
- `H16_m064`: kötelező INFORMATION; megengedett INFORMATION.
- `H16_m065`: kötelező INFORMATION; megengedett INFORMATION, INSTRUCTION.

Tiltott cselekvő-hozzárendelés: nincs külön eset-specifikus tiltás; az általános biztonsági szabályok érvényesek.

## H17 — PAYMENT

**H17_m070 · 08:01 · Zoli**

45000 Gergő,hétfőn!

Előzetes kézi elvárások:

- **C01:** 45000, Gergő és hétfőn szerepel: nem derül ki, hogy fizetés, átadás vagy terv, ki fizetett kinek és milyen pénznemben. A küldő nem válhat fizetővé.
- **C02:** A hétfőn nem kap kitalált abszolút dátumot; nincs rendelés fizetetté állítása vagy automatikus pénzügyi művelet.

Automatikus aktus-elvárások:

- `H17_m070`: kötelező INFORMATION; megengedett INFORMATION.

Tiltott cselekvő-hozzárendelés: H17_m070

## H18 — PRONOUN

**H18_m071 · 10:02 · Diszpecser**

Holnap 8:00 XIV.Remény krt.12 keran kft vinni

**H18_m072 · 10:04 · Diszpecser**

6m3 ami kint van

**H18_m073 · 10:04 · Diszpecser**

Eladva

Előzetes kézi elvárások:

- **C01:** A holnap 8:00, kivitel, 6m3 és a kint lévő tárgyra utaló ami kint van maradjon meg. A kint pontos helye és az egység azonosítója nincs megalapozva.
- **C02:** Az Eladva állapotközlés nem bizonyítja a kifizetést, kiszállítást vagy bérletlezárást. A méretet/helyhivatkozást nem lehet egy másik konténerhez önkényesen kötni.
- **C03:** A XIV.Remény krt.12 és a keran kft szöveges célmegnevezés; nincs ismert üzleti kötés, végrehajtó vagy abszolút naptári dátum.

Automatikus aktus-elvárások:

- `H18_m071`: kötelező INSTRUCTION; megengedett INSTRUCTION, INFORMATION.
- `H18_m072`: kötelező INFORMATION; megengedett INFORMATION.
- `H18_m073`: kötelező STATUS_CHANGE_REPORT; megengedett STATUS_CHANGE_REPORT, INFORMATION.

Tiltott cselekvő-hozzárendelés: H18_m071, H18_m072, H18_m073
Strukturált feloldatlansági jelzés: H18_m072: ami kint van

## Következő lépés

Tulajdonosi tartalmi/adatvédelmi és eseményfüggetlenségi jóváhagyás, majd külön adatcommit és javító PR. Közzététel és fizetős futás most nem történt. A korábbi R01–R10 továbbra is fejlesztői halmaz; a rajta mért 9/10 nem általánosítási eredmény. Ha a csomag megismerése miatt prompt/pontozó/rubrika változik, ez a halmaz fejlesztői adattá válik.
