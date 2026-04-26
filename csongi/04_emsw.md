Ez a jegyzet a **VIMIAD04_s1e4_emsw_2026.pdf** dokumentum alapján készült, a **kulcsszavak_ZH1.pdf**-ben kijelölt **4. prezentáció (5. előadás)** témaköreit követve.

---

## 4. prezentáció (5. előadás): Beágyazott szoftverrendszerek alapjai

### **Beágyazott rendszerek SW fejlesztési problémáinak fő okai**
**2. oldal**
* **Erőforrás korlátok**: A memória (RAM/Flash) és a számítási teljesítmény (CPU órajel) gyakran szűkös.
* **Közvetlen hardverelérés**: A szoftvernek gyakran közvetlenül kell kezelnie a regisztereket és a megszakításokat.
* **Valós idejűség (Real-time)**: Időbeli kényszerek betartása (határidők, determinisztikus futás).
* **Platformfüggőség**: A kód erősen kötődik az adott hardverarchitektúrához, ami nehezíti a hordozhatóságot.
* **Hibakeresési nehézségek**: Korlátozott láthatóság a rendszer belsejébe (szükség lehet JTAG, SWD interfészekre).

### **Beágyazott szoftver (firmware) és a "klasszikus SW" eltérése**
**4-5. oldal**
* **Életciklus**: A firmware ritkábban frissül, és gyakran a hardver teljes élettartama alatt fut.
* **Interfész**: Nincs vagy minimális a felhasználói felület (GUI helyett LED-ek, soros port).
* **Megbízhatóság**: Magasabb elvárások (pl. Watchdog timer használata a lefagyások ellen).
* **Végrehajtás**: A firmware gyakran közvetlenül a Flash memóriából fut (**Execute in Place - XIP**), míg a klasszikus szoftver a RAM-ba töltődik.

### **BSP (Board Support Package) definíciója és tulajdonságai**
**6. oldal**
* **Definíció**: A hardver és az operációs rendszer (vagy alkalmazás) közötti szoftverréteg, amely elfedi a hardverspecifikus részleteket.
* **Tartalma**:
    * **Bootloader**: A rendszer indításáért felelős kód.
    * **Eszközmeghajtók (Drivers)**: Perifériák kezelése (UART, I2C, SPI stb.).
    * **Konfigurációs fájlok**: Hardverparaméterek (órajel beállítások, memória térkép).
* **Célja**: Hordozhatóság biztosítása; ha a hardver változik, csak a BSP-t kell módosítani, az alkalmazást nem.

### **Operációs rendszerek összehasonlítása**
**7-8. oldal**
| Típus | Leírás | Előny/Hátrány |
| :--- | :--- | :--- |
| **NOOS (No OS)** | "Bare-metal" programozás, nincs szoftveres absztrakciós réteg. | Maximális kontroll, minimális overhead, de nehéz fejleszteni. |
| **Rejtett OS** | Olyan keretrendszer, ami elrejti az ütemezést (pl. Arduino), de nem teljes értékű OS. | Könnyű használat, de korlátozott rugalmasság. |
| **Beágyazott OS** | Kifejezetten szűkös erőforrásokra tervezett OS (pl. RTOS). | Többszálúság, erőforrás-kezelés, determinisztikus időzítés. |

### **Körforgó programszervezés (Round-robin)**
**11. oldal**
* **Működése**: Egy végtelen `while(1)` ciklusban egymás után hívódnak meg a feladatok.
* **Tulajdonságai**:
    * Nagyon egyszerű implementálni.
    * Nincs prioritáskezelés.
    * Egy hosszú feladat blokkolja az összes többit, ami rontja a válaszidőt.

### **Körforgó programszervezés megszakítással**
**12. oldal**
* **Működése**: Az alap körforgót kiegészítjük megszakításkezelőkkel (ISR). Az ISR-ek kezelik a sürgős eseményeket, a főciklus pedig a háttérfeladatokat.
* **Tulajdonságai**:
    * A kritikus eseményekre gyorsabb a válasz.
    * Az ISR-nek rövidnek kell lennie, csak jelzést (flag) szabad beállítania a főciklusnak.

### **Függvénysor alapú ütemezés (Function Queue Scheduling)**
**13. oldal**
* **Működése**: A feladatok (függvénymutatók) egy sorba kerülnek. A főciklus ebből a sorból veszi ki és hajtja végre őket.
* **Tulajdonságai**:
    * Dinamikusabb, mint a sima körforgó.
    * A megszakítások is ütemezhetnek feladatokat a sorba.
    * Lehetővé tesz egyfajta prioritáskezelést a sorrend módosításával.

---
Ez a jegyzet a 5. előadás szoftveres alapjait foglalja össze. Folytassuk a 6. előadással?