Ez a jegyzet a **VIMIAD04_s1e2_hwsw_2026.pdf**  dokumentum alapján készült, a **kulcsszavak.pdf**-ben kijelölt **2. prezentáció (3. előadás)** témaköreit követve.

---

## 2. prezentáció (3. előadás): Beágyazott HW és SW architektúrák

### **Beágyazott rendszerek HW komponensei és interfészei**
**3. oldal** 
* A beágyazott rendszerek alapvető hardveres építőelemei a következők:
    * **Központi egység**: Mikrovezérlő (**MCU**) vagy rendszerchip (**SoC**).
    * **Memória**: Belső és külső **RAM** (műveleti memória) és **Flash** (program és adattároló).
    * **Perifériák és interfészek**:
        * **Analóg/Digitális**: GPIO, ADC, DAC, PWM.
        * **Kommunikációs**: I2C, SPI, UART, USB, Ethernet, CAN.
        * **Vezeték nélküli**: Bluetooth (BT/BLE), WiFi, LoRa, ZigBee, 4G/5G.
    * **Tápellátás**: Power Management IC (PMIC), akkumulátor töltésvezérlés.

### **Mikrovezérlő (MCU) és Rendszerchip (SoC) összehasonlítása**
**4. oldal** 
| Jellemző | Mikrovezérlő (**MCU**) | Rendszerchip (**SoC**) |
| :--- | :--- | :--- |
| **Integráció** | Magas szintű, minden egy lapkán (CPU, RAM, Flash, perifériák). | Extrém magas szintű, de gyakran igényel külső memóriát. |
| **CPU teljesítmény** | Alacsonyabb órajel, egyszerűbb felépítés. | Magas órajel, többmagos architektúra, komplex utasításkészlet. |
| **Operációs rendszer** | Bare-metal (nincs OS) vagy **RTOS** (pl. FreeRTOS). | Komplex OS (pl. **Linux**, Android). |
| **Erőforrások** | Kicsi (pár KB - MB RAM/Flash). | Jelentős (MB - GB nagyságrendű RAM). |
| **Fogyasztás** | Nagyon alacsony, gyakran elemes működésre optimalizált. | Magasabb, de méretezhető; komolyabb hőmenedzsmentet igényelhet. |



### **Védelmi szintek és az MMU szerepe a rendszerchipekben**
**11-13. oldal** 
* **Védelmi szintek (Protection Rings)**: A processzorok hardveres szinten különítik el a futó kódok jogosultságait.
    * **Ring 0 (Kernel mód)**: Teljes hozzáférés a hardverhez és a memóriához.
    * **Ring 3 (User mód)**: Korlátozott hozzáférés, a rendszer stabilitását védi az alkalmazások hibáitól.
* **MMU (Memory Management Unit)**:
    * A rendszerchipek elengedhetetlen komponense, amely a **virtuális memóriacímeket** fizikaira fordítja le.
    * **Feladata**: Memóriavédelem (egy folyamat nem írhat bele a másikéba), jogosultságkezelés (olvasható/írható/futtatható területek kijelölése).
    * Lehetővé teszi, hogy minden folyamat úgy "érezze", övé a teljes memóriatartomány.

### **Kernel és User-space szétválasztása és következményei**
**14-16. oldal** 
* A **Linux** szigorúan különválasztja a kernel (rendszermag) és a felhasználói (user) területet.
* **Következmények**:
    * **Stabilitás**: Egy felhasználói alkalmazás összeomlása nem dönti meg a teljes rendszert.
    * **Biztonság**: Az alkalmazások csak ellenőrzött csatornákon (**System Call / Rendszerhívás**) keresztül kérhetnek erőforrásokat.
    * **Teljesítmény**: A két tér közötti váltás (**Context Switch**) és az adatok másolása többletterhelést (overhead) jelent a processzornak.

### **Virtuális memória teljesítménye általában és beágyazott rendszerekben**
**17-18. oldal** 
* A virtuális memória használata **nem-determinisztikus** viselkedést okozhat.
* **Problémák**:
    * **TLB Miss**: Ha a címfordító gyorsítótárban nincs benne a cím, lassabb memóriaműveletre van szükség.
    * **Page Fault**: Ha az igényelt memóriaoldal nincs a fizikai RAM-ban, a háttértárról (pl. SD kártya) kell betölteni, ami milliszekundumos késést jelent.
* **Beágyazott megoldás**: Valós idejű igények esetén a memóriát "le kell rögzíteni" (**memory locking**, pl. `mlockall` hívással), hogy elkerüljük az oldalcserék miatti bizonytalan futásidőt.

### **SoC, PoP, SoP, SoM, SBC definíciója és megkülönböztetésük**
**5-10. oldal** 
* **SoC (System on Chip)**: Maga a szilícium lapka, amin a fő komponensek vannak (CPU, GPU, perifériák).
* **PoP (Package on Package)**: Tokozási technológia, ahol pl. a RAM chipet közvetlenül a SoC tetejére forrasztják a helytakarékosság és a gyors elérés érdekében.
* **SoP (System on Package)**: Több különböző lapka (pl. processzor + rádiófrekvenciás modul) egyetlen közös tokozáson belül.
* **SoM (System on Module)**: Egy kisméretű kártya, amely tartalmazza a SoC-t, a memóriákat és a tápellátást. Ezt egy hordozó kártyára (Baseboard) kell illeszteni.
* **SBC (Single Board Computer)**: Egyetlen nyomtatott áramkörön lévő teljes számítógép minden szükséges csatlakozóval (pl. Raspberry Pi, BeagleBone).

---
Ez a jegyzet a 3. előadás legfontosabb pontjait tartalmazza. Folytassuk a következő témakörrel?