Elnézést kérek a korábbi előrejelzésemért! Teljesen jogos, szigorúan csak azokat a kiemelt témákat dolgozom ki, amelyek a feltöltött képen a 4. prezentáció (5. előadás) követelményeként szerepelnek. 

Íme a pontosan a képen szereplő vizsgatémakörökre fókuszáló, részletes jegyzet:

***

# Beágyazott Rendszerek - 5. Előadás Jegyzet

## 1. Beágyazott SW architektúrák alapvető tulajdonságai és eltéréseik
* **Bare metal (Csupasz fém):** Nincs operációs rendszer. A szoftver közvetlenül a hardveren fut, jellemzően egy végtelen főciklusból (superloop) és megszakításkezelőkből (ISR) áll. Egyszerű, gyors és alacsony erőforrásigényű, de komplex feladatok esetén a kód nehezen karbantarthatóvá és skálázhatóvá válik.
* **RTOS (Real-Time Operating System):** Valós idejű operációs rendszer. Kifejezetten a determinisztikus működésre és a garantált válaszidőkre tervezték. Kisebb méretű (footprint), biztosít ütemezőt, szinkronizációs primitíveket (szemaforok, mutexek), de általában nem ad teljes körű memóriavédelmet (MMU).
* **OS (Általános célú, pl. Embedded Linux):** Gazdag funkcionalitást nyújt (hálózati stack, fájlrendszerek, grafikus felület, memóriavédelem). Cserébe a válaszidők alapértelmezetten nem determinisztikusak (nem szigorúan valós idejű), és a hardverigénye (RAM, Flash, CPU) jelentős.
* **Hypervisor (Virtualizáció):** Lehetővé teszi több különböző operációs rendszer (például egy RTOS és egy Linux) egyidejű, egymástól teljesen elszigetelt futtatását egyetlen fizikai hardveren. Segít a hardveres erőforrások konszolidációjában és partícionálásában.

## 2. Hard és soft real-time eltérése
* **Hard real-time (Szigorú valós idejű):** A határidő (deadline) lekésése fatális hibát, a rendszer teljes csődjét vagy akár életveszélyes katasztrófát okoz (például egy légzsákvezérlő vagy repülésirányító szoftver esetén).
* **Soft real-time (Lágy valós idejű):** A határidő lekésése nem okoz katasztrófát, csupán a szolgáltatás minőségének (QoS) romlását vagy kényelmetlenséget eredményezi (például egy videó streamelés megakadása vagy egy hálózati csomag késése).

## 3. Ütemezési alapfogalmak
A feladatok (taskok) jellemzésére az alábbi paramétereket használjuk:
* **Task (Feladat):** A rendszer egy végrehajtható funkcionális egysége (szál vagy folyamat).
* **Job (Munkaegység):** A task egy konkrét, egyedi végrehajtási példánya (például egy periodikus task 5. lefuttatása a sorban).
* **Release time:** Kiadási idő; az a pontos pillanat, amikor egy job futásra késszé válik.
* **Execution time:** Végrehajtási idő; a job teljes lefutásához szükséges maximális processzoridő (WCET - Worst Case Execution Time).
* **Deadline:** Határidő; az a legkésőbbi időpont, ameddig a jobnak kötelezően be kell fejeznie a futását.
* **Period:** Periódusidő; periodikus taskok esetén az az időtartam, ami két egymást követő aktiválódás (release time) között eltelik.

## 4. Kooperatív és megszakítható, időosztásos és prioritásos ütemezés
* **Kooperatív vs. Megszakítható (Preemptive) ütemezés:**
    * *Kooperatív:* A CPU-t birtokló task önként adja át a vezérlést az ütemezőnek (például, ha végzett vagy várakozni kezd). Kívülről nem lehet megszakítani. Veszélye: egy hibás, végtelen ciklusba kerülő task az egész rendszert lefagyasztja.
    * *Megszakítható:* Az ütemező bármikor erőszakkal elveheti a CPU-t a futó tasktól (például akkor, ha egy nála magasabb prioritású task futásra késszé válik).
* **Időosztásos vs. Prioritásos ütemezés:**
    * *Időosztásos (Round Robin):* Minden task egy fix időszeletet (quantum) kap. A fő cél a méltányosság (fairness), hogy senki ne éhezzen ki. Általános rendszereknél (PC) jellemző.
    * *Prioritásos:* Minden task kap egy fontossági szintet. Mindig a legmagasabb prioritású, futásra kész task használhatja a processzort. A valós idejű rendszerek alapja.

## 5. Fix és dinamikus prioritás, RM és EDF ütemezés
* **Fix prioritás:** A prioritást fejlesztéskor (vagy a rendszer indulásakor) fixen beállítják, és a futás során sosem változik.
    * *Rate Monotonic (RM) koncepciója:* Fix prioritású algoritmus periodikus feladatokhoz. Alapelve: **Minél rövidebb egy task periódusideje, annál magasabb prioritást kap a rendszerben.**
* **Dinamikus prioritás:** A prioritás futás közben, a rendszer aktuális állapota vagy az idő múlása alapján dinamikusan változik.
    * *Earliest Deadline First (EDF) koncepciója:* Dinamikus prioritású algoritmus. Alapelve: **Egy adott pillanatban mindig az a task kapja meg a legmagasabb prioritást, amelyiknek a határideje (deadline) a legközelebb van.**

## 6. Kölcsönös kizárás, holtpont és a Coffman feltételek
* **Kölcsönös kizárás (Mutual Exclusion):** Kritikus erőforrások (pl. közös memóriaváltozók, perifériák) védelme. Biztosítja, hogy egy megosztott erőforrást egy időben szigorúan csak egy task módosíthasson (jellemzően mutexekkel).
* **Holtpont (Deadlock):** Olyan végzetes, feloldhatatlan állapot, amikor két vagy több task kölcsönösen egymásra (olyan erőforrásokra) vár, amiket a másik task már lefoglalt, így egyikük sem tud soha továbbhaladni.
* **Coffman feltételek:** A holtpont kialakulásának 4 szükséges feltétele. (Ha bármelyiket is megszüntetjük, a holtpont elkerülhető).
    1.  *Kölcsönös kizárás:* Az erőforrást nem lehet egyszerre több taskkal megosztani.
    2.  *Lefoglalva várakozás (Hold and Wait):* Egy task már birtokol legalább egy erőforrást, miközben folyamatosan újakra vár.
    3.  *Nincs megszakítás (No Preemption):* A már lefoglalt erőforrást nem lehet erőszakkal elvenni egy tasktól.
    4.  *Körkörös várakozás (Circular Wait):* Kialakul egy zárt ciklus (hurok) a várakozó taskok láncolatában.

## 7. Prioritás inverzió és megoldásai
* **A jelenség:** Egy anomália, amely során egy **magas prioritású task kénytelen várakozni egy alacsony prioritású taskra**, mert az éppen lefoglalt egy közös erőforrást (mutexet). A probléma akkor válik kritikussá, ha közben egy független, *közepes prioritású task* megszakítja az alacsonyat. Emiatt az alacsony prioritású task nem tudja elengedni az erőforrást, a magas prioritású task pedig kiszámíthatatlan ideig várakozni kényszerül (gyakorlatilag a közepes task blokkolja a magasat).
* **Megoldásai:**
    1.  **Priority Inheritance (Prioritás Öröklés):** Az alacsony prioritású task ideiglenesen "megörökli" a rá várakozó magas prioritású task fontossági szintjét az erőforrás használatának idejére. Így a közepes prioritású taskok nem tudják megszakítani, hamarabb végez, és át tudja adni az erőforrást.
    2.  **Priority Ceiling (Prioritás Plafon):** Minden közös erőforráshoz hozzárendelnek egy "plafon" prioritást (ez egyenlő a legmagasabb prioritású taskkal, ami valaha használni fogja). Amikor egy task lefoglalja ezt az erőforrást, a saját prioritása azonnal megemelkedik erre a plafon szintre, megelőzve az inverziót és a holtpontokat is.