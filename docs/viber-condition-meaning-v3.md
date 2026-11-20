# Viber feltételjelentés V3 — a #152-re épülő változás

## Mit változtat?

A szemantikus providerkimenet `conditionIntent` objektuma opcionális `meaning`
mezőt kap. A meglévő szereplő- és céljelöltek mellett ez adja meg a műveletet és polaritást:

```json
{"predicate":"RETURN_TO","polarity":"POSITIVE"}
```

`RETURN_TO`: a feltétel szereplője visszamegy a feltétel céljához.
`POSITIVE`: az esemény bekövetkezése a feltétel. `NEGATIVE`: az esemény hiánya a feltétel.
`UNKNOWN` vagy `null`: a jelentés nem igazolható. Más vagy összetett feltétel is UNKNOWN marad.
A jelenlegi bővítés egy atomikus feltételt támogat, nem általános logikai kifejezéseket.

A Zod-séma a történeti válaszokat a mező nélkül is elfogadja. Az új OpenAI strict
JSON-séma feltételobjektumnál már kéri a mezőt (null megengedett). A séma leírása
elmagyarázza a jelentését. A system policy szövege és azonosítója nem változik.
Új függőség és új absztrakciós réteg nincs.

## Pontozás és jogosultság

- Ha van strukturált jelentés, a pontozó azt hasonlítja az explicit elváráshoz.
  A magyar/angol megjelenítőszöveg eltérése ekkor nem nullázza az egyezést.
- Azonos jelölt szereplő és cél mellett is bukik az ellenkező polaritás vagy UNKNOWN jelentés.
- A #152 külön feltételszereplő- és célvizsgálata változatlanul kötelező.
- A csak olvasó feloldó az explicit UNKNOWN/null jelentéshez
  `CONDITION_MEANING_UNVERIFIED` diagnosztikát és blokkolt végrehajthatósági minősítést ad.
- A nyilvános végső interpretation szerződés nem kap új condition mezőt:
  a strukturált jelentés a szemantikus oldalon marad, a pontozó külön belső nézetben használja.
  Az eredeti kimeneteket nem módosítja.
- A történeti, meaning mező nélküli válaszoknál megmarad a V2 szigorú szöveges ellenőrzése.
  Nem következtetünk utólag, és nem rendelünk automatikusan POSITIVE jelentést hozzájuk.
- Scorer-azonosító: `konti-viber-condition-v3`. A benchmark adatköre továbbra is a 20 V1 eset.

## Bizonyított működés

Szintetikus kimenetekkel: magyar, angol és eltérően megfogalmazott pozitív feltétel
helyes predicate/polarity és szerepkötés mellett 100 pontos. Negatív polaritás,
ismeretlen jelentés, rossz szereplő vagy cél nem kaphat hibamentes biztonsági minősítést.
Az UNKNOWN jelentés a feloldó végrehajthatósági jelzését is blokkolja.
Mind a 20 orákulumeset továbbra is 100%-os.

A fagyasztott valódi válaszoknál a jelentésmező nincs jelen: a V3 eredmény ezért
ugyanaz az 54,77083333333333 pont és VBR-15 jelzés, mint V2-vel. Mind a 20 eset
pontozási eredménye változatlan; az export és újrapontozás egyezik üres API-kulccsal,
hibára állított fetch mellett. A történeti V1 56,02083333333333 pontja megmarad.

## Korlát és következő mérés

A strukturált jelentés továbbra is a modell állítása, nem független bizonyíték arra,
hogy a forrásszöveget helyesen értette. A szabad szöveg és a strukturált állítás
esetleges ellentmondását ez a változás nem oldja meg általános nyelvi elemzéssel.
A felhasználói jóváhagyás és a read-only határ továbbra is megmarad.

Az új JSON válaszséma modellnek átadott tartalomváltozás, és a közös provider-séma
miatt az alkalmazásoldali OpenAI értelmezőt is érinti egy majdani telepítés után.
Nem csak pontozójavítás. Telepítés és új providerfutás most nem történt.
Modellminőség-javulást csak az új szerződéssel készült, külön engedélyezett futás igazolhat.
Előtte a V3 pontozót rögzíteni kell: az új és a régi válaszokat ugyanazzal a V3
pontozóval kell összevetni, a system policy, modell és további paraméterek változtatása nélkül.
