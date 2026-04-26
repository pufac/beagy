Ez a jegyzet a **VIMIAD04** laboratóriumi anyagai és a **kulcsszavak.pdf**-ben meghatározott ZH-témakörök alapján készült. A laborokat heti bontásban, az előírt szigorú szekvencialitással és részletességgel dolgoztam ki.

---

## 1-2. Labor (1. és 2. hét)

### **strace célja és működése**
**VIMIAD04_lab1.pdf, 2. oldal**
* **Célja**: A program és az operációs rendszer (kernel) közötti interakció, azaz a **rendszerhívások** (system calls) és a kapott **jelzések** (signals) nyomon követése. 
* **Működése**: Megmutatja, milyen paraméterekkel hívódott meg egy rendszerhívás (pl. `open`, `read`, `write`), és mi volt annak a visszatérési értéke. Segít a hibakeresésben, ha nem tudjuk, miért bukik el egy művelet (pl. hiányzó jogosultság vagy fájl). 

### **Linux alkalmazás részei**
**VIMIAD04_lab1.pdf, 3. oldal**
* **Fájlformátum**: Az alkalmazások jellemzően **ELF** (Executable and Linkable Format) formátumúak. 
* **Szegmensek**:
    * **Text**: A futtatható gépi kód (csak olvasható). 
    * **Data**: Inicializált globális és statikus változók. 
    * **BSS**: Inicializálatlan globális változók (a betöltéskor nullázódnak). 
    * **Stack**: Lokális változók és függvényhívási adatok. 
    * **Heap**: Dinamikusan lefoglalt memória. 

### **Input/output és hibaüzenet csatornák, átirányítás**
**VIMIAD04_lab1.pdf, 5-6. oldal**
* **Alapértelmezett csatornák**:
    * **stdin (0)**: Szabványos bemenet (billentyűzet). 
    * **stdout (1)**: Szabványos kimenet (képernyő). 
    * **stderr (2)**: Szabványos hibaüzenet csatorna (képernyő). 
* **Átirányítás**:
    * `>`: stdout átirányítása fájlba (felülírás). 
    * `>>`: stdout átirányítása fájlba (hozzáfűzés). 
    * `2>`: stderr átirányítása fájlba. 
    * `|` (pipe): Egyik program kimenetének átadása a másik bemeneteként. 

### **Konfiguráció kezelése**
**VIMIAD04_lab2.pdf, 2. oldal**
* Az alkalmazások működését paraméterezni kell, hogy ne kelljen újrafordítani őket minden változtatáshoz. 
* **Módjai**: Parancssori paraméterek, környezeti változók vagy konfigurációs fájlok. 

### **A main() függvény prototípusai**
**VIMIAD04_lab2.pdf, 3. oldal**
* `int main(int argc, char *argv[], char *envp[])`
* **argc**: A parancssori paraméterek száma (beleértve a program nevét is). 
* **argv**: Mutatók tömbje a paraméter-karaktersorozatokra. 
* **envp**: A környezeti változók listája (név=érték formában). 

### **Parancssori paraméterek és a getopt()**
**VIMIAD04_lab2.pdf, 4-5. oldal**
* A paraméterek lehetnek **opciók** (jellemzően `-` jellel kezdődnek) és **argumentumok**. 
* **getopt()**: Szabványos függvény az opciók feldolgozására. Megadható neki egy "option string" (pl. `"ab:c"`), ahol a kettőspont jelzi, ha az adott opcióhoz érték is tartozik. 

---

## 3. Labor (3. hét)

### **getsubopt() és getopt_long()**
**VIMIAD04_lab3.pdf, 2. oldal**
* **getopt_long()**: Lehetővé teszi a hosszú opciónevek használatát (pl. `--help` a `-h` helyett). Segít a felhasználóbarátabb parancssori felület kialakításában. 
* **getsubopt()**: Opciók értékein belüli al-opciók (vesszővel elválasztott lista) feldolgozására szolgál (pl. `-o mount,rw,sync`). 

### **Naplózás és syslog.h**
**VIMIAD04_lab3.pdf, 3-4. oldal**
* **syslog**: Központi naplózási szolgáltatás Linux alatt. 
* **Loglevel**: A hibaüzenetek fontossági szintje (pl. `LOG_EMERG`, `LOG_ERR`, `LOG_WARNING`, `LOG_INFO`, `LOG_DEBUG`). 
* **Használata**: `openlog()` a kapcsolat megnyitásához, `syslog()` az üzenet küldéséhez, `closelog()` a lezáráshoz. 

### **Naplózás beágyazott rendszerekben**
**VIMIAD04_lab3.pdf, 5. oldal**
* Beágyazott környezetben korlátozott a tárolókapacitás (Flash memória kopása), ezért kerülni kell a túlzott naplózást. 
* Megoldás: Csak a kritikus hibák mentése, vagy hálózati naplószerver használata. 

---

## 4. Labor (4. hét)

### **BeagleBone és soros hozzáférés**
**VIMIAD04_lab4.pdf, 2-3. oldal**
* A BeagleBone egy népszerű beágyazott Linux kártya (SBC). 
* **Soros hozzáférés**: Gyakran az elsődleges hibakeresési csatorna (konzol). A `/dev/ttyS0` vagy hasonló fájlon keresztül érhető el. 
* **Jogosultságok**: A periféria fájlok eléréséhez a felhasználónak megfelelő jogosultsággal kell rendelkeznie (vagy a `dialout` csoport tagjának kell lennie), különben "Permission denied" hibát kap. 

---

## 5. Labor (5. hét)

### **Jelzések (Signals) életciklusa**
**VIMIAD04_lab5.pdf, 2. oldal**
1.  **Generation**: Az esemény bekövetkezte (pl. `Ctrl+C` vagy `kill` parancs). 
2.  **Delivery**: A kernel eljuttatja a jelzést a folyamathoz. 
3.  **Deposition**: A folyamat reagál rá (lefut a handler vagy az alapértelmezett akció). 

### **Reakciók és kezelés Linuxban**
**VIMIAD04_lab5.pdf, 3-4. oldal**
* **Lehetséges reakciók**: Alapértelmezett (ignore, terminate, stop), jelzés elnyomása (ignore), vagy saját kezelőfüggvény (**signal handler**) futtatása. 
* **Mit kezelünk?**: Pl. `SIGINT` (tiszta leállás), `SIGHUP` (konfiguráció újratöltése). Nem kezelhető/blokkolható a `SIGKILL` és `SIGSTOP`. 

### **Jelzések és blokkoló hívások (sleep példa)**
**VIMIAD04_lab5.pdf, 5. oldal**
* Ha egy folyamat blokkoló hívásban van (pl. `sleep()`), egy érkező jelzés **megszakítja** azt. A hívás hamarabb visszatér, és a visszatérési értéke jelzi a hátralévő időt. 

---

## 6-7. Labor (6. és 7. hét)

### **BeagleBone felépítése és GPIO**
**VIMIAD04_lab6.pdf, 2-3. oldal**
* A BeagleBone konzol portja (soros debug) kritikus az indulási folyamat követéséhez. 
* **GPIO sysfs**: A `/sys/class/gpio` mappában fájlműveletekkel kezelhető.
    * `export`: Létrehozza a fájlokat egy adott lábhoz. 
    * `direction`: Beállítja a lábat bemenetre (`in`) vagy kimenetre (`out`). 
    * `value`: Itt írható (`1`/`0`) vagy olvasható az érték. 

### **Az fflush() szerepe**
**VIMIAD04_lab6.pdf, 4. oldal**
* A Linux puffereli a fájlba írást a hatékonyság érdekében. Ha azonnal látni akarjuk az eredményt (pl. egy LED-nél a GPIO fájlba írás után), az `fflush()` kényszeríti a puffer kiürítését a lemezre/hardverre. 

### **Várakozás és rendszeridő**
**VIMIAD04_lab6.pdf, 5-6. oldal**
* **Várakozás**: `sleep()` (másodperc), `usleep()` (mikrosekundum), `nanosleep()` (nanoszekundum - ez a legpontosabb). 
* **Formátumok**: `time_t` (Unix timestamp, 1970 óta eltelt mp), `struct timespec` (másodperc + nanomásodperc). 

### **Óratípusok és Input kezelés**
**VIMIAD04_lab6.pdf, 7-8. oldal**
* **CLOCK_REALTIME**: Falióra idő (állítható, ugorhat). 
* **CLOCK_MONOTONIC**: Rendszerindulás óta telt idő (nem állítható, folyamatos, időmérésre ez a jó). 
* **Input kezelés**:
    * **Polling**: Ciklusban folytonos kérdezgetés (100% CPU-t eszik). 
    * **Lekérdezés várakozással**: Ciklus + `usleep` (kisebb CPU terhelés, de válaszidő késik). 
    * **poll()**: A kernel értesít, ha változás van (leghatékonyabb, 0% közeli CPU, azonnali válasz). 