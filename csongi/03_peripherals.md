Örömmel folytatom a munkát! Szintén a kért oktatósegédi precizitással, a kiadott kép sorrendjét követve, a megfelelő diákhoz (oldalszámokhoz) rendelve kidolgozom a 4. előadás (3. prezentáció) kiemelt témaköreit.

***

# Beágyazott Rendszerek - 4. Előadás (Perifériák) Jegyzet

## 1. Periféria szerkezete, illesztés, regiszter file, és periféria funkció szerepe

**4. és 6. lap: A periféria definíciója és szerkezete**
* **Mi a periféria?** A számítógép-architektúrában minden olyan komponens perifériának számít, ami nem a végrehajtó egység (CPU mag) és nem is a hozzá direktben csatolt operatív memória[cite: 250, 251]. Ide tartozik például a háttértár (HDD, SSD), a hálózati interfész, de maguk a belső összeköttetés-hálózatok (buszok) is[cite: 255, 257].
* **A beágyazott periféria felépítése (A diasoron lévő ábra magyarázata):**
  A mikrokontrollerek (MCU) és SoC-k (System on a Chip) belsejében a perifériák jellemzően három fő logikai blokkból épülnek fel[cite: 264, 265, 266, 268, 290, 291]:
  1.  **Illesztés (Interface):** Ez a modul felel a számítógép (processzor és a belső buszrendszer) felé történő digitális csatlakozásért[cite: 264].
  2.  **Regiszter fájl (Register file):** A hardver és a szoftver közötti kommunikációs felület. A szoftver ezen regiszterek írásával és olvasásával vezérli a perifériát és kap tőle adatokat[cite: 265].
  3.  **Periféria Funkció:** Ez maga a specializált hardver (lehet digitális vagy analóg), amely a tényleges feladatot végzi (pl. időzítés, jelszint-átalakítás, kommunikációs protokoll megvalósítása), és amely kapcsolatot tarthat a külvilággal (Fizikai rendszer)[cite: 266, 268].

---

## 2. Periféria címe, dedikált IO leképzés és memóriára leképzett IO

**7. lap: Illesztés a végrehajtóegységekhez**
A CPU-nak valamilyen módon címeznie kell a perifériákat, hogy kommunikálni tudjon velük[cite: 303, 306]. Erre két alapvető architektúrális megoldás létezik:
* **Dedikált IO (Port-mapped IO):** A processzor speciális gépi utasításokkal (pl. x86-on az `IN` és `OUT` utasítások) éri el a perifériákat[cite: 308]. Ennek teljesen különálló címtere van a normál memóriától.
* **Memóriára leképzett IO (Memory Mapped IO):** Ebben az esetben a perifériák regiszterei úgy látszanak a processzor számára, mintha a normál fizikai memória részei lennének[cite: 309]. Ugyanazokkal a standard gépi utasításokkal (pl. `LDR`, `STR` ARM-on) olvashatók és írhatók, mint a RAM, csak egy specifikus, nekik fenntartott címtartományban (memory map) helyezkednek el[cite: 309, 310]. (Ez a leggyakoribb a modern beágyazott rendszerekben) [cite: 311]. 
**10-12. lap: Példa a memórialeképzésre (AM3358)**
* A dokumentáció (Adatlap/Reference Manual) egyik legfontosabb része a **Memory Map** táblázat, amely megadja, hogy mettől meddig (Start-End address) tart egy adott komponens[cite: 369, 370, 393].
* Például egy 32 bites rendszernél (4 GB címzési tér) a `0x4000_0000` és `0x5FFF_FFFF` közötti terület lehet fenntartva a beágyazott perifériák regisztereinek[cite: 379, 384].

---

## 3. Megszakítása (Interrupts)

**8. lap: A megszakítás fogalma és működése**
* **Fogalma:** A processzoron futó aktuális program végrehajtásának felfüggesztése (megszakítása) egy külső hardveres esemény hatására[cite: 328]. Pl. ha adat érkezett egy soros porton vagy megnyomtak egy gombot[cite: 330].
* **Működése:** A perifériából egy megszakításkérés (IRQ) érkezik a processzor megszakításvezérlőjébe[cite: 339]. * **Típusai:**
  * **Maszkolható megszakítás:** A szoftver által ideiglenesen letiltható (maszkolható)[cite: 344].
  * **Nem maszkolható megszakítás (NMI):** Olyan kritikus hibajelzés, amit nem lehet letiltani, a processzornak mindenképp reagálnia kell rá[cite: 335, 344].
* A modern rendszerek (pl. ARM Cortex-M) **vektoros megszakításvezérlőt** (NVIC - Nested Vector Interrupt Controller) használnak, ami intelligensen kezeli a prioritásokat és az egymásba ágyazott megszakításokat[cite: 345].

---

## 4. Regiszter file struktúrája

**13. lap: A regiszterek típusai**
A perifériához tartozó memóriaterület jellemzően gépi szó méretű (pl. 32 bites) regiszterekre van felosztva[cite: 399]. Ezek struktúrája funkció szerint csoportosítható:
1.  **Adatregiszterek (Data):** Magát a hasznos adatot (pl. hangminta, bejövő soros byte) írjuk ide, vagy olvassuk ki innen[cite: 417, 418, 419].
2.  **Állapot-regiszterek (Status):** Ezeket jellemzően csak olvassa a szoftver, hogy lekérdezze a periféria aktuális állapotát (pl. "Van-e olvasatlan adat a FIFO-ban?", "Történt-e hiba?")[cite: 420, 421, 422].
3.  **Üzemmód-regiszterek (Configuration):** A periféria beállítására, működési módjának megadására szolgálnak (pl. milyen sebességgel működjön a soros port). Írhatók és visszaolvashatók[cite: 423, 424, 425].
4.  **Parancs-regiszterek (Control):** Olyan regiszterek, amelyekbe többnyire csak írni lehet, és egy adott folyamat elindítását, leállítását, vagy resetelését végzik (pl. Timer indítása)[cite: 426, 427, 428].

**15. lap: A regiszterek bitszintű szerkezete**
* A 32 bites regiszterek belseje **bitmezőkre (bitfields)** van osztva[cite: 492]. Például a 0-2 bitek mást jelentenek, mint a 3-5 bitek (pl. a `TIDR` regiszter ábráján a 10-8. bit a Major Revision-t adja meg)[cite: 545].
* Fontos fogalom a **Reserved (fenntartott)** bit: Ezeket a gyártó fenntartja jövőbeli használatra, szoftverből nem szabad őket módosítani[cite: 493, 509].
* **Hardveres kölcsönös kizárás (Árnyékregiszterek):** Ha egy adat hosszabb, mint a gépi szó (pl. egy 64 bites adat egy 32 bites rendszeren), azt nem lehet egyszerre kiolvasni. Ilyenkor a hardver árnyékregisztereket alkalmaz: amikor kiolvassuk az alsó 32 bitet, a hardver befagyasztja (egy árnyékregiszterbe másolja) a felső 32 bitet is, így a következő olvasásnál egy konzisztens állapotot kapunk[cite: 495, 496, 497].

---

## 5. PIN konfiguráció funkciója

**16-17. lap: Mit jelent a PIN konfiguráció?**
* A modern SoC-k több belső funkcióval (UART, I2C, PWM) rendelkeznek, mint ahány fizikai lábuk (PIN, Pad) van[cite: 552]. Ezért a lábak funkciója nem statikus, hanem szoftveresen beállítható[cite: 563].
* Egy belső multiplexer mátrix teremti meg az összeköttetést a belső periféria és a fizikai kivezetés között[cite: 563]. (Például az AM3358-nál egy láb akár 8 különböző funkciót is felvehet a `MUXMODE` regiszter beállításával) [cite: 566, 624].
* A funkció mellett a **PIN konfiguráció felel az elektromos tulajdonságok beállításáért is**, például engedélyezhető a bemeneti vevő (`RXACTIVE`), vagy kiválasztható a fel/lehúzó ellenállás típusa (`PULLTYPESEL`)[cite: 565, 603, 610].

---

## 6. GPIO illesztés: alapszabályok, logikai szint és áram

**18. lap: A GPIO fogalma**
A GPIO (General Purpose Input/Output) egy általános célú ki- és bemenet, amelynek értékét (magas/alacsony szint) a szoftver közvetlenül tudja olvasni vagy írni[cite: 632, 634, 636].

**20. lap: GPIO konfiguráció és alapszabályok**
* **Biztonságos indulás:** Reset után az alapértelmezett állapot szinte mindig **bemenet** (garantáltan energiamentes), hogy elkerüljék a hardveres rövidzárlatot, ha a külső áramkör is hajtja a vonalat[cite: 704, 710]. * A lábak lebegésének (bizonytalan állapotának) elkerülése érdekében **felhúzó (pull-up, Vcc-re)** vagy **lehúzó (pull-down, GND-re)** ellenállásokat kell konfigurálni, amíg a szoftver fel nem állítja a végleges üzemmódot[cite: 705, 706, 708].

**21-24. lap: Logikai szintek, áram és illesztés (Input/Output)**
* **Logikai szintek (Input):** A bemenetek feszültségét a hiszterézis határozza meg ($V_{UT}$ - felső küszöb, $V_{LT}$ - alsó küszöb)[cite: 737, 738, 748]. Eltérő feszültségszintű rendszereket (pl. 5V és 3.3V) **tilos közvetlenül összekötni**, mert a kisebb feszültségű tönkremehet, vagy bizonytalan lesz a működés! Erre **logikai szintillesztő IC-ket** (Level shifter) kell használni[cite: 746, 768, 785, 794].
* **Zavar- és túlfeszültségvédelem (Input):** A vezetékeken kialakulhat jelcsengés (ringing) [cite: 796, 800], illetve extrém túlfeszültség (pl. ESD, statikus feltöltődés)[cite: 813]. Ezek ellen a bemeneteket TVS diódákkal vagy szikraközökkel kell védeni[cite: 815, 818].
* **Áram és teljesítmény (Output):** Egy SoC GPIO kimenete nagyon kis áramot bír leadni (pl. max 6 mA)[cite: 750, 751, 851]. Ha túlterheljük, a logikai szint leesik, vagy a chip tönkremegy[cite: 853, 855]. Nagyobb teljesítményű eszközök (pl. motorok, relék, szelepek) meghajtására **kapcsolóüzemű erősítőket** (tranzisztorokat, FET-eket, reléket) kell a kimenetre kötni[cite: 857, 860, 861].

---

## 7. A/D és D/A: kvantálás, jeltartomány, mintavételi frekvencia, on-demand vs. időzített

**25. lap: Alaptulajdonságok**
* **Kvantálás:** Az Analóg-Digitális (A/D) átalakító a folytonos analóg feszültségszintet egy $N$ bites diszkrét digitális számmá alakítja (pl. 16, 24 bit). A D/A ennek a fordítottja[cite: 874, 876]. * **Jeltartomány:** Az az analóg feszültségintervallum, amit a konverter át tud alakítani. A $0$ digitális értékhez tartozik az alsó szint, a $2^{N}-1$ értékhez a maximális szint. Ezen kívül eső feszültségeknél alul- vagy túlcsordulás, extrém esetben a hardver károsodása lép fel[cite: 882, 883].
* **Mintavételi frekvencia:** Megadja, hogy az A/D másodpercenként hányszor vesz mintát a folytonos jelből (vagy a D/A hányszor állít elő új analóg értéket)[cite: 884, 891, 892]. A *Nyquist-Shannon mintavételi tétel* alapján ez legalább a mérendő jel maximális frekvenciakomponensének a kétszerese kell, hogy legyen (pl. emberi hallás 20kHz -> CD minőség 44.1 kHz)[cite: 885, 886, 887, 888].

**26. lap: On-demand és időzített mintavételezés eltérése**
* **On-demand (Igény szerinti):** A szoftver kézzel indítja el az A/D konverziót, amikor épp szüksége van egy adatra. A mintavételi frekvencia itt nem szigorúan állandó (jitteres lehet)[cite: 915]. Jellemző például egyszerű I2C szenzoroknál[cite: 923].
* **Időzített (Hardware-triggered):** Egy hardveres időzítő (Timer) szigorúan, precíz periódusidővel indítja a mintavételeket. Ez biztosítja az állandó mintavételi frekvenciát, az adatok pedig általában egy FIFO memóriába kerülnek, ahonnan DMA mozgatja őket a RAM-ba[cite: 916, 917, 918]. Hangkártyáknál (I2S) kötelező[cite: 925].

---

## 8. Digitális HW komponensek közötti kommunikáció lehetőségei (I2C, SPI, UART)

**29. lap: Az alapvető működések**
Ezek mind integrált áramkörök közötti, panelen (node-on) belüli alacsony sebességű buszok[cite: 1080]:
* **I2C (Inter-Integrated Circuit):** 2 vezetékes, szinkron kommunikációs busz. Egy órajel (SCL) és egy kétirányú adatvonal (SDA) alkotja[cite: 1081, 1082]. Az eszközök címzéssel (jellemzően 7 bit) azonosítják magukat[cite: 1083]. * **SPI (Serial Peripheral Interface):** Minimum 4 vezetékes, szinkron busz. Gyorsabb az I2C-nél. Vonalai: Órajel (CLK), Master-In-Slave-Out (MISO), Master-Out-Slave-In (MOSI), és a csipeket engedélyező Chip Select (CS)[cite: 1087, 1088].
* **UART (Universal Asynchronous Receiver-Transmitter):** 2 vezetékes (TX - adás, RX - vétel), **aszinkron** kommunikáció[cite: 1093, 1094]. Itt nincs külön órajel vezeték, az adó és a vevő előre megállapodott sebességgel (baud rate) működik. 
---

## 9. I2C és SPI eltérése az UART-tól adatelérés szempontjából

**30. lap és a kép leírása:**
A legfőbb elvi különbség az adat **elérésének struktúrájában** van:
* **I2C és SPI esetén:** Ezek az eszközök (pl. egy külső digitális hőmérő IC) **önmagukban is implementálnak egy saját regiszter fájlt**[cite: 1102]. Amikor I2C-n vagy SPI-n kommunikálunk velük, a protokoll része, hogy megcímezzük az eszköz belső memóriacímét (regiszterét), ahova írni vagy ahonnan olvasni akarunk[cite: 1100]. A kommunikáció tehát strukturált regiszter-hozzáférés.
* **UART esetén:** Az UART egy tiszta, nyers "adatcső"[cite: 1094]. **Nincs egységes szabvány vagy megoldás** arra, hogy mi a küldött bájtok jelentése (nincs beépített regiszter-címzés). Csak sorban küldjük és fogadjuk a bájtokat ("bármit és bárhogy küldhetünk"), a keretezést és a parancsok értelmezését a magasabb szintű szoftvernek (protokollnak) kell teljesen egyedileg megoldania.