Rendben, értem. Teljesen valid a frusztrációd. A React (és a deklaratív UI fejlesztés) egy **teljesen más mentális modell**, mint amit a klasszikus, imperatív (C++, Java, C#, WinForms) világban megszoktál.

A klasszikus világban te vagy a "főnök": te mondod meg, hogy `button.setText("Klikk")`, aztán `panel.add(button)`.
A React világban te csak egy "építész" vagy, aki lead egy tervrajzot, és egy "fekete doboz" (a React motor) dönti el, hogyan építse meg a házat.

Menjünk végig a **React alapok** diasoron (1-56. dia), és ízekre szedem neked, hogy mi történik a háttérben.

---

# React Alapok – A "Miért" és a "Hogyan" (1-7. dia)

### A Probléma: Miért nem jó a sima JavaScript? (3-5. dia)

A böngészőben a felületet a **DOM** (Document Object Model) írja le. Ez egy fa struktúra (HTML elemek).

**1. A "Favágó" módszer (`innerHTML`):**
```javascript
let name = "Leo";
div.innerHTML = `<span>Kedves, ${ name }!</span>`;
```
*   **Mit csinál?** Ez egy string összefűzés (template literal). A `${name}` helyére behelyettesíti a változót.
*   **Mi a baj?** A böngészőnek a szövegből (string) újra kell építenie a DOM elemeket.
    *   Ha van ott egy `input` mező, amibe írtál, és frissíted az `innerHTML`-t, a mező törlődik és újra létrejön. **Elveszik az állapot** (pl. a kurzor pozíciója).
    *   Lassú és villoghat.

**2. A "Sebész" módszer (DOM manipuláció):**
```javascript
let span = container.querySelector("div span.sum");
span.textContent = "Összeg: " + total;
```
*   **Mit csinál?** Megkeres egy konkrét elemet és csak a szövegét írja át.
*   **Mi a baj?** Ha a HTML szerkezete változik (pl. a `div`-ből `section` lesz), a kódod eltörik (`querySelector` nem talál semmit). Karbantarthatatlan spagetti kód lesz nagy projekteknél.

### A Megoldás: Struktúra szinkronizáció (6-7. dia)
A HTML egy fa. Ha változik az adat, a fát módosítani kell.
Két fa összehasonlítása matematikailag lassú ($O(n^3)$).
**A React trükkje:** Nem a valódi DOM-ot piszkálja, hanem létrehoz egy **Virtuális DOM**-ot (egy könnyű JS objektumot), és csak azt hasonlítgatja. Aztán kiszámolja a különbséget (diff), és csak a *szükséges minimumot* módosítja a valódi böngészőben.

---

# Komponens Alapú Gondolkodás (8-16. dia)

### Kompozíció (Composition) vs. Öröklés (Inheritance)
Programozóként megszoktad az öröklést (`class Dog extends Animal`). A Reactben **EZZEL LE KELL ÁLLNI**. Itt mindent egymásba ágyazással (kompozícióval) oldunk meg.

*   **Példa:** Nem csinálunk `SpecialButton` osztályt, ami örököl a `Button`-ból.
*   **Helyette:** Csinálunk egy `SpecialButton` komponenst, ami *tartalmaz* (renderel) egy `Button`-t, csak más paraméterekkel.

Ez a **Composite Design Pattern**. A lényeg: a felületet darabokra (komponensekre) szedjük. Egy komponens egy önálló egység (pl. egy Gomb, egy Kártya, egy Fejléc).

---

# A React Kód Felépítése (22-29. dia)

Itt jön a lényeg. Felejtsd el a HTML-t egy pillanatra. Ez **JavaScript**.

### A Hello World
```javascript
function HelloWorld(){
    return <h1>Hello, World!</h1>;
}
```
*   **Ez mi?** Ez egy **Függvény Komponens**. Egy sima JS függvény, ami visszaad valamit, ami HTML-nek néz ki.
*   **A furcsa szintaxis:** Az a `<h1>...</h1>` rész nem string, és nem is HTML. Ez **JSX** (JavaScript XML).

### Mi történik a háttérben? (23-25. dia)
A böngésző nem érti a JSX-et. Kell egy fordító (pl. **Babel**), ami átalakítja erre:

```javascript
function Greeter( p ) {
    // React.createElement(típus, tulajdonságok, tartalom...)
    return React.createElement( "h1", null, "Hello, ", p.name, "!" );
};
```
*   **Karakterről karakterre:**
    *   `function Greeter(p)`: A komponens egy függvény. A `p` (props) egy objektum, amiben a bemeneti adatok vannak.
    *   `return`: Visszaad egy **JavaScript Objektumot** (ezt hívjuk React Elementnek vagy Virtual DOM node-nak). NEM hoz létre valódi DOM elemet a böngészőben, csak egy leírást: "Szeretnék majd egy h1-et".

### Renderelés (24. dia)
```javascript
render( createElement( Greeter, { name: "Leo" } ), document.body );
```
*   Ez a sor mondja meg a Reactnak: "Vedd a `Greeter` komponensemet, a bemenő paramétere (props) legyen `{name: "Leo"}`, és az eredményt rakd be a `document.body`-ba".

### TSX - A Típusos JSX (26. dia)
Ha TypeScriptet használunk (és fogunk), akkor megadhatjuk, hogy a bemenő paraméter (props) milyen típusú legyen.
```typescript
// A 'p' paraméter egy objektum, aminek van egy 'name' tulajdonsága, ami string.
function Greeter( p: { name: string } ) {
    return <h1>Hello, { p.name }!</h1>;
}
```
*   `{ p.name }`: A JSX-ben a kapcsos zárójel `{}` jelenti azt, hogy "itt most JavaScript kód következik". Olyan, mint a string template `${}`-je.

### A React Fa (Virtual DOM) működése (27-29. dia)
1.  Meghívódik a függvényed (`Greeter`).
2.  Visszaad egy objektumot (`createElement` eredménye).
3.  A React összehasonlítja ezt az objektumot az előző állapottal.
4.  Ha változott, akkor a React nyúl hozzá a valódi HTML-hez.
    *   Ezért **TILOS** kézzel belepiszkálni a DOM-ba (`document.getElementById(...)`), mert a React a következő körben felülírja a változtatásaidat.

---

# Adatátadás: Props (Tulajdonságok) (30-36. dia)

A **Props** (Properties) a komponens bemeneti paramétere. Olyan, mint a függvény argumentuma. **Szülő adja a gyereknek.**

### Használata (31. dia)
```xml
<Greeter name="Leo" />
```
*   Ez a JSX kód a háttérben így néz ki: `Greeter({ name: "Leo" })`.
*   A `Greeter` függvény megkap egy objektumot: `{ name: "Leo" }`.

### Szabályok (32. dia)
*   **Read-only:** A komponens **NEM** módosíthatja a saját props-át. Azt kapta, kész.
*   **Bármi átadható:** Nem csak string. Szám, objektum, függvény, sőt, másik komponens is.
    *   Stringnél idézőjel: `name="Leo"`
    *   Minden másnál kapcsos zárójel: `age={25}`, `active={true}`, `user={{id: 1}}`.

### Props fogadása (33. dia)
A modern JS-ben "Destructuring"-ot (objektum szétbontást) használunk:

```typescript
// Sima fogadás
function Greeter(props) {
    return <h1>Hello, {props.name}!</h1>;
}

// Destructuring (Ezt használjuk 99%-ban!)
// Kapásból szedjük ki a 'name' mezőt a beérkező objektumból
function Greeter({ name }: { name: string }) {
    return <h1>Hello, { name }!</h1>;
}
```

### Opcionális Props (34. dia)
```typescript
function Header({ date = new Date(), name }: { date?: Date, name: string }) { ... }
```
*   `date?: Date`: A kérdőjel jelzi a TypeScriptnek, hogy ez nem kötelező.
*   `date = new Date()`: A függvény fejlécében alapértelmezett értéket adunk, ha `undefined` jönne be.

### A `children` prop (35. dia)
Ez egy speciális prop. Ez tartalmazza azt, amit a nyitó és záró tag közé írsz.
```xml
<Header name="Cím">
    <Greeter name="Leo" />  <-- Ez itt a children
</Header>
```
A `Header` komponensben így éred el:
```typescript
function Header({ name, children }) {
    return <div>
        <h1>{name}</h1>
        {children}  {/* Ide szúrja be a Greeter-t */}
    </div>
}
```

---

# Komponens Típusok: Osztály vs. Függvény (37-40. dia)

Kétféleképpen írhatsz komponenst.

**1. Osztály Komponens (Régi, de tudni kell róla):**
```typescript
class GreeterC extends Component<{name: string}> {
    render() {
        return <h1>Hello, {this.props.name}!</h1>;
    }
}
```
*   Örökölni kell a `Component`-ből.
*   Kell egy `render()` metódus.
*   Az adatokat a `this.props`-ban éred el.
*   **Hátrány:** Bonyolultabb kód, `this` kulcsszó körüli problémák.

**2. Függvény Komponens (Modern, ezt használd):**
Ez az, amit eddig néztünk. Csak egy függvény.
*   Nincs `this`.
*   Rövidebb.
*   (Későbbi anyag: Hook-okkal lesz állapota).

---

# Életciklus (Lifecycle) (41-44. dia)

A komponensek születnek (Mount), frissülnek (Update) és meghalnak (Unmount).

Osztály komponenseknél erre külön metódusok vannak:
*   `componentDidMount()`: Amikor a HTML bekerült a böngészőbe. Itt indíthatsz pl. időzítőt (`setInterval`).
*   `componentWillUnmount()`: Mielőtt kiveszik a HTML-ből. Itt takarítasz (pl. `clearInterval`), különben memóriaszivárgás lesz.
*   `render()`: Amikor frissíteni kell a kinézetet.

*A függvény komponenseknél ezt máshogy (Hooks - `useEffect`) kezeljük, de az a következő előadás anyaga lesz.*

---

# Renderelési Technikák (45-52. dia)

Mit adhat vissza a `render` függvény? Egy React Fát.

### Feltételes megjelenítés (Nagyon fontos!) (48. dia)
Hogyan írsz `if`-et JSX-ben? Mivel a `{}` belsejébe csak *kifejezés* (expression) írható, és az `if` utasítás (statement), ezért trükközni kell.

**1. A `&&` (ÉS) operátor:**
```javascript
// Ha a location létezik (igaz), akkor kiírja a span-t.
// Ha nem létezik (hamis), a React nem rajzol semmit.
{ location && <span>Welcome to { location }!</span> }
```

**2. A Ternary (hármas) operátor (`? :`):**
```javascript
// Ha string, akkor simán kiírja. Ha nem, akkor <time> komponenst használ.
{ typeof time === "string" ? time : <time value={...} /> }
```

### Listák megjelenítése (49-50. dia)
Hogyan írsz `for` ciklust JSX-ben? `map()` függvénnyel! A `map` átalakítja az adat-tömböt ReactElem-tömbbé.

```javascript
<ul>
  { items.map( item => <li key={item.id}>{item.name}</li> ) }
</ul>
```

**A `key` attribútum (KRITIKUS FONTOSSÁGÚ):**
*   Amikor listát renderelsz, a Reactnak tudnia kell, melyik elem melyik, hogy ha változik a sorrend, ne törölje és hozza létre újra az egészet, hanem csak mozgassa a DOM elemeket.
*   **Szabály:** Minden listaelemnek legyen egyedi `key`-e (pl. adatbázis ID).
*   **Hiba:** Ne használd a tömb indexét (`i`) kulcsnak, ha változhat a lista sorrendje, mert az összezavarja a Reactot (az 1. elem mindig az 1. indexű marad, hiába szúrtál be elé valamit).

### Fragment (51. dia)
Egy komponens csak **EGY** gyökérelemet adhat vissza.
Rossz:
```javascript
return <h1>Cím</h1> <p>Szöveg</p>; // Hiba! Két elem.
```
Megoldás 1: Csomagold be `div`-be. (De ez felesleges szemetet rak a DOM-ba).
Megoldás 2: Használj Fragmentet (`<> ... </>`). Ez egy láthatatlan konténer.
```javascript
return <>
    <h1>Cím</h1>
    <p>Szöveg</p>
</>; // Jó!
```

---

# Attribútumok (53-55. dia)

A JSX attribútumok 99%-ban megegyeznek a HTML attribútumokkal, de van pár kivétel, mert a JS-ben foglalt szavak:

*   `class` helyett **`className`**-et kell írni (mert a `class` osztályt jelent JS-ben).
*   `for` helyett **`htmlFor`**-t (mert a `for` ciklus).
*   `style`-nak nem stringet adsz, hanem objektumot: `style={{ color: 'red' }}`.

**Feltételes attribútumok:**
```javascript
// Ha a condition igaz, a 'disabled' attribútum értéke true lesz.
<input disabled={ condition } />
```
Vagy "Spread operator" (`...`): Ha van egy objektumod tele attribútumokkal, ráöntheted a komponensre:
```javascript
let attrs = { disabled: true, type: "text" };
<input { ...attrs } />
```

---

### Összefoglalva "normális" programozóknak:

1.  A **Komponens** egy Függvény.
2.  A **Props** a függvény paramétere (Objektum).
3.  A **JSX** nem HTML, hanem egy szintaxis, ami JS objektumokat hoz létre (`React.createElement`).
4.  A **Render** nem rajzol, csak visszaad egy adatszerkezetet (tervrajzot).
5.  A **React Motor** összehasonlítja a régi és új tervrajzot, és frissíti a DOM-ot helyetted.
6.  **Nincs if/for** a JSX-ben, helyette `&&` (feltétel) és `.map()` (ciklus) van.

Ha ez megvan, küldheted a következő diasort (valószínűleg a State/Hooks témakört)!