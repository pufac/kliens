Rendben, itt az utolsó, negyedik adag. Ez a **sebességről** szól. Ha lassú az alkalmazásod, az emberek utálni fogják.

De vigyázz! Nem mindegy, **MI** lassú.

---

# React Render és Sebesség

## 1. Kétféle Sebesség (3-4. dia)

Két dolog lehet lassú, és nem mindegy melyik:

1.  **React Render (Agyi munka):** Amikor a React számolgatja a memóriában, hogy "hmmm, változott az adat, mit is kéne átírni a képernyőn?".
    *   **Ha ez lassú:** Megakad a program (lag), nem reagál a gombokra.
2.  **HTML Render (Böngésző munka):** Amikor a böngésző ténylegesen kirajzolja a pixeleket (színek, árnyékok, elrendezés).
    *   **Ha ez lassú:** Darabos az animáció, szaggat a görgetés (alacsony FPS).

**Tűréshatár:**
*   A Reactnek (számolás) **nagyon gyorsnak** kell lennie. Ha van 10.000 elemed a fában, és mindet újra kell számolni, az másodpercekig tarthat -> Kuka.
*   A HTML megjelenítés (rajzolás) többet bír, de ott is érdemes spórolni (ne legyen 1 millió `div` a képernyőn egyszerre).

---

# 2. Virtualizáció - A Nagy Listák Titka (7. dia)

**Probléma:** Van egy táblázatod 100.000 sorral. Ha ezt mind kirajzolod (`.map`-el), a böngésző összeomlik.
**Megoldás:** **Virtualizáció**.
*   Csak azt az 50 sort rajzoljuk ki, ami épp **LÁTSZIK** a képernyőn.
*   Ha görgetsz, a felső sorokat eldobjuk, és alulra újakat rajzolunk be.
*   A felhasználó azt hiszi, ott az egész lista, de a memóriában csak töredéke van.

**Fontos szabály:** Amit nem lát a user (pl. másik fülön van), azt **NE** rejtsd el `display: none`-nal!
*   `display: none`: A React ugyanúgy kiszámolja és berakja a DOM-ba, csak láthatatlan. Ez lassú.
*   **Helyes:** `if (aktivFül) <Tartalom />`. Ha nem aktív, be se kerüljön a fába.

---

# 3. Render Optimalizáció: `memo` (8-10. dia)

**Hogyan működik a React?**
Ha a Szülő (`Parent`) újrarenderelődik (pl. mert változott az állapota), akkor **MINDEN** gyereke (`Child`) is automatikusan újrarenderelődik. Akkor is, ha a gyereknek semmi köze a változáshoz!

Ez nagy fánál lassú.

**Megoldás: `memo`**
A `memo` egy "őr" a komponens előtt. Megnézi: "Változott a bemenő adat (props)? Nem? Akkor nem engedlek újrarenderelődni!"

```javascript
// A Child csak akkor fut le újra, ha a props változott.
const Child = memo( () => <div>Én egy drága komponens vagyok</div> );
```

---

# 4. A `memo` buktatói: Instabil adatok (11-13. dia)

A `memo` csak akkor működik, ha a props **tényleg** nem változik. De JavaScriptben ez trükkös.

**A Probléma:**
```javascript
function Parent() {
    // Ez a függvény MINDEN rendereléskor ÚJRA létrejön a memóriában!
    // Ezért a Child azt hiszi, hogy "új" adatot kapott, és újrarenderel.
    const onClick = () => console.log("Katt");

    return <Child onClick={onClick} />; // A memo itt hatástalan!
}
```

**Megoldás 1: `useCallback` (Függvényekre)**
"Emlékezz erre a függvényre, és ne csinálj újat minden körben!"
```javascript
// Csak egyszer jön létre (a [] miatt).
const onClick = useCallback( () => console.log("Katt"), [] );
```

**Megoldás 2: `useMemo` (Adatokra)**
Ha egy bonyolult objektumot vagy tömböt adsz át, azt is "meg kell jegyezni".
```javascript
// Csak akkor szűri újra a listát, ha az 'items' változik.
const filteredItems = useMemo( () => items.filter(x => x > 10), [items] );
```

---

# 5. `key` attribútum - A lista lelke (18-19. dia)

Amikor listát renderelsz (`.map`), a Reactnak szüksége van egy `key`-re.
```javascript
items.map( item => <ListItem key={item.id} /> )
```

**Miért fontos?**
*   Ha beszúrsz egy elemet a lista elejére, a React `key` nélkül azt hinné, hogy minden elem megváltozott (mert elcsúszott az index).
*   `key`-jel tudja: "Áhá, a 42-es ID-jű elem eddig első volt, most második. Nem törlöm, csak arrébb rakom." Sokkal gyorsabb!

**Trükk:** A `key` nem csak listában jó!
Ha egy komponensnek megváltoztatod a `key`-ét, a React **eldobja a régit és újat csinál** helyette.
*   **Mire jó ez?** Ha van egy bonyolult űrlapod, és "Reset" gombot akarsz: változtasd meg a form `key`-ét, és minden állapota törlődik, mert teljesen új példány jön létre.

---

# 6. `createPortal` - A "teleport" (20-21. dia)

Néha a HTML-ben máshol akarsz megjeleníteni valamit, mint ahol a React kódban vagy.
Pl.: Egy **Modális ablakot** (felugró ablak) vagy Tooltipet. Ezeket közvetlenül a `<body>` végére érdemes rakni, hogy ne takarja ki őket semmi (z-index gondok).

```javascript
function Parent() {
    return createPortal(
        <Modal>Ez egy ablak</Modal>,
        document.body // Ide "teleportálja" a HTML-t
    );
}
```
A React logikában a `Modal` a `Parent` gyereke marad (működnek az események), de a böngészőben a `body`-ban lesz.

---

# 7. Animációk: `requestAnimationFrame` (22-23. dia)

Ha játékot írsz vagy nagyon sima animációt (60 FPS) akarsz (pl. egérkövetés), a React lassú lehet. A React render nem garantálja, hogy minden képkockára lefut.

**Helyes sorrend animációnál:**
1.  **CSS:** Ha lehet, használd a CSS `transition` vagy `animation` tulajdonságait. Ez a leggyorsabb (GPU gyorsított).
2.  **Közvetlen DOM:** Ha bonyolultabb (pl. egér X, Y koordináta), használd a `requestAnimationFrame`-et, és `useRef`-fel közvetlenül állítsd a DOM elem stílusát. **NE** `setState`-et hívj minden pixelmozdulatra!
    ```javascript
    // Rossz (lassú):
    onMouseMove={e => setPosition({x: e.x, y: e.y})}
    
    // Jó (gyors):
    divRef.current.style.transform = `translate(${e.x}px, ${e.y}px)`;
    ```
3.  **React Render:** Csak végső esetben, és akkor se minden frame-ben (pl. csak másodpercenként 10-szer).

---

### Összefoglalva "agybeteg" módon (a kérésed szerint):

1.  **Lassú?** Két fajta lassúság van:
    *   **React agyalása:** Túl sok komponenst vizsgál át. -> Használj `memo`-t!
    *   **Böngésző rajzolása:** Túl sok HTML elemet gyártasz. -> Használj virtualizációt (csak ami látszik)!
2.  **`memo`:** "Állj! Csak akkor dolgozz, ha új melót (props) kapsz!"
3.  **`useCallback` / `useMemo`:** "Ezt az adatot/függvényt tedd el a fiókba, és vedd elő ugyanazt, amíg nem muszáj cserélni." (Hogy a `memo` ne hisztizzen).
4.  **`key`:** "Hahó React, ez a Pistikés doboz, ez meg a Józsikás. Ne keverd össze őket, ha sorba állítom őket!"
5.  **Animáció:** Ne hívj `setState`-et másodpercenként 60-szor, mert megöli a gépet. Nyúlj be a motorháztető alá (`ref`), vagy bízd a CSS-re.