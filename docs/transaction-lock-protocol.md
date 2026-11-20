# Konti tranzakciós lockprotokoll — L0 szerződés

Állapot: **L0 szerződés + célzott operatív admission-adoption**

- protokoll: `konti-lock-protocol/v1`
- advisory namespace registry: `konti-advisory-namespace-registry/v1`
- row-rank registry: `konti-row-lock-rank-registry/v1`

Az L0 egységes lockidentitásokat, fizikai sorrendet és fail-closed guardot
ad a writer-adoption PR-ekhez. A célzott operatív íróutak már az alább
leírt közös EXCLUSIVE admissiont használják. Az L0 önmagában, az íróutak
adaptációja nélkül nem javít race-t vagy deadlockot.

## V1 előfeltétel: célzott operatív írásvédelem

A `runOperationalWrite(prisma, callback, options?)` a meglévő
`runWithTransactionLockContext` hitelesített tranzakcióját használja.
A közös `(74000001, 1)` EXCLUSIVE admission az első üzleti olvasás,
napi/dispatch/numbering lock, feltételes írás és audit ELŐTT megszerzésre kerül.
Commit vagy rollback felszabadítja; nincs session-szintű vagy process-local
írásvédelem. A callback ugyanazt a transaction clientet kapja. Nested command
nem támogatott: a tranzakción belüli helpernek a kapott `tx`-et kell használnia.

| Adaptált tranzakciógazda | Védett operatív változás |
| --- | --- |
| `services/operationalRecommendations.ts` | tartós ajánlás/döntés/preview/végrehajtás; #89 prepare/apply ugyanazon tx-en |
| `services/jobAssignment.ts` | Job nap, sofőr, jármű, sequence |
| `services/transportTask.ts` | kiosztott/kiosztatlan létrehozás, kiosztás, státusz |
| `services/transportTaskContent.ts` | döntéshez releváns tartalom |
| `services/dispatchOrdering.ts` | közös Job/TransportTask napi sorrend |
| `services/dailyOperations.ts` | napi terv cseréje és törlése |
| `services/rescheduleDecision.ts` | apply; a dismiss nem operatív írás |
| `services/jobStatus.ts` | START/COMPLETE/FAIL, Order/Container mellékhatások |
| `services/orderCancel.ts`, `services/customerOrderCancellation.ts` | lemondás, Job és Container változások |
| `api/admin/orders/[id]/{assign,jobs,reassign-kihelyezes}/route.ts` | kézi Job-létrehozás és konténerfoglalás |
| `api/admin/dispatch-runs/route.ts` | fuvarhoz kötés és rakodási/sorrendi mezők |
| `services/trucks.ts`, `services/containers.ts` | erőforrás create/update/delete |
| `services/resources.ts` | driver és legacy truck create/update; master user create/update/roles/delete |
| `lib/authorization.ts` | szerep adása/visszavonása |
| `services/gps.ts` | GPS párosítókód és eszköz-párosítás, műszak indítása/zárása, pontfeltöltés, eszköz visszavonása; a csak olvasó `gpsDeviceState` és `driverGpsDevices` is (lásd alább) |

Az útvonalak `src/` alatt értendők. A `jobOrdering.ts` delegál a védett
dispatch-gazdának; a napi terv/sequence transaction-client helper nem nyit
új tranzakciót. A customer cancellation írási belépése valódi PrismaClientet
igényel, a read-only quote felülete változatlan.

Ez admission-only adaptáció: a meglévő belső lockok, feltételes claim-ek,
auditok, no-opok és üzleti szabályok megmaradnak. Nem vezeti be a teljes
row-rank protokollt vagy új kézi kapacitáskorlátot. A scope-ba tartozó
írók sorosodnak; két kézi kiosztás egymás után mindkettő sikerülhet, a második
friss állapotból dolgozik, és audit before-image-e az első commitját tükrözi.
Automatikus retry nincs; a Prisma tranzakciós limitek továbbra is érvényesek.

A Job státusz-route csak a hitelesített DRIVER módból ad át
`expectedDriverId`-t. START/COMPLETE/FAIL a lock után újra ellenőrzi ezt,
és a státusz-claim WHERE feltétele is ugyanarra a sofőrre szűr. Eltérés:
403, üzleti/audit/outbox változás nélkül. A TransportTask meglévő
tranzakciós ownership-ellenőrzése és hibaszemantikája megmarad.

A PostgreSQL integrációs tesztek a pontos admission kulcsot, database-t
és a `pg_blocking_pids` tényleges kapcsolatát figyelik. A dokumentum-gate
régi unit tesztje továbbra is fake DB-t használ; az nem lockbizonyíték.

### GPS útvonalak

A `services/gps.ts` mind a nyolc exportált, adatbázist használó művelete
(`createGpsPairing`, `pairGpsDevice`, `startGpsShift`, `gpsDeviceState`,
`ingestGpsPoints`, `stopGpsShift`, `driverGpsDevices`, `revokeGpsDevice`)
`runOperationalWrite`-ot használ, egyedi tranzakciós opció nélkül. Ezek tehát
ugyanazt a `(74000001, 1)` EXCLUSIVE admissiont várják be, mint a többi
adaptált író, és sorosodnak velük. Ez a meglévő viselkedés rögzítése, nem új
szabály.

- **Csak olvasó utak is zárat vesznek.** A `gpsDeviceState` (az eszköz
  állapotlekérése) és a `driverGpsDevices` (böngészős eszközlista) nem ír
  üzleti adatot, mégis az írókkal közös admissionre vár. Ha ezek később
  kikerülnek az admission alól, az külön, kifejezett döntés és külön csomag.
- **Az `ingestGpsPoints` minden hívása a zárolt tranzakción belül törli** a
  `GPS_RETENTION_MS` (7 nap) előtti pontokat (`deleteMany`, `measuredAt`
  szerint). A szűrés nem az érintett műszakra vagy eszközre korlátozódik, hanem
  az összes lejárt pontra, ezért egy hívás költsége a törlendő sorok számával
  nő, és az admission a commitig él.
- **A kikapcsolt funkció a lock előtt lép ki.** Ha az `ENABLE_GPS_TRACKING`
  nem `"true"`, a művelet `GPS_DISABLED` (503) hibával tér vissza, mielőtt
  tranzakciót nyitna vagy admissionre várna.
- **A `latestGpsPositions` kívül marad.** Egyszerű `prisma` olvasás, nem
  `runOperationalWrite`.
- Az általános szabályok változatlanok: automatikus retry nincs, és a Prisma
  alapértelmezett interaktív tranzakciós limitjei érvényesek (a lockvárakozás
  is beleszámít a tranzakció idejébe), mert a GPS-utak nem adnak meg egyedi
  `timeout`/`maxWait` opciót.

A `tests/integration/gpsAdmission.test.ts` ugyanazzal a valódi PostgreSQL
backendes módszerrel (`blockedByAdmission`: pontos kulcs, `pg_locks`,
`pg_blocking_pids`) bizonyítja mind a nyolc útra, hogy az admissionre vár és
várakozás közben nem változtat adatot; a kikapcsolt funkciót a lock előtti
kilépésre; a pontfeltöltésnél pedig azt, hogy a retenciós törlés az admission
megszerzése után fut.

Ez NEM repository-wide coverage gate. Egyéb auth/session/password,
chat/retention, payment/invoice, Order-only, dokumentum- és maintenance/script
írók e körön kívül maradnak. Régi alkalmazáspéldányok nem vesznek részt az
új admissionben; a bounded garancia csak az összes érintett alkalmazásprocessz
frissítése és a régi tranzakciók kifutása után érvényes. Globális production
lockgarancia, SHARED rollout és row-order refaktor ezzel nem engedélyezett.

## Tranzakció tulajdonjoga és lifecycle

A támogatott publikus belépési pont a
`runWithTransactionLockContext(prisma, callback, options?)`. A helper nyitja
meg a Prisma interactive tranzakciót. A callback ugyanazt a valódi
`TransactionClient` példányt és az abból belsőleg létrehozott egyetlen
lock-contextet kapja.

A transaction ownernek runtime-ban a repository által használt
`@prisma/client` `PrismaClient` osztály valódi példányának kell lennie. Plain
object vagy strukturális `$transaction` facade még a `$transaction` meghívása
előtt elutasításra kerül. Ez a nominális kapu önmagában nem elegendő: a
callback-clientet külön PostgreSQL transaction-local attestation hitelesíti,
mielőtt context, application admission vagy business callback létrejöhetne.

Az attestation egy szerveroldali véletlen nonce-ot `set_config(..., true)`
hívással kizárólag az aktuális tranzakcióra állít be. Két külön, paraméterezett
query rögzíti, majd összeveti a nonce-ot, a `pg_backend_pid()` értékét és a
`txid_current()` szöveges értékét. Mindháromnak bitazonosan egyeznie kell. A
transaction-local beállítás commitnál és rollbacknél automatikusan eltűnik;
session-level capability vagy pool-maradvány nem keletkezik. Globális
PrismaClient autocommit queryjei és bound `$queryRaw` facade nem tudják a
nonce-ot két statementen át ugyanabban a tranzakcióban megőrizni. Az
attestation PostgreSQL-specifikus, a jelenlegi támogatott környezet PostgreSQL
16.

Közvetlen context-factory nincs exportálva. Egy `$queryRaw` metódust tartalmazó
facade, globális PrismaClient, context-copy, spread vagy plain object nem tud
érvényes capabilityt létrehozni. A runtime-ellenőrzés process-local
`WeakMap`-objektumazonosságot köt a helper által megnyitott tranzakcióhoz. Ez
nem adatbázisban megosztott identitás; a garanciát a transaction-owning wrapper
és a belső factory együtt adja.

A wrapper nem indít nested tranzakciót, és a callback lockműveletei kizárólag a
kapott transaction clientet használják. Az opcionális felület csak a Prisma
`maxWait`, `timeout` és `isolationLevel` interactive transaction opcióit
engedi.

A nested wrapperhasználatot process-local `AsyncLocalStorage` execution-scope
guard tiltja még a második `$transaction` meghívása előtt. Ugyanazon logical
async chainben azonos és másik PrismaClienttel sem nyitható új lock-protokoll
tranzakció. A kísérlet az outer contextet `FAILED` állapotba teszi, ezért az
outer callback által elkapott hiba után sem lehetséges commit. A callbackből
kiszivárgó késői async munka örökli a lezárt scope-ot és nem indíthat új
logical commandot. Független top-level requestek saját async scope-ot kapnak,
ez nem distributed vagy processzek közötti lock.

A context állapotai:

| Állapot | Jelentés |
| --- | --- |
| `ACTIVE` | új protokollművelet indítható |
| `BUSY` | egy protokollművelet az első `await` előtt lefoglalta a contextet |
| `FAILED` | sorrendi, ownership-, concurrency- vagy protokollhiba történt; commit tilos |
| `CLOSED` | a callback befejeződött, hibázott, commitált vagy rollbackelt |

Ugyanazon context átfedő használata nem kerül belső queue-ba:
`CONCURRENT_CONTEXT_USE` hibával fail closed lesz. A már futó művelet sem
jelentheti sikeresnek a contextet. `FAILED` állapotból nincs helyreállás, és a
wrapper akkor is rollbacket kényszerít, ha a callback elkapta az eredeti hibát.
`CLOSED` contexten azonos admission vagy row authorization sem lehet no-op.
Az aktív execution scope-ban átadott ismeretlen, másolt vagy másik scope-hoz
tartozó context az aktuális scope eredeti contextjét is azonnal `FAILED`
állapotba teszi. A másik, független top-level scope állapotát ez nem módosítja.

Automatikus tranzakció-retry nincs az L0-ban. Providerstart, network wait,
provider-output és `UNKNOWN` állapot nem lehet retryolható tranzakció része.
Később csak teljesen DB-lokális, idempotens command próbálható újra explicit,
szűk SQLSTATE allowlist alapján.

## Advisory sorrend

Minden writer az összes szükséges advisory scope-ot a row-lock fázis előtt
határozza meg. A normatív sorrend:

| Rank | Scope |
| ---: | --- |
| 10 | application-scope admission |
| 20 | retention/maintenance singleton |
| 30 | minden érintett régi és új üzleti nap |
| 40 | driver/date dispatch kulcsok |
| 50 | conversation creation/scope |
| 60 | numbering namespace |
| 70 | idempotency |
| utána | row-lock phase |

Az application-scope identitás a két `int4` kulcsú PostgreSQL advisory
térben `(74000001, 1)`. Az admission mód `SHARED` vagy `EXCLUSIVE`;
módváltás, különösen `SHARED → EXCLUSIVE`, tilos.

Az adoption átmeneti szakaszában minden adaptált production writer
`EXCLUSIVE` admissiont használ. `SHARED` writer mód csak teljes
writer-coverage és concurrency acceptance után engedélyezhető.

Rankon belül a tényleges fizikai `(namespace, key)` pár signed numerikus
sorrendje normatív. A planner fizikai kulcs szerint deduplikál, majd rank és
fizikai kulcs szerint rendez. Azonos rankon bekövetkező hash-collision egyetlen
lock. Inputpermutáció nem változtatja a tervet.

Minden megszerzett fizikai tuple a context teljes lifecycle-jára a saját
normatív rankjához és módjához kötött. Későbbi külön acquisition hívás nem
címkézheti át. Azonos tuple, rank és mód ismételt kérése idempotens no-op.
Eltérő rank vagy mód `FAILED` állapotot okoz. A rögzített namespace-eket a
caller nem rendelheti másik rankhoz:

| Namespace | Rank |
| ---: | ---: |
| `74000001` application admission | 10 |
| `74000002` retention/maintenance | 20 |
| `48480001` daily operations | 30 |
| `48480002` driver/date dispatch | 40 |
| `74000003` conversation scope | 50 |
| `56000001` task numbering | 60 |
| `74000004` idempotency | 70 |

### Meglévő production identitások

| Scope | Namespace | Key |
| --- | ---: | --- |
| daily operations | `48480001` | UTC `YYYYMMDD` egész |
| driver/date dispatch | `48480002` | 31 bites FNV-1a: `driverId:YYYY-MM-DD` |
| transport task number | `56000001` | négyjegyű év |

Az L0 a production UTC formattert használja a dispatch kulcshoz, beleértve az
1000 alatti év négyjegyű formáját. Mindhárom identitás a
`pg_advisory_xact_lock(int, int)` fizikai kulcsterében marad.

## Fizikai row identity és sorrend

Az API nem fogad SQL-táblanevet vagy caller által megadott rankot. A caller a
zárt `ROW_LOCK_KIND` registryből választ. A fizikai identity:

`(rank, subrank/kind, id)`

A planner először rank, majd subrank, végül ID szerint rendez. Az ID-ket csak
azonos kindon belül deduplikálja. Azonos `id` például `Address` és `Truck`
sorhoz két külön identity marad. A lock acquisition sorrend nem azonos az
üzleti sequence-, dispatchpozíció- vagy megjelenítési sorrenddel.

| Rank | Subrank és zárt kindok |
| ---: | --- |
| 10 | 10 `USER` |
| 20 | 10 `USER_ROLE` |
| 30 | 10 `USER_SESSION` |
| 40 | 10 `CUSTOMER`; 20 `PARTNER`; 30 `ADDRESS`; 40 `CONTAINER_TYPE`; 50 `TRUCK`; 60 `TRAILER` |
| 50 | 10 `CONVERSATION` |
| 60 | 10 `CONVERSATION_PARTICIPANT` |
| 70 | 10 `ORDER` |
| 80 | 10 `CONTAINER` |
| 90 | 10 `DISPATCH_RUN` |
| 100 | 10 `JOB` |
| 110 | 10 `TRANSPORT_TASK` |
| 120 | 10 `DAILY_OPERATIONS_PLAN` |
| 130 | 10 `DAILY_OPERATIONS_DRIVER` |
| 131 | 10 `DAILY_OPERATIONS_TRUCK` |
| 140 | 10 `MESSAGE` |
| 150 | 10 `MESSAGE_ATTACHMENT` |
| 151 | 10 `MESSAGE_DOMAIN_LINK` |
| 160 | 10 `AI_PROVIDER_ATTEMPT` |
| 161 | 10 `AI_INTERPRETATION_RUN` |
| 170 | 10 `AI_INTERPRETATION_RUN_SOURCE` |
| 180 | 10 `OPERATIONAL_OBSERVATION` |
| 190 | 10 `OPERATIONAL_OBSERVATION_EVIDENCE` |
| 191 | 10 `OPERATIONAL_OBSERVATION_REFERENCE` |
| 200 | 10 `EMAIL_VERIFICATION`; 20 `PASSWORD_RESET_TOKEN`; 30 `LEGAL_DOCUMENT_VERSION`; 40 `PAYMENT`; 50 `INVOICE`; 60 `PRICING_RULE`; 70 `DISTANCE_ZONE`; 80 `PRICING_CONFIG`; 90 `ORDER_OVERDUE_NOTIFICATION`; 100 `RESCHEDULE_SUGGESTION_DECISION`; 110 `DELIVERY_NOTE`; 120 `DELIVERY_NOTE_EMAIL` |
| 210 | 10 `AUDIT_INSERT` |

Az üres row batch teljes no-op: valid contextet és kindot igényel, de nem
változtat rankot vagy ID-cursort. Nem üres batchben az ID-k nem lehetnek üresek
vagy duplikáltak, és lexikusan rendezve kell érkezniük. Egy későbbi kind vagy
ID nem léphet vissza.

A command az audit actort (`USER`) és minden létrehozandó vagy módosítandó sor
FK-parentjét az első child előtt foglalja. A hiányzó parentet ugyanebben a
rendezett kindban kell ellenőrizni; existence-oracle hibaszöveg csak
authority-ellenőrzés után adható.

Az L0 row helper sorrendi authorizationt ad, de nem hajt végre
`SELECT ... FOR UPDATE` SQL-t. A későbbi table-specific adapternek pontosan a
jóváhagyott kind és rendezett ID-lista sorait kell lockolnia, és SQL-hibánál a
tranzakciót abortálnia.

## Adoption és coverage gate

Az L0 után kis prerequisite PR-ek adaptálják a writereket. A coverage gate csak
akkor zárható, ha repository-keresés és concurrency tesztek igazolják, hogy
minden production, maintenance, script és raw-SQL writer ugyanazt az admission-
és sorrendi protokollt használja. A rollout része a korábbi application
processzek és folyamatban lévő tranzakciók kifutása. Row-order refaktor csak e
gate után indulhat.

Az L0 eredeti, teljes coverage gate-jéből e célzott kör után is fennmarad:

- session, requester `User`, role és authority adminisztráció;
- conversation és participant create/update/delete;
- Message create/edit/soft-delete/hard-delete;
- MessageAttachment és MessageDomainLink írás;
- conversation retention és maintenance;
- az operatív scope-on kívüli Order-írások;
- audit writer a fenti adaptált tranzakciókon kívül;
- a befagyasztott PR #74 interpretation run/source/observation/evidence/
  reference persistence és retention útjai.

Amíg bármely felsorolt writer vagy maintenance út nincs adaptálva, globális
production lockgarancia nem állítható.

## Kötelező végső validáció egy későbbi commandban

Az advisory és row lockok után, közvetlenül írás előtt újra kell ellenőrizni a
requester session/authVersiont, authority scope-ot, optimistic verziókat és az
érintett kapcsolat/scope fingerprintet. Eltérésnél fail closed, üzleti és audit
delta nélkül. Providerhívás soha nem ismételhető meg tranzakció-retry
mellékhatásaként.
