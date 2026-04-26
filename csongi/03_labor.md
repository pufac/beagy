Örömmel segítek a felkészülésben! Oktatósegédi szerepkörömnek megfelelően, a kiadott követelmények és a csatolt PDF dokumentum alapján, szigorúan a kért sorrendben, lépésről lépésre kidolgozom a kiemelt témaköröket.

Íme a részletes, vizsgára fókuszáló jegyzet a 3. laborhoz:

***

# Beágyazott Rendszerek - 3. Labor Jegyzet

## 1. A getsubopt() és a getopt_long() alapvető céljai

*Megjegyzés: A követelmény szerint itt nem kell a C szintű implementációs részletekbe (ciklusok, switch-case szerkezetek) belemenni, csak az alapvető célt és a működési logikát kell ismerni.*

**5. lap: A `getsubopt()` célja és formátuma**
* **Alapvető cél:** Lehetővé teszi, hogy egyetlen parancssori opcióhoz (kapcsolóhoz) több, kulcs-érték (key-value) párokból álló al-opciót (suboption) adjunk meg[cite: 49, 50].
* **Formátum:** A paramétereket vesszővel elválasztva, `-x key1=value1,key2=value2` formátumban várja[cite: 50]. Ha egy kulcshoz nincs érték megadva, akkor azt bináris/logikai (igaz/hamis) tartalomként értelmezi a rendszer[cite: 51].
* **Példa a diáról:** Egy soros port (UART) paramétereinek megadása egyetlen `-s` kapcsolóval: `-s d=/dev/ttyUSB0,s=115200`[cite: 53, 54]. (Itt a `d` a port eszköze, az `s` a sebessége). Vagy jogosultságok megadása: `-f ro,name=/home/khazy/test.txt` (ahol `ro` = read-only)[cite: 52].

**9. lap: A `getopt_long()` célja**
* **Alapvető cél:** A hagyományos, egykarakteres kapcsolók (pl. `-s`) helyett a hosszú, beszédesebb parancssori opciók (pl. `--longoption`) feldolgozását teszi lehetővé[cite: 104, 105].
* **Használatának indokoltsága:** Komplex, rengeteg beállítást igénylő programoknál elkerülhetetlen a használata[cite: 107]. Adhatók hozzá paraméterek, és kombinálható a `getsubopt()`-tal is[cite: 106].
* **Mikor kerüljük?** Egyszerűbb szoftvereknél felesleges bonyolítást jelent. A hagyományos `getopt()` több mint 60 független egykarakteres opciót enged (kisbetű, nagybetű, számok), ami a `getsubopt()` flexibilitásával kiegészítve többnyire elegendő[cite: 108, 109].

---

## 2. Naplózás, loglevel, syslog.h használata

**11. lap: A naplózás (Logging) alapjai**
* **Célja:** A szoftver programozási és működési hibáinak eltárolása[cite: 124]. Sok esetben a hibakeresés csak a naplók analizálásával lehetséges[cite: 125].
* **Linux standard naplók:** A Linux beépített rendszere a **syslog**, melynek a naplófájljai a FHS (Filesystem Hierarchy Standard) szerint jellemzően a `/var/log/` könyvtár alatt találhatóak[cite: 127, 128].
* A naplózás finomsága (hogy miről készüljön bejegyzés és miről ne) beállítható[cite: 126].

**12. lap: Loglevel (naplózás szintjei)**
A `syslog` 8 különböző prioritási szintet (loglevel) különböztet meg, 0-tól 7-ig[cite: 142]. Ezek határozzák meg a hiba vagy esemény súlyosságát:
1.  **0 - Emergency (emerg):** A rendszer használhatatlan (System is unusable)[cite: 142].
2.  **1 - Alert (alert):** Azonnali beavatkozás szükséges (Action must be taken immediately)[cite: 142].
3.  **2 - Critical (crit):** Kritikus állapot (Critical Conditions)[cite: 142].
4.  **3 - Error (err):** Hibaállapot (Error conditions)[cite: 142].
5.  **4 - Warning (warning):** Figyelmeztetés (Warning Conditions)[cite: 142].
6.  **5 - Notice (notice):** Normális, de szignifikáns állapot (Normal but significant conditions)[cite: 142].
7.  **6 - Informational (info):** Informatív üzenet (Informational messages)[cite: 142].
8.  **7 - Debug (debug):** Fejlesztői, hibakeresési szintű üzenet (Debug-level messages)[cite: 142].

**13. lap: A `syslog.h` használata - Elmélet**
* Ez a Linux (és UNIX-ok) beépített C-s naplózó API-ja[cite: 148].
* **Fontos vizsgakérdés lehet:** A naplózás szintjét beállító `setlogmask` függvény **nem szálbiztos (not thread-safe)!** Erre több szálú (multithreaded) környezetben különösen figyelni kell[cite: 151, 153].
* **Korlátok:** A syslog csomag alapból nem tudja megoldani, hogy egy tetszőleges, saját fájlba naplózzunk (beleértve a terminál standard kimeneteit, a `stdout/stderr` fájlokat is)[cite: 154, 155].

**14. lap: A `syslog.h` használata - Példakód és megtekintés**
A diasor az alábbi minimális lépéseket mutatja be a syslog használatára:
```c
#include <syslog.h>
// 1. A maximális logolási szint beállítása (pl. mi csak a g_loglevel szintig kérjük a logokat)
setlogmask (LOG_UPTO (g_loglevel)); 

// 2. Kapcsolat megnyitása a naplózó daemonnal
openlog ("myprog", LOG_CONS | LOG_PID | LOG_NDELAY, LOG_USER);

// 3. Naplóüzenetek küldése a megadott prioritással (loglevel)
syslog (LOG_NOTICE, "Program started by User %d", getuid ());
syslog (LOG_INFO, "I am here!");

// 4. Kapcsolat lezárása
closelog ();
```
[cite: 169, 170, 171, 172, 173]
* **Hogyan nézzük meg az eredményt?** A terminálból a `tail /var/log/syslog` vagy a `tail -n X /var/log/syslog` parancsokkal[cite: 176].

**15. lap: Modern logolás - `journald`**
* Modern Linux rendszerekben (ahol systemd van) a naplókat a **journald** kezeli[cite: 181].
* Ezek már **bináris fájlok**, nem sima szöveges fájlok, és a `/var/log/journal/` alatt tárolódnak[cite: 182, 183].
* Megtekintésük a `journalctl -f` paranccsal lehetséges, amely folyamatosan mutatja az új bejegyzéseket[cite: 184]. Ebből generálódik vissza a hagyományos karakteres log (disztribúció függő módon)[cite: 186].

---

## 3. Naplózás kérdései a beágyazott rendszerekben

**11. lap: Kihívások és problémák**
A naplózás beágyazott környezetben (ellentétben egy PC-vel vagy szerverrel) komoly tervezési kérdéseket vet fel, elsősorban a szűkös hardveres erőforrások miatt:

1.  **Nincs írható fájlrendszer:** Egyáltalán nem biztos, hogy az adott beágyazott eszközön rendelkezésre áll írható fájlrendszer a lokális `/var/log` könyvtár számára[cite: 133].
2.  **Távoli naplózás (Remote log) hibái:** Megoldás lehet a hálózaton keresztüli (távoli) naplózás, de hálózati szakadás vagy hiba esetén a naplók elvesznek. (Ez gyakran UDP alapú, vagy `sshfs`-el felcsatolt rendszer) [cite: 133, 134]. Ha a naplók elvesznek, a hibakeresés szinte lehetetlenné válik[cite: 135]. (Megoldás lehet egy kis RAM diszk alkalmazása, de annak tartalma újraindításkor elvész [cite: 135]).
3.  **A FLASH memória tönkretétele (A legfontosabb probléma):** Ha van is háttértár, az többnyire kis méretű FLASH memória (pl. eMMC vagy SD kártya). Mivel a FLASH memóriák írási ciklusainak száma korlátozott, az állandó, gyors tempójú naplózás fizikailag **tönkreteszi** a tárolót[cite: 136].
    * *Példa az előadásból:* A tanszéken egy rosszul megírt (túl sokat naplózó) program 2 hét alatt tönkretett egy Beaglebone Black lapkára integrált 4 GB-os eMMC kártyát (míg SD kártyáról a rendszer jól tudna működni, mert az cserélhető)[cite: 136].