Megérkezett a harmadik pakk. Ez a diasor arról szól, hogy **ha már vannak komponenseid, hogyan beszélgessenek egymással?**

A "normális" programozásban (OOP) ha van egy `A` objektumod és egy `B` objektumod, akkor `A.method()` meghívja `B`-t, vagy `B` feliratkozik `A` eventjére. Kész.
Reactben ez **tilos**. Itt a fa-struktúra diktál.

Nézzük végig, milyen lehetőségeid vannak az adatok mozgatására.

---

# 1. Lokális adatmegosztás: "Lifting State Up" (2-4. dia)

**A probléma:** Van két testvér komponensed (`ChildA` és `ChildB`). `ChildA`-ban beírsz valamit, és azt `ChildB`-ben meg kéne jeleníteni.
Mivel a React adatfolyam **mindig fentről lefelé** csorog (szülő -> gyerek), a testvérek nem látják egymást.

**A megoldás (4. dia kódja):**
Az állapotot (state) **kivesszük** a gyerekekből, és felrakjuk a **szülőbe** (Parent). Ez a "Lifting State Up".

```javascript
function Parent(){
    // 1. A SZÜLŐ tárolja az adatot!
    let [ text, setText ] = useState( "" ); 
    
    return <div>
        {/* 2. A szülő leküldi az adatot (text) és a módosító függvényt (setText) */}
        <ChildA text={ text } onChange={ setText } />
        
        {/* 3. A másik gyerek is megkapja az adatot */}
        <ChildB text={ text } />
    </div>
}

function ChildA( { text, onChange }: { ... } ){
    // 4. A gyerek NEM tárolja az adatot, csak kapja.
    // Ha írnak bele (onInput), meghívja a SZÜLŐTŐL kapott függvényt.
    return <input value={ text } onInput={ e => onChange( e.currentTarget.value ) } />
}

function ChildB( { text }: { text: string } ){
    // 5. Ő csak megjeleníti, amit fentről kapott.
    return <span>{ text }</span>
}
```
*   **Analógia:** Két gyerek veszekszik egy játékon. Az anyuka (Szülő) elveszi tőlük, ő fogja (State), és odaadja annak, aki épp játszani akar vele (Props).

---

# 2. Globális Állapot (5-12. dia)

**A probléma:** Mi van, ha az adatot 10 szinttel mélyebben kell használni? Vagy teljesen független komponenseknek kell ugyanaz az adat (pl. User adatok)?
Ha mindent "Lifting State"-tel oldanál meg, akkor a gyökér komponensben lenne 500 state változó, és minden komponensen át kéne fűzni ("Prop Drilling").

**A megoldás:** Egy React-en kívüli, globális objektum (Singleton).

**A trükk (Observer minta):**
Mivel a React nem lát rá a sima JS változókra, értesíteni kell őt, ha változás van.

**Kód elemzés (9-10. dia - GlobalStore osztály):**
Ez egy sima TypeScript osztály, semmi React nincs benne.
```typescript
class GlobalStore<T> {
    #listeners: ( () => void )[] = []; // Itt tároljuk a feliratkozott komponenseket
    #value: T; // Ez maga az adat

    // ... constructor ...

    // Feliratkozás: A komponens ad egy callbacket ("szólj, ha változás van")
    subscribe( callback: () => void ) {
        this.#listeners = [ ...this.#listeners, callback ]; // Hozzáadjuk a listához
        // Visszaadunk egy takarító függvényt (leiratkozás)
        return () => this.#listeners = this.#listeners.filter( x => x !== callback );
    }

    // Adat módosítása
    setValue( v: T ){
        if ( !Object.is( this.#value, v ) ){ // Csak ha tényleg változott
            this.#value = v;
            // Értesítünk mindenkit! (Itt fog a React renderelni)
            for ( let listener of this.#listeners ) listener();
        }
    }
}
```

**Hogyan kötjük be Reactbe? (12. dia):**
Van egy speciális hook erre: `useSyncExternalStore`.
```javascript
const globalProp = new GlobalStore( 12 ); // Létrehozzuk a globális tárolót

export function Comp() {
    // Ez a hook "összedrótozza" a külső osztályt a React render ciklusával.
    // 1. subscribe: Hogyan kell feliratkozni?
    // 2. getSnapshot: Hogyan kell kiolvasni az adatot?
    let value = useSyncExternalStore( globalProp.subscribe, globalProp.getValue );
    
    return <span>{ value }</span>
}
```
*   Amikor a `globalProp.setValue`-t meghívja valaki bárhol az appban, ez a komponens automatikusan újrarenderelődik.

---

# 3. Context API (13-16. dia)

Ez a React beépített megoldása a "globális" adatokra, de fa-struktúrához kötve. Olyan, mint a környezeti változók.

**Működése (15. dia):**
1.  **Létrehozás:** `const ThemeContext = createContext(null);`
2.  **Szolgáltatás (Provider):** Egy komponens "sugározza" az adatot lefelé.
    ```javascript
    <ThemeContext.Provider value={ "dark" }>
       {/* Mindenki ezen a részfán belül látni fogja a "dark"-ot */}
       <App />
    </ThemeContext.Provider>
    ```
3.  **Fogyasztás (Consumer):** Bárhol mélyen a fában:
    ```javascript
    const theme = useContext( ThemeContext ); // "dark" lesz az értéke
    ```

**Írható Context (16. dia):**
Ha módosítani is akarjuk lentről, akkor nem csak az értéket ("dark"), hanem egy módosító függvényt is beteszünk a Context-be.
```javascript
// A Context értéke egy tömb: [érték, módosítóFüggvény]
const ThemeContext = createContext([ "dark", () => {} ]);
```

---

# 4. `useRef` - A "hátsó ajtó" (17-20. dia)

Ez az, amit OOP programozóként keresel: a **tagváltozó** (instance variable).

**Mi a `useRef`?**
*   Egy "doboz", amibe bármit tehetsz.
*   **A tartalma (`.current`) megmarad** a renderelések között (mint a `state`).
*   **DE: Ha írod, NEM renderel újra a React!** (Ez a lényeges különbség a `useState`-hez képest).

**Mire jó?**
1.  **DOM elérés (19. dia):** Ha közvetlenül akarsz piszkálni egy HTML elemet (pl. fókuszálás, mérés).
    ```javascript
    let inputRef = useRef(null);
    // React, ha kész vagy a rajzolással, rakd bele ezt az input elemet a változómba!
    <input ref={ inputRef } /> 
    // Később: inputRef.current.focus()
    ```
2.  **Technikai adatok tárolása:** Időzítő ID-k, előző értékek összehasonlításhoz, cache.

**Tisztaság (Purity) (20. dia):**
*   A `render` függvénynek "tisztának" kell lennie. Bemenet -> Kimenet. Mellékhatások nélkül.
*   Ezért **TILOS** a `useRef`-et írni/olvasni renderelés *közben* (a return előtti részben), mert az befolyásolhatja a végeredményt kiszámíthatatlan módon.
*   Használd `useEffect`-ben vagy eseménykezelőkben.

---

# 5. Custom Hooks - Logika kiszervezése (21-32. dia)

Ez a React "osztálykönyvtára". Ha van egy bonyolult logikád (pl. adatletöltés, időzítő, egérkövetés), azt kiszervezed egy saját függvénybe.

**Szabály:** A neve `use`-al kezdődjön (pl. `useClock`).

**Példa: Adatletöltés (30-31. dia):**
Itt látszik a React aszinkron nehézsége.
```javascript
function useJsonResource( uri: string ){
    let [ json, setJson ] = useState();

    useEffect( () => {
        let mounted = true; // Flag: él még a komponens?

        ( async function (){
            let x = await fetch( uri );
            let data = await x.json();
            // Csak akkor állítunk state-et, ha a komponens még létezik!
            // Ha közben átléptünk másik oldalra, és ez lefutna, hibát kapnánk.
            if ( mounted ) setJson( data );
        } )();

        // Takarító kód: ha megszűnik a komponens, átbillentjük a flaget.
        return () => mounted = false;
    }, [ uri ] ); // Ha változik az URI, újrafutunk.

    return json;
}
```
Ezt a logikát egyszer megírod, és 100 helyen használhatod: `const data = useJsonResource("/api/users");`.

---

# 6. `forwardRef` - Hozzáférés a gyerekhez (33-36. dia)

Alapesetben ha írsz egy saját komponenst `<MyInput />`, és a szülő ráteszi a `ref`-et (`<MyInput ref={...} />`), az **nem fog működni**. A React nem tudja, hova kösse a ref-et (a wrapper div-re? a belső inputra?).

**Megoldás:** A `forwardRef` egy csomagoló, amivel a komponens "továbbadja" a kapott ref-et egy belső HTML elemének.

```javascript
// (props, ref) -> a ref a második paraméterben jön be!
const ChildA = forwardRef( ( props, inputRef ) => {
    return <label>
        {/* Rákötjük a kapott ref-et a belső inputra */}
        <input ref={ inputRef } />
    </label>
});
```
Így a Szülő a `ChildA`-n keresztül közvetlenül eléri a benne lévő `<input>` DOM elemet.

---

# 7. Komponens paraméterként (37-42. dia)

Hogyan csinálj olyan komponenst (pl. `MessageBox`), aminek a belseje (pl. ikon, gombok) rugalmasan cserélhető?

**3. Megoldás: Függvény paraméter (Render Prop) - 42. dia:**
Ez a legprofibb megoldás. Nem egy kész ikont adsz át, hanem egy *függvényt*, ami *legyártja* az ikont.

```javascript
// Szülő
<MessageBox icon={ () => <Icon icon="warning" /> } />

// MessageBox (Gyerek)
function MessageBox( { icon } ) {
    return <div>
        {/* Itt hívjuk meg a függvényt. Ha nincs megadva, defaultot hívunk. */}
        { icon ? icon() : <Icon icon="ok" /> }
    </div>
}
```
**Miért ez a legjobb?** "Lazy evaluation". Ha a MessageBox úgy dönt, hogy nem rajzolja ki az ikont (pl. mobil nézetben), akkor a függvény nem hívódik meg, és az Ikon komponens létre sem jön. Spóroltunk erőforrást.

---

### Összefoglalás

1.  **Lifting State:** Ha testvéreknek kell adat, vidd fel a közös szülőbe.
2.  **Global Store:** Ha mindenkinek kell, vidd ki Reacten kívülre és szinkronizáld (`useSyncExternalStore`).
3.  **useRef:** Ha adatot akarsz tárolni renderelés nélkül, vagy DOM-hoz nyúlni.
4.  **Custom Hooks:** Így szervezd ki az ismétlődő logikát.
5.  **Render Props:** Így adj át UI elemeket paraméterként rugalmasan.

Mehetünk a negyedik (utolsó) diasorra? Az a **Render optimalizációról** szól.