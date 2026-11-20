# Konti — fejlesztési napló

## 2026.11.20. — GPS-javítások és dokumentációs állapot egyeztetése

**Ellenőrzött forrás:** main `53fc1ff97b9e392b48ba7aa24bf25bf581d17fd2` (#213).
A legutóbbi alkalmazáskód-változás továbbra is #212 (`46c470b`).
Ez GitHub-forrás- és PR-ellenőrzés; új tesztfuttatás, Render-ellenőrzés vagy fizikai tabletpróba nem történt.

- [#209](https://github.com/beeaibuild/kontener-app/pull/209) nyitott, mainre céloz;
  head `d74f0fe`. Az automatikus 17:00-s vagy igazolt halasztásos lezárás utáni GPS-sor kezelése.
- [#214](https://github.com/beeaibuild/kontener-app/pull/214) november 19-én létrehozott,
  nyitott javítás a #210 kemény lejárati esetére; head `e98ba0c`.
  Célága a #209 ága (`fix/ack-post-cutoff-gps-queue-20261117`), nem a main.
- #208 továbbra is nyitott; az útvonal-előnézeti teszt javítása nincs a mainben.

**#214 viselkedése:** a szerver megjelöli az általa automatikusan lezárt műszakokat.
Kemény lejárat után csak a hitelesített tablet saját műszakához és a hozzá tartozó,
azonos lejárati időpontban visszavont sofőrmunkamenethez érkező pontok nyugtázhatók tárolás nélkül.
A lejáratig mért érvényes pontok tárolódnak. A kézi kilépés utáni pontok elutasítása megmarad.

**Migráció:** új `GpsShift.serverClosed` mező; a régi sorok alapértéke false,
mert a történeti lezárási ok nem igazolható. A javítás nincs összevonva a mainbe.

**Tesztbizonyíték határa:** a #214 PR leírása PostgreSQL 18.6-on sikeres migrációt,
TypeScript-ellenőrzést és 37/37 GPS-/tabletintegrációs tesztet rögzít.
A tesztek HTTP-végpontokon vizsgálják a küldési sorrendet, az `acceptedIds` választ,
vegyes GPS-csomagot, újraküldést, kézi kilépést és másnapi belépést.
Az SQLite-sor ürítése modellezett; fizikai tabletpróba még hátravan.
Ezeket a futásokat e dokumentációs frissítésben nem ismételtük meg.

**Következő lépés:** #209 rendezése, majd #214 mainre célzása és az aktuális mainhez
tartozó kötelező ellenőrzések. A merge, migráció/élesítés és fizikai átvétel külön lépések.
Az utolsó dokumentált production-ellenőrzés november 17-i (`46c470b`);
a lentebbi deploy- és nyitott tabletpróba-adatok történeti bizonyítékok, nem november 20-i új igazolások.

## 2026.11.17. — README és aktuális kiadási állapot egyeztetése

**Ellenőrzött forrás:** main `46c470b277cdf3c4f95f6722e88e80cafb650067` (#212).
A GitHub PR-állapotok és a Render deploy-nyilvántartás alapján frissítve.
Ez dokumentációs ellenőrzés; új alkalmazásteszt vagy fizikai tabletpróba nem történt.

| Változás | Igazolt állapot |
| --- | --- |
| #203 — 17:00-s tabletes kiléptetés | Összevonva 2026.11.17. 01:37:42-kor (Budapest), merge `1adbadd`. A jelenlegi éles kiadás tartalmazza. |
| #205 — main ágvédelem és CI-besorolás | Összevonva, merge `27e86a2`; az aktiválás bizonyítékát a következő korábbi bejegyzés rögzíti. |
| #207 — kézi P63 élesítési kapu | Összevonva, merge `35eb5db`; pontos main SHA, kötelező checkek és azok forrásának ellenőrzése a Render-hívás előtt. |
| #211 — regisztrált partnerek aktivitása | Összevonva, merge `86b14d4`; a rendelés nélküli regisztrációkat is tartalmazó admin kimutatás. |
| #212 — aznapi/másnapi AI-ajánlásszám | Összevonva, merge `46c470b`; a meglévő minták külön kártyát és pontosított mutatófeliratot kaptak. |
| #208 — hidratált űrlap útvonaltesztje | Nyitott PR, nincs a mainben. |
| #209 — automatikus zárás utáni GPS-sor nyugtázása | Nyitott PR, nincs a mainben és a most ellenőrzött éles kiadásban. |

**Éles kiadás:** Render `dep-dbu3fvh42hec73bfj280`, commit `46c470b`,
LIVE 2026.11.17. 12:21:50 (Budapest). Előző kiadás: #211, deploy
`dep-dbu3a1942hec73bf3ptg`, 12:08:01-kor elkészült; az újabb kiadás leváltotta.

**Viselkedés:** a #203 budapesti 17:00-kor zárja a sofőrtabletes műszakot.
Az akkor már futó normál vagy belső fuvarnál a zárás 30 percnyi, 200 méteren belüli
GPS-megállásig halasztott, tartós adatbázis-állapottal. A kemény sessionhatár
a műszak következő 17:00-s határideje után 12 óra; a korábban kiadott cookie-k
az eredeti lejáratukat viszik tovább. Az admincsatornák külön munkamenetek.
A #204 szerinti aznapi sofőr- és teherautó-elérhetőség kijelentkezés után is megmarad.

**Partnerkimutatás:** a közvetlen ügyfélfiók-kapcsolat elsőbbséget kap;
ennek hiányában csak egyértelmű rendeléskori email-egyezés alapján számol.
Új adatbázis-migráció nincs.

**AI-mérés:** a #212 megjelenítési pontosítás, nem új modellmérés vagy pontosságjavulás.
Az elfogadási és végrehajtási mutatók másnapra szóló ajánlásokra vonatkoznak;
az aznapi ajánlások száma és részletei külön szerepelnek.

**Nyitott következő lépések:** #209 kötelező kapuinak és összevonhatóságának rendezése;
első éles 17:00-s zárás és következő reggeli belépés ellenőrzése;
fizikai Samsung APK-frissítés, háttér-GPS és push visszaigazolása.
A kemény lejáratú GPS-sor kezelése a #209-től is külön feladat.
A lentebbi november 16-i „#203 még nyitott” bejegyzések történeti állapotok.

## 2026.11.17. — Aktív main ágvédelem

Budapesti idő szerint 00:06-kor a GitHub már védettként jelezte a main ágat. A #205 PR összevonását a futó P63 mellett blokkolta. Az öt sikeres ellenőrzés után a GitHub engedélyezte az összevonást. [P63-futás](https://github.com/beeaibuild/kontener-app/actions/runs/43974406829) · [Ágvédelmi szabály](https://github.com/beeaibuild/kontener-app/settings/branches)

## Egyedi éles rendelésjavítások

A partner és a kihelyezési cím javítását a rendelés adatlapján végezd bejelentkezett adminisztrátorként. Ugyanezek a paraméterezett végpontok használhatók repón kívüli műveleti scriptből. A bemeneti JSON és a munkamenet sütije maradjon a repón kívül. A már alkalmazott migrációk tartalmát és checksumját meg kell őrizni.

| Művelet | Ellenőrzés mentés előtt | Az adatbázisban kereshető esemény |
| --- | --- | --- |
| `POST /api/admin/orders/{id}/partner` | A várt partner és számlázási mód egyezzen. Számla vagy fizetési előzmény blokkolja a javítást. | `ORDER_PARTNER_CORRECTED` |
| `POST /api/admin/orders/{id}/placement` | Az aktuális rendelésverzióval először kérj árat. Mentéskor add meg az elfogadott árat a `confirmedPrice` mezőben. | `ORDER_PLACEMENT_CORRECTED` |

A sikeres módosítás és az `AuditLog` bejegyzése közös tranzakcióban történik. Az auditban az `entityId` az érintett rendelés, a `userId` a hitelesített végrehajtó, a `createdAt` a művelet ideje. A `data` a partner és számlázási mód, illetve a cím, konténer és ár előtti és utáni értékeit tartalmazza. A puszta árlekérés nem ment módosítást. Ha a szerver ütközést jelez, olvasd újra a rendelést, mielőtt új paraméterekkel folytatod. A GitHub-naplóba elég a művelet típusa és az auditbejegyzés hivatkozása.

## 2026.11.16. — Ágvédelmi próba a #205-ön

A [teljes P63 futásban](https://github.com/beeaibuild/kontener-app/actions/runs/43935632909)
a besoroló 14/14 tesztje átment, az érintett workflow-fájlok teljes kört választottak.
A [külön CI](https://github.com/beeaibuild/kontener-app/actions/runs/43935632960), a teljes Vitest és
a szállítólevél E2E sikerült. A P63 böngészős csomagjában 172 teszt átment, öt elbukott.
Az összesítő a hibás P63 miatt szintén piros lett a `09766d4` commiton. A sofőrchat
megnyitása ezen a #203 nélküli kódon is elbukott. A közös gyökérok továbbra is nyitott.
A próba idején a `main` ágvédelme még inaktív volt.

## 2026.11.16. — A main ágvédelmének előkészítése

A README és a fejlesztési napló frissítése is saját PR-on, CI-kihagyó jelölés nélkül érkezik.
A normál PR és az éles incidens alatti kivétel lépéseit, a két kötelező check pontos nevét
és az egyhetes költségmérés módját [az ágvédelmi eljárás](main-merge.md) rögzíti.
Az aktiválás időpontja és a GitHubon ellenőrzött blokkolás külön naplóbejegyzést kap a beállítás után.

## 2026.11.16. — Tabletes belépésből aznapi elérhetőség (#204)

**Viselkedés:** sikeres új tabletes belépés hozzáadja a sofőrt és a tablet teherautóját a budapesti
naptári nap beosztásához. Meglévő beosztáshoz hozzáad; hiányzó tervet létrehoz, üreset kiegészít.
Kijelentkezés és műszakzárás után is megmarad a napi elérhetőség.
Jövőbeli napokat nem ír át, a kézi előretervezés megmarad.

**Megvalósítás:** a meglévő operatív tranzakció és napi advisory lock; duplikációvédett join-insert,
before/after audit `source=DRIVER_TABLET_LOGIN` jelöléssel, napi chat-tagság egyeztetése.
A sikertelen jelszó vagy műszakkonfliktus nem módosít elérhetőséget.
Azonos kérésazonosítójú retry nem ad hozzá ismét, így későbbi kézi kivételt sem von vissza.
Új adatbázisséma, függőség és APK-verzió nincs.
A terv létrejöttével a meglévő napi kiosztási korlát lép életbe; nem bejelentkezett
sofőr/autó kézzel továbbra is kijelölhető.

**Ellenőrzés:** 9 célzott unit teszt PASS mockolt tranzakcióval/adatbázissal: új/üres/meglévő terv,
megőrzés, retry, sikertelen belépés, sessionhiba miatti rollback, kijelentkezés,
téli/nyári budapesti napváltás. Két valódi DB-s integrációs regresszió hozzáadva:
párhuzamos belépések/idempotencia/kijelentkezés, valamint elutasított belépések.
A közös fixture-takarítás a belépésből keletkező napi tagságokat is kezeli.
A megváltozott TS-fájlokban nincs 140 karakternél hosszabb új sor.

**Kiadási kapu:** 2347/2347 unit teszt (207 fájl), 77/77 valódi PostgreSQL-integráció
(sofőrtablet, napi beosztás és kiosztási korlát), typecheck, production build és sorhosszkapu PASS.
Az eredeti PR 1171/1171 forrásblobja hash szerint egyezett a helyi forrással.
Az integrációs futás feltárta a chat-tagságok FK-védett takarítási hiányát;
a #204 második commitja (`316e6ba`) csak az üzenet nélküli teszttagságokat takarítja.
Az összes migráció lefutott az izolált PostgreSQL 18.6 tesztadatbázison; az éles verzió 16.
A teljes unit kör első futása hibásan rögzített APP_ORIGIN miatt 25 Origin-eltérést adott;
a normál tesztbeállítással újrafuttatott teljes kör 2347/2347 PASS.
Nincs új GitHub Actions-futás, függőség, migráció vagy APK.
**Élesítés:** #204 merge `c37e2eaad78c89eb7f92d1c5b58434a6ae8fc570`.
Az összevont és a PR-head fájlfája azonos (`e87f553`).
Render `dep-dbtjipgn74is73coqvr0`: LIVE 2026.11.16. 18:14:44 (Budapest).
Pre-deploy: 60 migráció, nincs függő migráció. 18:15:35-kor mind a production domain
mind a közvetlen Render-host cache-bypass health próbája HTTP 200, status/database ok,
és pontosan a merge SHA. Az első LIVE utáni lekérés átmeneti 503-at adott; az ismételt
ellenőrzés mindkét hoston sikeres. A vizsgált indulás utáni naplóablakban nincs error.
**Nyitott:** fizikai tabletes belépési próba. A már nyitott sessionöket nem szakítottuk meg;
az első új sikeres belépés jelöli a sofőrt és a teherautót a napra.

## 2026.11.16. — Rendeléskezelés, közös chat és GPS-állásriasztás

**Forrásállapot:** main `f8bb9f6`, #194–202 összevonva. A bejegyzés a PR-ek és commitok
ellenőrzésére épül; nem új production- vagy teljes regressziós teszt.

- #194–195: teljes cím a rendeléslistában, alapból budapesti mai dátum, csökkenő rendelésszám.
- #196–197: azonos diszpécseri email nem von össze külön kézi megrendelőket; kereshető
  partnercsere és új partner, rendelésenkénti elszámolás, tranzakciós audit.
- #198–199: látható magánszemély/cég választó, cégnév, adószám, kapcsolattartó;
  a céges partnerazonosítás adószám alapján történik. Állandó „Minden sofőr” beszélgetés.
- #200–201: kihelyezés címének, méretének és konténerének javítása. Mentés előtti
  szerveroldali ár-/útvonal-előnézet, újraellenőrzés, foglaláscsere, feladatfrissítés,
  értesítési esemény és audit egy tranzakcióban.
- #202: 07–17 óra (Budapest), aktív GPS-műszak, folyamatos mérésekkel igazolt 30 perc állás
  rögzített 200 m-es körben; hang/adminüzenet, GPS-lista és térképjelölés.
  A riasztás nyitott adminfelületet igényel.

**Nyitott:** #203 (`driver-1700-cutoff`) még nincs összevonva. A tervezett szabály:
17:00-s zárás, az akkor aktív normál/belső fuvarnál az első igazolt 30 perces,
200 méteren belüli megállásig halasztva; tartós DB-állapot. A PR szerint a korábbi sessionök
éjféli határideje megmarad. Integrációs és fizikai tabletpróba hátravan.
A #194–202 production-telepítését ez a frissítés nem igazolja újra.

## 2026.11.15. — Aláírt Konti Sofőr 2.0.2 és frissítési út

#193 mainbe került: `0799944`, a tesztelt fejjel azonos fájltartalom.
A chatben megosztható linkről letölthető, meglévő production tanúsítvánnyal aláírt
2.0.2 (versionCode 5) APK; az új app a frissítőoldalt külső böngészőben nyitja meg.
Nincs új migráció vagy függőség.

**Ellenőrzés a PR szerint:** 2330/2330 helyi unit teszt, typecheck, production build,
sorhosszkapu, Android unit/release build, aláírás, zipalign és manifest PASS.
Production deploy `dep-dbsrr2lg1s2s73818m30`: LIVE 2026.11.15. 15:13:32 (Budapest).
A 15:14-es cache-bypass próbában mindkét hoston health/database/commit, letöltőoldal,
APK MIME, 3664046 bájtos méret és SHA-256 egyezés igazolt. Függő migráció nem volt.
Új Actions-futás nem indult.

**Nyitott:** fizikai Samsung 2.0.1 → 2.0.2 frissítés; párosítás, belépés, feladatok,
háttér-GPS és push. A szerveroldali letöltési próba nem igazolja a tabletfrissítést.

## 2026.11.13–15. — Rendelésfelvétel és telemetria: korábbi helyi állapot frissítése

A lentebbi november 13-i „helyi / nincs merge” leírások történeti köztes állapotok.
#188 végleges, személyes Konti-alapú rendelésfelvétele `f4d4bcf` alatt mainbe került.
A merge rögzített bizonyítéka: azonos tesztelt fa, CI/P63 3586 Vitest és 177 UI teszt,
production- és Docker-build PASS; additív audit-eseménytípus migráció.
Az operátor ellenőrzi a javaslatot; automatikus rendelésmentés nincs.

#190 (`ae7d0b9`) és #192 (`e5bb974`) összevonva: heti fagyasztott telemetria,
append-only tárolás és SHA-256, majd `telemetry-snapshot/2` első kiosztási döntésekkel
a kiosztási auditból, csak VERIFIED hivatkozással. A korábbi fagyasztások változatlanok.
A heti cron aktuális üzemelése és a legutóbbi éles kimutatás tartalma most nem volt ellenőrizve.

## 2026.11.14. — Residency: két mérési történet és a bizonyítékok hozzáférhetősége

**Mit/miért:** két kapcsolódó kutatási eset forrásjegyzéke: téves evaluator-riasztások, majd változatlan konfiguráció mellett ingadozó baseline; a pályázati állításokhoz elérhető bizonyíték kell.
**1. Értékelőhiba:** [baseline-riport](../reports/2026-09-10-baseline-7398002.md) és [javítási riport](../reports/2026-09-11-postfix-975e45a9.md): 19/19 kritikus jelzés, dokumentált téves szereplő-/sorrend-/feltételértékelések; utómérés 20/20 pontozható, 0 kritikus, 50,63125 pont. A korábbi 40,8355 ponttal való különbség nem tiszta evaluator-hatás: friss modellválaszok és sémaillesztési javítás is volt. A huszadik eredeti eset dátumot adott ISO datetime helyett; a meglévő pass-szabály 100 pont és 0 kritikus hiba.
**2. Ingadozás:** [hétfutásos riport](../reports/2026-09-11-baseline-variability-975e45a9.md): közölt 7×20 eset, 45,70–51,45 pont, átlag 48,77, mintaszórás 2,22 pont; az 50,63125 az R1. VBR-15 két futásban kritikus jelzést kapott: a feltétel szövege megmaradt, de a benne hivatkozott feladat hiányzott a subject-hivatkozások közül. Változatlan evaluator és modellalias mellett backend-fingerprint nincs; az ingadozás backend/numerikus/sztochasztikus eredete nem választható szét.
**Bizonyítási határ:** ezek visszakereshető történeti riportok, nem most függetlenül rekonstruált nyers futások. A #57/#58 alatti korábbi bizonyítékhiány megállapítása érvényben marad; a riportban szereplő hash önmagában nem pótolja az eredeti fájlt. A ChatGPT-linkek nyilvános, bejelentkezés nélküli elérhetősége nincs igazolva.
**Nyitott a residency beadása előtt:** anonimizált, önálló bizonyítékcsomag (riportok, ellenőrzött kóddiffek és tesztek, elérhető eredeti bemenet/válasz/konfiguráció, hashjegyzék); hiányzó nyers fájlok explicit jelölése; tartós nyilvános URL és bejelentkezés nélküli megnyitás ellenőrzése. A privát repó teljes története és nyers ügyfélchat nem publikálandó.
**Kutatási kérdés:** hogyan különíthető el a viselkedésváltozás, az értékelő változása és a futások közötti ingadozás? Harmadik, zajsávhoz viszonyított javulást e bejegyzés nem állít igazoltnak; új modellmérés vagy közzététel nem történt.

## 2026.11.13. — A végleges rendelésfelvételi irány helyi megvalósítása

A korábbi külön hangos mezőkinyerés és determinisztikus szöveges panel helyett közös, AI-alapú
rendelésfelvétel készült. A mikrofon csak leiratot ad; a javítható szöveg személyes Konti-szálba kerül.
A strukturált javaslat a meglévő AI adaptert és a tartós generálási/idempotencia-kezelést használja.
A forrás és az emberi javítás auditált, a hiányzó mezők üresek, és csak sikeres mentés tanítja a kontextust.
A visszajelzés két külön, azonos partnerhez és címhez tartozó mentés egyezése után használható kapcsolati adatként.
A személyes szálhoz idegen felhasználó téves tagság esetén sem fér hozzá. Meglévő operátori közös szálak megmaradnak.
Részletek, korlátok és ellenőrzési bizonyíték: [rendelésfelvétel](order-capture-v1.md).
Az Actions-keret 100%-os: új távoli CI-futást nem indítunk, nincs költségkeret-/workflow-módosítás vagy telepítés.


## 2026.11.13. — Szöveges rendeléspiszkozat integrációja a mikrofonos ágra

- A #188 mikrofonos űrlapátadása megmarad; mellé külön szöveges előnézet és privát @Konti-forráslink kerül.
- Az előnézet nem alkalmaz adatot automatikusan. Külön áttöltés és szerkesztés után megerősítés kell;
  hiányzó adat nem örököl korábbi mezőt/alapértéket. Teljes válaszséma-ellenőrzés, abort és kérésazonosság-védelem.
- Település-/irányítószám-ellentmondás, eltérő közterülettípus és 12/A–12/B eltérés nem köthet régi címhez.
  Partner–cím eltérésnél a cím nem töltődik. Tagadás/javítás kézi ellenőrzést kér; „cserép” nem csere.
- Csere/ürítés/elszállítás csak meglévő rendelési hivatkozást ad, új rendeléshez áttöltés nincs.
- A read-only végpont friss OPERATOR-jogosultságot, same-origin védelmet, 16 KiB streamkorlátot,
  4000 karakteres szöveghatárt és felhasználónként 20 kérés/perces limitet alkalmaz.
  Törzsadatok csonkolásakor nincs partner-/címkövetkeztetés; üzenet csak kanonikus láthatósággal olvasható.
- Célzott React/szolgáltatás/HTTP tesztek, valódi helyi DB-s HTTP-próba: PASS. A Chromium helyben nem indul,
  ezért a meglévő adminShellNav csomag új böngészőtesztje a végleges CI-ben ellenőrizendő.
- Modellhívás, rendelésmentés, új függőség/migráció és workflow/költségkeret-módosítás nincs.
  Merge és élesítés nem történt. A végleges commit ellenőrzésének részletes eredménye a PR-ben szerepel.

## 2026.11.12. — Mikrofonos kézi rendelés előkészítése

- A közös admin Kézi rendelés CTA mellé mikrofon kerül mobilon, tableten és desktopon.
- Böngészős hangrögzítés → jogosultságvédett OpenAI-feldolgozás → meglévő rendelési űrlap.
  Nincs automatikus mentés, partnerkötés, rendelésmódosítás vagy hangarchiválás.
- A hiányzó adatok üresek, a már szerkesztett űrlap cseréje megerősítést kér.
  Felhasználóhoz és egyszeri tokenhez kötött, tíz percig érvényes böngészős átadás.
- 39 célzott ellenőrzés zöld, beleértve auth/CSRF, feltöltéskorlát, megszakítás,
  engedélyverseny és űrlapkitöltés. A teljes helyi unit futás 2246 tesztje zöld volt;
  egy sorhosszhibát javítottunk, az érintett kapu újrafuttatása is zöld.
  Typecheck és production build zöld. Mind az 55 migráció alkalmazható külön helyi PostgreSQL 18-on.
- Valódi helyi HTTP-próba: a kezdeti belépő dokumentumon és adminoldalon microphone=(self);
  voice API anonim 401, idegen Origin 403, hiányzó providerkulcs 503. Providerhívás nem történt.
  A policy dokumentumszintű: pusztán adminútvonalra engedélyezve a belépés utáni SPA navigáció tiltást örökölne.
  A mikrofon böngészőengedélyhez, a feldolgozás továbbra is operátori jogosultsághoz kötött.
- A helyi Chromium a futtatókörnyezet socketkorlátozása miatt nem indul; ez nem sikeres UI-próba.
  A szintetikus mikrofonos, providerhívás nélküli böngészős próbák a meglévő P63 adminShellNav tesztjébe kerültek.
  GitHub CI/P63 és fizikai Samsung-/élő providerpróba még ellenőrizendő; élesítés nem történt.
- Az Admin APK 1.1 mikrofonengedélyét a külön #172 PR tartalmazza. Új függőség és migráció nincs;
  workflow- és költségkeret-módosítás nincs. A kész webes változathoz egyetlen GitHub-futási kör a cél.

## 2026.11.08. — Nyitott megrendelések címrövidítése, helyi előkészítés

**Mit:** operátori címkeresés a nyitott megrendelésekben, a napi feladat nélküli kinti egységekhez is.
**Miért:** a műveletet a szövegből értjük, a rövid címet a jogosult üzleti adatokhoz kapcsoljuk.
**Adatvédelem:** csak meglévő üzenetrészlet válhat aliassá; új teljes cím/ügyféladat nem jut a providerhez.
**Bizonyíték:** [működés és korlátok](viber-order-address-context-v1.md), 335 offline teszt / 21 fájl,
benne 23 új, mockolt DB→provider→feloldó próbákkal. Részleges jelöltkörnél nincs biztos címkötés.
**Nyitott:** közzétételi jóváhagyás, teljes alkalmazás-CI/P63, majd külön engedélyezett élő ellenőrzés.
A történeti mérés változatlan; új providerhívás, függőség, migráció és telepítés nincs.

## 2026.11.09. — #166 frissítése a jelenlegi mainre

A main újabb V3 értékelője és megőrzési/feltétel-ellenőrzései megmaradnak;
a #166 üzenetenkénti utasításlefedési regressziói az aktuális hibakódokat ellenőrzik.
Az újabb mérési eredményeket nem írjuk felül a régi PR összesítésével.
Az öt fájl végi felesleges üres sor eltávolítva. A P63 minden jobja azonos,
rögzített PR-headet ellenőriz; a whitespace-kapu az explicit PR-base és head
közti eltérést vizsgálja. A workflow-változás teljes P63-at választ.
A teljes CI/P63 eredménye a futás befejezéséig függőben; telepítés nincs.

## 2026.11.08. — Tartalékhalmaz-mérés és célzott szerződésjavítás, helyi állapot

**Eredmény:** 3×18 valós válasz, automatikusan 6/18, 5/18, 5/18; rögzített rubrikával
4 stabil / 2 ingadozó / 12 FAIL. 246 hash és 54 eset offline újrapontozása egyezik.
**Miért:** az utasítás/cselekvő és kontextusvesztési hibák mellett annotációs viták is látszanak;
a tartalmi review asszisztensi, a gold nem változik. A 18 eset mostantól fejlesztői adat.
**Javítás:** [címrövidítés és feltételes címke](viber-reference-contract-v2.md) determinisztikus
ellenőrzése; általánosított prompt, új réteg/függőség nélkül. 312 offline teszt és célzott strict TS PASS.
**Nyitott:** emberi review, közzétételi jóváhagyás, CI/P63, később új fagyasztás és új tartalékhalmaz.
Új providerhívás vagy telepítés nincs; az éles rendelésaliasok betöltése nincs ezzel igazolva.

## 2026.11.08. — Előre annotált, 18 esetes Viber-tartalékhalmaz

**Mit:** a #169 utáni `59b33772012fc0ed999a340835ad8a249a93b3c6` kódfagyasztáshoz
18 új eseményablak, 43 álnevesített üzenet és 35 előzetes kézi elvárás készült.
**Miért:** a korábbi R01–R10 fejlesztői halmazon mért 9/10 nem általánosítási bizonyíték.
**Bizonyíték:** [esetek és rubrika](../data/benchmarks/viber-holdout-v1/REVIEW.md),
[forrásjegyzék](../data/benchmarks/viber-holdout-v1/source-ledger.json), [offline ellenőrzés](../data/benchmarks/viber-holdout-v1/validation.json).
4 munkanap, 3 sofőr, időrendi kvóták; 25 fagyasztott fájl változatlan. A nyers forrás és névtérkép nem kerül Gitbe.
**Nyitott:** három azonos konfigurációjú valódi futás és kézi review; 3/3 siker kell, ingadozás külön jelentendő.
Nincs új modellhívás, telepítés vagy igazolt minőségi eredmény. Közzététel felhasználói jóváhagyással.

## 2026.11.07. — Adatfüggetlen tartalékhalmaz-futtató, offline előkészítés

Új, külön CLI fogad 15–20 esetet, előre megírt jelzés-, cselekvő-, feloldatlanhivatkozás-
és kézi tartalmi elvárásokkal. A generikus pontozóban nincs R06/R08-azonosítós szabály.
Ellenőrzi a kiválasztási kvótákat és a bizonyítékhivatkozásokat; az elvárások nem jutnak
el a providerhez. A prompt, a korábbi R/VBR/K pontozók és a feloldó nem változnak.
A freeze-jegyzék teljes commitot és forráshasheket tartalmaz; új mintakiválasztás csak
utána történhet. A CLI ellenőrzi a fagyasztott kódot és a kiválasztás időrendjét.
Az export megőrzi a pontos bemeneteket, szűrt válaszokat, predikciókat, forrásjegyzéket,
freeze-jegyzéket és hasheket. Valós futásnál a pontos HTTP törzs is megmarad.
Az összesítő három külön eredeti futást és riporthoz kötött kézi review-t kér, majd
kulcs nélkül újrapontozza a csomagokat. 3/3 PASS, 1/3–2/3 UNSTABLE, 0/3 FAIL;
PENDING review nem siker. A célzott és az egyéb csoport külön szerepel.
301 célzott teszt 19 fájlban és strict TypeScript PASS. A CLI-t ideiglenes git checkoutban,
API-kulcs nélkül, tiltott fetch mellett is vizsgáltuk: export/replay, felülírás tiltása,
kódelmozdulás, sérült artifact és teljes háromfutásos összesítés. Az összes tesztadat
szintetikus; a három „eredeti” futás tesztfixture, nem valódi API-futtatás.
Teljes CI még nincs. Fagyasztási jegyzék, új valódi tartalékhalmaz, modellhívás és telepítés
nem készült. A korábbi 9/10 továbbra is kizárólag fejlesztői halmazon mért eredmény.
Részletek: [futtató](viber-holdout-runner-v1.md), [protokoll](viber-holdout-protocol-v1.md).

## 2026.11.07. — Fagyasztás előtti diagnosztika és mérési protokoll

Az R01–R10 ezentúl kifejezetten fejlesztői halmaz; a korábbi 9/10 eredmény megmarad,
fejlesztői halmazon mért automatikus korlátellenőrzésként. Nem általánosítási eredmény.
A V3 a „3 ked” és az „alólla” strukturált, megfelelő üzenethez kötött megőrzését is kéri.
Az üres utasítástárgy-lista SUBJECT_UNRESOLVED; a nem létező, megnevezett slot UNKNOWN_SLOT.
Az actorRef null önmagában megengedett, a végrehajtási blokkolás változatlan.
A tartalékhalmaz-protokoll előre rögzíti a napok, küldők és üzenettípusok kvótáit,
a három ismétlésből kötelező 3/3 sikert, és az 1/3–2/3 ingadozó esetek külön jelentését.
A korábbi három promptváltozat nem helyettesíti ezt az ismétléses vizsgálatot.
A prompt nem változik; új tartalékhalmaz nincs kiválasztva, fagyasztási commit még nincs.
A Windows ZIP hordozható másolatában a tartalom változatlan, 27 hash helyes, unzip exit 0.
293 célzott teszt 18 fájlban és strict TypeScript PASS. Kulcs nélküli, tiltott fetch
melletti teljes offline replay PASS, 23 exporthash helyes. A 20 VBR- és 8 K-kontroll
azonos pontozóval újrapontozott riportja változatlan. Négy további webes tesztfájl
a részleges helyi környezet hiányzó Next/alkalmazásmoduljai miatt nem indult el; teljes CI még nincs.
Részletek: [protokoll](viber-holdout-protocol-v1.md), [csomagolás](viber-portable-artifacts.md).

## 2026.11.07. — Harmadik tízes mérés és V3 utasítás-/címkeellenőrzés

A #165 mainverzióján (`0912325`) elkészült a harmadik tízes mérés ugyanazzal a modellel,
temperature 0-val: V2 szerint fejlesztői halmazon mért 9/10, 0 pontozhatatlan, mind a 27 hash és a teljes offline
replay helyes. R06-nál már nincs kitalált fizető/tárgy. R08 akadályeltávolítása INSTRUCTION,
de az elhozatal INFORMATION és a közlő indokolatlanul végrehajtó; a feltételes címkéhez
nincs feltétel. A determinisztikus réteg a végrehajtást blokkolja.
Az új `real-sample-boundaries-v3` külön ellenőrzi az R08 két üzenetének bizonyítékhoz
kötött utasításjelzését, valamint a feltételes címke feltételének jelenlétét. Összevont
és külön értelmezés is elfogadható; nincs kötelező rekordszám vagy egyetlen task-kind.
A `componentFailures` dimenziónként hibás esetszámot ad, nem állít teljes tartalmi
pontosságot. A három mentett futás közös V3-mal 8/10 → 8/10 → fejlesztői halmazon mért 9/10; a raw válaszok
változatlanok. A legújabb export szigorúan, kulcs nélkül visszajátszható, 23 alapfájl-hash
helyes. 289 célzott teszt 18 fájlban és strict TypeScript sikeres. Új modellhívás,
promptmódosítás és telepítés ebben a javításban nincs; a CI a PR után ellenőrizhető.

## 2026.11.06. — Nyelvi újramérés és a közlő/tárgy, utasítás/jogosultság szétválasztása

A #164 mainverzióján (`3ae5d4e`) ugyanaz a tíz eset újra lefutott, ugyanazzal a modellel
és temperature 0-val. fejlesztői halmazon mért 9/10 korlátellenőrzés, 0 pontozhatatlan; 27 hash helyes, teljes
kulcs nélküli replay-egyezés. R04 már nem talál ki végrehajtót, R05 érti a „köz” címszerepét,
R08 térbeli iránya helyes, de az utasítások INFORMATION jelzése megmaradt. R06 a közlőt
a fizetés tárgyává is teszi; R01 késésének jelentése megmarad, de a problémajelzés
STATUS_REPORT/STATUS_CHANGE_REPORT címkére váltott. Ez vegyes eredmény, nem stabil
összesített javulás bizonyítéka.
A rendszerprompt ezután kifejezetten az azonosítók/jogosultságok előtt kéri a beszédszándék
felismerését. A névtelen fizetési/állapotközlés közlője önmagában nem fizető/címzett/tárgy.
Nincs új parser vagy jogosultsági szabály. Két megfigyelt válasz új fixture-be került;
névtelen és megnevezett fizetővel is van szintetikus offline teszt. 281 célzott teszt
és strict TypeScript sikeres. Utólag ellenőrizve: a #165 merge commitján
(`0912325`) a [teljes CI](https://github.com/beeaibuild/kontener-app/actions/runs/42267225649)
zöld; a PR headjén a [P63 required checks](https://github.com/beeaibuild/kontener-app/actions/runs/42262979097)
és a teljes PR CI is zöld. A négy korábbi futás válaszainak offline újrapontozása
változatlan; a szigorú replay az eltérő rendszerprompt miatt helyesen elutasítja a régi
bemenetcsomagokat. Új modellhívás és telepítés nincs; a prompt hatása még nem mért.

## 2026.11.06. — Magyar címpontosítás és helyszíni utasítások promptja

A meglévő provider-sémaleírás pontosítja, hogy címkérdésben a „köz” közterületjelleg,
az „alólla” a hivatkozott tárgy alól eltávolítandó dologra utal, és a rövid „elhozni”,
„eltenni” főnévi igenév is kifejezhet utasítást. A helyszíni teendő nem pusztán
anyagjellemző, és önmagában nem új fuvar. Hiányzó végrehajtót/tárgyat nem szabad kitalálni.
Nincs új parser, függőség vagy sémamező. Két új szintetikus offline példa ellenőrzi,
hogy a címkérdésből nem lesz kiosztás, a két utasítás bizonyítéka külön megmarad,
és ismeretlen végrehajtóval a végrehajtás blokkolt. Ez nem modellmérés.
278 célzott teszt 17 fájlban és strict TypeScript sikeres; a mentett 20, 8 és 10 eset
pontozási riportja változatlan. A nyers R05/R08 hibák megmaradnak; javulást csak egy
külön engedélyezett valós mérés igazolhat. Új providerhívás és telepítés nem történt.

## 2026.11.06. — Valós Viber-mérés review és ismeretlen dátum javítása

A tíz új eset egyszeri valós mérése `gpt-4.1-2025-04-14` modellel elkészült:
fejlesztői halmazon mért 9/10 automatikus korlátellenőrzés, 10/10 értékelhető válasz, teljes offline riportegyezés.
Kézi review: hét eset lényegi tartalma megfelelő; R04 indokolatlan végrehajtót jelöl,
R05 félreérti a közterületjelleget, R08 megfordítja az „alólla” térbeli viszonyát és
utasítás helyett információt ad. Ez nem 90%-os tartalmi pontosság.
A runtime R04 TODAY értékéből helykitöltő dátumot képzett. Opcionális, rendszeroldali
`sourceCalendarDateKnown=false` mellett ezentúl nincs relatív dátumfeloldás; a régi
hívások működése megmarad. V2 korlátellenőrzés jelzi a nem igazolt végrehajtót.
A mentett tíz válaszon fejlesztői halmazon mért 9/10 marad, csak R04 értékelt eredménye változik; a nyers válaszok
érintetlenek. Három megfigyelt hibás válasz regressziós fixture-be került, a nyelvi hibák
kézi review-t igényelnek. 276 célzott teszt és strict TypeScript sikeres; a korábbi 20 és
8 eset teljes riportja változatlan. Új providerhívás vagy telepítés nincs.
[Részletes review és korlátok](viber-real-selection-v1.md).

## 2026.11.05. — Tíz további valós Viber-eset futtatása előkészítve

Korábbi chatképek kézi vizuális ellenőrzéséből 10 további eset / 11 üzenet készült,
álnevesítve, a már használt szövegek kizárásával. A `real-samples` suite a meglévő
provider- és exportutat használja; pontos bemenet, szűrt válasz, predikció és a JSON
definíció forráshash-e is megmarad. A hiányzó történeti dátumot a modell felé explicit
ismeretlenként jelzi; a runtime technikai helykitöltője nem eredeti időadat.
Az automatikus korlátellenőrzés és a kézi tartalmi review külön marad.
271 célzott teszt 16 fájlban és célzott strict TypeScript sikeres; CLI export/replay
23 ellenőrzőösszeggel ellenőrizve. A korábbi 20 és 8 eset teljes riportja változatlan.
Új modellhívás, függőség vagy telepítés nincs; teljes alkalmazásellenőrzés a PR CI feladata.
[Esetek, módszer és korlátok](viber-real-selection-v1.md).

## 2026.11.05. — Kontrollmérés: 5/8 és külön V2 értékelőaudit

A #159 mainverzióján (`5389bc8`) a nyolc szintetikus kontroll valós mérése elkészült:
5/8 automatikus megfelelés, 8/8 értékelhető válasz; teljes offline riportegyezés.
A feltárt értékelőhibák miatt külön, utólagos V2 audit készült. Azonos válaszokon:
strukturális tartalom 7/8, címkézés 6/8, jogosultság 8/8, együttes megfelelés 5/8.
A V1-riport változatlan; a V2 nem modelljavulást bizonyít. K3 bizonyíték nélkül Rékát
címzettként köti, ezt a V2 már jelzi. K1 entitásbizonytalanságát megjegyzésként kezeli,
K4/K6 tágabb szándékcímkéjét külön címkézési eltérésként őrzi meg.
[Szabályok és reprodukció](viber-controls-v2.md). Runtime/prompt nem változott,
új providerfuttatás vagy telepítés ebben a körben nem történt.

## 2026.11.05. — Viber: összevont értelmezési javítás és nyolc új kontroll

#154–#157: önmagára mutató sorrendkapcsolatok blokkolása, explicit beszélőhivatkozás,
célzott CI és társalgási szereplők védelme. Az egymást követő, külön engedélyezett
20 esetes mérések V3 pontozóval: 55,125 → 56,833333 → 59,6 → 60,666667.
A 59,6-os futás társalgási esetben kritikus jelzést adott; az utolsó nem.
Mindegyik benchmark FAIL, nem kiadási megfelelés. Az utolsó mérés forrása `62fe96b`,
modellje `gpt-4.1-2025-04-14`, temperature 0; 20/20 pontozható eset.
A 44 eredeti fájl ellenőrzőösszege és a teljes offline riportegyezés ellenőrizve.

[#158](https://github.com/beeaibuild/kontener-app/pull/158), main `1a0ae3c`:
QUESTION/SUGGESTION/STATUS_REPORT + INSTRUCTION ellentmondás blokkolása;
szereplő, tárgy, beszédszándék és konténeres szaknyelv közös pontosítása.
221 célzott teszt, teljes PR CI és célzott P63 sikeres. Új valós mérés ehhez még nincs.
[Review és a V1 reprezentációs korlátai](viber-interpretation-contract-review.md).

K1–K8 új, szintetikus kontroll külön `konti-viber-controls-v1` azonosítóval futtatható.
A runner pontos bemenetet, szűrt választ, predikciót, definícióazonosítót és SHA-256
összegeket ment; a csomag suite-azonosságot is ellenőrző offline replayt támogat.
Nincs kötött rekordszám vagy V1-gyel összeátlagolt pontszám. A jelentésszöveget külön
embernek kell ellenőriznie; a scripted próba nem modellminőség-mérés.
[Futtatás és előre rögzített elvárások](viber-controls-v1.md).
Új függőség, providerhívás, staging- vagy productiontelepítés nem történt ebben a körben.

## 2026.11.04. — Viber-benchmark: megőrzött valós mérés, offline reprodukció, V2/V3 javítás

**Mainben:** #147 álnevesített 20 esete, #151 esetszintű exportja és #146 mérési protokollja.
A mérés forrása `4003986c32b600db8a13ba0d84edb3929e82e6ba`.
A futtató a riport mellett pontos esetszintű bemeneteket, szűrt providerkimeneteket,
visszajátszható predikciókat, ellenőrzési azonosítókat és SHA-256 jegyzéket exportál.
[Exportleírás](viber-eval-artifacts.md), [protokoll](viber-eval-run-protocol.md).

**Valós mérés:** a felhasználó PC-jén, külön engedéllyel egyetlen 20 esetes
`gpt-4.1-2025-04-14` futás készült, temperature 0 mellett.
V1 eredmény: 56,02083333333333 pont, 20 pontozható eset, 0 pontozhatatlan eset,
`criticalSafetyFail=false`, `passed=false`.
A benchmark FAIL-hez tartozó 1-es kilépési kód nem szolgáltatói futáshiba.
Az archívum: `viber-real-20261104-165224.artifacts.zip`; a 45 ellenőrzőösszeg
és az offline újrapontozás független ellenőrzése sikeres.
Mind a 20 eset teljes pontozási eredménye egyezett az eredetivel.
A nyers csomagot külön őrizzük; bemenetek és providerválaszok nem kerülnek ebbe a repóba.

**Feltárt mérési rések:** a V1 egyes címkéknél nem jelzett felcserélt sofőrkötést;
a feltétel jelenléte önmagában elfogadást adhatott, miközben annak szereplője vagy jelentése eltért.
A feltételszöveg egyszerű egyezése a helyes, más nyelvű parafrázist is büntette.

**Javítások (beolvasztási sorrend: #152 → #153):** [#152](https://github.com/beeaibuild/kontener-app/pull/152)
címkétől független szerepkötés-ellenőrzést és szigorú feltételkaput vezet be
(`konti-viber-safety-v2`).
A ráépülő [#153](https://github.com/beeaibuild/kontener-app/pull/153) az egyetlen atomi
visszatérési feltételt `RETURN_TO` predikátummal és polaritással ellenőrzi
(`konti-viber-condition-v3`).
Az explicit null/UNKNOWN jelentés hiányos kötést és blokkolt végrehajthatósági minősítést ad.
A régi mentett válaszok változatlanok, jelentést nem egészítünk ki utólag.
A nyilvános végső feltételobjektum változatlan, de a közös OpenAI-válaszséma módosul;
ez egy későbbi telepítés után az alkalmazás providerét is érinti.

**Validálás:** V2: 122/122, V3: 133/133 célzott offline teszt PASS;
célzott strict TypeScript és whitespace ellenőrzés PASS. A 20 orákulumeset 100%.
A fagyasztott valós válaszok V2 és V3 eredménye azonos: 54,77083333333333 pont,
VBR-15 nem igazolt feltétellel. Mind a 20 eset export/visszajátszás eredménye egyezik
üres API-kulcs és hibára állított fetch mellett. Az eltérés a V1-hez képest pontozóhatás;
modellromlást vagy új modellminőség-javulást nem bizonyít.
A teljes repó ellenőrzését a PR-ek CI-je végzi; a célzott helyi tesztek nem teljes CI-bizonyítékok.

**Korlát / következő lépés:** a strukturált jelentés modellállítás, nem független nyelvi bizonyíték;
összetett és több feltétel általános kezelése nincs megoldva.
Előbb a #152, majd a #153 teljes CI-jének és review-jának lezárása szükséges.
Új szerződés hatását csak külön engedélyezett, rögzített V3 pontozóval összevetett mérés igazolhatja.
Új providerfutás és staging/productiontelepítés a javítások készítésekor nem történt.
Részletes változások: [V2](viber-safety-v2-change.md), [V3](viber-condition-meaning-v3.md).

## Bizonyítási hibák és ellenőrzési tanulságok — visszakereshető esetek (2026.10.31.)

**Értelmezési határ:** az alábbi esetekből **nem következik bizonyítottan**, hogy egy AI-agent tesztfutást vagy PASS-eredményt kitalált. A bizonyított tanulság a jelentés, a futási artefaktum, a tesztlefedettség és a valós rendszerállapot elkülönítése. AI-hallucinációt csak konkrét, visszakereshető modellállítás és annak ellentmondó futási bizonyítéka alapján szabad állítani.

### PR #58 — valós provider-benchmark bizonyítékhiánya (2026.09.11–13.)

**Helyzet:** a megelőző [PR #57](https://github.com/beeaibuild/kontener-app/pull/57) draft hét `gpt-4.1-mini` valós providerfutást és hét FAIL benchmarkítéletet jelentett; a hangolt policy merge-elését a jobb átlagpontszám mellett romló kritikus biztonsági eredmények miatt ellenezte. Ezek a közölt számok utólag nyers futási artefaktumokból nem voltak rekonstruálhatók, ezért **nem tekintjük őket igazolt modellteljesítménynek**. A következő PR #58 scripted oracle-ja és zöld CI-je nem helyettesített valódi provider-benchmarkot.
**Ellenőrzés:** a korábbi beszélgetésben a 7 futás utólagos ellenőrzésekor az elvárt `/tmp/konti-slice5a-real/run-{1..7}.json` artefaktumok és a providerkulcs nem álltak rendelkezésre az ellenőrzési környezetben; a metrikák reprodukciója blokkolt volt. A PR maga is kifejezetten rögzíti: „No real-provider benchmark was run.”
**Bizonyíték:** [PR #57](https://github.com/beeaibuild/kontener-app/pull/57), [PR #58](https://github.com/beeaibuild/kontener-app/pull/58), [CI 34620047165](https://github.com/beeaibuild/kontener-app/actions/runs/34620047165). Az ellenőrzés beszélgetésbeli megállapítása nem GitHubon tárolt nyers benchmarkartefaktum.
**Tanulság:** a scripted teszt PASS nem valós provider-benchmark; hiányzó nyers artefaktum esetén az eredmény **nem igazolt**, nem automatikusan „kitalált”.

### PR #59 — két tényleges, archivált, de már nem újrapontozható real-provider kampány (2026.09.14.)

**Helyzet:** a #57 (2026.09.11–13.) nem rekonstruálható állításaitól **külön** a [#59](https://github.com/beeaibuild/kontener-app/pull/59) manuális workflow két, GitHub Actions által dokumentált `gpt-4.1-mini` kampányt futtatott: [34809221837](https://github.com/beeaibuild/kontener-app/actions/runs/34809221837) (fagyasztott SHA `67696101fefd2652a3a53a28eb9cf30ef1d60950`) és [34832401936](https://github.com/beeaibuild/kontener-app/actions/runs/34832401936) (SHA `7f8fa28465ce10a063c91683f9401c7e1c0d4b14`, #60 utáni szemantikus szerződés).
**Ellenőrzés:** mindkét GitHub job SUCCESS, 7/7 valódi providerfutás és 140/140 pontozható eset; benchmark szemantikus eredmény mindkettőn FAIL. A megmentett GitHub-joblogok első kampányban 0, másodikban 10 kritikus hibás esetet jelentenek. Ez **összemosott összehasonlítás**, nem bizonyított modellromlás és nem is bizonyított pontozóhatás: a két SHA között egyszerre módosult a `scoring.ts` (pontozó), valamint a `policy.ts`, `providers.ts` és `semantic.ts` (a modellnek átadott tartalom és annak feldolgozása). Az egyes hatások így nem azonosíthatók. A korábbi nyers kimenetek újrapontozása az új pontozóval elkülöníthette volna a mérőeszköz változásának hatását; a bemeneti/policy változás hatásának becsléséhez további kontrollált futás kellett volna. A futási napló igazolja a 14 fájlos archívum sikeres feltöltését (artifact ID `10333738286`, illetve `10343520596`), de a 14 napos megőrzés után a nyers JSON/stderr fájlok már nem érhetők el; független esetszintű újrapontozás **nem végezhető**.
**Bizonyíték:** [első GitHub run](https://github.com/beeaibuild/kontener-app/actions/runs/34809221837), [második GitHub run](https://github.com/beeaibuild/kontener-app/actions/runs/34832401936), [PR #59](https://github.com/beeaibuild/kontener-app/pull/59), [PR #60](https://github.com/beeaibuild/kontener-app/pull/60); a két letöltött joblogot külön megőriztük a beszélgetés mellékleteként (nem nyers providerválaszok).
**Tanulság:** egyszerre csak egy tényezőt változtassunk kontrollált összehasonlításban; a nyers modellválaszok megőrzése teszi lehetővé a pontozó későbbi, API-hívás nélküli cseréjét és az eredmény újrapontozását. A sikeres futás és feltöltés önmagában nem tartós auditálhatóság. A következő eval protokollban a személyesadat-ellenőrzést követően tartós, verziózott és visszaellenőrizhető artefaktumarchívum szükséges; a régi két mérés összesített számai csak GitHub-logból származó, **nem újrapontozható** eredmények.

### PR #74 — zöld CI mellett feltárt tesztlefedettségi hiányok (2026.09.21.)

**Helyzet:** a Slice 7A provenance/idempotencia implementáció teljes CI-je sikeres volt, de célzott szakmai review további F1–F8 blokkoló kockázatokat talált.
**Ellenőrzés:** a korábbi felülvizsgálati körben külön vizsgálandó volt a cross-user/mode idempotencia, szemantikus fingerprint, hamis sidecar, forrásverseny és DB-invariánsok; a PR későbbi javító commitokat és új exact-tree CI-t dokumentál.
**Bizonyíték:** [PR #74](https://github.com/beeaibuild/kontener-app/pull/74), [első sikeres CI 35583951177](https://github.com/beeaibuild/kontener-app/actions/runs/35583951177), [javítás utáni CI 35593975672](https://github.com/beeaibuild/kontener-app/actions/runs/35593975672).
**Tanulság:** a zöld CI az adott tesztkészlet sikerét bizonyítja, nem a teljes biztonsági állítást. Az ellenőrzésnek a nem tesztelt invariánsokat is keresnie kell.

### PR #89 — részleges teszt-PASS nem teljes CI-PASS (2026.10.04.)

**Helyzet:** az átosztási deadlockjavításnál a célzott hozzáféréskezelési és átosztási regressziók sikeresek voltak, de a teljes [P63 37188054843](https://github.com/beeaibuild/kontener-app/actions/runs/37188054843) futás **FAILURE** maradt: 2599 teszt PASS, egy változatlan AI-orchestrator teszt FAIL (várt providerhívás 1, tényleges 0; `AUTHORITY_CONTEXT_STALE`).
**Ellenőrzés:** a PR a hibát, a pontos tesztet és azt is rögzíti, hogy a sikertelen teljes Vitest-kapu után production build/E2E/Docker nem futott. Későbbi javítások és futások külön ellenőrizendők; ez a bejegyzés az adott, piros CI-futás állapotát dokumentálja.
**Bizonyíték:** [PR #89](https://github.com/beeaibuild/kontener-app/pull/89), [P63 37188054843](https://github.com/beeaibuild/kontener-app/actions/runs/37188054843).
**Tanulság:** a célzott teszt-PASS nem azonos a teljes kapu sikerével. **Nem bizonyított**, hogy egy AI-agent ténylegesen nem futtatott tesztre állított volna PASS-t; ehhez a konkrét eredeti modellüzenet és az ellentmondó futási napló együtt szükséges.

### PR #136 — hamis STALE jóváhagyási hiba (2026.10.19.)

**Helyzet:** ugyanazon üzleti állapot rendezetlen Prisma-listái eltérő recommendationHash-t eredményezhettek, ezért érvényes ajánlás is STALE-ként záródhatott.
**Ellenőrzés:** a megfordított DB-sorrendet használó regresszió az eredeti implementáción hibázott, a javításon átment; 57 célzott teszt és helyi TypeScript-ellenőrzés PASS. A stagingben észlelt konkrét eltérés pontos mezője utólag nem volt forenzikusan bizonyítható.
**Bizonyíték:** [PR #136](https://github.com/beeaibuild/kontener-app/pull/136).
**Tanulság:** a modell-ajánlás jóváhagyhatósága determinisztikus szerveroldali állapotkezeléstől is függ; a reprodukált hibamechanizmus nem jogosít fel a staginges incidens teljes okának bizonyítottnak nevezésére.

## 2026.10.27. — Node 22 Docker runtime előkészítése (#145)

**Mit:** a közös Docker-alap Node 22.23.4 / Debian Bookworm verzióra vált; külön GitHub Actions biztonsági validációs workflow került a kódba.
**Miért:** a runtime frissítése a meglévő CA/OpenSSL/Prisma beállítások megtartásával.
**Bizonyíték:** [PR #145](https://github.com/beeaibuild/kontener-app/pull/145), 2026.10.27-én merge-elve.
**Nyitott:** a PR szerint a cache nélküli build, Node/OpenSSL/Prisma ellenőrzés, tesztek, staging migráció és production rollout **még nem igazolt**. A merge nem jelent deployt.

## 2026.10.26. — Azonos című konténer: csere vagy további rendelés (#144)

**Mit:** bejelentkezett ügyfél azonos címre és típusra történő rendelésekor kötelező csere/új konténer választás; tulajdonosi és tranzakciós újraellenőrzés, ismételt beküldés elleni védelem.
**Miért:** az aktív kint lévő konténer mellett érkező új rendelés szándékának tisztázása.
**Bizonyíték:** [issue #143](https://github.com/beeaibuild/kontener-app/issues/143), [PR #144](https://github.com/beeaibuild/kontener-app/pull/144), 64 célzott helyi teszt és TypeScript-ellenőrzés PASS; PR 2026.10.26-án merge-elve.
**Nyitott:** a PR-ben a GitHub Actions futás keret miatt kihagyva; stagingtelepítés és elfogadási próba hátra van, production bevezetés nincs igazolva.

## 2026.10.20. — Partner-/admin-kontextus és teljes útvonalas visszfuvar V1 (#142)

**Mit:** forráshoz kötött partnerelőzmények és ellenőrzött adminválasztások tanácsadó kontextusa; három privát AI-olvasóeszköz és flottaútvonalat összevető visszfuvar-felület.
**Miért:** az ajánlásoknál a döntési előzmények és a teljes hátralévő út számítson; hiányzó útadatból ne keletkezzen látszatoptimum, GPS-ből ne következtessünk automatikusan üres autóra.
**Bizonyíték:** [PR #142](https://github.com/beeaibuild/kontener-app/pull/142), staging LIVE 2026.10.20. 14:01 (commit `345ae1ce`), health OK; valódi AI tool-hívások, 403-as jogosultság/Origin ellenőrzések; CI 39418804253 SUCCESS.
**Nyitott:** PR **nyitott**, main/production változatlan; a teljes P63 regresszió a PR-jelentés idején még futott. Valós kétjárműves összevetés, pozitív adminpreferencia és üzemi haszon nincs igazolva.

## 2026.10.19. — Driver/AI/kaszkád production előkészítés és klienskiadások (#136–141)

**Mit:** #136 determinisztikus snapshot-sorrend a hamis STALE ellen; #137 driver/AI/kaszkád release-előkészítés; #138 aláírt Android release-eljárás; #139–141 külön telepíthető production és staging Windows WebView2 kliens.
**Miért:** döntés-jóváhagyás konzisztenciája, ellenőrizhető kiadási folyamat, tartós Android-frissíthetőség és kliensoldali környezetszeparáció új üzleti logika nélkül.
**Bizonyíték:** [#136](https://github.com/beeaibuild/kontener-app/pull/136), [#137](https://github.com/beeaibuild/kontener-app/pull/137), [#138](https://github.com/beeaibuild/kontener-app/pull/138), [#139](https://github.com/beeaibuild/kontener-app/pull/139), [#140](https://github.com/beeaibuild/kontener-app/pull/140), [#141](https://github.com/beeaibuild/kontener-app/pull/141); #137 CI/P63: 3226 teszt + 174 UI PASS; #138 release és emulátorpróbák PASS; #139 Windows telepítés/WebView2 login smoke PASS.
**Nyitott:** a #137 production előkészítési bizonyíték nem önmagában teljes rollout-igazolás; Android Samsung production telepítés/push, valamint Windows hitelesített dokumentum-/nyomtatási próba külön igazolandó. A staging fixture útmegtakarítása nem valós üzemi megtakarítás.

## 2026-10-19 — kapcsolható szerveroldali kaszkádfigyelő (még nem telepített)

A #131 alapját kiegészítő, böngészőtől független ellenőrzés. Adatbázisos admin főkapcsoló, generációs leállítási védelem, tartós futás- és költségfoglalás; kétperces GPS-/Google-ellenőrzés, munkafázis, ismétlést elnyomó belső riasztás és privát chat/outbox. Háttér-AI, automatikus átszervezés és partnerküldés nincs. Staging e-mail-tiltás megmarad; nincs új függőség vagy productionmódosítás. Részletek és fennmaradó átvételi kapuk: `docs/cascade-monitor.md`.

## 2026-10-19 — kaszkád staging elfogadási próba

2026-10-19 01:24–01:27 (Europe/Budapest), staging, `STAGING-CASCADE-20261019`: két mesterséges, ismert végpontú horgony és három tervezett belső tesztfuvar. Nem fizikai GPS-/tabletpróba; koordináták korábbi jelölt tesztvégpontokból, feladatok közvetlenül létrehozott, auditált fixture-ök. Valódi Google CURRENT útbecslés: A első feladat 01:50 körüli érkezésével kb. 17 perc késés, utána a második 02:23 körül kb. 22 perc késés. A napi terv nélküli első futás nem ajánlott cserét; a két tesztsofőr/jármű napi tervbe vétele után az AI az A1 ↔ B1 cserét emelte ki. A felületen megjelenő összes szigorú késéscsökkenés 37 perc (az összegzés és a sorok kerekítése eltér), az alternatívában minden érkezési határidő tartható, a legszűkebb tartalék 9 perc. Két ellenőrizhető ETA-/sofőr-/járműadatú partnertervezet elkészült; küldés és kiosztási végrehajtás nem történt.

A felületi időadatmentés valódi PATCH-útvonala és auditja igazolt: az A1 helyszíni idő üresre állításakor az A1 befejezése és az A2 érkezése ismeretlen lett, javító csere nem jelent meg. Az értéket 5 percre visszaállítottuk. Cleanup: 3 tervezett fixture LEMONDVA, 2 horgony SIKERTELEN; 0 aktív fixture, az ideiglenesen létrehozott napi terv eltávolítva, kapacitás nem változott, kiosztások változatlanok. 62 Order és 63 Job változatlan; 5 lezárt TransportTask teszttörténet megmaradt. Auditok: STAGING_CASCADE_FIXTURE_CREATED, STAGING_CASCADE_PLAN_CREATED, DISPATCH_TIMING_UPDATED, STAGING_CASCADE_FIXTURE_CLEANED. Éles adatot nem érintettünk.

Ez az alap kaszkád-/cserejavaslat-/hiányzóadat-próba elfogadva; nem bizonyítja a tényleges csere-végrehajtást, élő GPS késésdetektálását, háttérfigyelést vagy külső üzenetküldést.

## 2026-10-19 — kaszkádkockázat stagingtelepítés

2026-10-19 01:15:41 (Europe/Budapest): staging live, commit `2b40d85cd918e074929fcaaa3d3f4a5b112a9500`, Render deploy `dep-dbal51u1egvs73b4t1b0`. A 01:10-es staging export elkészült; a `20261019001000_dispatch_timing` migráció sikeres. A teljes CI zöld (3186 Vitest teszt, típusellenőrzés, migrációs paritás, build, böngészős regressziók, Docker). Health: alkalmazás/adatbázis OK, a kiadott commit egyezik. Adatszámok: 62 rendelés, 63 Job, 31 TransportTask, változatlanok. `KONTI_CASCADE_ENABLED=true` csak stagingen. A mai üres terven a panel betöltése és a vizsgálat hibamentes. Érdemi késéslánc/csere és időadatmentés staging elfogadási próbája még hátra van; productionhoz nem nyúltunk.

### Implementáció

A napi operáció opcionális időablakokat és helyszíni időket kap. A számítás továbbviszi a várható késést a sofőr ismert sorrendjén, a hiányos adatot ismeretlenként jelzi. Friss Google-forgalmi útbecslés, friss GPS vagy diszpécseri hátralévő idő szükséges. Pontos feladatcserék közül csak a szigorú késést javító, másik szigorú késést nem növelő változat jelenik meg. Az AI meglévő jelöltek közül választ; partnerüzenet csak másolható tervezet. Alapból kikapcsolt; nyitott oldalon opcionális kétperces figyelés, háttérszolgáltatás nincs. Job módosítási időbélyeg védi az időadatmentést és az elemzést az elavult feladatoktól. Részletes bevezetés és tesztek: `docs/cascade-impact.md`.

### 2026-10-18 esti elfogadási próba

A kétfeladatos Google/AI csere friss előnézete 67,1 → 18,4 perc hátralévő utazást mutatott, 48,6 perc megtakarítással. Az előnézetet a Mégse lépés zárta; a kiosztások nem változtak. A jelölt tesztfeladatok lezárva/lemondva, a tesztkapacitások visszaállítva. A tényleges végrehajtási próba továbbra is külön ellenőrzés.

## 2026-10-18 — első AI-kiosztás eredménye és mentett döntése

Az ajánláskérés technikai AuditLog-eseményt kap: eredmény, biztonságos okkód, szerveroldali idő,
útbecslés elérhetősége és az eredeti javaslat belső hivatkozásai.
A sikeres kiosztás meglévő auditja külön jelzi az elfogadást, a módosított mezőket,
az ajánlás nélküli kézi kiosztást és a mérés nélküli esetet.
A szerver az eredeti, ugyanahhoz a kezelőhöz és rendeléshez tartozó mérést ellenőrzi;
a kiosztási tranzakció visszagörgetése vagy ismételt mentése nem növeli a döntések számát.
A mérési esemény nem terheli a rendelés üzleti naplóját. Nincs új tábla vagy migráció.

Szerződés, értelmezési korlátok és heti összesítő SQL: [initial-assignment-telemetry.md](initial-assignment-telemetry.md).
A tesztek az elfogadás/módosítás, idegen mérési hivatkozás, időmérés, mérési íráshiba és duplamentés eseteit fedik.
[#129](https://github.com/beeaibuild/kontener-app/pull/129) összevonva, staging commit:
`3a18f54c5766b077d9356f9dbd607bdf47eff52d`; deploy: `dep-dbaf87rncjis73dbme3g`, live.
[CI 39143211357](https://github.com/beeaibuild/kontener-app/actions/runs/39143211357):
1933 unit, összesen 3159 Vitest / 245 fájl, 174 nagy UI-próba, külön szállítólevél- és célzott mobil/tabletpróbák,
production- és Docker-build sikeres. Az első kör két hideg next dev fordítás miatti 30 másodperces időtúllépése után
csak az érintett két összetett teszt kapott 90 másodperces keretet; az ellenőrzések változatlanok.

Élő OpenAI/Google staging-próba: R-2026-0056 (`cmve1p5pa0009sngq61na7250`) ajánlása 1988 ms,
útbecsléssel; mentés előtt 0 job és 0 döntés, mentés után 1 job, ACCEPTED, minden változásjelző false.
R-2026-0057 (`cmve1sa890010sngqcqsna5b5`) ajánlása 859 ms;
a 2026-10-18-as javasolt dátumot 2026-10-20-ra módosítva 1 job és MODIFIED keletkezett,
kizárólag changes.date=true mellett. Mindkét hivatkozás VERIFIED, rendelésenként pontosan 1 döntési audit.
A technikai mérés nem jelent meg az üzleti eseménynaplóban.
Mindkét tesztrendelés LEMONDVA, 0 aktív feladat; STG-AI-SPARE-1 és STG-AI-SPARE-2 SZABAD.
Health: status=ok, database=ok, a fenti commitSha egyezik.


**Aktuális állapot: 2026. október 18. (Europe/Budapest).**
**Forrás:** a repository verziótörténete, nyitott és összevont GitHub PR-ek és a `docs/` műszaki dokumentációja.
**Cél:** a termék fejlődésének, a fontos mérnöki döntéseknek és a bizonyítékoknak az egyhelyes, rövid, továbbvezethető rögzítése.

Ez **fejlesztéstörténet**, nem az alkalmazás üzleti AuditLogja és nem a szervernapló. Nem helyettesíti a PR-eket, a tesztjegyzőkönyveket vagy a release-runbookot. A korábbi szakaszok mérföldkőszintű rekonstrukciók, **nem** minden commitot tartalmazó teljes kronológia.

## Hogyan olvasd?

- **Kódba bekerült:** a jelzett változás GitHubon összevont PR-rel alátámasztott. Ez nem azonos a sikeres production deployjal.
- **Ellenőrzött:** csak a konkrétan hivatkozott teszt, CI, staging smoke vagy más bizonyíték keretein belül értendő.
- **Nyitott:** még igazolandó fizikai teszt, szolgáltatói integráció, telepítés vagy üzleti hatás.
- Dátumok: az események helyi, `Europe/Budapest` szerinti napja, ahol megállapítható. A PR-ek belső időbélyege UTC.
- Élő állapot vagy tesztszám esetén mindig a legújabb commit, CI-futás és deployment az irányadó; az itt szereplő számok **történeti pillanatképek**.

## Aktuális műszaki pillanatkép — 2026.10.18.

### 2026.10.18. — AI-ajánlás új rendelés első kiosztásánál

**Mit:** az első kiosztás automatikusan kitöltött sofőr–autó–dátum javaslatot kap a meglévő Konti AI-providerrel.
Az operátor átírhatja vagy mentéssel jóváhagyhatja; a javaslat nem foglal és nem értesít.
**Miért:** már a beérkező rendelés kiosztását segítse az AI, ne csak meglévő feladatok átrendezését.
**Bizonyíték:** [#126](https://github.com/beeaibuild/kontener-app/pull/126),
[teljes CI](https://github.com/beeaibuild/kontener-app/actions/runs/39128866581), végleges forrás `305c2ee12786aebde659d9caeee79e205708a8d6`.
1922 unit / összesen 3139 Vitest-teszt (244 fájl), 174 nagy UI-próba, a külön böngészős és szállítólevél körök,
TypeScript, migráció/séma-paritás, production- és Docker-build zöld. Az új három adatbázisos eset és a teljes
ajánlás → emberi jóváhagyás → kiosztás böngészős próba is sikeres, determinisztikus AI-providerrel.
**Staging:** `7f7a5375f1b787dac2bd6eafee3744fb2f30f5f3`, Render deploy `dep-dbacdqnbtjjs73ekhobg`,
2026.10.18. 15:19:55-kor live (Europe/Budapest). Nincs új migráció, titok vagy függőség.
15:20:28-kor a publikus `/api/health` válasza `status=ok`, `database=ok`, a várt commitazonosítóval.
Részletek: [kezdeti kiosztási szerződés](initial-assignment-recommendation.md).
**Valós szolgáltatói staging-próba (15:24–15:26):** a böngészőben, meglévő operátori munkamenettel létrehozott
R-2026-0055 jelölt tesztrendelés címét a Google geokódolta, telephelyi távolsága 8,3 km.
Az éles provideres ajánlás STAGING TEST Sofőr / STG-AI-01 / 2026-10-18; az ajánlás indulási pontjából 11,9 km / 20 perc
lekéréskori Google-útbecslést mutatott. Ez nem a telephelyi távolság és nem érkezési idő.
Mindhárom mező kézzel átírható volt; az „Ajánlás alkalmazása” visszaállította a javaslatot.
A konténer kiválasztása után a jóváhagyás gomb aktív lett, de nem kattintottunk rá.
Adatbázisos ellenőrzés: `jobs=0`, `assignment_audits=0`, `containerId=null`; STG-AI-SPARE-1 `SZABAD` maradt.
A tesztrendelés `LEMONDVA` végállapottal lezárva, feladat továbbra sincs. Ügyfélkapcsolat fiktív, cím nyilvános helyszín.
**Nyitott:** terhelt munkanapos üzemi próba és production-kiadás; ezen a szolgáltatói próbán nem mentettünk kiosztást.

- **Staging:** #114, #115, #118 és #120 telepítve; igazolt commit `da02fd6b295ed1cebdd67f97b3705c8c67e5ab79`, Render deploy `dep-dba6rngm7kps73fn5a60`, 2026.10.18. 08:59-kor live, health/adatbázis rendben. A #118 tartalmazza a #116 feliratjavítását is. #117 még nyitott/nem telepített. Az új additív migráció sikeresen lefutott. A két proaktív AI-kapcsoló csak stagingen bekapcsolva.
- **Production-történet:** a #110 október 16-i kiadása `71e2a555788196942e9d4e3c8c9208b976a403da` commiton sikeresen települt; a kiadási CI, a három migráció, a health/adatbázis és a publikus oldal ellenőrzése sikeres volt. Ez történeti kiadási bizonyíték, nem a mindenkori production HEAD állítása. Előtte friss Render DB-export készült; ebből visszaállítási próba nem történt. A mostani #114–118 csomag production-kiadása nincs igazolva.
- **Éjféli szerverellenőrzés:** október 18. 00:06-kor a munkamenet visszavonása/lejárata és a műszak lezárása pontosan `2026-10-17T22:00:00Z` (= budapesti éjfél). Utolsó mért GPS-pont 23:57:50, éjfél utáni pontok: 0. A tablet párosítása megmaradt.
- **Nyitott:** a tablet belépőképernyőjének/GPS-leállásának fizikai ellenőrzését a felhasználó másnap végzi. Két tabletes átosztás egyetlen rendelkezésre álló tablettel nem volt kipróbálható. További offline/késői push, tartós műszak- és akkumulátorpróba, a production-kiadás és a tényleges logisztikai haszon mérése is hátravan.

### Történeti kiindulópont — 2026.10.16.

A `main` ágba összeolvadt a [#110 production-promóciós PR](https://github.com/beeaibuild/kontener-app/pull/110). A csomag a stagingen már összerakott tablet/GPS, digitális szállítólevél, P01–P06 stabilizáció és az aznapi AI-ajánlás kódját integrálja. A #110 leírása szerint a forrás CI-jében **3034 Vitest, 173 UI és 5 szállítólevél E2E** ellenőrzés, valamint production/Docker build futott sikeresen. A stagingben az ajánlás preview, végrehajtás és stale-elutasítás böngészős ellenőrzése szerepel; az útvonal-szolgáltató ott MOCK módban futott.

**Nem igazolt kizárólag a #110 merge-ből:** a production Render-deploy tényleges sikere, az előírt production mentés ellenőrzése, az élő Google-útvonal működése és a fizikai Samsung GPS-műszak tesztje. Ezek állapotát külön deploy-/pilot-bizonyíték alapján kell bejegyezni. A #110 megnevezett alkalmazás-rollback alapja: `c0003741a21a256cd469be735addd2c2a9ed993b`; az additív sémát új rekordok törlésével visszagörgetni tilos.

Fontos hivatkozások: [release PR #110](https://github.com/beeaibuild/kontener-app/pull/110), [same-day ajánlási szerződés](same-day-assignment-recommendation.md), [GPS-pilot](android-gps-pilot.md), [migrációs workflow](migration-workflow.md), [szállítólevél V1](digital-delivery-note-v1.md).

## Fejlesztési mérföldkövek (legújabb elöl)

### 2026.10.18. — Stagingtelepítés és automatikus áthelyezési ajánlás valós próbája

**Telepítés:** #118 és #120 összevonva; staging `da02fd6b295ed1cebdd67f97b3705c8c67e5ab79`, deploy `dep-dba6rngm7kps73fn5a60` live. `20261017210000_proactive_recommendations` migráció kész; health 200, adatbázis ok. `KONTI_AI_PROACTIVE_ENABLED=true`, `KONTI_AI_REASSIGNMENT_ENABLED=true`, production nem módosult.
**Próba:** napi terv két meglévő tesztsofőrrel és STG-AI-01/02 autóval; T-2026-0022/0023 horgonyok és már kiosztott T-2026-0024. Minden indulási/célcím geokódolva. A chat láthatósága indította az ellenőrzést, kézi AI-kérés nélkül. Az első két, nem értékelhető horgonyra nem állított előnyt; a céltételre 09:10-kor egyetlen automatikus privát javaslat jelent meg. Google aktuális forgalom + OpenAI `gpt-4.1-mini`: jelenlegi beillesztési többletidő 39 perc, javasolt 7 perc; becsült előny 32 perc (82%), 31,1 km. Az indulás korábbi feladat végpontja, nem élő GPS; 0 aktív műszak.
**Előnézet és lezárás:** Elfogadom után 09:11-kor friss Google-újraszámolás, ugyanaz a 32 perc előny és külön „Biztos végrehajtsam?” lépés. DB-ben PREVIEWED alatt is az eredeti sofőr/STG-AI-01/sorrend 2 maradt. Végrehajtás helyett Mégse; végső státusz CANCELLED, confirmedAt és auditId null. Mindhárom tesztfuvar LEMONDVA, ma 0 aktív belső fuvar. Ügyfélrendelés 59 és job 61 maradt; ezek nem változtak a próbában.
**Határ:** ez a geokódolás → Google → automatikus AI-chat → friss előnézet folyamat átvétele, nem éles kiadás, új végrehajtási pilot vagy realizált megtakarítás bizonyítéka. A fizikai tablet/éjféli próba és az üzemi visszamérés továbbra is külön feladat.


### 2026.10.18. — Kiosztott feladat érdemi áthelyezési javaslata

**Mit:** a [#120](https://github.com/beeaibuild/kontener-app/pull/120) a #118-ra építve a már kiosztott, mai, még nem indított feladat egyetlen jobb elhelyezését is fel tudja ajánlani. A jelenlegi és javasolt beillesztési költséget közös, célfeladat nélküli tervhez viszonyítja; csak legalább 5 perc ÉS 20% javulás kerül a modell elé. Az útbecslést a Google adja, a különbséget a szerver számolja, az AI választ vagy visszautasít.
**Védelem:** generálás/elfogadás/előnézet alatt változatlan kiosztás, friss összehasonlítás és `IMPROVEMENT_LOST` elavulás, külön végrehajtási megerősítés és ismétlésvédelem. Automatikus ellenőrzéshez külön `KONTI_AI_REASSIGNMENT_ENABLED=true` kell, alapból kikapcsolt.
**Bizonyíték:** a PR és annak CI-je tartalmazza az összehasonlítás, adatminimalizálás, hiányzó referencia, eltűnő előny és a megerősített, megismételt végrehajtás tesztjeit. A végleges `a4c9041c493f3135b06609968b3c402c4cb63e2c` commit [teljes CI-kapuja](https://github.com/beeaibuild/kontener-app/actions/runs/39076221594) sikeres: migráció/sémaparitás, típusellenőrzés, 1879 unit és összesen 3088 Vitest teszt, 173 széles UI-regressziós próba és a célzott böngészős kapuk, production/Docker-build. A külön szállítólevélkapu is zöld. A GitHub artifact-feltöltésének átmeneti hálózati hibája után a változatlan kód újrafuttatása teljes egészében sikeres.
**Átvétel:** az október 18-i stagingtelepítés és valós Google/AI próba fent rögzítve; production és fizikai GPS/AI próba még nincs. Egyfeladatos áthelyezés, nem atomi csere vagy teljes napi optimalizálás. A becsült időnyereség tényleges üzleti haszonként történő visszamérése külön következő munka.

### 2026.10.17. — Proaktív, privát diszpécseri AI-javaslat

**Mit:** a nyitott, látható diszpécserchat automatikusan ellenőrzi a mai kiosztatlan feladatokat, és a meglévő Google/GPS/AI motor mentett ajánlását admin-only üzenetként kínálja fel; elfogadás, friss előnézet és külön végrehajtási megerősítés marad.
**Miért:** az üzemeltetőnek ne kelljen minden elhelyezési ajánlást külön lekérnie; tartós globális claim, percenkénti/napi híváskorlát és feladatonként napi egyszeri közzététel védi az ismétlést és a költséget. A sofőri API és a visszavont tagság nem fér hozzá.
**Bizonyíték:** [#118](https://github.com/beeaibuild/kontener-app/pull/118), `b844575ca0d43add12645f1acf40183b786cbc41`; [CI 39067499000](https://github.com/beeaibuild/kontener-app/actions/runs/39067499000) teljes egészében sikeres: migráció/sémaparitás, unit/integráció, build, böngészős regresszió és Docker. A párhuzamos közzététel, változatlan kiosztás és lemondás utáni elavulás adatbázisos teszttel igazolt.
**Nyitott:** PR-ben előkészítve, stagingre/productionbe nem telepítve; a kapcsoló alapból kikapcsolt. Valódi stagingpróba szükséges. A már kiosztott munkák optimalizálása, zárt chat melletti háttérfutás és szabad szöveg automatikus értelmezése nem része ennek a változatnak.

### 2026.10.17. — Google-forgalom és aznapi ajánlás valós stagingpróbája

**Mit:** az aznapi pont–pont Google-útbecslés aktuális forgalmat figyelembe vevő pillanatképet és időbélyeget kapott; az árazási útvonal továbbra is forgalom nélküli. A menetidőt a Google számolja, az AI engedélyezett elhelyezést választ.
**Miért:** a beosztási döntés valós közúti becslésre épüljön, anélkül hogy a későbbi indulás forgalmát, rakodási időt vagy garantált érkezést állítanánk.
**Bizonyíték:** [#115](https://github.com/beeaibuild/kontener-app/pull/115), staging `1ab4165`; valós geokódolás → Google → AI → friss előnézet: 32 perc/16,2 km becsült többletút. Végrehajtás nem történt; a T-2026-0020/0021 jelölt tesztfuvar lemondását DB-ellenőrzés igazolta. A próba során talált feliratjavítás [#116](https://github.com/beeaibuild/kontener-app/pull/116) CI-je zöld, még nincs telepítve.
**Nyitott:** production-kiadás és valódi operatív haszon mérése. Ez a próba nem igazol általános útvonal-optimumot vagy megtakarítást; a későbbi indulás forgalma eltérhet a lekéréskori helyzettől.

### 2026.10.16–17. — Egységes Sofőr V1 és fizikai push-próbák

**Mit:** az Android-app belépéssel indítja a GPS-t és a meglévő webes sofőrmunkafelületet; a tablet egyszeri rendszámhoz kötése megmarad. Kijelentkezéskor és budapesti éjfélkor lezárja a munkamenetet/követést; mentett feladatváltozás hangos és vizuális Android-értesítést kap, emberi vagy jóváhagyott AI-módosítástól függetlenül. Reggeli listabetöltés nem generál értesítést.
**Miért:** egyetlen sofőrappban legyen a feladatkezelés és a háttér-GPS, külön webes frontend újraírása nélkül; az értesítés az aktuális feladatot nyissa meg, a visszavontat ne lehessen elindítani.
**Bizonyíték:** [#114](https://github.com/beeaibuild/kontener-app/pull/114), staging és telepített Samsung APK; felhasználói próba: belépés/GPS, lezárt képernyős új feladat és módosítás hanggal/vizuális értesítéssel, részletmegnyitás, lemondás hanggal és érvénytelen feladat jelzése, kijelentkezés/újrabelépés. [#117](https://github.com/beeaibuild/kontener-app/pull/117): kézbesítési szűrés/retry tesztek és titkok nélküli hibakódos napló; CI zöld, még nincs telepítve.
**Nyitott:** a fizikai éjféli próba még nem lezárt; szerveroldali lejárat önmagában nem igazolja a natív GPS-szolgáltatás leállását. Két eszközös átosztás, további offline/késői push és teljes Samsung-műszakos megbízhatóság még igazolandó.

### 2026.10.14–16. — Aznapi AI-ajánlások GPS- és útvonalkontextussal

**Mit:** új `SAME_DAY_ASSIGNMENT_V1` ajánlástípus a mai, még nem megkezdett munkák elhelyezésére, beleértve a kiosztatlanokat; megbízható, friss GPS-pozíció és az azonos teherautóhoz tartozó munkasorrend figyelembevételével becsült közúti kitérő. Megmaradt a korábbi lejárt feladatokra vonatkozó ajánlás is. Az ajánlást ember hagyja jóvá; végrehajtás előtt újraellenőrzött preview szükséges.

**Miért:** a puszta lejártmunka-átszervezés önmagában kevés operatív értéket ad; a valós napi beosztásnál az erőforrás helyzete és a már vállalt következő munka is számít. A hiányzó útvonalbecslés **nem** teszi automatikusan érvénytelenné a jogszerű jelöltet: a pontosított prompt más rendelkezésre álló tényből is ajánlhat.

**Bizonyíték:** [#108](https://github.com/beeaibuild/kontener-app/pull/108) (39 célzott same-day integrációs teszt és provider-tesztek), [#109](https://github.com/beeaibuild/kontener-app/pull/109) (mentett bemeneteken ellenőrzött promptkorrekció), [#110](https://github.com/beeaibuild/kontener-app/pull/110) (integrált release). A #109 rövid prompttesztje **nem** bizonyít mérhető, valós logisztikai megtakarítást.

**Nyitott:** valós GPS/Google útvonalas smoke és operatív pilot; a becsült közúti kitérő nem garantált érkezési idő, nem tartalmaz forgalmat, rakodást vagy teljes hátralévő munkaidőt. Részletes szabály: [same-day-assignment-recommendation.md](same-day-assignment-recommendation.md).

### 2026.10.13–14. — Szállítólevél, konkurencia és tesztstabilitás (P01–P06)

**Mit:** a digitális szállítólevél jogosultsága aktív felhasználói módra és saját feladatra épül; adminoldali megnyitás, tranzakciós retry és auditált e-mail-helyreállítás készült. A budapesti üzleti nap és bizonylatév független lett a futtatókörnyezet időzónájától. A GPS-források olvashatóbbak; az eszközpárosítás újonnan 90 napos, a GPS-pontok megőrzése továbbra is 7 nap. Bekerültek lock-időzítési mérések és GPS admission-szerződéstesztek; az integrációs fixture-ek takarítása is szigorodott.

**Miért:** a több szerepkör, az e-mail-idempotencia, az időzónák és a konkurens írások nem pusztán UI-hibák: jogosulatlan művelethez vagy sérült üzleti állapothoz vezethetnek.

**Bizonyíték:** [#100](https://github.com/beeaibuild/kontener-app/pull/100), [#101](https://github.com/beeaibuild/kontener-app/pull/101), [#102](https://github.com/beeaibuild/kontener-app/pull/102), [#103](https://github.com/beeaibuild/kontener-app/pull/103), [#104](https://github.com/beeaibuild/kontener-app/pull/104), [#105](https://github.com/beeaibuild/kontener-app/pull/105), [#106](https://github.com/beeaibuild/kontener-app/pull/106), [#107](https://github.com/beeaibuild/kontener-app/pull/107). Részletek: [digitális szállítólevél](digital-delivery-note-v1.md) és [lock-mérések](lock-measurement-p06b.md).

**Nyitott:** a release utáni valós e-mail-/PDF- és párhuzamossági smoke eredményét deployment-azonosítóval kell dokumentálni; a zöld CI nem helyettesíti a production ellenőrzését.

### 2026.10.11–12. — Sofőrtablet és natív Android GPS-kísérő

**Mit:** kompakt, fekvő tabletmunkafelület az eredeti sofőr- és chatfunkciók újrahasznosításával. Külön Capacitor Android GPS-kísérő, natív háttérgyűjtéssel, helyi SQLite-pufferrel, HTTPS feltöltéssel, egyszer használatos párosítással és visszavonható eszközjogosultsággal. Az offline puffer feldolgozását javították. A meglévő sofőr UI továbbra is böngészőben marad.

**Miért:** böngészős geolocationtől nem várható megbízható, lezárt képernyős háttérkövetés. A pilotnál fontosabb az offline adatvesztés elkerülése és az eszköz-hozzáférés kontrollja, mint egy új sofőr-frontend építése.

**Bizonyíték:** [#95–97](https://github.com/beeaibuild/kontener-app/pull/97) (tablet UI), [#98](https://github.com/beeaibuild/kontener-app/pull/98) (natív GPS-kísérő és backend), [#99](https://github.com/beeaibuild/kontener-app/pull/99) (offline backlog és natív UI bridge). A GPS feature flag alapértelmezetten ki volt kapcsolva; a #98 eredeti célja staging-pilot volt.

**Nyitott:** Samsungon teljes műszak, képernyőzár, navigáció, mobilhálózat-kiesés, visszatöltés és akkumulátorhasználat mérése. Telepítési és elfogadási lista: [android-gps-pilot.md](android-gps-pilot.md).

### 2026.10.03–09. — Jóváhagyható, auditált Konti AI V1

**Mit:** védett operatív írások, atomi lejártmunka-átszervezés, változatlanul tárolt ajánlás és döntés, elutasítási ok, explicit preview/megerősítés, legfeljebb egyszeri végrehajtás. Az AI nem kap önálló kiosztási jogot. Később rövid UI, 7/30/90 napos mutatók és legfeljebb öt, kontextus alapján releváns, strukturált korábbi feedback-példa került be.

**Miért:** a nyelvi modell értelmezhet és ajánlhat, de a jogosultság, üzleti érvényesség és végrehajtás a szerver szabályozott, tranzakciós feladata. Az „elfogadom” és a „biztosan végrehajtom” két külön döntés.

**Bizonyíték:** [#88](https://github.com/beeaibuild/kontener-app/pull/88), [#89](https://github.com/beeaibuild/kontener-app/pull/89), [#90](https://github.com/beeaibuild/kontener-app/pull/90), [#91](https://github.com/beeaibuild/kontener-app/pull/91), [#92](https://github.com/beeaibuild/kontener-app/pull/92), [#93](https://github.com/beeaibuild/kontener-app/pull/93), [#94](https://github.com/beeaibuild/kontener-app/pull/94). A #94 release PR staging-bizonyítékot és egy ismert, külön kezelt UI-tesztkivételt dokumentál; a merge nem helyettesíti a production deployment ellenőrzését.

**További műszaki leírás:** [operational-recommendations.md](operational-recommendations.md), [operational-recommendation-metrics.md](operational-recommendation-metrics.md), [transaction-lock-protocol.md](transaction-lock-protocol.md).

### 2026.09.21–29. — Közös operatív kommunikáció és ügyfélkapcsolat

**Mit:** közös napi emberi chat jogosultsági korlátokkal, üzenetfrissítéssel, adminisztrálható írási politikával, rate limittel és privát képcsatolmányokkal (szerveroldali Cloudflare R2; képek kezelése és típus-/méretellenőrzése). Fekvő sofőrtablet-alap. Az ügyfélfiók a hozzá kapcsolt rendeléseket megjeleníti, és megjelent a regisztrációt lezáró e-mailfolyamat.

**Miért:** a Viber-féle kommunikációból fokozatosan saját, jogosultságkezelt munkafolyamat épüljön, anélkül hogy a sofőr megkapná a tervezési vagy más felhasználók privát adatait.

**Bizonyíték:** [#75](https://github.com/beeaibuild/kontener-app/pull/75), [#77–80](https://github.com/beeaibuild/kontener-app/pull/80), [#81](https://github.com/beeaibuild/kontener-app/pull/81), [#83](https://github.com/beeaibuild/kontener-app/pull/83), [#86](https://github.com/beeaibuild/kontener-app/pull/86), [#87](https://github.com/beeaibuild/kontener-app/pull/87). Műszaki anyag: [human-chat-reliability.md](human-chat-reliability.md), [private-chat-media.md](private-chat-media.md), [admin-chat-write-policy.md](admin-chat-write-policy.md).

### 2026.09.08–19. — Az AI architektúra megalapozása, read-only futtatás és értelmezési eval

**Mit:** verziózott, adatminimalizált AI-környezeti szerződés; visszavonható, aktív mód szerinti több szerepkörös session; tartós Conversation/Participant/Message modell; providerfüggetlen, read-only AI runtime. Ezt követte a Viber-értelmezés szabályzata, a strukturált kimenet, a tesztadatok és értékelési kapuk. A közös napi chat és az operátori/sofőri hozzáférések külön szabályozottá váltak.

**Miért:** előbb legyen ellenőrizhető a kontextus, a jogosultság és az értelmezés pontossága, csak azután szülessen valódi operatív írásra alkalmas döntéstámogatás.

**Bizonyíték:** [#48](https://github.com/beeaibuild/kontener-app/pull/48), [#49](https://github.com/beeaibuild/kontener-app/pull/49), [#51](https://github.com/beeaibuild/kontener-app/pull/51), [#52](https://github.com/beeaibuild/kontener-app/pull/52), [#54](https://github.com/beeaibuild/kontener-app/pull/54), [#58–63](https://github.com/beeaibuild/kontener-app/pull/63), [#66–71](https://github.com/beeaibuild/kontener-app/pull/71). Az értelmezés **nem jogosultsági igazságforrás**. Lásd [konti-viber-eval-safety-v2.md](konti-viber-eval-safety-v2.md).

### 2026.08.18–09.06. — A mini ERP napi működésének kiépítése

**Mit:** kihelyezési értesítés és „Megtelt a konténer” ügyféljelzés; belső fuvarfeladatok, admin számlázási alap/panel; diszpécsertérkép és jelölők; ügyfélregisztráció; kézi adminrendelés; e-mailes lemondási életciklus; heti kapacitás és szabályalapú, ember által elfogadható átszervezési javaslatok. A publikus landing is megújult; a `main` CI-kaput és a health-ben megjelenő commit-azonosítót kapott.

**Miért:** a rendelési alapfolyamatot operatív, kommunikációs és erőforrás-oldali rendszerként kellett továbbépíteni; a térkép és a kapacitás értelmezhetővé tette a napi munkát.

**Bizonyíték:** [#4](https://github.com/beeaibuild/kontener-app/pull/4), [#6](https://github.com/beeaibuild/kontener-app/pull/6), [#11](https://github.com/beeaibuild/kontener-app/pull/11), [#14](https://github.com/beeaibuild/kontener-app/pull/14), [#17](https://github.com/beeaibuild/kontener-app/pull/17), [#19](https://github.com/beeaibuild/kontener-app/pull/19), [#23](https://github.com/beeaibuild/kontener-app/pull/23), [#25](https://github.com/beeaibuild/kontener-app/pull/25), [#28](https://github.com/beeaibuild/kontener-app/pull/28), [#34](https://github.com/beeaibuild/kontener-app/pull/34), [#35](https://github.com/beeaibuild/kontener-app/pull/35), [#37](https://github.com/beeaibuild/kontener-app/pull/37), [#45](https://github.com/beeaibuild/kontener-app/pull/45). A #11 számlázási panel **nem** bizonyít önmagában külső számlázószolgáltatói kiállítást.

### 2026.07. — V1: a rendelési és üzemeltetési alap

**Mit:** publikus rendelés, alapárkalkuláció, adminrendelés-kezelés és kiosztás, sofőrfeladatok és teljesítési státuszok, konténer-/erőforrás-/feladatlisták, auditált tranzakciós állapotgépek. Ekkor a fizetés még mock volt. Verziózott Prisma-migráció, regressziós kapu, auth/CSRF-hardenelés, strukturált JSON-log és health-check, staging és mentés/visszaállítási bizonyíték került a folyamatba.

**Miért:** először helyes és visszaállítható üzleti magot kellett építeni, nem AI-val vagy látványos diszpécserfunkcióval kezdeni.

**Bizonyíték:** történeti [v1 állapotjelentés](release-state-v1.md), [v1 regression](final-regression-v1.md), [backup/restore proof](backup-restore-proof-p36.md), [logging és health](logging-health-p33.md), [migrációk](migration-workflow.md). Ezek a júliusi állapotot igazolják, **nem** a mai teljes funkciókészletet.

## Tartósan érvényes műszaki döntések

| Döntés | Indok és határ | Forrás |
| --- | --- | --- |
| AI javasol; ember hagyja jóvá és erősíti meg a végrehajtást | Modellkimenet nem közvetlen üzleti parancs; szerveroldali friss jogosultság és állapotellenőrzés kötelező | [#89–91](https://github.com/beeaibuild/kontener-app/pull/90) |
| Az operatív írásokhoz egységes lock-sorrend és célzott admission tartozik | Konkurens módosítások, holtpontok és stale állapotok kontrollja; nem általános automatikus retry | [#75](https://github.com/beeaibuild/kontener-app/pull/75), [#88](https://github.com/beeaibuild/kontener-app/pull/88) |
| Az LLM csak engedélyezett, adatminimalizált és verziózott kontextust kap | Kontextus és utasítás/jogosultság elkülönítése; a nyers chat nem jogosultsági forrás | [#48](https://github.com/beeaibuild/kontener-app/pull/48), [#54](https://github.com/beeaibuild/kontener-app/pull/54) |
| Natív háttér-GPS és újrahasznosított webes sofőr UI | A #98 külön kísérője után a #114 egységes APK-ban nyitja a webes felületet; nincs párhuzamos sofőr-frontend | [#98](https://github.com/beeaibuild/kontener-app/pull/98), [#114](https://github.com/beeaibuild/kontener-app/pull/114) |
| Séma kizárólag verziózott, előrefelé additív migrációval | Production adatok megőrzése és reprodukálható bevezetés | [migration workflow](migration-workflow.md), [#110](https://github.com/beeaibuild/kontener-app/pull/110) |
| PR/CI/staging igazolás és production deploy külön esemény | A merge és a zöld teszt nem bizonyítja az élő szolgáltatást | [#94](https://github.com/beeaibuild/kontener-app/pull/94), [#110](https://github.com/beeaibuild/kontener-app/pull/110) |

## Nyitott bizonyítási pontok a napló indulásakor

1. **Production release:** a #110-hez tartozó production backup, migráció, Render deploy, commit-SHA ellenőrzés és production smoke tényét külön rögzíteni kell. Addig ne legyen „production igazolt” címke.
2. **GPS helyszíni pilot:** teljes valódi Android/Samsung műszak, képernyőzár, háttérkövetés, offline helyreállás, akkumulátor és eszköz-visszavonás; lásd [pilot terv](android-gps-pilot.md).
3. **AI valós használhatósága:** nem csak séma és működő végrehajtás, hanem valódi diszpécseri döntések, visszajelzés és mérhető napi operatív haszon. A #92 telemetria alkalmas követésre, de önmagában nem bizonyít megtakarítást.
4. **Külső szolgáltatók:** éles Google-útvonal, e-mail-kézbesítés és egyéb integrációk csak saját futási bizonyítékkal nevezhetők ellenőrzöttnek.

Ezek a 2026.10.16-i induló nyitott pontok, **nem automatikus feladat- vagy hibajegyek**. Lezárásukhoz dátum + deploy/teszt-/pilot-hivatkozás kell.

## Továbbvezetési szabály — minimális adminisztráció

Csak **érdemi mérföldkő vagy termék-/architektúradöntés** esetén kerüljön új bejegyzés ide (általában release-enként, nem minden apró PR-nél). Legyen maximum 5–8 sor: **dátum, mit, miért, bizonyíték, mi maradt nyitva**. A részletes diff, teszteredmény, audit és rollout továbbra is a PR-ben/runbookban marad. Legújabb bejegyzés legfelül. A belinkelt dokumentumok és a termék állapota ne kapjanak bizonyítatlan „kész/éles” jelzőt.

### Bejegyzéssablon

```md
### ÉÉÉÉ.HH.NN. — Mérföldkő neve

**Mit:** [mi változott a termékben vagy architektúrában]
**Miért:** [melyik konkrét problémát oldja meg; fő döntés]
**Bizonyíték:** [PR és konkrét CI/smoke/deploy/pilot hivatkozás]
**Nyitott:** [1–2 érdemi, még nem igazolt pont; vagy „nincs ismert”]
```

**Ne kerüljön ide** jelszó, token, kulcs, személyes ügyféladat, nyers prompt vagy éles adatbázisrekord. A szakmai döntés és az ellenőrizhető bizonyíték az érték, nem a többoldalas munkanapló.

### 2026.10.18. — Kétfeladatos AI-csere és fizikai éjféli próba

**Fizikai átvétel:** a felhasználó reggeli képei belépőképernyőt, inaktív műszakot, 23:59:20-as helyi utolsó pontot és üres feltöltési sort mutattak. Újrabelépés után 09:37:33-as friss GPS-pont (17 m), aktív követés és üres sor látszott; a rendszámkötés megmaradt. A korábban nyitott éjféli → reggeli újrabelépés próba ezzel lezárva. Ez nem ezredmásodperces natív leállási mérés.

**Fejlesztés:** legfeljebb két mai, tervezett feladat pontos cseréje két külön sofőr–autó páros között. Közös kapacitásellenőrzés teli autóknál is, Google által számolt két hátralévő út összehasonlítása, legalább 5 perc és 20% előny, AI-választás a megengedett cserékből, egy privát chat-ajánlás, friss előnézet és atomikus végrehajtás. A terhelés feladatszám, nem munkaidő. Végrehajtás nélkül nincs kiosztásváltozás. Új adatbázis-migráció és APK nem szükséges.

**Állapot:** külön fejlesztési ág; automatikus keresés alapból kikapcsolt (`KONTI_AI_PAIR_REASSIGNMENT_ENABLED`). Staging- és production-telepítés nem történt ebben a lépésben. Tesztbizonyíték a PR-ben. A két-tabletes fizikai próba és a tényleges üzemi megtakarítás mérése továbbra is nyitott.

## 2026-10-19 – Teljes kaszkád főkapcsoló (#134)

- Az admin OFF most az időadatok olvasását/mentését, kézi számítást, AI-alternatívaválasztást és a háttérfigyelőt/kézbesítést együtt tiltja.
- Minden további kézi útvonalkérés, AI-hívás és eredmény előtt generációellenőrzés; a futó kérés nem vonható vissza, régi eredmény nem adható ki. Az időadatmentés a kapcsolással közös DB-zárat használ.
- A panel OFF mellett nem kér időadatot, nem mutat kézi vezérlőket vagy korábbi eredményt. Új csomagfüggőség nincs.
- Helyi validáció: 1986 unit teszt, típusellenőrzés és sorhosszellenőrzés sikeres. Új tesztek: master gate, visszakapcsolás/generáció, tiltott időadatmentés és teljes panelkapcsolás. Teljes CI és stagingátvétel külön rögzítendő.


### #134 staging kiadás és átvétel – 2026-10-19 09:58

- Teljes CI zöld: 3221 Vitest / 174 UI teszt, típusellenőrzés, migrációs paritás, production és Docker build. #134 merged: `38e033c23479404e2d93a35f583049a4a3ffeba2`.
- Staging live 09:54:31; migráció és health rendben. A 09:31-es DB-export kész. 62 Order / 63 Job / 36 TransportTask változatlan.
- Külön kétperces Render Cron: `crn-dbason6q1p3s73cvce30`; natív Node 22 trigger, új függőség nélkül. A telepítés közbeni első 404 után a 09:56-os és 09:58-as kör sikeres.
- Teljes admin főkapcsoló és kézi API-tiltás igazolt. Bezárt diszpécseroldal mellett egy OK háttérfutás; OFF→ON→OFF után a következő cron nem indít új számítást. Végállapot OFF, generáció 4, 0 úthívás/értesítés; domain adatokhoz nem nyúltunk.
- Bizonyítási határ: üres mai terv, nem élő GPS-es riasztási pilot. Stagingből nem ment levél, production változatlan. Részletes átvétel: docs/cascade-monitor.md.

## 2026-11-10 — Authentication audit findings 1–3

Password reset now increments authVersion in all four reset paths. Existing web,
tablet and GPS credentials lose authority; permanent truck/tablet binding remains.
Native login validates tablet authority before checking passwords and audits 401
failures without storing credentials. Binding links the setup device and logs
DRIVER_TABLET_BOUND. Revoke invalidates both associated GPS devices and the legacy
setup device with the tablet ID before deleting the binding. Tablet setup secrets
cannot start legacy GPS shifts; driver-app secrets cannot enroll tablets.
Dispatch-run enrollment eligibility uses the existing GPS 24-hour window.

Added regression coverage for password resets, unknown-tablet password probes,
legacy setup-device revocation, GPS shift rejection and replacement enrollment.
CI and P63 results must be checked on this PR head; no deployment is claimed.
Render was read-only checked: staging and production run 8c7c0b6 (#178).
Migration table contents and proxy X-Forwarded-For handling remain unverified.

## 2026-11-10 — AI audit findings 4–7

- ANSWER generation records GENERATED; SUCCEEDED is committed with the answer
  binding. Unbound/deleted answers cannot receive trace feedback.
- Provider work starts after audit/authority commit, with rollback regression.
- Explicit feedback retries retain event IDs, synchronous double submissions
  are blocked, and consecutive identical server decisions are deduplicated.
- Replay/recovery uses original audit identity in response header and events.
- Added integration and component regressions. CI/P63 validation pending on
  the new PR head; no deployment claimed. No migration or new dependency.

## 2026-11-11 — Audit integrity and remaining operational risks

- Added database audit lifecycle/immutability and append-only feedback guards,
  CHECK constraints and explicit 90-day retention (dry-run default, bounded batch).
- Staging proxy test confirmed spoofed leftmost XFF in LOGIN_FAILED audit.
  Shared account-only web/native login ceiling prevents credential-budget bypass
  through IP rotation. Trusted IP attribution remains unverified.
- Added opt-in slow/failed operational lock telemetry and native contention proof;
  actual staging recommendation/driver timing remains to be observed.
- At #182 the default budget was still 5 seconds; #183 superseded it with
  a 40-second operational-write default. No production maintenance/deletion.
  CI/P63 validation pending on the new PR.


## 2026-11-11 — #182 / #183 and independent recheck corrections

- #182 added AI audit database guards, retention and forwarded-IP mitigation.
- #183 raised the operational-write default from 5 to 40 seconds; the native
  contention regression covers a 6.5-second holder. Real staging pool pressure
  with three simultaneous drivers is still unmeasured.
- Invalid native tablet credentials no longer consume shared login-account
  limits or create LOGIN_FAILED audit rows. Valid tablets retain account limits.
- A new migration permits reference-only FK nulling on STARTED audit rows;
  immutable evidence and lifecycle protections remain in force.
- Legacy mobile GPS shift opening is disabled. Existing legacy shifts lapse
  and cannot block tablet login; pairing for tablet enrollment remains available.
- Added targeted regression coverage. CI/build/P63 results and deployed commit
  must be checked after the final PR head exists; no deployment is claimed here.

Validation before PR: native Prisma 5.22 / local PostgreSQL 18 applied all 54
migrations; 66/66 targeted audit/tablet/GPS integration tests passed. Typecheck
and the line-length gate passed; all 2203 unit tests passed. PostgreSQL 16
and full CI remain the CI gate.
Read-only Render check found staging and production live on #183 / 486607b;
these recheck corrections have not been deployed.

## 2026-11-11 — Közös admin feladatlista

- A Feladatok oldal a konténeres Job és a belső TransportTask sorokat együtt
  listázza; a fuvar saját száma, útvonala, rakománya és saját adatlaplinkje jelenik meg.
- Közös státusz-, típus-, sofőr-, autó-, dátum- és szövegszűrés, adatbázisos
  közös rendezés/lapozás és összesítés. A belső fuvarnak nincs ügyfélrendelése.
- A naptári dátumszűrés a közös UTC date-only konvenciót követi.
- Négy PostgreSQL-integrációs próba zöld (vegyes lista, szűrés, keresés,
  veszteségmentes közös lapozás); külön böngészős regresszió bekerült a P63-ba.
- Nincs új függőség, migráció vagy üzleti adatot módosító művelet.
  A teljes CI/P63 és a kiadás eredménye a PR-en rögzítendő.

## 2026-11-13 — Heti fagyasztott telemetria-kimutatás

- Új `TelemetrySnapshot` tábla, append-only DB-triggerrel; hetente egy sor,
  kanonikus JSON SHA-256 lenyomattal. Letöltés .json és .md formában a master adminban.
- Konti AI-hívások típus és verzió szerint (commitSha, promptHash), döntések,
  rendelésfelvételi mezőpontosság; a Telemetria oldalon élőben is látszik.
- A migrációt a sandboxban valódi PostgreSQL 16-on futtattuk az összes korábbi
  migrációval együtt: UPDATE/DELETE tiltva, azonos hét kétszer elutasítva,
  karbantartási törlés működik. A hash JSONB-oda-vissza út után is egyezik.
- Unit tesztek zöldek a sandboxban, negatív kontrollal. Prisma-engine a sandboxban
  nem tölthető le (binaries.prisma.sh 403), ezért typecheck, build és a DB-s
  integrációs teszt (`tests/integration/telemetrySnapshot.test.ts`) a CI-ra, illetve
  a lokális kapura vár. Render Cron Job még nincs létrehozva.

## 2026-11-15 — Telemetria-kimutatás: első kiosztási döntések (séma 2)

Hiba: a staging W45 próbafagyasztása után a telemetria oldal az INITIAL_ASSIGNMENT
hívásoknál „elfogadva nincs adat (0/0)” értéket mutatott. A döntés létezett, de a kiosztás
auditjában (`JOB_ASSIGNED_KIHELYEZES`), nem az `AiOutputFeedback` táblában, és az 1-es séma
csak ez utóbbit olvasta. A pilot fő AI-funkciójánál így a döntések kimaradtak volna a
2026-11-16-i első éles fagyasztásból, amely utólag nem javítható.

Javítás: a lánc a mérési auditon keresztül, csak VERIFIED hivatkozással; ACCEPTED/MODIFIED
verziónként, módosított mezők, eltérési ok, kézi kiosztások külön. Séma: `telemetry-snapshot/2`.

Bizonyíték: unit tesztek (érvényes és hamis/INVALID/ABSENT hivatkozás, kohorsz, régi séma
kiírása), integrációs teszt visszagörgetett tranzakcióban valódi DB-vel (CI).
Nyitott: a staging W45 1-es sémájú marad, ahogy kell.
