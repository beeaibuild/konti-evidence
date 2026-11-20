# Címrövidítés kapcsolása nyitott megrendelésekhez

A #171 feloldási védelmére épülő, közzététel előtti helyi javítás.

Operátori módban az értelmezési kontextus a nyitott megrendelésekben is keres
címrövidítést, nem csak a napi feladatlistában. Ez a már kint lévő konténerhez
új cserefeladat nélkül tartozó megrendelést is jelöltté teheti. A lekérdezés az
aktív OPERATOR módot, a canUseOperatorScope jogosultságot és a hívóazonosságot
ellenőrzi; DRIVER módban sem címkeresés, sem új Order-adatátadás nincs.

A szerver csak az id, address.city és address.street mezőket olvassa, legfeljebb
501 nem TELJESITVE/LEMONDVA megrendelésből. Ha több mint 500 nyitott megrendelés
van, az eredmény incomplete és minden címalias elmarad; nincs első találatra
szűkítés. Elavult/hiányos snapshot vagy a kanonikus jelöltek 500-as korlátja
miatti részlegesség esetén szintén nem állítunk cím-egyértelműséget.

A város/közterület teljes szövege a szerveren marad. A jelölt aliasa kizárólag
az üzenetben már szereplő szó szerinti részlet lehet. Ékezetet, kis-/nagybetűt
és írásjelet normalizálva, egész szavakra keresünk; nincs fuzzy helynévkitalálás.
A közterület neve, típusa, teljes utcacím és város+utcacím képezhet keresési mintát.
Több nyitott megrendelés közös rövidítése mindegyik jelöltön megmarad; a #171
REFERENCE_AMBIGUOUS védelme blokkolja az önkényes választást. Az AI nem kap új
címet, telefonszámot, ügyfélnevet, megjegyzést vagy számlázási adatot.

A címolvasást az order.context.v1 eredetjelzés rögzíti, az orderAddressLookup.complete
mező a jelöltkör teljességét jelzi. Az input legfeljebb 50 üzenet/50 000 karakter;
nagyobb bemenetnél az aliasolvasás incomplete. A művelet felismerése továbbra is
független a címkötéstől, és automatikus üzleti írás nincs.

Helyi ellenőrzés: 335 offline regresszió 21 tesztfájlban PASS, benne 23 új teszt.
A valós olvasóadaptertől a provider-bemenetig és a determinisztikus feloldóig vezető
utat is teszteljük, mockolt adatbázissal és scripted providerrel. A módosított
források/test sorhossza 140 alatt, whitespace-ellenőrzés PASS. Teljes alkalmazás-
típusellenőrzés, CI/P63 és élő adatbázisos próba ezen a változaton még nem futott.
Nincs új függőség, migráció, modellhívás vagy telepítés.

A történeti 18 esetes mérés és gold változatlan; ez a halmaz a további fejlesztésben
már fejlesztői adat. Az új kontextussal jobb modellminőséget most nem állítunk.
