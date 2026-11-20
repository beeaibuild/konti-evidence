# Szerveroldali kaszkádfigyelés – staging pilot, alapból OFF

A #131 alap kaszkádszámítását kiegészítő, böngészőtől független figyelő.
Állapot: #134 beolvasztva, stagingre telepítve, kétperces ütemező konfigurálva. A teljes funkció OFF állapotban maradt.
Productionváltoztatás nincs. Új alkalmazásfüggőség, AI-modell vagy üzenetsor nincs.

## Főkapcsoló

A Napi operáció panelen a teljes kaszkádfunkció ki-be kapcsolható. Alapból OFF.
Környezeti rendelkezésre állás: `KONTI_CASCADE_ENABLED=true` ÉS
`KONTI_CASCADE_MONITOR_AVAILABLE=true`; az adatbázisos kapcsoló ettől még OFF.
Az admin PATCH ugyanabban a tranzakcióban naplózza a kapcsolást és érvényteleníti
az előző generáció várakozó értesítéseit. Bekapcsoláskor a bejelentkezett operátor
lesz a címzett; aktív OPERATOR grant és aktív privát tervezési tagság szükséges.
Másik címzett tetszőleges azonosítója nem adható meg a kliensből.

OFF mellett nincs időadat-szerkesztés, kézi vagy automatikus vizsgálat, fizetős útvonal-/AI-kérés vagy
értesítés. Futás közbeni kikapcsolás után a következő útvonalkérés nem indulhat,
a régi generáció eredménye nem publikálható. A már szolgáltatóhoz elküldött kérés
nem vonható vissza; a kikapcsolás előtt megkezdett email-kézbesítés még befejeződhet.
Visszakapcsolás friss generációt kezd, régi értesítéseket nem játszik vissza.
A kézi vizsgálat és az időadatmentés API-ja is tiltott (503); a felület nem tölti be
az időadatokat, elrejti a kézi vezérlőket és törli a megjelenített eredményeket.
A korábbi időadatokat és auditokat a kikapcsolás nem törli.
A kézi futás minden fizetős útvonalkérés előtt, az AI-hívás előtt és eredményadás
előtt ellenőrzi az engedélyt és az indulási generációt. OFF→ON nem élesít régi eredményt.
Az időadatmentés a főkapcsolóval azonos DB-zároláson belül történik. A régi böngészőoldali
automatikus számítás megszűnik: nincs második automata, amely a főkapcsolót megkerülné.

## Ütemezés és költség

Külön Render Cron Job, `*/2 * * * *`, parancs:
`node scripts/trigger-cascade-monitor.mjs`. Stagingen létrehozott szolgáltatás: `kontener-cascade-monitor-staging`
(`crn-dbason6q1p3s73cvce30`), Frankfurt, Node 22, külön függőségtelepítés nélkül. Környezeti változói:

- `CASCADE_MONITOR_APP_BASE_URL`: a célkörnyezet pontos HTTPS originje.
- `CASCADE_MONITOR_CRON_SECRET`: legalább 32 karakteres véletlen titok; ugyanaz
  az alkalmazásban és a cronban, más hitelesítéstől elkülönítve.

A POST `/api/internal/cascade-monitor/run` önálló gépi hitelesítésű, nem kér
böngészős sessiont. GET nem indít feldolgozást. Az ütemező 110 másodperc után
megszakítja a várakozást; a szerver futáskerete 90 másodperc. A szerver eredménye
HTTP-hibával jelzi a hibás futást. A cron ismételt indítása nem ismétli a slotot.

Minden kétperces időszelethez globális, tartós foglalás tartozik. Előző futás
függő állapotában 120 másodpercig nincs új foglalás. Újraindítás nem nullázza
sem a foglalást, sem a budapesti naptári napi költségkeretet.

A következő 120 percben záródó szigorú ablakokhoz vezető teljes feladatláncot
vizsgáljuk. Egy távoli ablakhoz nem kérünk le csonka előzményláncot. Az eredeti
48 feladatos, 6 sofőrös, sofőrönként 12 feladatos korlát érvényes. Legfeljebb
24 fizetős szakasz/futás és 1500 háttérhívás/budapesti nap; a foglalás hálózati
hívás előtt, DB-ben történik, a sikertelen/bizonytalan hívás is fogyasztja.
A kerettúllépés nem eredményez részleges, zöldnek feltüntetett napi értékelést.
A pilot minden releváns ciklusban frissít, nincs ciklusok közti útvonalcache;
azonos szakaszt futáson belül egyszer kér. Az azonos pontok közti szakasz ingyenes.
Szekvenciális hívásokkal minden kérés előtt megállítási és keretellenőrzés van.
Automatikus retry és háttérbeli AI-/cserekeresés nincs. A 1500 hívás induló
pilotkeret, nem árajánlat; a tényleges Google SKU díja külön ellenőrizendő.

## GPS, munkafázis és hiányzó adat

A meglévő jogosult GPS-műszak és sofőr–autó páros ellenőrzése marad; csak nem
szimulált, legfeljebb 100 m hibájú és 120 másodperces pont használható.
Google CURRENT és érvényes, legfeljebb kétperces szolgáltatói időbélyeg kell.
Ez lekéréskori forgalom, nem a későbbi indulás forgalmi előrejelzése.

Az opcionális `operationTiming.phase` a régi rekordokat nem változtatja meg:

| Fázis | Számítás |
| --- | --- |
| `TOTAL_REMAINING` (alapérték) | A régi, explicit teljes hátralévő időből indul a feladat végpontján; 10 percig használható. |
| `TO_SITE` | Aktuális GPS → első helyszín, megadott helyszíni idő, szükség esetén belső fuvar célja, majd következő munkák. |
| `AT_SITE` | Explicit teljes hátralévő idő, beleértve a célhelyig utazást, ha van. GPS-ből nem talál ki munkavéget. |
| `TO_DESTINATION` | Csak belső fuvarhoz: GPS → cél, majd következő munkák. A már megtörtént felvétel ablakát nem riasztja újra. |

A fázist a diszpécser adja meg; ez nem automatikus geofence vagy sofőri
munkafázis-felismerés. TOTAL_REMAINING/AT_SITE esetén az éppen folyó feladat
első érkezését nem becsüljük utólag: a következő munkák becsülhetők. Hiányos vagy
lejárt inputból UNKNOWN lesz. Emiatt a funkció teljes operatív használatához
időablakok, helyszíni idők és megfelelő munkafázis szükségesek.

A feladatok, sorrend, koordináták, jármű- és sofőrállapot, jogosult műszakpárok
és a friss GPS-alap megléte a számítás körül újraellenőrzött. A GPS normál
elmozdulása önmagában nem teszi elavulttá a másodpercekkel előbb számolt becslést.
A legfeljebb 90 másodperces tanácsadó snapshot nem tranzakcionális kiosztási engedély.

## Riasztási életciklus

- Legfeljebb 10 perc tartalék: két egymást követő friss kör után figyelmeztetés.
- Legalább 5 perc késés: két kör után magasabb súlyosság.
- Legalább 15 perc késés és 30 percen belül záródó ablak: azonnali sürgős jelzés.
- Azonos lánc új GPS-pont miatt nem küld új értesítést. Új érintett feladat,
  magasabb súlyosság vagy legalább további 10 perc késés indokol új jelzést.
- Két teljes, 15 percnél nagyobb tartalékot mutató kör oldja fel az állapotot.
  UNKNOWN nem oldhat fel korábbi riasztást. Háromperces mérési rés megszakítja
  a megerősítési sorozatot.

A riasztási memória nap/generáció/sofőr szerint tartós. A szöveg felsorolja a
lánc veszélyeztetett feladatait, határidőit, becsült érkezéseit és késését.
Nem állít konkrét dugót vagy garantált érkezést. A korábbi chatüzenet időbélyeges
előzmény; új intézkedés előtt friss vizsgálat kell.

## Belső kézbesítés és emberi döntés

A tartós outboxból előbb privát `PLANNING:active` SYSTEM_EVENT készül,
nem AI által írt ajánlás. Aktív címzettet és tervezési tagságot a kézbesítéskor
is ellenőrzünk; nem hozunk létre tagságot és nem személyesítünk meg felhasználót.
A sofőr- és ügyfélfeed meglévő jogosultsági határa érvényes marad.

Böngésző bezárása mellett a belső e-mail a figyelmeztető csatorna. Az aktív
címzett User.email címét használjuk; partner/ügyfél címére ez a folyamat nem küld.
A meglévő Resend gateway, szanitizált hibakódok és stabil idempotenciakulcs marad.
A staging EMAIL_MODE tiltása érvényes: ott a chat létrejön, az email státusza
EMAIL_DISABLED, nem SENT. Későbbi bekapcsolás nem küldi ki ezeket a régi leveleket.
Bizonytalan kézbesítés NEEDS_REVIEW/SENDING marad, nincs vak újraküldés.
A legfeljebb kétperces várakozó értesítés állapotát és inputját újraellenőrizzük;
a lemondott/átosztott vagy más generációhoz tartozó értesítés CANCELLED.

Az alternatívakeresés továbbra is a kézi kaszkádpanel művelete. Sem a riasztás,
sem annak megnyitása nem oszt át feladatot. Friss előnézet és külön emberi
végrehajtási megerősítés marad szükséges. Külső partnerértesítés kizárólag a
meglévő másolható tervezet; nincs automatikus partnerküldés vagy új APK.

## Üzemeltetés és átvétel

A futás, hívásszám, összesített lefedettség, riasztás, kézbesítés és kapcsolási
audit DB-ben megmarad. A log csak összesítést/okkódot tartalmaz, nyers GPS-t nem.
Az adminpanel jelzi a hibát és a hatperces szívveréskimaradást. Ez nem külső
watchdog: Render cron-hibaértesítést/független szívverésfigyelést külön kell
konfigurálni, a leállt ütemező nem tud saját magáról értesítést küldeni.

Bevezetési kapuk: teljes CI és migrációs paritás; staging DB-mentés, migráció;
külön, alapból OFF ütemező; majd ellenőrzött staging pilot. Kötelező próbák:
bezárt diszpécseroldal melletti riasztás; kézi munkafázisok; hiányzó/lejárt GPS és
munkaidő; Google-kiesés; napi keret; két párhuzamos futás; OFF futás közben;
visszakapcsolás; tagság-visszavonás; lemondás/átosztás; chat jogosultsági határ;
email fake/sink próba és változatlan feladatkiosztás. Valós emailt stagingből nem
engedünk ki. A bezárt böngészős ütemezési próba kész; a fizikai GPS-es riasztási próba még nem végrehajtott bizonyítás.


## Staging átvételi bizonyítás – 2026-10-19 09:54–09:58 (Budapest)

- Kiadott commit: `38e033c23479404e2d93a35f583049a4a3ffeba2`; Render deploy `dep-dbasnkjbc2fs73cl9ngg`, live 09:54:31. A health alkalmazás/adatbázis OK és a commit egyezik.
- Staging logikai mentés: 09:31-es export elkészült a migráció előtt. A `20261019060000_cascade_monitor` migráció befejezett. Adatszámok előtte/utána: 62 Order, 63 Job, 36 TransportTask; változatlanok.
- CI a `793413d830c55b7e0f168345a34371c034bd696d` fejen: 1986 unit, összesen 3221 Vitest teszt 255 fájlban; 174 UI regresszió, külön szállítólevél-, jogosultsági, mobil/tablet- és stabilitási próbák; típusellenőrzés, migrációs paritás, production és Docker build sikeres. CI: https://github.com/beeaibuild/kontener-app/actions/runs/39216294862 .
- OFF: az adminpanel nem mutat időadatszerkesztést vagy kézi vizsgálatot; a közvetlen GET `/api/admin/cascade` válasza: „A teljes kaszkádfunkció kikapcsolt.”
- ON: a kézi vizsgálat az üres mai terven sikeres. A diszpécseroldalt és a staging API-lapot bezártuk; a 09:56:12-es cron ettől függetlenül HTTP 200-zal lefutott, DB-ben egy OK futás, 0 csoport, 0 fizetős szakasz, 0 értesítés.
- OFF→ON→OFF felületi próba sikeres. Végállapot: `enabled=false`, `generation=4`. A kézi vezérlők és az előző eredmény eltűntek. A 09:58:14-es cron ismét HTTP 200; nem keletkezett új számítás: futások száma továbbra is 1, úthívás 0, kézbesítés 0.
- Az ütemező 09:54:16-os első hívása még a régi alkalmazáspéldányra futott és 404-et kapott. A 09:54:31-es webes élesedés után mindkét ellenőrzött kör sikeres; nem állandó végponthiba.
- A staging e-mail-tiltás megmaradt, tényleges levél nem ment ki. A chat/e-mail kézbesítési, kvóta-, párhuzamossági és generációs eseteket valós PostgreSQL-integrációs tesztek fedik; az e-mail szolgáltató a tesztekben injektált helyettesítő.

Ez az átvétel a teljes főkapcsolót, a visszakapcsolást, a szerveroldali ütemezést és a leállás utáni tétlenséget bizonyítja. A mai terv üres volt: nem bizonyít élő GPS-es késésészlelést, valódi késésriasztás stagingkézbesítését vagy fizikai tablet-/sofőrpróbát. A korábbi #133-as cserepróba bizonyítási határa változatlan. Production nem változott.
