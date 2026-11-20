# Nyolc szintetikus Viber-kontroll — v1

Az eredeti értékelés megmarad alapértelmezésként. A mért válaszokból feltárt
értékelési korlátokat külön, választható [V2 audit](viber-controls-v2.md) kezeli.

Definíció: `src/services/ai/interpretation/viberControlsV1.ts`, K1–K8.
A mondatok az összevont reviewban előre rögzített példák. Nem új ügyféladatok,
nem független valós holdout, és nem részei a fagyasztott 20 esetes V1-nek.
Provider-séma, policy és V1-pontozó ebben a változásban nem módosul.

## Rögzített értékelés

| Eset | Automatikusan ellenőrzött kötés és jogosultság | Külön emberi jelentésellenőrzés |
| --- | --- | --- |
| K1 | Réka + Ibolya kiosztása, csak jóváhagyás után | Hatköbös = 6 m³; nincs kitalált hulladéktípus |
| K2 | Ibolya készjelentése Rékától, nem végrehajtási utasítás | Konténercsere, nem műszakváltás; nincs kitalált teli/üres állapot |
| K3 | Ibolya → Tölgyes sorrend, csak jóváhagyás után | Nincs kitalált időpont |
| K4 | Réka felajánlása a Tölgyesre, csak javaslat | Nem elfogadott kiosztás |
| K5 | Ádám + Tölgyes kiosztása; Réka nem maradhat végrehajtó | Sofőrcsere, nem konténercsere |
| K6 | Réka saját ismert teherautójának tervezett járműművelete | Jövőbeli gumizás, nem készjelentés vagy fuvar |
| K7 | Kérdés; feloldatlan „ezt”; nincs kitalált tárgy | Nem helyettesíthető ismert személlyel |
| K8 | Csak társalgás, nincs üzleti szereplő/tárgy/jelzés | Nincs burkolt üzleti állítás |

Minden esetnél kötelező a jelen üzenet bizonyítéka, a helyes beszédaktus és jogosultság,
az automatikus írás tiltása. Idegen tárgy, szereplő, jelentéstípus vagy kapcsolat hibát ad.
Nincs megkövetelt rekordszám: a méret az összevont kiosztás szövegében is szerepelhet;
K6-nál a járműművelet és a terv külön rekordban is megengedett.
K7-hez nincs ismert feladat, de az ismert beszélő jelen van. Más esetekben a megadott
jelöltek és egyértelmű címkék biztosítják a feloldáshoz szükséges kontextust.
A mintaválasz és az elvárások soha nem kerülnek a provider bemenetébe.

## Offline használat

```powershell
$env:KONTI_EVAL_REAL_PROVIDER = "false"
node --import tsx scripts/eval-konti-viber-v1.ts --suite controls `
  --artifacts "tmp/control-offline.artifacts" --output "tmp/control-offline.json"

node --import tsx scripts/eval-konti-viber-v1.ts --suite controls `
  --scripted "tmp/control-offline.artifacts/predictions.json" --output "tmp/control-replay.json"
```

A `tmp` szülőmappa legyen meg; az artifact célmappa új legyen. Meglévő mérést nem ír felül.
Az első parancs beépített scripted válaszokat használ, nem modellt. API-kulcs nem szükséges.
V1 továbbra is a kapcsoló nélküli alapértelmezés; a másik suite csomagját visszautasítja.

## Riport és export

Azonosító: `konti-viber-controls-v1`; ellenőrzési verzió: `structured-controls-v1`.
Az `automaticChecksPassed` 0–8 közötti esetszám. `passed=true` csak azt jelenti, hogy
mind a nyolc strukturális kontroll sikeres; **nem jelenti az emberi jelentésreview sikerét**.
Az `evaluationScope=STRUCTURED_CHECKS_ONLY` és `manualReviewStatus=PENDING` ezt jelzi.
Nincs V1-es százalékpontszám; a két sorozat eredményét nem átlagoljuk össze.
A pontozó V1-verziója változatlan; a kontrollok külön tartalmi kapuk, nem új V1-pontozó.

Minden esethez mentjük a pontos providerbemenetet, inputHash-t, szűrt választ és
visszajátszható predikciót. A futás runId-t, a riport a definíció egészének SHA-256
azonosítóját kapja; a manifest a forrásfájlokat hash-eli. Valós mód esetén az exact HTTP
kérés törzse is megmarad, credential/header nélkül. SHA256SUMS ellenőrzi az exportfájlokat.
A `--scripted` futás a suite-ot, az esetsorrendet, a bemenethash-t és a mentett
válasz/predikció egyezését ellenőrzi. Nyers csomag és kulcs nem kerül GitHubra.

Új valós futás ebben a körben nem történt. Következő modellmérés előtt a változatot
commit SHA-val kell rögzíteni; a meglévő modellt (`gpt-4.1-2025-04-14`, temperature 0)
és az exportprotokollt kell használni, külön jóváhagyott futásban. A modellválaszok
alapján az elvárásokat utólag nem módosítjuk; a jelentésreview eredményét külön rögzítjük.

## Offline regresszió

243 teszt sikeres 13 célzott fájlban, köztük 11 új kontrollteszt. Az actual CLI export
19 ellenőrzőösszege helyes; a nyolc teljes eredmény kulcs nélkül, tiltott fetch mellett
azonosan visszajátszható. Hibás és szolgáltatói hibás válasz is megőrződik; a suite-ok
felcserélése hibát ad. A legutóbbi valós V1 teljes riportja változatlan (60,666667 pont).
Célzott strict TypeScript ellenőrzés sikeres. A teljes alkalmazás ellenőrzése PR CI-feladat;
a helyi részleges checkoutban az alkalmazásfüggő tesztek és a teljes build nem igazoltak.
