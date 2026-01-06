Íme a „Kliensoldali rendszerek” tárgy **TypeScript és CSS** témakörű előadásának részletes feldolgozása a diák alapján.

---

# Kliensoldali Rendszerek – TypeScript és CSS

## 1. JavaScript alapok és a jelenlegi helyzet (1-7. dia)

### JavaScript támogatás és szabványok
*   **Szabvány:** A nyelvnek létezik hivatalos szabványa (ECMAScript, pl. ES2024), amit a böngészők – bár némi késéssel – követnek. A támogatás az elmúlt 15 évben rengeteget javult.
*   **Fordítók (Compilers):** A fejlesztés során gyakran modern kódot írunk, amit egy fordító alakít át a böngészők által értett verzióra (tipikusan ES6-ra/ES2015-re).
    *   **Eszközök:** Babel, TypeScript.
    *   Ez a folyamat ma már megbízhatóan és jól működik (ellentétben a HTML/CSS régebbi kompatibilitási gondjaival).
*   **Polyfill:** Ha egy funkció hiányzik a böngészőből (pl. `String.prototype.startsWith`), azt kódból pótoljuk (polyfill), így a régi böngésző is képes futtatni az új parancsokat.

### Böngészőpiac és kompatibilitás
*   A "caniuse" jelentések alapján a **Chrome** (és a Chromium alapú **Edge**) áll az élen a szabványok támogatásában.
*   Bár a különbségek a böngészők (Firefox, Safari, Chrome) között kicsik, nem elhanyagolhatók:
    *   *Példa:* Push notification és WebGL2 már mindenhol van.
    *   *Példa:* WebGPU Androidon van, de iOS-en (még) nincs.

### Gyengén típusosság (Weakly Typed)
*   **Típusok a JS-ben:** Vannak típusok (`number`, `string`, `boolean`, `Object`, `Symbol`, `function`, `bigint`, `null`, `undefined`), de ezek ellenőrzése **csak futásidőben** történik.
*   **Automatikus konverzió:** A rendszer mindenáron megpróbálja végrehajtani a kódot (pl. típuskonverzióval), és csak végső esetben dob hibát.
*   **Előny:** Kis programoknál rövid, átlátható kódot eredményez, mert nem kell kiírni a típusokat.
*   **Hátrány (Közepes és nagy szoftvereknél):**
    *   Nincs fordítási idejű ellenőrzés → sokkal többet kell tesztelni.
    *   Hiányos fejlesztői eszköztámogatás (Tooling): a kódkiegészítő (autocomplete) nehezen ad tippeket, az ellenőrző (linter) kevesebb hibát talál meg.
    *   Nehezebb megérteni a kódot (dokumentáció nélkül nem látszik, mit vár egy függvény).

---

## 2. TypeScript – Általános bevezetés (8-15. dia)

### Mi a TypeScript (TS)?
*   Lényegében **típusos JavaScript**. Minden érvényes JS kód egyben érvényes TS kód is.
*   A típusok használata **opcionális**, de amint használjuk őket, TypeScriptről beszélünk.
*   **Szintaxis:** A típust a változó neve után adjuk meg (pl. `function A(a: number, b: number)`).

### Tudásbeli egyezés
*   A TS **nem tud többet** futásidőben, mint a JS. Ha kivesszük a típusokat, sima JS-t kapunk.
*   A `typeof` operátor futásidőben itt is csak a JS típusokat ismeri fel.
*   **A különbség a fordítási lépés:** A TS fordító ellenőrzi a típusokat, majd kiveszi őket a kódból, és szabványos JS-t generál (transpilálás), amit a célplatform (böngésző) megért.

### Miért használunk típusokat?
1.  **Dokumentáció:** A kód "önmagát dokumentálja". A paraméterek típusából azonnal látszik, mit vár a függvény (nem csak a névből kell következtetni).
2.  **Tooling (Eszköztámogatás):**
    *   Jobb kódkiegészítés (IntelliSense).
    *   Hatékonyabb hibaellenőrzés (Linter).
3.  **Fordítási idejű ellenőrzés:** A hibák jelentős része már kódoláskor kiderül, nem az ügyfélnél futás közben.
4.  **Költségcsökkentés:** Bár a kódolás lassabbnak tűnhet, a tesztelés költsége és a hibajavítás ideje jelentősen csökken.

### Objektum Orientáltság (OOP) kérdése
*   A típusosság és az OOP függetlenek. A JS támogatja az OOP-t (osztályok, egységbe zárás, öröklés, absztrakció).
*   **TS és OOP:**
    *   Támogatott: Osztályok, Interfészek, Absztrakt osztályok, Öröklés, Láthatósági módosítók (`public`, `private`, `protected`), Statikus elemek.
    *   *Nem* támogatott (mert a JS alap nem tudja): Valódi metódus túlterhelés (overloading), többszörös öröklés, típusonkénti több konstruktor.
*   **Megközelítés:** A modern kliensoldali fejlesztés (pl. React) inkább **komponens alapú** és a kompozíciót preferálja az örökléssel szemben. A TS-t elsősorban a típusbiztonság miatt használjuk, nem az osztályok erőltetése miatt.

---

## 3. TypeScript típusrendszer (16-28. dia)

### Alaptípusok
*   A JS alapjain túl:
    *   Tömb: `number[]`
    *   Tuple (rögzített típusú pár/sor): `[number, string]`
    *   Enum: `enum Color { Red, Green }`
    *   `any`: kikapcsolja a típusellenőrzést.
    *   String literal: `let e: "red" | "green"` (csak ezeket a konkrét szövegeket veheti fel).

### Összetett típusok
*   **Unió (`|`):** `string | number` (vagy ez, vagy az). Nagyon gyakori, pl. polimorfizmus kezelésére. A függvényen belül `if` vizsgálattal döntjük el, melyiket kaptuk.
*   **Metszet (`&`):** `ObjA & ObjB` (mindkét típus tulajdonságaival rendelkeznie kell).

### Függvények
*   Használhatók default (`= "érték"`) és opcionális (`?`) paraméterek.
*   Ami nem opcionális, azt **kötelező** megadni (szigorúbb, mint a JS).

### Osztályok (Classes)
*   Rövidítés konstruktorban: `constructor(public name: string) {}` (létrehozza és beállítja a `name` mezőt).
*   A `private`/`protected` csak fordítási időben véd, futásidőben a JS kódban nem látszik (kivéve az új `#field` szintaxist, ami valódi privát mező).
*   Vannak getterek és setterek.

### `this` kezelése
*   A `this` kontextusa JS-ben változhat. A TS segít, de nem old meg mindent.
*   **Megoldás:** Callback függvényeknél mindig **arrow function**-t (`() => {}`) használjunk, mert az megőrzi a külső `this` kontextust.

### Type Guards (Típusőrök)
*   A fordító követi a logikai ágakat.
*   Ha egy `if (typeof x === "string")` ágban vagyunk, a fordító tudja, hogy ott az `x` biztosan string, és engedi a string műveleteket.

### Generics (Paraméteres típusok)
*   Hasonló a C# vagy C++ templatekhez.
*   Előre nem ismert típusokkal dolgozhatunk típusbiztosan: `function concat<T>(a: T, b: T)`.
*   A fordító képes kitalálni (inferálni) a típust a hívásból.
*   Kényszereket is adhatunk: `<T extends HasLength>`.

### Interfészek (Interfaces)
*   Definiálhatnak objektum tulajdonságokat, függvényeket, indexelőket.
*   Példa: `interface Indexable { [key: string]: string; }`.

### Strukturális típusosság (Structural Typing)
*   **Kulcsfontosságú fogalom!** Két változó típusa akkor azonos, ha a **szerkezetük** (felépítésük) azonos. A típus neve nem számít.
*   Ha egy objektumnak van egy `a: string` mezője, és egy interfész is ezt írja elő, akkor kompatibilisek, még ha nem is implementálta explicit módon.
*   Ez igaz függvényekre is (JS-ben megengedett kevesebb paraméterrel hívni egy függvényt, ha a típusok egyébként egyeznek).

---

## 4. Modulok és névterek (29-35. dia)

### Névterek (Namespaces)
*   `namespace NS { ... }`.
*   Egy fordítási egységen belüli kóddarabolásra való.
*   Ma már **ritkán használt**.

### Modulok (Modules)
*   **Régi megoldások:** CommonJS (Node.js), RequireJS (AMD). Ezekhez modul betöltő kellett.
*   **Natív modulok (ESM):** Ezt használjuk ma.
    *   Exportálás: `export class C {}`
    *   Importálás: `import { C } from "./c";`
    *   A modern csomagolók (pl. Vite) megoldják, hogy a böngésző megértse a TS importokat (automatikusan fordítják és feloldják a kiterjesztéseket).

### Típusdeklarációs fájlok (`.d.ts`)
*   Olyan fájlok, amelyek csak típusleírásokat tartalmaznak (`type`, `declare`, `interface`), futtatható kódot nem.
*   **Célja:** Külső JS könyvtárakhoz (pl. jQuery) típusinformációt adni a TS fordítónak.
*   A legtöbb modern könyvtár tartalmazza, vagy külön telepíthető (`@types/csomagnev`).
*   Hasznos akkor is, ha más nyelven (pl. C#, Java) írt szerver adatait akarjuk típusosan kezelni a kliensen (akár automatikusan generálható).

---

## 5. Aszinkron programozás (36-42. dia)

### Promise
*   Egy osztály, ami az aszinkron műveletek kezelését szabványosítja (callback hell helyett).
*   **Állapotok:** Folyamatban, Sikeres (resolve), Sikertelen (reject).
*   **Használat:** `.then(callback)` a sikerhez, `.catch(callback)` a hibákhoz.
*   Láncolható (`.then().then()`).
*   Egységes hibakezelést biztosít (try-catch a Promise-on belül).

### async / await
*   Modern szintaxis (ES2017+), ami a Promise-okra épül.
*   **async:** A függvény mindig Promise-t ad vissza.
*   **await:** Megvárja a Promise feloldását. A kód úgy néz ki, mintha szinkron lenne ("megáll" az await-nél), de valójában a fordító/futtató környezet az await utáni részt egy `.then()` ágba teszi.
*   Nagyban javítja a kód olvashatóságát.

### Szálkezelés JS-ben
*   A JavaScript **egyszálú** (single-threaded).
*   Az aszinkron műveletek nem külön szálon futnak, hanem **kiütemezik** őket. Amikor az eredmény megérkezik (pl. lejárt az időzítő, megjött a válasz a szervertől), a feladat visszakerül a fő szál végrehajtási sorába.
*   Emiatt a fő szálon nem kell szinkronizációval (lock) bajlódni.
*   (Megjegyzés: Web Workerekkel lehet valódi háttérszálakat létrehozni, de azok külön kontextusban futnak).

---

## 6. CSS – Nem csak design (43-46. dia)

### Layout vs. Design
*   Webes alkalmazásoknál a CSS nem csak színezést jelent.
*   **Layout (Elrendezés):** Ez fejlesztői felelősség. Az alkalmazás szerkezetének, rácsrendszerének kialakítása, hogy minden eszközön jól jelenjen meg (pl. ne lógjon le a képernyőről). A webes appok gyakran nem görgethető dokumentumok, hanem fix ablakméretű alkalmazások.
*   **Stílus:** Színek, árnyékok, betűtípusok – ez maradhat a designer feladata.

### display: grid
*   A legmodernebb és leggyakrabban használt eszköz webalkalmazásokhoz (2017 óta támogatott).
*   Lehetővé teszi a **2 dimenziós** (oszlopok és sorok) elrendezést.
*   Lehetőséget ad "fentről lefelé" (ablak mérethez igazodó) tervezésre, szemben a régi "lentről felfelé" (tartalom mérethez igazodó) módszerekkel.

### CSS változók (Custom Properties)
*   2016 óta támogatott.
*   Szintaxis: `--változónév: érték;` (definiálás), `var(--változónév)` (használat).
*   **Előnyök:**
    *   Egy helyen definiálható, bárhol felhasználható.
    *   Hierarchikusan működik (egy részfában felüldefiniálható a változó értéke).
    *   JavaScriptből is írható/olvasható (dinamikus színezés, témázás).