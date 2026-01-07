Íme a „Kliensoldali rendszerek” tárgy **React kommunikáció a fában** című előadásának részletes feldolgozása.

---

# Kliensoldali Rendszerek – React Kommunikáció és Állapotkezelés

Ez az előadás arról szól, hogyan osztunk meg adatokat a komponensek között, hogyan kezeljük a globális állapotokat, és hogyan kommunikálhatunk a fában le- és felfelé.

## 1. Szétosztott Állapot és a "Lifting State Up" (2-4. dia)

### A probléma
Gyakran előfordul, hogy két testvér komponensnek (pl. `ChildA` és `ChildB`) ugyanazt az adatot kell használnia. Mivel a Reactben az adatfolyam alapvetően egyirányú (lefelé), a testvérek nem tudnak közvetlenül beszélgetni.

### A megoldás: Lifting State Up (Állapot felemelése)
*   **Módszer:** Az állapotot kivesszük a gyerekekből, és felmozgatjuk a **legközelebbi közös ősbe** (szülőbe).
*   **Adatlefolyás:** A szülő `props`-ként adja le az adatot a gyerekeknek.
*   **Visszajelzés:** Ha egy gyerek (pl. `ChildA` input mezője) módosítani akar, egy a szülőtől kapott callback függvényt (`onChange`) hív meg.
*   **Előny:** Jól működik lokális állapotoknál, kevés komponens esetén.
*   **Hátrány ("Prop Drilling"):** Ha az adatot nagyon mélyre kell leküldeni, sok köztes komponensen kell átvezetni a props-ot, amiknek amúgy semmi közük azadathoz. Ez csúnya és nehezen karbantartható kódot eredményez.

---

## 2. Globális Állapot (5-12. dia)

Ha az adatot az alkalmazás számos pontján el kell érni (pl. felhasználó adatai, téma), a "Lifting State" nem praktikus.

### Működési elv
*   Az állapotot egy React-en kívüli **globális változóban** (Store) tároljuk.
*   **Feliratkozás:** Mivel a React nem lát rá a külső változókra, gondoskodni kell értesítésről.
    *   Kell egy lista a feliratkozókról (listeners).
    *   Ha az adat változik, minden feliratkozott komponenst értesíteni kell (hogy rendereljenek újra).
*   Ez hasonlít a DOM `addEventListener` működéséhez.

### Implementáció (`GlobalStore` példa)
*   **Osztály:** Tárolja az értéket (`#value`) és a feliratkozókat (`#listeners`).
*   **Subscribe:** Feliratkozó függvény, ami visszaad egy `unsubscribe` (leiratkozó) függvényt. Fontos az *immutable* lista kezelés (másolat készítése feliratkozáskor), hogy iterálás közben ne legyen gond.
*   **SetValue:** Beállítja az új értéket és meghívja az összes listenert. (Hasznos a `Object.is` összehasonlítás a felesleges frissítések elkerülésére).

### React integráció: `useSyncExternalStore`
Ez a modern, hivatalos React hook külső tárolók (store-ok) bekötésére.
*   Paraméterei: `subscribe` függvény és `getSnapshot` (érték olvasó) függvény.
*   Automatikusan kezeli a fel- és leiratkozást, valamint a frissítést.

---

## 3. Context API (13-16. dia)

A Context egy beépített React mechanizmus az "öröklődésre" a fában, a prop drilling elkerülésére.

### Jellemzők
*   **Globális jelenlét, lokális érték:** Mindenhol elérhető, de az értéke a fában elfoglalt helytől függhet (pl. téma).
*   **Működés:**
    1.  `createContext(defaultErtek)`: Létrehozás.
    2.  `<Context.Provider value={...}>`: Érték "sugárzása" egy részfára.
    3.  `useContext(Context)`: Érték kiolvasása bármilyen mélyen lévő gyerekben.
*   **Írható Context:** Ha nem csak olvasni akarjuk, a Context-be egy objektumot vagy tömböt teszünk, ami tartalmazza az **értéket** ÉS a **módosító függvényt** (setter) is.

---

## 4. `useRef`: A "Tagváltozó" (17-20. dia)

Függvény komponenseknél nincs `this`, így nehéz adatot tárolni két render között anélkül, hogy újrarajzolást váltanánk ki. Erre való a `useRef`.

*   **Működés:** Visszaad egy objektumot: `{ current: ... }`.
*   **Stable Identity:** Minden renderelésnél ugyanazt a referencia objektumot kapjuk vissza.
*   **Nem okoz rendert:** A `.current` tulajdonság írása nem vált ki újrarajzolást (ellentétben a `useState`-tel).
*   **Felhasználás:**
    *   HTML elemek (DOM) közvetlen elérése.
    *   Cache-elt adatok, időzítő ID-k tárolása.
    *   Számított értékek megtartása.
*   **Tisztaság (Purity):** Fontos szabály, hogy renderelés *közben* (a render fázisban) ne írjuk/olvassuk a ref-et, mert az mellékhatásnak minősül. Használjuk `useEffect`-ben vagy eseménykezelőkben.

---

## 5. Logika kiszervezése: Custom Hooks (21-32. dia)

A modern React célja a funkcionális programozáshoz hasonló, tiszta komponensek létrehozása.

### Hook szabályok és saját hook-ok
*   A hook-ok (pl. `useState`) sorrendje számít, nem lehetnek feltételes blokkban.
*   **Saját Hook:** Ha van egy logikánk (pl. számláló növelése, ablakméret figyelése), kiszervezhetjük egy `use...` kezdetű függvénybe.
*   **Állapot tulajdona:** Bár a hook-ban van a `useState`, az állapot valójában ahhoz a komponenshez tartozik, **amelyik meghívja** a hook-ot. Minden hívásnál új, független állapot jön létre.
*   **Dekompozíció:** Segít a kódot olvashatóbbá tenni és újrahasznosítani.

### Aszinkronitás és React
*   A React renderelése és a hook-ok szinkron működésűek.
*   `useEffect` nem fogad el `async` függvényt közvetlenül.
*   **Megoldás:** Az `useEffect`-en belül definiálunk egy async függvényt (vagy IIFE-t) és azt hívjuk meg.
*   **Adatletöltés (Fetch):** Figyelni kell a komponens életciklusára. Ha a letöltés alatt a komponens megszűnik (unmount), nem szabad `setState`-et hívni (memóriaszivárgás/hiba lehet). Megoldás: egy `mounted` flag használata a `useEffect`-ben.

---

## 6. Referencia továbbadása: `forwardRef` (33-36. dia)

Alapértelmezésben a `ref` attribútum csak HTML elemekre tehető, React komponensekre nem (mert nem egyértelmű, mihez akarunk hozzáférni).

*   **forwardRef:** Egy "csomagoló" (HOC), ami lehetővé teszi, hogy egy komponens megkapja a szülőtől érkező `ref`-et második paraméterként.
*   **Használat:** A gyerek komponens ezt a kapott ref-et ráteszi egy belső HTML elemére (pl. input). Így a szülő közvetlenül elérheti a gyerek belső input mezőjét (pl. fókuszáláshoz).
*   **Korlátok:** A szülő csak a DOM elemet éri el, a gyerek komponens belső állapotát vagy függvényeit nem (ez az egységbe zárás védelme).

---

## 7. Komponens átadása (Composition Patterns) (37-43. dia)

Hogyan tehetünk egy komponenst rugalmassá, hogy a tartalmát kívülről határozhassuk meg (pl. egy `MessageBox` ikonja vagy gombjai)?

### 1. `children` prop (Alapértelmezett)
Ez a leggyakoribb. Amit a nyitó és záró tag közé írunk, az `children`-ként érkezik.

### 2. Komponens mint prop (Egyéb megoldások)
Ha nem a tartalmat, hanem pl. egy ikont vagy gombot akarunk cserélhetővé tenni:

*   **A) React Elem átadása:**
    *   Kód: `<MessageBox icon={<WarningIcon />} />`
    *   Előny: Egyszerű.
    *   Hátrány: Nehéz a `MessageBox`-on belül paraméterezni (pl. ha a `MessageBox` akarja megadni az ikon színét). `cloneElement` kellhet hozzá.
*   **B) Komponens Típus átadása:**
    *   Kód: `<MessageBox icon={WarningIcon} />` (nem példány, hanem a függvény/osztály).
    *   A `MessageBox` belül példányosítja: `<IconComponent color="red" />`.
    *   Hátrány: A props-okat külön objektumban kell átadni.
*   **C) Render Prop (Függvény):**
    *   Kód: `<MessageBox icon={() => <WarningIcon />} />`
    *   A `MessageBox` hívja meg a függvényt rendereléskor.
    *   Előny: Ez a legrugalmasabb ("Lazy evaluation"). A belső kód tud paramétereket átadni a függvénynek.

Minden megoldásnak van létjogosultsága a feladattól függően.