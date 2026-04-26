Ez a jegyzet a **VIMIAD04_s1e5_emswper_2026.pdf** dokumentum alapján készült, a **kulcsszavak_ZH1.pdf**-ben kijelölt **5. prezentáció (6. hét)** témaköreit követve.

---

## 5. prezentáció (6. hét): Perifériakezelés szoftverarchitektúrája

### **Rendszerhívás, API és ABI összehasonlítása**
**3-4. oldal**
* **Rendszerhívás (System Call)**: Az a mechanizmus, amellyel egy felhasználói program szolgáltatást kér az operációs rendszer magjától (kerneltől). Ez egy kapu a védett kernel módba.
* **API (Application Programming Interface)**: Forráskód szintű interfész (pl. C függvények). Meghatározza, hogyan hívjuk meg a szolgáltatásokat a kódban (pl. `open()`, `read()`).
* **ABI (Application Binary Interface)**: Bináris szintű interfész. Meghatározza, hogyan történik az adatátadás gépi szinten: melyik regiszterbe kerülnek a paraméterek, hogyan néz ki a verem, és mi a rendszerhívás száma. Az ABI biztosítja, hogy a lefordított binárisok fussanak az adott OS/architektúra kombináción.

### **I/O műveletek tipikus működése**
**5. oldal**
* A legtöbb periféria elérése Linux alatt a **VFS (Virtual File System)** rétegen keresztül történik, egységes fájlműveletekkel:
    * **open()**: Kapcsolat létrehozása az eszközzel.
    * **read() / write()**: Adatok olvasása az eszközből vagy küldése az eszközre.
    * **close()**: Erőforrások felszabadítása.
    * **ioctl()**: Eszközspecifikus vezérlőműveletek, amelyek nem illeszthetők a tiszta adatfolyam modellbe.

### **Linux kernel interfész, device file és IOCTL**
**6, 8-9. oldal**
* **Device file**: A perifériák a `/dev` könyvtárban speciális fájlokként jelennek meg (pl. `/dev/ttyS0`, `/dev/i2c-1`).
* **Karakteres eszközök**: Adatfolyam alapúak, bájtonként érhetők el (pl. soros port).
* **Blokkos eszközök**: Adatblokkokban (pl. 512 bájt) kezelik az adatot, véletlen elérést tesznek lehetővé (pl. SD kártya).
* **IOCTL (Input/Output Control)**: Olyan rendszerhívás, amellyel a fájlmodellbe nem férő paramétereket állíthatjuk (pl. soros port baud rate, I2C slave cím beállítása).

### **Kernel és user-mode driverek összehasonlítása**
**10. oldal**
* **Kernel-mode driver**:
    * **Előny**: Közvetlen hardverelérés, alacsony késleltetés (latency), megszakítások kezelése.
    * **Hátrány**: A driver hibája az egész rendszert lefagyaszthatja (Kernel Panic), nehezebb fejleszteni.
* **User-mode driver**:
    * **Előny**: Biztonságos (csak a program omlik össze), könnyű debuggolni standard eszközökkel (gdb), könyvtárak használata (pl. libusb).
    * **Hátrány**: Lassabb a context switch és az adatmásolás miatt, korlátozott megszakításkezelés.

### **Beágyazott perifériák életciklusa**
**11. oldal**
1.  **Registration**: A driver regisztrálása a kernelnél (tudatjuk a rendszerszoftverrel, hogy létezik ilyen típusú eszköz).
2.  **Probe**: Az adott hardverpéldány megkeresése és inicializálása.
3.  **Operation (Running)**: Aktív használat, adatátvitel.
4.  **Removal / Unregister**: Az eszköz leválasztása vagy a driver eltávolítása, erőforrások lezárása.



### **GPIO funkciók és SW architektúra Linux-ban**
**12-14. oldal**
* **Funkciók**: Bemenet (olvasás), kimenet (írás), megszakítás generálás (pl. gombnyomásra).
* **Architektúra**: A hardver felett egy **GPIO Chip Driver** áll, efelett a **gpiolib** kernel alrendszer, ami egységesíti a különböző chipgyártók megoldásait. Ez teszi elérhetővé a funkciókat a felhasználói tér felé.

### **A sysfs és chardev GPIO elérés összehasonlítása**
**15-16. oldal**
* **sysfs (Legacy)**:
    * `/sys/class/gpio` alatt érhető el.
    * Szöveges alapú (fájlokba `0` vagy `1` írása).
    * Könnyen használható scriptből, de lassú és elavult (deprecated).
* **chardev (Modern - gpiod)**:
    * `/dev/gpiochipN` karakteres eszközön keresztül.
    * Hatékonyabb, támogatja a lábak csoportos kezelését és a pontosabb eseménykezelést (interrupts).
    * A `libgpiod` könyvtár használata javasolt hozzá.

### **Linux I2C alrendszer és kezelési módok**
**17-18. oldal**
* **Kernel-space**: A periféria saját driverrel rendelkezik a kernelben (pl. egy hőmérő szenzor), ami kinyerhető adatokat szolgáltat más alrendszereknek.
* **User-space (i2c-dev)**: A `/dev/i2c-N` eszközön keresztül a felhasználói program maga állítja össze az I2C üzeneteket. Ez rugalmas, nem kell hozzá új kernelmodult írni minden új szenzorhoz.

### **libusb alapvető tulajdonságai**
**19. oldal**
* Lehetővé teszi az USB eszközök kezelését közvetlenül a **felhasználói térből** (user-space).
* **Javasolt**: Ha nem standard osztályú eszközről van szó (pl. nem egér vagy háttértár), vagy ha hordozható alkalmazást (Linux, Windows, macOS) akarunk írni.

### **TUN és TAP alapötlete**
**20. oldal**
* **Virtuális hálózati interfészek**, amelyek nem valódi hardverhez, hanem egy felhasználói programhoz kapcsolódnak.
* **TUN (Network Tunnel)**: IP csomagok (Layer 3) kezelésére.
* **TAP (Network Tap)**: Ethernet keretek (Layer 2) kezelésére.
* **Alkalmazás**: VPN szoftverek, emulátorok (pl. QEMU hálózat).

---
Ez a jegyzet a 6. hét legfontosabb szoftveres interfészeit és perifériakezelési módjait dolgozza fel. Jöhet az utolsó rész, a 6. prezentáció?