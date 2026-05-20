Szia! Megérkeztünk a legvégére, ez a **10. labor (Időkezelés Linux alatt)**. Ez az anyag tökéletesen lezárja a 10. (Időkezelés elosztott rendszerekben) és a 11. (TSN hálózatok) elméleti előadásokat, megmutatva, hogyan állítjuk be a szinkronizációt a gyakorlatban, parancssorból.

*(A `` hivatkozásokat a megszokott módon a pontok és bekezdések végére rejtettem!)*

Íme a 2. ZH kulcsszavaira maximálisan fókuszáló összefoglaló a 10. laborból:

---

# 10. Labor: Időkezelés és Szinkronizáció (NTP, PTP)

## 1. A `date` program és a Linux órái

A legegyszerűbb, idővel kapcsolatos felhasználói eszköz a Linuxban a `date` parancs.

* **Alapfunkciók:** Paraméterek nélkül kiírja az aktuális helyi időt. Képes tetszőleges formátumban (format string) kiírni az időt, például UNIX timestampként (másodperc.nanomásodperc), vagy megmutatni, hogy egy másik időzónában (pl. Hawaii-on) mennyi az idő.
* **Időbeállítás:** Bár lehet vele időt is beállítani (pl. egy teljesen rossz dátumot), ez modern, hálózatba kötött rendszerekben **veszélyes és kerülendő**. Ha a `date`-tel hirtelen "átugrasztod" az időt (Stepping), a futó programok (pl. adatbázisok, makefile-ok, időzítők) összeomolhatnak.
* **A háttér:** A `date` a Linux **`CLOCK_REALTIME`** nevű szoftveres rendszeróráját kérdezi le. Óraszinkronizáció esetén az a cél, hogy ezt az órát folyamatosan, lassan gyorsítva/lassítva (Slewing) igazítsuk a pontos időhöz, ne pedig hirtelen ugrásokkal.

## 2. Az alapértelmezett SNTP kliens: `systemd-timesyncd` és `timedatectl`

A modern Linux disztribúciók (mint az Ubuntu vagy a Debian) alapértelmezetten egy nagyon pehelysúlyú megoldást használnak az idő szinkronizálására.

* **`systemd-timesyncd` tulajdonságai:** Ez egy **SNTP (Simple NTP) kliens**. Lényeges korlátja, hogy *csak szinkronizálni tudja* a mi gépünket egy távoli Internetes szerverhez, de **nem képes NTP szerverként viselkedni**, azaz nem tudja kiszolgálni a helyi hálózatunk többi gépét. (Ráadásul kevésbé kifinomult a hálózati jitter kiszűrésében).
* **`timedatectl`:** Ez a parancssori eszköz a `systemd` időkezelő moduljának vezérlőpultja. Segítségével lekérdezhetjük a szinkronizáció állapotát (`timedatectl status`), beállíthatjuk az időzónát, vagy ki/bekapcsolhatjuk a hálózati szinkronizációt (`sudo timedatectl set-ntp true/false`).

## 3. Haladó NTP és az elszigetelt hálózatok: A `chrony` (ZH KÉRDÉS!)

Ha komolyabb NTP megoldásra van szükség (pl. a BeagleBone-unkkal akarjuk szinkronizálni a többi lokális szenzort), akkor a `systemd-timesyncd`-t le kell cserélni a sokkal robusztusabb `chrony`-ra. A `chrony` kliensként és szerverként is kiválóan funkcionál.

**Konfiguráció és a `local stratum 10` trükk:**

* A beállítása az `/etc/chrony/chrony.conf` fájlban történik (itt adjuk meg, pl. a `pool.ntp.org` szervereit).
* **Mi van, ha nincs Internet? (Nagyon fontos koncepció):** Ha egy ipari gép vagy egy autó hálózata (ahol a BeagleBone a főnök) el van vágva a külvilágtól (Izolált hálózat), az NTP szerverek normál esetben leállítják a szinkronizációt, mert ők maguk sem látnak referenciát.
* Erre való a konfigurációban a **`local stratum 10`** direktíva! Ez megmondja a `chrony`-nak, hogy: *"Ha megszakad a kapcsolat a külső pontos órákkal, akkor tekintsd a saját (esetleg pontatlanabb) belső hardverórádat 10-es Stratum szintű referenciának, és **továbbra is szolgáld ki a helyi hálózatot**!"*. Így az elszigetelt gépek legalább egymáshoz képest tökéletesen szinkronban maradnak.

**Diagnosztika `chronyc`-vel:**
A `chrony` démon működését a `chronyc` (Chrony Client) paranccsal vizsgálhatjuk:

* **`chronyc tracking`**: Megmutatja az aktuális szinkronizáció állapotát, a kiválasztott referenciát és a mi óránk eltérését.
* **`chronyc sourcestats`**: Részletes statisztikát ad a beállított NTP forrásokról (melyik mennyire stabil, mekkora a késleltetésük).

## 4. PTP támogatás ellenőrzése és az `ethtool`

Mielőtt mikroszekundumos pontosságú IEEE 1588 (PTP) hálózatot építenénk, ellenőrizni kell, hogy a hardverünk (Ethernet kártyánk) alkalmas-e a **hardveres időbélyegzésre** (Hardware Timestamping).

* **Az `ethtool` szerepe:** Ezzel a programmal lehet lekérdezni a hálózati kártya (NIC) driverének és fizikai rétegének képességeit.
* **A parancs:** `ethtool -T eth0` (ahol az eth0 a kártya neve).
* **A kimenet:** Ha a kimenetben szerepelnek a `SOF_TIMESTAMPING_TX_HARDWARE` és `RX_HARDWARE` flag-ek, illetve a PTP Hardware Clock mezőben egy id (pl. 0), akkor a kártyánk képes a hardveres bélyegzésre. Ha csak szoftveres flagek vannak, a PTP csak NTP szintű pontosságot fog hozni.

## 5. A `linuxptp` csomag és a `ptp4l` program

A Linux alatt a PTP (IEEE 1588) protokollt a `linuxptp` szoftvercsomag valósítja meg. Ennek a lelke a **`ptp4l`** démon.

* **Szerepe:** A `ptp4l` felel a hálózaton a PTP csomagok (Sync, Follow_Up, Delay_Req, Delay_Resp) küldéséért és fogadásáért, a BMC (Best Master Clock) algoritmus futtatásáért, valamint a hálózati kártyán lévő fizikai PTP hardveróra (PHC) frekvenciájának és fázisának állításáért.
* **Futtatása:** Parancssorból pl. a `sudo ptp4l -m -q -i eth0` paranccsal indítható, ahol az `-i` a hálózati interfészt, az `-m` pedig a terminálos kiíratást (logolást) jelenti.
* **Master/Slave kikényszerítése (`priority2`):** Normál esetben a BMC automatikusan dönti el, ki lesz a Grandmaster. Ha viszont egy adott tesztgépet (pl. a tanári asztalon lévőt) mindenképpen Masterré akarunk tenni, a paraméterek között a **`priority2`** értéket kisebbre kell állítani (alapértelmezett a 128, ha 100-ra állítjuk, biztosan ő nyer az "előválasztáson").