Íme a „Kliensoldali rendszerek” tárgy **React render és optimalizáció** témakörű előadásának részletes, diáról-diára történő feldolgozása.

---

# Kliensoldali Rendszerek – React Render és Teljesítmény

Ez az előadás a React alkalmazások sebességével, a renderelési folyamattal és annak optimalizálásával foglalkozik.

## 1. Render Sebesség és Teljesítmény Alapok (2-6. dia)

### Alapvetések
*   A React renderelése alapvetően gyors, feltéve, hogy a komponensfa (React tree) mérete korlátos.
*   Jelenleg a böngészők rendelkeznek a leggyorsabb UI motorral (évekig tartó optimalizáció eredménye), gyakran gyorsabbak, mint a natív desktop technológiák (pl. WPF).
*   Azonban a "React render" és a "HTML render" nem ugyanaz, és a kettő együtt néha mégsem elég gyors.

### Kétféle sebesség megkülönböztetése
Fontos szétválasztani két fogalmat, mert a hiba forrása eltérő:

1.  **React Render sebesség:**
    *   Ez a **JavaScript** oldali számítás: a Virtual DOM felépítése, összehasonlítása (diffing) és a változások kiszámolása.
    *   **Ha ez lassú:** Az alkalmazás megakad, "laggol", a fő szál blokkolva van.
    *   **Tűréshatár:** Alacsony. Egy render ciklusban az 1000 elem feletti fa már lassulást okozhat. 10.000 elemnél már másodperces nagyságrendű lehet a blokkolás.

2.  **HTML (Böngésző) sebesség:**
    *   Ez a böngésző natív feladata: a DOM elemek kirajzolása, layout számítás, festés (painting).
    *   **Ha ez lassú:** Alacsony FPS (képkockaszám), darabos görgetés vagy animáció.
    *   **Befolyásoló tényezők:** DOM fa mérete, CSS effektek (árnyék, blur), nagy képek, sok SVG.
    *   **Tűréshatár:** Magas. 10.000 egyszerű DOM elemmel még nincs gond, sőt, minimalista CSS-sel akár 1 millió elem is megjeleníthető (de ez ritka a gyakorlatban).

*Megjegyzés: A hálózati lassulással (network latency) ebben az anyagban nem foglalkozunk.*

---

## 2. Nagy adathalmazok kezelése (7. dia)

Ha a React fa túl nagyra nőne (pl. nagy listák, táblázatok), optimalizálni kell.

### Virtualizáció (Windowing)
*   **Probléma:** 1 millió soros táblázatot nem lehet egyszerre renderelni (React render is lassú lenne, és a memória is elfogyna).
*   **Megoldás:** Csak azt a részt rendereljük le, ami éppen **látszik a képernyőn** (plusz egy kis ráhagyást/buffert a görgetéshez).
*   Így a felhasználó azt hiszi, ott az egész lista, de a DOM-ban csak pl. 50 sor van.

### "Nem látszó" részek kezelése
*   Ami nem látszik (pl. egy inaktív fül egy TabControlban, vagy egy másik oldal), azt **ne** rejtsük el CSS-sel (`display: none`).
*   **Helyes megoldás:** Vegyük ki teljesen a React fából (ne rendereljük le). A `display: none` elemeket a React ugyanúgy kiszámolja és karbantartja, ami felesleges erőforrás-pazarlás.

---

## 3. Render Optimalizáció: Mikor és Hogyan? (8-16. dia)

### Mikor fut le a render?
Egy komponens render függvénye akkor fut le, ha:
1.  A **szülője** újrarenderelődött (ez a leggyakoribb ok).
2.  A komponens **saját állapota** (state) megváltozott (vagy `forceUpdate` hívás történt).

**Fontos szabály:** Reactben alapértelmezetten, ha a szülő renderel, az **összes** gyereke (a teljes részfa) is újrarenderelődik, függetlenül attól, hogy változott-e az adatuk.

### `memo` és `PureComponent`
Ezek az eszközök arra valók, hogy megállítsák a felesleges renderelési láncolatot.
*   **PureComponent (Osztályoknál):** Csak akkor renderel, ha a props vagy state változott (sekély összehasonlítással).
*   **`memo` (Függvényeknél):** Egy magasabb rendű komponens (HOC), ami burkolja a saját komponensünket.
    *   **Működése:** Ha a szülő renderel, a `memo` megnézi, hogy a kapott `props` változott-e az előzőhöz képest. Ha minden prop azonos (referencia szerint), akkor **megszakítja a renderelést** az adott ágon.
    *   **Testreszabás:** A második paraméterében megadható egy `arePropsEqual` függvény egyedi összehasonlításhoz.

### A `memo` kihívása: A stabilitás problémája
A `memo` csak akkor működik, ha a szülő "stabil" adatokat ad át.
*   **Probléma:** Ha a szülőben így adjuk át a függvényt: `<Child onClick={() => {}} />`, akkor minden rendereléskor **új függvény objektum** jön létre.
*   Emiatt a `memo` úgy látja, hogy a `props` megváltozott -> **újrarenderel**, tehát a `memo` használata felesleges volt.

### Megoldások a stabil referenciákra (Stable Identity)

1.  **`useCallback` hook:**
    *   Arra való, hogy egy függvényt "elmentsünk".
    *   Csak akkor hozza létre újra a függvényt, ha a függőségi listájában (`dependency array`) lévő változók megváltoznak.
    *   `const stableClick = useCallback(() => { ... }, []);` -> Ez mindig ugyanazt a referenciát adja vissza.
    *   **Closure buktató:** Ha a függvényben külső változót használunk (pl. `text`), azt fel kell venni a függőségi listába (`[text]`), különben a függvény a régi, "beragadt" értéket fogja látni (stale closure).

2.  **`useMemo` hook:**
    *   Hasonló a `useCallback`-hez, de ez **értékeket** (objektumokat, tömböket, számítási eredményeket) cache-el.
    *   Csak akkor számolja újra/hozza létre az objektumot, ha a függőségek változnak.
    *   Hasznos "drága" számítások gyorsítótárazására is.

3.  **`useRef` mint végső megoldás:**
    *   A `useRef` egy olyan tároló, ami megőrzi a tartalmát renderelések között, és a referenciája sosem változik.
    *   Ez a legrugalmasabb (és legtöbb kódot igénylő) módja az optimalizációnak, ha mi akarjuk kézzel vezérelni, mikor mi frissüljön (pl. manuális összehasonlítás).

### Optimalizációs stratégia
*   **Ne optimalizálj mindent előre!** A `memo`, `useMemo` használatának is van költsége (memória, összehasonlítási idő).
*   Egy komplex alkalmazásban a komponensek kb. **1-10%-át** érdemes csak `memo`-ba csomagolni (azokat, amik nagy részfát tartalmaznak vagy gyakran renderelődnek feleslegesen).
*   A szülő-gyerek együttműködés elengedhetetlen: hiába van `memo` a gyereken, ha a szülő instabil props-ot küld.

---

## 4. Speciális React Technikák (17-21. dia)

### A `key` attribútum jelentősége
*   **Listáknál:** Segít a Reactnak azonosítani az elemeket (melyik új, melyik törölt, melyik mozdult el).
    *   **Index használata:** Kerülendő, ha a lista sorrendje változhat, mert ez bugokhoz vezethet (az állapotok összekeveredhetnek). Mindig használjunk egyedi ID-t (pl. adatbázis ID).
*   **Komponenseknél (nem listában):**
    *   Azonos kulcs -> Azonos komponens (megmarad az állapot).
    *   Más kulcs -> **A régi törlődik, új jön létre.**
    *   *Trükk:* Ha egy bonyolult állapotú komponenst teljesen alaphelyzetbe (reset) akarunk állítani, csak változtassuk meg a `key`-jét.

### `createPortal`
*   Lehetővé teszi, hogy egy komponenst a DOM fa egy teljesen más pontjára (pl. közvetlenül a `document.body`-ba) rendereljünk, miközben logikailag a React fában marad (context, események működnek).
*   **Use-case:** Dialógus ablakok (Modals), Tooltipek, amiknek "ki kell lógniuk" a szülőjükből (z-index problémák elkerülése végett).

### Dialógus ablakok kezelése
*   A HTML `<dialog>` elem létezik, de még vannak korlátai.
*   A Portal jó megoldás, de figyelni kell az élettartamra: ha a szülő komponens megszűnik, a Portalon keresztül renderelt dialógus is eltűnik.
*   **Profi megoldás:** Egy globális API (pl. `showDialog`), ami a komponensektől függetlenül, globálisan kezeli a megnyitott ablakokat.

---

## 5. Animációk és Játékok (22-23. dia)

### `requestAnimationFrame` (RAF)
*   Ha 60 FPS-es, folyamatos animációra van szükség (pl. egérkövetés, játék), a React renderelési ciklusa túl lassú lehet.
*   Ilyenkor a böngésző `requestAnimationFrame` API-ját használjuk.

### RAF és React viszonya
*   **Kerüljük a React rendert RAF-ban:** Ne hívjunk `setState`-et minden frame-ben! A React nem tudja garantálni a 60 FPS-t a Virtual DOM számítások miatt.
*   **Közvetlen DOM manipuláció:** RAF callback-ben módosítsuk közvetlenül a DOM elemet (ref segítségével). Ez a leggyorsabb.
*   **Sorrend animációra:**
    1.  **CSS:** Ha lehet, oldjuk meg CSS transition/animation segítségével (leggyorsabb, GPU gyorsított).
    2.  **RAF + Direct DOM:** Ha logikát igényel (pl. egérpozíció).
    3.  **React Render:** Csak ha nagyon muszáj, és nem kell a 60 FPS (használjunk debounce-ot).