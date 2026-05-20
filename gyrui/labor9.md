Szia! Tökéletes, következik a **9. labor (I2C busz és szenzorok illesztése)** anyaga. Ebben a laborban a digitális szenzorok elérésének egyik legelterjedtebb módját, az I2C (Inter-Integrated Circuit) busz Linux alatti kezelését nézzük meg.

A 2. ZH kulcsszavai alapján itt a parancssori eszközökre (`i2c-tools`), a címzési logikára és a user-space / kernel-space C programozási különbségekre fognak koncentrálni.

*(A hivatkozásokat továbbra is a bekezdések és mondatok végére rejtem el!)*

---

# 9. Labor: I2C busz használata és Szenzorok illesztése

## 1. Az I2C kommunikáció alapvető tulajdonságai

Amikor egy SoC-hoz (pl. a BeagleBone processzorához) digitális szenzorokat (hőmérő, gyorsulásmérő) akarunk kötni, gyakran az I2C buszt használjuk.

* **Fizikai réteg:** Mindössze két vezetéket használ: az **SCL** (Serial Clock - órajel) és az **SDA** (Serial Data - adat) vonalakat. Mivel ezek "Open-Drain" (nyitott nyelő) kivezetések, kötelező **felhúzó ellenállásokat (pull-up resistors)** tenni mindkét vonalra, hogy nyugalmi állapotban magas (logikai 1) szinten legyenek.
* **Sebesség:** A szabványos sebesség 100 kHz (Standard mode) vagy 400 kHz (Fast mode), bár léteznek gyorsabb (High-speed) kiterjesztések is.
* **Címzés:** A buszon lévő minden eszköznek (Slave) egyedi címe van. Alapesetben **7 bites címzést** használnak (elméletileg 128, gyakorlatilag a fenntartott címek miatt 112 eszköz fűzhető fel egy buszra), de létezik 10 bites címzés is (bár ez ritka).
* **Irány (R/W bit):** Amikor a Master megszólít egy eszközt, a 7 bites címhez a 8. bitként hozzácsapja a művelet irányát (0 = Write/Írás, 1 = Read/Olvasás).

## 2. Rendszerszintű diagnosztika az `i2c-tools` csomaggal (ZH KÉRDÉS!)

A Linux nagyon erős parancssori eszköztárat biztosít az I2C busz hibakereséséhez és teszteléséhez anélkül, hogy egyetlen sor C kódot kéne írnunk. Ezeknek a parancsoknak a nevét és szerepét garantáltan tudni kell!

* **`i2cdetect`**: Ez a parancs "letapogatja" a megadott I2C buszt, és egy mátrix formájában kilistázza, hogy **milyen című eszközök (szenzorok) válaszoltak** (küldtek ACK bitet) a hálózaton.
* *Tipikus használat:* `i2cdetect -y -r 1` (letapogatja az 1-es számú I2C buszt). Ha egy szenzor címe 0x48, akkor a mátrixban a 40-es sor 8-as oszlopában megjelenik a `48` felirat.


* **`i2cget`**: Egy konkrét I2C eszköz egy **konkrét belső regiszterének tartalmát olvassa ki**.
* *Paraméterei:* Meg kell adni a busz számát, az eszköz I2C címét, és az eszközön belüli regiszter címét (pl. `i2cget -y 1 0x48 0x00`).


* **`i2cset`**: Segítségével értéket (egy bájtot vagy szót) **tudunk beírni** egy I2C eszköz megadott regiszterébe (pl. egy szenzor konfigurálására, felbontásának beállítására).

## 3. C programozás: User-space vs. Kernel-space driver (KULCSKONCEPCIÓ)

Ahogy az elméleti előadáson is szó volt róla, egy I2C szenzor elérésére két szoftveres út létezik a Linuxban. A 9. laborban a User-space (felhasználói tér) módszert gyakoroljuk.

**A. Kernel-space mód (A transzparens módszer):**
Írsz vagy betöltesz egy dedikált Linux kernel modult (drivert) a szenzorhoz. A kernel driver a háttérben kezeli a teljes I2C kommunikációt, te pedig felhasználóként már csak egy normál fájlt látsz (pl. a `sysfs`-ben egy `temp1_input` fájlt), amiből sima `cat` paranccsal kiolvasod a kész Celsius fokot. Az I2C protokoll részletei el vannak rejtve.

**B. User-space mód (A `libi2c-dev` módszer) - Ezt csináltuk a laboron:**
Nem írsz kernel drivert, hanem közvetlenül az I2C busz eszközfájlját (pl. `/dev/i2c-1`) nyitod meg a C programodból. Ezt alacsony szintű API-nak hívjuk.

* **A kommunikáció menete:** 1. Megnyitod a `/dev/i2c-1` fájlt az `open()` rendszerhívással.
2. A speciális **`ioctl()` rendszerhívással** konfigurálod a buszt: megmondod a kernelnek, hogy melyik I2C című Slave eszközzel akarsz beszélgetni (az `I2C_SLAVE` paraméter megadásával).
3. Ezután szabványos `read()` és `write()` hívásokkal, vagy a `libi2c-dev` könyvtár által biztosított (pl. `i2c_smbus_read_byte_data()`) segédfüggvényekkel konkrét bájtokat küldesz és fogadsz a szenzor regisztereiből.
4. *Fontos:* Ebben a módban a C programodnak (neked) kell tudnia, hogy a szenzor milyen regisztertérképpel rendelkezik, és neked kell a nyers bájtokat (pl. a két bájton érkező hőmérséklet-adatot) biteltolással (bit-shift) összerakni és Celsius fokká konvertálni.

## 4. Analízis Protokoll Dekóderrel

Ha a szoftver nem működik, meg kell vizsgálni a fizikai vonalat (SDA/SCL) egy oszcilloszkóppal vagy logikai analizátorral.

* A műszer beépített protokoll dekódere képes a fizikai magas/alacsony jelszinteket visszafordítani a képernyőn olvasható hexadecimális I2C bájtokká. Ezzel ellenőrizhető, hogy az eszközök küldenek-e **ACK (Acknowledge - Nyugta)** biteket, vagy **NACK (Not Acknowledge)** jön, ami azt jelenti, hogy rossz címet szólítottunk meg, vagy a szenzor nincs bedugva.

---

Ez a 9. labor fókuszált anyaga, ami az I2C busz mesterszintű linuxos kezelését fedi le. Ha feldolgoztad, jelezd, és mehetünk is a **10. laborra** (időkezelés, date, chrony, PTP)!