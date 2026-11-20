# Globális operatív lock mérése (P06-B)

Állapot: **csak mérés**. A zárolás, a GPS-szolgáltatás és az üzleti viselkedés
változatlan. Cél: megmutatni, mennyi ideig tartják a `runOperationalWrite`
műveletek a közös `(74000001, 1)` EXCLUSIVE admissiont, és mennyit várnak
mögötte a többi írók.

A hook alapból ki van kapcsolva. Ez a csomag csak a teszt- és mérési
folyamatban kapcsolja be, ezért **stagingből még nincs megfigyelésünk**.

## Mit mér a hook

`lockTiming.sink` (`src/lib/transactions/lockProtocol.ts`) alapból `null`; ekkor
nincs időmérés és nincs viselkedésváltozás. A sink jelenlétét a tranzakció
**elején** döntjük el: ami mérés nélkül indult, azt menet közben bekapcsolt sink
sem méri, és ami mérve indult, arról akkor is jön esemény, ha közben
kikapcsolták. Minden mért `runWithTransactionLockContext` tranzakcióról egy
eseményt kapunk, sikerről és hibáról is:

| mező | jelentése |
| --- | --- |
| `site` | a hívó első stack-framje a lockmodulokon kívül (`függvény (fájl:sor)`) |
| `acquired` | megszerezte-e a tranzakció az admissiont; `false`, ha a várakozás hibával ért véget |
| `waitMs` | az admission megszerzésére várt idő; **sikertelen várakozásnál is** a ténylegesen megfigyelt idő, 0, ha sosem kértük |
| `holdMs` | az admission megszerzésétől a commit/rollback végéig; **0, ha az admission nem lett meg** |
| `txMs` | a teljes interaktív tranzakció, kapcsolatvárakozással és commit-tal |
| `outcome` | `commit` vagy `error` |

A hiba az eredeti formájában jut el a hívóhoz; a sink kivétele nem változtat a
viselkedésen. A `holdMs` az, amit a többi író végül "fizet": a lock a commitig él.

## Futtatás

Mindkét mód külön tesztadatbázist igényel (`TEST_DATABASE_URL`, lásd
`docs/testing-v1.md`). Nem része az `npm run test`-nek és a CI-nek.

**A nagy adatmennyiséget létrehozó és telítéses mérések kizárólag izolált
tesztadatbázison futhatnak, működő stagingadatbázison soha.** A futtatók ezt
ellenőrzik: a `DATABASE_URL`-nek helyi hostra kell mutatnia, az adatbázis nevének
`test`, `ci` vagy `delivery_note` részt kell tartalmaznia, különben a mérés
hibával leáll (`assertIsolatedDatabaseUrl` a `tests/measure/lockTimingSink.ts`-ben).
A stagingen tervezett mérés csak a lenti, passzív hookot használja.

```bash
# Célzott GPS-forgatókönyvek (S1–S7)
KONTI_LOCK_RESULTS_FILE=/tmp/lock-gps.json \
  npx vitest run -c vitest.lock-measure.config.ts

# A meglévő integrációs tesztek összes írótranzakciója hívóhely szerint
HUMAN_MESSAGE_USER_RATE_LIMIT=10000 HUMAN_MESSAGE_CONVERSATION_RATE_LIMIT=10000 \
KONTI_LOCK_SURVEY=1 KONTI_LOCK_TIMING_FILE=/tmp/lock-survey.jsonl \
  npx vitest run -c vitest.lock-measure.config.ts
node tests/measure/summarize.mjs /tmp/lock-survey.jsonl
```

A survey nem futtatja azokat a teszteket, amelyek maguk tartják az admissiont
vagy maguk kezelik a sinket (`operationalWriteAdmission`, `lockProtocol`,
`dispatchRunRace`, `cancelPaymentRace`, `lockTiming`, `gpsAdmission`); a normál CI-ban ezek
mind futnak.

Állítható: `KONTI_LOCK_MEASURE_RUNS` (S1 ismétlések, alap 40),
`KONTI_LOCK_MEASURE_SECONDS` (S4 lépésenként, alap 5),
`KONTI_LOCK_MEASURE_S2_REPS` (alap 5), `KONTI_LOCK_MEASURE_S3_REPS` (alap 3),
`KONTI_LOCK_MEASURE_S6_REPS` (alap 3), `KONTI_LOCK_MEASURE_S7_SECONDS` (alap 90).

## Terhelési helyzetek

A pilot **3 táblettel** indul; a **10 eszköz tartalék terhelési forgatókönyv**,
nem terv. A két üzemmódot külön mérjük:

- **Normál működés:** a kísérőalkalmazás háttérfeltöltése 15 másodperces
  ütemezésű (`scheduleWithFixedDelay`), és ha van elküldetlen pont, azt, különben
  az állapotot kéri. Az S7 csak a pontfeltöltéses ciklust modellezi; az
  állapotkérést nem. A 3 másodperces UI-frissítés csak helyi állapotot olvas, a
  szervert nem éri el, ezért a mérésben nem szerepel.
- **Offline feltöltési kitörés:** hálózatkimaradás után az eszköz a
  `GpsRetryJob` futásonként legfeljebb 10 darab, egyenként legfeljebb 100 pontos
  batch-et tölt fel egymás után (S3).

## Forgatókönyvek

- **S1** minden GPS-művelet tartási ideje egyedül futva.
- **S2** az `ingestGpsPoints` retenciós törlése: tartási idő a lejárt pontok
  számának függvényében (0 – 250 000; mintaszám a táblában).
- **S3** offline kitörés 3 és 10 eszközzel, ismételve; közben egy másik író
  admission-várakozását és a hívó által látott késleltetést mérjük.
- **S4** telítési görbe: 1–128 párhuzamos kliens, 70% pontfeltöltés / 30% állapot.
  A v2 nem futtatta újra; a v1 eredménye érvényes, a kód változatlan.
- **S5** utasításszám és Postgres-oldali SQL-idő műveletenként (csak ha a
  `pg_stat_statements` telepítve van). A v2 nem futtatta újra, a kód változatlan.
- **S6** egy hosszan tartó író mögé sorolt GPS-kérés, **az állapotkérés és a
  pontfeltöltés külön holderrel**, 1,5 s és 6,5 s tartással, az alapértelmezett
  Prisma-korlátokkal. Minden esetnél külön látszik a várakozás, a tartás, a
  tranzakció teljes ideje, a hívás teljes ideje és a kimenet. A kérés
  sorbanállását a PostgreSQL maga igazolja (`pg_blocking_pids`).
- **S7** 15 másodperces ütemű, **kizárólag pontfeltöltést** (`ingest-1`) modellező
  forgatókönyv: egyenletesen szórt fázissal 3 és 10 eszköz, mellette egy független
  író 250 ms-onként. A 90 s-os futás eszközönként 6 kérést indít, mind feltöltés;
  állapotkérés nincs benne. A lock foglaltsága = **az összes** tartási idő összege
  (a független próbaíróé is) / eltelt idő, tehát nem csak a GPS-eszközök terhelése.

## Hogyan olvassuk

- A hold nem csak SQL. A Postgres-oldali SQL-idő a tartásnak töredéke (S5: 0,08–2,6
  ms 9–15 utasításnál), a maradék alkalmazás-, adapter- és ütemezési költség:
  driver/adapter, Node-ütemezés, protokoll-körök és commit. Ez **nem**
  azonosítható tisztán hálózati round-trippel, és élesben a hálózati rész is
  hozzájön.
- A retenciós törlés a lejárt pontok számával nő, és a lockon belül fut: a
  megfigyelt legnagyobb érték 0,6 s volt (a v1 egy futásának p95-e), tehát a
  „legfeljebb 0,4 s" állítás nem tartható. A v2 250 000 lejárt pontnál 5
  mintából p50 ≈ 0,45 s, max ≈ 0,55 s.
- A tabletek újrapróbálkozása **kódáttekintési megállapítás**, nem szerveroldali
  mérés eredménye: a `GpsStore` a nyugtázatlan pontokat megtartja, és csak az
  `acceptedIds` után törli; a `GpsRetryJob` exponenciális backoffal ütemez újra.
  A mérés csak azt mutatja, mit kap a szerver, azt nem, mit csinál ezután a
  kliens.
- Az alapértelmezett Prisma-korlátok (`maxWait` 2 s, `timeout` 5 s, a lockvárakozás
  a timeoutba számít) mellett 1,5 s-os holder mögött a kérés sikerült, 6,5 s-osnál
  elbukott és nem írt semmit (`acquired=false`, `holdMs=0`). A sandboxban a hiba
  akkor érkezik meg, amikor a holder elengedi a lockot, nem 5 s-nál: ez a
  wasm/driver-adapteres Prisma viselkedése lehet, natív enginen nem mértük.

## Mit nem bizonyít a helyi mérés

Kis mintaszámok (a táblák `n` oszlopa mutatja; egy-egy eset 3–10 ismétlés, a
terhelési futások 90 s-osak), egyetlen sandbox, egyszeri futás. Az eredmény
nagyságrendet és hibamódot mutat, nem megbízási intervallumot.

- Prisma a sandboxban **wasm/driver-adapteren** (`@prisma/adapter-pg`) fut, nem a
  natív query engine-en; a kapcsolatpool és a `maxWait` viselkedését natív
  CI-ben vagy stagingen kell megmérni.
- A PostgreSQL `name` típusát az adapter nem tudja deszerializálni, ezért a
  `tests/integration/lockProtocol.test.ts` a sandboxban fájlszinten elbukik (a
  tiszta alapon is); ott a natív CI az érvényes eredmény.
- Helyi Postgres ugyanazon a gépen, kicsi táblák, hálózati késleltetés nélkül.
  Az abszolút ezredmásodpercek nem vihetők át élesre.
- A „350 tablet" jellegű szám csak helyi extrapoláció volt, **nem igazolt
  kapacitás**, nem szabad tervezési alapnak használni.

Natív CI-t vagy stagingmérést igényel:

- a valós adatméretű táblákon futó írók (különösen az AI-ajánlás műveletek
  `buildRecommendationCandidates` lépései) tartási ideje;
- a valódi Prisma pool mérete (`connection_limit` nincs beállítva, így
  alapból 2×vCPU+1) és az, hogy mekkora terhelésnél kezdenek hibázni a kérések;
- a tényleges hálózati round-trip az alkalmazás és az adatbázis között.

## Javaslat a stagingmérés be- és kikapcsolására (nincs megvalósítva)

Ez a csomag **nem** valósítja meg, csak a legkisebb lehetséges módot írja le,
hogy a döntés előtt lehessen vitatni.

- Egy környezeti változó (például `LOCK_TIMING_LOG=1`) a szerver indulásakor
  beállítaná a `lockTiming.sink`-et egy olyan függvényre, amely eseményenként
  egy strukturált naplósort ír (`site`, `acquired`, `waitMs`, `holdMs`, `txMs`,
  `outcome`), személyes adat és SQL-paraméter nélkül.
- Kikapcsolás: a változó törlése és újraindítás. Sémaváltozás, új függőség és
  új absztrakció nem kell; az alapértelmezés kikapcsolt marad.
- Mivel a sink-döntés tranzakciónként az elején történik, a be- és kikapcsolás
  futás közben sem hagy félig mért tranzakciót.
- A kimenetet a meglévő naplógyűjtés viszi; az aggregálás (`site` szerinti
  percentilisek) a `tests/measure/summarize.mjs` mintájára offline történhet.
- Nyitott kérdés a döntéshez: a naplótérfogat és a minta (esetleg csak a
  lassú vagy sikertelen tranzakciók naplózása).
