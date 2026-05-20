Szia! Megértettem a stratégiát: a 2. ZH-ra való felkészülésnél sokkal mélyebben, részletesebben és a legkisebb technikai nüanszokra is kiterjedően fogjuk feldolgozni az anyagokat, hogy a legapróbb részletkérdések se okozhassanak gondot.

A kérésednek megfelelően a 7. előadást (Ütemezés alapjai) kihagyjuk, és azonnal a következő nagy, önálló témakörrel kezdünk, ami az első feltöltött anyagod: a **Bluetooth Low Energy (BLE)**. A Markdown formátumot megtartjuk, a hivatkozásokat pedig igyekszem a lehető legdiszkrétebben elhelyezni a mondatok végén, hogy ne zavarják a GitHubos olvasást.

---

# Bluetooth Low Energy - BLE (8. előadás)

A Bluetooth Low Energy (BLE) nem a klasszikus Bluetooth egyszerűsített változata, hanem egy teljesen alapjaitól újratervezett, eltérő fizikai réteggel és protokollkészlettel rendelkező vezeték nélküli technológia. A technológia gyökerei a Nokia 2001-es Wibree projektjéig nyúlnak vissza, majd 2010-ben integrálódott a Bluetooth 4.0 szabványba.

## 1. Célkitűzések és eltérések a Classic Bluetooth-hoz képest

A klasszikus Bluetooth (BR/EDR) nagy adatátviteli sebességre és folyamatos streamingre (pl. hangátvitel, nagy fájlok) lett tervezve, ami magas áramfogyasztással jár. Ezzel szemben a BLE tervezési fókuszában az alábbiak álltak:

* **Ultra-alacsony fogyasztás:** Olyan kisméretű, telepes eszközök kiszolgálása, amelyek akár évekig képesek működni egyetlen gombelemről (pl. CR2032).
* **Alacsony kitöltési tényező (Low Duty Cycle):** Az eszköz az idő több mint 99%-ában mélyalvásban van, és csak ritkán, nagyon rövid időre (pár milliszekundumos bursts) ébred fel adatot küldeni.
* **Gyors kapcsolatfelépítés:** Míg a klasszikus Bluetooth-nál a párosítás és kapcsolatfelépítés másodpercekig tarthat, a BLE-nél ez néhány milliszekundun alatt lezajlik.

A piacon kétféle chipmegvalósítás létezik:

1. **Single-mode (Smart):** Csak a BLE protokollt támogatja (pl. kis szenzorok, okosórák).
2. **Dual-mode (Smart Ready):** A klasszikus Bluetooth-t és a BLE-t is képes kezelni (pl. okostelefonok, notebookok, átjárók).

## 2. A Fizikai Réteg (PHY) és Frekvenciagazdálkodás

A BLE a licencmentes **2.4 GHz-es ISM sávban** működik.

* **Csatornakiosztás:** A sávot 40 darab, egymástól 2 MHz távolságra lévő csatornára osztja fel (0-tól 39-ig indexelve).
* **Elsődleges Hirdetési Csatornák (Primary Advertising Channels):** Három dedikált csatorna létezik: a **37, 38 és 39-es**. Ezek kritikusak, mert a szoftver ezeken keresztül hirdeti az eszköz létezését, és itt történik a kapcsolatfelvétel is. Úgy lettek kijelölve a frekvenciasávban, hogy pontosan a legelterjedtebb Wi-Fi csatornák (1, 6, 11) közötti "üres" sávokba essenek, radikálisan csökkentve a Wi-Fi okozta interferenciát.
* **Adatcsatornák (Data Channels):** A maradék 37 csatorna (0-36) kizárólag a már felépült kapcsolatok alatti adatátvitelre szolgál.
* **Interferenciavédelem (FHSS):** A BLE adaptív frekvenciaugratásos (Adaptive Frequency Hopping) technológiát használ. Ha egy adatcsatorna zajossá válik (pl. Wi-Fi forgalom miatt), a protokoll automatikusan kihagyja azt a csatornalistából az ugrálási sorrend során.

## 3. A Link Layer (Kapcsolati réteg) Állapotai és Szerepkörei

A legalacsonyabb hardverközeli logikai szint a Link Layer, amely szigorú állapotgéppel szabályozza az eszköz viselkedését. Öt fő állapota van:

1. **Standby:** Alapállapot, nincs rádiós aktivitás, minimális fogyasztás.
2. **Advertising:** Az eszköz periodikusan hirdetési csomagokat küld ki a 37, 38, 39-es csatornákon.
3. **Scanning:** Az eszköz hallgatózik a hirdetési csatornákon, keresi a hirdetőket.
4. **Initiating:** A pásztázó eszköz észleli a kiválasztott hirdetőt, és egy speciális kapcsolódási kérelmet (CONNECT_IND) küld neki.
5. **Connection:** A kapcsolat létrejött, az eszközök innentől kezdve az adatcsatornákon kommunikálnak.

**Szerepkörök a kapcsolat létrejötte után:**

* **Master (Központi):** Az az eszköz, amelyik az `Initiating` állapotból lépett be (kezdeményező). Ő határozza meg az időzítéseket, a kapcsolat paramétereit, és ő a hálózati koordinátor (Central).
* **Slave (Periféria):** Az az eszköz, amelyik `Advertising` állapotból lépett be. Követi a Master által diktált időzítéseket (Peripheral).

## 4. GAP (Generic Access Profile) Szerepkörök

A GAP határozza meg, hogy a külvilág felé hogyan viselkedik az eszköz, hogyan fedezhető fel, és hogyan épít fel kapcsolatot. Négy alapvető GAP szerepkört különítünk el:

* **Broadcaster (Műsorszóró):** Csak hirdetési csomagokat küld, de nem fogad bejövő kapcsolatokat. Tipikus példái a **Beaconök** (pl. iBeacon, Eddystone), amelyek helymeghatározási vagy reklámadatokat szórnak a környezetükbe.
* **Observer (Megfigyelő):** Csak pásztázza a környezetet (Scanning), gyűjti a Broadcasterek adatait, de nem kezdeményez kapcsolatot.
* **Peripheral (Periféria):** Hirdet és **képes kapcsolatot fogadni**. Egyszerre csak egyetlen Central eszközhöz kapcsolódhat. Jellemzően ez a fizikai szenzor vagy beavatkozó.
* **Central (Központi):** Pásztázza a hirdetéseket, kapcsolatokat kezdeményez, és képes egyszerre több Perifériát is kezelni egy csillagtopológiás hálózatban (pl. okostelefon).

## 5. ATT és GATT Architektúra (KULCSKONCEPCIÓ - KIEMELT ZH FÓKUSZ)

A BLE adatszerkezetének alapja a Kliens-Szerver modell, amelyet az ATT (Attribute Protocol) és a GATT (Generic Attribute Profile) valósít meg. **Itt nagyon fontos észben tartani a szerepkörök felcserélhetőségét:** a hardveres Periféria (szenzor) az adatok szempontjából GATT Szerverként működik, míg a hardveres Központi eszköz (telefon) a GATT Kliens.

### ATT (Attribute Protocol)

A szerver az adatokat **Attribútumok** formájában tárolja egy lapos struktúrájú táblázatban. Minden egyes attribútum szigorúan négy részből áll:

1. **Handle (Kezelő):** Egy egyedi, 16 bites index (cím) a táblázatban (0x0001 - 0xFFFF). Ezen keresztül hivatkozik a szoftver közvetlenül az adott adatmezőre.
2. **UUID (Univerzális Egyedi Azonosító):** Meghatározza az attribútum típusát (hogy mi az az adat). A szabványos Bluetooth profilok 16 bites rövidített UUID-t használnak, míg az egyedi, saját fejlesztésű adatokhoz teljes 128 bites UUID-t kötelező generálni.
3. **Value (Érték):** Maga a nyers adat (bájtsorozat).
4. **Permissions (Jogosultságok):** Szabályozza a hozzáférést (Olvasható, Írható, kell-e hozzá titkosítás vagy hitelesítés).

### GATT (Generic Attribute Profile)

A GATT az ATT lapos táblázatát egy emberileg és szoftveresen is jól érthető hierarchiába szervezi: **Profile $\rightarrow$ Service $\rightarrow$ Characteristic $\rightarrow$ Descriptor**.

* **Profile (Profil):** Magas szintű definíció, amely leírja az eszköz teljes működését (pl. szívritmusmérő óra profilja).
* **Service (Szolgáltatás):** Logikailag összetartozó adatok gyűjteménye. Egy profil több szolgáltatásból is állhat. Például a szívritmusmérő tartalmaz egy *Heart Rate Service*-t és egy *Battery Service*-t (akkumulátor állapot).
* **Characteristic (Jellemző):** A tényleges adatot hordozó egység. Tartalmazza magát az értéket (Value) és a hozzá tartozó tulajdonságokat (Properties).
* *Properties (Műveletek):* Meghatározza, hogyan érhető el az adat (Read, Write, Write Without Response, Notify, Indicate).


* **Descriptor (Leíró):** Opcionális metaadatok a jellemzőhöz (pl. mértékegység szövegesen).
* *CCCD (Client Characteristic Configuration Descriptor):* Egy speciális, rendkívül fontos leíró (UUID: 0x2902). A kliens ebbe írással tud feliratkozni a változások automatikus küldésére.



### Notification vs. Indication (FONTOS RÉSZLETKÉRDÉS)

Ha a Szerver (szenzor) aszinkron módon, azonnal el akarja küldeni a megváltozott adatot a Kliensnek, két módszere van:

* **Notification (Értesítés):** A szerver kiküldi az adatot, de a protokoll szintjén **nem vár érte visszaigazolást**. Rendkívül gyors, minimális rádiós időt és energiát igényel. Hátránya, hogy ha a csomag elvész a levegőben, a szerver nem szerez tudomást róla.
* **Indication (Jelzés):** A szerver elküldi az adatot, és **szigorúan megvárja az alkalmazásszintű nyugtázást** a klienstől. Biztonságos, de lassabb, és a hosszabb rádióidő miatt jelentősen több energiát fogyaszt. Amíg a nyugta meg nem érkezik, újabb Indication csomag nem küldhető.

## 6. Hardver Architektúrák és Particionálás (KULCSKONCEPCIÓ - ZH FÓKUSZ)

Amikor beágyazott rendszerbe BLE-t tervezünk, el kell dönteni, hogyan osztjuk fel a szoftveres rétegeket (Alkalmazás, Host, Controller) a fizikai chipek között. Három fő megközelítés létezik:

### 1. Single-chip / SoC (System-on-Chip)

* **Működés:** Az alkalmazási kód és a teljes BLE protokollkészlet (Host + Controller) **egyetlen egy fizikai mikrovezérlőn** (SoC-on, pl. Nordic nRF52, TI CC26xx, ESP32) fut.
* **Előnyök:** A legkisebb fizikai méret, a legalacsonyabb gyártási költség (BOM), és a legjobb energiahatékonyság (nincs chipek közötti kommunikációs áramfelvétel).
* **Hátrányok:** Az alkalmazásodnak szigorúan osztoznia kell a CPU időn és a memórián (RAM/Flash) a BLE szoftveres stack-kel. Ha az alkalmazásod blokkolja a CPU-t, a BLE kapcsolat megszakadhat (időzítési konfliktusok).

### 2. Dual-chip / NCP (Network Co-Processor)

* **Működés:** A teljes BLE protokollkészlet (Host + Controller) egy dedikált, gyárilag előre programozott vezeték nélküli chipen fut. A te fő alkalmazásod egy teljesen különálló, tetszőleges fő processzoron (Host MCU vagy AP) kap helyet. A két chip között soros interfészen (UART vagy SPI) keresztül, egy egyedi parancskészlettel (pl. BGAPI) zajlik a kommunikáció.
* **Előnyök:** Teljesen tehermentesíti a fő processzort a rádiós feladatok alól. Kiválóan alkalmas arra, hogy egy már létező, régebbi beágyazott rendszert utólagosan, minimális kódmódosítással BLE funkcióval egészítsünk ki.
* **Hátrányok:** Nagyobb NYÁK méret, magasabb ár (két chip kell), és nagyobb áramfogyasztás a soros busz folyamatos meghajtása miatt.

### 3. RCP (Radio Co-Processor)

* **Működés:** A rádiós céleszközön csak a legalacsonyabb rétegek (PHY és a Link Layer egy része) futnak. A magasabb szintű protokollrétegek (HCI, L2CAP, ATT, GATT) és maga az alkalmazás már a fő processzoron (tipikusan egy erős Alkalmazás Processzoron - AP, pl. Linux alatt) futnak. A fizikai összeköttetés szabványos **HCI (Host-Controller Interface)** protokollal történik USB-n vagy UART-on keresztül.
* **Alkalmazása:** Ez a klasszikus okostelefonos, notebookos, vagy beágyazott Linuxos (pl. Raspberry Pi) architektúra.

## 7. Fejlesztőrendszer komponensek

A modern BLE fejlesztés már ritkán jelent tisztán bit-szintű assembly vagy C kódolást a rádióra; komoly szoftveres ökoszisztéma támogatja:

* **Rádió konfigurátorok:** Grafikus varázslók az adóteljesítmény, hirdetési periódusidők beállítására.
* **GATT Editor:** Egy vizuális felület (noCode/lowCode megközelítés), ahol egérrel kattintgatva össze lehet rakni a kívánt Szolgáltatásokat és Jellemzőket, beállítani az UUID-kat és jogosultságokat, a szoftver pedig automatikusan kigenerálja a hozzá tartozó C forráskódot.
* **Hálózati analizátorok (Sniffer):** Egy céleszköz segítségével a levegőben lévő rádiócsomagok lehallgathatók és vizuálisan elemezhetők (pl. Wireshark-ban), ami elengedhetetlen a protokollhibák felderítéséhez.
* **Energiafogyasztás-monitorozás (Energy Profiler):** Mivel a BLE lényege az ultra-alacsony fogyasztás, a fejlesztőkártyák beépített árammérő hardverrel rendelkeznek. Grafikonon látható, hogy egy-egy felébredés vagy adatküldés pontosan hány milliampert fogyasztott, így szoftveresen optimalizálható a hardver élettartama.

---

Ez a BLE előadás teljes, részletes anyaga a ZH követelmények szerint. Ha átrágtad magad rajta, küldd a következő előadásanyagot (például a Valós-idejű ütemezés / E01507 fóliát), és folytatjuk ugyanilyen alapossággal!