Rendben, vágjunk bele a második pakkba! Ez talán a legfontosabb rész, mert itt dől el, hogyan "gondolkodik" a React. A "normális" programozásban (OOP) megszoktad, hogy vannak változóid, amiket átírsz, és kész. Itt ez tilos.

Itt **Állapot (State)** van és **Események (Events)**. Nézzük meg, miért és hogyan.

---

# React Állapot- és Eseménykezelés (1-3. dia)

## 1. Hooks - A paradigmaváltás
A React fejlesztés iránya megváltozott. Régen osztályokat (`class`) írtunk, ma függvényeket (`function`).

*   **A probléma:** Az osztályok bonyolultak (this context, öröklés, boilerplate kód).
*   **A megoldás:** Írjunk sima függvényeket. De egy függvény "felejt". Ha lefut, a lokális változói törlődnek.
*   **A Hook (Horog):** Ez a varázslat. Egy Hook (pl. `useState`) olyan, mintha a függvényed egy **külső memóriarekeszbe** nyúlna ki, amit a React kezel neki. Így a függvény "emlékezni fog" dolgokra két futás között is.

---

# 2. Állapot (State) - A komponens "agya" (4-13. dia)

**Elmélet:**
A `State` (állapot) az az adat, ami idővel változhat, és ha változik, a képernyőnek is frissülnie kell (újra kell renderelni).
*   Egy sima változó (`let x = 0`) nem jó, mert ha átírod, a React nem tud róla, és nem frissíti a képernyőt.
*   Ezért kell a `useState`.

**Kód elemzése (11. dia - A legfontosabb sor!):**
```javascript
let [ date, setDate ] = useState( defaultDate ?? new Date );
```
*   `useState(...)`: Ez a függvényhívás szól a Reactnak: "Hahó, szükségem van egy memóriarekeszre!"
*   `defaultDate ?? new Date`: Ez a **kezdőérték**. Csak EGYETLEN EGYSZER, az legelső létrehozáskor számít. Később a React ignorálja.
*   `let [ ... ] = ...`: Ez az "Array Destructuring". A `useState` egy tömböt ad vissza két elemmel.
    1.  `date`: A jelenlegi érték (amit épp tárolunk).
    2.  `setDate`: Egy függvény (setter), amivel módosítani tudod az értéket.

**Hogyan működik a háttérben? (12-13. dia)**
A React nem a változó neve alapján tudja, melyik state melyik, hanem a **SORREND** alapján.
*   Ha van 3 `useState` hívásod egymás alatt, a React tudja: "Az első az 'A' adat, a második a 'B', a harmadik a 'C'".
*   **Ezért tilos `if` vagy `for` ciklusba tenni Hook-ot!** Ha az egyik renderben lefut az `if`, a másikban nem, elcsúszik a sorrend, és a React rossz adatot ad vissza rossz változónak.

**Aszinkron setState (7-8. dia - A csapda!):**
```javascript
inc() {
    this.setState( { c: this.state.c + 1 } ); 
}
```
*   Ha ezt egy eseménykezelőben 3x egymás után meghívod, nem 3-mal nő az érték, hanem csak 1-gyel.
*   **Miért?** Mert a `setState` **NEM AZONNAL** hajtódik végre. Csak "szól" a Reactnak, hogy "majd ha ráérsz, frissíts".
*   Ha 3x hívod, mindháromszor a *régi* értéket veszi alapul (pl. 0 + 1), és a végeredmény 1 lesz.
*   **Megoldás (8. dia):** Függvényt adsz át a setternek.
    ```javascript
    // "state" itt a garantáltan legfrissebb előző érték lesz
    this.setState( state => { c: state.c + 1 } );
    ```

---

# 3. Események és Inputok (14-22. dia)

Hogyan kapunk adatot a felhasználótól?

**Eseménykezelés (15. dia):**
```javascript
<button type="button" onClick={ () => setCount( count + 1 ) }>Inc</button>
```
*   `onClick={ ... }`: Ez a React szintaxisa az eseménykezelőre (camelCase).
*   `() => setCount(count + 1)`: Egy nyíl függvény (arrow function). Amikor kattintasz, ez lefut, meghívja a settert, a React észleli a változást, és újrarajzolja a komponenst az új számmal.

**Input mezők kezelése - Két út van:**

**A) State-ben tárolt (Controlled Component) - 17. dia - EZ A JAVASOLT:**
```javascript
function TextInput(){
    let [ text, setText ] = useState( "" ); // 1. State létrehozása
    // 2. Az input értéke a state-ből jön (value={text})
    // 3. Ha ír a user, frissítjük a state-et (setText)
    return <input value={ text } onInput={ e => setText( e.currentTarget.value ) } />
}
```
*   Ez egy körkörös folyamat: State -> Input megjelenik -> User ír -> Esemény -> State frissül -> Input újrarajzolódik.
*   **Miért jó?** Mert a programod minden pillanatban tudja, mi van a mezőben (pl. azonnali validációhoz).

**B) DOM-ban tárolt (Uncontrolled Component) - 19. dia:**
```javascript
function TextInputDOM(){
    // useRef: Egy "doboz", ami tárolhat egy referenciát egy HTML elemre
    let inputRef = useRef( null ); 
    // ref={inputRef}: "React, ha létrehoztad ezt az inputot, rakd bele ebbe a dobozba!"
    return <input ref={ inputRef } />
}
```
*   Itt nem a React tárolja az adatot, hanem a böngésző natív input mezője.
*   Ha kell az adat, a `inputRef.current.value`-val kéred el.
*   **Mikor kell?** Ha nem akarod minden betűleütésre újrarajzolni az oldalt, vagy `<input type="file">` esetén (biztonsági okokból fájl inputot nem vezérelhetsz JS-ből).

**Use Case: Alapérték props-ból (22. dia):**
Ez egy gyakori minta.
```javascript
// defaultText jön kintről, de utána szerkeszteni akarjuk
let [ text, setText ] = useState( defaultText ?? "" );
```
*   A `useState` csak az **első** rendereléskor veszi figyelembe a `defaultText`-et. Ha később a szülő megváltoztatja a `defaultText` propot, az input mező **NEM** fog frissülni magától (mert a state már létrejött). Ezt sokan elrontják.

---

# 4. Komplex Állapot: useReducer (23-27. dia)

Ha a `useState` már kevés (pl. sok összefüggő adat van, bonyolult logika), akkor jön a `useReducer`. Ez gyakorlatilag a "Redux" minta kicsiben.

**Kód elemzése (25-26. dia):**

1.  **A Reducer függvény (az "agy"):**
    ```javascript
    function reducerPerson( state, action ) {
        switch ( action.type ) {
            case "age": return { age: action.age }; // Új állapotot gyárt
            // ...
        }
    }
    ```
    *   Ez egy tiszta függvény. Megkapja a *régi* állapotot és egy *akciót* (utasítást), és visszaadja az *új* állapotot.

2.  **Használata a komponensben:**
    ```javascript
    let [ state, dispatch ] = useReducer( reducerPerson, { age: 10 } );
    ```
    *   `state`: A jelenlegi adat.
    *   `dispatch`: A "postás". Ezzel küldesz üzenetet (akciót) a reducernek.
    *   Használat: `dispatch({ type: "age", age: 12 })` -> "Hahó reducer, állítsd át a kort 12-re!"

---

# 5. Életciklus kezelés: useEffect (28-34. dia)

Ez a legnehezebb rész az osztály-alapú fejlesztőknek.
Osztályoknál voltak konkrét metódusok: "Amikor Létrejött", "Amikor Frissült", "Amikor Törlődött".
A React Hooks-ban **EGYETLEN** eszköz van mindenre: a `useEffect`.

**A szintaxis:**
```javascript
useEffect( () => {
    // 1. A kód, ami lefut (Effect)
    return () => {
        // 2. Takarító kód (Cleanup)
    };
}, [ függőség1, függőség2 ] ); // 3. Függőségi tömb (Dependency Array)
```

**Mikor fut le? (32. dia - EZ A KULCS):**
A **Függőségi tömb** (`[]`) dönti el, hogy mikor fut le az effekt.

1.  **`useEffect(..., [])` (Üres tömb):**
    *   Csak **EGYSZER**, az első megjelenéskor fut le.
    *   Ez felel meg a `componentDidMount`-nak (pl. inicializálás, adatletöltés).

2.  **`useEffect(...)` (Nincs tömb):**
    *   **MINDEN** egyes renderelés után lefut.
    *   Veszélyes, könnyű végtelen ciklust csinálni.

3.  **`useEffect(..., [adat])` (Van benne változó):**
    *   Az első rendereléskor lefut.
    *   Később csak akkor fut le újra, ha az `adat` változó értéke **megváltozott**.
    *   Ez felel meg a `componentDidUpdate`-nek.

**Takarító kód (33. dia):**
Ha a `useEffect` függvénye visszatér egy függvénnyel (`return () => ...`), az a **takarító kód**.
*   Ez akkor fut le, amikor a komponens megszűnik (pl. átlépünk másik oldalra), VAGY mielőtt az effekt újra lefutna.
*   **Mire jó?** Időzítők (`clearInterval`) leállítására, feliratkozások törlésére. Ha ezt lehagyod, memóriaszivárgás lesz (a háttérben ketyeg tovább az óra, akkor is, ha már nem látod).

**Példa kód (31. dia):**
```javascript
useEffect( () => {
    // Ez a kód elindít egy órát, ami másodpercenként frissíti a state-et
    let timer = setInterval( () => setTime(...) , 1000 );
    
    // Ez a kód állítja le az órát, ha a komponens eltűnik
    return () => clearInterval( timer );
}, [] ); // A [] miatt ez csak egyszer, az elején indul el.
```

---

### Összefoglalás "normális" programozói nyelven:

1.  **State (`useState`):** Memória, ami túléli a függvényhívásokat. Ha írod, a React újrarajzolja a képernyőt. Aszinkron módon működik!
2.  **Refs (`useRef`):** Hátsó ajtó a memóriához vagy a DOM-hoz. Ha írod, a React NEM rajzolja újra a képernyőt.
3.  **Inputok:** Kösd össze a State-tel (`value={state}`, `onChange={setState}`), így te kontrollálod, mi jelenik meg.
4.  **Lifecycle (`useEffect`):** Nem időpontokhoz (létrejött, frissült) kötjük a logikát, hanem **adatváltozáshoz**. "Ha megváltozik az 'X' adat, futtasd le ezt a kódot." (A dependency array irányítja).

Mehetünk a következőre (React kommunikáció a fában)?