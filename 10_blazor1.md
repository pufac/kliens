Tökéletes. Visszatérünk a hazai pályára. Ha C#/Java világból jössz, a Blazor olyan lesz, mint egy falat kenyér a React káosz után.

Itt **nincsenek** "mágikus hook-ok", amiknek a sorrendje számít.
Itt **nincsenek** fura "props" objektumok.
Itt **valódi osztályok**, **valódi property-k** és **erős típusosság** van.

Nézzük végig az **összes diát**, kódról kódra, ahogy kérted.

---

# 1. Mi ez az egész? (1-4. dia)

*   **Lényeg:** A Microsoft megcsinálta, hogy **C# kód fusson a böngészőben**. Nem kell JavaScriptet írnod.
*   **Hogyan?**
    *   Vagy letölti a .NET futtatókörnyezetet a böngészőbe (WebAssembly - WASM).
    *   Vagy a szerveren fut a kód, és egy folyamatos kapcsolaton (SignalR) küldi le a képernyő frissítéseket (Server).
*   **Razor:** Ez a fájlformátum (.razor). HTML és C# keveréke.

---

# 2. Projekt felépítése (5. dia)

Ez egy szabványos .NET Console/Web projekt struktúra.

*   `Program.cs`: **A belépési pont.** Itt indul a `main`. Itt állítod be a Dependency Injection-t (szolgáltatások). Ugyanaz, mint bármelyik modern .NET appnál.
*   `wwwroot`: A statikus fájlok (CSS, képek, ikonok). Ide a C# nem nyúl, ezt csak kiszolgálja a webszerver.
*   `Components`: Itt laknak a **Razor komponensek**.
    *   `App.razor`: A Gyökér komponens. Minden ezen belül jön létre.
    *   `Routes.razor`: Ez dönti el az URL alapján, melyik oldalt kell betölteni.
    *   `Pages`: Azok a komponensek, amik önálló oldalként (URL-lel) is működnek.
    *   `Layout`: A keretrendszer (fejléc, lábléc), ami minden oldalon közös.

---

# 3. Az Első Komponens: Counter (6-7. dia)

Ez a "Hello World". Nézzük sorról sorra a kódot!

**`Counter.razor` fájl tartalma:**

```csharp
// 1. ROUTING
// Ez mondja meg, hogy ez a komponens egy OLDAL, ami a "/counter" URL-en érhető el.
// Ha ezt törlöd, akkor csak egy beágyazható komponens lesz, nem nyitható meg böngészőből.
@page "/counter"

// 2. HTML CÍM BEÁLLÍTÁSA
// Ez nem jelenik meg a lapon, hanem a böngésző FÜLÉN (Tab) írja át a címet.
<PageTitle>Counter</PageTitle>

// 3. HTML TARTALOM
<h1>Counter</h1>

// 4. C# VÁLTOZÓ KIÍRATÁSA
// A @ jel jelzi, hogy C# kód jön.
// A 'currentCount' egy privát változó a lenti @code blokkból.
// Amikor a változó értéke nő, a Blazor automatikusan frissíti ezt a számot a képernyőn.
<p role="status">Current count: @currentCount</p>

// 5. ESEMÉNYKEZELÉS
// Ez egy szabványos HTML gomb.
// DE: az 'onclick' attribútum helyett '@onclick'-et írunk.
// Ez azt jelenti: "Ha klikkelnek, hívd meg a C# IncrementCount metódust".
// Nincs JavaScript!
<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>

// 6. LOGIKA (CODE-BEHIND)
// Ez itt tiszta C# osztály belső.
@code {
    // Egy sima privát mező (field). Ez tárolja az állapotot.
    private int currentCount = 0;

    // Egy sima privát metódus.
    private void IncrementCount()
    {
        // Megnöveljük a változót.
        // A Blazor észleli, hogy lefutott egy eseménykezelő,
        // megnézi, mi változott, és újrarajzolja a fenti <p> taget.
        currentCount++;
    }
}
```

---

# 4. Komponens Használata Másikban (8. dia)

A `Home.razor` fájlban vagyunk. Be akarjuk tenni a fenti számlálót.

```html
@page "/"
<PageTitle>Home</PageTitle>

<h1>Hello, world!</h1>

Welcome to your new app.

<!-- ÍGY PÉLDÁNYOSÍTJUK A KOMPONENST -->
<!-- A Blazor a fájlnév alapján (Counter.razor) tudja, hogy a <Counter /> tag -->
<!-- a Counter osztály egy példányát jelenti. -->
<Counter />
```

---

# 5. Paraméterek (Props) átadása (9. dia)

A "normális" programozásban, ha adatot akarsz adni egy objektumnak, beállítod a **publikus property**-jét. Itt is ez történik.

**A gyerek (`Counter.razor`):**
```csharp
@code {
    private int currentCount = 0;

    // [Parameter] attribútum: Ez teszi lehetővé, hogy a HTML-ből értéket kapjon.
    // Ez egy publikus property.
    // Alapértelmezett értéke 1.
    [Parameter]
    public int IncrementAmount { get; set; } = 1;

    private void IncrementCount()
    {
        // Használjuk a beállított property-t
        currentCount += IncrementAmount;
    }
}
```

**A szülő (`Home.razor`):**
```html
<!-- Itt állítjuk be a publikus property-t. -->
<!-- Az IntelliSense (kódkiegészítő) látja a C# osztályt, ezért felajánlja az 'IncrementAmount'-ot. -->
<!-- Ha elírod, vagy stringet adsz int helyett, FORDÍTÁSI HIBÁT kapsz (piros hullám). -->
<Counter IncrementAmount="3" />
```

---

# 6. Layoutok - A közös keret (10-11. dia)

A Layout is csak egy komponens, de speciális feladata van: ő a "Master Page".

**`MainLayout.razor` kódja (11. dia):**
```csharp
// Örökölnie kell a LayoutComponentBase-ből, hogy tudja kezelni a @Body-t.
@inherits LayoutComponentBase

<div class="page">
    <div class="sidebar">
        <!-- Behúzza a navigációs menü komponenst -->
        <NavMenu />
    </div>

    <main>
        <div class="top-row px-4">
            <!-- Fejléc linkek... -->
            <a href="...">About</a>
        </div>

        <article class="content px-4">
            <!-- A LÉNYEG: @Body -->
            <!-- Ide fogja renderelni azt az oldalt, amit épp megnyitottál (pl. a Counter-t). -->
            <!-- Ez a "placeholder" vagy "content place holder". -->
            @Body
        </article>
    </main>
</div>
```

---

# 7. Layout beállítása (12-15. dia)

Hogy mondjuk meg, hogy melyik Layoutot használja az alkalmazás?

**Globálisan (14. dia):**
A `_Imports.razor` fájlba írjuk. Ez egy speciális fájl, az itt lévő dolgok minden fájlra érvényesek a mappában.
```csharp
// Mindenki a MainLayout-ot használja alapértelmezetten.
@layout MainLayout
```
*Figyelem: Ne tedd a gyökérbe ugyanazzal a névvel, mert körkörös hivatkozás lehet.*

**Oldalanként (15. dia):**
Ha egy specifikus oldalon mást akarsz (pl. Login oldalon nincs menü).
```csharp
@page "/weather"
// Ez felülírja a globális beállítást.
// A Weather oldal a MyMainLayout keretet kapja.
@layout Layout.MyMainLayout

<h1>Weather</h1>
...
```

---

# 8. Routing (Útválasztás) (16-20. dia)

Hogyan találja meg a rendszer a `/counter` URL-hez a `Counter` osztályt?

**`Routes.razor` (20. dia):**
Ez a fájl fut le először, amikor az oldal betölt.
```html
<!-- Megkeresi az összes komponenst a Program assembly-ben (projektben), aminek van @page direktívája. -->
<Router AppAssembly="typeof(Program).Assembly">
    
    <!-- Ha talált illeszkedő oldalt (routeData): -->
    <Found Context="routeData">
        <!-- Megjeleníti az oldalt a megadott Layoutban. -->
        <RouteView RouteData="routeData" DefaultLayout="typeof(Layout.MainLayout)" />
        <!-- Ha navigálunk, fókuszálja a h1-et (akadálymentesítés). -->
        <FocusOnNavigate RouteData="routeData" Selector="h1" />
    </Found>

    <!-- Ha NEM talált ilyen URL-t (404): -->
    <NotFound>
        <PageTitle>Not found</PageTitle>
        <LayoutView Layout="typeof(Layout.MainLayout)">
            <p role="alert">Sorry, there's nothing at this address.</p>
        </LayoutView>
    </NotFound>
</Router>
```

---

# 9. URL Paraméterek (21-26. dia)

Hogyan szedjük ki az adatot az URL-ből? Pl.: `/szamlalo/3`

**Kód (21. dia):**
```csharp
// A kapcsos zárójel jelzi a paraméter helyét.
// :int => Ez egy KÉNYSZER (Constraint). Csak akkor illeszkedik, ha számmal hívják.
// Ha "/szamlalo/alma"-t írsz, az 404 lesz.
@page "/szamlalo/{IncrementAmount:int}"

@code {
    // A Property nevének (IncrementAmount) EGYEZNIE KELL az URL-ben lévő névvel!
    // A [Parameter] kötelező.
    // A típusnak (int) egyeznie kell a kényszerrel.
    [Parameter]
    public int IncrementAmount { get; set; }
}
```

**Query String (25-26. dia):**
Ha az URL így néz ki: `/counter?inc=5`

```csharp
// A Name="inc" mondja meg, mit keressen az URL ?-je után.
[SupplyParameterFromQuery(Name = "inc")]
public int Increments { get; set; }
```
*   Ez **bármelyik** komponensben működik, nem csak azokban, amiknek van `@page` direktívája (nem csak route-olható komponensekben).

---

# 10. Kódgenerálás (Code-Behind) (27-31. dia)

Mit csinál a fordító a `.razor` fájllal?

*   A `Counter.razor`-ból generál egy `Counter` nevű C# osztályt.
*   Ez az osztály a `ComponentBase`-ből származik.
*   A HTML részből C# utasítások lesznek, amik felépítik a "Render Tree"-t (ez a Blazor Virtual DOM-ja).

**Partial Class (Szétválasztás) - 30-31. dia:**
Ha nem szereted, hogy a HTML és a C# egy fájlban van, szétszedheted.
1.  **`CounterPartialClass.razor`:** Csak a HTML marad benne.
2.  **`CounterPartialClass.razor.cs`:** Létrehozol egy új fájlt.
    ```csharp
    // Fontos a 'partial' kulcsszó! Ez jelzi, hogy ez az osztály nem teljes,
    // a másik felét a Razor generátor csinálja a .razor fájlból.
    public partial class CounterPartialClass
    {
        private int currentCount = 0;
        // ... többi logika
    }
    ```
Ez a "normális" programozóknak nagyon kényelmes, tiszta kód.

---

# 11. Razor Szintaxis "Mágia" (32-33. dia)

*   `@`: Váltás HTML-ről C#-ra.
*   `@(...)`: Kifejezés kiértékelése. Pl. `@(currentCount * 10)`.
*   **Vezérlési szerkezetek:** Mivel ez C#, használhatsz `if`-et és `for`-t a HTML között!

```csharp
@if (value % 2 == 0) // Ha páros
{
    <p>Páros!</p> // HTML-t renderel
}

@for (var i = 0; i < people.Length; i++) // Ciklus
{
    // Minden körben létrehoz egy komponenst, és átadja az adatot
    <PersonComponent Name="@people[i].Name" Age="@people[i].Age" />
}
```
Ez sokkal olvashatóbb, mint a React `.map()` függvényei.

---

# 12. Hosting Modellek (37-51. dia) - A Legfontosabb Technikai Rész

Hol fut a C# kódod? Három lehetőség van.

**1. Blazor Server (41-42. dia):**
*   **Hol fut?** A szerveren (ASP.NET Core).
*   **Hogy jut el a böngészőbe?** A böngésző csak egy "buta terminál". Van egy nyitott **SignalR** (WebSocket) kapcsolat.
*   **Működés:** Klikkelsz -> A klikk esemény felmegy a szerverre -> A szerver futtatja a C# kódod -> A szerver kiszámolja, hogy a `<div>`-ből `<p>` lett -> Visszaküldi a különbséget -> A böngésző frissít.
*   **Előny:** Azonnal indul, nem kell letölteni semmit. Hozzáférsz az adatbázishoz közvetlenül.
*   **Hátrány:** Kell folyamatos net. Minden felhasználó a szerver memóriáját eszi.

**2. Blazor WebAssembly (WASM) (43-49. dia):**
*   **Hol fut?** A te gépeden (Kliens), a böngészőben.
*   **Hogy?** Letölt egy `dotnet.wasm` fájlt (ez egy mini .NET keretrendszer), és a te kódodat `.dll`-ben.
*   **Működés:** Minden a gépeden történik, offline is megy.
*   **Hátrány:** Első indításkor le kell tölteni 10-20 MB-ot (a keretrendszert). Lassú indulás.
*   **AOT (Ahead-of-Time) (49. dia):** A C# kódot nem futásidőben értelmezi (interpretálja), hanem fordításkor lefordítja gépi kódra (WASM). 5x gyorsabb, de 2x nagyobb fájlméret.

**3. Hybrid Blazor (38-39. dia):**
*   Ez asztali/mobil app (MAUI, WPF). Egy natív ablakban (WebView) futtatja a Blazort. Hozzáfér a fájlrendszerhez, kamerához.

**Összehasonlítás (51. dia táblázat):**
*   Server: Teljes .NET API, gyors indulás, de nincs offline mód.
*   WASM: Offline mód, szerver tehermentesítve, de lassú indulás és korlátozott API (nem érhetsz el direktben adatbázist).

---

# 13. Render Módok (.NET 8 újdonság) (58-69. dia)

A .NET 8-ban bevezették, hogy **komponensenként** döntheted el, hol fusson.

**A 4 mód:**
1.  **Static Server (Alapértelmezett):** A szerver legyártja a HTML-t, leküldi, kész. Mint egy PHP oldal. A gombok **NEM MŰKÖDNEK** (nincs JS/SignalR kapcsolat).
2.  **Interactive Server:** Bekapcsolja a SignalR-t. A gombok működnek, az események a szerveren futnak.
3.  **Interactive WebAssembly:** Letölti a kódot a böngészőbe és ott futtatja.
4.  **Interactive Auto:** A "Szent Grál".
    *   Amíg töltődik a WASM fájl, addig Server módban fut (azonnali indulás).
    *   Amint letöltődött a WASM, átvált kliens oldalra (gyorsaság, szerver kímélése).

**Beállítás (59-60. dia):**
```csharp
// A komponens tetejére írod:
@attribute [RenderModeInteractiveServer] 
// Vagy röviden a direktívával:
@rendermode InteractiveServer
```
Ha egy komponenst interaktívvá teszel, a **gyerekei is azok lesznek** (propagálás - 65. dia). Nem lehet statikus komponens egy interaktív belsejében.

---

# 14. Teljesítmény trükkök (72-77. dia)

**Prerendering (72-74. dia):**
*   Mielőtt a SignalR vagy WASM elindulna, a szerver gyorsan generál egy "buta" HTML-t, hogy láss valamit.
*   **Veszély:** Az `OnInitialized` metódusod **KÉTSZER** fog lefutni. Egyszer a prerenderkor (szerveren), egyszer a valódi induláskor. Ha itt adatbázist hívsz, duplán terheled.

**Streaming Rendering (75-77. dia):**
*   Ha az oldalad 90%-a kész, de egy adat (pl. időjárás) lassan jön az adatbázisból.
*   Nem várjuk meg! Leküldjük a keretet, és a tartalom helyére egy "Loading..."-ot.
*   Amikor megjön az adat, ugyanazon a HTTP kapcsolaton leküldi a maradék HTML-t, és a böngésző kicseréli.

**Példa kód (76. dia):**
```csharp
@attribute [StreamRendering] // Bekapcsoljuk a streamelést

@if (forecasts == null)
{
    <p>Loading...</p> // Ez jelenik meg azonnal
}
else
{
    // Ez jelenik meg, ha kész az await (aszinkron betöltés)
    <table>...</table>
}

@code {
    protected override async Task OnInitializedAsync() {
        // Ez a lassú művelet (500ms).
        // A StreamRendering miatt nem blokkolja az első megjelenítést.
        await Task.Delay(500); 
        forecasts = ...;
    }
}
```

---

# Összefoglalás (78-79. dia)

**Miért jó neked (C# fejlesztőnek)?**
*   Ismerős környezet: Osztályok, interfészek, típusok.
*   Kódmegosztás: A `WeatherForecast` osztályt megírod egyszer, és használja a szerver (adatbázis) és a kliens (megjelenítés). Nem kell TypeScript interfészt másolgatni.
*   Stabilitás: A fordító azonnal szól, ha elírtál valamit.

**Mikor ne használd?**
*   Ha a felhasználóidnak nagyon rossz az internete (Server módnál szakad, WASM-nál lassan tölt le).
*   Ha milliónyi egyidejű felhasználód van (a Server mód memóriát eszik minden user után).

Ennyi a Blazor. Egy tiszta, objektumorientált megközelítés a webfejlesztéshez. Érthetőbb így?