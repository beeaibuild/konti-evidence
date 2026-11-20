# Digitális szállítólevél V1

Kiinduló repository-verzió: `9cb202fde89db34f90b1e8bc4d9a611cbe74dbaf`.
Production service, DB, environment, secret, DNS és deploy módosítása nélkül.

## Meglévő rendszer és illeszkedés

Next.js 15 App Router, PostgreSQL / Prisma, JWT session, DRIVER / ADMIN / MASTER_ADMIN.
A vevői Order feladata a Job; a TransportTask külön, vevő nélküli belső fuvar.
A meglévő Resend emailGateway és az APP_ENV staging email-tiltás megmarad.

A V1 rendeléshez kapcsolt Jobonként egy végleges szállítólevelet kezel.
Nem változtat rendelés-, konténer-, fizetési vagy feladatstátuszt.
A sofőr saját feladatkártyáján, az admin a rendelés adatlapján nyitja meg.
A lezárt dokumentum ugyaninnen megnyitható és letölthető.

## Adatok és papírminta

- Szállító: az alkalmazás közzétett ÁSZF v1.0 szerinti vállalkozó és székhely.
- Vevő/cégnév/adószám/email: az Order Customer adatai.
- Szállítási cím: Job.address; nem állítjuk, hogy ez számlázási cím.
- Rendelésszám, feladatazonosító, járat, sofőr, rendszám: meglévő kapcsolatok.
- Kihelyezésnél a javasolt tétel a konténertípus, 1 db; a sofőr ellenőrzi.
- Más feladatnál a hulladékjelleg előtöltött, a tényleges mennyiséget a sofőr adja meg.
  A konténer névleges térfogatát nem tekintjük igazolt hulladékmennyiségnek.
- A rakomány, mennyiség és mértékegység az aláírás előtt módosítható;
  legfeljebb három tizedesjegy tárolható, így a PDF nem kerekít észrevétlenül.
- A4 fekvő PDF: a minta fejlécét, vevő/szállító blokkjait, hét tételsorát,
  mennyiség-/egységár-/értékoszlopait és alsó átvételi blokkját követi.
  Hosszabb rakománynév esetén az első sor magasabb.
- ÁFA, besorolási kód, bankszámlaszám, egységár és érték nem kerül kitalálásra.
  A szolgáltatás rendelési ára nem automatikusan a hulladék egységára.

## Aláírás és lezárás

1. A sofőr ellenőrzi az előtöltött adatokat és a rakományt.
2. Az ügyfél érintéssel vagy tollal aláír a canvas területén.
3. A sofőr lezárja. Az adatokhoz való visszatérés törli a korábbi aláírást.
4. A szerver ismét ellenőrzi az aktív felhasználót, szerepkört és feladatkiosztást.
5. A forrásadatok SHA-256 lenyomata megakadályozza, hogy időközben megváltozott
   adatokra kerüljön a korábbi aláírás. Ilyenkor új ellenőrzés és aláírás szükséges.
6. Serializálható tranzakció és Job sorzár mellett készül a PDF, majd az adatpillanatkép,
   az aláírás pontjai, a végleges PDF, annak SHA-256 lenyomata, a szerver által rögzített
   aláírás/lezárás ideje, a lezáró felhasználó és az email-sor együtt kerül mentésre.
7. A PDF-generálás hibája teljes rollbacket okoz. Dupla kérés ugyanazt a dokumentumot adja.

A kézi aláírás grafikus átvételi igazolás; nem minősített elektronikus aláírás.
A szerver az aláírás beérkezésének/lezárásának időpontját rögzíti, nem bízik a készülék órájában.
Az aláírás méretkorlátos, normalizált pontokból áll, üres/félreformázott tartalom elutasítva.
A PDF-ben a rajz méretarányait megtartjuk. Nincs nyilvános dokumentum-URL vagy tárhely.
A PDF bájtjai BYTEA mezőben vannak: a dokumentum nem függ a Render ideiglenes fájlrendszerétől.

## Migráció

`20260905190000_digital_delivery_notes`:

- `DeliveryNote`: egyedi Job és szállítólevélszám, Order/User FK,
  snapshot/signature JSONB, időpontok, PDF BYTEA, SHA-256.
- `DeliveryNoteEmail`: dokumentumonként egy küldési rekord, címzett-pillanatkép,
  állapot, próbálkozások, időzítés és foglalás, szolgáltatói azonosító, rövid hibakód.
- Sorszámsequence: `SZ-év-000001`; rollback után számozási hézag lehetséges.
- Adatbázis-trigger tiltja a végleges dokumentum UPDATE/DELETE műveletét.
- Külön ellenőrzés biztosítja a Job/Order megfelelést és az email-címzett változatlanságát.
- Nincs meglévő rendelési adat átírása és nincs destruktív migráció.

Az adatbázis tulajdonosa a triggereket adminisztratív úton eltávolíthatja;
V1-ben az alkalmazáson keresztüli és közvetlen szokásos SQL-módosítás tiltott,
a PDF olvasásakor és email-küldéskor sértetlenségi ellenőrzés fut.

## Email és helyreállítás

Az email a dokumentum-tranzakció commitja után indul. Hibája nem vonja vissza a PDF-et.
Az adapter a pontos archivált PDF-et küldi base64 mellékletként a lezáráskori vevői címre.
A küldés állapota a dokumentum oldalán látszik; az admin FAILED állapotot újrapróbálhat.

A feldolgozó atomi foglalást, kétperces lejáró foglalást és stabil idempotenciakulcsot használ.
Legfeljebb öt automatikus próbálkozás, exponenciális várakozással. A lejárt foglalás helyreállítható.
23 óra után a bizonytalan küldés NEEDS_REVIEW lesz, és nem indul vak újraküldés:
[a Resend idempotenciavédelme 24 órás](https://resend.com/docs/dashboard/emails/idempotency-keys).
Az admin újrapróbálás ezt az időablakot nem nullázza.

### Operátori helyreállítás (P03)

Egy végpont (`POST /api/jobs/[id]/delivery-note/retry`), három kifejezett művelet. Mindegyik
a látott állapotot is elküldi (`expectedCycle`, RETRY-nál `expectedStatus`), így a dupla kattintás
vagy közbejött változás 409 `STATE_CHANGED`, nem második küldési ciklus.

| Művelet | Honnan | Hová | Idempotenciakulcs |
| --- | --- | --- | --- |
| `RETRY` | FAILED, PENDING | PENDING, `attemptCount` 0 | változatlan; `firstAttemptAt` marad (ez az ablak kezdete) |
| `REENABLE` | DISABLED | PENDING | változatlan |
| `RECOVER_NEW_CYCLE` + `confirmNotDelivered: true` | NEEDS_REVIEW | PENDING, `cycle`+1, számlálók és időpontok újraindulnak | új: `delivery-note-<id>-c<cycle>` |
| `RETRY`/`REENABLE` lejárt ablaknál | FAILED, PENDING, DISABLED | NEEDS_REVIEW (rögzítve, auditálva), válasz 409 `IDEMPOTENCY_WINDOW_EXPIRED` | – |

Írás nélkül elutasítva: SENT (`ALREADY_SENT`), PROCESSING (`SEND_IN_PROGRESS`; a lejárt foglalást az
ütemező ugyanazzal a kulccsal folytatja), és minden olyan konfiguráció, amely a küldést nem engedi
(`EMAIL_SENDING_DISABLED`: staging, `EMAIL_MODE` ≠ production, hiányzó provider-beállítás).
Megerősítés nélküli helyreállítás: 400 `INVALID_EMAIL_ACTION`. Az 1. ciklus kulcsa a korábbi
`delivery-note-<id>`, így a már futó rekordok nem változnak; a ciklus az adatbázisban nem csökkenhet.

A friss OPERATOR jogosultság, a sorzárak (P01: UserSession → User FOR SHARE, majd az email-sor
FOR UPDATE), az állapotváltás és az audit egy admission-tranzakcióban fut; a provider csak commit
után. Audit: `DELIVERY_NOTE_EMAIL_RETRY`, `…_REENABLED`, `…_RECOVERY_NEW_CYCLE`, `…_WINDOW_EXPIRED`,
`entity=Order`; az adatban a bizonylat, a teljes korábbi és új állapot, új ciklusnál a megerősítés
és a régi/új kulcs. A válasz a küldési kísérlet utáni tényleges állapotot adja; a felület csak
SENT esetén jelez sikert.

A küldő (`deliverNoteEmail`) minden állapotírása feltételes arra, amit beolvasott: ugyanarra a
ciklusra (régi ciklusból származó munka nem írhat egy közben megnyitott újba), és csak foglalás
nélküli vagy lejárt foglalású sorra. Aktív foglalású sorról senki más nem dönt, az
idempotenciaablak lejárta után sem: a providerhívást indító worker a saját tokenjével, az
eredeti ciklusban rögzíti az eredményt. Lejárt foglalás átvételekor (NEEDS_REVIEW) a foglalás
törlődik, így a régi tulajdonos késői eredménye nem írhat.

A meglévő `/api/internal/overdue-notifications/run` feldolgozó bekapcsolt feature mellett
futtatja ezt a sort is. Külön célzott endpoint: `POST /api/internal/delivery-notes/run`,
ugyanazzal az `OVERDUE_CRON_SECRET` Bearer hitelesítéssel. Automatikus későbbi próbálkozáshoz
az ütemezőnek ténylegesen hívnia kell valamelyik endpointot; új fizetős cron nem készült.

**Minden nem-production környezetben a V1 szállítólevél-email tiltott**, akkor is, ha
valaki Resend-kulcsot állít be. DISABLED nem jelent sikeres elküldést. Integrációs tesztben
kizárólag injektált, hálózat nélküli fake gateway használható. Éles vevő nem kapott tesztlevelet.

## Fejlesztési ellenőrzés

Izolált, helyi PostgreSQL 16 tesztadatbázis (`delivery_note_dev`), kizárólag mesterséges
`example.invalid` címzettek. A tesztek helyi, `delivery_note` nevű DB-t követelnek meg.
A végleges rekordok törlésének tiltása miatt a tesztadatbázis eldobható;
a tesztek nem kapcsolják ki a dokumentumvédelmi triggereket takarításhoz.

Parancsok (csak a saját development DB URL-jével):

```sh
npx prisma generate
npx prisma migrate deploy
npm test -- tests/unit
npm test -- tests/integration/deliveryNote.test.ts tests/integration/jobStatus.test.ts tests/integration/orderCancel.test.ts tests/integration/orderOverdueNotifications.test.ts
npx playwright test -c playwright.delivery-note.config.ts
npm run typecheck
npm run build
```

A külön Playwright-konfiguráció helyi dev szervert indít staging email-tiltással,
375 és 768 px-es nézetben CDP érintési eseményekkel tesztel; normál telepített Chromiumot,
vagy opcionálisan `NOTE_CHROMIUM_PATH` tesztböngészőt használ.

## Aktiválás és nyitott kérdések

A feature alapértelmezésben kikapcsolt: `DELIVERY_NOTES_ENABLED=false`.
Development/stagingben a migráció után állítható `true`-ra. Production aktiválás/merge/deploy
nem része ennek az átadásnak. A távoli Render staging sem került átállításra ebben a munkában.

Nyitott üzleti kérdések:

1. Kellenek-e szállítólevelek a vevő nélküli belső TransportTask fuvarokhoz is?
   Ha igen, honnan jön a vevő és az email-cím? V1 ezeket nem alakítja át ügyfélrendeléssé.
3. Szükséges-e a jelenleg üres egységár/érték vagy besorolási mező? Ehhez nincs
   megbízható, megfelelő jelentésű adat; automatikus átszámítás nem készült.
4. A tényleges hulladékmennyiség jelenleg sofőr által ellenőrzött adat; mérlegjegy-integráció nincs.

Technikai korlát: aláírás előtt nincs offline szinkronizálás. Sikertelen hálózati kérésnél
az aláírás az adott nyitott oldalon megmarad; a készülék bezárása előtt figyelmeztetés jelenik meg.
Egy sikeresen lezárt dokumentumot oldalfrissítés vagy későbbi adatváltozás nem generál újra.

## Lezáró ellenőrzés eredménye

- 1016/1016 unit teszt sikeres.
- 51/51 célzott integrációs és kapcsolódó regressziós teszt sikeres, PostgreSQL 16-on.
- 375 és 768 px-es böngészős teljes érintési workflow: 2/2 sikeres.
- TypeScript typecheck és production build sikeres.
- A4/PDF vizuális ellenőrzés sikeres; magyar ékezetek és aránytartó aláírás rendben.
- 32 migráció alkalmazva az izolált development DB-n, nincs függő migráció.
- A lezáró kódellenőrzés és `git diff --check` nem talált további hibát;
  a sikeres ellenőrzéseket a munka folytatásakor nem ismételtük meg.
- Production és a távoli Render staging változatlan; nincs merge vagy deploy.

Státusz: FEATURE READY a rendeléshez kapcsolt Job V1 development workflow-ra.
A belső, rendelés nélküli fuvarok és a fent felsorolt üzleti kérdések továbbra is nyitottak.

## Indítás és teljesítés sorrendje

Bekapcsolt digitális szállítólevélnél a rendeléshez kapcsolt feladat sorrendje:
Indítás → adatok ellenőrzése → ügyfél aláírása → szállítólevél lezárása → Teljesítve.
TERVEZETT állapotban a sofőr gombja letiltott, a közvetlen API-lekérés és új
dokumentum lezárása is 409 választ ad. Új dokumentum csak FOLYAMATBAN
állapotban készülhet. A completeJob tranzakción belül ellenőrzi a FOLYAMATBAN
állapotot és a lezárt dokumentum meglétét minden fizikai mellékhatás előtt.
Az email sikertelensége vagy staging tiltása nem akadályozza a teljesítést.
A már archivált dokumentum később is olvasható; az ismételt lezárás idempotens.
A feature kikapcsolt állapota és a vevő nélküli belső fuvarok nem változnak.

## Az üzemeltetővel véglegesített konténermozgatási workflow

Kihelyezés, elszállítás és csere esetén egyaránt kötelező a digitális szállítólevél.
A közös sorrend: Kiosztva → Indítás → fizikai konténermozgatás → szállítólevél
→ ügyfél aláírása → lezárás → Teljesítve. A fizikai mozgás nem új alkalmazásállapot:
a sofőr annak elvégzése után kéri az aláírást. A csere egy feladat és egy dokumentum,
amely a tele konténer elszállítását és az üres konténer kihelyezését igazolja.
A PDF fejlécében a művelet típusa szerepel, csere esetén az alsó műveletleírás
mindkét irányt nevesíti. A dokumentumkötelezettség közös szerveroldali kapuja
a feladattípus szerinti készlet- és rendelésmódosítások előtt fut.
A bevezetési feature flag megmarad; bekapcsolt funkciónál nincs típus szerinti kivétel.
