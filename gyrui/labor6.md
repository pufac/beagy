Íme a 6. labor zanzásított, kifejezetten a vizsgakérdésekre (kulcsszavakra) fókuszáló összefoglalója! Ez a labor nagyrészt gyakorlati, de két nagyon fontos elméleti kérdést is tartalmaz a ZH-hoz.

---

# 6. Labor: GPIO használata és a Sysfs interfész

## 1. AP PIN-ek és a funkciók multiplexelése
A BeagleBone processzorán (AP) a lábaknak (PIN-eknek) több funkciója is lehet.
* [cite_start]Egy fizikai láb maximum 8 különböző belső funkcióhoz (0-7) rendelhető hozzá[cite: 1367].
* [cite_start]Reset (újraindulás) után az alapértelmezett beállítás jellemzően a 7-es funkció, ami a sima GPIO (általános célú ki/bemenet)[cite: 1373].
* [cite_start]Rendszerinduláskor ezek a lábak többnyire magas impedanciájú bemenetként (input) funkcionálnak, gyakran egy felhúzó (pull-up) vagy lehúzó (pull-down) ellenállással, hogy a jelszint ne lebegjen[cite: 1373, 1374].
* [cite_start]A BeagleBone panelen a fizikai kivezetéseket a kártya szélén lévő csatlakozók alapján (CAPE ID) `P8_x` vagy `P9_x` formátumban azonosítják[cite: 1525].

## 2. GPIO elérés sysfs-en keresztül (KULCSKONCEPCIÓ)
Ahogy az elméletben is volt, a Linux a hardvert fájlokként kezeli. A régi (klasszikus) megoldás a GPIO lábak kezelésére a `sysfs` virtuális fájlrendszer.

A használat lépései és a kimenet konfigurálása paranccsorból (vagy C-ből fájlműveletekkel):
1. [cite_start]**Export (Aktiválás):** Ahhoz, hogy egy lábat a felhasználói térből (user-space) elérjünk, a láb számát be kell írni az `export` fájlba[cite: 1592]. [cite_start]Ez létrehoz egy dedikált könyvtárat a lábnak (pl. `/sys/class/gpio/gpioX`)[cite: 1595]. [cite_start](A visszaadás az `unexport` fájlon keresztül történik [cite: 1593]).
2. **Direction (Irány beállítása):** A létrehozott könyvtáron belül a `direction` fájlba írással határozzuk meg az irányt. [cite_start]Ha kimenetet akarunk, az `"out"` szót, ha bemenetet, az `"in"` szót kell beleírni[cite: 1611, 1612].
3. [cite_start]**Value (Érték beállítása vagy olvasása):** A `value` nevű fájlon keresztül zajlik az érdemi munka[cite: 1620]. Ha a láb kimenet, akkor egy `"1"`-es vagy `"0"`-s karakter beírásával magas (High) vagy alacsony (Low) feszültségszintre húzzuk a lábat. Ha bemenet, akkor ugyanebből a fájlból olvashatjuk ki a hardver aktuális állapotát.

[cite_start]*Megjegyzés: Itt lehet beállítani a hardveres megszakítást szimuláló eseményeket is az `edge` fájl segítségével (rising, falling, both)[cite: 1614, 1615].*

## 3. Az `fflush()` szerepe C nyelvű programozásnál (KULCSKONCEPCIÓ)
Ha a sysfs alapú GPIO kezelést egy C programból csináljuk (pl. egy LED-et akarunk másodpercenként villogtatni), előfordulhat egy tipikus hiba: beírjuk az `"1"`-est a `value` fájlba (`fprintf` segítségével), de a LED mégsem gyullad fel.

**Miért van ez, és mi az `fflush()` szerepe?**
* [cite_start]**A probléma:** A fájlműveletek (mint az `fprintf`) a C szabványkönyvtárban és az operációs rendszerben **pufferelve (buffered)** vannak[cite: 1637]. A rendszer nem küldi ki azonnal minden egyes karakter írásánál az adatot a hardvernek, hanem vár, amíg a memóriapuffer megtelik, hogy hatékonyabb legyen az $I/O$ művelet. 
* [cite_start]**A megoldás (`fflush`):** Mivel mi azt akarjuk, hogy a LED azonnal felgyulladjon, az írás (`fprintf`) után **ki kell adni az `fflush(fd)` parancsot**[cite: 1637]. [cite_start]Ez arra kényszeríti az operációs rendszert, hogy a puffer tartalmát azonnal ürítse (flush) bele a fájlba, ami azonnal átbillenti a GPIO láb fizikai állapotát[cite: 1637].

## 4. Diagnosztikai eszközök (Röviden)
Ha a fejlesztés során nem tudjuk, hogy egy láb éppen mit csinál, két parancs is a segítségünkre lehet:
* [cite_start]**`gpioinfo`:** A modern (chardev) generikus lekérdező programja, ami megmutatja az összes GPIO chip és láb állapotát (foglalt-e, kimenet/bemenet)[cite: 1671].
* [cite_start]**`show-pins`:** Ez egy BeagleBone specifikus (TI) segédprogram, ami részletes, de kicsit nyersebb formában kilistázza a lábak multiplexelési beállításait[cite: 1672].

---
*Ezzel a végére is értünk a kiadott tananyagnak! A fókuszált koncepciók alapján (különösen a kernel/user-space különbségek, a Linux jogosultságok, az MMU vs MCU eltérések, a megszakítások vs poll, és a fájlalapú hardverkezelés) nagyon jó esélyekkel indulsz a ZH-n. Sok sikert a felkészüléshez!*