# Konti Sofőr Android-app – staging pilot

Egyetlen Android-app tartalmazza a natív GPS/műszakkezelést és a meglévő Konti sofőrnézetet. A helyben csomagolt Capacitor 7 kezdőképernyő indítja/zárja a GPS-műszakot. A „Sofőr feladatok megnyitása” külön, natív hídtól mentes WebView-ban nyitja meg a staging /sofor oldalt. A közösségi background-geolocation plugin 1.2.26 MIT licencű Android gyűjtőjét egy Activitytől független foreground service használja. Nincs fizetős GPS-plugin vagy előfizetés.

## Telepítés
1. A staging APK-t telepítsd a Samsung tabletre (Android 6 vagy újabb; Google Play szolgáltatások szükségesek).
2. Az app „Sofőr feladatok megnyitása” gombjával jelentkezz be sofőrként a staging Kontiba, nyisd meg a /sofor/gps oldalt és kérj párosítókódot. A kód egyszer használható, legfeljebb öt percig érvényes.
3. Az appban add meg a kódot és az eszköz nevét. A párosítás 90 napig érvényes (a szerverpontok hétnapos megőrzésétől független); a sofőr a /sofor/gps oldalon bármikor visszavonhatja. A natív GPS nem tárol webes jelszót vagy sessiont; a külön feladat-WebView a szokásos webes session cookie-jával tartja fenn a belépést.
4. Engedélyezd a pontos helyet és az értesítéseket. Android 11-től az appbeállításokban külön válaszd a helyengedély „Mindig” lehetőségét. A kérés gombját az értesítési engedélyhez szükség esetén ismét nyomd meg.
5. Samsung: akkubeállításokban az app legyen korlátozás nélküli és az alvó állapotba sosem kerülő alkalmazások listáján. A menüpontok One UI verziónként eltérhetnek.
6. Válaszd a hozzád rendelt autót, nyomd meg a Műszak indítása gombot. Az állandó értesítésnek meg kell jelennie. Műszak végén az appban állítsd le.
7. Admin: /admin/gps – a pont mérési ideje és pontossága látszik. Két percnél régebbi pont elavult; a késve feltöltött pont nem válik friss méréssé.

## Működés és határok
- A GPS callback 15 másodperces intervallumot kér, legfeljebb 30 másodperces gyűjtési csomagolással. Mozgásban legalább 25 méter és 15 másodperc, állásban 60 másodperc után ment új pontot. Android ezt akkukímélés vagy rossz GPS-vétel miatt késleltetheti; nem garantált pontos ütemezés.
- A natív SQLite-sorban minden pontnak stabil UUID-ja van. Feltöltés natív HTTPS-en, a WebView JavaScriptjétől függetlenül történik. Csak visszaigazolt pontazonosító törlődik. Hálózati hibára natív JobScheduler is újrapróbál.
- Műszak indításához hálózat kell. Leállítás helyben működik; a zárás később feltöltődik. Előbb a zárást, majd az időben még a műszakhoz tartozó pontokat fogadja a szerver.
- A korábbi GPS-kísérőapp műszaka legfeljebb 16 órás; a Sofőr V1 sessionje az adott napi magyar éjfélig tart. A szerverpontok megőrzése hét nap. 30 000 függő pontnál leáll a gyűjtés és hibát jelez; nincs csendes adateldobás. Jogosultsági vagy hibás csomag miatti végleges visszautasításnál leáll és megőrzi a sort.
- Az AndroidKeyStore AES-GCM kulcsával titkosított állapotban él az önálló GPS-credential; a koordináták appprivát SQLite-ban vannak. Backup és cleartext forgalom tiltott. A webes sessionnel a natív API nem érhető el, a GPS-token pedig nem webes JWT.
- A feladatkapcsolat a szerver által aláírt, legfeljebb 60 másodperces friss feladatpillanatkép. Offline lejárat után a pont feladata ismeretlen marad. A feltöltés idején aktív feladattal nem címkézzük át a régi pontokat. Ez nem teljes feladat-eseménytörténet; anomáliadetektálásnál a bizonytalanságot meg kell tartani.
- Bezárt Activity után a natív service tovább él. Force-stop után Android nem enged automatikus újraindulást. Tablet újraindítása vagy force-stop után nyisd meg az appot: a nyitott műszakban a „GPS folytatása” gomb a szerver ellenőrzése után újraindítja a szolgáltatást. Le is zárhatod az előző műszakot, majd indíthatsz újat. A próba nem ígér automatikus reboot-resume működést.
- Az APK staging csomagnévvel és debug aláírással készül. Éles terjesztés előtt saját tartós aláírókulcs és frissítési folyamat kell.
- Az ENABLE_GPS_TRACKING kapcsoló alapértéke kikapcsolt; kizárólag stagingben engedélyezett. Production kiadás/engedélyezés külön munka.

## Kötelező fizikai pilot
Két tableten teljes műszak: 30 perc zárt képernyő, legalább 30 perc Google Maps navigáció, 15 perc hálózat nélkül, majd visszakapcsolás; műszakzárás offline; appváltás és Activity bezárása; párosítás visszavonása; töltőről levétel; újraindítás. Ellenőrizd a szerver mérési időit, a hiányzó szakaszokat, a duplikációt és az akkufogyasztást. A CI emulátoros tesztje natív tárolást és az Activity nélküli callback-fogadást igazol, nem a Samsung GPS-rádió/One UI teljes műszakos megbízhatóságát.


## Egységes sofőrnézet (1.1 — korábbi köztes változat)
- Feladatlista, indítás/lezárás, fotófeltöltés és szállítólevél az eddigi webes felülettel, az appon belül. A feladatkezeléshez internet kell; offline csak a meglévő natív GPS-sor működik.
- A feladat-WebView nem Capacitor BridgeActivity, nincs benne JavaScript–natív híd. Nem kapja meg a GPS-credentialt. Csak a staging HTTPS-origin navigál az appon belül; a navigációs link és a telefonszám a rendszer megfelelő alkalmazását nyitja. SSL-hibánál nincs továbblépés.
- A rendszer fájlválasztója kezeli a feltöltést, képnél kamera is választható. A PDF-ek a rendszer letöltéskezelőjébe kerülnek, az értesítésből megnyithatók.
- A „GPS / műszak” gomb visszatér a kezdőképernyőre, nem zárja le a műszakot. A webes űrlapot visszatérés előtt mentsd; a feladatnézet újranyitáskor újratölt.
- A csomagnév változatlan, versionCode 2. A CI debug aláírókulcsa futásonként eltérhet. Ha a régi próbaappra nem telepíthető: előbb zárd le a műszakot, ellenőrizd a nulla függő pontot, majd távolítsd el a régi appot, telepítsd az újat, és párosítsd újra. Függő GPS-pontot tartalmazó appot ne távolíts el.
- Kézi átvételi próba: belépés → feladatlista → tesztfeladat indítás/lezárás → fotó kamera és fájlválasztó útján → szállítólevél aláírás/PDF → külső navigáció → visszatérés → friss GPS-pont → műszakzárás. A CI a natív elkülönítést, URL-szabályokat és a GPS Activitytől független működését is ellenőrzi; a teljes belépéses folyamatot a Samsung tableten kell végigpróbálni.


## Sofőr V1 (2.0) — új működés
Az elfogadott program: [driver-v1.md](driver-v1.md). Az app első telepítésekor a böngészőben kért, egyszer használatos sofőrkóddal párosíts, válaszd ki a tablet teherautóját, majd kösd a rendszámhoz. Utána a natív kezdőképernyőn email/jelszó belépés indítja a GPS-t és nyitja az appon belüli feladatnézetet. A jelszót nem tároljuk; a webes HttpOnly cookie-t natív HTTPS-válaszból telepítjük, nem adjuk át a távoli JavaScriptnek. Az új belépés megvárja a korábbi GPS-sor feltöltését. A login újrapróbálása idempotens.

A GPS / műszak gomb csak visszalép a kezdőképernyőre; a Kijelentkezés leállítja a GPS-t. A feladatnézet logout-kérését a natív kliens azonos originen, egyetlen konkrét POST-útvonalon kezeli, hogy a helyi követés a hálózati kérés előtt leálljon. Nincs általános távoli natív híd. Minden nap Europe/Budapest éjfélkor lejár a session és a műszak. Az offline gyűjtő sem fogad éjfél utáni pontot. Korábban mért, később feltöltött pont megmarad.

A rendszám szerkesztése/újratelepített tablet leválasztása az admin /admin/gps oldalon van. Aktív sofőr mellett nincs csendes autócsere. A staging debug keystore CI-gyorsítótára csak átmeneti: törlődhet vagy áganként eltérhet, ezért az APK-k közötti közvetlen frissítés nem garantált. A 1.1-es APK aláírása még eltér: a cseréhez előbb zárt műszak és nulla függő pont kell, majd eltávolítás. Éles terjesztéshez továbbra is külön tartós release-aláírás kell.

### Valódi FCM push bekötése
A Firebase projektben regisztrálandó Android csomagnév: hu.beeaibuild.kontigps.staging. A staging webszolgáltatás változói:
- FIREBASE_ANDROID_CONFIG_JSON: {"apiKey":"…","applicationId":"1:…:android:…","projectId":"…","senderId":"…"} — a google-services.json nyilvános Android-konfigurációjából.
- FIREBASE_SERVICE_ACCOUNT_JSON: a Firebase Cloud Messaging HTTP v1 küldésre jogosult service account JSON-ja. Titok; kizárólag Render secret, soha nem kerül APK-ba/Gitbe.
- ENABLE_DRIVER_PUSH=true. A Cloud Messaging HTTP v1 API legyen engedélyezett.

A két konfiguráció projectId értéke egyezzen. A Routes/Geocoding-kulcsok nem helyettesítik ezeket. Az APK futáskor tölti le a nyilvános Firebase-beállítást; a szerverkulcsot soha. Konfiguráció nélkül a belépés/GPS működhet, de a feladatértesítés nem tekinthető késznek és ezt az app/admin kiírja. Értesítés: data-only high-priority FCM, natív hangos csatorna, sessionhez kötött lejárat és helyi ismétlésvédelem. Kijelentkezett/másik sofőr régi értesítése nem jelenhet meg. Az outbox retry miatt az átvitel legalább egyszeri; az eszköz eventId szerint szűri az ismétlést. A rendszer/One UI engedélyei és hálózata a tényleges kézbesítést befolyásolhatják, ezért lezárt képernyős fizikai átvétel kötelező.

## Tartós autós tablet-kötés (2026-11-09)

A tablet–rendszám kötés kézi adminisztrátori leválasztásig él, a már bekötött tableteknél is.
A DriverTablet.expiresAt korábbi értéke nem korlátozza a belépést, GPS-állapotot vagy push értesítést;
az oszlop és a bind-válasz mezője kompatibilitásból megmarad. Új kötésnél 9999-12-31 az értéke.
Nincs adatbázis-migráció és nincs szükség APK-cserére vagy újrapárosításra.
A külön GPS-párosítási hozzáférés továbbra is 90 napos; a napi sofőr-session magyar idő szerint éjfélkor lejár.
Az inaktív autó, visszavont kötés, illetve visszavont sofőrjogosultság továbbra is kizárja a használatot.


## Tablet-only shifts — 2026-11-11

New GPS shifts require a logged-in driver-app device bound to the truck tablet.
Legacy phone pairing credentials and tablet setup credentials return
LOGIN_REQUIRED from shift start. Existing legacy phone shifts are no longer
active authority and are closed during the next tablet login, freeing the truck.
Pairing remains an enrollment step; it is not driver login.
