# P39-r1 — Production törzsadat-bootstrap + homogén/vegyes árazás

Állapot: KÓD KÉSZ — a lokális bizonyítási kapu lefuttatásáig NEM GO.
Előzmény: P38-r1 (Render deploy bizonyított, app Live, DB üres).

## 1. Mi változott és miért

A dev seed (prisma/seed.ts) helyesen tiltott production-ben, ezért az
éles DB-nek külön, seed-mentes törzsadat-útvonal kell. Emellett az
üzleti árazás homogén/vegyes megkülönböztetést kíván, amit a P38-as
modell nem tudott kifejezni.

Modell-döntés: a hulladék jellege (homogén/vegyes) a RENDELÉS
tulajdonsága (`Order.wasteKind`), nem a fizikai konténeré — ugyanaz a
vas viszi mindkettőt, ezért NEM 8 konténertípus készült, hanem egy
`WasteKind` dimenzió és egy `wasteKind=VEGYES` címkés fix felárszabály.
Az "5 km benne van" a `PricingRule.includedKm` mezővel általánosít:
PER_KM szabálynál az első `includedKm` km díjmentes.

A 7 nap alapbérlet v1-ben kommunikált adat (a típusleírás része);
bérletnap-alapú árlogika nincs a megállapodott árképzésben, ezért
szándékosan nem került az engine-be.

## 2. Érintett fájlok

- prisma/schema.prisma — WasteKind enum; Order.wasteKind (NOT NULL,
  default HOMOGEN); PricingRule.wasteKind (null=mindre), includedKm.
- prisma/migrations/20260725120000_p39_waste_kind_pricing/ — additív
  migráció, adatvesztés-mentes (részletek a migration.sql fejlécében).
- src/services/pricing.ts — wasteKind-szűrt szabályok; billedKm =
  max(0, distanceKm − includedKm); breakdown bővítve (wasteKind,
  includedKm, billedKm).
- src/lib/validation.ts — wasteKind a publicOrderSchema-ban és a
  priceEstimateSchema-ban, default("HOMOGEN") a visszafelé
  kompatibilitásért (régi payload változatlan árat kap).
- src/app/api/orders/route.ts, src/app/api/price-estimate/route.ts —
  wasteKind átvezetve; audit bővítve (wasteKind, includedKm, billedKm).
- src/components/OrderForm.tsx — homogén/vegyes választó; a becslés a
  díjmentes km-t is mutatja.
- src/app/page.tsx — üres állapot production-ben semleges szöveg, dev
  seedre utaló technikai segítség csak nem-éles környezetben.
- src/app/admin/rendelesek/[id]/page.tsx — Hulladék sor az adatlapon.
- scripts/bootstrap-production.mjs — ÚJ: idempotens production
  törzsadat-bootstrap (guardok a fájl fejlécében dokumentálva).
- package.json — `db:bootstrap:production` script.
- Dockerfile — a bootstrap script a runtime image része.
- tests/ — unit: bootstrapProduction; integráció: bootstrapProduction,
  pricing P39-esetek; fixtures bővítve (wasteKind, includedKm).

## 3. Kanonikus üzleti törzsadat

| Méret | Homogén | Vegyes | Készlet |
|-------|---------|--------|---------|
| 4 m³  | 60 000  | 65 000 | 20 db (K4-001…K4-020) |
| 6 m³  | 70 000  | 75 000 | 15 db (K6-…) |
| 8 m³  | 80 000  | 85 000 | 30 db (K8-…) |
| 10 m³ | 90 000  | 95 000 | 5 db (K10-…) |

Távolság: 5 km-ig díjmentes, felette 2 000 Ft/km a többletre
(DIST_PER_KM, includedKm=5). Vegyes: +5 000 Ft fix (VEGYES_FELAR,
wasteKind=VEGYES). Alapbérlet: 7 nap (leírásban). A konténerkód
K4-001 formátumú — szándékosan eltér a dev seed K-4-001 kódjaitól,
hogy dev/staging DB-n futó bizonyítás ne ütközzön.

## 4. Lokális bizonyítási kapu (PowerShell, a P-metodika szerint)

```powershell
cd C:\dev\kontener-p31\kontener
npm ci
npx prisma migrate dev    # felveszi a p39_waste_kind_pricing migrációt
npm run typecheck
npm run test              # Vitest: unit + integráció (köztük az új P39 fájlok)
npm run build
npm run test:e2e          # Playwright (futó dev stackkel, a megszokott módon)
docker build -t kontener-app:p39 .
```

Bootstrap-CLI bizonyítás FRISS lokális DB-n (nem a seedelt dev DB-n —
a bootstrap dev-seed usert észlelve szándékosan megáll):

```powershell
# 1. friss DB + migráció (pl. külön compose DB-vel), majd:
$env:NODE_ENV="production"; $env:BOOTSTRAP_ALLOW_LOCAL_DB="1"
npm run db:bootstrap:production      # elvárt: [BOOTSTRAP] + ártábla
npm run db:bootstrap:production      # elvárt: [NOOP][BOOTSTRAP], 0 új
npm run db:seed                      # elvárt: "Seed megtagadva" + exit 1
Remove-Item Env:NODE_ENV; Remove-Item Env:BOOTSTRAP_ALLOW_LOCAL_DB
```

Runtime image-ben a scriptek futtathatóságának bizonyítása (a lokális
compose stack DB-jével, ahogy a P34/P36 proofoknál):

```powershell
docker run --rm --network kontener_default `
  -e NODE_ENV=production -e BOOTSTRAP_ALLOW_LOCAL_DB=1 `
  -e DATABASE_URL="<a lokális proof-DB URL-je>" `
  kontener-app:p39 npm run db:bootstrap:production
# elvárt: [BOOTSTRAP]/[NOOP] — tsx nélkül, prod node_modules-ból fut

docker run --rm --network kontener_default `
  -e DATABASE_URL="<a lokális proof-DB URL-je>" `
  -e ADMIN_EMAIL="proba@example.com" -e ADMIN_PASSWORD="<erős, egyszeri>" `
  kontener-app:p39 node scripts/create-admin.mjs
# elvárt: [CREATED] proba@example.com
```

Kezdőoldal-ellenőrzés: bootstrapelt DB-vel a 4 típus megjelenik; üres
DB-vel production buildben a semleges szöveg látszik (seed-utalás
nélkül), dev módban a technikai segítség.

## 5. Render-műveletek (a lokális kapu UTÁN, ebben a sorrendben)

1. Commit + push a main-re → Render auto-deploy. A pre-deploy
   `npx prisma migrate deploy` felveszi a P39 migrációt, majd az új
   image Live lesz. (ÚJ DEPLOY KELL: a bootstrap script és a
   wasteKind-kód csak az új image-ben létezik.)
2. Render Shell, törzsadat:
   ```bash
   npm run db:bootstrap:production
   ```
   Elvárt: `[BOOTSTRAP]` összesítő + ártábla. Ismételt futás: `[NOOP]`.
3. Admin (ha még nincs) — jelszó SOHA nem kerül parancssorba,
   history-ba vagy logba; a `read -s` nem echóz:
   ```bash
   export ADMIN_EMAIL="<valódi admin email>"
   read -r -s ADMIN_PASSWORD && export ADMIN_PASSWORD
   node scripts/create-admin.mjs
   unset ADMIN_PASSWORD ADMIN_EMAIL
   ```
   Elvárt: `[CREATED] <email>` (ismételten: `[NOOP]`).
4. Ellenőrzés: kezdőoldalon a 4 típus látszik; árbecslés-szúrópróba a
   3. pont táblája szerint (pl. 4 m³ vegyes, 5 km-en belül → 65 000);
   /api/health zöld; admin belépés működik.

## 6. Amihez a bootstrap garantáltan NEM nyúl

Meglévő konténer státusza/címe, Order, Customer, User, Truck, Job,
Payment, meglévő típus `orderable` kapcsolója és `maxWeightKg` értéke.
Törlés SEMMILYEN ágon nincs. A guardok (NODE_ENV, DATABASE_URL,
migráció-megléte, dev-seed-detektálás, kódütközés) a script fejlécében.
