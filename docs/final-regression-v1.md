# V1 final regression — demo freeze checklist

Cél: a v1 lezárás előtti, egyben végigkattintható teljes regressziós
kör. Nem helyettesíti a csomagonkénti részletes listákat
(`manual-test-package-3…18.md`), hanem a lezáráshoz szükséges
minimumot fedi le egy ülésben (~45–60 perc). Ha itt minden zöld, a
v1 demózható és átadható.

Jelölés: minden pont önállóan pipálható. A pontok sorrendje szándékos:
egy rendelés-életciklusra fűzi fel az ellenőrzéseket, minimális
újrakattintással.

## 0/A. Automatizált regressziós kapu (P28 után, PowerShell)

A kézi checklist ELŐTT ez a parancssor fut le; bármely eltérés az
elvárt eredménytől blokkoló. Windows / PowerShell:

```powershell
docker compose up -d
npx prisma migrate deploy
npm ci
npx prisma generate
npm run db:seed
npm audit
npm run test
npm run test:e2e
npm run test:e2e:ui
npm run typecheck
npm run build
```

Elvárt eredmények (P28 / next@15.5.20 referenciafutás):

| Parancs | Elvárt |
|---|---|
| `npm audit` | 2 moderate, mindkettő a Next-be ágyazott postcss lánc (dokumentált kivétel) — semmi más |
| `npm run test` | 64/64 passed |
| `npm run test:e2e` | 58/58 passed |
| `npm run test:e2e:ui` | 8/8 passed |
| `npm run typecheck` | 0 hiba |
| `npm run build` | zöld, Next 15.5.20, 24 route, static pages 11/11 |

*Előremenő invariáns-frissítés (a fenti tábla a P29 RC történeti
állapotát rögzíti, azt nem írjuk át): a 33. csomagtól a route-invariáns
24 → 25 (+1 új route: `GET /api/health`), a unit/e2e darabszámok pedig
a P31–P33 csomagokkal nőnek — az aktuális elvárt számokat mindig az
adott csomag doksija adja (P33: `logging-health-p33.md`).*

Jegyzet — workspace-root warning: ha a Next build a
`C:\Users\PC\package-lock.json` miatt workspace-root warningot ír
(a Next a home könyvtárban talált kósza lockfile-t hiszi
monorepo-gyökérnek), az LOKÁLIS KÖRNYEZETI warning, nem app
regresszió. Megoldás: a fölös `C:\Users\PC\package-lock.json`
törlése, vagy később `outputFileTracingRoot` beállítása a
next.config-ban. A kapu szempontjából a warning nem blokkoló.

Jegyzet — mappaszennyezés: a P28 utáni validációnál előfordult, hogy
régi csomagmappák (1/2/3/4/5/6) a projektfa alatt typecheck/build
hibát okoztak. Tiszta fán futtass; gyanú esetén ellenőrizd, hogy a
`tsc` nem lát-e a projekten kívüli forrásokat.

## 0/B. Kézi kör előfeltételei

- [ ] Docker fut, Postgres elindítva: `docker compose up -d`
- [ ] `.env` létrehozva: `cp .env.example .env` (fejlesztéshez a
      defaultok jók)
- [ ] `npm install`
- [ ] `npx prisma generate`
- [ ] `npm run db:reset` — séma + seed hibátlanul lefut
- [ ] `npm run typecheck` — 0 hiba
- [ ] `npm run build` — sikeres
- [ ] `npm run dev` — http://localhost:3000 betölt

Login adatok (csak fejlesztés):

| Szerep | Email | Jelszó |
|---|---|---|
| ADMIN | (fejlesztői seed-fiók, nyilvános másolatból törölve) | — |
| DRIVER | (fejlesztői seed-fiók, nyilvános másolatból törölve) | — |
| DRIVER | (fejlesztői seed-fiók, nyilvános másolatból törölve) | — |

Tipp: admin és sofőr két külön böngészőben (vagy normál + inkognitó).

## 1. Publikus oldal

- [ ] 1.1 Főoldal belépés nélkül betölt, 4 konténertípus alapárral.
- [ ] 1.2 Árbecslés: 5 m³ + `1131 Budapest, Reitter Ferenc utca 12.`,
      hétköznapi dátum → 45 150 Ft; hétvégi dátumnál +15% felár sor
      (51 923 Ft).
- [ ] 1.3 Rendelésleadás sikeres → `/rendeles/sikeres` oldalon
      rendelésszám (`R-2026-XXXX`) + "Fizetés indítása" gomb.
      Jegyezd fel a rendelésszámot — ez lesz a fő demo-rendelés.
- [ ] 1.4 Tegnapi kívánt dátummal a leadás hibát ad (400).

## 2. Mock fizetés (a fő rendelésen + egy másodikon)

- [ ] 2.1 "Fizetés indítása" → `/fizetes/mock` oldalon rendelésszám
      és összeg látszik; start után a rendelés "Fizetésre vár".
- [ ] 2.2 Sikeres szimuláció → admin adatlapon "Fizetve" + Payment
      tranzakciósor; FIZETVE-n újbóli start 409.
- [ ] 2.3 Admin "Visszatérítve jelölés" (megerősítéssel) →
      Visszatérítve, több fizetési művelet nincs.
- [ ] 2.4 Adj le egy MÁSODIK rendelést, indíts fizetést, futtasd a
      SIKERTELEN ágat → "Fizetés sikertelen"; majd admin "Fizetettnek
      jelölés" → Fizetve. (A failed ág így legalább egyszer lefutott.)

## 3. Admin rendeléslista (14. csomag)

- [ ] 3.1 `/admin/rendelesek`: mindkét rendelés látszik.
- [ ] 3.2 Keresés rendelésszámra és ügyfélnévre → találat.
- [ ] 3.3 Státusz-szűrő és fizetés-szűrő működik, kombinálva is;
      "Szűrők törlése" nulláz; lapozó megjelenik, ha 20+ rendelés van
      (seedelt DB-nél jellemzően nem — elég a szűrők ellenőrzése).

## 4. Rendelés-adatlap: kiosztás + audit (5/15. csomag)

- [ ] 4.1 A fő rendelés adatlapján kihelyezés kiosztása: szabad 5 m³
      konténer + Kovács Béla + teherautó → "Kihelyezésre vár", a
      konténer FOGLALT.
- [ ] 4.2 Az adatlap alján az Eseménynapló mutatja időrendben
      (legfrissebb elöl): ORDER_CREATED_PUBLIC, PAYMENT_* események,
      JOB_ASSIGNED_KIHELYEZES; a "Nyers adat" lenyitható, formázott
      JSON.

## 5. Sofőr flow + teljes státuszgép (4/5. csomag)

- [ ] 5.1 Kovácsként `/sofor`: a kártyán rendelésszám, ügyfél +
      telefon, cím, konténerkód, rendszám, Maps-link. A másik sofőr
      (Nagy) NEM látja ezt a jobot.
- [ ] 5.2 KIHELYEZES: Indítás → Folyamatban; Teljesítve → kártya
      eltűnik; a rendelés "Kihelyezve", a konténer "Használatban" az
      ügyfél címén.
- [ ] 5.3 URITES: admin kiosztja → sofőr teljesíti → a rendelés
      visszaáll "Kihelyezve"-re, a konténer kint marad.
- [ ] 5.4 CSERE: admin kiosztja (új konténer kötelező) → sofőr
      teljesíti → az adatlapon már az új konténer, a régi SZABAD.
- [ ] 5.5 ELSZALLITAS: admin kiosztja → sofőr teljesíti → a rendelés
      "Teljesítve", a konténer SZABAD, cím nélkül.
- [ ] 5.6 FAIL ág: adj le egy HARMADIK rendelést, oszd ki, sofőrként
      "Sikertelen" okkal → a job SIKERTELEN, az adatlapon megjelenik
      az újrakiosztási blokk.

## 6. Lemondás (7. csomag)

- [ ] 6.1 A harmadik (sikertelen kihelyezésű) rendelést mondd le:
      megerősítés után "Lemondva", a konténer felszabadul (SZABAD),
      az Eseménynaplóban ORDER_CANCELLED.

## 7. Admin listák (9/16/17/18. csomag)

- [ ] 7.1 `/admin` dashboard: a csempék darabszámai a fenti műveletek
      utáni állapotot tükrözik; a fejlécből minden lista elérhető.
- [ ] 7.2 `/admin/kontenerek`: a demo során használt konténerek helyes
      státusszal; a lezárt rendelés konténere SZABAD, "Telephely /
      nincs kint"; kereséssel megtalálható kód/ügyfél alapján.
- [ ] 7.3 `/admin/eroforrasok`: Kovácsnál az aktív feladatszám a
      valós állapotot mutatja (a teljesítettek nem számítanak);
      aktivitás-szűrő működik.
- [ ] 7.4 `/admin/feladatok`: az összes eddigi job látszik
      esedékesség szerinti sorrendben; státusz/típus/sofőr/dátum
      szűrők működnek; a rendelés-link az adatlapra visz.

## 8. Jogosultság

- [ ] 8.1 Kijelentkezve bármely `/admin/...` URL → `/login`.
- [ ] 8.2 Sofőrként belépve `/admin` (és al-oldalai) → `/sofor`.
- [ ] 8.3 Adminként a `/sofor` betölt és mutatja az aktív feladatokat.

## 9. Repo / zip hygiene

- [ ] 9.1 A csomagban VAN `.env.example`.
- [ ] 9.2 NINCS valódi `.env`.
- [ ] 9.3 NINCS `node_modules`, `.next`, `tsconfig.tsbuildinfo`.

Gyors ellenőrzés zipre:
```bash
zipinfo -1 <zip> | grep ".env.example"           # találat kell
zipinfo -1 <zip> | grep "/.env$"                 # üres kell
zipinfo -1 <zip> | grep -cE "node_modules|\.next/|tsbuildinfo"  # 0 kell
```

## Ismert, szándékos v1 korlátok (lezáráskor is érvényben)

Nincs valódi fizetési szolgáltató (a fizetés mock), nincs számlázás,
nincs email/SMS értesítés, nincs ügyfélportál, nincs admin CRUD
user/teherautó/konténer létrehozásra, nincs útvonaloptimalizálás,
nincs production hardening. Migration workflow a 21. csomag óta VAN
(baseline migration + `prisma migrate deploy`); a `db push` csak
fejlesztői gyorsút.
Részletek: README "Ismert v1 korlátok" és
[`handover-checklist-v1.md`](handover-checklist-v1.md) 5. szakasz.

## Sandbox-megjegyzés

Fejlesztői sandboxban (ahol a `binaries.prisma.sh` nem elérhető) a
`prisma generate` a `docs/manual-test-package-15.md` végén leírt
env-var workarounddal futtatható typecheck/build erejéig; futtatáshoz
(dev szerver, db) valódi engine kell, tehát a fenti checklist teljes
körét normál környezetben kell végigvinni.
