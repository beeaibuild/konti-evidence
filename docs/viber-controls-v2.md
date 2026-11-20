# Kontrollértékelés V2 — utólagos audit, nem új modellmérés

A 2026.11.05-i, `5389bc8b15f8ca588c03e8f1f2932659e28b10ba` forrású mérés
`gpt-4.1-2025-04-14` modellel, temperature 0 mellett 5/8 strukturális kontrollt teljesített.
Az eredeti `structured-controls-v1` riport, definíció és beépített mintaválasz változatlan.
A V2 kifejezetten a válaszok megismerése után készült értékelőaudit: nem független
holdout, nem javuló modellminőség bizonyítéka. Új providerhívás nem szükséges hozzá.

## Mit választ külön?

Esetenként `componentChecks.content`, `.taxonomy` és `.authority` tárolja a hibákat.
A `componentResults` külön sikeres esetszámot ad a három csoportra. A `checks`,
`automaticChecksPassed` és `passed` továbbra is az összes csoport együttes megfelelését
mutatja. Egy címkézési eltérés tehát nem tűnik el az összesített eredményből.
A `manualReviewStatus=PENDING` megmarad: a szabad jelentésszöveg helyességét ember ellenőrzi.

| Eset | V2 szabály | A megőrzött válasz megítélése |
| --- | --- | --- |
| K1 | A pontos „hatköbös” kifejezés csak Container típusú feloldatlansága megengedett megjegyzés, nem kitalált entitás. Más szereplő-/tárgybizonytalanság továbbra is hiba. | Helyes 6 m³ jelentés, Réka és Ibolya; `SIZE_WITHOUT_CONTAINER_ID` megjegyzés. |
| K3 | A szöveg és a kontextus nem azonosít címzettet. Bármely konkrét szereplőkötés hiba; null/UNKNOWN_ACTOR megengedett. | A sorrend helyes, de Réka kitalált címzett: `UNSUPPORTED_RECIPIENT`. |
| K4 | A helyes szereplővel/tárggyal adott PLAN_OR_INTENT + PLAN tartalmi párosíthatósága elfogadható; a SUGGESTION címkétől/jelzéstől való eltérés külön látszik. | Tartalmi kötés és jogosultság helyes; `KIND_MISMATCH`, `SIGNAL_MISMATCH`. |
| K6 | A saját ismert teherautó tervezett művelete PLAN_OR_INTENT címkével is tartalmilag párosítható; a VEHICLE_OPERATION hiánya külön címkehiba. | Helyes saját autó és terv; `KIND_MISMATCH`. |

A párosítás csak az értékelő belső másolatán történik. Sem a provider nyers válasza,
sem a runtime eredménye vagy jogosultsági döntése nem változik. Téves szereplő/tárgy,
fordított sorrend és nem támogatott beszédaktus továbbra is tartalmi hibát ad.
Az engedélyeket mindig a tényleges, változatlan runtime-kimenet alapján ellenőrizzük.
K3 címzettkitalálása az értelmezési útban továbbra is lehetséges; ez a változtatás felismeri, nem javítja.

## Megőrzött mérés újraértékelése

```powershell
$env:KONTI_EVAL_REAL_PROVIDER = "false"
node --import tsx scripts/eval-konti-viber-v1.ts --suite controls --controls-version v2 `
  --scripted "tmp/viber-controls-20261105-204029/new.artifacts/predictions.json" `
  --output "tmp/controls-v2-audit.json"
```

A `--controls-version` kizárólag a controls suite-hoz használható; alapértelmezése v1.
Ugyanaz a predikciócsomag mindkét értékelés bemenete: esetsorrend, runId és inputHash
azonos marad. A V2 riport és az új export manifestje `structured-controls-v2` verziót jelez.
Új artifact exporthoz külön, még nem létező `--artifacts` célmappát kell megadni;
az eredeti csomagot nem írjuk felül. A futásmanifest a megváltozott értékelő forrását is hash-eli.

| A mentett válaszok eredménye | Sikeres eset |
| --- | --- |
| Eredeti V1, teljes riportegyezéssel újra előállítva | 5/8 |
| V2 strukturális tartalmi ellenőrzés | 7/8 |
| V2 címkézés | 6/8 |
| V2 jogosultság | 8/8 |
| V2 minden csoport együtt | 5/8 |

A ZIP 21 ellenőrzőösszege helyes (a replay riporttal együtt), a forráshash-ek
LF/Windows CRLF különbség figyelembevételével egyeznek. A független CLI-visszajátszás
API-kulcs nélkül, tiltott fetch mellett reprodukálta a teljes eredeti V1-riportot.
K2, K5, K7 és K8 kézi tartalmi átnézése nem talált eltérést. Ebből nem következik
éles használatra kész állapot vagy önálló kiosztási jogosultság.

## Regressziós bizonyíték

253 célzott teszt sikeres 14 fájlban, köztük 10 új V2 kontrollteszt. A célzott strict
TypeScript és diff-ellenőrzés is sikeres. A V2 CLI-export mind a 19 ellenőrzőösszege
helyes, a visszajátszott teljes riport egyezik. A V2 export predikciócsomagja az eredetivel
azonos: a javítás nem módosítja a mentett válaszokat vagy bemeneteket.
A teljes alkalmazásellenőrzés a PR CI feladata. Új modellhívás és telepítés nem történt.
