Remek taktika! Valóban felesleges elveszni a laborok hosszú leírásaiban, ha a vizsgán célzottan ezekre a fogalmakra lőnek. [cite_start]Nézzük is az 1. és 2. heti laborok sűrítményét, kifejezetten a megadott ZH-kérdésekre fókuszálva[cite: 18].

Ezek a koncepciók alapvetőek a Linux alatti C programozáshoz, és remekül összekapcsolódnak az elméleti órákon tanult kernel/user-space elmélettel.

---

# 1. és 2. Labor: Linux programozás alapjai

## 1. Linux alkalmazás részei (Memóriakép)
[cite_start]Amikor egy lefordított C program elindul Linux alatt, az operációs rendszer egy jól meghatározott memóriastruktúrát (folyamat virtuális gép) hoz létre számára[cite: 20]. A program a memóriában alapvetően az alábbi logikai részekre (szegmensekre) oszlik:

* **Text (Code) szegmens:** Maga a végrehajtható gépi kód. Általában csak olvasható (read-only), hogy a program ne tudja véletlenül felülírni a saját utasításait.
* **Data szegmens:** Az inicializált globális és statikus változók (pl. `int x = 5;`).
* **BSS szegmens:** A nem inicializált (vagy nullára inicializált) globális és statikus változók (pl. `int y;`). Az OS induláskor automatikusan nullákkal tölti fel.
* **Heap (Kupac):** A dinamikusan foglalt memória területe (C-ben a `malloc()`, `calloc()` által kért memória). Felfelé növekszik.
* **Stack (Verem):** A lokális változók, a függvényhívások paraméterei és a visszatérési címek tárolási helye. Lefelé növekszik.

## 2. Input/output és hibaüzenet csatornák, és azok átirányítása
[cite_start]A Linux (és minden UNIX-szerű rendszer) alapelve, hogy minden futó program kap alapból 3 darab megnyitott adatcsatornát (fájlleírót)[cite: 21]:

* **`stdin` (0 - Standard Input):** Alapértelmezetten a billentyűzet. Innen olvas a `scanf()`.
* **`stdout` (1 - Standard Output):** Alapértelmezetten a képernyő (terminál). Ide ír a `printf()`. Itt jelennek meg a program normál "eredményei".
* **`stderr` (2 - Standard Error):** Szintén a képernyő, de ez egy **különálló csatorna**, ami kizárólag a hibaüzenetek és diagnosztikai adatok kiírására szolgál.

**Átirányítás (Redirection):**
Mivel ezek fájlként viselkednek, a parancssorból könnyen átirányíthatjuk őket fájlokba vagy más programokba:
* `> fájl.txt`: A `stdout`-ot beleírja egy fájlba (felülírja, ha már létezik).
* `>> fájl.txt`: A `stdout`-ot hozzáfűzi egy fájl végéhez.
* `< fájl.txt`: A `stdin`-t a billentyűzet helyett egy fájlból kapja meg.
* `2> hiba.log`: Csak a `stderr`-t (hibaüzeneteket) irányítja fájlba (ez a '2' azonosító miatt van így).
* `|` (Pipe): Az egyik program `stdout`-ját egyenesen beköti a másik program `stdin`-jébe (pl. `ls -l | grep "txt"`).

## 3. Konfiguráció kezelés és a parancssori paraméterek
Egy jól megírt beágyazott szoftvernek rugalmasnak kell lennie. 
* [cite_start]**Általános feladata[cite: 22]:** Lehetővé teszi, hogy a program működését, beállításait (pl. melyik I2C címen keressen eszközt, vagy milyen részletes legyen a logolás) a kód újrafordítása nélkül tudjuk módosítani.
* Ennek leggyorsabb módja a program indításakor megadott **parancssori paraméterek** használata.

### A `main()` függvény prototípusai
A C program belépési pontja a `main()` függvény. [cite_start]Ahhoz, hogy a Linux át tudja adni a parancssori paramétereket a programunknak, a `main`-t különböző prototípusokkal lehet definiálni[cite: 23]. Csak a jellegüket kell tudni:

1.  **Paraméterek nélküli:** `int main(void);` (Ha nem érdekelnek a bemenetek).
2.  **Parancssori paraméteres:** `int main(int argc, char *argv[]);`
    * Itt kapjuk meg az indításkor beírt szavakat. Az `argc` a paraméterek darabszáma, az `argv` pedig egy tömb, ami magukat a szavakat (sztringeket) tartalmazza.
3.  **Környezeti változós:** `int main(int argc, char *argv[], char *envp[]);`
    * Ugyanaz, mint az előző, de egy harmadik paraméterként megkapjuk az operációs rendszer környezeti változóit is (pl. `PATH`, `USER` változók értékeit).

### A paraméterek formái és a `getopt()`
[cite_start]A parancssori paraméterek jellemzően opciókból (kapcsolókból) és argumentumokból állnak (pl. `-h` helphez, vagy `-f config.txt` fájl megadásához)[cite: 23].
* Mivel ezeknek a kézi feldolgozása (az `argv` tömbben való keresgélés) nehézkes és hibalehetőségekkel teli, a C szabványkönyvtár biztosítja a **`getopt()`** függvényt.
* **Célja:** Szabványos, robusztus módon "kiparcolja" (feldolgozza) a rövid (egybetűs, mínusz jellel kezdődő) parancssori kapcsolókat. [cite_start]Nem kell tudnod a pontos paraméterezését, csak azt, hogy **ez a beépített eszköz a parancssori argumentumok kulturált feldolgozására**[cite: 23].

## 4. Az `strace` célja és működése
[cite_start]Ez egy igazi "jolly joker" eszköz Linux alatt a hibakereséshez[cite: 19].
* **Célja:** Diagnosztikai, hibakereső és oktatási eszköz, amivel beleláthatunk egy lefordított, fekete dobozként működő program lelkébe (anélkül, hogy a forráskód meglenne).
* **Működése:** Elfogja (interceptálja) és kilistázza a képernyőre az összes **rendszerhívást (system call)**, amit a vizsgált program a Linux kernel felé tesz. Mivel a programok minden I/O művelethez (fájl olvasás, memóriafoglalás, hálózati kommunikáció) rendszerhívást használnak, az `strace` megmutatja a program pontos viselkedését, a bemeneteket, a kimeneteket, és az esetleges hibakódokat (pl. ha nem talál meg egy konfigurációs fájlt).