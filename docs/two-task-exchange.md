# Kétfeladatos AI-csere

Két külön sofőr–autó páros egy-egy mai, `TERVEZETT` feladatának pontos helycseréje. A beérkező feladat az eltávozó helyére kerül; a többi feladat sorrendje megmarad. Rendelési munka és belső fuvar egyaránt részt vehet. Fuvarcsoporthoz kötött, megkezdett, lemondott, más napra tervezett vagy hiányosan kiosztott feladat nem cserélhető.

A közös végállapotot ellenőrizzük: napi terv, aktív sofőr és autó, nyitott GPS-műszak párosítása, konténer/rendelési feltételek, napi kapacitás. A két csere megőrzi a feladatszámokat, ezért két teljesen megtelt autó is cserélhet. Ez nem igazol súly-, méret- vagy fizikai kompatibilitást.

## Google és AI

A Google számolja mindkét hátralévő tervezett út utazási idejét és távolságát, a jelenlegi és a cserélt változatra. A közös irányított útszakaszokat egy kérésen belül egyszer kérjük le, legfeljebb négy párhuzamos hívással. A kérés költségkorlátja 64 különböző útszakasz; ha a teljes összehasonlítás ezt meghaladná, egyetlen Google-hívás sem indul. Azonos szakasz többszöri tényleges megtétele többször számít az összegbe.

A kiindulás a friss, elfogadható GPS-pont, vagy a folyamatban lévő feladat végpontja ugyanazon az autón. Ismeretlen kezdőpont, feladatkoordináta, járműfolytonosság vagy szolgáltatói útszakasz esetén nincs számszerű ajánlás. Nincs légvonalból számolt menetidő vagy MOCK-visszaesés. Legfeljebb 16 partner és autónként 12 hátralévő tervezett feladat vizsgálható; ez korlátozott keresés, nem napi optimum.

Csak legalább 5 perc **és** 20% közös becsült utazási előnyt adó jelölt jut az AI elé. Az AI az alkalmazás által ellenőrzött csereazonosítót választhatja ki vagy elutasíthatja az összeset; saját kiosztást, címet vagy megtakarítást nem írhat elő. Az AI-nak álnevesített terhelési tények és szolgáltatói számok mennek, nem sofőrnevek, rendszámok vagy GPS-koordináták.

A forgalom a lekéréskori, azonnali indulásra számolt pillanatkép minden szakaszon. Nem későbbi forgalom-előrejelzés. Az összeg nem tartalmaz rakodást, munkaidőt, érkezési időt vagy kitalált telephelyre visszatérést. A terhelés feladatszám, nem munkaóra. A megtakarítás becslés, nem realizált üzemi eredmény.

## Jóváhagyás és végrehajtás

Egy privát diszpécserchat-ajánlás tartalmazza mindkét feladat változtatását és a közös becsült előnyt. Elfogadás után új Google-értékelés készül. Ha a második feladat, bármelyik beosztás vagy jogosultság megváltozott, illetve az előny eltűnt, a teljes ajánlás érvénytelen. A tíz percnél régebbi előnézet nem hajtható végre.

A külön végrehajtási megerősítés hash-sel kötődik a két változtatáshoz. A két kiosztás, mindkét tervezett sorrend javítása, a standard naplók, értesítési outbox és az ajánlás végrehajtási eredménye egy adatbázis-tranzakcióban készül el. Hiba esetén mindkettő visszagördül. Sikeres kérés ismétlése a mentett eredményt adja vissza; nem cserél újra. Előnézet és Mégse nem oszt át feladatot.

## Bekapcsolás és állapot

A meglévő proaktív AI-feltételek mellett `KONTI_AI_PAIR_REASSIGNMENT_ENABLED=true` engedélyezi az automatikus cserék keresését. Az egyszerű kiosztási/áthelyezési ellenőrzések elsőbbséget kapnak. Alapérték kikapcsolt. A fejlesztési PR elkészítése nem kapcsolja be és nem telepíti a funkciót. Nincs új migráció, APK vagy függőség.

Az adatbázisos tesztek a teli autók cseréjét, az előnézet írásmentességét, a második írás hibájának teljes visszagörgetését, a társ feladat közbeni lemondását és a sikeres végrehajtás ismétlését ellenőrzik. A CI eredménye a PR-ben látható. Valódi Google- és chat-átvétel külön staging-próbát igényel.
