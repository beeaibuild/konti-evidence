# Tíz további valós Viber-eset — mérés előtti válogatás

Állapot: első valós mérés és kézi review elkészült (2026.11.06.). A strukturált esetleírás:
`src/services/ai/interpretation/viberRealSelectionV1.json`. A külön futtatható suite neve `real-samples`;
az adatválogatást és az elvárásokat új modellmérés előtt rögzítettük.

Három korábban feltöltött képernyőképből, kézi vizuális ellenőrzéssel választottunk
10 esetet, összesen 11 üzenetet. A források eredeti fájlneve és SHA-256 lenyomata a
JSON-ban szerepel; a képek és a visszafejtési névtérkép nem kerülnek a repóba.
A személyek, helyek és ügyfélcímkék álnevesítettek; telefonszám és céges adat nincs bennük.
R04 márkanevének autómárka-jellege az álnévben megmarad; ebből önmagában nem következik,
hogy a közlés járműcseréről szól, és nem egy így nevezett ügyfél/hely konténercseréjéről.

## Rögzített elvárások

| Eset | Álnévesített bemenet | Tartalmi ellenőrzés |
| --- | --- | --- |
| R01 | „Jó reggelt késni fogok kb 8.00 körül érek be!” | Saját késés és hozzávetőleges beérkezés; nem biztos ETA vagy új feladat. |
| R02 | „Nádas kivittem” | Megtörtént kivitel közlése, nem jövőbeli kihelyezési utasítás. |
| R03 | „Hársalját elhoztam” | Megtörtént elhozatal; nincs bizonyított konténerazonosító vagy teli/üres állapot. |
| R04 | „Autómárka-B csere majd ma” | Ma később esedékes csere; nincs megnevezett végrehajtó vagy pontos óra. |
| R05 | „Nyírdombin nem a köz?” | Címet pontosító kérdés, nem automatikus címjavítás. |
| R06 | „Fizetve 1000” | Fizetési visszajelzés; a pénznem, rendelés, fizetési mód és címzett ismeretlen. |
| R07 | „Címke-A vigyem vissza??” | Visszaviteli kérdés, nem megtörtént visszavitel vagy jóváhagyott kiosztás. |
| R08 | „3 ked Fűzér utca 7 elhozni kint van” → „Betontéglát alólla eltenni az útról” | Elhozatal és helyszíni akadály eltávolítása; a „3 ked” nem bizonyított dátum/mennyiség, az „alólla” konkrét tárgya ismeretlen. |
| R09 | „Tudtam borítani,!” | Sikeres ürítés közlése; az érintett feladat nincs megnevezve. |
| R10 | „Vendel szálló reggel csere” | Reggeli csere adatainak sofőri közlése; nincs biztos naptári nap, óra vagy assignee. |

A közlők szerepét a projektben ismert identitások alapján rögzítjük. A diszpécser
saját végrehajtásról is beszámolhat: az operátori szerep nem változtat egy készjelentést
utasítássá. A két sofőr következetes álneve Bence és Tibi.

## Kontextus és bizonytalanság

Nem áll rendelkezésre a screenshot idejével egyező adatbázis-pillanatkép, ezért nincs
kitalált task-, order-, truck- vagy container-azonosító. A szövegben szereplő álnév
nem válik automatikusan ismert üzleti referenciává. A következő mérésben külön kell
kezelni a szöveg megértését és a strukturált referenciák feloldhatóságát.

Csak a látható órákat őrizzük meg; a naptári dátum és időzóna nem ismert.
A futtatható adaptáció nem tüntethet fel kitalált történeti timestampet igazolt adatként.
R09 előtt a képen egy másik sofőr helymegjelölése látszik: nincs reply vagy más
bizonyíték, ami az ürítéshez kötné. Ezért azt nem használjuk az érintett task kitöltésére.
R08 két egymást követő, azonos közlőtől érkező üzenete viszont együtt marad.

## Átfedés és mérési korlát

A 20 VBR-eset és K1–K8 szövegeitől eltérő közléseket választottunk. Kizártuk például
a „Sorrend… Ezt egyben”, „4m3”, „Akkor menjek…”, „Holnap 9…”, számlás screenshot,
feltételes visszameneteli utasítás és társalgási reakció már felhasznált példáit.
A művelettípusok átfedhetnek: új készjelentés attól még új eset, hogy korábban is
vizsgáltunk készjelentést. A puszta új álnév nem számít új példának.

Ez a gyűjtemény korábban már látott forrásanyagból származik, ezért nem független
holdout és nem reprezentatív üzemi pontosságbecslés. Az elvárások új modellválasz
megismerése előtt készültek. Nincs előírt rekordszám, kényszerített taxonómia vagy
új pontszám: a tartalmi, címkézési és jogosultsági eredményeket külön kell majd jelenteni.

## Futtatás és riport

```powershell
$env:KONTI_EVAL_REAL_PROVIDER = "false"
node --import tsx scripts/eval-konti-viber-v1.ts --suite real-samples `
  --artifacts "tmp/real-samples-offline.artifacts" --output "tmp/real-samples-offline.json"
node --import tsx scripts/eval-konti-viber-v1.ts --suite real-samples `
  --scripted "tmp/real-samples-offline.artifacts/predictions.json" --output "tmp/real-samples-replay.json"
```

A `tmp` szülőmappa legyen meg, az artifact célmappa legyen új. Alapértelmezésben ez
scripted próba: nem valódi modellmérés. A másik suite predikciócsomagját visszautasítja.
Azonosító: `konti-viber-real-samples-v1`, ellenőrzési verzió: `real-sample-boundaries-v3`.
A riport `evaluationScope=BOUNDARIES_DIRECTIVES_AND_PRESERVATION` és
`manualReviewStatus=PENDING` jelzése kötelező: a `passed` és `automaticChecksPassed`
a korlátokat és a meghatározott utasítás-/feltételcímke-ellenőrzéseket jelzi,
nem a tíz üzenet teljes tartalmi helyességét. Nincs új százalékos modellpontszám.

Az automatikus ellenőrzés a bizonyítékhivatkozásokat, a kitalált üzleti slotokat,
a beszélőnek az ismeretlen tárgy helyére behelyettesítését, a végrehajtási jogosultságot
és a nem igazolható naptári dátumokat vizsgálja. A megfigyelt kind és signal külön látszik;
nincs kötött rekordszám vagy egyetlen kötelező címke. A jelentést a rögzített elvárások
és a riportban megőrzött értelmezés alapján kézzel kell ellenőrizni.

A provider pontos bemenetében `UNKNOWN_SOURCE_CALENDAR_DATE`, illetve az ismert óra
és `DATE_UNKNOWN`/`TIMEZONE_UNKNOWN` jelölés szerepel. A történeti dátum nem kitalált ISO
időpont. A runtime jelenlegi validátora belül technikai helykitöltő ISO dátumot igényel;
ez nem kerül a modellhez, és a riport `SYNTHETIC_VALIDATOR_PLACEHOLDER` jelölése azonosítja.
A request `sourceCalendarDateKnown=false` jelzése megakadályozza, hogy a runtime
a helykitöltőből NOW/TODAY/TOMORROW dátumot számítson. A relatív szöveg megmarad,
a `resolvedAt` null. A mező hiánya vagy true értéke megtartja a korábbi működést.
A modell által kitalált explicit dátumot a temporal ellenőrzés továbbra is hibának jelzi.
A V2 korlátellenőrzés R04/R08/R10 esetén a nem igazolt végrehajtót is jelzi
(`UNSUPPORTED_PERFORMER`); nem korrigálja vagy rejti el a nyers modellválaszt.

Az export a pontos, dátumhiányt is jelölő bemenetet, szűrt válaszokat, predikciókat,
runId-t, inputHash-t és a teljes definíció lenyomatát őrzi. A manifest a JSON-adatkészletet
és a futtató forrását is hash-eli; SHA256SUMS ellenőrzi az export 23 fájlját. A megoldás
a meglévő provider-, szűrési és exportútvonalat használja, új függőség és production
szerződésmódosítás nélkül. Az eredeti V1 és K kontrollok változatlanok.

Az első valós mérés felhasználói jóváhagyással, a felhasználó PC-jén készült.
A javításhoz csak a mentett predikciók offline visszajátszását használjuk; új modellhívás és telepítés nincs.

## Offline ellenőrzés

271 célzott teszt sikeres 16 fájlban, köztük 7 új teszt a dátumhiányra, adatminimalizált
bemenetre, hibás válaszokra és valódi CLI export/replayre. A célzott strict TypeScript
és diff-ellenőrzés sikeres. A beépített scripted válaszok 10/10 korlátellenőrzést
teljesítenek; ez kizárólag a futtató próbája. Mind a 23 exporthash helyes, és kulcs nélkül,
tiltott fetch mellett a teljes új riport azonosan visszajátszható. A korábbi 20 és 8 eset
mentett válaszainak teljes riportja változatlan. A teljes alkalmazásellenőrzés a PR CI feladata.


## Első valós mérés és kézi review — 2026.11.06.

Forráscommit: `4ed086ca542d698d31c5cca4ff69ce1a4da5ee45`, modell:
`gpt-4.1-2025-04-14`, temperature 0. Bizonyítékcsomag:
`viber-real-samples-20261106-122516.zip`. Mind a 25 SHA256SUMS-bejegyzés helyes
(23 alapfájl + operátori manifest + replay); a teljes pontozási riport offline egyezik.
10/10 válasz értékelhető. A V1 automatikus eredmény 9/10, nem tartalmi pontosságbecslés.

| Eset | Kézi tartalmi értékelés |
| --- | --- |
| R01 | Megfelelő: saját késés, hozzávetőleges 8 órai beérkezés, kitalált feladat nélkül. |
| R02 | Megfelelő: megtörtént kivitel, konkrét üzleti azonosító nélkül. |
| R03 | Megfelelő: megtörtént elhozatal, teli/üres állapot kitalálása nélkül. |
| R04 | Részleges: ma esedékes csere, de MESSAGE_SENDER alapján indokolatlan végrehajtó. |
| R05 | Hibás: kérdés marad, de a „köz” közterületjelleg helyett „public area” jelentést kap. |
| R06 | Megfelelő: fizetési közlés, összeggel, kitalált pénznem vagy rendelés nélkül. |
| R07 | Megfelelő: engedélykérő visszaviteli kérdés, nem kiosztás. |
| R08 | Részleges: elhozatal és akadályeltávolítás megmarad, de a térbeli viszony megfordul, az utasítás INFORMATION. |
| R09 | Megfelelő: sikeres ürítés, konkrét feladathoz kötés nélkül. |
| R10 | Megfelelő: reggeli csere közlése, kitalált dátum vagy végrehajtó nélkül. |

R04-ben a modell TODAY/„ma” értéket adott, explicit dátum nélkül. A V1 runtime ebből
`2000-01-01T00:00:00.000Z` dátumot képzett a technikai helykitöltő alapján. Ez runtime-hiba,
nem a modell által kitalált dátum. A javított feloldó null dátumot tart meg; a V2 értékelő
ugyanakkor jelzi az indokolatlan actorRef-et. Így az automatikus eredmény továbbra is 9/10.
Csak R04 értékelt esete változik; mind a tíz nyers válasz változatlan.

R05-ben az eredeti válasz „public (köz) area”; R08-ban „The item to be picked up is under
concrete blocks”. Az utóbbi az „alólla” irányát fordítja meg. Ezeket a V2 strukturális
ellenőrzője sem képes tartalmi hibaként felismerni: a kézi review szükséges marad.
A három szűrt modellválasz külön regressziós fixture-ben marad meg
(`tests/fixtures/viber-real-observed-20261106.json`). A teszt bizonyítja, hogy a nyers hibák
megmaradnak auditálhatónak, és a korlátellenőrzés nem állít tartalmi megfelelést.
Ez a változtatás nem javítja a modell magyar nyelvi vagy utasításértelmezési hibáit.

A javítás ellenőrzése: 276 célzott teszt 16 fájlban, strict TypeScript, valamint a mentett
20 VBR- és 8 K-válasz teljes riportjának változatlan offline visszajátszása. A tíz új eset
mentett válaszai kulcs nélkül, tiltott fetch mellett újrapontozhatók. Teljes alkalmazás-CI
és Docker-build csak a jóváhagyás utáni pull requestben futtatható.


## Célzott nyelvi promptpontosítás — mérés előtt

A provider meglévő `meaning` és `semanticActionSignal` leírásai általános magyar
nyelvi támpontokat kaptak, az R05/R08 konkrét üzeneteinek vagy elvárt válaszainak
beillesztése nélkül. Címkérdésben a „köz” közterületjelleg; az „alólla” szóval jelölt
térbeli irányt meg kell őrizni, az ismeretlen referens azonosítóját nem szabad kitalálni.
A diszpécseri főnévi igenév kontextustól függően utasítás, a külön helyszíni művelet
nem puszta anyagadat és nem automatikusan új fuvarfeladat. Hiányzó üzleti azonosító
nem teszi a nyelvi utasítást egyszerű információvá, és nem pótolható kitalált adattal.

A két új, más szövegű szintetikus példa csak a helyes válasz downstream kezelését
ellenőrzi: nincs címjavítás/kiosztás egy kérdésből, a két utasítás bizonyítékai külön
maradnak, ismeretlen végrehajtóval a végrehajtás blokkolt. A 278 teszt és a mentett
38 válasz változatlan offline riportja nem bizonyítja a prompt modellre gyakorolt hatását.
R05/R08 kézi tartalmi hibastátusza változatlan, amíg új modellválasz nincs.


## Nyelvi újramérés — 2026.11.06. 18:11 (Europe/Budapest)

Forráscommit `3ae5d4e136195d01f2f07793b6d0f1672dbd5b90`, modell
`gpt-4.1-2025-04-14`, temperature 0. Csomag: `viber-language-20261106-181106.zip`.
Mind a 27 ellenőrzőösszeg helyes, az új riport teljes pontozása offline azonos.
A baseline predikciófájl byte-ra egyezik az első mérésével; mind a tíz provider-request
változatlan, a HTTP-kérés sémaleírásai tartalmazzák az új promptot.
Azonos V2 értékelővel a korlátellenőrzés 9/10 → 9/10, pontozhatatlan eset nincs.

| Eset | Új megfigyelés a korábbihoz képest |
| --- | --- |
| R01 | A késés és kb. 8 óra megmarad, de DELAY_OR_PROBLEM/PROBLEM_REPORT helyett STATUS_REPORT/STATUS_CHANGE_REPORT. |
| R02/R03 | A megtörtént kivitel/elhozatal jelentése megfelelő marad. |
| R04 | A nem igazolt végrehajtó eltűnt; a „ma” relatív marad, resolvedAt null. |
| R05 | A „köz” helyesen street type/suffix, kérdés marad, nem címjavítás. |
| R06 | Új SPEAKER_AS_UNKNOWN_OBJECT: a fizetési közlés subjectSlots mezőjébe speaker kerül. |
| R07/R09/R10 | A lényegi tartalom megmarad, kitalált üzleti azonosító nincs. |
| R08 | A betontégla van a hivatkozott tárgy alatt; az irány helyes. Mindkét teendő továbbra is INFORMATION. |

Ez célzott jelentésjavulást és új kötési/címkézési eltérést együtt mutat. Egyetlen
ismétlésből nem állítunk stabil összesített javulást. R08 utasításjelzésének hibáját
a V2 automatikus korlátellenőrző még nem jelezte; a későbbi V3 ezt már jelzi.

## Következő promptpontosítás — még nincs mérve

A rendszerszintű utasítás kifejezetten a hiányzó azonosítók, jogosultság és read-only mód
előtt kéri a beszédszándék felismerését. INSTRUCTION mellett lehet null actor és üres
subjectSlots: a végrehajtást a determinisztikus réteg ettől függetlenül blokkolja.
A névtelen fizetési/állapotközlés közlője önmagában nem bizonyítja, hogy ő a fizető,
címzett, végrehajtó vagy a közlés tárgya. Megnevezett, igazolt személy továbbra is köthető.
A meglévő provider-sémaleírások ezt ugyanígy rögzítik; új parser és sémamező nincs.

A legutóbbi R06/R08 szűrt válaszok a
`tests/fixtures/viber-language-observed-20261106.json` fixture-ben megmaradnak.
A tesztek a hibás raw outputot megőrzik; a V3 már az R08 hiányzó utasításjelzését is
jelzi, a jelentés teljes kézi review-ja továbbra is szükséges. Nem javítják át a modell válaszát. A névtelen és a kifejezetten megnevezett fizető két pozitív
szintetikus szerződéspéldája sem modellmérés.

281 célzott teszt és strict TypeScript sikeres. A mentett 20, 8, korábbi 10 és újabb
10 válasz külön offline újrapontozásának teljes riportja változatlan. Ehhez a már ellenőrzött
csomagok elmentett válaszait sorrendtartó `--scripted` tömbként adtuk át, kulcs nélkül,
tiltott fetch mellett. Ez válasz-újrapontozás, nem a megváltozott prompt mérése.
A bemenethez kötött szigorú replay helyesen elutasítja a régi rendszerpromptos csomagot;
ellenőrzése nincs lazítva. Eredeti futás hű visszajátszásához az eredeti commit szükséges.
Új mérésnél az új commiton exportált csomag ugyanott továbbra is visszajátszható.


## Harmadik valós mérés — 2026.11.07. 10:02 (Europe/Budapest)

Forráscommit `09123253607df793f52f3a7020e6ae020b5075e6`, modell
`gpt-4.1-2025-04-14`, temperature 0. Csomag: `viber-speech-act-20261107-100156.zip`.
Mind a 27 ellenőrzőösszeg helyes; a teljes offline replay egyezik. A baseline fájl byte-ra
azonos az előző méréssel. A rendszerpromptot kivéve a tíz provider-request változatlan.
V2 eredmény: 9/10, 0 pontozhatatlan. R06 már nem köt fizetőt vagy tárgyat a közlőhöz.
R08 második teendőjének térbeli iránya és INSTRUCTION jelzése helyes. Az elhozatal viszont
INFORMATION marad, MESSAGE_SENDER végrehajtóval; a második teendő CONDITIONAL_INSTRUCTION
címkéje mellett nincs conditionIntent. A végrehajtás BLOCKED_INCOMPLETE_BINDING.
R01 továbbra is STATUS_REPORT/STATUS_CHANGE_REPORT; R05 címszerep-értelmezése megmaradt.

## V3 értékelő és azonos szabályú offline összehasonlítás

Az R01–R10 fejlesztői halmaz; minden alábbi eredmény ezen mért eredmény.
A tartalékhalmaz eljárását a [külön protokoll](viber-holdout-protocol-v1.md) rögzíti.

Az előző részek V1/V2 számai történeti eredmények. A `real-sample-boundaries-v3`
kiegészíti az értékelőt; a modellprompt, az esetdefiníció és a nyers válaszok nem változnak.

- R08 mindkét forrásüzenetéhez kell legalább egy RESOLVED bizonyítékhivatkozású INSTRUCTION.
  Hiányában üzenetenként DIRECTIVE_NOT_RECOGNIZED jelzés keletkezik.
- Egy összevont értelmezés is lefedheti mindkét teendőt; külön értelmezések és külön
  tényközlő részletek is maradhatnak. Nincs kötelező rekordszám vagy pontos task-kind.
- CONDITIONAL_INSTRUCTION mellett hiányzó conditionIntent esetén külön
  CONDITIONAL_KIND_WITHOUT_CONDITION címkézési jelzés keletkezik.
- R08 „3 ked” és „alólla” kifejezése külön, a megfelelő forrásüzenetre mutató
  unresolvedSemanticReferences bejegyzést igényel. A puszta prózabeli említés nem elég.
  Hiányuk külön preservation hibát ad; a kifejezések jelentését nem találjuk ki.
- A componentChecks megőrzi az evidence/references/authority/temporal dimenziókat,
  és külön speechAct/taxonomy/preservation dimenziót ad. A componentFailures dimenziónként a hibás,
  pontozható esetek számát adja, nem hibakódszámot és nem pontosságszázalékot.
  A pontozhatatlan eseteket továbbra is külön kell számolni a caseResults-ból.

| Mentett futás | Közös V3 megfelelés | Kötési hiba | Utasításjelzési hiba | Feltételcímke-hiba | Megőrzési hiba (R08) |
| --- | --- | --- | --- | --- | --- |
| Első, 2026.11.06. 12:25 (`4ed086c`) | 8/10 | R04 | R08 mindkét üzenet | nincs | „3 ked”, „alólla” |
| Nyelvi, 2026.11.06. 18:11 (`3ae5d4e`) | 8/10 | R06 | R08 mindkét üzenet | nincs | „3 ked” |
| Beszédszándék, 2026.11.07. 10:02 (`0912325`) | 9/10 | R08 | R08 első üzenet | R08 második értelmezés | „3 ked”, „alólla” |

Ez ugyanazon három tízes válaszhalmaz utólagos, közös értékelése; nem új modellmérés,
nem független holdout és nem üzemi pontosságbecslés. Mindháromnál nulla evidence-,
authority- és temporal-hibás eset van az alkalmazott automatikus ellenőrzések szerint.
A V3 sem vizsgálja teljesen a szabad szöveges jelentést, például a „köz” fordítását vagy
a térbeli irány természetes nyelvi megfogalmazását; a kézi review továbbra is szükséges.

A legutóbbi R08 szűrt válasza új fixture-ben marad:
`tests/fixtures/viber-speech-act-observed-20261107.json`. A célzott tesztek ellenőrzi az
összevont/külön utasításokat, az üzenetenkénti lefedést, a hibás bizonyítékot, a feltétel
nélküli címkét és a három megfigyelt válaszminta változatlan megőrzését. Az érintett
korábbi tesztek az új hibajelzéseket várják, nem törlik a régi modellhibákat.

293 célzott teszt 18 fájlban és strict TypeScript sikeres. Az első két futás mentett,
ellenőrzött válaszait tömbként pontoztuk újra, mert a promptjuk a jelenlegitől eltér.
A harmadik futás a szigorú, bemenethez kötött replayvel is működik: a V3-export predikciói
JSON-tartalmukban az eredeti csomagéval azonosak. Mind a 23 új exporthash helyes,
és az új teljes riport kulcs nélkül, tiltott fetch mellett azonosan visszajátszható.
A korábbi 20 VBR- és 8 K-válasz teljes újrapontozott riportja is változatlan.
Új modellhívás, promptmódosítás és telepítés ebben az értékelőjavításban nincs.
