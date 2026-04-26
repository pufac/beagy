Örömmel segítek a laboranyagok rendszerezésében és kidolgozásában. A megadott szempontok alapján, az 1. és 2. laboratóriumi útmutatók segítségével az alábbi jegyzetet készítettem el a kért témakörök szerint haladva.

### 1. **strace** célja, működése
Ez a szakasz az 1. laboratórium anyagára épül.

* **31. oldal – strace használata**:
    * **Célja**: Az **strace** egy diagnosztikai és hibakereső segédprogram Linux rendszereken, amelynek célja a futó program által végrehajtott **rendszerhívások** (system calls) és a kapott szignálok nyomon követése.
    * **Működése**: A programot az `strace` paranccsal indítva a konzolon megjelenik minden olyan interakció, amelyet a program a kernel felé intéz (pl. fájlok megnyitása, hálózati kommunikáció, memóriakezelés).
    * **Példa**: A dokumentum az `strace ./helloworld` parancsot hozza példaként, ahol megfigyelhető a program futása közben generált összes rendszerhívás.

### 2. Linux alkalmazás részei
Az alkalmazások felépítését és futtatását az 1. labor diái részletezik.

* **25. oldal – Egyszerű alkalmazás futtatása**:
    * **Forráskód**: A fejlesztés alapja a C nyelvű forrásfájl (példa: `helloworld.c`), amely tartalmazza a program logikáját.
    * **Futtatható állomány**: A fordítóprogram (pl. **gcc**) által generált bináris fájl. A dokumentum a `gcc helloworld.c -o helloworld` parancsot mutatja be, ahol a `-o` kapcsoló határozza meg a kimeneti fájl nevét.
    * **Indítás**: Az alkalmazás az aktuális könyvtárból a `./` előtaggal indítható (pl. `./helloworld`), ami jelzi a shell számára, hogy ne a rendszer elérési útvonalaiban (**PATH**), hanem helyben keresse a fájlt.

### 3. Input/output és hibaüzenet csatornák, és azok átirányítása
A standard csatornák kezelése kritikus a beágyazott rendszerek naplózása és diagnosztikája szempontjából.

* **31. oldal – Csatornák átirányítása**:
    * **Standard csatornák**: A Linux folyamatok alapértelmezett kimeneti csatornái:
        * **1**: Standard kimenet (**stdout**), ide kerülnek a program normál üzenetei.
        * **2**: Standard hibacsatorna (**stderr**), ide kerülnek a hibaüzenetek és diagnosztikai adatok (pl. az strace kimenete).
    * **Átirányítás**: A kimenetek fájlba vagy más eszközre irányíthatók.
        * **Példa**: `strace ./helloworld 1>out.txt 2>&1`.
        * **Magyarázat**: Az `1>out.txt` a normál kimenetet menti a fájlba, a `2>&1` pedig a hibaüzeneteket is ugyanoda irányítja, ahová a standard kimenet mutat (így minden egy fájlba kerül).

### 4. Konfiguráció kezelése általános feladata
A 2. labor bevezetője tárgyalja a konfiguráció szükségességét.

* **3. oldal – Konfiguráció kezelése**:
    * **Általános feladat**: A programnak fel kell térképeznie a környezetét, és paraméterezhetőnek kell lennie anélkül, hogy a forráskódot módosítani kellene.
    * **Kérdések, amikre a konfiguráció válaszol**:
        * Hol van a kezelendő eszköz?
        * Mi a távoli szerver címe?
        * Mi az eszköz azonosítója (ID)?
        * Milyen üzemmódban működjön az alkalmazás?
        * Milyen részletességű legyen a naplózás (**logging**)?

### 5. A main() függvény prototípusai
A parancssori paraméterek és környezeti változók átvétele a `main` függvényen keresztül történik.

* **8. oldal – A main() függvény prototípusai**:
    * **Két paraméteres forma**: `int main(int argc, char *argv[])`.
        * **argc**: Az átadott paraméterek száma (beleszámítva a program nevét is).
        * **argv**: Mutatókat tartalmazó tömb, amely a paraméter-sztringekre mutat.
    * **Három paraméteres forma**: `int main(int argc, char *argv[], char *envp[])`.
        * **envp**: A környezeti változókat tartalmazó sztringtömb, amelyen keresztül a program hozzáfér a rendszerkörnyezethez.

### 6. Parancssori paraméterek lehetséges formái és azok kezelésének eszközei (**getopt()**)
A paraméterek feldolgozása szabványosított formákban történik.

* **4. oldal – Parancssori paraméterek**:
    * **Pozícionális paraméterek**: Fix helyen lévő értékek (pl. `./program file.txt`).
    * **Rövid opciók**: Egy kötőjellel jelölt, egykarakteres kapcsolók (pl. `-v`).
    * **Hosszú opciók**: Két kötőjellel jelölt, beszédesebb nevek (pl. `--verbose`).
* **10. oldal – A getopt() függvény**:
    * **Eszköz**: A **getopt()** egy szabványos C könyvtári függvény, amelynek feladata a rövid opciók kényelmes és egységes feldolgozása.
    * **Működése**: Ciklusban hívva sorra veszi a parancssorban megadott kapcsolókat, kezeli az argumentumokat igénylő opciókat, és visszaadja a megtalált karaktert.