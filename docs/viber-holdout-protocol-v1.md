# Viber tartalékhalmaz — előzetes protokoll

Állapot: DRAFT / NOT_FROZEN. Ez a dokumentum nem jelenti, hogy új tartalékhalmaz már
elkészült vagy általánosítási eredmény rendelkezésre áll. A jelenlegi R01–R10 fejlesztői
halmaz; a promptot ezek ismeretében módosítottuk. A fejlesztői halmazon mért 9/10 nem
általánosítási bizonyíték. A súlyok tanítása nélkül is létrejön promptfejlesztési átfedés.

## Kötelező sorrend

1. A hiányzó kötés diagnosztikáját és a V3 összes jelenleg ismert ellenőrzését lezárjuk,
   ideértve a „3 ked” és az „alólla” strukturált, bizonyítékhoz kötött megőrzését.
2. A promptot, provider-sémát, determinisztikus feloldót, pontozási szabályokat és az
   adatfüggetlen tartalékhalmaz-futtatót egy teljes commit-hash-sel rögzítjük. A futtató
   új eseteket fogadó útját a kiválasztás előtt kell elkészíteni és tesztelni; a jelenlegi
   R08-specifikus V3 ellenőrző önmagában nem általános holdout-pontozó.
   A fagyasztási jegyzékbe kerül a policy.ts, providers.ts, resolution.ts, a pontozó,
   a futtató és e protokoll SHA-256 lenyomata, a modell snapshotja és temperature 0.
   A jelenlegi prompt eredete: #165, main `09123253607df793f52f3a7020e6ae020b5075e6`.
   Ez nem helyettesíti a még hiányzó végleges freezeCommit értékét.
   Az új [futtató](viber-holdout-runner-v1.md) implementációja külön review és CI után fagyasztható.
3. Csak a fagyasztás után választjuk ki az új valódi források 15–20 esetét, és írjuk meg
   a manuális elvárásokat. Egy eset lehet több összefüggő üzenet/reply, nem feltétlenül
   egyetlen üzenet. A teljes szükséges kontextus maradjon meg.
4. Az álnevesített esetcsomag és az előzetes elvárások külön adatcommitot kapnak. Az
   ellenőrzési rubrika futtatás előtt kész; a modell nem kapja meg az elvárt választ.
5. Azonos fagyasztott programmal, modellel, bemenettel és beállításokkal három teljes,
   külön könyvtárba mentett futás készül. Előttük nincs próbafutás az új eseteken.
6. Minden futás szűrt válasza, predikciója, pontos bemenete, azonosítói, forráshash-ei és
   operátori jegyzéke megmarad. Független offline újrapontozás és kézi tartalmi review követi.

A tartalékhalmaz megismerése után nincs prompt-, kód-, rubrika- vagy küszöbhangolás.
Ha bármelyik új eset miatt ilyen változtatás történik, az eset onnantól fejlesztői adat.
Az érintett tartalékhalmaz-változatot visszavonjuk általánosítási mérésből; a többi esetet
nem számoljuk át utólag megtisztított holdoutként. Az eredeti eredmény megmarad auditnak.

## Mintakiválasztás — a modellválaszok előtt

Cél: 18 eset; elfogadható tartomány 15–20, az adatcsomag fagyasztásakor egyetlen N rögzül.
Az alábbi kvóták a kiválasztás előtt rögzített szempontok, nem eredmény alapján cserélhetőek.

| Réteg | Előírt esetszám |
| --- | ---: |
| Főnévi igenévi, rövid utasítás | 3 |
| Személytelen fizetési vagy állapotjelzés | 3, benne mindkét típus |
| Névmásos hely-/tárgyhivatkozás, szükséges kontextussal | 3 |
| A mostani célzott szabályokban nem nevesített egyéb mintázat | N − 9 |

Az egyéb csoportot külön jelentjük, nem rejtjük a célzott kilenc eset átlaga mögé.
Egy eset egy elsődleges réteghez tartozik, nem tölthet ki egyszerre több kvótát.

- Legalább három munkanapból származó anyag; egy napról legfeljebb a csomag fele.
- Legalább két sofőr és legalább három külön küldő; egy küldőtől legfeljebb a csomag fele.
- Lehetőleg olyan napok és sofőrök is legyenek, amelyek az R-esetekben nem szerepeltek.
  A forrásjegyzék rögzíti, mi teljesült; új sofőr nélkül nincs új sofőrökre szóló állítás.
- Nincs átfedés a VBR-, K- vagy R-esetekkel, azok átfogalmazásával vagy ugyanazon
  beszélgetés szomszédos, korábban látott változatával. A névcsere nem új eset.
- A régi, már elemzett screenshotok nem minősíthetők át korábban nem látott forrássá.
- A jogosult forrásokból időrend szerinti kiválasztás történik a kvóták betöltéséig;
  nem a könnyen értelmezhető példákat válogatjuk. Azonos időpontnál forrásfájlnév, majd
  üzenetsorszám a sorrend. A kizárások és indokuk megmaradnak a forrásjegyzékben.
- Hiányos/olvashatatlan forrás kizárható még a modellfuttatás előtt, dokumentáltan.
  Ha egy kvóta nem tölthető be, új forrásanyagot kérünk; nem lazítjuk utólag a kvótát.

A forrásjegyzék dátumot, álneves küldőazonosítót, réteget, eredeti forrás SHA-256-ot,
kontextusablakot és kizárási indokot tartalmaz. A nyers kép és a visszafejtési névtérkép
nem kerül Gitbe vagy a providerhez. Ismeretlen időadatot nem helyettesítünk történeti ténnyel.

## Három ismétlés — előre rögzített eseteredmény

Egy ismétlés egy esete csak akkor PASS, ha értékelhető a válasz, minden kötelező
strukturális/biztonsági ellenőrzés teljesül, és a futás előtt rögzített kézi tartalmi
elvárásokat is teljesíti. PENDING review vagy pontozhatatlan válasz nem PASS.
Az utasításjelzés, szereplő-/tárgykötés, feltétel és hivatkozások hibái külön oszlopok.

| A három futásból sikeres | Végső esetminősítés |
| --- | --- |
| 3/3 | Sikeres |
| 2/3 vagy 1/3 | Ingadozó, nem sikeres |
| 0/3 | Következetesen sikertelen |

Nincs többségi szavazás, legjobb futás választása vagy sikerig ismétlés.
A csomag végső sikeraránya: a 3/3-as esetek száma / N. Az ingadozó esetek száma,
a futásonkénti eredmény és a hibák típusa/üzenetbizonyítéka külön látszik.
A célzott és az egyéb mintázatokra külön összesítés készül.
Három ismétlés ugyanazon N eseten nem 3N független példa.

A három futás külön executionId-t és startedAt értéket kap. A predikciók tartalomalapú
runId-ja egyezhet, ha a válaszok azonosak; ez nem bizonyítja vagy helyettesíti a három
külön futás megtörténtét. A futások minden azonosítója és költsége megmarad.
Szállítási/API-hiba esetén sem választunk csendben másik futást: az eredeti kísérlet
jelölve marad, a batch érvényességét külön vizsgáljuk. Nincs automatikus új modellhívás.

## Előre rögzített értelmezési korlát

Egy-két esetnyi különbség ezen a kis halmazon nem minősül igazolt javulásnak.
Nagyobb eltérésből sem következik általános üzemi pontosság. Egyetlen konfiguráció
három ismétlése stabilitási/hibaképet ad; javulási összehasonlításhoz külön, előre
rögzített összehasonlítási konfiguráció és azonos eljárás kell.
Esetenként összevetjük, melyik hiba tűnt el, keletkezett vagy ingadozott; a változatlan
összesített szám nem fedheti el az R06/R08-hoz hasonló hibacserét.
A modell snapshotja és temperature 0 nem indok az ismétlések elhagyására.

Még nincs kiválasztott vagy lefuttatott új tartalékhalmaz. A korábbi archívum keresése
nem igazolta egy új, a fejlesztési folyamatban korábban nem látott csomag függetlenségét.
A mérés előtt friss, eddig nem elemzett valódi forrásanyag szükséges.
