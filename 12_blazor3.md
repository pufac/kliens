Íme a "Blazor 3" előadásanyag részletes, kódról-kódra történő feldolgozása. Ez a diasor a **navigációval, rendereléssel és az életciklusokkal** foglalkozik, amelyek kritikus fontosságúak egy modern SPA (Single Page Application) építésénél.

---

# 1. Navigáció (3-17. dia)

A webes alkalmazásoknál alapvető a navigáció (oldalak közötti váltás). A Blazorban ez történhet deklaratívan (HTML-ből) vagy programozottan (C# kódból).

### Deklaratív navigáció: `NavLink` (3. dia)
Sima `<a>` teg helyett érdemes a `NavLink` komponenst használni.
*   **Előnye:** Automatikusan ráteszi az `active` CSS osztályt, ha az URL megegyezik a hivatkozással (így könnyű kiszínezni az aktív menüpontot).
*   **`Match` paraméter:**
    *   `NavLinkMatch.All`: Csak akkor aktív, ha a *teljes* URL egyezik (pl. Főoldal).
    *   `NavLinkMatch.Prefix` (alapértelmezett): Akkor is aktív, ha az URL csak *kezdődik* vele (pl. `/products` aktív marad `/products/123` esetén is).

### Programozott navigáció: `NavigationManager` (4-6. dia)
Ha C# kódból akarsz átirányítani (pl. sikeres mentés után), akkor ezt a szolgáltatást (Service) kell használnod.

1.  **Injektálás:** `@inject NavigationManager Navigation` (vagy konstruktorban).
2.  **Navigálás:** `Navigation.NavigateTo("/counter");`
3.  **URL kezelés:**
    *   `Navigation.Uri`: A teljes jelenlegi URL.
    *   `Navigation.BaseUri`: A gyökér URL (pl. `localhost:5000/`).
    *   `ToBaseRelativePath(url)`: Levágja a domaint, csak az útvonalat adja vissza.

**Query paraméterek kezelése (6. dia):**
Hogyan fűzzünk paramétereket az URL-hez (`?count=5`)?
```csharp
// Ez egy kényelmi függvény, ami összeállítja az új URL-t
// Ha null-t adsz értéknek, kiveszi a paramétert.
Navigation.GetUriWithQueryParameter("IncrementAmount", 5);
```

### Navigáció események (10-13. dia)
Néha tudnunk kell róla, ha a felhasználó elnavigál (pl. nem mentett adatok figyelmeztetése).

*   **`LocationChanged` event:** Ez az esemény sül el, amikor megváltozik az URL.
*   **`RegisterLocationChangingHandler` (13. dia):** Ez a modern (.NET 7+) megoldás.
    *   Lehetővé teszi a navigáció **megszakítását** (`PreventNavigation`).
    *   Használat:
        ```csharp
        // Regisztráljuk a kezelőt
        var registration = Navigation.RegisterLocationChangingHandler(OnLocationChanging);
        
        private ValueTask OnLocationChanging(LocationChangingContext context) {
            if (nemMentettAdat) {
                context.PreventNavigation(); // Megállítjuk a navigációt!
            }
            return ValueTask.CompletedTask;
        }
        ```
    *   **Fontos:** A komponenst `IDisposable`-ként kell implementálni, és a `Dispose`-ban le kell iratkozni (`registration.Dispose()`), különben memóriaszivárgás lesz!

---

# 2. Eseménykezelés (19-25. dia)

### Szinkron vs. Aszinkron események
*   **Szinkron (19-20. dia):** Ha a metódus `void` visszatérésű. Ilyenkor a blokkoló művelet (pl. `Thread.Sleep`) megakasztja az egész UI-t. **Kerülendő!**
*   **Aszinkron (21. dia):** Ha a metódus `async Task` visszatérésű. Ilyenkor `await`-elhetünk (pl. `Task.Delay`), a UI reszponzív marad.

### Esemény paraméterek (22. dia)
A Blazor minden szabványos DOM eseményhez (kattintás, egérmozgás, billentyűzet) biztosít C# osztályt (pl. `MouseEventArgs`, `KeyboardEventArgs`), amiben megkapjuk a részleteket (koordináták, leütött gomb).

### Saját esemény publikálása (23-24. dia)
Hogyan szólhat a gyerek a szülőnek? (Reactben ez a callback prop).
1.  A gyerekben definiálunk egy `EventCallback` típusú paramétert:
    ```csharp
    [Parameter]
    public EventCallback<string> OnClickCallback { get; set; }
    ```
2.  Amikor történik valami, meghívjuk:
    ```csharp
    await OnClickCallback.InvokeAsync("Üzenet");
    ```
3.  A szülő feliratkozik rá:
    ```html
    <Child OnClickCallback="ShowMessage" />
    ```

### Események módosítása (25. dia)
*   **`preventDefault`:** Megakadályozza az alapértelmezett működést (pl. form elküldése, vagy input mezőbe írás).
    `<input @onkeydown:preventDefault />`
*   **`stopPropagation`:** Megakadályozza, hogy az esemény "felbuborékoljon" a szülő elemekhez (pl. egy gomb kattintása ne váltsa ki a befoglaló div kattintását is).

---

# 3. Életciklus (Lifecycle) (27-36. dia)

Ez a rész kulcsfontosságú a helyes működéshez. A komponenseknek meghatározott életszakaszaik vannak.

### 1. Létrehozás (Initialization)
*   **`OnInitialized` / `OnInitializedAsync`:** Akkor fut le, amikor a komponens létrejön. Itt szoktunk adatot letölteni az adatbázisból.
*   **Figyelem:** Prerendering (SSR) esetén ez **kétszer** fut le!

### 2. Paraméter változás
*   **`OnParametersSet` / `OnParametersSetAsync`:** Akkor fut le, ha a szülő új paramétereket ad át (vagy inicializáláskor). Ha az URL paraméter változik (pl. `/user/1` -> `/user/2`), akkor is ez fut le, az `OnInitialized` nem!

### 3. Renderelés után
*   **`OnAfterRender(bool firstRender)`:** Akkor fut le, amikor a böngésző már kirajzolta a HTML-t.
*   **Mire jó?** JavaScript interop hívásokhoz (pl. térkép inicializálása, fókusz állítás), mert ilyenkor már léteznek a DOM elemek.
*   A `firstRender` paraméter jelzi, ha ez az első kirajzolás.

### 4. Megszűnés (Disposal)
*   Ha a komponens eltűnik a képernyőről (pl. elnavigálunk), meg kell hívni a `Dispose`-t.
*   Ehhez implementálni kell az `IDisposable` (vagy `IAsyncDisposable`) interfészt.
*   **Feladat:** Eseményekről leiratkozás, Timerek leállítása.

### Renderelés szabályozása: `ShouldRender` (35. dia)
*   Alapból a Blazor minden esemény után újrarenderel.
*   Ha ezt felülírod és `false`-t adsz vissza, letilthatod a frissítést (teljesítmény optimalizáció).

---

# 4. Adatforgalom Szülő és Gyerek között (37-41. dia)

Ez a "Data Down, Actions Up" elv, mint Reactben, de a Blazor kétirányú kötései miatt kicsit trükkösebb.

### A probléma: Paraméter felülírása (37-38. dia)
*   A gyerek kap egy paramétert (`[Parameter] public bool IsHidden`).
*   A gyerek ezen belül átírja (`IsHidden = true`).
*   **Hiba:** Amikor a szülő legközelebb újrarenderelődik, ő "visszaírja" az eredeti értéket a gyerekbe, felülírva a gyerek belső módosítását.
*   **Szabály:** A gyerek sose írja át közvetlenül a kapott paramétert!

### A megoldás: Kétirányú kötés (41. dia)
Használjuk a `@bind-Param` szintaxist.
1.  A gyerek szól a szülőnek (`EventCallback`), hogy "hé, változni szeretnék".
2.  A szülő megkapja az eseményt, megváltoztatja a saját állapotát.
3.  A szülő leküldi az új értéket a gyereknek.

---

# 5. QuickGrid (42-43. dia)

A Microsoft beépített, nagy teljesítményű táblázat komponense.
*   Nem kell mindent kézzel megírni (`table`, `tr`, `td`, `foreach`).
*   Támogatja a lapozást (pagination), rendezést, szűrést.
*   Közvetlenül ráköthető `IQueryable` adatforrásra (pl. Entity Framework), így hatékonyan kérdezi le az adatbázist (nem tölt le mindent a memóriába).

---

# 6. Konfiguráció (44-50. dia)

Hogyan olvassuk be a beállításokat (pl. API URL) kliens oldalon?

*   A szerver oldali ASP.NET Core konfigurációs rendszert ( `appsettings.json`) használhatjuk a kliensen is (WASM-ban is!).
*   **`IConfiguration` service:** Ezt injektáljuk be.
*   **`appsettings.json`:** A `wwwroot` mappában van, letöltődik a böngészőbe.
*   **Típusos konfiguráció (Options minta):** A JSON beállításokat egy C# osztályra (`GreetingsConfig`) mappelhetjük, így nem kell stringekkel ("GreetingText") bajlódni, hanem property-ket kapunk.

---

# Összefoglalás

Ez a diasor a Blazor "működési mechanizmusát" fedte le.
1.  **Navigáció:** A `Router` komponens és a `NavigationManager` service végzi.
2.  **Életciklus:** `OnInitialized` (létrehozás), `OnParametersSet` (frissítés), `OnAfterRender` (JS interop), `Dispose` (takarítás). Ezeket kell felülírni (`override`), ha be akarunk avatkozni.
3.  **Adatfolyam:** A paramétereket (`[Parameter]`) a szülő adja, a gyerek nem írhatja át őket közvetlenül, hanem eseményt (`EventCallback`) kell küldenie.

Ezzel átvettük a kliensoldali fejlesztés legfontosabb Blazor-specifikus részeit. Van kérdésed?