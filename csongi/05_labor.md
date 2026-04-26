Igen, tökéletesen látod! A feltöltött képen szereplő témakörök (jelzéskezelés, időmérés, alvó függvények) **pontosan ebben a most feltöltött `VIMIAD04_lab4.pdf`-ben** találhatók meg. 

Oktatósegédi szerepkörömnek megfelelően, szigorúan a képen kiemelt témák sorrendjében, a vonatkozó diák oldalszámaival kezdve kidolgozom neked a vizsgajegyzetet.

***

# Beágyazott Rendszerek - 4. Labor Jegyzet

## 1. A Linux alkalmazás részei, a signal handler futási kontextusa és a globális változók használata

**3. lap: A Linux alkalmazás részei**
* Egy tipikus, robusztus beágyazott Linux alkalmazás több fő logikai részből épül fel:
    1.  **Inicializálás és parancssori argumentumok feldolgozása** (pl. `getopt`, `getsubopt`).
    2.  **Naplózás beállítása** (logging, pl. `syslog`).
    3.  **Jelzéskezelés (Signal handling) beállítása:** Felkészülés az operációs rendszertől érkező aszinkron eseményekre.
    4.  **Eseményhurok (Event loop):** A program fő végtelen ciklusa (pl. `while(1)`), amely várja és feldolgozza az eseményeket, vagy passzívan várakozik.

**6. lap: A signal handler futási kontextusa és az ebből fakadó problémák**
* **Futási kontextus:** Amikor egy jelzés (signal) megérkezik, az operációs rendszer **aszinkron módon** megszakítja a főprogram futását. A jelzést lekezelő függvény (a *signal handler*) alapértelmezetten a megszakított folyamat (process) **saját normál vermében (stack)** fut le. (Kritikus memóriahibák – pl. `SIGSEGV` – esetén van lehetőség alternatív verem, a `sigaltstack` használatára is, mert olyankor a normál verem már sérült lehet).
* **Az ebből fakadó problémák (Re-entrant / Signal safe kód fontossága):**
    * Mivel a handler bármikor "rárobbanhat" a főprogramra, szigorú szabályok vonatkoznak arra, hogy mit szabad benne csinálni.
    * A handlerben **kizárólag aszinkron jelszint-biztos (async-signal-safe)** függvényeket szabad meghívni.
    * *Mi az a re-entrant (újra beléphető) kód?* Egy függvény akkor re-entrant, ha a végrehajtása közben megszakítható, majd a megszakítási rutinból újra meghívható anélkül, hogy a belső állapota (pl. statikus memóriája) megsérülne.
    * *Tiltott példák:* A `printf()` vagy a `malloc()` **NEM** signal safe függvények! Ha a főprogram épp egy `printf`-et hajt végre (ami egy belső mutexet foglal le a kimenetre), és bejön egy jelzés, majd a handler is hív egy `printf`-et, a program azonnal **holtpontra (deadlock)** fut vagy összeomlik.
    * *Helyes gyakorlat:* A signal handler legyen a lehető legrövidebb! Ne végezzen bonyolult I/O műveleteket, csak állítson be egy jelzőbitet (flag), amire a főprogram a saját idejében reagál.

**13-14. lap: Globális változók használata signal handler esetén**
* Ha a handler csak egy jelzőbitet állít be, azt egy globális változóban kell tárolnia, amit a főprogram (az event loop) folyamatosan vizsgál.
* **A `volatile sig_atomic_t` típus:** Ezt a két kulcsszót kötelező használni a jelzőváltozónál!
    * **`sig_atomic_t`**: Garantálja, hogy a változó írása és olvasása hardveresen **atomi művelet** (egy órajel alatt, megszakíthatatlanul megy végbe). Így sosem fordulhat elő, hogy a főprogram egy félig felülírt, "szemét" értéket olvas ki.
    * **`volatile`**: Megtiltja a C fordítónak, hogy kioptimalizálja a változót (pl. hogy csak egyszer olvassa be a CPU egy regiszterébe). A `volatile` rákényszeríti a programot, hogy minden egyes hivatkozáskor ténylegesen a RAM-ból olvassa ki az értéket, hiszen azt a signal handler a "háttérben", aszinkron módon bármikor átírhatta.

---

## 2. Jelzések kezelése sleep(), nanosleep() és a select() esetében

**16-17. lap: Blokkoló hívások és a jelzések kapcsolata (`EINTR`)**
* Ha a főprogramnak passzívan kell várakoznia (nem pörgeti a CPU-t), blokkoló rendszerhívásokat használ, mint például a `sleep()`, `nanosleep()`, `clock_nanosleep()` vagy az I/O multiplexelésre szolgáló `select()`.
* **Mi történik, ha bejön egy jelzés?**
    * Ezek a várakozó függvények **azonnal megszakadnak**, amint a folyamat kap egy jelzést, és lefut a hozzá tartozó signal handler.
    * Nem várják meg a beállított idő leteltét! A függvények ilyenkor hibával térnek vissza (jellemzően `-1` értékkel), és a globális `errno` változó beállítódik **`EINTR`** (Interrupted system call - megszakított rendszerhívás) értékre.
* **A függvények specifikumai:**
    * **`sleep(seconds)`**: Egész másodperces várakozás. Megszakadás esetén a függvény visszatérési értéke a *még hátralévő (le nem aludt)* másodpercek száma.
    * **`nanosleep(&req, &rem)`**: Nagyon precíz, nanoszekundum pontosságú várakozás. Előnye a `sleep`-pel szemben, hogy ha megszakad, a második paraméterként átadott `rem` (remainder) struktúrába a kernel hajszálpontosan beleírja a **még hátralévő időt**. Ezt felhasználva egy `while` ciklusban azonnal újra lehet indítani a várakozást a maradék idővel.
    * **`clock_nanosleep()`**: Hasonló a `nanosleep`-hez, de itt explicit módon megadható, hogy melyik órához (clock id) viszonyítva akarunk várakozni (pl. `CLOCK_MONOTONIC`, amire nincs hatással a rendszeridő átállítása).
    * **`select()`**: I/O eseményekre vár. Jelzés érkezésekor szintén megszakad `EINTR` hibával. A programozónak ilyenkor manuálisan újra kell indítania a `select()` hívást, ha nem akarta megszakítani a hálózati/fájl kommunikációt.

---

## 3. Az idő mérése

**18. lap: Idő lekérdezési lehetőségek a Linuxban**
Beágyazott rendszerekben gyakran kell időt mérni (pl. periódusidő tartásához vagy timeout számításhoz). Erre két alapvető POSIX C függvény áll rendelkezésre:

1.  **`time()` függvény:**
    * **Felbontás:** Csak **másodperc** pontosságú.
    * **Működés:** A klasszikus UNIX időbélyeget adja vissza (az 1970. január 1., 00:00:00 UTC óta eltelt másodpercek száma). `time_t` típussal tér vissza. Finomabb beágyazott feladatokra (pl. mikroszekundumos szenzorolvasás) a pontatlansága miatt alkalmatlan.

2.  **`clock_gettime()` függvény:**
    * **Felbontás:** **Nanoszekundum** pontosságú (a modern rendszerek alapértelmezett, precíziós időmérője).
    * **Működés:** Egy `timespec` struktúrát tölt fel, ami tartalmazza a másodperceket (`tv_sec`) és a nanoszekundumokat (`tv_nsec`).
    * **Használata:** Különféle órák (Clock ID) kérdezhetők le vele:
        * `CLOCK_REALTIME`: A tényleges falióra idő (Wall-clock time). Hátránya: Ha egy NTP szerver vagy a felhasználó átállítja a rendszeridőt, az idő "ugorhat" előre vagy hátra, ami összezavarja az időtartam-méréseket.
        * `CLOCK_MONOTONIC`: Olyan óra, ami a rendszer indulása (boot) óta csakis monoton növekszik. **Beágyazott rendszerekben időtartamok (eltelt idő) mérésére kötelezően ezt kell használni**, mert teljesen immunis a rendszeridő manuális vagy hálózati átállítására.