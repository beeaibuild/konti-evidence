# Prisma migration workflow (22. csomag)

A 22. csomaggal a séma-kezelés `prisma db push`-ról verziózott
`prisma migrate` folyamatra áll át. A `schema.prisma` NEM változott;
a baseline init migráció
(`prisma/migrations/20260705120000_init/migration.sql`) a v1 RC séma
pontos DB-struktúráját rögzíti — új mező, enum, index vagy constraint
nélkül.

## Alapszabály

**Élesben (és stagingen) kizárólag `npm run db:migrate:deploy` fut.**
A `db:push` és a `db:reset` fejlesztői eszköz marad: `db push`-nak
éles adatbázison SEMMILYEN körülmények között nincs helye (nem
verziózott, drift esetén adatot dobhat), a `db:reset` pedig
definíció szerint töröl.

## Scriptek

| Script | Mit csinál | Hol használható |
|---|---|---|
| `npm run db:migrate:dev` | migráció készítése + alkalmazása + generate | csak dev |
| `npm run db:migrate:deploy` | függő migrációk alkalmazása, semmi más | staging, prod (és dev) |
| `npm run db:push` | séma rátolása migráció nélkül | csak dev, eldobható DB |
| `npm run db:reset` | force-reset + seed | csak dev |

## 1. Friss DB setup (új gép, üres adatbázis)

```bash
docker compose up -d
cp .env.example .env
npm install
npx prisma generate
npm run db:migrate:deploy   # a baseline init migráció felépíti a sémát
npm run db:seed             # dev adatok
npm run dev
```

Ez váltja a korábbi `npm run db:reset` első lépését friss környezetben.
(A `db:reset` továbbra is működik dev-resethez, de push-alapú marad.)

## 2. Meglévő dev DB kezelése (db push-sal épült)

A már létező dev adatbázisok sémája azonos a baseline-nal, csak a
`_prisma_migrations` könyvelőtábla hiányzik. Két lehetőség:

**A) Eldobható a dev adat (egyszerűbb):**
```bash
npm run db:reset            # vagy: drop DB + friss setup az 1. pont szerint
```
Megjegyzés: a `db:reset` push-alapú, ezért utána a migrate a DB-t
baseline-olatlannak látja. Ha migrate-tel akarsz továbbmenni, a friss
setup (1. pont) a tiszta út.

**B) Megtartod az adatot — baseline-olás resolve-dal:**
```bash
npx prisma migrate resolve --applied 20260705120000_init
npx prisma migrate status   # elvárt: "Database schema is up to date"
```
Ez csak beírja a könyvelésbe, hogy az init már "alkalmazott", a sémához
nem nyúl. Élesítéskor ugyanez a minta érvényes minden olyan környezetre,
amely korábban push-sal épült.

## 3. Új migráció készítése (fejlesztés, v2 csomagok)

```bash
# 1. schema.prisma módosítása (csak nyitott csomag scope-jában!)
npm run db:migrate:dev -- --name <rovid_nev>
# 2. a generált SQL átnézése commit előtt — kötelező lépés
# 3. typecheck + smoke, majd a migráció a csomag zipjének része
```

Elvek: additív migrációk preferálva (oszlop/tábla DROP külön,
késleltetett csomagban); egy csomag = legfeljebb egy migráció;
kézzel a `migrations/` mappába utólag belenyúlni tilos (kivéve a
dokumentált baseline-esetet).

## 4. Staging / prod deploy folyamat

```bash
# minden migrációs deploy előtt KÖTELEZŐ:
pg_dump "$DATABASE_URL" > backup-$(date +%Y%m%d-%H%M%S).sql

npx prisma migrate status     # mi fog lefutni — átnézés
npm run db:migrate:deploy     # csak a függő migrációk, interakció nélkül
# ezután: app build/indítás, smoke a célkörnyezeten
```

A `migrate deploy` nem generál, nem seedel, nem resetel — pontosan
ezért ez az egyetlen megengedett séma-parancs élesben. Seed élesben
tilos (gyenge fix jelszavak; guard a 23. csomagban jön).

## 5. Rollback / backup alapelv

Prisma-migráció automatikusan nem vonható vissza, ezért a védelem
sorrendje: (1) mentés minden migráció előtt (fenti pg_dump — rutin,
nem opció), (2) hiba esetén restore a mentésből + előző app-verzió
indítása, (3) additív sémaváltozások, hogy az előző app-verzió az új
sémán is fusson (roll-forward barát). Félbeszakadt migrációnál a
`prisma migrate resolve --rolled-back <nev>` rendezi a könyvelést a
restore után. Részletes próba: 30. csomag (backup/restore stagingen).

## 6. Sandbox-korlát és kötelező saját gépi ellenőrzés

A fejlesztő-sandboxban a Prisma engine binárisok letöltése blokkolt
(`binaries.prisma.sh` — lásd
[`runtime-verification-v1.md`](runtime-verification-v1.md)), ezért a
baseline SQL ott nem `prisma migrate diff`-fel generálódott, hanem a
`schema.prisma`-ból determinisztikusan, a Prisma 5.x PostgreSQL
konvencióival készült. Emiatt merge előtt saját gépen KÖTELEZŐ a
drift-ellenőrzés:

```bash
# friss, üres DB-n (pl. külön compose DB vagy droppolt lokális DB):
npx prisma generate
npx prisma migrate deploy
npx prisma migrate diff \
  --from-url "$DATABASE_URL" \
  --to-schema-datamodel prisma/schema.prisma \
  --exit-code
# elvárt: exit 0, "No difference detected"
```

Ha a diff bármit jelez, a baseline SQL-t kell a jelzett pontokon a
tényleges Prisma-kimenethez igazítani (vagy friss DB-n `prisma migrate
dev --name init`-tel újragenerálni és a fájlt lecserélni) — a
`schema.prisma`-hoz továbbra sem nyúlunk.
