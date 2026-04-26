Igen, tökéletesen látod! Az általad feltöltött 5. labor (Lab 5) diasora valóban azokat a témaköröket tartalmazza, amik a képen a 4. heti labor követelményként voltak kiírva (Beaglebone megismerése, soros hozzáférés, jogosultságok). 

Kidolgozom neked a kért témaköröket a feltöltött PDF alapján, követve az oktatósegédi, diáról diára haladó módszertant.

***

# Beágyazott Rendszerek - 5. Labor Jegyzet (A 4. heti követelmények alapján)

## 1. Ismerkedtünk a Beaglebone-nal...

**A diasor vonatkozó lapjai: A Beaglebone Black (BBB) hardver architektúrája**
* **Mi is az a Beaglebone Black?** Egy alacsony költségű, nyílt forráskódú, közösség által támogatott fejlesztői platform, amelyet kifejezetten beágyazott rendszerek fejlesztésére és oktatásra terveztek.
* **Főbb hardveres komponensek (A SoC és a memóriák):**
  * **Processzor (SoC):** A kártya "agya" egy **Texas Instruments (TI) Sitara AM3358**-as chip, amely egy **ARM Cortex-A8** magot tartalmaz (jellemzően 1 GHz-es órajelen).
  * **RAM:** **512 MB DDR3** operatív memória található rajta, ami bőségesen elegendő egy beágyazott Linux rendszer (pl. Debian) futtatásához.
  * **Háttértár (FLASH):** A panelre integrálva található egy **4 GB-os eMMC** (embedded MultiMediaCard) flash memória, ezen tárolódik alapértelmezetten az operációs rendszer.
  * **Kiegészítő processzorok:** A chip tartalmaz egy 3D grafikus gyorsítót, egy NEON lebegőpontos gyorsítót, valamint két darab **PRU**-t (Programmable Real-Time Unit). A PRU-k 32 bites, 200 MHz-es mikrokontrollerek, amik szigorú valós idejű (hard real-time) feladatok elvégzésére képesek a fő Linux rendszertől függetlenül.
* **Fizikai interfészek és csatlakozók:**
  * **Tápellátás:** 5V-os DC hálózati adapterről vagy a miniUSB csatlakozón keresztül biztosítható.
  * **Hálózat:** 10/100 Mbps Ethernet csatlakozó.
  * **Kijelző:** microHDMI port videó és audió kimenethez.
  * **Bővíthetőség (Tüskesorok):** Két darab 46 tűs kivezetés (header) található rajta (P8 és P9). Ezeken érhetők el a multiplexelt perifériák: GPIO-k, I2C, SPI, UART, CAN buszok, PWM kimenetek, valamint beépített A/D konverterek analóg bemenetei.
  * **Soros debug port:** Egy dedikált 6 tűs csatlakozó a soros (UART) kommunikációhoz (erről szól a következő pont).

---

## 2. Soros hozzáférés konfigurálása és a jogosultságok fontossága periféria elérés esetén

**A diasor vonatkozó lapjai: Soros porti csatlakozás és a Host PC beállítása**
* **A soros hozzáférés fizikai konfigurálása:**
  * A Beaglebone-hoz való legalacsonyabb szintű hozzáférést a **soros konzol** (UART) biztosítja. Ez azért kritikus, mert ezen keresztül akkor is kapunk hibaüzeneteket és hozzáférést a rendszerhez, ha a hálózati kapcsolat (Ethernet/SSH) még nem él, vagy a rendszer épp bootol.
  * Csatlakozás: Egy **USB-Serial TTL konverter** kábelt használunk a PC és a BBB dedikált 6 tűs debug csatlakozója között.
  * *Figyelem!* Az AM3358-as processzor jelszintje **3.3V**. Szigorúan 3.3V-os TTL kábelt kell használni, az 5V-os kábelek tönkretehetik a processzort!
* **A soros port szoftveres konfigurálása (Paraméterek):**
  * A terminál emulátor (pl. PuTTY Windows-on, vagy `minicom` / `screen` Linuxon) beállításai a következők kell, hogy legyenek:
    * **Baud rate (Sebesség):** 115200 bit/s
    * **Data bits (Adatbitek):** 8
    * **Parity (Paritás):** None (Nincs)
    * **Stop bits (Stop bitek):** 1
    * Flow control (Áramlásszabályozás): None

**A diasor vonatkozó lapjai: Jogosultságok fontossága a periféria elérésnél**
* **Eszközfájlok a Linuxban:** Linux alatt a hardveres perifériák fájlként jelennek meg a `/dev` könyvtárban. A csatlakoztatott USB-soros átalakító a Host PC-n jellemzően a `/dev/ttyUSB0` (vagy `/dev/ttyACM0`) nevet kapja.
* **A probléma a jogosultságokkal:**
  * Ha kiadjuk az `ls -l /dev/ttyUSB0` parancsot, valami ilyesmit látunk a kimenetben: 
    `crw-rw---- 1 root dialout ... /dev/ttyUSB0`
  * Ez azt jelenti, hogy az eszköz tulajdonosa a **`root`** felhasználó, a csoportja pedig a **`dialout`** csoport. Csak ők rendelkeznek írási (`w`) és olvasási (`r`) joggal az adott hardverhez.
  * Egy normál (mezei) felhasználónak alapértelmezetten **nincs joga** közvetlenül írni vagy olvasni a hardveres perifériákat (Permission denied).
* **A jogosultságok fontossága:** Ez egy kritikus biztonsági és stabilitási funkció az operációs rendszerben. Megakadályozza, hogy jogosulatlan programok vagy felhasználók közvetlenül manipulálják a hardvert, ami rendszerösszeomláshoz vagy adatlopáshoz vezethetne.
* **A megoldás (Periféria elérése szabályosan):**
  * Sose futtassuk a fejlesztői eszközeinket (terminál program, saját kód) `root` (vagy `sudo`) jogosultsággal, ha nem muszáj, mert az komoly biztonsági kockázat!
  * Ehelyett a normál felhasználónkat **hozzá kell adni a `dialout` csoporthoz**, amely csoport birtokolja a soros portokhoz való hozzáférést.
  * Parancs Linuxon: `sudo usermod -a -G dialout <felhasználónév>`
  * A parancs kiadása után a felhasználónak ki kell jelentkeznie, majd újra be, hogy a csoporttagság érvénybe lépjen. Ezt követően a periféria (soros port) szabadon, biztonságosan olvasható és írható a normál felhasználó számára is. (Ez a koncepció igaz a beágyazott eszközön lévő I2C, SPI és GPIO eszközfájlok elérésére is).