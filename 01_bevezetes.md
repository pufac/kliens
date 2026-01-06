Íme a „Kliensoldali rendszerek” előadásanyagának részletes, tanulható formátumú feldolgozása. Az anyagot logikai blokkokra osztottam a diák alapján, minden információt megtartva és kifejtve.

---

# Kliensoldali Rendszerek – Előadás Jegyzet (1. Előadás: Bevezetés)

## 1. Általános Tantárgyi Információk (1-8. dia)

### Oktatók és Elérhetőség
A tárgyat a Villamosmérnöki és Informatikai Kar (VIK) Automatizálási és Alkalmazott Informatikai Tanszéke (AUT) gondozza.
*   **Dr. Kővári Bence** (Tárgyfelelős, UX, Kliensoldali alapok): `kovari.bence@vik.bme.hu`
*   **Rajacsics Tamás** (React, TypeScript, Angular, Vue): `Rajacsics.tamas@vik.bme.hu`
*   **Albert István** (Blazor): `albert.istvan@vik.bme.hu`
*   **Hivatalos honlap:** [https://edu.vik.bme.hu](https://edu.vik.bme.hu) (Moodle rendszer)

### Oktatási forma
*   **Előadás:** Minden héten szerdán 16:15-kor (Helyszín: IB.025).
*   **Gyakorlat:** A 4. héttől kezdődik (beosztás és termek a Moodle-ben).

### Számonkérés és Pontozás
A félév végi jegy és az aláírás megszerzése az alábbiak szerint áll össze (Összesen 100 pont):

1.  **Gyakorlatok (Aláírás feltétele):**
    *   Az első 6 gyakorlatból legalább **4-en részt kell venni**.
    *   **Fontos:** A gyakorlatok pótlására nincs lehetőség!

2.  **Házi feladat (40 pont):**
    *   A tárgyhonlapon meghirdetett feladatok közül egyet kell megvalósítani.
    *   **Bemutatás:** A 7. gyakorlaton.
    *   **Pótbeadás:** A pótlási héten lehetséges.
    *   **Szükséges minimum:** Az aláíráshoz a 40 pontból legalább 50%-ot (20 pontot) el kell érni.

3.  **Vizsga (60 pont):**
    *   Írásbeli vizsga a vizsgaidőszakban.
    *   **Szükséges minimum:** Legalább 50%-ot (30 pontot) el kell érni a sikeres vizsgához.

4.  **Bónuszpontok (Max. 10 pont):**
    *   **Gyakorlat:** Az utolsó 2 gyakorlaton való részvétel alkalmanként 2-2 pluszpontot ér.
    *   **Előadás:** Az utolsó 6 előadáson való részvétel összesen 6 pluszpontot érhet.
    *   Ezek a pontok a vizsgapontszámhoz adódnak hozzá.

### Tematika
A félév során a modern kliensoldali technológiák széles skáláját fedik le:
1.  Kliensoldali alkalmazások elmélete (Kővári Bence)
2.  Ergonómia és UX (Kővári Bence)
3.  TypeScript alapok
4-8. React keretrendszer
9-11., 13. Blazor technológia
12. Angular és Vue keretrendszerek áttekintése

### Kapcsolódó tárgyak és előkövetelmények
A tárgy épít a korábbi ismeretekre (Programozás alapjai 3 - Java/Swing, Szoftvertechnikák - C#, Mobil- és webes szoftverek - HTML/CSS/JS, Adatvezérelt rendszerek - EF, WebAPI).
*   **Elvárás:** A 3. hétig a Moodle-ben található HTML, CSS és JS felzárkóztató anyagok átnézése kötelező.

---

## 2. Mesterséges Intelligencia (AI) a tárgyban (10-12. dia)

### Eszközök
*   **GitHub Copilot:** Diákoknak ingyenesen elérhető (GitHub Student Developer Pack). Ez egy kódkiegészítő AI, amely segítheti a fejlesztést.

### Irányelvek és Hatások
*   **Tanulási hatás:** Kutatások (Marina Lepp et al., 2025) kimutatták, hogy bár az AI használatának gyakorisága negatívan korrelálhat a teszteredményekkel (ha gondolkodás helyett használják), a segítségként való alkalmazása hasznos lehet.
*   **Szabályok a házi feladathoz:**
    *   Használható „tanácsadóként” (pl. „Hogyan lehet középre igazítani CSS-ben?”).
    *   **Tilos:** A teljes kódot az AI-val megíratni. A kódot egyénileg kell elkészíteni.
    *   **Dokumentálás:** A beadott feladat mellékleteként fel kell sorolni a használt AI eszközöket és a fontosabb promptokat.

---

## 3. Bevezetés a Kliensoldali Szoftverekbe (14-20. dia)

### Definíció
A **kliensoldali szoftver** olyan program, amely közvetlenül a felhasználó eszközén (PC, okostelefon, tablet) fut.
*   Bár kommunikálhat távoli szerverrel (adatok lekérése), a **végrehajtás, számítás és adatfeldolgozás** helyben, a kliensen történik.
*   Típusai: Webes alkalmazások, Mobil alkalmazások, Asztali alkalmazások.

### 3 Rétegű Architektúra
A modern szoftverek általában három rétegre tagolódnak (a réteghatárok gyakran hálózati határok is):
1.  **Megjelenítési réteg (Presentation Layer - UI):** Felhasználói felület, UI komponensek.
2.  **Üzleti logika réteg (Business Logic Layer - BLL):** Üzleti folyamatok, szabályok, entitások.
3.  **Adatelérési réteg (Data Access Layer - DAL):** Adatbázisok, szolgáltatás kliensek.

### Web alapú alkalmazások
Fontos különbséget tenni egyszerű weboldalak és webes alkalmazások között.
*   **Jellemzők:**
    *   Webes technológiákat használnak (HTML, CSS, JS).
    *   Nem egyszerűen weboldalak, hanem **alkalmazásként viselkednek** (design, működés, OS integráció pl. drag&drop).
    *   Nem linkelnek ki más oldalakra, zárt egységet képeznek.
*   **Multiplatform:** A cél, hogy a kód egyszeri megírásával a lehető legtöbb eszközön fusson (Windows, macOS, iOS, Android – >95% lefedettség).
*   **Technológiák:**
    *   HTML+CSS: Tartalomfogyasztó (pl. Twitter) és utility appok.
    *   Canvas: Játékok, multimédia (nagy teljesítményigény esetén).

---

## 4. Történeti Áttekintés és "Miért most?" (21-30. dia)

### A Böngésző Háborúk
1.  **Korszak (1995-2009):** Internet Explorer vs. Netscape/Firefox.
2.  **Korszak (2010-től):** A Google Chrome dominanciája (jelenleg kb. 65%-os részesedés), Safari (iOS miatt), Edge, Firefox.
    *   2010 után kialakult egy **stabil ökoszisztéma**: a böngészők elkezdték szigorúan követni a szabványokat és drasztikusan javult a JavaScript végrehajtási sebessége (JS motorok).

### Fejlődési Mérföldkövek
*   **2007:** iPhone bemutatása – a web mint mobil app platform víziója.
*   **2010:**
    *   **NPM (Node Package Manager):** A csomagkezelés forradalma.
    *   **AngularJS:** Megjelennek a komoly kliensoldali keretrendszerek.
*   **2013:**
    *   **React:** Új megközelítés a UI fejlesztésben.
    *   **Electron:** Webes technológiák használata asztali alkalmazások készítésére (mivel a böngészők már elég gyorsak lettek).
*   **2014-2015:**
    *   **Csomagolók (Bundlers):** Webpack, Rollup – a kód optimalizálása (Tree-shaking, lazy loading).
    *   **PWA (Progressive Web Apps):** Vezérlés a cache felett, offline működés.
    *   **Eszközök:** Visual Studio Code térhódítása.
*   **2020:** ESM csomagolók (pl. Vite) – gyorsabb fejlesztői élmény.

### Jelenlegi Helyzet
*   Lassan változó szokások, nagy "legacy" (régi) kódbázisok.
*   Az alkalmazások mérete folyamatosan nő.
*   A fejlesztői eszközpark (toolchain) rendkívül összetett és folyamatosan változik, nincs egyetlen integrált környezet.

---

## 5. Kliens vs. Szerver oldali Keretrendszerek (31-36. dia)

### Architektúrális Különbség
*   **Szerver oldali (pl. ASP.NET, PHP, JSP):**
    *   A kliens (böngésző) HTTP kérést küld.
    *   A szerver feldolgozza és **kész HTML-t** (plusz CSS/JS) küld vissza.
    *   Előnye: Működhet JS nélkül is.
*   **Kliens oldali (pl. React, Angular, Vue, Blazor):**
    *   A szerver csak az adatokat küldi (**JSON** formátumban), vagy statikus fájlokat.
    *   A kliens (böngészőben futó JS kód) építi fel a HTML-t az adatokból.
    *   Blazor esetén: WebAssembly-t használ, C# kód fut a böngészőben.

### Kliens oldal előnyei
1.  **Skálázhatóság:** A renderelés (megjelenítés) terhe lekerül a szerverről, a kliens gépét használjuk.
2.  **Sebesség és Élmény:**
    *   Navigáláskor nem kell az egész oldalt újratölteni/letölteni, csak az adatokat (JSON < HTML).
    *   Animációkkal elrejthető a várakozási idő (gyorsabbnak *tűnik*).
3.  **Funkcionalitás:**
    *   Hozzáfér eszköz-specifikus funkciókhoz (értesítések, megosztás).
    *   Offline működés (PWA).
    *   Több szerverrel is kommunikálhat közvetlenül.

---

## 6. Kihívások a Kliensoldalon (37-43. dia)

A kliensoldali fejlesztés számos olyan nehézséggel küzd, ami szerveroldalon nem, vagy máshogy jelentkezik:

1.  **Környezet:** Heterogén (sokféle OS, böngésző verzió, felbontás), ismeretlen konfigurációk, ellenőrizetlen biztonsági környezet.
2.  **Felhasználó:** Kiszámíthatatlan, türelmetlen, mindenkinél máshogy viselkedhet az app.
3.  **Hibakezelés:**
    *   A hibák detektálása nehéz, mivel a kliens gépén történnek.
    *   A hibajavítás költsége az idő előrehaladtával exponenciálisan nő (lásd a grafikont a 40. dián).
4.  **Technológiai stack:** Rohamosan változó keretrendszerek és eszközök (ami ma menő, holnap elavult).
5.  **Ergonómia (UX):** Fontos a reszponzivitás, intuitív felületek tervezése, prototipizálás.

---

## 7. Botok és SEO Problémák (44-49. dia)

Mivel a kliensoldali appoknál a HTML üresen érkezik és JS tölti fel, ez gondot okoz a robotoknak.

### Indexelő Crawlerek (Google, Bing)
*   Feladatuk a keresőmotorok adatbázisának építése.
*   Többségük **nem futtat JavaScriptet**, így üres oldalt lát.
*   A Google és Bing képes már JS futtatásra, de ez lassú és hibázhat komplex oldalaknál.

### Share Link Botok (Social Media)
*   Facebook, Twitter, Viber stb., amikor megosztasz egy linket.
*   Keresik a címet, leírást, képet (`og` tagek).
*   **Soha nem futtatnak JS-t**. Ha a meta tageket is a kliens generálja, a megosztás hibás lesz (nem jelenik meg kép/cím).

### AI Botok
*   Adatgyűjtés (tanítás) vagy RAG (Retrieval-Augmented Generation) céljából.
*   Nagy terhelést okozhatnak és torzíthatják a látogatási statisztikákat.

### Megoldás: SSR (Server Side Rendering)
*   **Hibrid megoldás:** Az oldal első betöltésekor a szerver (pl. Node.js) futtatja le a JS kódot és kész HTML-t küld vissza.
*   Ezután a böngésző átveszi az irányítást (hydration), és kliensoldali appként működik tovább.
*   Ez megoldja a SEO és a Social Media megosztás problémáit.

### Mikor NEM számítanak a botok?
*   Intranet rendszerek (belső hálózat).
*   Jelszóval védett tartalmak (admin felületek).
*   Webes játékok, utility alkalmazások, ahol nem cél a keresőoptimalizálás.