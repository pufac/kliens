Rendben, itt az utolsó rész a PDF-ből: **Teljesítmény, Csomagolás és Formátumok**. Ez már nem React-specifikus, hanem általános webes tudás, ami elengedhetetlen egy modern kliensoldali rendszernél.

Vágjunk bele, egyszerűen és érthetően!

---

# 1. Teljesítmény (Performance) - Mitől lesz gyors? (2-6. dia)

Miért érzünk egy oldalt gyorsnak vagy lassúnak? Két fő mérőszám van:
1.  **Letöltés/Betöltés:** Mennyi idő, amíg megjelenik valami? (Hálózati sebesség + fájlméret).
2.  **Futás:** Mennyire reszponzív, ha kattintgatok? (JavaScript sebesség).

### Kulcsfontosságú mérőszámok (4-5. dia)
Ezeket méri a Google Lighthouse is:
*   **FP (First Paint):** Megjelent valami a képernyőn (akár csak egy háttérszín). Nem üres fehérség van.
*   **FCP (First Contentful Paint):** Megjelent az első *tartalom* (szöveg, kép).
*   **TTI (Time to Interactive) - A LEGFONTOSABB:** Mikor válik használhatóvá az oldal?
    *   Lehet, hogy látod a gombot, de ha rákattintasz és nem történik semmi 3 másodpercig (mert a JS még töltődik a háttérben), az frusztráló.
    *   **53%-a a felhasználóknak lelép**, ha 3 másodperc alatt nem tölt be az oldal.

### Hogyan optimalizáljunk? (6. dia)
*   **Caching:** Ne töltsük le, ami már megvan.
*   **SSR (Server Side Rendering):** A szerver küldjön kész HTML-t, ne a kliensnek kelljen mindent kiszámolnia a nulláról.
*   **Tree Shaking & Minify:** Csak a szükséges kódot küldjük, és azt is tömörítve.
*   **Lazy Loading:** Csak azt töltsük le, ami épp kell.
*   **Modern formátumok:** WebP képek, WOFF2 fontok.

---

# 2. Csomagolás (Bundling) - A kód előkészítése (7-11. dia)

A böngésző nem érti a TypeScriptet, a JSX-et, és nem szeret ezer kis fájlt letölteni. Ezért kell a **Build folyamat**.

**1. Fordítás (Transpiling):**
*   TS -> JS
*   JSX -> JS
*   Új JS (ES6+) -> Régi JS (hogy fusson régi böngészőn is).

**2. Source Map (Térkép):**
*   Ha a kódodat átalakítod és összetömöríted, a böngészőben egy katyvaszt látsz. Ha hiba van, nem tudod hol keresd.
*   A **Source Map** (`.map` fájl) megmondja a böngészőnek: "Ez a tömörített sor az eredeti kód 42. sorának felel meg". Így tudsz debuggolni.

**3. Bundling (Csomagolás):**
*   Régen: Minden fájlt egyetlen nagy `bundle.js`-be gyúrtak össze (Webpack).
*   Ma (Vite, Rollup): Fejlesztés közben a böngésző natív modulkezelését használjuk (gyors!), élesben pedig okosan darabolt csomagokat készítünk.

---

# 3. Kódméret csökkentése (12-15. dia)

Minél kisebb a fájl, annál gyorsabban jön le.

**1. Tree Shaking (Fa rázás):**
*   Képzeld el, hogy behúzol egy hatalmas matematikai könyvtárat, de csak az `osszeadas()` függvényt használod belőle.
*   A Tree Shaking "megrázza a fát": a nem használt kód (dead code) lehullik, és nem kerül bele a végleges csomagba.
*   *Nehézség:* Dinamikus nyelveknél (JS) nehéz biztosan tudni, mi nem kell, ezért nem 100%-os.

**2. Minify (Tömörítés):**
*   A kód olvashatóságát feláldozzuk a méretért.
*   Szóközök, sortörések törlése.
*   Változónevek rövidítése (`function calculateTotalPrice(price)` -> `function a(b)`).
*   Eredmény: 3-4x kisebb fájl.

---

# 4. Lazy Loading (Lusta betöltés) (16. dia, 20. dia)

*   **Elv:** Ne töltsünk le mindent induláskor!
*   **Kód:** A "Beállítások" oldal kódját csak akkor töltsük le, ha a felhasználó rákattint a "Beállítások" menüpontra.
    *   Ez a **Code Splitting** (kód darabolás).
    *   `import(...)` függvénnyel dinamikusan importálunk.
*   **Képek (22. dia):** `loading="lazy"` attribútum a HTML-ben. A kép csak akkor töltődik le, ha görgetéskor közel kerül a képernyőhöz.

---

# 5. Modulok (17-19. dia)

Hogyan szervezzük a kódot fájlokba?

*   **Régi módszerek:** CommonJS (`require`), AMD. Ezeket főleg Node.js-ben vagy régi rendszereknél látod.
*   **ES Modules (ESM) - A JELEN:**
    *   `import` és `export` kulcsszavak.
    *   A böngészők natívan támogatják.
    *   A modern eszközök (Vite) erre építenek.

---

# 6. Média és Formátumok (23-28. dia)

A weboldalak méretének nagy részét a képek és videók teszik ki. Itt lehet a legtöbbet spórolni.

**Képformátumok:**
*   **JPEG:** Fotókhoz jó, veszteséges.
*   **PNG:** Grafikonokhoz, logókhoz jó (éles vonalak, átlátszóság), veszteségmentes.
*   **WebP:** A Google formátuma. Kisebb mint a JPEG és a PNG, tud átlátszóságot. Használd ezt!
*   **AVIF:** Még újabb, még jobb tömörítés, de a támogatottsága még most érik be.

**Tömörítés a hálózaton (27. dia):**
*   Amikor a szerver elküldi a HTML/CSS/JS fájlt, összetömöríti (mint egy ZIP-et).
*   **Gzip:** A régi standard.
*   **Brotli:** Az új standard. 20%-kal jobb tömörítés szöveges fájlokon. Használd ezt!

**HTTP/2 és HTTP/3 (28. dia):**
*   **HTTP/1.1:** Egyszerre csak pár fájlt tudott letölteni egy szerverről.
*   **HTTP/2:** Multiplexing. Egyetlen kapcsolaton keresztül párhuzamosan ömlenek a fájlok. Nem kell trükközni a fájlok összefűzésével.
*   **HTTP/3:** UDP alapú, még gyorsabb és stabilabb rossz hálózaton.

---

# 7. Betűtípusok és Subpixel Rendering (29-31. dia)

**Font formátumok:**
*   `WOFF2`: Ez a szabvány. Tömörített, minden modern böngésző viszi. (A `ttf` és `otf` már a múlté weben).

**Subpixel Rendering (Amiért a betűk élesek):**
*   Egy LCD pixel valójában 3 sávból áll: Piros, Zöld, Kék (RGB).
*   A Subpixel Rendering nem teljes pixeleket színez, hanem ezeket a részeket külön-külön, így **megháromszorozza a vízszintes felbontást**.
*   Ettől olvasható a szöveg kis méretben is.
*   **Korlát:** Csak asztali rendszereken működik jól. Mobilon (ahol elforgathatod a kijelzőt) és Retina kijelzőkön (ahol amúgy is hatalmas a felbontás - PPI) nem használják, helyette szürkeárnyalatos (antialiasing) simítást alkalmaznak.

---

### Összefoglalás "normális" programozónak:

1.  **Build:** A kódod átmegy egy darálón (TS -> JS, Minify, Tree Shaking), mielőtt kikerül. Használj modern eszközt (Vite).
2.  **Sebesség:** A felhasználó türelmetlen. Amit nem lát, ne töltsd le (Lazy Loading). Amit lát, az legyen kicsi (WebP, Brotli, Minify).
3.  **TTI:** A legfontosabb mérőszám. Minél hamarabb lehessen kattintani az oldalon.
4.  **Média:** Felejtsd el a BMP-t és a TIFF-et. WebP és WOFF2 a barátod.
5.  **Hálózat:** A HTTP/2 megoldotta a párhuzamos letöltés gondját, nem kell mindent egyetlen óriás fájlba gyúrni, de azért ne legyen 1000 kis fájl se.

Ezzel végére értünk a "Kliensoldali rendszerek" teljes anyagának! Remélem, így már sokkal emészthetőbb volt az egész React és a modern webes világ működése. Van esetleg még kérdésed bármelyik résszel kapcsolatban?