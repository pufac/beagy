Ez a harmadik prezentáció (4. előadás) az egyik leggyakorlatiasabb rész, hiszen itt nézzük meg, hogyan beszélget a szoftver a valódi hardverrel regisztereken és fizikai lábakon keresztül.

Íme a **3. prezentáció: Perifériák és interfészek** részletes, GitHub-kompatibilis összefoglalója a ZH-fókuszpontok alapján.

---

# Perifériák és Interfészek (3. prezentáció)

## 1. A periféria szerkezete és a regiszterfájl (KULCSKONCEPCIÓ)
A periféria nem csak egy passzív csatlakozó, hanem egy önálló digitális logika, amelynek saját "felülete" van a CPU felé.

* [cite_start]**Szerkezet:** A periféria két fő részből áll: a funkcionális **digitális logikából** (ami pl. méri az időt vagy kezeli a soros vonalat) és egy **regiszterfájlból**[cite: 3].
* **Regiszterfájl:** Ez a periféria "vezérlőpultja". Ezen keresztül látja a CPU a perifériát. [cite_start]A regiszterek típusai[cite: 5]:
    * **Control (Vezérlő) regiszterek:** Írható regiszterek, itt állítjuk be a működési módot (pl. engedélyezés, sebesség).
    * **Status (Állapot) regiszterek:** Általában csak olvashatók, itt jelzi a periféria, ha végzett egy feladattal vagy hiba történt.
    * **Data (Adat) regiszterek:** Itt zajlik a tényleges adatcsere (be- és kimeneti adatok).

---

## 2. Periféria címzése: Dedikált vs. Memóriára leképzett IO (KULCSKONCEPCIÓ)
Hogyan éri el a processzor ezeket a regisztereket? [cite_start]Két fő iskola létezik[cite: 4]:

1.  **Memóriára leképzett IO (Memory Mapped IO - MMIO):**
    * A periféria regiszterei úgy jelennek meg, mintha memóriacímek lennének.
    * Ugyanazokat az utasításokat (`load`, `store`) használjuk az írásukhoz/olvasásukhoz, mint a RAM esetén.
    * **Előnye:** Nem kell külön utasításkészlet, egyszerű programozni. Ez a legelterjedtebb a modern rendszerekben (pl. ARM).
2.  **Dedikált IO leképzés (Port Mapped IO):**
    * A perifériák egy külön címtartományban (portok) laknak.
    * Speciális CPU utasítások kellenek az elérésükhöz (pl. x86-nál az `in` és `out`).
    * **Előnye:** Nem foglal helyet a memóriacímtartományból.

---

## 3. PIN konfiguráció és Funkció (MUX) (KULCSKONCEPCIÓ)
A modern mikrovezérlőknek sokkal több belső funkciója van, mint amennyi fizikai lába (PIN).

* [cite_start]**Pin Multiplexing (MUX):** Egy fizikai lábhoz több belső periféria is hozzárendelhető[cite: 9]. Például ugyanaz a láb lehet egy egyszerű kapcsoló bemenete (GPIO), vagy egy soros port adó lába (UART TX), vagy egy PWM kimenet.
* **Funkciója:** A szoftvernek a regiszterekben (Pin Control Register) kell konfigurálnia, hogy az adott láb éppen melyik belső funkcióhoz kapcsolódjon.

---

## 4. GPIO: Általános célú I/O portok (KULCSKONCEPCIÓ)
[cite_start]A GPIO a legegyszerűbb periféria: egy bitnyi információt (alacsony vagy magas szint) tud kiadni vagy fogadni[cite: 10].

[cite_start]**Betartandó alapszabályok illesztésnél[cite: 11, 12]:**
* **Feszültségszintek:** Figyelni kell a logikai szintekre ($V_{IL}, V_{IH}, V_{OL}, V_{OH}$). Ha 3.3V-os a kimenet, nem biztos, hogy meghajt egy 5V-os bemenetet.
* **Áramkorlátok:** Egy láb csak véges áramot tud kiadni (Source) vagy elnyelni (Sink). Ha túlterheljük, a chip tönkremegy.
* **Pull-up / Pull-down:** Bemenet esetén, ha nincs rákötve semmi (lebeg), a logikai szint bizonytalan lesz. Ezért belső ellenállásokkal fix szintre kell húzni a lábat.

---

## 5. A/D és D/A átalakítás (KULCSKONCEPCIÓ)
A fizikai világ analóg, a számítógép digitális. [cite_start]Az átalakítás két fő lépése az időbeli és az amplitúdóbeli diszkretizáció[cite: 14].

* **Mintavételezés ($f_s$):** Időbeli diszkretizáció. [cite_start]A Nyquist-Shannon elv szerint a mintavételi frekvenciának legalább a mérendő jel legnagyobb frekvenciájának kétszeresének kell lennie[cite: 16].
* **Kvantálás:** Amplitúdóbeli diszkretizáció. [cite_start]A felbontást (Resolution) bitekben adjuk meg (pl. 12 bit = 4096 szint)[cite: 15].
* [cite_start]**Mintavételezési módok[cite: 17]:**
    * **On-demand:** A szoftver kéri a mérést, megvárja, amíg kész. (Lassú, de egyszerű).
    * **Időzített:** Hardveres timer indítja a mérést pontos időközönként. (Valós-idejű feldolgozáshoz elengedhetetlen).

Az alábbi szimulátorral kipróbálhatod, hogyan befolyásolja a felbontás (kvantálás) és a mintavételi sűrűség a jel hűségét:

{
  "component": "LlmGeneratedComponent",
  "props": {
    "height": "600px",
    "prompt": "Create an educational ADC (Analog-to-Digital Converter) simulator. Display an original smooth sine wave. Allow the user to adjust two parameters: 'Sampling Frequency' (number of dots along the x-axis) and 'Bit Depth/Resolution' (number of discrete levels on the y-axis, from 1 to 8 bits). Visually represent the 'digitized' signal as a stepped staircase line or dots on the discrete levels. Show the 'Quantization Error' as the difference between the original and digitized signal. Labels and titles should be in Hungarian: 'Eredeti jel', 'Digitális jel', 'Mintavételi frekvencia', 'Felbontás (bit)', 'Kvantálási hiba'. Use a clean layout with controls at the bottom."
  }
}

---

## 6. Digitális kommunikációs protokollok (UART, I2C, SPI)
Ha két digitális alkatrész beszélget, protokollra van szükség. [cite_start]A ZH-ban ezek összehasonlítása kulcskérdés[cite: 21, 24].

| Tulajdonság | UART | I2C | SPI |
| :--- | :--- | :--- | :--- |
| **Szinkronitás** | [cite_start]Aszinkron (nincs órajel) [cite: 21] | [cite_start]Szinkron (SCL órajel) [cite: 22] | [cite_start]Szinkron (SCK órajel) [cite: 23] |
| **Vezetékek száma** | 2 (TX, RX) | 2 (SDA, SCL) | 4 (MOSI, MISO, SCK, CS) |
| **Topológia** | Point-to-point (2 eszköz) | Multi-master / Multi-slave | Master-Slave |
| **Címzés** | Nincs | Cím alapú (7/10 bit) | Chip Select (CS) lábbal |
| **Sebesség** | Alacsony | Közepes (100k - 3.4M) | Magas (akár >50M) |

### Fontos eltérés adatelérés szempontjából (KULCSKONCEPCIÓ)
* **I2C és SPI:** Ezek **regiszterfájl alapúak**. Egy periféria regisztereit érjük el rajtuk keresztül, mintha egy távoli memóriahelyre írnánk/olvasnánk. [cite_start]Az elérhető adatok szerkezete kötött[cite: 24].
* **UART:** Ez egy **stream-alapú** kommunikációs megoldás. [cite_start]Nincs egységes regisztermodell; bármit és bárhogy küldhetünk rajta, a szoftver dolga értelmezni a beérkező bájtok folyamát[cite: 24].

---

## 7. Megszakítás (Interrupt) (KULCSKONCEPCIÓ)
[cite_start]A periféria jelezheti a CPU-nak, ha esemény történt, így a CPU-nak nem kell folyamatosan kérdezgetnie (polling)[cite: 7].

1.  **Esemény:** A periféria jelez.
2.  **Megszakítás:** A CPU felfüggeszti az aktuális munkát.
3.  **ISR (Interrupt Service Routine):** Lefut a szoftveres kezelőfüggvény.
4.  **Visszatérés:** A CPU folytatja ott, ahol abbahagyta.

---
Ez az anyag lefedi a ZH-hoz szükséges összes fontos hardver-közeli fogalmat. Mehetünk tovább a 4. prezentációra (szoftverfejlesztési problémák és operációs rendszerek)?