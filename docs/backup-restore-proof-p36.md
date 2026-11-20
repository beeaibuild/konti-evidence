# Proof-of-restore — 36. csomag

Cél: bizonyítani, hogy a P34-es backup nemcsak elkészül, hanem külön
üres céladatbázisba ténylegesen visszaállítható, migrálható, és az app
a visszaállított adatbázissal működik.

**Állapot: a próba VÉGREHAJTVA és ZÖLD (2026-07-12).** A lenti
jegyzőkönyv a tényleges lefutás eredményeit rögzíti. A folyamat leírása
változatlanul érvényes minden későbbi ismételt próbához — a restore-
képességet érdemes időnként (pl. nagyobb sémaváltozások után) újra
bizonyítani, nem egyszeri tényként kezelni.

## A folyamat (pontos PowerShell parancsok)

Előfeltétel: `.env.staging` a szokásos kötelezőkkel
(POSTGRES_PASSWORD, AUTH_SECRET). Minden parancs a projekt gyökeréből.

```powershell
# 0. Kenyelmi valtozo a hosszu parancsokhoz
$C = @("--env-file", ".env.staging", "-f", "docker-compose.staging.yml")

# 1. Staging stack indul (db + app)
docker compose @C up -d
docker compose @C ps   # elvart: db healthy, app healthy

# 2. Ismert tesztadat biztositasa. Friss/ures staging DB-nel egyszeri
#    seed MEGENGEDETT (ez staging, nem production!):
docker compose @C --profile tools run --rm migrate
# seed: a repo-bol futtatva a staging DB-re — ehhez a DB-portot NEM
# nyitjuk ki; a legegyszerubb ismert adat kezzel, SQL-lel is rogzitheto,
# vagy hasznald a mar korabban feltoltott staging adatokat. A lenyeg:
# jegyezz fel EGY konkret, visszakeresheto adatot, pl.:
docker compose @C exec -T db psql -U kontener -d kontener -c "SELECT email FROM \"User\" ORDER BY email LIMIT 3;"
# -> ird fel az egyik emailt (ellenorzo adat), es a rendelesek szamat:
docker compose @C exec -T db psql -U kontener -d kontener -c "SELECT count(*) FROM \"Order\";"

# 3. Backup a P34 script Docker agaval (P36-r2: az -EnvFile a compose
#    --env-file parametere lesz - nelkule a ${POSTGRES_PASSWORD:?} elhal)
.\scripts\backup-postgres.ps1 -Docker -EnvFile .env.staging
# -> jegyezd fel a kiirt fajlnevet: backups\kontener-<ts>.dump

# 4-6. Restore kulon ures DB-be + migrate deploy + app-restore + health
#    (a script kezi megerositest ker: ird be a celnevet)
.\scripts\restore-postgres-staging.ps1 `
  -BackupFile backups\kontener-<ts>.dump `
  -EnvFile .env.staging `
  -WithApp

# 7. SQL-ellenorzes a restore DB-n (a script is fut kettot; kezzel:)
docker compose @C exec -T db psql -U kontener -d kontener_restore -c "SELECT count(*) FROM _prisma_migrations;"
docker compose @C exec -T db psql -U kontener -d kontener_restore -c "SELECT email FROM \"User\" WHERE email = '<a 2. lepesben felirt email>';"
docker compose @C exec -T db psql -U kontener -d kontener_restore -c "SELECT count(*) FROM \"Order\";"
# elvart: a migracioszam > 0, az email megvan, a rendelesszam egyezik a 2. lepesben felirttal

# 8. Admin login smoke a restore app ellen (3100!)
curl.exe -i -X POST http://localhost:3100/api/auth/login -H "content-type: application/json" -H "origin: http://localhost:3100" -d '{"email":"<staging admin email>","password":"<staging admin jelszo>"}'
# elvart: 200, Set-Cookie session Max-Age=43200

# 9. Read-only uzleti ellenorzes: bongeszoben http://localhost:3100/admin
#    -> belepes utan a rendeleslista betolt, es a 2. lepesben ismert
#    adat(ok) latszanak. SEMMIT nem modositunk.

# 10. Cleanup — a restore-proba felszamolasa (az elo stacket nem erinti)
# FONTOS: semmilyen `docker compose down` — az projekt-szintu es az elo
# app/db service-t is leallithatna. Kizarolag service-szintu stop + rm,
# kizarolag az app-restore-ra; a db service es a staging volume marad.
docker compose @C -f docker-compose.restore.yml --profile restore stop app-restore
docker compose @C -f docker-compose.restore.yml --profile restore rm -f app-restore
docker compose @C exec -T db psql -U kontener -d postgres -c "DROP DATABASE IF EXISTS kontener_restore;"
# A DROP elott ellenorizd, hogy a parancsban a _restore nev all!
```

## Operációs tanulság a végrehajtott próbából

Az első proof-kör félrevezető eredményt adott: a staging adatbázis
seed nélküli, ÜRES állapotban lett mentve, így a restore is üres
adatbázist állított vissza — ami formálisan „sikeres", de semmit nem
bizonyít az üzleti adatok visszaállásáról. Ez nem restore-hiba volt,
hanem próba-tervezési hiba. Tanulság, beépítve a folyamatba: a 2. lépés
(ismert tesztadat feljegyzése) NEM opcionális — mentés előtt mindig
győződj meg róla, hogy az adatbázisban van visszakereshető üzleti adat,
és a jegyzőkönyvbe konkrét rekordot írj fel. Seed után új backup és új
restore futott, és az admin felület bizonyította az adatok visszaállását.

## Rollback döntési pont

Ha a 6–9. lépés bármelyike nem zöld: a restore-próba SIKERTELEN, és ezt
kell jegyzőkönyvezni — nem szabad addig „tesztelt backupról" beszélni.
Ilyenkor: app-restore logok (`docker compose ... logs app-restore`),
a dump épség-ellenőrzése (`pg_restore --list`), és a hiba elhárítása
után a teljes próba elölről. Az élő staginget a próba egyetlen lépése
sem módosította, ott rollbackre nincs szükség.

Éles következmény (P34 runbook): mivel az app-image visszaállítása nem
állítja vissza a DB-t, éles incidensnél mindig a legutóbbi ITT, ezzel a
folyamattal bizonyított mentés számít visszaállítási pontnak.

## Végrehajtási jegyzőkönyv (a helyi próba után töltendő ki)

| Mező | Érték |
|---|---|
| Dátum, végrehajtó | 2026-07-12, Tamás (helyi Windows/Docker staging) |
| Használt backup fájl | backups/kontener-<ts>.dump — a SEEDELT staging DB második mentése (az első, üres-DB mentés érvénytelen próbának minősítve, lásd az operációs tanulságot) |
| Ellenőrzött ismert adat | 1 db KIHELYEZÉSRE_VÁR státuszú rendelés + admin user |
| SQL-ellenőrzés (migrations/user/order) | zöld — migrate deploy: „No pending migrations to apply"; user- és rendelésadatok jelen |
| Health a restore app ellen | 200 OK, database: ok (app-restore, 3100-as port) |
| Admin login smoke | sikeres, dashboard betöltött |
| Read-only üzleti ellenőrzés | a rendelés a helyes státuszban megjelent, konténerstatisztikák konzisztensek |
| Cleanup megtörtént | igen (stop + rm -f app-restore, restore DB drop) |

**Záró állítás:** A backup visszaállíthatósága staging környezetben
bizonyítva: 2026-07-12, a seedelt staging adatbázis pg_dump custom
formátumú mentéséből, külön restore-adatbázisba, teljes app-szintű
ellenőrzéssel (health 200, admin login, üzleti adat visszaállt). A P36
a csomagtervben ELKÉSZÜLT.
