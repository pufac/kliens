Íme a „Blazor 2” előadásanyag részletes, kódról-kódra történő feldolgozása. Ez a diasor már nem az alapokról szól, hanem a **haladóbb funkciókról**: adatkötés (data binding), űrlapok (forms), validáció és fájlkezelés.

---

# 1. Adatkötés (Data Binding) (3-8. dia)

A Blazor egyik legnagyobb erőssége, hogy a HTML elemek és a C# változók között nagyon könnyű kapcsolatot teremteni.

### Az alapelv (3. dia)
Kétirányú kapcsolatot akarunk:
1.  Ha a C# kódban változik az érték -> frissüljön az input mező a képernyőn.
2.  Ha a felhasználó ír az input mezőbe -> frissüljön a C# változó.

### Egyszerű adatkötés: `@bind` (4. dia)
```csharp
<!-- Kétirányú kötés a 'IncrementAmount' property-hez -->
<input @bind="IncrementAmount" />

@code {
    public int IncrementAmount { get; set; }
}
```
*   **Hogyan működik?** Alapértelmezésben a Blazor a `change` eseményt figyeli. Tehát amikor **kikattintasz** az input mezőből (fókuszvesztés), akkor frissül a változó.

### Felület frissítése (5. dia)
Fontos: A felület (UI) **nem frissül minden karakterleütésre** automatikusan, csak akkor, ha a komponens újrarendereli magát.
*   Példa: Van egy gomb, ami törli a mezőt (`Clear`). Amikor rákattintasz, lefut a `OnClear` metódus, megváltozik a `text` változó, és a Blazor újrarajzolja a komponenst -> az input mező kiürül.

### Aszinkron események (6-7. dia)
Mi van, ha a változás nem felhasználói interakcióból jön, hanem pl. egy időzítőből (Timer)?
```csharp
// Ez egy háttérszálon futó Timer callback
timer = new Timer((_) => {
    text = "your name";
    // KÖTELEZŐ: Értesíteni kell a Blazort, hogy változás volt!
    // Mivel ez nem UI szál, InvokeAsync kell.
    this.InvokeAsync(StateHasChanged); 
}, ...);
```
*   **`StateHasChanged()`**: Ez a metódus szól a Blazornak: "Hahó, változtattam valamit a kódban, kérlek rajzold újra a komponenst!". Normál gombnyomásnál (`@onclick`) ezt a Blazor hívja helyetted, de Timer-nél neked kell.

### Kötés más eseményre: `oninput` (8. dia)
Ha azt akarod, hogy **minden egyes karakter leütésekor** frissüljön az adat (pl. keresőmezőnél), nem elég az alapértelmezett `change` esemény.
```html
<!-- event="oninput": Minden gépelésre frissít -->
<input @bind="IncrementAmount" @bind:event="oninput" />
```

---

# 2. Speciális Kötések (9-16. dia)

### Stílus változtatása dinamikusan (9. dia)
```html
<!-- Ha az email helyes (zöld), ha nem (piros) -->
<input style="color: @color" ... />

@code {
    // Regex ellenőrzés C#-ban
    set { 
       email = value;
       color = emailRegex.IsMatch(value) ? "green" : "red";
    }
}
```
*   Itt látszik a C# ereje: használhatsz `Regex`-et, validációs logikát közvetlenül a setterben.

### Értesülés a változásról (10-11. dia)
Gyakran nem elég csak bekötni a változót, hanem futtatni is akarunk valamit, amikor megváltozik (pl. keresés indítása).
Három módszer van:
1.  **`@bind:after` (Modern):** A változó beállítása *után* lefut egy metódus.
    ```html
    <input @bind="searchText" @bind:after="PerformSearch" />
    ```
2.  **`@bind:get` / `@bind:set` (Kézi vezérlés):** Szétválasztod az olvasást és írást.
3.  **Property setter (Hagyományos):** A C# property `set` ágába írod a logikát.

### `select` és `multiple` (15-16. dia)
A legördülő listáknál a `value` attribútumot kötjük. Ha többet is ki lehet választani (`multiple`), akkor egy **tömbhöz** kell kötni.
```csharp
<select @bind="increments" multiple>
    <option value="1">egy</option> ...
</select>

@code {
    private int[] increments = []; // Tömb a kiválasztott értékeknek
}
```

---

# 3. Adatkonverzió és Formázás (17-18. dia)

A HTML-ben minden `string` (szöveg). A C#-ban viszont vannak `int`, `DateTime`, stb. típusok.
*   A `@bind` **automatikusan konvertál**. Ha `int` változóhoz kötsz egy inputot, megpróbálja számmá alakítani.
*   **Hiba esetén:** Ha a felhasználó "kismacska"-t ír egy szám mezőbe, a konverzió sikertelen. A Blazor ilyenkor visszaállítja az eredeti értéket (visszajavítja).
*   **Dátum formázás (18. dia):**
    ```html
    <!-- Megadhatod, milyen formátumban jelenjen meg/fogadja el a dátumot -->
    <input @bind="startDate" @bind:format="yyyy-MM-dd" />
    ```

---

# 4. Kötés Saját Komponensre (19. dia)

Hogyan csinálj olyan komponenst (`TextInput`), amire a szülő tud `@bind`-olni?
Konvenció alapú a dolog:
1.  Kell egy `Value` paraméter (bemenet).
2.  Kell egy `ValueChanged` paraméter (kimenet, `EventCallback`).

```csharp
@code {
    [Parameter]
    public string? Value { get; set; }

    [Parameter]
    public EventCallback<string> ValueChanged { get; set; }
}
```
Ha ez a kettő megvan, a szülő használhatja így: `<TextInput @bind-Value="szuloValtozo" />`.

---

# 5. Kaszkádolt Értékek (Cascading Values) (24-26. dia)

Ez a React `Context API` megfelelője. Adatot küldünk le a fában mélyre, anélkül, hogy minden komponensen átfűznénk (prop drilling).

1.  **Küldés (Szülőben):**
    ```html
    <CascadingValue Value="@themeInfo">
        @Body <!-- Minden gyerek megkapja -->
    </CascadingValue>
    ```
2.  **Fogadás (Bárhol a mélyben):**
    ```csharp
    [CascadingParameter]
    public ThemeInfo? ThemeInfo { get; set; }
    ```
Ez nagyon hasznos pl. a téma (dark mode) vagy a bejelentkezett felhasználó adatainak terjesztésére.

---

# 6. Űrlapok (Forms) és Validáció (27-35. dia)

A Blazor beépített támogatást ad az űrlapokhoz (`EditForm`).

### Az `EditForm` komponens (29. dia)
Nem sima `<form>`-ot használunk, hanem ezt.
*   **Model:** Ehhez az objektumhoz kötjük az űrlapot.
*   **OnValidSubmit:** Ez a metódus fut le, ha a felhasználó elküldi az űrlapot, ÉS minden adat helyes.

### Validáció (Ellenőrzés) (31-32. dia)
Nem kell if-ekkel ellenőrizgetni. C# attribútumokat teszünk a modell osztályra (Data Annotations).
```csharp
public class RegistrationModel {
    [Required] // Kötelező
    [StringLength(30, ErrorMessage = "Túl hosszú!")] // Max hossz
    public string Name { get; set; }
}
```
Az űrlapban pedig hozzáadjuk a validátorokat:
*   `<DataAnnotationsValidator />`: Aktiválja az ellenőrzést.
*   `<ValidationSummary />`: Kilistázza az összes hibát egy helyen.
*   `<ValidationMessage For="..." />`: Egy konkrét mező hibáját írja ki.

---

# 7. Adatbeviteli Komponensek (36-37. dia)

A Blazor ad kész komponenseket, amik együttműködnek az `EditForm`-mal és a validációval. Használd ezeket a sima HTML inputok helyett űrlapokon belül!

*   `InputText` -> `<input type="text">`
*   `InputNumber` -> `<input type="number">` (ez automatikusan kezeli a szám konverziót)
*   `InputCheckbox`, `InputDate`, `InputSelect`, `InputTextArea`...

---

# 8. Fájl feltöltés (38-39. dia)

A fájlkezelés trükkös weben. A Blazor erre az `<InputFile>` komponenst adja.

**Kód (39. dia):**
```csharp
private async Task LoadFiles(InputFileChangeEventArgs e)
{
    // Végigmegyünk a feltöltött fájlokon
    foreach (var file in e.GetMultipleFiles(maxAllowedFiles))
    {
        // KÖTELEZŐ BIZTONSÁGI LÉPÉS:
        // Nem használjuk az eredeti fájlnevet mentésnél (hacker feltölthetne .exe-t)
        var trustedFileName = Path.GetRandomFileName();
        
        // Fájl mentése (stream-elés)
        await using FileStream fs = new(path, FileMode.Create);
        await file.OpenReadStream(maxFileSize).CopyToAsync(fs);
    }
}
```
**Fontos biztonsági szabályok:**
1.  Korlátozd a fájlméretet (`maxFileSize`).
2.  Korlátozd a darabszámot.
3.  Sose bízz meg a kliens által küldött fájlnévben!

---

### Összefoglalás "normális" programozónak:

1.  **Adatkötés:** `@bind="Valtozo"`. Kétirányú, automatikus.
2.  **Esemény:** `@onclick="Metodus"`. C# metódust hív.
3.  **Paraméter:** `[Parameter] public int Ertek { get; set; }`. Így kap adatot a szülőtől.
4.  **Űrlap:** Használj `EditForm`-ot és `DataAnnotations` attribútumokat (`[Required]`) a modellen. A rendszer magától validál.
5.  **Fájl:** `<InputFile>` és stream-elés.

Ez a Blazor "lelkivilága". Sokkal strukturáltabb, mint a React, és a típusrendszer végig fogja a kezed. Van kérdésed?