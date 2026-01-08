Íme az utolsó, "Blazor 4" előadásanyag részletes, kód-szintű elemzése. Ez a rész már a "hardcore" témákkal foglalkozik: **állapotkezelés** (mikor szakad meg a net), **JavaScript integráció** (ha valami nincs meg C#-ban), **HTTP hívások** (WebAPI) és **naplózás**.

---

# 1. Állapotkezelés (State Management) (3-12. dia)

A Blazor Server egyik legnagyobb előnye és egyben kihívása is, hogy az állapot a szerveren van.

### A probléma: Circuit (Áramkör)
*   Amikor megnyitsz egy Blazor Server oldalt, felépül egy SignalR kapcsolat (Circuit).
*   **Ameddig él a kapcsolat, él az állapot.** (Például egy számláló értéke).
*   **Ha megszakad a net (4. dia):** A szerver egy darabig vár (kb. 3 perc), addig megőrzi az állapotot. Ha a kliens visszajön, folytatja. Ha nem, az állapot elvész.
*   **Ha bezárod a böngészőt:** Azonnal vége.

### Megoldás: Állapot perzisztencia (5-7. dia)
Hogy ne vesszen el az adat frissítéskor (F5) vagy újraindításkor, el kell menteni valahova.
1.  **URL:** A legbiztosabb. Ha elküldöd a linket, másnak is ugyanaz jön be. (Query string paraméterek).
2.  **Szerver adatbázis:** Ha felhasználóhoz kötött.
3.  **Böngésző tárhely (LocalStorage/SessionStorage):** Ez a leggyakoribb ideiglenes mentésre.

### Tárolás a böngészőben (6. dia)
A Blazorhoz van egy beépített, védett tároló szolgáltatás: `ProtectedLocalStorage`.
Ez automatikusan titkosítja az adatot (hogy a felhasználó ne tudja átírni az F12-vel), és JSON-ba szerializálja.

```csharp
// Be kell injektálni a szolgáltatást
@inject ProtectedSessionStorage ProtectedSessionStore

@code {
    private async Task IncrementCount()
    {
        currentCount++;
        // Elmentjük a böngészőbe a "count" kulcs alá. Aszinkron!
        await ProtectedSessionStore.SetAsync("count", currentCount);
    }

    // Betöltés (7. dia)
    protected override async Task OnInitializedAsync()
    {
        // Megpróbáljuk kiolvasni. Lehet, hogy még nincs ott (null).
        var result = await ProtectedSessionStore.GetAsync<int>("count");
        if (result.Success)
        {
            currentCount = result.Value;
        }
    }
}
```

### Pre-rendering csapdák (9-10. dia)
*   **A probléma:** A Blazor alapból **kétszer** futtatja az `OnInitialized`-et. Először szerver oldalon (prerender), hogy a keresőmotorok lássák a HTML-t, aztán kliens oldalon (SignalR indulásakor).
*   **A gond:** Prerender közben **még nincs böngésző kapcsolat**, tehát **NINCS LocalStorage**. Ha ilyenkor hívod a fenti kódot, kivételt dob.
*   **Megoldás 1 (Kikapcsolás):** `prerender: false`.
*   **Megoldás 2 (Ellenőrzés - 10. dia):** Csak az `OnAfterRenderAsync`-ben nyúlunk a LocalStorage-hoz, mert az csak a kliens oldali renderelés után fut le (amikor már biztosan van kapcsolat).

---

# 2. JavaScript Interop (14-24. dia)

Bár a Blazor célja a JS kiváltása, néha muszáj használni (pl. térkép, geo-location, speciális JS könyvtárak). A Blazor lehetővé teszi a kétirányú kommunikációt.

### JS hívása C#-ból (16-17. dia)
1.  **Előkészület:** Kell egy JS fájl (pl. `export function askName(...)`), amit el kell érni.
2.  **Injektálás:** `@inject IJSRuntime JS`.
3.  **Modul betöltés:** A modern JS modulokat (`export`) először be kell importálni.
    ```csharp
    // Betöltjük a JS fájlt, és kapunk egy referenciát (module)
    var module = await JS.InvokeAsync<IJSObjectReference>("import", "./script.js");
    ```
4.  **Függvényhívás:**
    ```csharp
    // A modulon belül meghívjuk az 'askName' függvényt
    var name = await module.InvokeAsync<string>("askName", "Üzenet");
    ```

### HTML Elem átadása JS-nek (18-20. dia)
Sok JS könyvtárnak (pl. Google Maps) szüksége van egy konkrét `<div>`-re, amibe rajzolhat.
*   **HTML:** `<div @ref="mapElement"></div>`
*   **C#:** `private ElementReference mapElement;`
*   **JS hívás:** `JS.InvokeVoidAsync("showMap", mapElement);` -> A JS oldalon ez egy valódi DOM elementként érkezik meg.

### .NET hívása JavaScriptből (23-24. dia)
Ez a fordított irány. Ha a JS kódnak kell szólnia C#-nak (pl. a térképen kattintottak).

1.  **Statikus metódus hívása:**
    *   C#: `[JSInvokable]` attribútumot kell tenni a statikus metódusra.
    *   JS: `DotNet.invokeMethodAsync("AssemblyNeve", "MetodusNeve", parameterek)`.

2.  **Példány metódus hívása (24. dia):** Ez a gyakoribb.
    *   C#: Át kell adni a `this` referenciát a JS-nek (`DotNetObjectReference.Create(this)`).
    *   JS: A kapott objektumon hívható az `invokeMethodAsync("MetodusNeve")`.
    *   *Fontos:* A referenciát a végén `Dispose()`-olni kell a memóriaszivárgás elkerülése végett!

---

# 3. HTTP és REST hívások (25-33. dia)

A kliens (WASM) oldali Blazor ugyanúgy kommunikál a szerverrel, mint egy React app: HTTP kérésekkel (JSON).

### HttpClient használata (26-28. dia)
A `.NET`-ben a `HttpClient` az alaposztály erre.
```csharp
// Program.cs - beállítjuk az alap címet
builder.Services.AddScoped(sp => new HttpClient { BaseAddress = ... });

// Komponensben
@inject HttpClient Http

@code {
    // JSON letöltés és automatikus deszerializálás C# objektummá
    var todos = await Http.GetFromJsonAsync<TodoItem[]>("todoitems");
    
    // Küldés (POST)
    await Http.PostAsJsonAsync("todoitems", newItem);
}
```
*   Ez sokkal egyszerűbb, mint a JS `fetch` API-ja, mert típusos. Nem kell `JSON.parse` vagy `JSON.stringify`.

### Szerver oldali hívás (27. dia)
Blazor Servernél kicsit más a helyzet. Mivel a kód már a szerveren fut, nem feltétlenül kell HTTP kérés (hívhatnánk direktben az adatbázist). De ha mikro-szolgáltatások vannak, akkor ott is kell `HttpClient`.
*   Ott a `IHttpClientFactory`-t érdemes használni a socket kimerülés elkerülésére.

### JSON Testreszabás (31. dia)
A `.NET` a `System.Text.Json`-t használja.
*   **PATCH hívás:** Ez trükkös, mert csak a *változást* kell küldeni, nem a teljes objektumot. Ehhez külön library (`Newtonsoft` vagy `Microsoft.AspNetCore.JsonPatch`) kellhet.

### Hitelesítés (Tokenek) (33. dia)
Ha védett API-t hívsz, a fejlécbe be kell tenni a JWT tokent (`Authorization: Bearer ...`).
*   Erre érdemes egy `DelegatingHandler`-t írni, ami minden kéréshez automatikusan hozzáadja a tokent, így nem kell minden egyes hívásnál másolni a kódot.

---

# 4. Konfiguráció (44-51. dia)

Hogyan állítjuk be az API URL-jét vagy egyéb paramétereket anélkül, hogy újrafordítanánk a kódot?

### `appsettings.json`
Ez a szabványos .NET konfig fájl.
```json
{
  "GreetingText": "Hello",
  "OrderQueue": {
    "MaxSize": 32
  }
}
```

### Használata (50-51. dia)
1.  **Injektálás:** `@inject IConfiguration Config`
2.  **Olvasás:** `@Config["GreetingText"]` vagy típusosan: `Config.GetValue<int>("OrderQueue:MaxSize")`.

**A profi módszer (IOptions):**
Létrehozol egy C# osztályt, ami tükrözi a JSON szerkezetét, és a rendszer automatikusan feltölti.
```csharp
// Program.cs
builder.Services.Configure<OrderQueueConfig>(
    builder.Configuration.GetSection("OrderQueue"));

// Komponensben
@inject IOptions<OrderQueueConfig> QueueConfig
// Használat: @QueueConfig.Value.MaxSize
```
Ez azért jó, mert ha elírod a nevet a JSON-ban, az alkalmazás induláskor szól (vagy default értéket használ), és a kódban IntelliSense támogatást kapsz.

---

# 5. Naplózás (Logging) (46-50. dia)

Soha ne használj `Console.WriteLine`-t éles kódban! Használd az `ILogger`-t.

### Használata (47. dia)
```csharp
@inject ILogger<Counter> Logger

@code {
    void Increment() {
        // Strukturált naplózás
        Logger.LogWarning("Valaki kattintott! Új érték: {Count}", currentCount);
    }
}
```
*   **Szerver oldalon:** Ez a szerver konzoljára, fájlba vagy adatbázisba ír.
*   **WASM oldalon:** Ez a böngésző konzoljára (F12) ír.

### Szemantikus (Strukturált) naplózás (50. dia)
A fenti példában a `{Count}` nem csak behelyettesítődik a szövegbe. A naplózó rendszer (pl. Elasticsearch, Application Insights) külön mezőként is elmenti a `Count` értékét, így később tudsz rá keresni (pl. "mutasd azokat a logokat, ahol Count > 100").

---

### Összefoglalás "normális" programozóknak:

1.  **Állapot:** Ha a net elmegy, a memória törlődik. Használd a `ProtectedLocalStorage`-ot a fontos adatok mentésére.
2.  **JS Interop:** Ha nincs C# megoldás valamire, áthívhatsz JS-be. De ez aszinkron és lassabb, csak akkor használd, ha muszáj.
3.  **HTTP:** A `HttpClient` a barátod. Típusos, `GetFromJsonAsync`-el egy sorban megvan az adatod.
4.  **Config:** Ne égess be URL-eket a kódba! Tedd `appsettings.json`-ba és használd az `IOptions` mintát.
5.  **Logging:** `ILogger` mindenhova. Segít megtalálni a hibát, amikor az ügyfél gépén fut a kód.

Ezzel a végére értünk a teljes Blazor anyagnak. Most már van egy átfogó képed a rendszerről: komponensek, navigáció, állapotkezelés és a külvilággal való kommunikáció. Van még kérdésed?