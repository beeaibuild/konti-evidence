# Kaszkádkockázat – első változat

Állapot: staging pilot, #131 beolvasztva. 2026-10-19 01:15:41 (Europe/Budapest): staging live, commit `2b40d85cd918e074929fcaaa3d3f4a5b112a9500`, Render deploy `dep-dbal51u1egvs73b4t1b0`. A 01:10-es staging export elkészült; a `20261019001000_dispatch_timing` migráció sikeres. A teljes CI zöld (3186 Vitest teszt, típusellenőrzés, migrációs paritás, build, böngészős regressziók, Docker). Health: alkalmazás/adatbázis OK, a kiadott commit egyezik. Adatszámok: 62 rendelés, 63 Job, 31 TransportTask, változatlanok. `KONTI_CASCADE_ENABLED=true` csak stagingen. A mai üres terven a panel betöltése és a vizsgálat hibamentes. Az alap késéslánc/cserejavaslat és időadatmentés stagingpróbája sikeres; részletes bizonyítás alább. Productionhoz nem nyúltunk.

## Használat

Migráció után `KONTI_CASCADE_ENABLED=true` kapcsolja be a mai Napi operáció oldalon. Csak aktív admin vagy magasabb jogosultság érheti el. A szerveroldali figyelő bevezetésétől az adatbázisos admin főkapcsolónak is ON állapotban kell lennie; OFF a teljes kézi és automatikus funkciót tiltja. Az időadatok szerkesztése nem indít Google- vagy AI-kérést. A kézi vizsgálat külön gomb. A korábbi, csak látható oldalon futó kétperces automatikát a külön főkapcsolós szerveroldali figyelő váltja fel; lásd [kaszkádfigyelő](cascade-monitor.md). A háttérfigyelő alapból OFF, külön migráció és ütemező szükséges hozzá.

Az érkezési ablak a feladat első helyszínére vonatkozik, belső fuvarnál a felvételre. A helyszíni idő az érkezés és továbbindulás közötti összes munka. A belső fuvar céljáig számolt utazást a rendszer külön hozzáadja. Az első változat nem tárol külön céloldali slotot vagy külön felrakási/lerakási időket. A mezők Budapest szerinti helyi percek, a tervezett naphoz kötve. Éjfélen átnyúló ablak nem adható meg; a DST miatt nem létező vagy kétértelmű időpont ismeretlen marad.

Folyamatban lévő feladatnál a teljes hátralévő időt kell megadni, az utazással együtt. A szerver mentési időponttól számol; tíz perc után új megerősítés szükséges. Nem következtet a GPS-sebességből a munkavégzés végére. Üres mező ismeretlen, az explicit nulla megengedett.

## Számítás és korlátok

A meglévő validált feladatsorrendből, friss GPS-ből vagy ismert végpontból és friss hátralévő becslésből indul. Az ablak kezdetéig várakozást, helyszíni időt és szakaszonkénti utazást továbbviszi. Hiányzó koordináta, idő, útbecslés vagy járműfolytonosság esetén nem talál ki nullát. Az érkezés még ismert lehet, miközben a befejezés és a következő feladat már nem értékelhető. Tíz percnél kisebb/egyenlő tartalék kockázatjelzés.

Csak valódi Google `CURRENT` útbecslés használható. Ez a lekéréskori forgalom pillanatképe minden szakaszra, nem a délutáni forgalom előrejelzése és nem garantált ETA. Nem állítja, hogy a késés oka konkrét dugó. Legfeljebb 48 aktív feladat, 6 érintett sofőr, sofőrönként 12 feladat, 12 pontos csere és 64 egyedi útvonalszakasz elemezhető; túllépéskor nincs részleges optimumnak beállított eredmény. Legfeljebb négy párhuzamos útlekérés, azonos pontok között nincs fizetős hívás. Elemzésenként egy meglévő provider-konfigurációjú AI-kérés, retry nélkül; a meglévő felhasználói rate limit érvényes.

Az alternatívakeresés a meglévő pontos csere-validálást használja (kiosztás, kapacitás, konténer, státusz, zárolás), nem teljes napi optimalizáló. Nem keres egyirányú átadást, üres sofőrnek kiosztást vagy tetszőleges új sorrendet. Csak teljesen ismert, azonos feladathalmazú eredményt mutat, amely legalább egy szigorú késést csökkent és más szigorú késést nem növel. Puha határidő romolhat; a teljes előtte/utána ütemezés megtekinthető. Legfeljebb három alternatíva jelenik meg.

Az AI csak a szerver által kiszámított jelöltek azonosítóját és számszerű késés-/utazásadatait kapja. Név, cím, rendszám és GPS nem része ennek a kérésnek. Csak létező azonosítót választhat, vagy elutasíthat. Hibás vagy nem elérhető AI mellett a számítás használható. Nem generál új időpontot vagy okot. Az alternatíva tanács, nem mentett végrehajtási engedély: a tényleges átszervezéshez külön friss ellenőrzés és jóváhagyás szükséges.

A késési partnerüzenet determinisztikus, másolható tervezet a számított ETA-val és sofőr/jármű adataival. Nincs címzettfeloldás, küldési végpont vagy automatikus küldés. Külső értesítés külön, jóváhagyást igénylő későbbi fejlesztés.

## Konzisztencia és bevezetés

Az időadatmentés ugyanabban a tranzakcióban auditál, és a várt `updatedAt` időbélyeget valamint az aktív státuszt ellenőrzi. Az elemzés végén újraolvassa a feladatokat, elutasítja a megváltozott feladathalmazt és a két percnél régebbi vagy más napra átnyúló elemzést. Erőforrás/GPS változás a kérés közben külön végrehajtási friss ellenőrzést igényel; ez nem tranzakcionális kiosztási snapshot.

Migráció: két nullable JSONB `operationTiming`, továbbá Job `updatedAt` időbélyeg. A meglévő feladatok időadatai üresek maradnak. Staging bevezetés előtt adatbázis-mentés, migrációs paritás és CI szükséges; production változtatás nincs ebben a körben. Kikapcsolás: `KONTI_CASCADE_ENABLED=false`; az additív mezők megmaradhatnak.

## Ellenőrzés

Helyben: `npx tsc --noEmit`, Prisma kliensgenerálás, 27 célzott unit/API teszt (`cascadeImpact`, `cascadeChoice`, `cascadeTimingApi`). Utóbbi API-tesztek mockolt tranzakciót használnak; nem helyettesítik a valós PostgreSQL-integrációt. A PR teljes CI-je és migrációs paritása sikeres. Staging kézi próba még szükséges: két egymást követő szigorú slot, egy menthető csere, hiányzó helyszíni idő, elavult hátralévő becslés, egyidejű módosítás, Google-kiesés, AI-kiesés. Tablet nem kell az adminoldali első próbához.


## Staging elfogadási bizonyítás

2026-10-19 01:24–01:27 (Europe/Budapest), staging, `STAGING-CASCADE-20261019`: két mesterséges, ismert végpontú horgony és három tervezett belső tesztfuvar. Nem fizikai GPS-/tabletpróba; koordináták korábbi jelölt tesztvégpontokból, feladatok közvetlenül létrehozott, auditált fixture-ök. Valódi Google CURRENT útbecslés: A első feladat 01:50 körüli érkezésével kb. 17 perc késés, utána a második 02:23 körül kb. 22 perc késés. A napi terv nélküli első futás nem ajánlott cserét; a két tesztsofőr/jármű napi tervbe vétele után az AI az A1 ↔ B1 cserét emelte ki. A felületen megjelenő összes szigorú késéscsökkenés 37 perc (az összegzés és a sorok kerekítése eltér), az alternatívában minden érkezési határidő tartható, a legszűkebb tartalék 9 perc. Két ellenőrizhető ETA-/sofőr-/járműadatú partnertervezet elkészült; küldés és kiosztási végrehajtás nem történt.

A felületi időadatmentés valódi PATCH-útvonala és auditja igazolt: az A1 helyszíni idő üresre állításakor az A1 befejezése és az A2 érkezése ismeretlen lett, javító csere nem jelent meg. Az értéket 5 percre visszaállítottuk. Cleanup: 3 tervezett fixture LEMONDVA, 2 horgony SIKERTELEN; 0 aktív fixture, az ideiglenesen létrehozott napi terv eltávolítva, kapacitás nem változott, kiosztások változatlanok. 62 Order és 63 Job változatlan; 5 lezárt TransportTask teszttörténet megmaradt. Auditok: STAGING_CASCADE_FIXTURE_CREATED, STAGING_CASCADE_PLAN_CREATED, DISPATCH_TIMING_UPDATED, STAGING_CASCADE_FIXTURE_CLEANED. Éles adatot nem érintettünk.

Ez az alap kaszkád-/cserejavaslat-/hiányzóadat-próba elfogadva; nem bizonyítja a tényleges csere-végrehajtást, élő GPS késésdetektálását, háttérfigyelést vagy külső üzenetküldést.
