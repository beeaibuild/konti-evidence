# AI-ajánlás új rendelés első kiosztásához

Az új rendelés kiosztási űrlapja automatikusan ajánl sofőrt, autót és napot.
Felirat: **AI-ajánlás – hagyd jóvá, vagy módosítsd, ha szükséges.**
A konténert az üzemeltető választja; a javasolt három mező szabadon módosítható.
Csak a kiosztás mentése hoz létre munkát a meglévő `/assign` útvonalon.
Az ajánlás kérése nem foglal konténert, nem módosít rendelést, nem ír chatbe és nem küld pushértesítést.

## Választás

- A kért nap (múltbeli igénynél Budapest szerinti ma) az első. Ha nincs megfelelő páros, legfeljebb további hat napot vizsgálunk.
- Beállított napi terv esetén csak az ott szereplő aktív sofőrök és autók ajánlhatók; üres terv nem jelent szabad napot.
  Terv hiányában a meglévő kézi kiosztás szabályával egyezően minden aktív erőforrás jelölt lehet.
- Az autó aktív rendelési és belső feladatai egyaránt fogyasztják a `maxFuvarPerNap` korlátot.
- Ma a hiteles nyitott GPS-műszakok párosítását is figyelembe vesszük.
- A meglévő kiosztás a sofőr napjának végére illeszt. Az útbecslés ehhez igazodik: az utolsó feladat végpontja,
  üres napnál friss és megbízható GPS. Járműváltásnál nincs kitalált indulási pont.
- Google-útbecslés csak aznapi jelöltnél: lekéréskori forgalom, nem későbbi érkezési idő.
  A hiányzó útbecslés ismeretlen, nem nulla. Jövőbeli napra csak a terv és a feladatszám ad támpontot.
- Az AI csak ellenőrzött jelöltazonosítót választhat. A megjelenített neveket, dátumot és indoklást a szerver adja.
  Nem kap nevet, rendszámot, belső azonosítót, pontos címet vagy GPS-koordinátát.
- AI-válasz után új olvasás ellenőrzi a kiválasztott jelöltet. Közbeni lényeges változásnál `STALE`, új ajánlás kérhető.

## Határok és működés

`POST /api/admin/orders/[id]/assignment-recommendation`: same-origin védelem, friss OPERATOR jogosultság a hívás
előtt és után, `no-store`, felhasználónként 6 kérés/perc. A meglévő Konti AI-konfigurációt és provideradaptert használja.
Legfeljebb 128 jelölt, 16 különböző útszakasz (4 párhuzamos lekérés), egy AI-hívás automatikus ismétlés nélkül.
Túl nagy kontextusnál, hibás AI-válasznál vagy kikapcsolt szolgáltatásnál kézi kitöltés marad.
Nincs új függőség, adatbázis-migráció vagy titok.

Az AI késői válasza nem írja felül a már kézzel módosított mezőket. A javaslat külön gombbal újra alkalmazható.
Az eredeti kézi kiosztás szerződése megmarad, beleértve a kézi kapacitás-felülbírálatot; az AI ellenőrzése nem foglalás.
Mentéskor a meglévő tranzakciós aktív erőforrás-, napi terv-, konténer- és rendelésállapot-védelem fut.

A feladatszám nem munkaidő, és az ajánlás nem igazol rakodási időt, teherbírási megfelelést vagy határidőre érkezést.
2026.10.18-án a valós OpenAI/Google és az operátori felület staging-próbája sikeres (R-2026-0055).
Ajánlás: STAGING TEST Sofőr / STG-AI-01 / 2026-10-18, 11,9 km / 20 perc Google-útbecsléssel.
A mezők átírhatók és a javaslat visszaállítható. Mentés nélkül nem keletkezett job vagy kiosztási audit,
a kiválasztott konténer szabad maradt. A tesztrendelés lemondva. Ezen a próbán tényleges kiosztást nem mentettünk;
azt a fake-provideres teljes böngészős CI fedi. Terhelt valós munkanapos és production-átvétel továbbra is külön lépés.

## Eredmény és mentett döntés mérése

Az ajánláskérés technikai auditmérést ír, üzleti kiosztást továbbra is csak a kifejezett mentés végez.
A részletes szerződés és heti SQL-összesítés: [initial-assignment-telemetry.md](initial-assignment-telemetry.md).
