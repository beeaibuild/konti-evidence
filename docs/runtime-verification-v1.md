# Runtime verification v1 — 10–11. csomag (stabilizáció + polish)

Cél: az 1–9. csomagos alkalmazás futtatható állapotának ellenőrzése.
A dokumentum két futtatás eredményét rögzíti: a fejlesztői sandboxét
(környezeti korlátokkal) és a standard fejlesztői gépét (teljes
runtime smoke). A 11. csomag a standard gépi smoke során talált két
UI-polish javítást tartalmazza (lásd 5. szakasz).

## 0. Standard gépi futtatás eredménye (mérvadó)

Környezet: Windows + WSL2 + Docker Desktop, Postgres Dockerben.

| Parancs | Eredmény |
|---|---|
| `npm install` | ✅ |
| `npx prisma generate` | ✅ (workaround nélkül) |
| `npm run db:reset` | ✅ |
| `npm run typecheck` | ✅ |
| `npm run build` | ✅ |
| `npm run dev` | ✅ |

Ténylegesen végigkattintott flow-k (mind zöld): publikus
rendelésleadás; admin rendeléslista + adatlap; KIHELYEZES kiosztás;
sofőr START/COMPLETE; URITES kiosztás + teljesítés; ELSZALLITAS
kiosztás + teljesítés; mock fizetés indítás; admin MARK_PAID; admin
REFUND; dashboard-ellenőrzés. A DB-darabszámok Prisma Studióban
konzisztensek voltak a felülettel.

A smoke során előjött két UI-polish megjegyzés (nem üzletilogika-hiba):
1. A `/sofor` feladatkártyán nem látszott a rendelésszám.
2. A tervezett dátum több helyen `2:00` időponttal jelent meg
   (timezone-következmény: az éjfélre tárolt dátum helyi időben
   02:00), miközben a feladatok üzletileg napra tervezettek.

Mindkettő a 11. csomagban javítva — részletek az 5. szakaszban.

## 1. Sandbox környezet

Sandbox: Ubuntu 24, Node 22, npm 10. Korlátozott hálózat: az npm
registry elérhető, a `binaries.prisma.sh` (Prisma engine binárisok)
**nem** (403 Forbidden). A sandboxban nincs előre futó PostgreSQL és
nincs Docker.

## 2. Parancsok eredménye (standard, módosítatlan projekt)

| Parancs | Eredmény | Megjegyzés |
|---|---|---|
| `npm install` | ✅ lefutott | 121 csomag, tiszta, az eredeti lockfile szerint |
| `npx prisma validate` | ⚠️ env-workarounddal ✅ | módosítás nélkül elhasal az engine-letöltésen (lásd 3.); lokális env-változókkal lefut: **"The schema at prisma/schema.prisma is valid"** |
| `npx prisma generate` | ⚠️ env-workarounddal ✅ | ugyanaz; a kliens típusgenerálás teljes és valódi (a generálás WASM parserrel megy, a natív engine-t csak másolná) |
| `npm run db:reset` | ❌ környezeti | a `prisma db push` a natív schema-engine binárist futtatná, ami nem tölthető le — **"Error: Schema engine error"** |
| `npm run typecheck` | ✅ hibátlan | `tsc --noEmit`, 0 hiba |
| `npm run build` | ✅ sikeres | "Compiled successfully" + típusellenőrzés + 16/16 route; egy Prisma runtime-hiba a statikus prerender közben várt (nincs engine/DB), a buildet nem töri el |
| `npm run dev` + flow-k | ❌ környezeti | DB és futtatható query engine nélkül a standard stack nem tud kérést kiszolgálni |

## 3. A környezeti blokkolók pontosan

**Mi a hiba:** minden engine-t igénylő Prisma CLI művelet ezzel áll
meg:

```
Error: Failed to fetch sha256 checksum at
https://binaries.prisma.sh/all_commits/6051973…/debian-openssl-3.0.x/
libquery_engine.so.node.sha256 - 403 Forbidden
```

**Miért környezeti:** a sandbox kimenő hálózata a `binaries.prisma.sh`
domaint nem engedi, ezért a Prisma 5 natív query engine
(`libquery_engine.so.node`) és schema engine binárisa nem tölthető
le. Ez nem a projekt hibája — normál fejlesztői gépen a letöltés
átmegy.

**Lokális env-workaround a validate/generate/typecheck/buildhez** (a
projektet nem érinti, csak shell-env): a
`PRISMA_QUERY_ENGINE_LIBRARY` és `PRISMA_SCHEMA_ENGINE_BINARY`
változók egy létező (dummy) fájlra állítva a letöltés kimarad, így a
`validate` és a `generate` lefut — mindkettő a CLI-be épített WASM
schema-parsert használja, tehát az eredményük valódi. A **futtatás**
(db push, query-k) ettől nem működik: ahhoz működő natív engine + DB
kellene.

**Amit emiatt standard stacken NEM lehetett végigkattintani a
sandboxban:** a 2. szakasz (teljes v1 smoke flow, A–G) és a 3. szakasz
(jogosultsági ellenőrzések) minden DB-t érintő pontja.

## 4. Megjegyzés: elkülönített harness-futtatás (nem mérvadó)

A teljes átláthatóság kedvéért: a verifikáció közben egy **ideiglenes,
a csomagból teljesen visszaállított** tesztkörnyezetben (sandboxba
telepített PostgreSQL 16 + a Prisma bundled WASM engine-je
driver-adapterrel, kézzel felvitt, a sémát tükröző DDL) az A–G flow-k
és a jogosultsági ellenőrzések HTTP-n (curl) ténylegesen lefutottak.
Ez a harness a leszállított csomag **egyetlen fájlját sem**
módosította (schema.prisma, src/lib/prisma.ts, prisma/seed.ts,
package.json, package-lock.json diff-fel igazoltan az eredeti), és
dependency sem került be.

Az így szerzett megfigyelés informatív, de **nem helyettesíti** a
standard stacken (natív engine + Postgres) futó ellenőrzést — a
lenti 7. szakasz teendői ettől függetlenül kötelezők. A megfigyelt
eredmény: az A–G flow-k és a jogosultsági esetek (401/403/redirect)
mind a dokumentált módon viselkedtek; **alkalmazáskód-hibát nem
találtunk**. Néhány konkrétum:

- A (publikus): árbecslés 1131/5 m³ → 45 150 Ft alap + hétvégi +15%
  (a futtatás szombatra esett) = 51 923 Ft; leadás → UJ_RENDELES;
  tegnapi dátum → 400.
- B–C: kiosztás → KIHELYEZESRE_VAR/FOGLALT/TERVEZETT; START →
  FOLYAMATBAN; COMPLETE → KIHELYEZVE/HASZNALATBAN; másik sofőr → 403;
  kijelentkezve → 401; admin bármely jobot kezelhet.
- D: URITES → vissza KIHELYEZVE; CSERE → konténercsere, régi SZABAD;
  ELSZALLITAS → TELJESITVE, konténer SZABAD, depóban.
- E: FAIL → SIKERTELEN completedAt nélkül, `[SIKERTELEN] ok` a
  note-ban, konténer FOGLALT marad; újrakiosztás működik; CSERE-FAIL
  felszabadítja az új konténert; lemondás lezárja az aktív jobot
  (0 aktív job marad), újralemondás 409.
- F: start (idempotens, `existing: true`) → SUCCESS/FAIL →
  MARK_PAID / MARK_FAILED / REFUND; az Order.paymentStatus és az
  utolsó Payment.status minden ágon egyező; nincs duplikált
  FIZETESRE_VAR.
- G: a dashboard minden csempéje és listája a DB `groupBy`
  eredményével egyezett (rendelés-, fizetés-, konténer-darabszámok,
  lejárt jelölés, sikertelen ok, linkek).
- Hibautak: TELJESITVE-n újra-assign/complete → 409; ismeretlen
  action → 400; rossz rendelésszám → 400/404.

## 5. Talált hibák és javítások

**Üzletilogika- vagy státuszgép-hiba: nem került elő** sem a
sandboxban, sem a standard gépi smoke-ban.

UI-polish javítások (11. csomag):
1. **Rendelésszám a sofőr kártyán** — a `/sofor` feladatkártyáról
   hiányzott a rendelés száma. Javítás: `src/components/JobCard.tsx`
   (új `orderNumber` mező a `DriverJob` típusban + "Rendelés:" sor a
   kártya adatlistája elején, `—` fallbackkel) és
   `src/app/sofor/page.tsx` (a mező feltöltése a már lekért
   `order` relációból — a Prisma query nem változott, az `include`
   eddig is visszaadta az `orderNumber` skalármezőt). DRIVER és ADMIN
   `/sofor` nézetben egyaránt látszik, a mobil elrendezést nem töri
   (egy sor a meglévő `dl` listában).
2. **plannedDate date-only megjelenítés** — az admin rendelés-adatlap
   feladatlistája `huDateTime`-mal mutatta a tervezett dátumot, ami
   az éjfélre tárolt érték miatt `2:00`-ként jelent meg. Javítás:
   `src/app/admin/rendelesek/[id]/page.tsx` — `huDateTime(j.plannedDate)`
   → `huDate(j.plannedDate)`. A többi érintett hely már eddig is
   date-only volt: `/sofor` kártya és a dashboard mindkét joblistája
   (mai/lejárt, sikertelen) `huDate`-et használ, ahogy a "Kívánt
   dátum" is. Szándékosan `huDateTime` maradt két helyen, mert ott
   valódi időbélyeg a tartalom: a fizetési tranzakciók `createdAt`-ja
   az adatlapon és a rendelés `updatedAt`-ja a dashboard fizetési
   listáján.

Egyik javítás sem érint státuszgépet, adatmodellt, Prisma query-t
vagy üzleti szabályt — tisztán megjelenítés.

Dokumentációpontosítás (10. csomag):
- `docs/smoke-test-v1.md` 4. pont: a 45 150 Ft-os árbecslés csak
  hétköznapi kívánt dátumra igaz; hétvégén a +15% felárral 51 923 Ft
  a helyes érték. A pont ezt most jelzi, hogy hétvégi teszteléskor ne
  tűnjön hibának. (Nem érint státuszgépet, adatmodellt, üzleti
  szabályt — a PricingService viselkedése dokumentáltan ilyen.)

## 6. Maradó ismert korlátok

Változatlanok a README "Ismert v1 korlátok" szakaszában felsoroltak
(mock fizetés, ügyfél-login hiánya, lemondás–függő fizetés lyuk,
dashboard fix 10-es listái stb.). Új korlát nem került be, meglévő nem
szűnt meg — ez a csomag nem nyúlt az üzleti logikához.

## 7. Merge előtti teendők a saját gépen

A standard gépi futtatás (0. szakasz) a 10. csomagra megtörtént és
zöld. A 11. csomag két UI-javítása után elegendő egy rövid
újraellenőrzés:

1. `npm run typecheck` és `npm run build`.
2. `npm run dev` mellett: a `/sofor` kártyán látszik a
   "Rendelés: R-…" sor (admin nézetben is), és az admin
   rendelés-adatlap feladatlistájában a tervezett dátum időpont
   nélkül jelenik meg (nincs `2:00`).
3. Ha bármi elakad, a `docs/smoke-test-v1.md` 20 pontja és a
   csomag-tesztlisták (`manual-test-package-3…9.md`) szűkítik a
   hibát.
