Íme a „Kliensoldali rendszerek” tárgy **React állapot- és eseménykezelés** témakörű előadásának részletes feldolgozása a diák alapján.

---

# Kliensoldali Rendszerek – React Állapot- és Eseménykezelés

## 1. Bevezetés: Osztály vs. Függvény Komponensek (1-3. dia)

A React fejlődése során a fejlesztés súlypontja eltolódott az osztály alapú megközelítéstől a függvény alapúig.

### Függvény Komponensek (Functional Components) és Hooks
*   **Miért váltunk?** Sok fejlesztő szerint az Objektum Orientált Programozás (OOP) felesleges komplexitást visz a UI fejlesztésbe. A függvény komponensek kódja tömörebb és jobban olvasható.
*   **Működés:** A függvény komponens logikailag megfelel az osztály komponens `render` függvényének. Önmagában "nem tud többet", de a **Hook**-ok segítségével képessé válik mindenre, amit egy osztály tud.
*   **Mik azok a Hook-ok?** Olyan speciális függvények, amelyekkel "beakaszkodhatunk" a React működésébe:
    *   `useState`: Állapotkezelés.
    *   `useReducer`: Komplex állapotkezelés.
    *   `useEffect`: Életciklus események kezelése.
    *   `useRef`: Belső változók és DOM referenciák kezelése.

---

## 2. Állapotkezelés (State) (4-9. dia)

Az állapot (state) az az adat, amely a komponens életciklusa alatt változhat, és amelynek változása újrarajzolást (re-render) vált ki.

### Állapot általános jellemzői
*   **Belső állapot:** A komponens saját adata, ellentétben a `props`-szal, amit kívülről kap. (Léteznek állapotmentes komponensek is).
*   **Perzisztencia:** Az értéke megmarad a rajzolások között.
*   **Osztály komponenseknél (Régi módszer):**
    *   A konstruktorban inicializáljuk: `state = { name: "" };`
    *   Kizárólag a `setState` függvénnyel módosítható (pl. `this.setState({ c: this.state.c + 1 })`).
    *   A `setState` hívás váltja ki az újrarajzolást.

### A `setState` aszinkron működése
*   A `setState` nem azonnal írja át az értéket. Optimalizációs okokból a React "összevárhatja" a módosításokat, és a folyamat végén egyszerre frissít.
*   **Veszély:** Ha az új állapot az előző állapottól függ (pl. számláló növelése), és többször hívjuk meg egymás után, a sima objektumos beállítás hibás lehet (régi értéket olvas).
*   **Megoldás:** A `setState`-nek függvényt adunk át: `this.setState(state => { c: state.c + 1 })`. Így biztosan a legfrissebb előző állapotot kapjuk meg.

### State Merge (Csak osztályoknál!)
*   Osztály komponensnél a `setState` "sekélyen" (shallow) összefésüli az objektumokat. Ha a state `{ a: 1, b: 2 }`, és mi hívunk egy `setState({ a: 10 })`-et, a `b` tulajdonság érintetlen marad. **Ez a Hook-oknál máshogy működik!**

---

## 3. Állapotkezelés Hook-okkal: `useState` (10-13. dia)

Függvény komponenseknél a `useState` hook biztosítja az állapotot.

### Használata
```javascript
let [ date, setDate ] = useState( defaultDate ?? new Date );
```
1.  **Hívás:** `useState(kezdoErtek)` – megadjuk a kezdőértéket.
2.  **Visszatérési érték:** Egy két elemű tömböt ad vissza (Destructuring-al szedjük szét):
    *   1. elem (`date`): Az aktuális érték.
    *   2. elem (`setDate`): A beállító függvény.
3.  **Frissítés:** A beállító függvény meghívása (`setDate(ujErtek)`) újrahívja a komponenst, ahol a `date` változó már az új értéket fogja tartalmazni.

### Működési mechanizmus (Mental Model)
*   A háttérben a React nyilvántartja az állapotokat egy tömbben.
*   A komponens újrafuttatásakor a `useState` hívások **sorrendje** alapján tudja a React, hogy melyik érték melyik változóhoz tartozik.
*   **Szabályok (Rules of Hooks):**
    *   A `useState` hívások **nem lehetnek feltételesek** (nem tehetjük `if`-be).
    *   A sorrendjük nem változhat.
    *   Mindig a függvény legtetején kell lenniük, vezérlési szerkezetek (ciklus, elágazás) nélkül.

---

## 4. Eseménykezelés és Beviteli Mezők (14-22. dia)

### Események (Events)
*   A HTML elemek eseményeire `onEsemenyNev` formátumban iratkozhatunk fel (PascalCase, pl. `onClick`, `onInput`).
*   Több száz esemény érhető el, ugyanúgy, mint natív JS-ben.

### Beviteli típusok (Input)
*   Szövegdoboz (`text`), jelölőnégyzet (`checkbox`), rádiógomb, fájlválasztó, legördülő lista (`select`), stb.
*   Ezek kezelése eltérő lehet (pl. `value` vs `checked` tulajdonság).

### Állapot tárolásának helye
Kétféle megközelítés létezik az űrlap elemek kezelésére:

**1. State-ben tárolt állapot (Controlled Component) - Javasolt**
*   Minden billentyűleütésre (`onInput` vagy `onChange`) frissítjük a React state-et.
*   Az input mező `value` attribútumát a state-ből kötjük vissza: `<input value={text} />`.
*   **Előny:** Validálhatunk gépelés közben, segíthetjük a bevitelt, teljes kontrollunk van.
*   **Megjegyzés:** React-ben az `onChange` esemény minden változáskor lefut (úgy viselkedik, mint a szabványos `onInput`), nem csak fókuszvesztéskor.

**2. DOM-ban tárolt állapot (Uncontrolled Component)**
*   Nem tároljuk az értéket state-ben, "hagyjuk", hogy a böngésző kezelje.
*   Ha szükség van az adatra (pl. űrlap küldésekor), egy referencián (**useRef**) keresztül olvassuk ki közvetlenül a DOM-ból.
*   **useRef:** Olyan, mint egy "doboz", ami megőrzi tartalmát renderelések között, de változása nem vált ki újrarajzolást.
    *   `inputRef.current` az első renderkor még `null`, utána elérhető a DOM elem.
*   **Kivétel:** A `<input type="file">` elemet csak így (vagy eseménykezelőből) lehet kezelni, mert a `value`-ja írásvédett biztonsági okokból.
*   Kezdőérték megadása DOM-os módban: `defaultValue`.

---

## 5. Komplex Állapotkezelés: `useReducer` (23-27. dia)

Ha az állapotkezelő logika bonyolult, vagy több helyen újra akarjuk használni, érdemes kiszervezni a komponensből. Erre való a `useReducer`.

### Reducer függvény
*   Ez egy tiszta függvény: `(régiÁllapot, akció) => újÁllapot`.
*   Az `action` egy objektum, ami leírja, mi történt (pl. `{ type: "increment" }`).
*   A függvény egy `switch-case` szerkezettel dönti el, hogyan változzon az állapot.

### Használata
```javascript
let [ state, dispatch ] = useReducer( reducerFuggveny, kezdoAllapot );
```
*   A `dispatch` függvénnyel küldünk üzeneteket (akciókat), pl.: `dispatch({ type: "age", val: 20 })`.
*   Érdekesség: A `useState` a háttérben valójában egy egyszerűsített `useReducer`-t használ.

---

## 6. Életciklus Kezelés: `useEffect` (28-34. dia)

Osztály komponenseknél külön függvények voltak az életciklus eseményekre (`componentDidMount`, `componentDidUpdate`, `componentWillUnmount`). Függvény komponenseknél ezt egyetlen hook, a **useEffect** helyettesíti.

### Működése
A `useEffect` egy függvényt vár, amit a renderelés **után** futtat le.

### 1. Paraméter: A mellékhatás (Effect)
Ide kerül a kód, amit futtatni akarunk (pl. időzítő indítása, adatlekérés, feliratkozás).

### 2. Paraméter: Függőségi tömb (Dependencies)
Ez szabályozza, mikor fusson le az effekt:
*   **Nincs megadva:** Minden egyes renderelés után lefut (`componentDidUpdate`).
*   **Üres tömb `[]`:** Csak az első renderelés után fut le egyszer (`componentDidMount`).
*   **`[valtozo]`:** Az első renderelés után, ÉS akkor, ha a `valtozo` értéke megváltozott.

### Takarító kód (Cleanup function)
*   A `useEffect`-nek átadott függvény visszatérhet egy másik függvénnyel. Ez a **takarító kód**.
*   **Mikor fut le?**
    1.  Mielőtt a komponens végleg törlődik a DOM-ból (`componentWillUnmount`).
    2.  Mielőtt az effekt újra lefutna (ha változott a függőség), hogy eltakarítsa az előző futás maradványait.
*   **Példa:** `setInterval` indítása az effektben, `clearInterval` a takarító kódban.

### Összefoglalva a `useEffect` viselkedése
Mindig párban működik a fő effekt és a takarító kód:
1.  Első render -> Effekt lefut.
2.  Változás (update) -> Takarító kód lefut (előző köré) -> Új Effekt lefut.
3.  Törlés (unmount) -> Takarító kód lefut.

---

## Összegzés
A React modern fejlesztése a függvény komponensekre és a Hook-okra épül.
*   **useState:** Egyszerű adatok tárolása, UI frissítés.
*   **useReducer:** Bonyolultabb állapotlogika kiszervezése.
*   **useRef:** DOM elemek elérése, értékek tárolása újrarajzolás nélkül.
*   **useEffect:** Életciklus események és mellékhatások (pl. időzítők, API hívások) kezelése, beleértve a takarítást is.