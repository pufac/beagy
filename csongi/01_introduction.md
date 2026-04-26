Üdvözöllek! **PDFolvaso** oktatósegédként elkészítettem neked a kért részletes jegyzetet a **VIMIAD04** tantárgy első ZH-felkészítő anyaga alapján. A kidolgozás során szigorúan a megadott kulcsszavak sorrendjét követtem, feltüntetve az érintett diaoldalakat és pontos forrásmegjelöléseket.

---

## 1. Beágyazott információs rendszerek alapfogalmai

### **Beágyazott rendszer definíciója**
**10. oldal** 
* **Definíció**: Olyan speciális számítógépes rendszerek, amelyeket egy jól meghatározott feladatra terveztek.
* **Működés**: A feladat ellátása érdekében intenzív információs kapcsolatban állnak a külvilággal: érzékelik a paramétereit (szenzorok), és gyakran be is avatkoznak abba (beavatkozók).
* **Interfészek**:
    * **Gép-környezet interfész**: Szenzorok, beavatkozók, kommunikációs felületek.
    * **Felhasználói felület**: Humán operátor számára.
* **Platform**: Nem zárja ki a PC használatát, de ebben az esetben az dedikált feladatot lát el, akár standard Windows vagy Linux operációs rendszerrel (bár nem erre lettek kitalálva).

### **Valós-idejű rendszer definíciója**
**11-13. oldal** 
* **Szakmai köznyelvben (Reszponzív)**: A rendszer az ember számára "azonnal" reagál.
* **Szigorú (Biztonságkritikus) definíció**: A rendszernek **garantálnia kell** a határidők betartását.
    * Járművek esetén ez jellemzően **20-50 ms**.
    * Az ipari elosztott rendszerekben a megengedett ingadozás (jitter) a **μs** tartományban van.
* **Típusok**:
    * **Hard Real-Time**: A határidő nem teljesítése katasztrofális következményekkel jár (pl. szervokormány, fékrendszer, atomerőmű védelem). A rendszer hibásnak minősül, ha késik.
    * **Soft Real-Time**: A határidők statisztikailag (pl. 99.9%) teljesülnek, a késésnek nincsenek katasztrofális következményei (pl. telekommunikáció, energiaellátás).
* **Analógia**: Az ittas vezetés reakcióideje megnő, ami analóg a valós-idejűség elvesztésével biztonságkritikus környezetben.

### **Valós-idejű operációs rendszer (RTOS) definíciója**
**14. oldal** 
* **Definíció**: Olyan OS, amelynek felépítése lehetővé teszi, hogy szolgáltatásai (pl. HW megszakítás kezelése) meghatározott időintervallumon belül (többnyire néhány μs) lefussanak.
* **Fontos**: Az RTOS csak **lehetővé teszi** a valós-idejűséget, magának az alkalmazásnak is annak kell lennie.
* **Példák**: A standard Linux és Windows nem RTOS, de léteznek kiterjesztéseik (pl. Real-Time Linux patch, IntervalZero RTX64).
* **Beágyazott Linux vs. RT Linux**: Nem minden beágyazott rendszer igényel szigorú valós-idejűséget (pl. okos TV, hálózati eszközök), ahol elég a "best effort" működés.

---

## 2. Kiber-fizikai rendszerek (CPS)

### **CPS definíciója és eltérése az IoT/IIoT-től**
**2, 26. oldal** 
* **CPS Definíció**: Olyan elosztott rendszer, ahol az informatikai rész a begyűjtött adatok alapján beavatkozik a fizikai folyamatokba (szabályozási hurok).
* **3C**: Communicate (Kommunikáció), Compute (Számítás), Control (Vezérlés).
* **Eltérés az IoT-től**: Az IoT/IIoT alkalmazásokban a szabályozási hurkot gyakran még az **ember zárja** (nem merünk automatizálni), míg a CPS célja az automatikus optimalizálás emberi felügyelet nélkül.

### **CPS architektúra (5 szint)**
**29-33. oldal** 
| Szint | Megnevezés | Funkció |
| :--- | :--- | :--- |
| **5.** | **Automatizmusok (Self-X)** | Önkonfigurálás, adaptivitás, önoptimalizálás, döntéshozatal és végrehajtás (Orchestration). |
| **4.** | **Kognitív szint** | Helyzetértékelés, a változások okainak "megértése", döntési javaslatok szoftveres úton. |
| **3.** | **Kiber-fizikai szint** | Digitális iker (Digital Twin) felépítése, környezeti modellek (gráf, diff. egyenlet) és rendszeridentifikáció. |
| **2.** | **Adat-információ konverzió** | Intelligens analízis, adatok újrahasznosítása, szenzorfúzió (pl. GNSS + inerciális szenzorok), historikus tárolás. |
| **1.** | **Intelligens összeköttetés** | Szenzorhálózat, mérések, IT hálózati infrastruktúra, energiafogyasztás mérése. |

> **Példa (Nyitott ablak)**: Az 1. szinten a szenzor méri a hőmérsékletet, a 2. szinten tároljuk az adatot, a 3. szinten anomáliát észlelünk (eltérés a modelltől), a 4. szinten rájövünk, hogy nyitva az ablak, az 5. szinten pedig értesítést küldünk vagy becsukjuk az ablakot .

### **Internet technológiák problémái CPS környezetben**
**35. oldal** 
* **Valós-idejűség**: Az Ethernet/TCP/IP eredetileg "best effort" alapú, nem valós-idejű.
* **Megbízhatóság**: A hibatűrés (útvonalkeresés) másodperceket vehet igénybe, a CPS-ben ms nagyságrendű elvárás van.
* **Vezeték nélküli kommunikáció**: Biztonságkritikus környezetben a zavarérzékenység miatt nem alkalmas, ott a vezetékes megoldás az elvárt.
* **Jövő**: **TSN (Time-Sensitive Networking)** alkalmazása a valós-idejű Ethernet érdekében.

### **IT biztonság, élettartam és minősítés**
**37-39. oldal** 
* **Élettartam-olló**: A CPS rendszerek 10-50 évig működnek, de a biztonsági protokollok (pl. SSL) csak pár évig élnek. A régi hardver nem biztos, hogy bírja az új titkosítási algoritmusokat.
* **Minősítés (Certification)**: Szigorúan szabályozott iparágakban (vasút, repülés) minden szoftverfrissítés (biztonsági javítás is) drága és lassú újraminősítést igényelhet.
* **CIA-hármas a CPS-ben**:
    * **Titkosság (Confidentiality)**: Sokszor kevésbé kritikus, mert az adatok (pl. idő, sebesség) amúgy is érzékelhetőek.
    * **Integritás (Integrity)**: Kritikus, hogy ne lehessen meghamisítani a vezérlőjeleket.
    * **Elérhetőség (Availability)**: A legfontosabb, mert a kiesés (DoS támadás) katasztrófát okozhat.

### **Domén szakértők szerepe**
**40. oldal** 
* **Interdiszciplináris terület**: A mérnökökön kívül szükség van fizikusokra, vegyészekre, orvosokra, pszichológusokra (UI design) és jogi/számviteli szakértőkre is.
* **Feladatuk**: Az alkalmazási terület specifikus előírásainak és fizikai kényszereinek meghatározása.

---

## 3. Szabványok és környezeti hatások

### **ANSI/IEC 60529-2004 (IP védettség)**
**42. oldal** 
Formátum: **IPxyzv**
* **x (Szilárd testek)**: 0-6 közötti szám. Pl. 2: 12.5 mm-es tárgyak (ujj), 6: pormentes (Dust Tight).
* **y (Folyadékok)**: 0-8 közötti szám. Pl. 1: függőlegesen hulló cseppek, 7: rövid idejű merítés (15 cm - 1 m), 8: tartós víz alatti használat.
* **z**: Mechanikai ütés (0-9).
* **v**: Egyéb (pl. "f" - olajálló).

### **MIL-STD-810 (Katonai szabvány)**
**43. oldal** 
* **Alkalmazás**: Bár katonai eredetű (NATO), gyakran használják ipari vagy professzionális eszközök minősítésére is.
* **Cél**: Az eszköznek a Föld bármely pontján üzemszerűen működnie kell a specifikált környezetben.
* **Tesztek**: Több mint 30 speciális teszt (pl. alacsony nyomás, tengervíz-korrózió, vibráció, gomba/baktérium, lőszer-találat).

---

## 4. Szenzorok és beavatkozók

### **Szenzor (Érzékelő)**
**46. oldal** 
* **Definíció**: Fizikai mennyiség elektromos jellé alakítására szolgáló eszköz.
* **Kimenet**:
    * **Analóg**: Feszültség vagy áram (hibák: nemlinearitás, ofszet, zaj).
    * **Digitális**: A/D átalakítás utáni egész szám. Fontos a mintavételezés (Nyquist-Shannon elv) és a kvantálás.
* **Tulajdonságok**: Felbontás (kisebb változás, amit még észlel) és pontosság (a mért érték valódisága).

### **Beavatkozó**
**47. oldal** 
* **Definíció**: Elektromos jelet fizikai mennyiséggé alakít, amit rákényszerít a környezetre.
* **Működés**: Általában erősítő is egyben, mert jelentős teljesítményt igényel a beavatkozás.
* **Típusok**:
    * **Hidraulikus/Pneumatikus** (pl. munkahengerek).
    * **Elektromos/Mágneses** (pl. motorok, relék).
    * **Repülőgép példa**: Slat-ek, szárnysegédlapok (Flap), spoilerek mozgatása.

---
Sok sikert a ZH-ra való felkészüléshez! Ha szükséged van a 2. prezentáció kidolgozására is, csak szólj!