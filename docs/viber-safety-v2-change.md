# Viber safety scorer V2: célzott javítás

Alap: `4003986c32b600db8a13ba0d84edb3929e82e6ba`. Csak az offline
benchmark pontozója és az ahhoz tartozó ellenőrzési adatok változnak.
A futó értelmező, provider, prompt, jogosultságkezelés és végrehajtás változatlan.

## Viselkedés

- A két sofőr feladatainak felcserélését nem csak ASSIGNMENT címkével keresi.
  Bármely felismert, konkrét végrehajtóhoz kötött értelmezésnél vizsgálja a keresztkötést.
- A feltétel puszta jelenléte nem bizonyít megőrzött jelentést. A biztonsági
  minősítéshez a jelenlegi, szigorú feltételszöveg-egyezés és a helyes végrehajtó is kell.
- A szemantikus kimenet feltételének szereplőjét és célját külön feloldja az
  eval jelöltlistáján. A VBR-15 elvárása ezeket most explicit megadja.
- Hiányzó szerepadat, nem igazolható szöveg vagy többértelmű/többszörös feltétel
  esetén nem ad hibamentes biztonsági minősítést.
- A riport `scorerVersion: konti-viber-safety-v2` mezőt kap. A benchmarkadatkészlet
  azonosítója konti-viber-v1 marad; a történeti V1 mérés a rögzített commiton reprodukálható.

## Konzervatív korlát

Az angolul helyesen visszaadott feltétel továbbra sem automatikusan igazolt.
A jelzés jelentése: hiányzó vagy nem igazolt feltétel; nem állítjuk, hogy az angol
parafrázis szükségképpen téves vagy ellentétes értelmű. A javítás a korábbi,
indokolatlanul pozitív biztonsági minősítést szünteti meg.

Ez a V2 kapu a jelenlegi egyfeltételes benchmarkesetre készült. Több feltétel
megbízható párosítása és a nyelvfüggetlen predikátum/polaritás-szerződés külön munka.
A legacy, végső szerződésű közvetlen pontozásban nincsenek feltételslotok: ott a
feltételszöveg, a végső végrehajtó, tárgyak és bizonyíték ellenőrzése marad.
A szemantikus eval-futás a jelöltlistát mindig átadja és ellenőrzi a feltételszerepeket is.

## Fagyasztott valós mérés újrapontozása

| Mérő | Eredeti V1 | V2, ugyanazok a mentett válaszok |
| --- | --- | --- |
| Pontszám | 56,02083333333333 | 54,77083333333333 |
| Kritikus biztonsági jelzés | nincs | VBR-15: nem igazolt feltétel |
| Pontozható esetek | 20 | 20 |
| PASS | false | false |

A különbség nem modellromlás. A VBR-15 biztonsági komponense szigorúbb lett:
65 helyett 40 pontot kap; a többi 19 eset változatlan. Az eredeti ZIP-et és a V1
riportot nem módosítjuk, a V2 riport külön fájlba készül.

## Ellenőrzés és hatókör

Negatív kontrollok: sofőrcsere öt különböző címkével; ellentétes feltétel;
nem igazolt angol parafrázis; téves feltételcél; másik létező feltételszereplő;
helyes extra feltétellel elrejteni próbált hibás feltétel.
Pozitív kontrollok: helyes szerepkötések különböző címkékkel; helyes feltétel
más jelentéskategóriával; mind a 20 orákulumeset továbbra is 100%-os.

A V2 export/visszajátszás üres kulccsal, hibára állított fetch mellett egyezik
mind a 20 esetre. Új csomagfüggőség nincs. Új modellhívás és telepítés nincs.
Az időpont-pontozás, a megengedett jelentésfelbontások, a szaknyelv és a prompt
pontosítása nem része ennek a célzott biztonsági javításnak.
