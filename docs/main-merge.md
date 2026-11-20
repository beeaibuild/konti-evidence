# A main ágra kerülő változások

## A GitHub-szabály beállítása

A szabály a `main` ágat védi. Dokumentáció is PR-ban érkezik. A privát, személyes repóhoz
GitHub Pro szükséges. Az előfizetés és a szabály aktiválása két külön lépés.

1. A bevezető CI-PR-on várd meg a teljes, sikeres GitHub Actions-futást. A két alábbi
   név akkor választható ki, ha volt sikeres futásuk a repóban az előző hét napban.
2. A branch protection beállításainál kapcsold be a **Require a pull request before merging** és a
   **Require status checks to pass before merging** opciót. Mindkét check forrása **GitHub Actions**.
3. Kapcsold be a **Require branches to be up to date before merging** és a
   **Do not allow bypassing the above settings** opciót. A közvetlen admin push is a szabály hatálya alá esik.
4. Ha a bevezető PR még nyitva van, azt már az aktív szabály mellett vond össze.
   A következő dokumentációs PR-on ellenőrizd a gyors út mindkét zöld eredményét.
   Az aktuális `main`-re frissített piros PR-nál ellenőrizd, hogy a konkrét hibás check blokkol.
   A blokkolás ellenőrzéséhez elég megnyitni a PR-t. Írd a fejlesztési naplóba az
   aktiválás budapesti idejét, a szabály és az ellenőrzött PR-ok linkjét.

| Kötelező check pontos neve | Mit igazol? |
| --- | --- |
| `Unit tests, typecheck and build` | Általános CI. A dokumentációs kivételnél a fájllistát ellenőrzi. |
| `Konti P63 merge gate` | Ellenőrzi a besorolás, a P63 és a szállítólevél job tényleges eredményét. |

Az összesítő megvárja a `Select P63 scope and verify isolated interpretation changes`,
a `P58 + P59 + P60 + P61-A + P62.1 + P63 required checks` és a `Delivery note operator access`
jobot. Teljes körnél mindháromnak sikerülnie kell. Szűkített körnél a besorolásnak és az abban
előírt teszteknek kell sikerülniük, a két nagy jobnál a várt eredmény `skipped`.
Hiányzó, hibás, megszakított vagy váratlanul kihagyott eredmény esetén az összesítő piros.

A nevek egyediek. Átnevezés előtt futtasd az új nevet a régi mellett, majd a sikeres futás
után add hozzá az újat a szabályhoz. Csak ezután vedd ki a régi nevet és a régi jobot.

## Mi fut a PR-on?

Mindkét workflow megkapja a `main`-re célzó PR-eseményt. A besorolás a közös őstől
számított teljes fájldiffet vizsgálja. Az átnevezés törölt és hozzáadott útvonala is számít.

| Változás | Általános CI | P63 és szállítólevél |
| --- | --- | --- |
| Kizárólag `README.md`, `docs/DEVELOPMENT-LOG.md`, `docs/main-merge.md` szöveges módosítása | Olcsó fájlellenőrzés | A besoroló job lefut, a két nagy job `skipped`. |
| A P63 útvonallistáján kívüli egyéb módosítás | Unit, typecheck, build | A két nagy job `skipped`. |
| Kizárólag a meglévő szűk AI-értelmezési körbe férő módosítás, érintett P63-útvonallal | Unit, typecheck, build | Célzott tesztek a kötelező besoroló jobban. |
| Egyéb érintett P63-útvonal, módosított CI-besorolás vagy bizonytalan diff | Unit, typecheck, build | Teljes P63 és szállítólevél E2E. |

A többi Markdown-fájlra a szokásos útvonalszabályok érvényesek.
A zárolási protokoll dokumentuma például teljes P63-at kér. A kézi és heti P63-futás teljes marad.

A PR commitjaiból és a merge-üzenetből hagyd ki a CI-kihagyó jelöléseket, köztük a
`[skip ci]`-t. Ha a GitHub ezért `Expected` állapotot mutat, küldj a PR-ra jelölés nélküli
új commitot. Más esetben ellenőrizd a PR célágát, a workflow eseményét, valamint
a job nevét és forrását. Egy workflow kihagyása függő ellenőrzést hagyhat maga után,
míg az elindult workflow szabályosan kihagyott jobját a GitHub elfogadja.

## Éles incidens, miközben áll a CI

1. A kiadást végző tulajdonos rögzíti az incidens kezdetét, az érintett funkciót,
   az ügyfelekre vagy sofőrökre gyakorolt hatást és a CI-kiesés bizonyítékát. Először
   az utolsó igazoltan jó Render-kiadásra visszaállást mérlegeli az aktuális adatbázissémával.
2. Ha az üzem csak sürgős kódjavítással állítható helyre, a javítás saját PR-ba kerül.
   A PR-ban még a kivétel előtt legyen ott a pontos head SHA, a célzott teszt és
   eredménye, a kimaradt ellenőrzés, a visszaállítási terv és a tulajdonos döntése.
   Egy lassú futás vagy egy kiadásra váró funkció a normál PR-folyamatban marad.
3. A tulajdonos menti a védelem eredeti beállítását, felfüggeszti a többi merge-et,
   és átmenetileg engedélyezi az adminisztrátori felülbírálást. Csak a rögzített SHA-jú
   incidens-PR-t vonja össze. A merge után azonnal visszakapcsolja és ellenőrzi a
   szigorú védelmet, még a többi PR folytatása előtt.
4. A kiadott SHA-n ellenőrzi az egészségvégpontot és az érintett üzleti műveletet.
   A CI helyreállásakor ugyanarra a javításra lefuttatja a kimaradt kaput. Az eredményt
   a futás linkjével együtt a PR-hoz csatolja, és a fejlesztési naplóban is rögzíti.

**A rendkívüli merge naplóbejegyzése** tartalmazza az incidens és PR linkjét, a döntéshozót,
a head/merge/éles SHA-t, a kivétel indokát, a védelem módosításának és visszaállításának
budapesti idejét, a visszaellenőrzést, az éles próbát és a pótolt CI eredményét.
A kimaradt ellenőrzés addig nyitott tétel, amíg a független futás le nem zárult.

## Az első hét Actions-költsége

Az aktiválás utáni hét napot hasonlítsd az előző héthez. Rögzítsd a PR-ok és a teljes
P63-futások számát, a runner-perceket, a PR-onkénti perceket és az ismétlések okát.
Külön számold a `main` frissítése miatt megismételt futásokat és a hibák újrapróbálását.
Ha az ismétlés oka vagy a számlázott perc nem hozzáférhető, jelöld külön.
A jobok futási idejének összege a runnerhasználatot méri. A számlázás szerinti értéket
a GitHub Actions használati kimutatásából vedd.

[GitHub útmutató a védett ágakról](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches) · [A kötelező ellenőrzések hibakeresése](https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/troubleshooting-required-status-checks).

## Élesítés

A `P63 customer auth gate` workflow kézi indításakor válaszd a `main` ágat és
jelöld be a `deploy_production` opciót. A P63 a kiválasztott commiton fut.
A deploy job az aktuális védett main SHA-ját, a GitHubon kötelezőként beállított
checkek nevét, forrásalkalmazását és sikeres lezárását ellenőrzi. Hiba esetén
a job kiírja az érintett checket. A Render ugyanezt a teljes SHA-t kapja.

Az egyszeri beállításhoz a Render éles szolgáltatásának deploy hookját a
`production` GitHub environment `RENDER_PRODUCTION_DEPLOY_HOOK` secretjeként
mentsd el, és az environment deployment branch beállításában csak a `main` ágat
engedélyezd. Az éles szolgáltatás auto-deploy beállítása legyen kikapcsolva.
A Render Dashboard közvetlen kézi indítása megkerüli ezt az ellenőrzést.
