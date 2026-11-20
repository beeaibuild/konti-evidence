# Tartalékhalmaz-futtató V1

Állapot: implementáció review alatt; végleges fagyasztási commit még nincs.
A futtatóhoz nem választottunk új valódi esetet, és nem hívtunk modellt.
A regressziós teszt csomagja kizárólag generált, szintetikus tesztadat.

## Hatókör

Külön CLI: `scripts/eval-konti-viber-holdout.ts`. Nincs új függőség vagy általános
értékelőkeretrendszer. A meglévő provider, feloldó és pontos bemenethez kötött
predikcióexport szolgálja ki; a korábbi VBR/K/R futtató változatlanul használható.
A prompt és a provider-séma nem változik.

A vizsgálat szöveges, álnevesített Viber-beszélgetések értelmezésére vonatkozik,
üzleti adatbázis-kontextus nélkül. Több küldő és reply támogatott, képek és üzleti
referenciák betöltése nincs. A dátum a mintavételi jegyzékben szerepel; időzóna híján
nem állítunk abszolút eseményidőt. A modell órát és ismeretlen dátumjelzést kap.
A bemeneti forrás és a kimenet a meglévő adatvédelmi szűrésen halad át.
A privacyReviewed csak operátori nyilatkozat: a szűrő nem garantál teljes anonimizálást.

## Sorrend és parancsok

Előbb review, teljes CI és összevonás. Ezután a fagyasztandó commit tiszta checkoutján:

```powershell
$env:KONTI_EVAL_REAL_PROVIDER = "false"
node --import tsx scripts/eval-konti-viber-holdout.ts --prepare-freeze --output tmp/freeze.json
```

A freeze.json teljes commitot, forrásfájl-hasheket, modell snapshotot és temperature 0-t
rögzít. Ezt követően választható ki a 15–20 valódi eset a
[protokoll](viber-holdout-protocol-v1.md) szerint, előzetes manuális elvárásokkal.
A fagyasztás nem igényel API-kulcsot. A jövőbeli modell: `gpt-4.1-2025-04-14`;
a történeti összevethetőség miatt ebben a mérésben nem váltunk más modellre.

A selection.json formátumát a `holdoutSelectionSchema` határozza meg:

- format: konti-viber-holdout-selection-v1; freezeHash: artifactHash(freeze.json).
- selectedAt, expectationsWrittenAt: ISO időpontok ebben a sorrendben, a freeze után.
- privacyReviewed: true; exclusions: locator/reason párok, a kizárásokkal együtt.
- cases: 15–20 elem, id, stratum, source, primaryMessageId, messages, expectations.
- stratum: INFINITIVE / PAYMENT / STATUS / PRONOUN / OTHER; rendre 3 / együtt 3 / 3,
  a maradék OTHER. PAYMENT és STATUS egyaránt szerepeljen.
- source: sha256, day (YYYY-MM-DD), locator, previouslySeen: false.
- messages: id, senderAlias (álnévazonosító), senderRole (OPERATOR/DRIVER), text,
  displayTime (HH:mm), replyToMessageId (azonosító vagy null).
- expectations.signals: minden üzenetre messageId, allowed és required jelzéslista, előre rögzítve.
  A required mindegyike kell; az allowed más jelzést kizár. Vegyes közlésnél például
  az INSTRUCTION kötelező, míg INFORMATION kiegészítés külön megengedhető.
- expectations.noActorMessageIds: azok az üzenetek, amelyekhez nincs igazolt cselekvő.
- expectations.unresolved: messageId és variants szöveglista a megőrzendő részletekhez.
- expectations.manual: id/description listában a tartalmi elvárások, legalább egy.

A kvótákat és hivatkozásokat a futtató ellenőrzi. A kronológiai kiválasztást, a valódi
forrást, a korábbi fejlesztési átfedés hiányát és a kézi elvárások helyességét ember
ellenőrzi és a forrásjegyzék igazolja; ezek nem bizonyíthatók egy JSON állításából.
A source.locator a privát forrásban visszakereshető hely, ne tartalmazzon személyes adatot.
Az elvárások, forráshash-ek és rétegek nem kerülnek a provider bemenetébe.

Új valódi futás külön jóváhagyást igényel. A CLI egy hívása egyetlen teljes csomagot futtat;
nem végez automatikus háromszorozást vagy újrapróbálást. A valós módot a meglévő
KONTI_EVAL_REAL_PROVIDER / KONTI_EVAL_PROVIDER / KONTI_EVAL_MODEL beállítások vezérlik.
Az offline használat az új, korábban még nem használt célkönyvtárba ír:

```powershell
$env:KONTI_EVAL_REAL_PROVIDER = "false"
node --import tsx scripts/eval-konti-viber-holdout.ts `
  --selection tmp/selection.json --freeze tmp/freeze.json `
  --scripted tmp/run-1/predictions.json --artifacts tmp/replay-1
```

A valós futás ugyanez --scripted nélkül, külön engedélyezett valós móddal.
A könyvtárat a futtató a modellhívás előtt foglalja le. Nem ír felül meglévő futást.
Az export: manifest.json, freeze.json, selection.json, report.json, predictions.json,
esetenként inputs és raw JSON, valamint SHA256SUMS. Valós futásnál az inputs a pontos
HTTP törzset is tartalmazza. Az executionId egyedi; a tartalomalapú runId egyezhet.
Az automatikus ellenőrzés kilépési kódja 0/1; a 0 sem tartalmi PASS, a review PENDING.

## Kézi review és összesítés

Minden eredeti riporthoz külön review JSON kell. Mezői: executionId, reportHash
(artifactHash a teljes report.json objektumon), reviewer, reviewedAt és cases.
Esetenként caseId és criteria lista: a kiválasztáskor rögzített id-k mindegyike pontosan
egyszer, verdict PASS/FAIL/PENDING és note. Ez az értékelő nyilatkozata; nem modellbíró.
A review a teljes riport hash-éhez kötődik, más futásra nem vihető át.

Az összesítő bemenete egy JSON: selection és freeze útvonalak, runs listában pontosan
három artifacts/review útvonalpár. Minden útvonal a parancs munkakönyvtárához relatív.

```powershell
node --import tsx scripts/eval-konti-viber-holdout.ts `
  --aggregate tmp/aggregate-input.json --output tmp/aggregate-report.json
```

Az összesítés ellenőrzi a fájlhasheket, a fagyasztott kódot, az azonos esetcsomagot,
modellbeállítást és pontos bemeneteket, majd mindhárom csomagot offline újrapontozza.
Csak külön executionId-jú eredeti valós futások fogadhatók el, scripted másolatok nem.
A jegyzékek bizonyítékláncot adnak, nem kriptográfiai igazolást arról, hány API-hívás történt;
az operátori eredet és a három külön futás ténye is auditálandó.
Csak automatikus és kézi PASS együtt számít. 3/3 PASS a siker; 1/3 vagy 2/3 UNSTABLE,
0/3 FAIL. Minden futás hibája és a célzott/egyéb réteg külön látszik. PENDING nem PASS.
Sem az összesítés, sem a replay nem használ API-kulcsot vagy indít modellhívást.
