Íme a „Kliensoldali rendszerek” tárgy **React alapok** témakörű előadásának részletes, diáról-diára történő feldolgozása. Az anyagot logikai blokkokra bontottam a könnyebb tanulhatóság érdekében.

---

# Kliensoldali Rendszerek – React Alapok

## 1. A kliensoldali renderelés problémaköre (1-7. dia)

Miért van szükségünk keretrendszerekre? Miért nem elég a natív JavaScript és HTML manipuláció?

### A probléma gyökerei
1.  **HTML string manipuláció (`innerHTML`):**
    *   *Módszer:* `div.innerHTML = '<span>' + name + '</span>';`
    *   *Előny:* Könnyű előállítani, egyszerűbb komponensekre bontható.
    *   *Hátrány:* Olvashatatlan kód (stringben nincs szintaxiskiemelés), biztonsági kockázatok.
    *   **Fő gond:** A teljes HTML cseréje történik. Ez **villogást** okoz, **lassú** (10-500 ms elemszámtól függően), és **elveszíti az állapotot** (pl. ha épp gépeltünk egy input mezőbe, a fókusz és a beírt szöveg elveszik a frissítéskor).

2.  **Manuális DOM manipuláció (`querySelector`):**
    *   *Módszer:* Megkeressük az elemet és frissítjük: `document.querySelector(...).textContent = ...`
    *   *Hátrány:* A kód erősen függ a HTML szerkezetétől. Ha változik a HTML struktúra, a kód törik. Nehezen karbantartható.

### A strukturális kihívás
A HTML egy fa szerkezet. Amikor változás történik (pl. egy új elem kerül a listába), szinkronizálni kell a régi és az új állapotot (Tree Reconciliation).
*   Két fa összehasonlítása és a minimális módosítások (insert, remove) generálása algoritmikusan nehéz ($O(n^3)$), ami lassú.
*   *Naiv megoldás:* Mindent újragenerálni (lásd fent: lassú, állapotvesztés).

### A Megoldás: Modern Keretrendszerek
Olyan eszközt készítünk, ami:
*   Legyártja helyettünk a HTML-t.
*   **Intelligensen frissít:** Nem villog, gyors, és nem veszti el a fókuszt/állapotot.
*   **Adatkötést (Data Binding)** biztosít.
*   Jó fejlesztői eszközöket (Tooling) ad: kódellenőrzés fordítási időben, debuggolás.

---

## 2. Keretrendszerek és a Kompozíció elve (8-16. dia)

### Vizsgált technológiák
A félév fő témája a **React** (legnépszerűbb, csak UI könyvtár, hatalmas ökoszisztéma). Érintőlegesen szóba kerül a *Vue* (szintén UI fókuszú) és az *Angular* (teljes keretrendszer).

### Kompozíció (Összetétel) tervezési minta
A modern UI fejlesztés alapja a **Composite Design Pattern**.
*   **Cél:** A felületet (UI) kisebb, újrafelhasználható egységekre, úgynevezett **komponensekre** bontjuk.
*   **Struktúra:** Fa szerkezet (szülő és 0 vagy több gyerek).
*   **Egységes kezelés:** A minta lényege, hogy egy elemet (levél) és egy konténert (ami más elemeket tartalmaz) azonos interfészen keresztül kezelünk. Ez egyszerűsíti a fejlesztést, nem kell minden típussal külön foglalkozni.

### Implementációs kérdések
*   **Öröklés vs. Kompozíció:** Bár meg lehetne oldani örökléssel (pl. `Container` és `Leaf` osztályok), a gyakorlatban a kivételek (pl. `<input>` nem lehet gyereke) miatt ez nehézkes.
*   **React megközelítés:** **"Composition over Inheritance"**. A React (és a modern irányzat) kerüli az öröklést. Helyette a komponensek egymásba ágyazását (kompozíció) és a tulajdonságok (props) átadását preferálja a specializációhoz.

### Komponensek jellemzői
*   **Dekompozíció:** A bonyolult felületet részekre bontjuk.
*   **Egységbe zárás (Encapsulation):** A komponens csak a saját működéséért felel.
*   **Újrafelhasználhatóság:** Jól definiált határok miatt máshol is bevethető.
*   **Tartalma:** Nézet leírása (HTML/sablon), interakciók (események), állapotkezelés.

---

## 3. Alkalmazástervezés lépései (17-21. dia)

Hogyan állunk neki egy React alkalmazásnak?

1.  **Dekompozíció:** Iteratív folyamat. A felületet addig bontjuk kisebb komponensekre, amíg a fejlesztő számára átlátható nem lesz. Figyelembe kell venni a CSS/Design szempontokat is.
2.  **Statikus verzió:** Először készítsük el az alkalmazást dinamikus adatok nélkül. Ez gyors visszajelzést ad és segít látni az újrafelhasználható részeket.
3.  **Állapot (State) tervezése:**
    *   Csak a minimálisan szükséges változó adatokat tároljuk (pl. mi van kiválasztva, mit gépeltek be).
    *   A konstansok vagy számítható adatok nem állapotok!
    *   **Hol tároljuk?** Tipikusan abban a közös szülő komponensben (vagy globálisan), amelynek a részfájában szükség van rá ("Lifting state up").
4.  **Adatfolyam:**
    *   A React-ben **egyirányú adatfolyam** van (lefelé).
    *   Nincs automatikus kétirányú adatkötés (mint pl. Angularban). Ha a gyerek módosítani akar, a szülőtől kapott visszahívó függvényt (callback) kell használnia.

---

## 4. React Technológiai Alapok (22-29. dia)

### Hello World és a `createElement`
A legegyszerűbb React komponens egy függvény, ami visszaad egy UI leírást.
```javascript
function HelloWorld() {
  return <h1>Hello World!</h1>;
}
```
A háttérben ez nem HTML string, hanem JavaScript objektumok létrehozása:
*   A `return <h1>...</h1>` valójában `React.createElement("h1", null, "Hello World")` hívássá fordul.
*   A `ReactDOM.render` végzi el ezeknek az objektumoknak a konvertálását valódi DOM elemekké a böngészőben.

### JSX és TSX
*   **JSX:** Szintaktikai cukorka (syntactic sugar). Úgy néz ki, mint a HTML, de JavaScript kódba írjuk. A fordító (Babel) alakítja át `createElement` hívásokká.
*   **TSX (TypeScript JSX):** Ugyanez, csak típusellenőrzéssel. Előnye, hogy fordítási időben kiderülnek a hibák (pl. elírt tulajdonság név) és működik a kódkiegészítés.

### Virtual DOM és Működés
*   **React Fa (Virtual DOM):** A React nem a böngésző DOM-ját, hanem egy saját, memóriában lévő objektum-fát (ReactElement objektumok) épít fel.
*   **Szinkronizáció (Reconciliation):**
    1.  Minden változáskor (pl. állapot változás) a React újrahívja a render függvényt.
    2.  Létrejön az új Virtual DOM fa.
    3.  Összehasonlítja a régivel (Diffing).
    4.  Kiszámolja a különbséget (parancslista).
    5.  Csak a szükséges változtatásokat hajtja végre a valódi DOM-on.
*   *Eredmény:* Gyors működés, nincs villogás, megmarad az input fókusz. Ezért **tilos** a DOM-ba kézzel beleírni, mert a React a következő frissítésnél felülírja.

---

## 5. Props (Tulajdonságok) (30-36. dia)

A **Props** a komponens publikus interfésze, ezen keresztül kommunikál a szülő a gyerekkel (Szülő -> Gyerek adatfolyam).

*   **Használat:** Úgy néz ki, mint a HTML attribútum: `<Greeter name="Leo" />`.
*   **Jellemzők:**
    *   **Read-only:** A gyerek komponens nem módosíthatja a saját props-ait.
    *   **Típusok:** Bármilyen JS típus átadható (string, szám, objektum, függvény, másik komponens). Nem string típusnál kapcsos zárójelet kell használni: `active={true}`.
    *   **Fogadás:** Függvény komponensnél paraméterként érkezik. Gyakori a *destructuring*: `function Greeter({ name }) { ... }`.
*   **Opcionális props:** TypeScriptben `?`-el jelöljük (`name?: string`). Ha nincs megadva, értéke `undefined`.
*   **Children prop:** A komponens nyitó és záró tagje közötti tartalom a `children` prop-ban érkezik. Ez teszi lehetővé a komponensek egymásba ágyazását.
    ```javascript
    <Card>
       <h1>Cím</h1>  {/* Ez lesz a children */}
    </Card>
    ```
*   **Adatkötés szintaktika (`{}`):** A JSX-ben kapcsos zárójelek közé írhatunk JavaScript kifejezéseket. (Pl.: `{user.name}`). Vezérlési szerkezetek (`if`, `for`) nem írhatók ide, helyettük ternáris operátort (`condition ? true : false`) vagy `.map()`-et használunk.

---

## 6. Komponens Típusok: Osztály vs. Függvény (37-40. dia)

### Osztály Komponens (Class Component)
*   A "régi" módszer.
*   Szintaxis: `class MyComp extends Component { render() { ... } }`.
*   Jellemzők: Van belső állapota (state), életciklus metódusai, de a kódja hosszabb, és kezelni kell a `this` kontextust (bindolás vagy arrow function).

### Függvény Komponens (Function Component)
*   A modern szabvány (2019+).
*   Egyszerű JS függvény, ami JSX-et ad vissza.
*   **Hooks:** A React 16.8 óta a függvény komponensek is képesek állapotot kezelni és életciklus eseményekre reagálni (pl. `useState`, `useEffect`).
*   Előny: Rövidebb, tisztább kód.

---

## 7. Életciklus (Lifecycle) (41-44. dia)

A komponensek élete során különböző fázisok vannak, ahol beavatkozhatunk (főleg osztály komponenseknél voltak ezek explicit metódusok):

1.  **Mounting (Létrehozás):**
    *   `constructor`: Inicializálás.
    *   `render`: A fa felépítése.
    *   `componentDidMount`: Első kirajzolás után fut le (pl. itt indítunk hálózati kérést vagy időzítőt).
2.  **Updating (Frissítés):**
    *   `render`: Újra lefut, ha változik a props vagy a state.
    *   `componentDidUpdate`: Frissítés után fut le.
3.  **Unmounting (Megszűnés):**
    *   `componentWillUnmount`: Mielőtt a komponenst kivennék a DOM-ból (takarítás, pl. `clearInterval`).

*Megjegyzés:* Függvény komponenseknél ezeket a `useEffect` hook segítségével érjük el.

---

## 8. Renderelési technikák (45-52. dia)

*   **Render visszatérési értéke:** Mindig egy fát (ReactElement) ad vissza.
*   **Feltételes renderelés:**
    *   `{ condition && <Component /> }`: Ha a feltétel igaz, megjelenik, ha hamis (false, null, undefined), a React nem rajzol semmit. (Vigyázat: a 0 szám kirajzolódik!)
    *   `{ condition ? <TrueComp /> : <FalseComp /> }`: Választás két ág között.
*   **Listák renderelése:**
    *   Tömböt adunk vissza `.map()` segítségével.
    *   **Fontos a `key`:** Minden listaelemnek egyedi `key` attribútumot kell adni (lehetőleg adatbázis ID-t). Ez alapján tudja a React hatékonyan követni, melyik elem jött létre, melyik törlődött vagy változott. Az index használata kerülendő (lassú lehet átrendezésnél).
*   **Fragment (`<>...</>`):** Ha több elemet akarunk visszaadni egy komponensből, de nem akarunk feleslegesen egy burkoló `<div>`-et létrehozni (ami elronthatná pl. a CSS Flexbox/Grid layoutot), akkor `React.Fragment`-et vagy az üres taget használjuk.

---

## 9. Attribútumok kezelése (53-55. dia)

*   **Feltételes attribútumok:**
    *   Ha egy attribútum értéke `false` vagy `undefined`, a React nem rakja be a DOM-ba.
    *   Példa: `<input disabled={isDisabled} />` (ha `isDisabled` false, az attribútum eltűnik).
    *   Használható a spread operátor is objektumok átadására: `<input {...props} />`.
*   **Elnevezési különbségek (JSX vs HTML):**
    *   Mivel a JSX JavaScriptre fordul, a foglalt szavakat kerülni kell.
    *   `class` helyett -> **`className`**
    *   `for` helyett -> **`htmlFor`**
*   **Class listák:** A CSS osztályok dinamikus összefűzése gyakori feladat, erre léteznek segédkönyvtárak (pl. `classnames`), de sablon stringekkel is megoldható.