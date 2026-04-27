Szuper, érkezik is a **6. prezentáció (7. hét)** anyaga! Ez a rész a **Szoftverarchitektúrákról, a párhuzamosságról és a szinkronizációról** szól. Itt dől el, hogyan írunk olyan kódot, ami nem fagy le, nem eszi meg a processzort (CPU-t), és biztonságosan kezel egyszerre több feladatot.

Íme a részletes, vizsgára kihegyezett összefoglaló a kulcsszavak alapján!

---

# Szoftverarchitektúrák és párhuzamosság (6. prezentáció)

## 1. Rendszerhívás, várakozás és a feladatok állapotai (KULCSKONCEPCIÓ)
Amikor egy program fut, ritkán végzi csak tisztán a számításokat. Gyakran kell a hardverhez fordulnia (pl. hálózatról adatot olvasni, vagy gombnyomásra várni) egy **rendszerhíváson (syscall)** keresztül.

* **Blokkolás:** Ha az adat még nem áll rendelkezésre (pl. a `read()` függvényt meghívtuk, de még nem jött csomag a hálózaton), az operációs rendszer (OS) **blokkolja** a programot. A program "alszik", kikerül a processzorból, hogy ne pazarolja az erőforrást, és az OS más programoknak adja a CPU-t.
* **A feladat (Taszk / Process) állapotai és átmenetei:**
    1.  **Készen áll (Ready):** A feladat futna, minden adata megvan, de a CPU épp mással van elfoglalva. Sorban áll az ütemezőnél.
    2.  **Fut (Running):** A feladat épp birtokolja a CPU-t és hajtja végre az utasításokat.
    3.  **Blokkolt / Várakozik (Blocked / Waiting):** A feladat valamilyen eseményre (pl. I/O művelet befejezése, vagy egy zár/lock feloldása) vár. *Nem fogyaszt CPU időt.*
* **Állapotátmenetek háttere:**
    * *Ready $\rightarrow$ Running:* Az OS ütemezője (Scheduler) kiválasztja futásra (Dispatch).
    * *Running $\rightarrow$ Ready:* Lejárt a feladat időszelete, vagy egy magasabb prioritású feladat kiütötte (Preemption).
    * *Running $\rightarrow$ Blocked:* A program olyat kért az OS-től (rendszerhívás), amire várni kell.
    * *Blocked $\rightarrow$ Ready:* Az esemény, amire a feladat várt, megtörtént (pl. megjött a hálózati adat, vagy hardveres megszakítás jelezte a művelet végét). A feladat visszakerül a sorba.

---

## 2. Polling és Megszakítás (Interrupt) összehasonlítása (KULCSKONCEPCIÓ)
Hogyan tudja meg a CPU, hogy a periféria végzett egy feladattal (pl. megjött egy bájt az UART-on)? Két alapvető stratégia van:

| Szempont | Polling (Lekérdezés) | Megszakítás (Interrupt) |
| :--- | :--- | :--- |
| **Működés** | A CPU egy ciklusban folyamatosan olvassa a periféria állapotregiszterét: *"Kész vagy? Kész vagy?"* | A CPU csinálja a dolgát, és a periféria egy hardveres jellel szól neki: *"Kész vagyok, gyere ide!"* |
| **CPU terhelés** | **Magas (100%).** A processzor feleslegesen pörög és áramot fogyaszt. | **Minimális.** A CPU addig mást csinálhat vagy aludhat. |
| **Válaszidő (Késleltetés)** | Nagyon gyors lehet (ha a ciklus sűrűn fut), de kiszámíthatatlan, ha a ciklus mást is csinál. | Nagyon gyors és determinisztikus. |

**A Megszakítás (Interrupt) problémái:**
Bár sokkal jobb megoldás, mint a Polling, megvannak a maga árnyoldalai:
* **Overhead (Kontextusváltás):** Amikor jön egy interrupt, a CPU-nak le kell mentenie az aktuális regisztereket, át kell ugrania az ISR-re (Interrupt Service Routine), majd visszatérni. Ez időbe telik.
* **Kiszámíthatatlanság (Jitter):** A megszakítások bármikor érkezhetnek (aszinkron módon), megzavarva a normál programfutás időzítését. Ha több interrupt jön egyszerre, egymást is megszakíthatják (nested interrupts).
* **Konkurencia (Race conditions):** A normál kód és a megszakításkezelő könnyen összeakadhat, ha ugyanazokat a változókat használják (közös erőforrás).

---

## 3. A `poll()` rendszerhívás és az I/O Multiplexálás
Képzeld el, hogy a programodnak 5 különböző hálózati kapcsolatra és 2 szenzorra kell figyelnie egyszerre. Ha a hagyományos blokkoló `read()`-et használod az elsőn, és az nem küld adatot, a programod megáll, és a többi 6-ot észre sem veszed.

* **A `poll()` (vagy a modernebb `epoll()`, `select()`) szerepe:** Ez egy rendszerhívás, aminek átadsz egy *listát* az összes figyelendő fájlról (perifériáról, socket-ről). A `poll()` blokkolja a feladatot, tehát *nem eszik CPU-t*, de **amint BÁRMELYIK fájl a listán olvasásra vagy írásra késszé válik, a feladat felébred**.
* **Alkalmazása:** Eseményvezérelt (event-driven) rendszerek, web- és hálózati szerverek, ahol egyetlen szál (thread) akar hatékonyan, "polling CPU-pazarlás" nélkül kezelni rengeteg perifériát/kapcsolatot.

---

## 4. Többszálú architektúra vs. A `poll()` (KULCSKONCEPCIÓ)
Az előbbi problémát (több dolgora kell figyelni) máshogy is meg lehet oldani:

* **Többszálú (Multithreaded) architektúra:** Létrehozol 7 különböző szálat (thread), és mindegyik kap egy dedikált blokkoló `read()` hívást a saját eszközére.
    * *Előny:* A kód nagyon egyszerű és lineáris ("olvasom, feldolgozom"). Többmagos (Multi-core) CPU-n a szálak fizikailag is egyszerre futhatnak.
    * *Hátrány:* Minden szálnak saját memóriája (stack) van. Rengeteg szál = rengeteg memóriafogyasztás és hatalmas kontextusváltási overhead az OS részéről. Káosz lehet a közös erőforrások használatakor (zárak kellenek).

**Összehasonlítás:**
* **A `poll()` architektúra** kis memóriájú, egyszálas, egymagos környezetben brillírozik (hatékony, nincs szálak közötti szinkronizációs hiba, kevés overhead).
* **A többszálú architektúra** akkor nyer, ha ténylegesen nagy számítási igényű, hosszú ideig tartó párhuzamos adatfeldolgozást kell csinálni egy többmagos processzoron.

---

## 5. Közös Erőforrás és Kölcsönös Kizárás (Mutual Exclusion)
Ha több feladat (szál vagy megszakítás) fut egyszerre, óhatatlanul keresztezik egymás útját.

* **Erőforrás:** Bármi, amit a szoftver használ (egy változó a memóriában, egy I2C busz, egy fájl, egy hardveres periféria regisztere).
* **Közös erőforrás:** Olyan erőforrás, amihez több szál vagy megszakítás fér hozzá *egy időben*.
* **Kölcsönös kizárás (Mutual Exclusion - Mutex):** Az a programozási technika/elmélet, ami garantálja, hogy egy időben **szigorúan csak egyetlen egy** feladat használhatja a közös erőforrást. Ha ezt nem tartjuk be, adatsérülés (Race Condition) történik (pl. az egyik szál félig írja felül a változót, amikor a másik már kiolvassa).

---

## 6. Aktív vs. Passzív várakozás és a Linux Lock-ok (KULCSKONCEPCIÓ)
Ha egy erőforrás foglalt (valaki más használja), a mi szálunknak várnia kell. Ennek két módja van:

* **Passzív várakozás (Blocking wait):** A szál szól az OS-nek, hogy *"Szólj, ha szabad az erőforrás, addig alszom"*.
    * *Hatása:* A szál `Blocked` állapotba kerül. A CPU **0%-ra** esik, más feladatok futhatnak. Viszont a kontextusváltás (elaltatás, felébresztés) időbe telik.
* **Aktív várakozás (Busy waiting / Spinlock):** A szál egy `while(foglalt) {}` ciklusban pörögve másodpercenként milliószor lekérdezi, hogy szabad-e már a zár.
    * *Hatása:* A szál `Running` állapotban marad. A CPU terhelés **100%**.

### Kölcsönös kizárásra szolgáló Linux megoldások:
1. **Mutex (Mutual Exclusion Object):**
    * **Passzív várakozást** használ.
    * Jellegre: Ha foglalt a zár, a kérő szál elalszik. Kiválóan alkalmas hosszú ideig tartó kritikus szakaszok (pl. komplex számítás, I/O várakozás) védelmére a user-space-ben.
2. **RWLock (Read-Write Lock):**
    * Szintén **passzív várakozás**.
    * Jellegre: Kétféle zárolást tesz lehetővé. Ha mindenki csak *olvasni* akarja az adatot, akkor egyszerre több szálat is beenged. Ha viszont valaki *írni* akar (módosítani), azt csak kizárólagosan (mindenki mást kizárva) engedi. Olyan adatoknál jó, amit sokszor olvasnak, de ritkán módosítanak.
3. **Spinlock:**
    * **Aktív várakozást** használ!
    * Jellegre: A kérő szál pörög a CPU-n, amíg meg nem kapja a zárat. Szigorúan csak **nagyon rövid** (pár processzor órajelciklusnyi) kritikus szakaszok védelmére használják, jellemzően a kernel térben vagy megszakításkezelőkben, ahol *tilos* elaludni (blokkolódni), mert az az egész rendszert megbénítaná. Többmagos (SMP) rendszereknél van értelme, ahol egy másik magon futó kód oldja fel a zárat.