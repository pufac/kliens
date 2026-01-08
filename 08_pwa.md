Megérkezett a PWA (Progressive Web Apps) diasor. Ez egy teljesen más téma, mint a React komponensek. Itt nem a UI-t legózzuk össze, hanem azt próbáljuk elérni, hogy **a weboldalunk úgy viselkedjen, mint egy telepített .exe vagy mobil app.**

Ez a "híd" a web és a natív világ között. Nézzük végig, hogyan verjük át az operációs rendszert, hogy alkalmazásnak higgyen minket.

---

# Progressive Web Apps (PWA) – A Weboldal, ami Appnak képzeli magát

## 1. Mi az az Electron? (2. dia) – Csak a kontextus miatt
Mielőtt a PWA-ba vágnánk, egy gyors kitérő.
*   **Mi ez?** Egy technológia, amivel webes nyelveken (HTML/JS) írhatsz asztali alkalmazást (pl. Discord, VS Code, Slack).
*   **Hogy működik?** Becsomagolnak egy komplett Node.js-t (szerver a gépeden) és egy Chrome böngészőt egy `.exe`-be.
*   **Baja:** Dögnehéz. Egy Hello World app is 100MB+ és zabálja a RAM-ot, mert minden apphoz jár egy saját böngésző motor.
*   **PWA válasza:** Ne csomagoljunk böngészőt, használjuk azt, ami már ott van a felhasználó gépén!

---

## 2. Mi az a PWA? (3-5. dia)

**A lényeg:** Egy weboldal, ami:
1.  **Telepíthető:** Kitesz egy ikont az asztalra/telefonra.
2.  **App-szerű:** Eltűnik a böngésző címsora (URL bar), teljes képernyőn fut.
3.  **Offline képes:** Ha kihúzod a netkábelt, akkor is működik (nem a dínós játék jön be).
4.  **OS integráció:** Tud értesítést küldeni (Push), fájlokat kezelni.

**A "Szentháromság", ami kell hozzá (5. dia):**
Ahhoz, hogy a Chrome felajánlja a "Telepítés" gombot, három dolog **kötelező**:
1.  **HTTPS:** Biztonságos kapcsolat (kivéve localhost fejlesztésnél).
2.  **Manifest fájl:** Egy "személyi igazolvány" az appnak (név, ikon, szín).
3.  **Service Worker:** Egy háttérben futó szkript, ami megoldja az offline működést.

---

## 3. A Manifest – Az App Személyi Igazolványa (6-9. dia)

Ez egy sima JSON fájl (`manifest.json`), ami leírja az appot az operációs rendszernek.

**Hogyan kötjük be? (6. dia):**
A HTML `<head>`-be kell tenni:
```html
<link rel="manifest" href="manifest.json" />
```
*(Az iOS, mint mindig, különcködik, nekik külön `<meta>` tagek kellenek, lásd a 6. dia kódját).*

**A fájl tartalma (7. dia):**
```json
{
  "name": "My Chat App",           // Hosszú név
  "short_name": "MyChat",          // Ikon alatti név
  "start_url": "index.html",       // Mit nyisson meg indításkor
  "display": "standalone",         // KULCSFONTOSSÁGÚ: Ez tünteti el a böngésző keretet!
  "icons": [ ... ]                 // Ikonok többféle méretben (Androidnak, Windowsnak...)
}
```
Ez tisztán konfiguráció, itt nincs programozás. Ha ezt megcsinálod, mobilon már ki lehet tenni a kezdőképernyőre "szép" ikonnal.

---

## 4. Service Worker – A Hálózat Rendőre (10-18. dia)

Ez a PWA lelke, és programozói szempontból a legérdekesebb/legnehezebb rész.

**Mi ez? (11. dia)**
A Service Worker (SW) egy **JavaScript fájl**, ami a böngésző és a hálózat KÖZÖTT ül.
*   **Független:** Nem a weboldaladban fut! Külön szálon, a háttérben.
*   **Életciklus:** Akkor is futhat, ha bezártad a weboldalt (pl. push értesítés fogadása).
*   **Proxy:** Minden hálózati kérés (kép letöltés, AJAX hívás) átmegy rajta. Ő dönt: "Ezt engedem a netre", vagy "Ezt kiszolgálom a zsebemből (cache)".

**Hogyan telepítjük? (13. dia)**
A *fő* JavaScript kódunkban (nem a Service Workerben!) regisztráljuk:
```javascript
// index.js (Fő szál)
// "Hahó böngésző, használd az sw.js-t service workernek!"
let reg = await navigator.serviceWorker.register('sw.js');
```

**Működése – Fetch esemény (14. dia):**
Ez a legdurvább rész. Az `sw.js`-ben elkaphatjuk a hálózati kéréseket.
```javascript
// sw.js (Service Worker szál)
self.addEventListener('fetch', event => {
    // Megállítjuk az eredeti kérést, és mi válaszolunk helyette (respondWith)
    event.respondWith(
        // Megnézzük: megvan ez a fájl a Cache-ben?
        caches.match(event.request)
            .then(response => {
                // Ha megvan (response nem null), visszaadjuk a Cache-ből.
                // Ha nincs (null), akkor elindítjuk a valódi netes kérést (fetch).
                return response ?? fetch(event.request); 
            })
    );
});
```
*Ez a kód teszi lehetővé az offline működést.* Ha nincs net, de a fájl a cache-ben van, az oldal betöltődik.

**Életciklus (15-16. dia):**
A SW frissítése trükkös.
1.  Ha módosítod az `sw.js` kódját, a böngésző letölti az újat.
2.  De **NEM** aktiválja azonnal, amíg a régi verziót használó ablakok nyitva vannak (hogy ne törje el a futó appot).
3.  "Waiting" állapotba kerül. Csak újraindítás után veszi át az uralmat.

**Web Worker vs Service Worker (12. dia):**
*   **Web Worker:** Csak arra való, hogy ne fagyjon le a UI, ha számolni kell (multithreading). Csak addig él, amíg az oldal nyitva van.
*   **Service Worker:** Hálózati proxy és offline képességek. Túlélheti az oldal bezárását.

---

## 5. Cache Stratégiák – Mit mikor tároljunk? (19-28. dia)

Hogyan döntsük el, mit szolgáljunk ki cache-ből és mit a netről? Erre vannak "Design Pattern"-ek.

**Cache API (21. dia):**
Ez nem a HTTP cache! Ez egy programozható tárhely, amit mi vezérlünk JS-ből.
*   `caches.open('tarolo-neve')`: Megnyitunk egy dobozt.
*   `cache.addAll(['/index.html', '/style.css'])`: Letöltjük és eltesszük a fájlokat.

**Stratégiák:**

1.  **Cache Only (24. dia):**
    *   Mindent csak a cache-ből próbálunk. Ha nincs ott -> Hiba.
    *   *Mire jó?* Számológép app, amihez nem kell net.

2.  **Network Only (25. dia):**
    *   Mindent a netről szedünk. (Ez a sima weboldal működése).
    *   *Mire jó?* Ha nincs SW, ez az alapértelmezett.

3.  **Cache, majd Hálózat (Cache First) (26. dia):**
    *   Először megnézzük a cache-t. Ha ott van, visszaadjuk (gyors!). Ha nincs, irány a net.
    *   *Mire jó?* Képek, CSS, JS fájlok, amik ritkán változnak. (Ez volt a példa a 14. dián).

4.  **Hálózat, majd Cache (Network First) (27. dia):**
    *   Megpróbáljuk letölteni a frisset. Ha sikerül, szuper (és elmentjük cache-be). Ha nincs net, akkor adjuk a régit a cache-ből.
    *   *Mire jó?* Hírek, friss adatok, ahol fontos az aktualitás, de offline is mutatni kell valamit.

5.  **Hálózat és Cache (Stale-while-revalidate) (28. dia):**
    *   A legbonyolultabb. Azonnal visszaadjuk a régit a cache-ből (hogy gyors legyen), de a háttérben letöltjük a frisset, elmentjük, és szólunk a UI-nak, hogy "hé, van új adat, frissíts!".

---

## 6. Kommunikáció a Szerverrel (29-34. dia)

Hogyan beszélget a JS a szerverrel? (Ez nem csak PWA, ez általános web).

**1. XHR (XMLHttpRequest) (30-31. dia) - A RÉGI:**
*   Ezzel indult az AJAX forradalom.
*   Eseményvezérelt (`addEventListener('load')`), csúnya, callback-alapú.
*   Ma már ritkán írjuk kézzel.

**2. Fetch API (32. dia) - AZ ÚJ:**
*   Modern, Promise alapú.
*   Szebb szintaxis (`await fetch(...)`).
*   **Hátrány:** Alapból nem dob hibát (exceptiont), ha a szerver 404-et vagy 500-at válaszol (csak hálózati hibánál). Ezt kézzel kell ellenőrizni (`if (!res.ok) ...`).

**3. WebSocket (33-34. dia) - ÉLŐ KAPCSOLAT:**
*   A HTTP kérés (XHR/Fetch) olyan, mint a levél: elküldöd, vársz, megjön a válasz, vége.
*   A WebSocket olyan, mint a telefonhívás: felépül a kapcsolat, és nyitva marad.
*   **Kétirányú:** A szerver is tud küldeni bármikor adatot a kliensnek (pl. chat üzenet, tőzsdei árfolyam).
*   *Fontos:* Nincs cache, nincs tömörítés (brotli), ez egy nyers adatcsatorna.

---

## 7. Extrák: Share API (35-36. dia)

Hogyan érjük el a mobil "Megosztás" gombját?

*   **Kifelé (35. dia):** `navigator.share({ title: "...", url: "..." })`. Ez feldobja a natív megosztó panelt (Messenger, Email, stb.).
*   **Befelé (36. dia):** Mi is megjelenhetünk a megosztási listában! Ehhez a `manifest.json`-ban kell beállítani a `share_target`-et. Ha valaki megoszt egy képet, az operációs rendszer a mi weboldalunkat nyitja meg, és POST kéréssel átadja az adatot.

---

### Összefoglalás "normális" programozónak:

1.  **PWA:** Egy weboldal, ami kap egy JSON konfigot (`manifest`) és egy háttérfolyamatot (`Service Worker`), hogy az OS appnak nézze.
2.  **Service Worker:** Egy **kliens oldali proxy szerver**. Te írod JS-ben. Minden kérés átmegy rajta. Eldöntheted, hogy a netről szolgálod ki, vagy egy lokális adatbázisból (Cache API).
3.  **Offline mód:** Nem mágia. A SW az első látogatáskor letölti a fájlokat (`html`, `css`, `js`) a Cache-be. A második látogatáskor, ha nincs net, a SW kiveszi őket a Cache-ből és visszaadja a böngészőnek. A böngésző észre se veszi, hogy nem a szervertől kapta.
4.  **Kommunikáció:** Használj `fetch`-et (modern HTTP) vagy `WebSocket`-et (real-time). Az `XHR` a múlté.