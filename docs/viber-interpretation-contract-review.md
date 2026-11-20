# Összevont értelmezési review és következő kontroll

Alap: `62fe96b746d68d0e438122fa85e466380654210a`. A legutóbbi mérés 60,666666666666664
pontos, kritikus jelzés nélküli FAIL. A társalgási javítás önmagában +3,125 pontot adott,
a többi változás együtt -2,058333 pontot. Ez nem általános modellminőség-javulás.

## Mit mutatnak a megőrzött válaszok?

| Terület | Konkrét megállapítás | Kezelés |
| --- | --- | --- |
| Szerepkötés | VBR-03 kérdése a sofőrt teszi tárgynak a feladat helyett. | A beszélő, végrehajtó/címzett és üzleti tárgy fogalma külön marad. |
| Benchmarkreprezentáció | VBR-04 a végrehajtót a subjectRefs-ben várja, explicit actorRef nélkül. | V1-et most nem írjuk át; egy helyes actor-kötés eltérő alakja is pontvesztést okozhat. |
| Besorolás | VBR-09 három külön rekordot vár, köztük az indoklást OTHER-ként. | A kötött felbontás befolyásolja a pontszámot; az nem közvetlen üzleti pontossági százalék. |
| Szándék és jogosultság | QUESTION/SUGGESTION/STATUS_REPORT mellett INSTRUCTION eddig operátori AFTER_APPROVAL lehetett. | Ellentmondó pár blokkolása; a nyers állítás megőrzésével. |
| Terv és utasítás | VBR-02 egy terv követésére adott operátori utasítást is tartalmaz. | PLAN_OR_INTENT nem tiltja automatikusan az INSTRUCTION jelzést. |
| Szaknyelv | A modell konténeres cserét műszakváltásnak vagy általános változásnak értelmez. | Konténeres alapjelentés; explicit sofőr-/autócsere felülírja, hiányzó tényt nem találunk ki. |

## Egyetlen összevont változás

A közös provider-séma leírásai együtt pontosítják a szerepeket, tárgyhivatkozásokat,
besorolást, beszédaktust és szaknyelvet. Nincsenek bennük benchmarknevek vagy mintaválaszok.
A determinisztikus feloldó ACTION_SIGNAL_CONFLICT diagnosztikával blokkolja a fenti három
ellentmondó párt: READ és BLOCKED_INCOMPLETE_BINDING. Egy valódi operátori tervutasítás
továbbra is csak jóváhagyás után végrehajtható; sofőri utasításból csak javaslat lehet.
Nem kötünk automatikusan feladatot a modell helyett, és nem emelünk jogosultságot szerep alapján.
A system policy, mezők, V1-adatok, oracle és pontozási képlet változatlanok. Új függőség nincs.

## Új szintetikus kontrolljelöltek — még nem futtatott modellmérés

Az Ibolya/Tölgyes feladatot és Réka/Ádám személyt külön, ismert jelöltként kell átadni;
üres feladatkontextust csak K7-nél. Ezeket a mondatokat a runtime útmutató nem tartalmazza.
Nem követelünk önkényes rekordszámot: a tartalmi kötéseket és jogosultságokat kell ellenőrizni.

| Azonosító | Mondat és szerep | Előre rögzített ellenőrzés |
| --- | --- | --- |
| K1 | OP: „Réka, az Ibolya sorra a hatköböst vidd.” | Réka végrehajtó, Ibolya feladat, 6 m³ attribútum; csak jóváhagyás után. |
| K2 | DRIVER: „Kész a csere az Ibolya soron.” | Konténercsere állapotjelentése; nincs automatikus lezárás vagy kiosztás. |
| K3 | OP: „Ibolya után menj a Tölgyesre.” | Helyes feladatsorrend; a beszélő nem válik feladattárggyá; csak jóváhagyás után. |
| K4 | DRIVER: „Be tudnám vállalni a Tölgyest is.” | Javaslat, ismert Tölgyes tárggyal; nem jóváhagyott kiosztás. |
| K5 | OP: „Réka helyett Ádám vigye a Tölgyest.” | Explicit végrehajtócsere Ádámra; nem konténercsere, nem automatikus írás. |
| K6 | DRIVER: „Gumizást tervezek a saját teherautómra.” | Járműművelet és terv, ismert saját autóval; nem fuvarmegrendelés. |
| K7 | DRIVER: „Ezt is elvigyem?” | Kérdés, feloldatlan „ezt”; üres feladatkontextusból nem találhat ki tárgyat. |
| K8 | DRIVER: „Köszi, jó munkát! 🙂” | Társalgás, üzleti szereplő/tárgy/akció nélkül. |

## Ellenőrzött és nem ellenőrzött állítások

221 teszt PASS 11 fájlban, köztük 10 új, V1-től eltérő szintetikus szerződéskontroll.
Ezek megadott szemantikus válaszokat tesztelnek, nem a modell nyelvértését.
Célzott strict TypeScript, whitespace és új sorhossz-sértések ellenőrzése PASS.
A 20 orákulumeset 100%. A két legutóbbi valós csomag saját, kulcs nélküli visszajátszása
hibára állított fetch mellett a teljes riportot változatlanul adja: 59,6 és 60,666666666666664.

K1–K8 elvárásait modellfuttatás előtt kell véglegesíteni. Nem független üzemi minták;
egy további, eddig nem látott valós üzenetkészlet szükséges az általánosítás megítéléséhez.
A következő kontrollban ugyanaz a rögzített modell és pontozó maradjon, az új sorozat külön
eredményként, a V1-gyel össze nem átlagolva. Új providerfutás vagy telepítés most nem történt.
A teljes repó ellenőrzését a későbbi PR CI-je végzi.
