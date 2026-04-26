Ez a jegyzet a **VIMIAD04_s1e7_schedule_2026.pdf** dokumentum alapján készült, a **kulcsszavak_ZH1.pdf**-ben kijelölt **6. prezentáció (7. hét)** témaköreit követve.

---

## 6. Prezentáció (7. hét): Ütemezés és szinkronizáció

### **Rendszerhívás, várakozás és blokkolás**
**4. oldal**
* **Rendszerhívás**: Egy olyan mechanizmus, amely során a felhasználói program (User Mode) átadja a vezérlést a kernelnek (Kernel Mode) egy szolgáltatás elvégzése érdekében.
* **Várakozás (Wait)**: Akkor következik be, ha egy programnak egy külső eseményre (pl. hálózati adat megérkezése, gombnyomás) vagy időzítésre van szüksége a folytatáshoz.
* **Blokkolás (Blocking)**: Ha egy rendszerhívás blokkoló jellegű, a hívó folyamat futása felfüggesztődik, amíg az igényelt erőforrás vagy esemény rendelkezésre nem áll. A folyamat ilyenkor nem használ CPU-időt.

### **A feladat állapota és állapotátmenetei**
**7. oldal**
A folyamatok életciklusuk során különböző állapotokon mennek keresztül:
* **Új (New)**: A folyamat létrehozása folyamatban van.
* **Futásra kész (Ready)**: A folyamat várakozik, hogy a processzor (ütemező) hozzárendelje a futási időt.
* **Futó (Running)**: Az utasítások végrehajtása zajlik a CPU-n.
* **Várakozó / Blokkolt (Waiting/Blocked)**: A folyamat valamilyen eseményre (pl. I/O művelet befejezése) vár.
* **Befejezett (Terminated)**: A folyamat befejezte futását.



### **Polling és megszakítás összehasonlítása**
**10-12. oldal**
| Szempont | **Polling (Lekérdezés)** | **Interrupt (Megszakítás)** |
| :--- | :--- | :--- |
| **Működés** | A CPU ciklikusan ellenőrzi a periféria állapotát. | A periféria jelez a CPU-nak, ha esemény történt. |
| **Hatékonyság** | Alacsony, felesleges CPU ciklusokat pazarol. | Magas, a CPU csak akkor foglalkozik az eszközzel, ha kell. |
| **Válaszidő** | Függ a lekérdezési ciklus gyakoriságától. | Gyors, azonnali reakciót tesz lehetővé. |

### **Megszakítás problémái**
**13. oldal**
* **Kontextusváltás (Overhead)**: A megszakítás kiszolgálása (ISR hívás) elmentéssel és visszaállítással jár, ami időbe telik.
* **Megszakítási vihar (Interrupt Storm)**: Túl sűrűn érkező megszakítások teljesen lefoglalhatják a CPU-t, ellehetetlenítve a normál feladatok futását.
* **Nem-determinisztikus futás**: A megszakítások bármikor érkezhetnek, ami megnehezíti a valós idejű garanciák betartását.

### **A poll() rendszerhívás szerepe és alkalmazása**
**15-16. oldal**
* **Szerepe**: Lehetővé teszi, hogy egy folyamat egyszerre több fájlleírót (pl. több hálózati socketet vagy perifériát) figyeljen meg várakozás közben.
* **Működése**: A kernel blokkolja a folyamatot, amíg a listában szereplő fájlleírók közül legalább egynél be nem következik a kért esemény (pl. olvasható adat érkezik).
* **Alkalmazás**: Hatékonyabb, mint minden fájlhoz külön szálat rendelni vagy pollingolni.

### **Többszálú architektúra és összehasonlítása a poll()-lal**
**17, 20. oldal**
* **Többszálúság (Multithreading)**: A programot több, párhuzamosan futó végrehajtási egységre osztjuk. Alkalmazása indokolt, ha valódi párhuzamosságra van szükség (pl. többmagos CPU) vagy komplex háttérfeladatok futnak.
* **Összehasonlítás**:
    * A `poll()` egyetlen szálon kezeli a várakozást, kisebb a memóriaigénye és egyszerűbb a szinkronizációja.
    * A többszálúság rugalmasabb, de bonyolult zárolási mechanizmusokat (mutex) igényel az adatintegritás megőrzéséhez.

### **Erőforrás, közös erőforrás és kölcsönös kizárás**
**22-23. oldal**
* **Erőforrás**: Bármi, amire egy folyamatnak szüksége van (memória, fájl, periféria).
* **Közös erőforrás**: Olyan erőforrás, amelyet egyszerre több szál vagy folyamat is el akar érni.
* **Kölcsönös kizárás (Mutual Exclusion)**: Annak biztosítása, hogy egy időben csak egyetlen szál férhessen hozzá a kritikus szakaszhoz (az erőforráshoz), elkerülve az adatok sérülését (Race Condition).

### **Passzív és aktív várakozás**
**24. oldal**
* **Aktív várakozás (Busy Wait / Spinlock)**: A szál egy ciklusban folytonosan ellenőrzi az erőforrás elérhetőségét. CPU-t pazarol, de nincs kontextusváltási késleltetés.
* **Passzív várakozás (Sleeping / Mutex)**: A szál elalszik (blokkolódik), a kernel pedig felébreszti, ha az erőforrás szabaddá válik. CPU-hatékony, de a felébresztés időbe telik.

### **Kölcsönös kizárásra szolgáló Linux megoldások**
**25-27. oldal**
* **Mutex (Mutual Exclusion)**: A legegyszerűbb zároló elem. Passzív várakozást valósít meg. API: `pthread_mutex_lock()`, `pthread_mutex_unlock()`.
* **Spinlock**: Aktív várakozást használ. Csak nagyon rövid kritikus szakaszoknál javasolt, ahol a kontextusváltás lassabb lenne, mint maga a várakozás.
* **Rwlock (Read-Write Lock)**: Megkülönbözteti az olvasást és az írást. Több olvasó egyszerre bent lehet a kritikus szakaszban, de az író kizárólagos jogot kap.

---
Ezzel a ZH-ra kijelölt összes elméleti és laboratóriumi témakört feldolgoztuk a megadott források alapján. Sok sikert a felkészüléshez! Van még valami, amiben segíthetek?