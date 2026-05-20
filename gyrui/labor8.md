Szia! Nagyon jól haladunk, jöhet is a **8. labor**! Ebben az anyagban a szoftveres sebességoptimalizálás (és annak veszélyei) kerül fókuszba. Itt hagyjuk el a kényelmes, de lassú Linuxos fájlrendszert, és nyúlunk le a "fémig", azaz közvetlenül a fizikai memóriacímekig.

A ZH-n ez a téma (különösen a mmap, a `/dev/mem` és a `volatile` kulcsszó) szinte biztosan előkerül, szóval kifejezetten részletesen dolgozzuk fel!

---

# 8. Labor: GPIO sebesség és Memóriára leképzett (mmap) elérés

## 1. Ütemezés vizsgálata GPIO-val

A labor kiindulópontja az volt, hogyan tudjuk megmérni, hogy egy program (taszk) valójában milyen gyorsan fut, és mikor kapja meg a processzort az operációs rendszertől.

* **A módszer:** Írunk egy programot, ami egy végtelen ciklusban folyamatosan billenti (ki/be kapcsolja) a GPIO kimenetet. Ha erre rákötünk egy oszcilloszkópot vagy logikai analizátort, a négyszögjel frekvenciájából és torzulásaiból (jitter) pontosan látszik a szoftver sebessége és az OS ütemezőjének viselkedése.
* **A probléma:** A jel frekvenciája nemcsak a processzor sebességétől függ, hanem drasztikusan korlátozza a **GPIO elérés módja** (az alkalmazott szoftveres architektúra).

## 2. A `chrt` program használata (KULCSKONCEPCIÓ)

Ahhoz, hogy a tesztprogramunkat ne zavarja meg a Linux alapértelmezett, nem valós-idejű ütemezője, be kell tudnunk állítani a futó folyamat (processz) prioritását és ütemezési osztályát parancssorból. Erre való a `chrt` (change real-time) segédprogram.

*A legfontosabb kapcsolók, amiket ismerni kell:*

* **`chrt -m`**: Kilistázza a rendszerben elérhető ütemezési osztályokhoz (pl. FIFO, RR) tartozó érvényes prioritási tartományokat (min-max értékeket).
* **`chrt -p PID`**: Lekérdezi egy már futó program (ahol a PID a folyamatazonosító) aktuális ütemezési osztályát és prioritását.
* **`chrt -f <priority> PID`**: A megadott programot átteszi a kőkemény valós-idejű **SCHED_FIFO** (First In, First Out) ütemezési osztályba, a megadott prioritással.
* **`chrt -r <priority> PID`**: A programot átteszi a valós-idejű **SCHED_RR** (Round-Robin) osztályba.
* **`chrt -o 0 PID`**: Visszateszi a programot a normál, nem valós-idejű (**SCHED_OTHER**) osztályba (ennek a prioritása kötelezően 0).

## 3. A `sysfs` és `chardev` elérés sebességproblémái (KULCSKONCEPCIÓ)

Hiába tettük a programunkat SCHED_FIFO osztályba, a GPIO billentés frekvenciája még mindig relatíve alacsony (csak pár kHz) maradt, ha a `/sys/class/gpio/.../value` fájlba írást (sysfs) vagy a chardev megoldást használtuk.

* **Az ok:** Mind a sysfs, mind a chardev megoldás **rendszerhívásokra (syscall)** – mint például a `write()` vagy az `ioctl()` – épül.
* **A következmény:** Minden egyes GPIO értékváltásnál a szoftvernek át kell lépnie a felhasználói térből (user-space) a kernel térbe (kernel-space), majd vissza. Ez a kontextusváltás rengeteg processzor-órajelbe kerül (hatalmas az overhead), ami fizikailag bekorlátozza a maximális kapcsolási frekvenciát.

## 4. Memória alapú periféria elérés - `mmap()` (KULCSKONCEPCIÓ)

Ha extrém gyors (akár MHz-es nagyságrendű) GPIO kezelésre van szükség felhasználói térből, a rendszerhívásokat meg kell kerülni. Erre való a memóriára leképzett (Memory Mapped I/O) technika.

A lépései és alapfogalmai:

* **A fizikai címek kikeresése:** A processzor Reference Manual-jából (adatlapjából) ki kell keresni a GPIO vezérlő pontos fizikai memóriacímét (pl. hol van a SET és CLEAR regiszter).
* **A `/dev/mem` speciális fájl:** Ez a fájl a Linuxban a gép **teljes fizikai memóriáját** (RAM-ot és a hardveres regisztereket) reprezentálja. (Megnyitásához szigorúan root jogosultság szükséges!).
* **Az `mmap()` API:** A C programunkban megnyitjuk a `/dev/mem` fájlt, majd meghívjuk az `mmap()` függvényt. Ez a függvény a kért fizikai memóriacímet (a GPIO regisztert) **belinkeli (leképzi) a programunk virtuális memóriaterületére**.
* **Közvetlen hozzáférés:** Innentől kezdve, ha a C programban a kapott memóriamutatóra (pointerre) írunk egy értéket, az *közvetlenül, a kernel megkerülésével*, azonnal a hardveres GPIO regiszterben landol. Nincs syscall, nincs overhead!

### A `volatile` kulcsszó szerepe (FOGALOM!)

Amikor C-ben a memóriába leképezett hardverregiszterekre mutató pointereket definiálunk, **kötelező** a `volatile` (illékony) kulcsszó használata (pl. `volatile unsigned int *gpio_set_reg;`).

* **Miért?** A fordítóprogramok (compiler, pl. GCC) nagyon okosak. Ha látják, hogy egy ciklusban folyamatosan ugyanarra a memóriacímre írunk, vagy ugyanazt olvassuk anélkül, hogy a kódunk megváltoztatná az értéket, hajlamosak "kioptimalizálni" a műveletet (pl. beteszik az értéket egy CPU regiszterbe, és nem nyúlnak ki a valódi memóriához).
* A `volatile` jelzi a fordítónak, hogy ezen a címen az adat *a program tudtán kívül is megváltozhat* (hiszen egy fizikai hardverről van szó). Így rákényszeríti a fordítót, hogy **minden egyes olvasási és írási műveletet ténylegesen hajtson végre a fizikai címen**, tilos optimalizálni!

## 5. Az `mmap` elérés előnyei és hátrányai (ZH KÉRDÉS!)

Ezt az összehasonlítást garantáltan tudni kell a vizsgán:

**Előnyök:**

* **Extrém sebesség:** Mivel nincsenek rendszerhívások, ez a leggyorsabb módja a perifériák elérésének (a GPIO lábakat a CPU natív órajelével arányos sebességgel lehet kapcsolgatni).

**Hátrányok:**

* **Nem portábilis (Hardverfüggő):** A kódba fixen, "beleégetve" (hardcoded) szerepelnek a BeagleBone specifikus memóriacímek. Ha átviszed a kódot egy Raspberry Pi-ra, nem fog működni, sőt, összeomlik. (A sysfs/chardev ezzel szemben platformfüggetlen).
* **Biztonsági kockázat és instabilitás:** Mivel kikerüljük a kernelt, a Linux operációs rendszer elveszti a kontrollt a periféria felett. A `/dev/mem` írásához root jog kell. Ha a C kódban elszámolod a pointer-aritmetikát, és rossz memóriacímre írsz be, azzal szó szerint **felülírhatod a futó Linux kernel memóriáját, ami azonnali kékhalálhoz (Kernel Panic) és a gép fagyásához vezet**.
* **Fejlesztési nehézség:** Bonyolult bitműveletekkel (maszkolás, bit-eltolás) kell konfigurálni a hardvert, ami hibalehetőségekhez vezet.

---

Ez a labor a beágyazott fejlesztés "sötét, de piszkosul gyors" oldalát mutatta be. Ha ezt a `/dev/mem` + `mmap` + `volatile` szentháromságot így átlátod, a ZH vonatkozó kérdése ujjgyakorlat lesz!

Mehetünk a 9-es laborra?