Király, haladjunk is tovább! A 3. heti labor a parancssori paraméterek haladóbb feldolgozásáról és a rendszerszintű naplózásról (logging) szól. Bár ez egy gyakorlatiasabb anyag, a ZH-n a mögöttes célokra és a beágyazott rendszerspecifikus problémákra kérdeznek rá.

Íme a 3. labor zanzásított, vizsgafókuszú összefoglalója!

---

# Haladó paraméterezés és Naplózás (3. Labor)

## 1. A `getopt_long()` és `getsubopt()` alapvető céljai
A parancssori argumentumok feldolgozására az 1. laborban a sima `getopt()` függvényt használtuk, ami csak a rövid, egybetűs kapcsolókat (pl. `-h`, `-v`) ismeri. A bonyolultabb programoknál ez már kevés.

* **`getopt_long()` célja:** Lehetővé teszi a "hosszú", dupla kötőjeles parancssori kapcsolók feldolgozását (pl. `--help`, `--version`, `--config=file.txt`). Használata sokkal olvashatóbbá, öndokumentálóbbá és felhasználóbarátabbá teszi a programot a terminálban.
* **`getsubopt()` célja:** Az alopciók (suboptions) feldolgozására szolgál egyetlen parancssori argumentumon belül. Gyakran használják olyan esetekben, amikor egyetlen kapcsolóhoz egy egész vesszővel elválasztott listát vagy kulcs-érték párokat adunk meg (pl. a Linux `mount` parancsánál: `-o ro,sync,uid=1000`). Ez a függvény segít ezt a hosszú sztringet darabokra vágni és értelmezni a kódodon belül.

## 2. Naplózás (Logging), Loglevel és a `syslog.h`
Beágyazott rendszereknél ritkán van rákötve egy monitor a gépre, így ha a szoftver összeomlik, a naplófájlok (logok) jelentik az egyetlen esélyt a hiba kiderítésére.

**A Loglevel (Naplózási szint) koncepciója:**
Nem minden üzenet egyformán fontos. A naplózási szintekkel kategorizáljuk az üzeneteket, így a végleges rendszerben kiszűrhetjük a felesleges "zajt", míg fejlesztés alatt mindent láthatunk. Tipikus szintek sorrendben:
* `DEBUG`: Fejlesztői, nyomkövetési információk (nagyon bőbeszédű).
* `INFO`: Normál, de fontos üzemi események (pl. "Hálózati kapcsolat felépült").
* `WARNING`: Nem kritikus probléma, a program még fut tovább.
* `ERR`: Hiba történt, egy funkció nem működik.
* `CRIT` / `EMERG`: Végzetes hiba, a rendszer leáll vagy összeomlott.

**A `syslog.h` használata Linux alatt:**
Ahelyett, hogy minden program saját fájlokba irogatna, a Linux biztosít egy központi naplózó szolgáltatást (syslog daemon). Ezt a `<syslog.h>` könyvtár függvényeivel érjük el:
* `openlog()`: Inicializálja a kapcsolatot a rendszerszintű naplózóval, és beállítja, hogy a programunk neve minden log bejegyzés mellé odafűződjön.
* `syslog()`: Ezzel küldjük el magát az üzenetet. Paraméterként meg kell adni a prioritást (loglevelt, pl. `LOG_ERR` vagy `LOG_INFO`) és magát a formázott szöveget (mint a `printf`-nél).
* `closelog()`: Lezárja a kapcsolatot a logoló szolgáltatással.

## 3. Naplózás kérdései és problémái beágyazott rendszerekben (KULCSKONCEPCIÓ)
A naplózás PC-n nem nagy ügy, de egy korlátos beágyazott rendszerben komoly mérnöki fejtörést okoz. A ZH-n ezt a részt nagyon szeretik kérdezni!

**Fő problémakörök:**
* **Flash memória elhasználódása:** A beágyazott eszközök (pl. Raspberry Pi, egyedi IoT panelek) háttértára legtöbbször SD kártya vagy eMMC Flash memória. Ezek csak véges számú írási ciklust bírnak el. Ha a szoftvered folyamatosan, másodpercenként írja a logokat a kártyára (főleg `DEBUG` módban), azzal **hetek vagy hónapok alatt fizikailag tönkreteheti (elhasználhatja) a memóriát**.
* **Erőforrás (I/O és CPU) lefoglalása:** A fájlba írás lassú művelet. Ha a programod túl sokat logol, a rendszerhívások és az I/O blokkolás miatt drasztikusan lelassulhat az érdemi (valós-idejű) feladatok végrehajtása.

**Tipikus megoldások beágyazott környezetben:**
* **Szűrés:** A végleges termékben (Release) csak a kritikus hibákat (`ERR`, `CRIT`) szabad a háttértárra menteni, a `DEBUG` és `INFO` logokat ki kell kapcsolni.
* **Memóriába logolás (tmpfs):** A logfájlokat nem a fizikai flash memóriára, hanem egy RAM-diszkre (memóriába) írják. Így kímélik a flash-t és nagyon gyors, viszont áramszünet esetén a logok elvesznek.
* **Távoli syslog (Remote Logging):** A beágyazott eszköz egyáltalán nem ment a saját tárhelyére, hanem a hálózaton keresztül UDP csomagokban átküldi a logokat egy távoli, központi szerverre.