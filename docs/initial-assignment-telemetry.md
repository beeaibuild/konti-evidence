# Első AI-kiosztás: mérés

Az első ajánlás és a sikeresen mentett kiosztás a meglévő AuditLog táblában kapcsolódik össze.
Nincs új adatbázistábla, migráció vagy külső mérőrendszer.

## Ajánláskérés

Az `AI_INITIAL_ASSIGNMENT_RESULT` esemény minden jogosult, a sebességkorláton átjutott kérés eredményét rögzíti:
`version`, `requestId`, `status`, `reason`, `durationMs`, `routeAvailable`, valamint a kiadott ajánlás belső
sofőr-, autóazonosítója és naptári dátuma. Név, rendszám, cím, GPS-pont, prompt és szolgáltatói hibaüzenet nem kerül ebbe az eseménybe.
Az API `measurementId` mezője az esemény azonosítója. Az idő a szerveroldali ajánláskészítés ideje,
az utólagos jogosultságellenőrzéssel együtt; nem böngészőben mért válaszidő és nem a naplóírás ideje.

A `RECOMMENDATION` sikeres ajánlás. A `NO_CANDIDATE`, `CONTEXT_LIMIT`, `STALE`, `NO_RECOMMENDATION`, `UNAVAILABLE`,
`REQUEST_REJECTED`, `SERVER_ERROR` eltérő okból maradó kézi kiosztást jelöl.
A `reason` megkülönbözteti például a modell tudatos visszalépését (`MODEL_DECLINED`), hibás kimenetét (`INVALID_OUTPUT`),
a szolgáltatói időtúllépést (`TIMEOUT`) és a kikapcsolt konfigurációt.
A megtagadott jogosultságú, CSRF-hibás vagy korlátozott kérésből nem keletkezik ilyen mérés.
Naplóírási hiba esetén az ajánlás használható marad, de `measurementId: null` érkezik, és biztonságos figyelmeztetés kerül a szervernaplóba.
A technikai esemény nem jelenik meg a rendelés üzleti eseménynaplójában.

## Mentett döntés

A meglévő `JOB_ASSIGNED_KIHELYEZES` esemény `data.initialAssignment` mezője tartalmazza a döntést.
A szerver csak az ugyanahhoz a rendeléshez és az aktuális kezelőhöz tartozó eredeti mérésre támaszkodik;
a kliens ajánlási mezőit nem fogadja el viszonyítási alapként.

| decision | Jelentés |
| --- | --- |
| ACCEPTED | A mentett sofőr, autó és dátum megegyezik a kiadott ajánlással. |
| MODIFIED | Legalább egy eltér; a changes.driver/truck/date jelzi, melyik. |
| MANUAL_NO_RECOMMENDATION | Volt mérés, de nem volt használható ajánlás. |
| MANUAL_UNMEASURED | Nem volt ellenőrizhető mérési hivatkozás. |

A `reference` értéke VERIFIED, INVALID vagy ABSENT; az `outcome` a kiadott eredmény.
A mentési tranzakció visszagörgetésekor döntés sem marad vissza. Dupla mentés nem növeli a sikeres kiosztások számát.
A félbehagyott vagy újratöltött űrlap nem elutasítás. Az ACCEPTED az értékek egyezését jelzi,
nem bizonyítja, hogy a kezelő elolvasta az indoklást. Több ajánláskérés külön próbálkozás;
a mentés csak a hozzá küldött, ellenőrzött méréshez kapcsolódik.

## Lekérdezés

Az alábbi PostgreSQL lekérdezések az utolsó hét próbálkozásait és mentett döntéseit külön összesítik.
Az elfogadási arányhoz csak az ACCEPTED és MODIFIED döntések tartoznak; a hiányzó mérés nem modell-elutasítás.

```sql
SELECT data->>'status' AS outcome, data->>'reason' AS reason,
       count(*) AS attempts,
       round(avg((data->>'durationMs')::numeric)) AS avg_duration_ms
FROM "AuditLog"
WHERE action = 'AI_INITIAL_ASSIGNMENT_RESULT'
  AND "createdAt" >= now() - interval '7 days'
GROUP BY 1, 2 ORDER BY attempts DESC;
```

```sql
SELECT data->'initialAssignment'->>'decision' AS decision,
       count(*) AS saved_assignments,
       count(*) FILTER (WHERE data->'initialAssignment'->'changes'->>'driver' = 'true') AS driver_changed,
       count(*) FILTER (WHERE data->'initialAssignment'->'changes'->>'truck' = 'true') AS truck_changed,
       count(*) FILTER (WHERE data->'initialAssignment'->'changes'->>'date' = 'true') AS date_changed
FROM "AuditLog"
WHERE action = 'JOB_ASSIGNED_KIHELYEZES'
  AND data->'initialAssignment'->>'version' = '1'
  AND "createdAt" >= now() - interval '7 days'
GROUP BY 1 ORDER BY saved_assignments DESC;
```

A módosítva mentett kiosztásnál az `overrideReason` a kezelő által választott ok, az `overrideNote` az opcionális kiegészítése. Hiányzó mérési hivatkozásnál a kezelő által megadott ok külön `reportedOverrideReason` mezőbe kerül, az ellenőrzött módosítások összesítésén kívül. A korábbi döntéseknél az ok hiányozhat. A mérési azonosítóval az eredeti javaslat is visszakereshető.

Az alábbi bontás minden ellenőrzött, módosítva mentett kiosztást egyszer számol. Egy döntés több mezőt is érinthet.

```sql
SELECT COALESCE(d->>'overrideReason', 'NOT_RECORDED') AS reason,
       count(*) AS modified_assignments,
       count(*) FILTER (WHERE d->'changes'->>'driver' = 'true') AS driver_changed,
       count(*) FILTER (WHERE d->'changes'->>'truck' = 'true') AS truck_changed,
       count(*) FILTER (WHERE d->'changes'->>'date' = 'true') AS date_changed
FROM "AuditLog" CROSS JOIN LATERAL (SELECT data->'initialAssignment' AS d) AS a
WHERE action = 'JOB_ASSIGNED_KIHELYEZES'
  AND d->>'decision' = 'MODIFIED' AND d->>'reference' = 'VERIFIED'
  AND "createdAt" >= now() - interval '7 days'
GROUP BY 1 ORDER BY modified_assignments DESC, reason;
```

Ez a mérés nem tartalmaz tényleges menetidőt, megtett kilométert vagy pushkézbesítést.

## Ellenőrzött staging-állapot — 2026-10-18

[#129](https://github.com/beeaibuild/kontener-app/pull/129), commit `3a18f54c5766b077d9356f9dbd607bdf47eff52d`.
A teljes CI 3159 Vitest- és 174 UI-próbája sikeres. Élő szolgáltatói próbában a változatlan mentés ACCEPTED,
a kizárólag dátumban módosított mentés MODIFIED / changes.date=true eredményt adott; mindkét hivatkozás VERIFIED.
Az ajánlások ideje 1988 és 859 ms volt. Mindkét tesztrendelés (R-2026-0056 és R-2026-0057) lemondva,
0 aktív tesztfeladat, a konténerek szabadok. Ezek tesztadatok, nem üzemi elfogadási arányt igazoló minták.
