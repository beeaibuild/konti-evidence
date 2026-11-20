# Strukturált logging + health check — 33. csomag

## JSON log séma

Minden szerveroldali logbejegyzés egyetlen JSON-sor (`src/lib/logger.ts`),
külső függőség nélkül. Mezők:

| Mező | Mikor | Tartalom |
|---|---|---|
| `timestamp` | mindig | ISO 8601 |
| `level` | mindig | `info` \| `warn` \| `error` |
| `event` | mindig | stabil gépi eseménynév (lásd lent) |
| `message` | mindig | rövid emberi leírás |
| `requestId` | kéréshez kötött lognál | szanitizált vagy generált azonosító |
| `method` | ha ismert | HTTP metódus |
| `route` | ha ismert | route-minta (pl. `/api/admin/orders/[id]/assign`) |
| `status` | ha ismert | a kimenő HTTP státusz |
| `error.name/message/stack` | váratlan hibánál | `serializeError` kimenete |
| `auditAction` | audit-írás hibájánál | melyik audit-akció bukott |

A logger `info`/`warn`/`error` a megfelelő console metódusra ír; a
kimenet gépi feldolgozásra való. A sorosítás soha nem dob: a BigInt
stringgé alakul, körkörös/nem sorosítható metadata esetén minimális
helyettesítő rekord készül a hiba tényével.

## Stabil event nevek

`admin_assign_failed`, `admin_cancel_failed`, `admin_jobs_failed`,
`admin_payment_failed`, `admin_reassign_failed`, `job_status_failed`,
`payment_start_failed`, `payment_complete_failed` — a nyolc váratlan
500-as catch-ág; `audit_write_failed` (`auditAction`: `LOGIN_FAILED`
vagy `ORDER_CREATED_PUBLIC`) — a best-effort audit-írások hibaága, ami
továbbra sem töri meg a fő üzleti flow-t; `health_db_error` — a health
probe DB-hibája; `log_serialization_failed` — a logger saját fallbackje.

## Request ID szabály

A bejövő `x-request-id` csak akkor használható újra, ha illeszkedik a
`^[A-Za-z0-9._-]{1,128}$` mintára — minden más (sortörés, szóköz,
ékezet, idézőjel, túl hosszú érték) elutasítva, és `crypto.randomUUID()`
készül helyette. Így felhasználói értékből nem kerülhet sortörés vagy
korlátlan hosszú szemét a logba (log-injection ellen). A request-
kontextus a route-okban szándékosan a CSRF guard UTÁN készül. A váratlan
500-as válaszok `x-request-id` fejlécet kapnak (a JSON body változatlan),
így egy hibabejelentésből a konkrét logsor visszakereshető; a health
válasz minden esetben tartalmazza.

## Érzékeny adatok tiltólistája

A logger SOHA nem logol: jelszót, session cookie-t, Authorization
fejlécet, payment tokent, teljes request bodyt, AUTH_SECRET-et,
adatbázis URL-t. Három védelmi vonal: (1) a logger API-ja nem fogad
Request objektumot — a hívó csak explicit, szűkre válogatott mezőket
adhat át, automatikus header/body-beolvasás nincs; (2) az érzékeny
mintájú metadata kulcsok (password/jelszó/cookie/authorization/token/
secret/session/body) kimaradnak a rekordból — beágyazott objektumokban
is; (3) P33-r2 óta érték-szintű redakció (`redactSensitiveText`) fut
minden logba kerülő stringen, az Error.message-en és a stacken is: a
DATABASE_URL és AUTH_SECRET pontos env-értéke, a credentialt tartalmazó
adatbázis/queue URI-k (postgresql, postgres, mysql, mongodb,
mongodb+srv, redis, rediss, amqp), a Bearer tokenek és az egyértelmű
cookie/session értékadások `[REDACTED]` jelölésre cserélődnek —
tipikus eset a Prisma kapcsolódási hibája, amely a teljes DB-URL-t
hordozná. Credential nélküli URL-ek és normál metadata érintetlenek.

## Health endpoint szerződés

`GET /api/health` (`src/app/api/health/route.ts`), belépés nélkül:

Működő adatbázissal: HTTP 200,
`{ "status": "ok", "database": "ok", "timestamp": "<ISO>" }`.
Adatbázis-hibánál: HTTP 503,
`{ "status": "degraded", "database": "error", "timestamp": "<ISO>" }`.

Minden válaszon: `Cache-Control: no-store` és `x-request-id`. A próba
valódi DB-ping (`SELECT 1` Prisma `$queryRaw`-val); üzleti adatot nem
olvas és nem ír, AuditLog sort nem készít. A DB konkrét hibaüzenete
SOHA nem megy ki a kliensnek — az a strukturált `health_db_error`
logba kerül. `export const dynamic = "force-dynamic"` és Node runtime:
build/static generation alatt nem fut és nem cache-elődik. CSRF guard
nem kell (GET), rate limit ebben a csomagban szándékosan nincs rajta.

Miért nincs auth: a hosting/LB health probe-oknak nincs sessionjük, és a
végpont nem árul el üzleti adatot — csak azt, hogy az app és a DB él.
Pont ezért nincs benne verzió, uptime, kapcsolatszám vagy bármi más
belső részlet sem.

Hosting health-check beállítás: az ellenőrzendő útvonal `/api/health`,
elvárt státusz 200, minden más (503, timeout) unhealthy.

## A 200/503 logika tesztelése

A döntés tiszta, injektálható függvényben él
(`src/lib/health.ts / evaluateHealth(probe)`), így a hiba-ág a probe
elrontásával determinisztikusan tesztelhető — NINCS `HEALTH_FORCE_FAIL`
env, test-only paraméter vagy publikus hibakényszerítő backdoor, és a
közös e2e Postgres konténert sem kell (és nem is szabad) leállítani.

Kézi DB-down smoke (nem a tesztsuite része):

1. `docker compose stop db`
2. `curl -i http://localhost:3000/api/health` → várt: `503`,
   `"status":"degraded"`, `"database":"error"`, és a szerver-logban egy
   `health_db_error` JSON-sor;
3. `docker compose start db`
4. `curl -i http://localhost:3000/api/health` → várt: `200`, `"ok"`.

## Hogyan kell validálni

`npm run test` — az új `tests/unit/logger.test.ts` (rekord-felépítés fix
idővel, error-sorosítás, request-ID szabály, tiltólista, körkörös/BigInt
metadata, console-spy) és `tests/unit/health.test.ts` (200/503 modell,
belső hiba nem szivárog a body-ba). `npm run test:e2e:api` — az új
`e2e/api/health.spec.ts` (200-as szerződés, fejlécek, request-ID
visszaköszönés/szanitizálás, auth és Origin nélkül). A korábbi 68 e2e és
8 UI teszt változatlanul zöld kell legyen. A route-szám a health-tel
24-ről 25-re nő.

## Ismert korlátok

Nincs még log aggregator, alerting, APM vagy metrika (Sentry,
OpenTelemetry, Prometheus szándékosan nem része a csomagnak) — a JSON
formátum ezek későbbi bekötését készíti elő. A logger a processz
stdout/stderr-jére ír; a gyűjtés/rotálás a hosting dolga lesz.
