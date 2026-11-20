# Viber-benchmark — reprodukálható valós modellfuttatás protokollja

**Cél:** egyetlen, szabályosan végrehajtott valós providerfutás, amelynek eredményét bárki **API-kulcs és új modellhívás nélkül** újrapontozhatja a mentett artefaktumokból.
**Nem cél:** modellminőség vagy statisztikai érvényesség igazolása. Egy futás a mérési folyamat működését bizonyítja, nem a modell megbízhatóságát.
**Háttér:** a [PR #57](https://github.com/beeaibuild/kontener-app/pull/57) hét valós futást jelentett, de a nyers artefaktumok a futtatókörnyezettel együtt elvesztek, így a jelentett eredmények bizonyíthatósága bukott el. Az nem bizonyított, hogy maga a futtatás hibás volt. Ez a protokoll ezt a rést zárja.

## Kapuk (sorrendben; egyik sem hagyható ki)

1. **Adatvédelmi ellenőrzés.** A 20 eset (`viberBenchmarkV1.ts`), az orákulum és a várható kimenetek átnézése személyes adatra (név, telefonszám, cím, rendszám, cégadat). Eredmény: „megosztható” / „anonimizálandó” esetenként, dátummal. Amíg ez nincs lezárva, **nincs futtatás**. Az esetek valós Viber-üzenetekből készültek, ezért **valós providerhez csak jóváhagyott, álnevesített bemenet mehet** ([PR #147](https://github.com/beeaibuild/kontener-app/pull/147)); a megfeleltetési táblázat nem kerül a repóba és az artefaktumcsomagba. Korábbi adatkezelési esemény: a 2026.09.14-i két kampány (és a #57-ben jelentett futások) még az eredeti, nem álnevesített esetekkel hívták az OpenAI API-t. Ezt rögzítjük; jogi minősítéséhez az adatkezelési szerepek és a jogalap tisztázása szükséges (az ügyféladatok a partnercég adatkezelésében vannak).
2. **Futtatási manifest.** A futtató `manifest.json` fájlja a kezdési időt, Node-verziót, módot, modellt, hőmérsékletet és a forrásfájlok, package és lockfile hash-eit automatikusan rögzíti. A további adatokat a futás előtt külön `operator-manifest.json` fájlban kell rögzíteni:
   - commit SHA (tiszta munkafa, `git status` üres);
   - provider, pontos modellazonosító dátumverzióval, hőmérséklet és minden további paraméter;
   - a gépi forráslenyomat hivatkozása; policy: `konti-dispatch-policy-v1`;
   - telepített csomagverziók (például `npm ls --depth=0` kimenete);
   - futtató személy, idő (Europe/Budapest), engedély hivatkozása.
3. **Egyetlen valós futás** a `docs/konti-viber-eval-safety-v2.md` szerinti, külön engedélyezett, nem production környezetben. Mentendő minden esetre: a modellnek küldött pontos bemenet, a szűrt provider-JSON (kulcs és fejlécek nélkül), a validált predikció, valamint hiba esetén a hibakód és a szakasz. Pótfutás nincs; a provider hibája eredményként rögzítendő. Transport-, refusal- és JSON-dekódolási hibánál csak általános hibakód mentődik, kivételszöveg és teljes HTTP-válasz nélkül. A `raw/` nem bájtpontos HTTP-archívum.
4. **Független újrapontozás** kizárólag a mentett predikciókból: `--scripted <predictions.json>`, hálózat és API-kulcs nélkül, lehetőleg másik gépen vagy tiszta klónon.
5. **Egyezőség ellenőrzése.** Az újrapontozott aggregátumnak, a komponenspontszámoknak és a kritikus hibák listájának egyeznie kell az eredeti riporttal. Eltérés = a futás **nem igazolt**.

## Elkészült kódbeli feltétel és offline ellenőrzés

A [#147](https://github.com/beeaibuild/kontener-app/pull/147) álnevesítése és a [#151](https://github.com/beeaibuild/kontener-app/pull/151) exportjavítása mainbe került. A futtató `--artifacts <új könyvtár>` kapcsolóval külön exportcsomagot készít. Valós módban export nélkül nem indul: ha csak `--output report.json` szerepel, az export a `report.json.artifacts/` könyvtárba kerül. Offline módban a `--output` továbbra is csak riportot ment. Meglévő exportkönyvtár nem írható felül.

A csomag: `manifest.json`, `inputs/`, `raw/`, `predictions.json`, `report.json`, `SHA256SUMS`. Az esetazonosító, futásazonosító és bemenethash összeköti az esetenkénti fájlokat. A valós futás pontos HTTP body-ja az `inputs/` fájlokban megmarad; fejlécek és kulcsok nem. A hibás esetek is visszajátszhatók. A titokszűrés nem általános személyesadat-anonimizálás: az 1. kaput nem helyettesíti. Részletek: [viber-eval-artifacts.md](viber-eval-artifacts.md).

A következő próba kizárólag offline orákulumot használ. Ez az export és az újrapontozás működését igazolja, nem a modell minőségét. A `tmp/` szülőkönyvtár létezzen, a célkönyvtár még ne:

```sh
mkdir -p tmp
OPENAI_API_KEY= KONTI_EVAL_REAL_PROVIDER=false node --import tsx scripts/eval-konti-viber-v1.ts \
  --artifacts tmp/viber-review.artifacts > tmp/original-report.json
OPENAI_API_KEY= KONTI_EVAL_REAL_PROVIDER=false node --import tsx scripts/eval-konti-viber-v1.ts \
  --scripted tmp/viber-review.artifacts/predictions.json --output tmp/rescore-report.json
(cd tmp/viber-review.artifacts && sha256sum -c SHA256SUMS)
```

Az egyezőségellenőrzés a futás időpontját, provider/model metaadatait és a csak archivált riportban szereplő `runId` mezőt hagyja figyelmen kívül; az összes pontozási és esetenkénti mezőt összeveti:

```sh
python3 - <<'PYTHON'
import json
from pathlib import Path

def score(path):
    report = json.loads(Path(path).read_text())
    for field in ('generatedAt', 'providerMode', 'provider', 'model', 'runId'):
        report.pop(field, None)
    return report

original = score('tmp/viber-review.artifacts/report.json')
replayed = score('tmp/rescore-report.json')
assert len(original['caseResults']) == len(replayed['caseResults']) == 20
assert original == replayed, 'Eltérő újrapontozás: a futás nem igazolt'
print('PASS: mind a 20 eset teljes pontozása egyezik')
PYTHON
```

A benchmark nem tökéletes pontozásnál vagy pontozhatatlan esetnél 1-es kilépési kódot ad, de a riport és az export ettől még elkészülhet. Ez mérési eredmény, nem önmagában infrastruktúrahiba; az archiválás és az egyezőségellenőrzés ilyenkor is szükséges. A hiányzó vagy hiányos export viszont nem igazol futást.

A bemenetek változása, hibás sorrend vagy sérült csomag esetén a visszajátszás meghiúsul. A regressziós teszt ezt, a providerhibák megőrzését és a titokszűrést is ellenőrzi: `npx vitest run tests/unit/aiInterpretationEvalArtifacts.test.ts`. Ez a teszt külön CLI-folyamatban, üres kulccsal és hibára állított `fetch`-csel is lefut.

## Összegzés tartalma

Az összegzés **mind a 20 esetre** írja ki az esetenkénti pontozhatóságot, átlagot és kritikus hibák számát, ne csak kiemelt esetekre. A 2026.09.14-i második kampányban a 10 kritikus hibából 7 a három kiemelt eseten kívül történt, és utólag nem azonosítható. A biztonsági mutatókat (authority safety, tiltott akciók elkerülése, kritikus hibák) mindig a recall és a domain-ref pontosság mellett kell közölni, mert alacsony lefedettség mellett a kevés kritikus hiba nem jelent biztonságos viselkedést.

## Összehasonlító futás

Két verzió összevetésénél egyszerre csak egy tényező változhat: vagy a pontozó, vagy a modellnek átadott tartalom (policy, provider, szemantikus szerződés). Pontozóváltozás hatását a korábbi futás mentett predikcióinak újrapontozásával kell mérni, új modellhívás nélkül. A 2026.09.14-i két kampány (run [34809221837](https://github.com/beeaibuild/kontener-app/actions/runs/34809221837) és [34832401936](https://github.com/beeaibuild/kontener-app/actions/runs/34832401936)) között mindkettő változott (PR #60), ezért a 0 → 10 kritikus hiba nem értelmezhető.

## Tárolás

A [PR #59](https://github.com/beeaibuild/kontener-app/pull/59) workflow-ja (`slice5a-real-provider-benchmark.yml`) történeti hivatkozás, nem használható változtatás nélkül az új protokollhoz: korábbi commitot rögzít, és a valós futásnak külön biztonsági korlátai vannak. A 14 napos `retention-days` miatt a 2026.09.14-i nyers artefaktumok elvesztek. A sikeres futás és feltöltés önmagában nem tartós auditálhatóság: a csomagot a futás után azonnal releasehez kell csatolni.

Verziózott artefaktumcsomag egy GitHub-release mellékleteként (`viber-eval-run-ÉÉÉÉHHNN.zip`): `manifest.json`, `operator-manifest.json`, `inputs/`, `raw/`, `predictions.json`, `report.json`, `rescore-report.json`, `SHA256SUMS`. A futtató ellenőrzőösszegei a gépileg létrehozott fájlokra vonatkoznak; az operátori manifest és az újrapontozott riport archiválásakor ezek hash-eit is hozzá kell adni. A repóba nyers modellkimenet nem kerül. Ha az 1. kapu szerint nem minden eset osztható meg, a csomag privát marad, és csak a hash-ek kerülnek nyilvános helyre.

## Lezárás

A futás akkor „igazolt”, ha mind az öt kapu teljesült. Bejegyzés a `DEVELOPMENT-LOG.md`-be a szokásos sablonnal: mit, miért, bizonyíték (release-link, `SHA256SUMS`, egyezőségi eredmény), nyitott (pl. „egy futás, nem modellteljesítmény”).
