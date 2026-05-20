Szia! Megértettem, a 2. ZH-nál valóban mélyebbre kell ásnunk, úgyhogy a 7. előadás (Ütemezés) anyagát most a legapróbb részletekig, a diákon szereplő technikai finomságokkal együtt dolgozzuk fel.

*(Megjegyzés: A rendszerem szigorú belső szabályzata megköveteli a forrásmegjelölések `` formátumban történő alkalmazását minden kinyert információnál, így ezeket nem tudom teljesen kikapcsolni. Igyekszem őket a mondatok/bekezdések végére rejteni, hogy a legkevésbé zavarják az olvasást a GitHubon!)*

Íme a nagyon részletes, a 2. ZH kulcsszavaira fókuszáló összefoglaló a 7. előadás alapján.

---

# Ütemezés és Valós-idejű Ütemezés (7. előadás)

## 1. Valós-idejű szoftver aktivitás

A beágyazott rendszerek alapja a valós-idejűség, aminek nagyon szigorú definíciója és feltételrendszere van.

* **Definíció:** Egy olyan szoftveres kódrészlet (aktivitás), amely egy adott külső esemény hatására **garantált válaszidővel teljesen lefut** (run-to-completion).
* **Függetlenség:** Ennek a lefutásnak függetlennek kell lennie a rendszerben lévő többi szoftveres aktivitástól és a rendszer pillanatnyi állapotától.
* **WCET (Worst-Case Execution Time):** Az alapkövetelmény, hogy a legrosszabb esetre vonatkozó végrehajtási idő (WCET) mindig rövidebb legyen, mint az előírt válaszidő. Ha egy algoritmus iterációinak számára nem adható felső korlát, az nem lehet valós-idejű.

### Klasszikus vs. Modern végrehajtási környezet

* **Klasszikus CPU modell:** Teljesen determinisztikus. Állandó az utasítások végrehajtási ideje, nincs CACHE (vagy prediktálható), nincs megszakítás (csak polling), egy mag futtatja a kódot, és a perifériák elérési ideje fix. Itt a WCET könnyen számolható.
* **Modern CPU környezet (pl. ARM Cortex-A, x86):** Extrém módon **nem determinisztikus**, mert az áteresztőképességre (throughput) és nem a késleltetésre (latency) vannak optimalizálva.
* *Zavaró tényezők:* Többszintű CACHE, szuperskalár és out-of-order végrehajtás, branch prediction (elágazás becslés), virtuális tárkezelés, DMA és periféria pufferek.
* *Megoldás:* Heterogén architektúrák. Az időkritikus feladatokat dedikált, egyszerűbb végrehajtóegységekre (pl. egy különálló kis mikrokontroller magra) teszik a SoC-on belül, ahol nincs spekulatív végrehajtás és a memória gyorsan, késleltetés nélkül elérhető.



## 2. Feladatok megvalósítása: Folyamat (Process) és Szál (Thread)

### Folyamat (Process)

* **Fogalma:** A végrehajtás alatt álló program (heavyweight process). Ugyanabból a programból több független folyamat is futhat.
* **Tulajdonságai és Szeparációja:** Minden folyamat saját, védett virtuális memóriaterülettel rendelkezik (Verem, Halom, Adat, Kód), amit az MMU és az operációs rendszer biztosít. Más folyamat memóriájába nem láthat bele.
* **Létrehozása:** Erőforrás-igényes (sok adminisztráció). Windows alatt `CreateProcess()`, Linux alatt `fork() + exec()` API hívásokkal történik. (A FreeRTOS nem is támogatja a folyamatokat, csak a szálakat!). Létrejön egy szülő-gyermek hierarchia (Process fa).
* **Kommunikáció (IPC):** Mivel a memóriájuk szeparált, csak nagyon lassú, OS-szintű rendszerhívásokkal (Inter-Process Communication) tudnak adatot cserélni.
* **Befejezés:** `exit()` vagy `return` hívással. A nyitott fájlokat lezárja, a visszatérési értéket a szülő kapja meg. Ha a szülő meghal, az OS kezeli le az árva folyamatot (pl. init folyamat veszi át).

### Szál (Thread)

* **Fogalma:** Pehelysúlyú folyamat (lightweight process), a CPU-használat alapértelmezett egysége.
* **Tulajdonságai:** Egy folyamaton belül több szál futhat. **Minden szálnak saját virtuális CPU-ja és saját Verme (Stack) van**, DE közösen osztoznak a folyamat Kód, Adat és Halom (Heap) memóriáján, valamint a nyitott fájlokon.
* **Támogatás és API:** Modern OS-ek natívan támogatják (Linux ütemező "taskokat" ütemez, amik lehetnek szálak is). Tipikus API: `Pthreads` (Linux) vagy a Java threadjei.
* **Összehasonlítás a folyamattal (Előny/Hátrány):** * *Előny:* Létrehozásuk és megszüntetésük kb. egy nagyságrenddel (10x) gyorsabb és kevesebb erőforrást igényel. A közös memória miatt a kommunikáció extrém gyors.
* *Hátrány:* A közös memória használata veszélyes. Ha nem használunk kölcsönös kizárást (lockokat), az adatok sérülnek (versenyhelyzet).



## 3. Ütemezés és Időskálái

Az ütemező (Scheduler) dönti el, hogy a "Futásra kész" (Ready) sorból melyik feladat kapja meg a processzort (Running állapot). A modern operációs rendszerek megszakítás vezéreltek, az állapotátmenetek mindig valamilyen megszakítás (HW IT, SW syscall, kivétel) hatására történnek.

**Időskálák:**

1. **Rövid távú (CPU ütemező):** Sűrűn, 10-20 ms-enként (vagy gyorsabban) lefut, kiválasztja a következő futó folyamatot.
2. **Középtávú (Swapping):** Ha a memória betelik, a feladatok memóriáját a háttértárra (HDD/SSD) mozgatja. *Beágyazott rendszerben ez kritikus probléma:* tönkreteszi az SD kártyát/Flash-t a sok írással, és a valós-idejűséggel is összeegyeztethetetlen (nem tudjuk, mikor töltődik vissza a kód a memóriába). Megoldás: ki kell kapcsolni a swap-et.
3. **Hosszú távú:** Bootoláskor, időzítve vagy nagy eseményekre indít feladatokat (pl. `crontab`, `systemd`).

## 4. Linux Ütemezés és Valós-idejű osztályok

A Linuxban többféle ütemező algoritmus létezik egyszerre, amelyeket **ütemezési osztályokba (scheduling classes)** sorolnak. Prioritás szerint növekvő sorrendben a legfontosabbak:

1. **`SCHED_OTHER`:** Ez a normál, nem valós-idejű programok osztálya. Régebben a CFS (Completely Fair Scheduler) végezte, a modern kernelekben (6.6-tól) az EEVDF algoritmus ütemezi őket.
2. **`SCHED_RR` (Round-Robin):** Valós-idejű ütemező, amely azonos prioritású feladatok között időszeletekkel váltogat.
3. **`SCHED_FIFO`:** Valós-idejű ütemező, időszeletelés nélkül. Ha egy magas prioritású FIFO feladat futni kezd, addig fut, amíg magától le nem mond a CPU-ról, vagy egy *még magasabb* prioritású feladat ki nem üti.
4. **`SCHED_DEADLINE`:** A legmagasabb prioritású, Earliest Deadline First (EDF) elven működő, garantált határidőket biztosító ütemező.

*Ütemező befolyásolása:* A Linuxban a `nice` és `chrt` parancssori eszközökkel (vagy programozottan rendszerhívásokkal) tudjuk állítani a feladatok prioritását és ütemezési osztályát.

### Preempt-RT Patch

A sima Linux kernel bizonyos rendszerhívások közben nem megszakítható (nem adja át a CPU-t egy fontosabb feladatnak azonnal).

* **Célja:** A Preempt-RT patch a kernelt teljesen (hard) valós-idejűvé teszi azáltal, hogy szinte mindenhol megszakíthatóvá (preemptable) teszi a kernel kódját.
* **Következmény/Mérlegelés:** Brutálisan lecsökkenti a válaszidőt (latency), cserébe megnő az overhead, így a rendszer általános áteresztőképessége (throughput) csökken. (A 6.12-es kerneltől már be van építve a fába).

## 5. Többprocesszoros ütemezés

Modern, többmagos (SMP - Symmetric Multiprocessing vagy NUMA - Non-Uniform Memory Access) környezetben az ütemező feladata drasztikusan bonyolódik.

* **Processzor affinitás (Processor Affinity):** A processzormagok saját, lokális CACHE memóriával rendelkeznek. Ha az OS egy feladatot átdob az 1-es magról a 2-es magra, a kód és az adat nincs ott a 2-es mag CACHE-ében, újra be kell tölteni a lassú RAM-ból. **Cél:** A feladatot mindig ugyanazon a magon tartani. Létezik Laza (OS próbálkozik) és Kemény (garantált, API-n beállított) affinitás. NUMA architektúra esetén még bonyolultabb, mert a távoli RAM elérése is drasztikusan lassítja a programot.
* **Terhelés megosztás (Load balancing):** Hogy ne csak egy mag dolgozzon, a feladatokat el kell osztani. Két módszer:
* *Push:* Egy OS folyamat figyeli a magokat, és a terhelt magról áttolja a feladatot egy üresre.
* *Pull:* Az üres (idle) mag bekukkant a többi mag feladatsorába (futásra kész sor), és áthúz magához egy feladatot. A Linux mindkettőt (Push/Pull) alkalmazza. (A big.LITTLE rendszereknél, ahol teljesítmény és takarékos magok vannak vegyesen, ez különösen nehéz feladat).



## 6. Időkezelés a kernelben (Virtuális / SW Timer)

Az OS-nek követnie kell az időt a timeout-okhoz (`sleep()`, `poll()`), és az időszeleteléshez.

* **System tick (jiffies):** A hardveres timer periodikusan (pl. 4 ms vagy 1 ms) generál egy megszakítást, ilyenkor az OS frissíti a rendszeridőt.
* **SW Timer felépítése:** Ha kiadunk egy 10 másodperces `sleep()`-et, az OS nem áll meg. Kiszámolja a *Futási időpontot (Aktuális idő + Kért várakozás)*, és ezt egy listába teszi sorba rendezve. Minden system tick-nél az OS csak a lista elejét nézi meg: eljött-e már a legkisebb időpont? Ha igen, a feladat "Futásra kész" állapotba kerül.
* **Felbontás és pontosság:** A pontosságot a megszakítások gyakorisága adja (kvantálási hiba). Ha a tick 1 ms, nem tudunk fél milliszekundumot várni.
* **Nagy pontosságú mérés:** Ehhez a CPU belső hardveres számlálóját (Timestamp Counter - TSC) kell kiolvasni, aminek a felbontása az órajel (pl. 10 ns).
* **Tickless kernel:** Mobiloknál az akku kímélése érdekében az OS kikapcsolja a periodikus tick-et, ha a CPU üresjáratban (idle) van, és csak külső eseményre (pl. hálózati csomag) ébred fel.

## 7. Párhuzamos programozásból következő hibák

Ezek a hibák a közös erőforrások (pl. memóriaváltozók) nem megfelelő védelméből fakadnak. Nem determinisztikusak, időzítésfüggőek, nehéz őket debugolni.

* **Versenyhelyzet (Race condition):** Több szál ír/olvas egyszerre egy közös változót védelem (mutex) nélkül. Az adat megsérül, inkonzisztenssé válik.
* **Kiéheztetés (Starvation):** Egy feladat soha nem jut hozzá a szükséges erőforráshoz (pl. a magas prioritású feladatok teljesen lefoglalják a CPU-t, így a legkisebb prioritású "éhen hal").
* **Holtpont (Deadlock):** A legsúlyosabb hiba (rendszerfagyás). Két feladat vár egymásra úgy, hogy a feloldáshoz szükséges erőforrást kölcsönösen birtokolják. Példa: A feladat lefoglalja az X erőforrást és vár Y-ra. B feladat lefoglalja Y-t és vár X-re.
* **Livelock:** Ugyanaz a végeredmény, de a folyamatok nem állnak meg, hanem aktívan pörögnek (pl. kölcsönösen folyamatosan elengedik és újra megpróbálják lefoglalni az erőforrást, de érdemi munkát nem végeznek, "két kiskecske a hídon" szindróma).

## 8. Prioritás Inverzió (KIEMELT ZH KÉRDÉS!)

Ez egy klasszikus probléma valós-idejű OS-ekben (pl. emiatt fagyott le a Mars Pathfinder).
**A feltétele:** Kell hozzá 3 feladat (Magas (T3), Közepes (T2), és Alacsony (T1) prioritás), valamint egy közös Erőforrás, amit a Magas és az Alacsony is használni akar.

**A folyamat:**

1. A T1 (alacsony) feladat fut, és lefoglalja az Erőforrást.
2. A T3 (magas) feladat felébred, kiüti a T1-et, de látja, hogy az Erőforrás foglalt, ezért blokkolódik (elalszik), amíg T1 el nem engedi azt.
3. Ekkor T1 folytatná a futást, DE felébred a T2 (közepes) prioritású, CPU-intenzív feladat.
4. **Az eredmény (Inverzió):** A T2 kiüti a T1-et, és elkezd futni. Mivel a T1 nem kap CPU-t, nem tudja elengedni az erőforrást. Mivel az erőforrás foglalt marad, a T3 sosem indul el. *Végeredményben a közepes prioritású feladat feltartotta a legmagasabb prioritású, valós-idejű feladatot!*.

**Megoldási lehetőségek:**

1. **Prioritás öröklés (Priority Inheritance - PI):** Futási idejű (dinamikus) megoldás. Amikor a T3 (magas) elkezd várni a T1-re, a T1 **ideiglenesen megörökli** a T3 magas prioritását. Emiatt a T2 (közepes) nem tudja kiütni. A T1 nyugodtan befejezi a munkát, elengedi az erőforrást (visszaesik alacsony prioritásra), és a T3 azonnal átveheti a futást.
2. **Prioritás plafon (Priority Ceiling - PC):** Fordítási idejű (statikus) megoldás. Minden közös erőforráshoz hozzárendelnek egy "plafon prioritást" (ami a legnagyobb őt használó feladat prioritása). Ha a T1 lefoglalja az erőforrást, a prioritása azonnal felugrik erre a plafon szintre. Így eleve esélytelen, hogy bárki megszakítsa, míg dolgozik vele.

---

Ez a rész rendkívül fontos vizsgaanyag. Készen állsz a folytatásra a 10. Laborral (Időkezelés, PTP, Chrony stb.)?