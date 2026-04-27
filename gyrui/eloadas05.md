Nagyszerű, folytassuk a **5. prezentációval (6. hét előadása)**! Ebben a részben már belépünk a szoftver és a hardver határvidékére, egészen pontosan abba, hogy egy komplex operációs rendszer (mint a Linux) hogyan kezeli a beágyazott perifériákat, és mi, programozók hogyan tudjuk ezeket elérni.

Íme a GitHubra szánt, vizsgafókuszú összefoglaló a kulcsszavak alapján:

---

# Perifériák elérése szoftverből (5. prezentáció)

## 1. API és ABI, Rendszerhívás (KULCSKONCEPCIÓ)
Ahhoz, hogy megértsük, hogyan beszélget a szoftver a hardverrel, tisztázni kell a szinteket. Amikor a felhasználói térből (user-space) hardvert akarunk elérni, át kell lépnünk a kernel térbe. Ezt a **rendszerhívás (system call)** teszi lehetővé.

* **API (Application Programming Interface):** Ez a programozóknak szól. Egy magas szintű C függvény (pl. `open()`, `read()`, `printf()`), amit a forráskódban meghívsz. A fordító (compiler) ez alapján tudja, mit szeretnél csinálni. Ezt a szabványos C könyvtár (libc, glibc) biztosítja.
* **ABI (Application Binary Interface):** Ez már a gépi szint, a "fémközeli" kommunikáció. Az ABI határozza meg, hogy bináris (lefordított) szinten hogyan történik a rendszerhívás: melyik CPU regiszterbe kell tenni a hívás sorszámát, melyikbe a paramétereket, és milyen assembly utasítással (pl. `syscall` vagy szoftveres megszakítás) kell átadni a vezérlést a kernelnek.
* **Összehasonlítás:** Az API forráskód szintű kompatibilitást biztosít (ha újrafordítod a programot egy másik gépen, működni fog). Az ABI bináris szintű kompatibilitást jelent (a lefordított `.exe` vagy ELF fájl futni fog-e az adott operációs rendszeren és processzoron).

---

## 2. Linux Kernel interfész és I/O műveletek tipikus működése
A UNIX/Linux rendszerek egyik legfőbb alapelve: **Minden egy fájl!** A hardveres perifériák is fájlként (Device File) jelennek meg a `/dev` könyvtárban.

* **Tipikus I/O működés lépései:**
    1. `open()`: Megnyitjuk a perifériát (fájlt).
    2. `read()` / `write()`: Adatot olvasunk vagy írunk (bájtsorozatként).
    3. `close()`: Lezárjuk az eszközt.
* **Az IOCTL (Input/Output Control) (KULCSKONCEPCIÓ):** Nem minden művelet írható le egyszerű olvasásként vagy írásként. (Pl. hogyan állítod be az UART sebességét (baud rate), vagy hogyan adsz ki egy reset parancsot a szenzornak?). Erre való az `ioctl()` rendszerhívás. Ezzel speciális, hardver-specifikus parancsokat lehet leküldeni a kernel drivernek.

---

## 3. Kernel vs. User-space driverek (KULCSKONCEPCIÓ)
Ki írja a drivert, és hol fusson? Két lehetőségünk van Linux alatt:

| Szempont | Kernel-mode driver | User-space driver |
| :--- | :--- | :--- |
| **Hol fut?** | A kernel memóriájában, a legmagasabb jogosultsággal. | Felhasználói térben, korlátozott jogosultsággal. |
| **Sebesség** | Nagyon gyors, azonnali HW hozzáférés, nincs kontextusváltás. | Lassabb, mert minden HW kéréshez rendszerhívás kell. |
| **Megszakítások (IRQ)** | Közvetlenül tudja kezelni a hardveres megszakításokat. | Nem tud közvetlenül megszakítást kezelni (csak polling vagy blokkoló várakozás fájlon). |
| **Biztonság és Stabilitás**| **Kritikus!** Egy hiba (pl. null pointer) a driverben azonnal kékhalált / Kernel Panic-ot okoz, az egész gép leáll. | **Biztonságos.** Ha a driver kifagy, csak az az egy processz hal meg, az OS fut tovább. |
| **Fejlesztés nehézsége** | Bonyolult (speciális kernel API-kat kell ismerni, GPL licenc kérdések). | Egyszerű (sima C/C++/Python program, szabványos API-k). |

---

## 4. Beágyazott perifériák életciklusa
Egy beágyazott periféria nem "csak úgy" létezik a szoftverben, meghatározott életciklusa van:
1. **Probe/Init:** A kernel felismeri a hardvert (vagy a Device Tree-ből kiolvassa), és betölti hozzá a megfelelő drivert.
2. **Open/Configure:** A felhasználói program megnyitja és beállítja a paramétereket.
3. **Működés (Read/Write/Ioctl):** A tényleges használat.
4. **Close/Release:** A program elengedi az eszközt.
5. **Remove:** Ha a modult eltávolítják (pl. USB kihúzása), a driver lekapcsolja a hardvert és felszabadítja a lefoglalt memóriát.

---

## 5. GPIO Linux alatt: Sysfs vs. Chardev (KULCSKONCEPCIÓ)
A GPIO (Általános I/O láb) a leggyakoribb periféria. Tipikus funkciói: digitális ki/bemenet, és hardveres megszakítás (interrupt) generálása.

Linux alatt kétféle SW architektúra van az elérésükre. **Ezt a vizsgán szinte biztosan kérdezik!**

### A régi módszer: `sysfs` (Sysfs GPIO)
* **Lényege:** A `/sys/class/gpio` mappában szöveges fájlokba írunk és olvasunk. (Pl. `echo 1 > value` a bekapcsoláshoz).
* **Előnye:** Végtelenül egyszerű, akár shell scriptből is használható.
* **Hátránya:** Elavult (deprecated). **Lassú**, mert a számokat karakterlánccá (string) kell konvertálni. Nem lehet biztonságosan, egyszerre több lábat olvasni/írni (nincs atomi művelet).

### Az új szabvány: `chardev` (Character Device)
* **Lényege:** Megjelentek a `/dev/gpiochipX` karakteres eszközfájlok. Itt nem szöveget írogatunk, hanem `ioctl()` rendszerhívásokkal állítjuk a lábakat.
* **Előnye:** **Gyors** (nincs string konverzió). **Atomi műveleteket** tesz lehetővé (egyszerre be tudsz állítani 8 lábat, és senki nem tud közbeszólni). Erre épül a modern `libgpiod` könyvtár.
* **Hátránya:** Bonyolultabb a C kód megírása, nem lehet egy sima bash `echo` paranccsal vezérelni.

---

## 6. Linux I2C alrendszer és a libusb
Hogyan kezelünk bonyolultabb buszokat felhasználói térből?

### I2C periféria kezelés módjai (KULCSKONCEPCIÓ)
* **1. Kernel-space mód:** Van egy I2C eszközöd (pl. egy Hőmérő). Írsz hozzá egy kernel drivert. A kernel I2C alrendszere lekezeli a kommunikációt, a driver pedig létrehoz egy szabványos fájlt (pl. `/dev/hwmon0`). A felhasználó csak olvassa a hőmérsékletet, az I2C protokoll rejtve marad.
* **2. User-space mód (`i2c-dev`):** Ha nem akarsz kernel drivert írni, megnyitod a `/dev/i2c-1` eszközt. Ekkor a felhasználói programnak (C/Python) kell kézzel összeraknia az I2C bájtokat (cím, regiszter, adat), és `ioctl()` hívásokkal küldi le a kernelnek, ami szimplán csak "kiküldi" a buszra a nyers adatot.

### A `libusb` alapvető tulajdonságai
* Ha egy egyedi USB eszközt (pl. egy saját fejlesztésű FPGA kártyát) kötsz a Linuxra, normál esetben kernel drivert kéne írnod hozzá.
* A **libusb** egy user-space (felhasználói szintű) C könyvtár, ami megkerüli ezt. Lehetővé teszi, hogy közvetlenül a felhasználói programból kommunikálj az USB eszközzel (Bulk, Interrupt, Control transzferek) kernel programozás nélkül. Gyors fejlesztéshez és zárt forráskódú szoftverekhez (ahol nem akarod a kernel GPL licencét átvenni) tökéletes.

---

## 7. TUN és TAP alapötlete
Ezek virtuális hálózati interfészek a Linuxban. Hardveresen nem léteznek, a kernel szoftveresen hozza létre őket.

* **Alapötlet:** Amikor egy program egy TUN/TAP interfészre küld adatot, az nem egy fizikai kábelyen megy ki, hanem **egy másik felhasználói program (user-space app) kapja meg**. És fordítva is: a user-space app tud bejövő forgalmat generálni a rendszer számára.
* **TAP (Terminal Access Point):** Layer 2 szintű (Ethernet kereteket, MAC címeket kezel). Switch-elésre, virtualizációhoz (pl. virtuális gépek hálózatba kötése) használják.
* **TUN (Network TUNnel):** Layer 3 szintű (IP csomagokat kezel). VPN-ek (pl. OpenVPN) építésére használják. A VPN kliens megkapja az IP csomagot a TUN interfészről, titkosítja, majd kiküldi a valós internetre.

---
Ezzel az 5. prezentációt is lefedtük. Ha feldolgoztad, jelezd, és nézhetjük is a következőt!