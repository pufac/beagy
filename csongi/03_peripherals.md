Ez a jegyzet a **VIMIAD04_s1e3_peripherals_2026.pdf** dokumentum alapján készült, a **kulcsszavak.pdf**-ben kijelölt **3. prezentáció (4. előadás)** témaköreit követve.

---

## 3. prezentáció (4. előadás): Perifériák és interfészek

### **Periféria szerkezete, illesztés, regiszter fájl és funkció szerepe**
**3. oldal**
* **Periféria szerkezete**: A perifériák a processzor számára regiszterek halmazaként (**Register File**) jelennek meg.
* **Illesztés**: A periféria belső logikája végzi a tényleges feladatot (pl. számlálás, jelátalakítás), míg a regiszterek biztosítják az interfészt a szoftver felé.
* **Regiszterek szerepe**:
    * **Control (Vezérlő) regiszter**: A periféria működési módjának beállítása (pl. engedélyezés, sebesség választás).
    * **Status (Állapot) regiszter**: A periféria aktuális állapotának visszajelzése (pl. adat beérkezett, hiba történt, kész a küldésre).
    * **Data (Adat) regiszter**: Itt történik a tényleges adatcsere (írás/olvasás) a CPU és a periféria között.

### **Periféria címe, dedikált I/O leképzés és memóriára leképzett I/O**
**4-5. oldal**
* **Periféria címe**: Minden periféria regiszternek egyedi címe van a rendszer buszon, amelyet a processzor a címbuszon keresztül ér el.
* **Címdekódolás**: A periféria csak akkor válaszol, ha a címbuszon az ő tartományába eső cím jelenik meg.
* **Leképzési módok**:
    * **Dedikált I/O leképzés (Isolated I/O)**: A perifériák egy külön címtartományban vannak, elérésükhöz speciális processzorutasítások kellenek (pl. x86 `IN`, `OUT`).
    * **Memóriára leképzett I/O (Memory-Mapped I/O - MMIO)**: A perifériák a rendszermemória címtartományának egy részét foglalják el. Ugyanazok az utasítások használhatók (pl. `LOAD`, `STORE`), mint a memória eléréséhez.

### **Megszakítások (Interrupts)**
**6. oldal**
* **Célja**: Lehetővé teszi, hogy a periféria jelezze a processzornak, ha egy esemény történt (pl. adat érkezett), így a CPU-nak nem kell folyamatosan várakoznia/lekérdeznie (polling).
* **Működése**: Az esemény hatására a processzor felfüggeszti az aktuális feladatát, és végrehajt egy előre definiált megszakításkezelő függvényt (**Interrupt Service Routine - ISR**).

### **Regiszter fájl struktúrája**
**3. oldal**
* A regiszterek általában fix méretűek (pl. 8, 16, 32 bit), és egy báziscímhez képest meghatározott eltolással (**offset**) érhetők el.
* A struktúra logikusan választja szét a konfigurációs, állapotjelző és adatátviteli funkciókat, biztosítva a szoftveres vezérelhetőséget.

### **PIN konfiguráció funkciója**
**10. oldal**
* **Pin Muxing**: A modern rendszerchipeken (SoC) a fizikai kivezetések (lábak) száma korlátozott, ezért egy lábhoz több belső funkció is hozzárendelhető (pl. egy láb lehet GPIO, de UART kimenet is).
* **Konfiguráció**: Szoftverből kell beállítani, hogy az adott fizikai láb éppen melyik belső perifériához kapcsolódjon.
* **I/O tulajdonságok**: A konfiguráció része lehet a felhúzó/lehúzó ellenállások (**Pull-up/Pull-down**) beállítása és a meghajtó képesség (áramerősség) szabályozása is.

### **GPIO, A/D és D/A alapok**
**11-15. oldal**
* **GPIO alapszabályok**:
    * **Kimenet**: Ügyelni kell a maximális terhelhetőségre (áram), hogy ne károsítsuk a chipet.
    * **Bemenet**: A logikai szinteknek (Vih, Vil) meg kell felelniük a specifikációnak; lebegő bemenet esetén pull-up/down szükséges.
* **A/D (Analóg-Digitális) átalakítás**:
    * **Kvantálás**: A folytonos analóg jel felbontása véges számú digitális lépcsőre.
    * **Mintavételi frekvencia**: Meghatározza, milyen gyakran veszünk mintát a jelből (Nyquist-elv).
    * **Mintavételi módok**: **On-demand** (szoftver kéri a mérést) vagy **időzített** (hardveres timer indítja periodikusan).
* **D/A (Digitális-Analóg) átalakítás**: Digitális értékből folytonos feszültségszintet állít elő. Jellemzői a felbontás (bit) és a beállási idő.

### **Digitális kommunikáció (I2C, SPI, UART)**
**18-20. oldal**
* **UART**: Aszinkron, pont-pont kapcsolat, két vezeték (TX, RX). Nincs közös órajel, a sebességet (baud rate) előre meg kell egyezni.
* **SPI**: Szinkron, master-slave architektúra, legalább 4 vezeték (SCLK, MOSI, MISO, CS). Nagyon gyors, full-duplex átvitelre képes.
* **I2C**: Szinkron, busz rendszer (több eszköz), két vezeték (SDA, SCL). Lassabb, de rugalmasabb, címzés alapú eszközkezelés.

### **Adatelérési eltérések: I2C/SPI vs. UART**
**19-20. oldal**
* **I2C és SPI**: Ezek a protokollok általában **regiszter fájl alapú** elérést valósítanak meg a periférián belül. Az elérhető adatok szerkezete kötött, címzéssel közvetlenül hozzáférhetünk egy adott regiszterhez (pl. egy szenzor hőmérséklet regiszteréhez)[cite: 19, 20].
* **UART**: Ez egy tisztán **kommunikációs csatorna**, nincsenek benne alapértelmezett hardveres regiszterek a távoli oldalon. Bármilyen adatot, bármilyen formátumban küldhetünk rajta; a feleknek szoftveres protokollban (pl. AT parancsok) kell megegyezniük az adatok értelmezéséhez.

---
Ez a jegyzet a 4. előadás kulcspontjait fedi le. Jöhet a következő rész?