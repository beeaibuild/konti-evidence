# Személyes Konti rendelésfelvétel — v1

Fejlesztési állapot, 2026.11.13. Nem élesített képesség.

## Használat

A Kézi rendelés melletti mikrofon felvételéből csak leirat készül. A leirat felhasználóhoz kötött,
egyszeri, tíz percig érvényes böngészős átadással kerül a kézi rendelési oldalra, ahol küldés előtt javítható.
A gépelt szöveg és a leirat ugyanabba a személyes Konti-rendelésfelvételi végpontba kerül.
A chatből érkező rendelési kéréseket egy routing-jelzés ugyanehhez az AI-kezelőhöz irányítja;
a routing nem jelent üzleti döntést. Kérdés vagy többértelmű kérés esetén az AI-nak pontosítást kell kérnie.

A Konti strukturált mezőjavaslatot ad forrásüzenet-azonosítóval és szó szerinti bizonyítékkal.
A modell a meglévő KONTI_AI_MODEL és AI adapter alapján fut; nincs külön hangos mezőkinyerő modell.
A hangátiratot a meglévő, környezeti változóval felülírható transzkripciós beállítás készíti.

Az űrlap a külön áttöltés után zölddel jelöli a szövegből kinyert értéket, kékkel a kontextuskiegészítést,
pirossal az üres, szükséges mezőt. Szöveges jelölés is van; a szín önmagában nem információforrás.
Minden előtöltött mező szerkeszthető. Szerkesztett űrlap cseréje megerősítést kér. Nincs automatikus mentés.

## Adat és hozzáférés

- A személyes szál kulcsa `PLANNING:user:<userId>`. A normál chatben is látszik az érintett felhasználónak.
- A meglévő közös operátori PLANNING-szál változatlanul megmarad; a személyes szál attól különálló.
- A szerver nem csak tagságot, hanem scope-tulajdont is vizsgál. Más operátor, sofőr és tévesen felvett résztvevő sem olvashatja.
- A személyes szál résztvevőlistája alkalmazásból nem szerkeszthető. A feed és a jogosultsági epoch ugyanazt a kört használja.
- Friss OPERATOR jogosultság, same-origin védelem és 6 kérés/10 perc/felhasználó limit szükséges.
- Legfeljebb 3900 karakteres leirat/szöveg, 32 KiB tényleges JSON stream, legfeljebb 90 másodperc/6 MiB hang.
- Automatikus fizetős újrapróbálás nincs. A meglévő tartós provider-attempt kezeli az ismételt vagy bizonytalan eredményű kérést.
- A modell csak a személyes emberi szövegelőzményt kapja, korlátozott méretben. A partner-/címtörzs teljes tömege nem kerül hozzá.
- A partner és a cím feloldása szerveroldali, korlátos olvasás. Részleges jelöltkészletből nincs biztos rekordkötés.
- A szükséges dátum/méret/ügyfél hiányát nem fedjük el mai dátummal vagy első konténertípussal.

## Javítások és későbbi kontextus

A meglévő AiOutputAudit tárolja a forrást, az eredeti hangátiratot (ha volt), a küldött szöveget,
a promptot, a verziót/commitot, a modellbeállítást, a mezőjavaslatot és a tokenhasználatot.
A nyers hangot nem archiváljuk. Ezért a hangfelismerés teljes újrafuttatása hangarchívumból nem áll rendelkezésre.

A sikeres rendelésmentés ugyanabban az adatbázis-tranzakcióban új, append-only AiOutputFeedback eseményt rögzít:
mezőnként javasolt érték, végleges érték, eredet és UNCHANGED/CORRECTED/ADDED eredmény.
A félbehagyott szerkesztés, a sikertelen mentés és a visszautasított áttöltés nem tanítási példa.
Egy vázlat egyszer menthető; ismételt mentés esetén a teljes üzleti tranzakció visszagördül.
Meglévő partnernél a rendeléshez megadott javítás a rendelés ügyfélpillanatképére kerül, a partner törzsadatát nem írja át automatikusan.

Az automatikus kiegészítés a felhasználó legutóbbi 50 mentési visszajelzéséből keres.
Csak azonos partner ÉS pontos cím mellett, két külön rendelésben egyező legutóbbi kapcsolati adat használható fel.
Az explicit szöveg elsőbbséget élvez, ellentmondásnál nincs tanult felülírás. Egyetlen példa nem lesz általános szabály.
Ez visszakereshető kontextus, nem modellátképzés. A meglévő, explicit audit-retenció 90 napos korlátja erre is vonatkozik.

## Csere, ürítés, elszállítás

Ezek nem tölthetők új rendelésként a kézi űrlapra. A rendszer a feloldott meglévő rendeléshez vezet.
A feladatűrlap csak az ahhoz a rendeléshez és aktuálisan kiosztható művelethez illeszkedő vázlatból tölt dátumot/megjegyzést.
A sofőr, teherautó és cserekonténer választása, valamint a mentés továbbra is kézi és a meglévő szerverellenőrzésekhez kötött.
Eltérő konténertípusnál nincs automatikus feladat-előtöltés. A sikeres feladatmentés visszajelzése külön eseményadatot tartalmaz;
az nem válik új ügyfélkapcsolati tanítási példává.

## Migráció és ellenőrzés

`20261113100000_order_capture_audit_kinds` csak az audit/visszajelzés engedélyezett eseménytípusait bővíti.
Új tábla, függőség, előfizetés-, workflow- vagy költségkeret-módosítás nincs. Éles migráció nem futott.

Helyi bizonyíték: típusellenőrzés, sorhossz-kapu, teljes unit-készlet és production build;
külön adatbázisos integráció: privát láthatóság, egyszeri providerhívás, mentés+visszajelzés atomikussága, újrafelhasználás tiltása.
A provider ezekben helyettesített: nincs valódi magyar beszédfelismerési/modellminőségi mérés.
A böngészőteszt frissítve, de a helyi Chromium socketkorlátozás miatt nem futott.
A GitHub Actions-keret elfogyott; a CI-kihagyással mentett commit nem tekinthető CI/P63 által ellenőrzöttnek.
Élesítés előtt a végleges headen teljes CI/P63 és fizikai Samsung-/élő hangpróba kell.
