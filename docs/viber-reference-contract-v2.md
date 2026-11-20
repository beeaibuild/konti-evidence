# Viber-értelmezési szerződés — célzott javítás, review előtt

## Mi változik

A kért művelet felismerése és a konkrét üzleti objektum feloldása külön történik.
Egy rövid címmegnevezés lehet a megrendelés/feladat jelöltként átadott aliasa.
A feloldó az AI által választott slotot ellenőrzi: ha az adott bizonyítékban szereplő
rövid alias több különböző, azonos entitástípusú jelölthöz tartozik, nem enged önkényesen
választani. `REFERENCE_AMBIGUOUS` marad, üres subjectRef-fel és blokkolt végrehajtással.
Kifejezetten szereplő egyedi teljes cím feloldhatja ezt; két slot ugyanahhoz az objektumhoz
nem két külön megrendelés. A művelet INSTRUCTION minősítése a hiányzó kötéstől nem változik.

Nincs új adatbázis-kereső vagy automatikus aliasgenerálás. A jogosult, aktuális megrendelések
jelöltjeit továbbra is a kontextusforrásnak kell szolgáltatnia. Ez a változtatás nem bizonyítja,
hogy az éles adatbetöltő már előállítja a szükséges címrövidítéseket. Nincs új absztrakció,
függőség, migráció vagy automatikus végrehajtás.

A `CONDITIONAL_INSTRUCTION` conditionIntent nélkül most `CONDITION_MISSING` diagnózist
kap a determinisztikus feloldóban is; ismert megrendelés mellett sem válhat végrehajthatóvá.

A prompt és provider-sémaleírás általánosítva különíti el a közlőt a leírt cselekvés
szereplőjétől, kéri a teljes ablak feldolgozását (a primary utáni üzenetekkel együtt),
és az idézett állítás megerősítésének/korlátozásának megőrzését. A hiányzó entitás nem
feltétel, az anonimizálási helyőrző önmagában nem üzleti jelentés. Ezek szemantikai
utasítások: új valós modellmérés nélkül nem állítjuk, hogy a modellhibákat megoldották.
Az actorRef-et nem töröljük vakon a jelentéstípus alapján; a valódi saját cselekvés megmarad.

## Üzemi jelentés és kontextus

A felhasználó megerősítette: a Tetőmester jellegű megnevezés címrövidítés, amely a
megrendelésekhez kapcsolható; a csere és a 4m3 ettől függetlenül felismerendő teendő.
A platós náluk Ford Transit kisteherautó, nem konténeres autó. Ezt üzemi kontextusként
kell szolgáltatni, nem általános, minden vállalkozásra érvényes rendszerprompt-szabályként.
A márka-/típusinformáció az eredeti mérési bemenetben nem szerepelt, nem pontozzuk vissza.

## Mérési státusz

Az eredeti, fagyasztott mérés és gold változatlan. A három futás automatikus eredménye
6/18, 5/18, 5/18; a rögzített rubrikával 4 stabil, 2 ingadozó, 12 sikertelen eset.
A tartalmi review asszisztensi, nem független emberi minősítés. A 246 fájlhash és mind az
54 esetre a független offline újrapontozás egyezett. A vitatható aktusgoldokat, a közlőből
levezetett cselekvő diagnosztikáját és a törölt telefon helyőrzőjének kötelező értelmezését
külön mérőeszköz-problémaként kell megőrizni; utólag nincs javított sikerarány.

A 18 eset megismerése utáni változtatás miatt ez a csomag a további munkában **fejlesztői
halmaz**. A korábbi mérés történeti tartalékhalmaz-eredmény marad. Az új konfigurációra
általánosítási állítás új fagyasztást és új, előre annotált tartalékhalmazt igényel.
A korábbi R01–R10 is fejlesztői halmaz, a rajta mért 9/10 továbbra sem üzemi pontosság.

## Ellenőrzés és korlátok

312 offline regresszió / 20 fájl PASS, ebből 11 új, szintetikus teszt. A 20 kiválasztott
értelmezési teszt és az értelmezési források strict TypeScript-ellenőrzése PASS.
Az offline tesztek a feloldó és export/szabályozás működését ellenőrzik, nem a modell
új prompt melletti megértését. Teljes alkalmazás-CI és P63 ezen a változaton még nem futott.
Nincs providerhívás, staging- vagy productiontelepítés.
