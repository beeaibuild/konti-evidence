# Konti Sofőr V1 — elfogadott program (2026-10-16)
A #114 első APK-ja köztes alap; a kibővített V1 csak az alábbi ellenőrzések után kész.

- Egy Android-app: feladatlista, indítás, lezárás, fotó, szállítólevél, külső Maps; natív háttér-GPS és meglévő offline pontsor.
- A tablet teherautóhoz/rendszámhoz kötött. A napi sofőr belépése aktiválja a sofőr–autó kapcsolatot és GPS-t; kézi autóválasztás megmarad. Egy sofőr/autó egyszerre egy aktív kapcsolatban lehet.
- Kijelentkezés lezárja a GPS-műszakot. Minden nap Europe/Budapest éjfélkor a még nyitott app-session és műszak lejár. Nem 24 órás szabály. A lejáratot szerver és offline natív kliens is érvényesíti.
- Minden mentett feladatváltozás értesít: kiosztás, tartalom/dátum/sorrend módosítás, átosztás, visszavonás/törlés. Kézi és jóváhagyott AI-végrehajtás ugyanazt a mechanizmust használja. Átosztás mindkét érintett sofőrt értesíti. A listabetöltés/bejelentkezés önmagában soha.
- Hangos, vizuális Android push zárolt képernyőn is; érintéskor szerveroldali friss jogosultságvizsgálattal az aktuális feladat nyílik. Másik sofőrre átadott/törölt feladat adatai nem kerülhetnek vissza.
- A diszpécsertérkép mutatja a teherautó helyét, a mérési időt és a pontosságot; az elavult/műszak nélküli adatot elkülöníti.
- Feladatkezelés kezdetben online. Nem ígérünk teljes offline feladatkezelést.
- Valódi pushhoz Firebase Android-alkalmazás és FCM szerverjogosultság szükséges; a Google Routes-kulcs erre nem használható.

Átvétel: belépés/GPS; kijelentkezés; éjfél (téli/nyári és DST-nap, offline is); ütköző belépés; kézi tablet/autó kötés; új/módosított/átosztott/törölt feladat push; reggeli zajmentesség; offline/késői/ismételt push; értesítésből friss feladat/jogvesztés; élő/elavult térképi pozíció; Samsung képernyőzár/Maps/fotó/PDF.
