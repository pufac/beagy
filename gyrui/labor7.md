Szia! Íme a **7. labor (GPIO bemenet és a poll() rendszerhívás)** rendkívül részletes, mérnöki fókuszú összefoglalója. Ez az anyag köti össze az elméleti megszakítás-kezelést a gyakorlati Linux programozással.

*(A hivatkozásokat a kérésednek megfelelően a mondatok, illetve bekezdések végére csúsztattam, így a szöveg tiszta és jól olvasható marad!)*

---

# 7. Labor: GPIO bemenetek és Eseményvezérelt várakozás

## 1. A GPIO bemenet és a klasszikus Polling problémája

Amikor egy mikrovezérlőn (pl. BeagleBone) egy gombnyomásra várunk, a GPIO lábat bemenetté kell konfigurálni (a `sysfs`-ben az `echo in > direction` paranccsal).

* **A naiv megközelítés (Busy Waiting / Polling):** Ha a C programunkban egy végtelen `while(1)` ciklusban folyamatosan olvassuk a `value` fájlt (pl. `read()` hívással), a programunk ugyan azonnal reagál a gombnyomásra, de **a CPU terhelése 100%-ra ugrik**.
* **Következmény:** Ez beágyazott Linux környezetben megengedhetetlen, mert felemészti a processzoridőt, melegszik a chip, és a többi, valóban fontos futó feladat (taszk) elől veszi el az erőforrást.

## 2. Hardveres megszakítás szimulálása Linux alatt (`edge`)

Hogy elkerüljük az aktív várakozást, a Linux kernel lehetővé teszi, hogy a fizikai GPIO megszakításokat (interruptokat) felhasználói térből (user-space) is lekezeljük.
Ehhez a `sysfs` felületen a lábhoz tartozó **`edge`** fájlt kell konfigurálni. A lehetséges értékek:

* `rising` (Felfutó élre generáljon eseményt).
* `falling` (Lefutó élre generáljon eseményt).
* `both` (Mindkét élváltozásra reagáljon).
* `none` (Kikapcsolja a megszakítás-figyelést).

## 3. A `poll()` rendszerhívás szerepe és előnyei (KULCSKONCEPCIÓ)

Ha az `edge` fájlt beállítottuk, a programunknak "el kell aludnia" (blokkolódnia), amíg a hardver nem jelez. Erre való a `poll()` (vagy a régebbi `select()`) rendszerhívás. Ezt a ZH-n gyakran hasonlítják a többi megoldáshoz.

* **Miért a `poll()` és nem a `select()`?** A `select()` egy nagyon régi, archaikus felület, nehézkes a használata. A `poll()` sokkal elegánsabb, memóriahatékonyabb, és mivel része a POSIX szabványnak, minden UNIX-szerű rendszeren hordozható (portábilis).
* **Mi a helyzet az `epoll()`-lal?** Az `epoll()` a létező leghatékonyabb megoldás hatalmas mennyiségű fájlleíró (pl. többezer hálózati kapcsolat) kezelésére, viszont **kizárólag Linux-specifikus**, nem POSIX szabvány, így BSD vagy MacOS rendszereken nem lefordítható.
* **Működése:** A `poll()` egy tömböt vár (`struct pollfd`), amiben felsoroljuk, mely fájlokat (perifériákat) akarjuk figyelni. A rendszerhívás blokkolja a programot (0% CPU), és csak akkor tér vissza, ha a felsorolt fájlok **bármelyikén** esemény történik, vagy letelik a paraméterként megadott időtúllépés (timeout). Ezzel egyetlen programszállal (thread) tudunk egyszerre akár tucatnyi szenzorra és hálózati portra figyelni, aktív várakozás nélkül.

## 4. A `poll()` technikai buktatói GPIO esetén (KULCSKONCEPCIÓ)

Ezt a gyakorlati trükköt szeretik kérdezni, mert ezen áll vagy bukik a kód működése!

Ha a `poll()`-t GPIO érték (`value` fájl) megszakítására használjuk a `sysfs`-ben, két nagyon fontos szabályt kell betartani a C kódban:

1. **A várt esemény típusa:** Normál esetben fájlolvasásnál a `POLLIN` eseményre várunk (van bejövő adat). **GPIO élvezérelt megszakítás esetén viszont a `POLLPRI` (Kiemelt prioritású adat/esemény) és a `POLLERR` (Hiba) eseményekre kell figyelni!**. Ha csak `POLLIN`-t kérsz, a `poll()` nem fog felébredni a gombnyomásra!
2. **A "Dummy Read" (A megszakítási flag törlése):** Amikor a `poll()` felébred egy gombnyomásra, a kernelben lévő megszakítási jelzőbit (flag) aktív marad. Ha ilyenkor újra meghívod a `poll()`-t, az **azonnal visszatér**, azt higyve, hogy egy újabb gombnyomás történt (végtelen ciklusba kerülsz).
* *A megoldás:* A `poll()` visszatérése után a fájlmutatót vissza kell tekerni az elejére a `lseek(fd, 0, SEEK_SET)` hívással, majd kötelezően végre kell hajtani egy olvasást a `read()`-del (ezt nevezik dummy read-nek). Ez az olvasás értesíti a kernelt, hogy lekezelted a megszakítást, és törli a flaget, így a következő `poll()` újra szépen, blokkolva fog várakozni a következő gombnyomásra.



---

Ez volt a 7. labor esszenciája! Mehetünk tovább a 8. laborral (vagy ami a listádban következik)?


Igazad van, ne haragudj a felületesebb áttekintésért! A ZH-kban (különösen a laboros tesztkérdéseknél) nagyon szeretnek ezekre a terminális parancsokra és a chipek belső állapotára rákérdezni.

Pótoljuk a hiányosságot, és ássunk le mélyebbre a **GPIO PIN használat, a `gpioinfo` és a `show-pins**` rejtelmeibe a 6. labor anyaga alapján!

---

# Kiegészítés: GPIO PIN-ek részletes kezelése és diagnosztikája

## 1. AP PIN-ek és a hardveres multiplexelés (Pinmux)

A processzoron (AP) található fizikai lábaknak (PIN-eknek) több szerepe is lehet, mivel egyetlen lábhoz több belső hardveres egység is csatlakozhat.

* **8 Funkció:** Egyetlen PIN-nek maximum 8 különböző funkciója lehet (0-tól 7-ig indexelve). Ezek lehetnek például: A/D konverter bemenet, memória busz, I2C, SPI, vagy a PRU (Programmable Real-Time Unit) funkciói.
* **Alapértelmezett állapot (Reset után):** Amikor a processzor újraindul, a lábak alapértelmezett funkciója tipikusan a **7-es funkció**, ami az általános célú GPIO.
* **Hardveres védelem:** Reset után a lábak biztonsági okokból **magas impedanciájú bemenetként (input)** konfigurálódnak. Hogy a rajtuk lévő feszültség ne "lebegjen", többnyire egy belső felhúzó (pull-up) vagy lehúzó (pull-down) ellenállást is bekapcsol a hardver.

## 2. A GPIO szoftveres archiktetúrái Linux alatt

A Linux kétféle architektúrát kínál a GPIO-k elérésére, mindkettő alapja a kernel `gpiolib` modulja.

* **A klasszikus (elavult) `sysfs`:** Ez a `/sys/class/gpio` könyvtáron keresztül, egyszerű szöveges fájlokba (`direction`, `value`, `edge`) írással/olvasással működik. *Hátránya:* Nagyon problémás, ha több program egyszerre akarja használni a GPIO-kat (erőforrás-megosztási problémák).
* **A modern `chardev`:** Ez a 4.8-as kerneltől a támogatott, modern megoldás. Itt a GPIO-k karakteres eszközként (character device) jelennek meg. Fájlok írogatása helyett csak programozottan (C kódból, speciális `ioctl` hívásokkal) érhető el. A BeagleBone mindkét módszert támogatja.

## 3. PIN konfiguráció lekérdezése: `gpioinfo`

A `gpioinfo` a modern (`chardev` alapú), generikus Linuxos segédprogram a GPIO-k állapotának lekérdezésére.

**Mit mutat meg a kimenete és mik a buktatók?**

* **Chip szintű bontás:** A kimenet `gpiochip`-enként van listázva. A BeagleBone-ban összesen 4 darab, egyenként 32 bites chip van (gpiochip0 ... gpiochip3).
* **Az ébresztés (Wake-up) trükkje (KULCSKÉRDÉS):** Az első chip (`gpiochip0`) hardveresen mindig kap tápfeszültséget (Always-on). A többi chip (1-től 3-ig) azonban csak "power on" módban aktív! **Ha a rendszer alvó (sleep) állapotban van, és egy gombnyomásra fel akarjuk ébreszteni, azt a gombot kötelező a `gpiochip0` valamelyik lábára kötni**, különben a rendszer nem fog felébredni!.
* **Információk soronként:** Megmutatja a fizikai kivezetés nevét (pl. `P9_22`), az eredeti alapfunkcióját (pl. `[spi0_sclk]`), azt, hogy aktuálisan használatban van-e a szoftver által (`[used]` vagy `unused`), az irányt (`input` vagy `output`), és az aktív szintet (pl. `active-high`).

## 4. PIN konfiguráció lekérdezése: `show-pins`

Míg a `gpioinfo` általános Linux parancs, a `show-pins` egy kifejezetten **BeagleBone (Texas Instruments) specifikus** eszköz.

**Tulajdonságai és kimenete:**

* **Használata terminálban:** A program színes kimenetet generál (escape szekvenciákat használ). Ha lapozni akarunk a kimenetben, a sima `less` parancs eltöri a színezést. **Helyette a `show-pins | less -R` (Repaint screen) vagy a `show-pins | more` parancsot kell használni!** Ezt vizsgán is kérdezhetik!.
* **Mit mutat meg?** Ez a parancs közvetlenül a hardver (SoC) Pinmux regisztereinek állapotát dekódolja. Megmutatja:
* A CAPE csatlakozó számát (pl. `P8.22`).
* Az aktuálisan kiválasztott multiplexelt belső funkciót (pl. `eMMC d5`).
* A sebességet (`fast` vagy `slow` slew rate).
* Az RX (vétel) állapotát (engedélyezve van-e a bemenet: `rx`).
* A beépített ellenállás állapotát (`up` = pull-up, `down` = pull-down).



---

Remélem, ez a részletes kiegészítés pontosan lefedi azokat a "trükkös" technikai részleteket, amikre az oktatók a ZH-n vadásznak!

Készen állok, **jöhet a 8. labor anyaga**, és azt is maximális alapossággal dolgozom fel!