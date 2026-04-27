Folytatjuk a **5. heti laborral**, ami egy rendkívül fontos szoftveres koncepciót jár körbe: a **Jelzéseket (Signals)**. 

A jelzések lényegében a Linux "szoftveres megszakításai". Ezekkel tud az operációs rendszer vagy egy másik program aszinkron módon üzenni a te futó programodnak (pl. hogy "állj le!", vagy "baj van a memóriával!"). A ZH-n az alábbi elméleti megfontolásokat kérik számon:

---

# 5. Labor: Jelzések (Signals) kezelése Linuxban

## 1. A jelzések életciklusa / lépései (KULCSKONCEPCIÓ)
Egy jelzés (signal) feldolgozása a Linux kernelben három jól elkülöníthető fázisra bontható:

1.  **Generation (Kiváltás / Generálás):** Az a pillanat, amikor a jelzés létrejön. Kiválthatja egy felhasználói interakció (pl. `Ctrl+C` lenyomása a terminálban), egy szoftveres hiba (pl. nullával osztás), vagy az OS / másik folyamat (pl. a `kill` parancs kiadása).
2.  **Delivery (Kézbesítés):** Amikor az operációs rendszer (a kernel) ténylegesen átadja a jelzést a célfolyamatnak, és rákényszeríti, hogy reagáljon rá. A generálás és a kézbesítés között eltelhet egy kis idő (ilyenkor a jel *pending*, azaz függőben lévő állapotban van).
3.  **Disposition / Deposition (Rendelkezés / Reakció):** Ez az a szabály, ami megmondja, hogy a folyamatnak *mit kell csinálnia*, amikor a jelzést megkapja (kézbesítik). Ezt a szabályt a folyamat előre beállítja magának.

## 2. Lehetséges reakciók a jelzésekre
Amikor egy folyamat megkap egy jelzést (Delivery), a következő előre definiált módokon (Disposition) reagálhat rá:

* **Ignorálás (Ignore):** A program úgy tesz, mintha mi sem történt volna, eldobja a jelet.
* **Alapértelmezett művelet (Default action):** Ha a programozó nem írt külön kódot a jel kezelésére, a Linux végrehajtja a gyári alapértelmezést. Ez leggyakrabban a folyamat azonnali **leállítása (Terminate)**, esetleg egy memóriakép készítése a hibakereséshez (Core dump), vagy a program ideiglenes szüneteltetése (Stop).
* **Elkapás / Egyedi kezelés (Catch):** A programozó ír egy saját, úgynevezett **Jelkezelő függvényt (Signal Handler)**. Amikor a jelzés beérkezik, a program fő futása megszakad, lefut ez a mi saját függvényünk, majd (ha nem léptünk ki belőle) a főprogram ott folytatódik, ahol abbamaradt.

## 3. Milyen jelzéseket kezelünk, és mit NEM tudunk? (KULCSKONCEPCIÓ)

**Jelzések, amiket tipikusan ELKAPUNK (kezelünk):**
* `SIGINT` (Signal Interrupt): Ezt küldi a terminál, ha nyomsz egy `Ctrl+C`-t.
* `SIGTERM` (Signal Terminate): Ez a szabványos, "finom" leállítási kérés (a sima `kill` parancs alapértelmezése).
* **Miért kezeljük őket?** A **Kulturált leállás (Graceful Shutdown)** miatt! Egy beágyazott rendszerben (pl. ami egy motort vezérel vagy GPIO lábakat kapcsolgat) nagyon veszélyes, ha a program csak úgy "kiszakad" a memóriából. Ha elkapjuk a leállítási kérelmet, a saját függvényünkben lesz időnk: lezárni a nyitott fájlokat, biztonságos állapotba (pl. LOW) húzni a kimeneti lábakat, és csak utána kilépni az `exit()` hívással.

**Jelzések, amiket NEM TUDUNK elkapni (vagy ignorálni):**
Bizonyos jelzések felett a kernel fenntartja az abszolút hatalmat, hogy egy "megvadult" programot mindenképp le lehessen lőni.
* **`SIGKILL` (kill -9):** Azonnali, kíméletlen halál. A kernel szó szerint kilövi a folyamatot, esélyt sem adva neki, hogy lefuttasson bármilyen lezáró (takarító) kódot.
* **`SIGSTOP`:** Felfüggeszti (elaltatja) a folyamatot. Később a `SIGCONT` jellel lehet felébreszteni.

## 4. Jelzések hatása blokkoló API hívásokra (`sleep()` példa) (KULCSKONCEPCIÓ)
Mi történik, ha a programunk éppen blokkolva alszik (pl. vár egy billentyűleütésre a `read()`-del, vagy várakozik a `sleep()` függvénnyel), és közben beérkezik egy jelzés (amit elkapunk)?

* **A hatás:** A beérkező és elkapott jelzés **félbeszakítja (megszakítja) a blokkoló rendszerhívást**!
* **A `sleep()` példa:** Tegyük fel, hogy kiadsz egy `sleep(10)` parancsot (aludj 10 másodpercet). A 3. másodpercben érkezik egy `SIGINT` (Ctrl+C), amihez te írtál egy kezelőfüggvényt (tehát a program nem áll le). 
    A megszakításkezelőd lefut, DE utána a `sleep()` nem folytatja az alvást! A `sleep()` **azonnal visszatér**, és visszatérési értékként megadja a *hátralévő időt* (esetünkben 7-et). 
* **`EINTR` hiba:** Ha ez egy fájlolvasás (`read()`) lett volna, a függvény -1-gyel tért volna vissza, és az `errno` nevű hibaváltozót `EINTR` (Interrupted System Call - Félbeszakított rendszerhívás) értékre állította volna. A programozónak fel kell készülnie erre: egy `while` ciklussal újra kell indítania az olvasást/várakozást, ha ilyen hiba történik.

---
Ezzel is megvagyunk, nagyon szépen rajzolódik ki a tananyag! Jöhet az utolsó, 6. labor (és esetleg a 7. hét anyaga, ha az is van egy fájlban)?