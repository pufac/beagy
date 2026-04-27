Nagyszerű, vesszük is a 4. heti labort! Ahogy a kulcsszavakból is látszik, ez a labor már hardverközelibb volt, hiszen bejött a képbe a BeagleBone, mint célhardver. 

A ZH szempontjából itt egy nagyon fontos, klasszikus Linuxos koncepcióra kérdeznek rá a hardver elérésével kapcsolatban. Íme a fókuszált összefoglaló!

---

# 4. Labor: BeagleBone és Periféria elérés (Jogosultságok)

## 1. Ismerkedés a BeagleBone-nal
A BeagleBone (Black/Green) egy népszerű SBC (Single Board Computer), ami az Alkalmazás Processzorok (AP) kategóriájába esik (TI AM335x SoC fut rajta).
* **Szerepe a fejlesztésben:** Ez a "célhardver" (Target). A programot a PC-den (Host) írod és fordítod (keresztfordítás), de ezen a kártyán fogod futtatni és tesztelni a valós hardveres I/O műveleteket.
* A panelen teljes értékű Linux (többnyire Debian) fut, így a PC-n megszokott Linuxos környezetben (fájlrendszer, parancssor) dolgozhatsz.

## 2. Soros hozzáférés konfigurálása
Amikor egy beágyazott eszközön nincs monitor és billentyűzet (Headless mód), valahogy kommunikálnunk kell vele. A legalapvetőbb és legbiztosabb módszer a **soros konzol (UART)**.

* **Fizikai kapcsolat:** A fejlesztői PC-t egy USB-Soros átalakítóval (vagy a panel beépített debug USB portján át) kötjük össze a BeagleBone-nal.
* **Szoftveres elérés:** A Linuxon ez a kapcsolat egy eszközfájlként jelenik meg (pl. `/dev/ttyUSB0` vagy `/dev/ttyACM0`).
* **Konfiguráció:** Ahhoz, hogy értelmezhető szöveget lássunk (ne csak szemetet), a terminál emulátor programban (pl. PuTTY, minicom, picocom) be kell állítani a soros port paramétereit. A legtipikusabb beállítás:
    * **Baud rate (sebesség):** `115200` bps
    * **Adatbitek:** `8`
    * **Paritás (Parity):** Nincs (`None`)
    * **Stop bit:** `1`
    *(Ezt szokás röviden `115200 8N1` formátumban emlegetni).*

## 3. Jogosultságok fontossága periféria elérés esetén (KULCSKONCEPCIÓ)
Ez a labor legfontosabb elméleti része, amit vizsgán is biztosan számonkérnek!

Ahogy korábban láttuk: **Linux alatt minden egy fájl.** A hardveres perifériák is (pl. a soros port `/dev/ttyUSB0`, vagy a GPIO pinek a `/sys/class/gpio` alatt). 
Ebből egyenesen következik a probléma: **A fájlokra szigorú Linuxos jogosultságkezelés (Read/Write/Execute) vonatkozik!**

**A probléma:**
Alapértelmezésben a hardvert reprezentáló fájlok tulajdonosa a `root` (rendszergazda). Egy sima, mezei felhasználó (user) szoftvere **nem kap írási/olvasási jogot** a perifériához. Ha a programod megpróbálja megnyitni a portot, "Permission denied" (Hozzáférés megtagadva) hibát kapsz.

**A lehetséges (és helyes) megoldások:**
1.  **Futtatás root-ként (`sudo`):** * Lusta megoldás. Ha a programot `sudo ./programom` paranccsal indítod, mindent szabad neki.
    * *Miért rossz?* Hatalmas biztonsági rés. Egy hibás (vagy rosszindulatú) program root joggal az egész operációs rendszert tönkreteheti.
2.  **Csoport jogosultságok használata (A helyes út 1.):** * A perifériák gyakran dedikált felhasználói csoportokhoz (group) tartoznak. Például a soros portok a `dialout` csoporthoz, az I2C buszok az `i2c` csoporthoz.
    * Megoldás: Hozzá kell adni a saját felhasználódat a megfelelő csoporthoz (pl. `usermod -a -G dialout usernevem`). Ezt követően a programod (a te nevedben futva) biztonságosan, root jogok nélkül is eléri a hardvert.
3.  **udev szabályok (udev rules) írása (A helyes út 2.):**
    * Ha egy teljesen egyedi hardvert (pl. egy USB-s mérőműszert) dugsz a gépbe, az operációs rendszer (a `udev` démon) hozza létre az eszközfájlt.
    * Lehet írni egy szabályt (rule-t), ami megmondja a Linuxnak: *"Ha ezt a konkrét USB eszközt bedugják, a hozzá tartozó fájl kapjon 666-os (mindenki által írható/olvasható) jogosultságot, vagy kerüljön a 'mérések' csoportba!"*.

---

Ez a rész rövidebb volt, de a **jogosultságkezelés perifériáknál** egy sarkalatos koncepció a beágyazott Linux fejlesztésben. Jöhet a következő anyag?