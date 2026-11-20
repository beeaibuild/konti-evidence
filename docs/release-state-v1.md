# Konténerrendelő alkalmazás — állapotjelentés (v1, 2026. július 12.)

## Mit tud a rendszer most

A webalkalmazás a konténeres hulladékszállítás teljes alapfolyamatát
lefedi. Az ügyfél a publikus felületen rendel: címet és konténertípust
ad meg, azonnali árkalkulációt kap, és leadja a rendelést. Az admin
felületen a rendelések teljes életciklusa kezelhető: kihelyezés
ütemezése, sofőr hozzárendelése, csere, elszállítás, lemondás, fizetés
követése. A sofőrök saját nézetben látják a napi feladataikat és
jelentik a teljesítést. Minden lépésről visszakereshető napló készül.
A fizetés jelenleg szimulált (teszt) módban működik — valódi
bankkártyás fizetés bekötése a következő ütem része.

## Milyen minőségben

A rendszert több mint 200 automatizált teszt védi (a belső logikától a
teljes felhasználói folyamatokig), és minden fejlesztési lépés csak
akkor kerül be, ha a teljes tesztsor zöld. Beépített védelem van a
jelszó-próbálgatásos támadások ellen, a más oldalakról indított hamis
kérések ellen, és a naplózás úgy készült, hogy jelszó vagy más titok
soha nem kerülhet bele. A rendszer állapota egyetlen címen bármikor
ellenőrizhető (health endpoint) — ezt a felügyelet automatikusan is
figyelheti.

## Ami külön kiemelendő: az adatok biztonsága

Az adatbázisról szkriptelt mentés készíthető, és — ez a ritkább — a
visszaállítást ténylegesen ki is próbáltuk: egy mentésből külön
adatbázisba visszaállítottuk a rendszert, elindítottuk ellene az
alkalmazást, és ellenőriztük, hogy a rendelési adatok hiánytalanul
visszajöttek. Tehát nem csak „van mentés": bizonyított, hogy baj
esetén vissza is állítható belőle a működés.

## Mi következik

A mostani változat kipróbálásra kész: zárt előnézetként webre kerül,
hogy valós használat közben derüljön ki, mi hiányzik és mit érdemes
másképp. A visszajelzések alapján ütemezzük a következő köröket —
a már betervezett nagyobb tételek: valódi bankkártyás fizetés,
automatikus email-értesítések, számlázás-integráció és ügyfél-oldali
rendeléskövetés. Az alap úgy készült, hogy ezek bővítésként épülnek
rá, nem átépítést igényelnek.

## Egy mondatban

A rendszer üzleti alapfolyamata működik és tesztelt, a biztonsági és
üzemeltetési alapok bizonyítottak — az éles indulás előtt a fizetés
élesítése és a kapcsolódó kényelmi funkciók vannak hátra.
