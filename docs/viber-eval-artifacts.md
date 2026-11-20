# Viber eval: mentés és offline újrapontozás

A #146 protokoll kódbeli exportfeltételét valós modellhívás nélkül megvalósító javítás.
Alap: #147 álnevesített esetei. Az adatvédelmi kapu és a valós futás külön engedélye továbbra is szükséges.

## Offline próba

```sh
node --import tsx scripts/eval-konti-viber-v1.ts --artifacts tmp/viber-review.artifacts
node --import tsx scripts/eval-konti-viber-v1.ts --scripted tmp/viber-review.artifacts/predictions.json --output tmp/rescore-report.json
```

Mindkét parancs API-kulcs nélkül fut; a `KONTI_EVAL_REAL_PROVIDER` ne legyen `true`.
Az első parancs orákulumot használ: a 100% itt az export/visszajátszás működését bizonyítja, nem modellminőséget.

## Export

`--artifacts <új könyvtár>` külön csomagot készít. Valós módban kötelező az export:
ha csak `--output report.json` szerepel, a csomag `report.json.artifacts/` alá kerül.
Offline módban a `--output` megőrzi a korábbi, csak riportot mentő viselkedést.
A célkönyvtárat a futtató a providerhívások előtt foglalja le; meglévő csomagot nem ír felül.

| Fájl | Tartalom |
| --- | --- |
| `manifest.json` | Kezdési idő, Node, mód, modell, hőmérséklet, az értelmező források, CLI, package és lockfile SHA-256 értékei |
| `inputs/VBR-XX.json` | Eset- és futásazonosító, bemenethash, pontos provider-request; valós módban a tényleges HTTP body is |
| `raw/VBR-XX.json` | Szűrt válasz, providereredmény, esetleges hibaszakasz és hibakód; fejléc és kulcs nélkül |
| `predictions.json` | Verziózott, mind a 20 esetet tartalmazó visszajátszható csomag: bemenet, szűrt kimenet, validált predikció, hibás esetek |
| `report.json` | Esetenkénti és aggregált pontozás; prompt és bemenet nélkül |
| `SHA256SUMS` | A csomag összes fenti fájljának ellenőrzőösszege |

A `raw` a provider által visszaadott JSON-t jelenti, nem a teljes HTTP-választ. Transport-, refusal- vagy JSON-dekódolási hibánál csak általános providerhiba mentődik; nem tárolunk kivételszöveget vagy HTTP-fejléceket. Sémában hibás JSON-objektum megőrződik és visszajátszható.

A predikció a szemantikus szerződésre validált modellkimenet; régi scripted bemenetnél a legacy szerződés. Az újrapontozás újra lefuttatja a determinisztikus feloldást és a pontozást, a mentett pontszámokat nem használja. A providerhibák is eredményként játszódnak vissza. Hiányzó, átrendezett, sérült vagy az aktuális bemenettel nem egyező csomag esetén a CLI meghiúsul, és nem ír hitelesnek látszó újrapontozott riportot.

A szűrés eltávolítja a credential jellegű mezőket, az ismert API-kulcsot és az `sk-…` mintát; a pontozás ugyanazt a szűrt kimenetet kapja, amelyet mentünk. Credentialt tartalmazó bemenet exportja meghiúsul. Ez nem általános személyesadat-anonimizáló: csak a jóváhagyott, álnevesített fixture használható. A csomag privát (könyvtár 0700, fájlok 0600), és nem kerülhet a repóba. Az alapértelmezett `.artifacts/` könyvtárak gitignore alatt vannak; egyedi cél megadásakor a futtató felel a privát helyért.

A manifest gépi forráslenyomatot ad. A #146 szerinti tiszta git commit, futtató személy, engedélyhivatkozás, csomagverziók és tartós, megfelelő hozzáférésű archiválás továbbra is a futtatási protokoll része. A hash integritást ellenőriz, nem digitális aláírás.

## Regresszió

```sh
npx vitest run tests/unit/aiInterpretationEvalArtifacts.test.ts
```

A teszt mind a 20 eset teljes pontozását összeveti, hibás válaszokat és providerhibát is visszajátszik, ellenőrzi a titokszűrést és a sérült csomag elutasítását. A CLI-t külön folyamatban is futtatja üres kulccsal és hibára állított `fetch`-csel. A meglévő legacy scripted tömb továbbra is elfogadott.
