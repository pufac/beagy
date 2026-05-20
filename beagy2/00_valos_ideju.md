Szia! Tökéletes, a feltöltött fájlok alapján teljesen világos a kép.

A helyzet a következő: a három PDF szervesen összetartozik az **1. Blokkon (Valós-idejű ütemezés és szoftver architektúrák)** belül.

* Az **Előadás (VIMIAD04_s1e7)** adja a masszív elméleti hátteret: hogyan futnak a folyamatok, miként osztja el a processzoridejüket a Linux, és milyen végzetes hibák (pl. holtpont, prioritás inverzió) léphetnek fel a háttérben.
* A **6. Labor** bemutatja a fizikai hardvert (BeagleBone Black GPIO lábai) és azt, hogyan tudunk a Linux fájlrendszerén (sysfs) keresztül jeleket kiadni (kimenet).
* A **7. Labor** pedig erre építve bemutatja a bemenetek olvasását, és a legfontosabbat: hogyan kapcsolódik össze a szoftveres ütemezés a hardverrel (busy-waiting / "pörgetés" helyett megszakítások és a `poll()` függvény használata).

Mivel kérted, hogy ne "35 évig tartson" átnézni, de a részletességet is meg kell tartani a diaszámokkal együtt, **először a teljes Előadás anyagát dolgozom fel neked veszteségmentesen, de logikusan tömörítve.** Ha ezt átnézted, a következő üzenetben azonnal jöhet a két Labor gyakorlati megvalósítása.

Íme az elméleti blokk tökéletesített jegyzete:

---

### 1. Blokk / Elmélet: Valós-idejű ütemezés és Konkurencia

**[Ea 4-6. dia] Ütemezés és Feladat állapotok**

* Egy feladat a következő állapotokon megy keresztül: **Létrejön** $\rightarrow$ **Futásra kész (Ready)** $\rightarrow$ **Fut (Run)** $\rightarrow$ **Befejeződik**. 


* A futó feladattól a processzort (CPU) csak az operációs rendszer (OS) tudja elvenni. 


* Minden állapotátmenetet **megszakítás (interrupt)** vezérel, az OS teljesen esemény-vezérelt.  Megszakítás lehet:


* 
*Hardveres:* Külső esemény (pl. hálózati csomag érkezik). 


* 
*Kivétel:* Belső hiba (pl. nullával osztás). 


* 
*Szoftveres:* Rendszerhívás az operációs rendszer felé. 





**[Ea 8-12. dia] Valós-idejű szoftver aktivitás**

* 
**Definíció:** Olyan szoftverkódrészlet, amely egy külső esemény hatására **garantált válaszidővel, teljesen lefut** (Run-to-completion). 


* **WCET (Worst-Case Execution Time):** A legrosszabb esetre vonatkozó végrehajtási idő. Ennek ismerete alapkövetelmény a valós-idejűséghez. 


* 
**A modern HW problémája:** A modern processzorok (többszintű cache, utasítás-elágazás becslés, sok mag) és a virtuális tárkezelés *nem determinisztikusak*. A tipikus és a legrosszabb (WCET) végrehajtási idő között nagyságrendi különbség lehet. 


* *Megoldás:* **Heterogén architektúra**. Ami időkritikus, azt dedikált, egyszerű (cache és predikció nélküli) végrehajtóegységen futtatják. 



**[Ea 14-24. dia] Folyamat (Process) vs. Szál (Thread)**

* 
**Folyamat:** A végrehajtás alatt álló program (nehézsúlyú). 


* Saját, védett memóriával rendelkezik: **Kód, Adat, Halom (Heap), Verem (Stack)**. 


* Létrehozása (`fork()`) és kommunikációja (IPC) rendkívül erőforrás-igényes, mert az OS-en keresztül történik (a virtuális memória szeparációja miatt). 




* **Szál (Thread):** Pehelysúlyú folyamat. Egy folyamaton belül több szál is futhat. 


* Saját virtuális CPU-ja és **saját verme (stack)** van. 


* Viszont **osztozik a kódon, adaton és a halmon (heap)** a folyamat többi szálával. 


* 
*Előnye:* Nagyságrendekkel gyorsabb létrehozás és villámgyors kommunikáció a közös memórián át. 


* 
*Veszélye:* A közös memória miatt sérülhet az adatok konzisztenciája, az operációs rendszer által biztosított **kölcsönös kizárást (mutex)** kell alkalmazni. 





**[Ea 26-33. dia] Ütemezés és a Linux Ütemező (Scheduler)**

* 
**Időskálák:** * *Rövid távú:* CPU ütemezés, ami gyakran (pl. 10-20 ms) dönt a futásra kész feladatok közül. 


* *Középtávú:* Memóriakezelés (Swapping). **Beágyazott rendszerben ezt kerülni kell**, mert tönkreteszi a Flash memóriát és kizárja a valós-idejűséget! 


* 
*Hosszú távú:* Rendszer indulásakor vagy időzítve lefutó feladatok (`crontab`, `systemd`). 




* **Linux Ütemezési Osztályok:**
* **SCHED_OTHER:** Alapértelmezett, nem valós-idejű feladatok. (2.6-os kerneltől CFS algoritmus, 6.6-tól az új EEVDF algoritmus). 


* 
**SCHED_FIFO / SCHED_RR:** Valós-idejű ütemezők (First-In-First-Out és Round-Robin). 


* 
**SCHED_DEADLINE:** Hard real-time (Earliest Deadline First) ütemező. 




* **Preempt-RT:** Régen egy "patch" volt, a 6.12-es Linux kerneltől gyárilag beépített. Képessé teszi a kernelt arra, hogy szinte bárhol megszakítható legyen, drasztikusan csökkentve a késleltetést (latency), bár az áteresztőképesség (throughput) rovására. 



**[Ea 34-43. dia] Többprocesszoros ütemezés és Időkezelés**

* **Processzor Affinitás (Processor Affinity):** A cache felépítése (és a cache koherencia hiánya a magok között) miatt egy feladatot érdemes ugyanazon a processzormagon tartani, különben a cache-t újra kell építeni, ami nagyon lassú. Lehet laza (OS törekszik rá) vagy kemény (programozó kényszeríti). 


* **SMP vs. NUMA:** SMP-nél csak a cache a gond. NUMA (memóriazónák) esetén a memóriához való fizikai távolság is számít; a feladatnak a "közelebbi" memóriát kell használnia. 


* **Időkezelés (Tickless kernel):** A feladatokat időzítő megszakítások (System Ticks / Jiffies) ütemezik (kb. 4 ms-onként). A **Tickless kernel** lényege, hogy ha a rendszer üresjáratban van, kikapcsolja a folyamatos óraütést, így rengeteg energiát spórol (pl. okostelefonoknál), és csak külső eseményre ébred fel. 



**[Ea 45-48. dia] Párhuzamos programozás hibái (Konkurencia)**

* **Versenyhelyzet (Race condition):** Közös erőforrás hibás (nem védett) használata. Időzítésfüggő, nehezen reprodukálható hiba. 


* 
**Kiéheztetés (Starvation):** A rendszer működik, de egy bizonyos feladat sosem jut processzoridőhöz vagy erőforráshoz. 


* **Holtpont (Deadlock):** A legsúlyosabb hiba. Feladatok körkörösen várnak egymás lefoglalt erőforrásaira. Nincs futásra kész folyamat, a **rendszer lefagy**. Beágyazott rendszernél katasztrófa. 


* **Livelock:** A holtpont feloldásának hibás próbálkozása. A rendszer dolgozik, de nem halad (példa: két ember folyton ugyanabba az irányba lépve kerülgeti egymást az ajtóban). 



**[Ea 54-65. dia] Prioritás inverzió (Priority Inversion)**

* 
**A probléma (pl. Mars Pathfinder hiba):** 1.  Van egy alacsony (T1), egy közepes (T2) és egy magas (T3) prioritású feladat. 
2.  T1 lefoglal egy 'A' erőforrást. 
3.  T3 felébred, használná az 'A' erőforrást, de mivel T1 fogja, T3 blokkolódik (T1-re vár). 
4.  Ekkor felébred a CPU-intenzív T2. Mivel T2 prioritása nagyobb, mint T1-é, elveszi a CPU-t T1-től. 
5.  **Eredmény:** T2 fut a végtelenségig, T1 nem tudja befejezni a munkát és elengedni 'A'-t, így a legmagasabb prioritású T3 egyáltalán nem jut szóhoz. 


* **Megoldás 1: Prioritás öröklés (Priority Inheritance - PI):** Futási idejű megoldás. Amikor T3 várni kezd T1-re, T1 átmenetileg **megörököli T3 magas prioritását**, így T2 nem tudja beelőzni, be tudja fejezni a dolgát, és átadja a stafétát T3-nak. 


* **Megoldás 2: Prioritás plafon (Priority Ceiling - PC):** Fordítási idejű megoldás. Az 'A' erőforráshoz előre hozzárendelnek egy maximális prioritást. Bárki foglalja le, megkapja ezt a "plafon" prioritást a használat idejére. 



---

Ezzel a gerinc elméletét lefedtük, egyetlen definíció sem maradt ki. **Rögtön folytathatom a GPIO és a sysfs/poll Labor anyagok összefoglalásával (szintén így, egyértelmű struktúrába öntve)?**