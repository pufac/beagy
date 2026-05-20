Szia! Értem a kérést, a 2. ZH-nál valóban a legkisebb technikai részletekbe is belekérdezhetnek. A **9. előadás (Szenzorhálózatok)** anyagát a kulcsszólista alapján dolgozzuk fel, kifejezetten mély, mérnöki fókusszal.

*(Rövid technikai megjegyzés: A rendszerem szigorú belső szabályai miatt kötelező forrásmegjelöléseket használnom a generált szövegben, így a `` formátumot nem tudom teljesen eltüntetni. Azonban, hogy a GitHubon is jól olvasható maradjon, a hivatkozásokat igyekszem a bekezdések és felsorolások legvégére, diszkréten elhelyezni!)*

Íme a 9. előadás vizsgafókuszú, részletes összefoglalója!

---

# Szenzorhálózatok és Ipari Kommunikáció (9. előadás)

## 1. Hálózat alapvető követelményei beágyazott/CPS rendszerekben

Amikor beágyazott és kiber-fizikai rendszereket (CPS) hálózatba kötünk, a klasszikus irodai (IT) hálózatoktól merőben eltérő elvárásoknak kell megfelelni.

* **Valós-idejűség (Real-Time):** Nem az átlagos sebesség a lényeg, hanem a **garantált felső korlát a késleltetésre** (determinizmus) és az alacsony **Jitter** (a késleltetés ingadozása).
* **Megbízhatóság (Safety):** Extrém környezeti feltételek (zaj, hőmérséklet, vibráció) mellett is garantálni kell a helyes működést vagy a biztonságos leállást (Fail-Safe).
* **Biztonság (Security):** Védelem a szándékos, rosszindulatú külső támadások ellen.
* **Sávszélesség és Skálázhatóság:** Megfelelő adatátviteli kapacitás a növekvő számú szenzorhoz és a hálózat fizikai bővíthetősége.

## 2. Megbízhatóság (Safety) vs. Biztonság (Security)

Ezt a különbséget szinte biztosan kérdezik, nagyon fontos tisztázni!

* **Megbízhatóság (Safety / Reliability):** A rendszer belső hibái (hardver meghibásodás, szoftveres bugok) vagy a véletlenszerű fizikai környezeti hatások (pl. elektromágneses zaj) elleni védekezés. *Célja: Ha a gép elromlik, ne tegyen kárt az emberben vagy a környezetben.*
* **IT Biztonság (Security):** A szándékos, rosszindulatú, külső emberi tényezők (hackerek, vírusok, illetéktelen behatolók) elleni védelem. *Célja: Megvédeni a rendszert attól, hogy valaki szándékosan átvegye felette az irányítást, és így okozzon kárt.*

## 3. Vezetékes vs. Vezeték nélküli hálózatok

A fizikai közeg választása drasztikusan meghatározza a rendszer képességeit:

* **Vezetékes hálózatok (Wired):** Zárt, jobban kontrollálható közeg. Robusztusabb a fizikai zajokkal szemben, nagyobb sávszélességet és stabilabb valós-idejűséget biztosít, ráadásul a tápellátás (PoE) is megoldható ugyanazon a kábelen. Hátránya a drága kiépítés és a rugalmatlanság (mozgó géprészeknél a kábel megtörik).
* **Vezeték nélküli (Wireless):** Maximális mobilitás, könnyű és olcsó telepítés (pl. forgó alkatrészeken is működik). Hátránya az erős kitettség a környezeti zavaroknak (interferencia), az IT biztonsági kockázatok (bárki befoghatja a jelet a levegőből), és a korlátozott, akkumulátoros tápellátás.

## 4. Fieldbus (Terepi busz) definíciója

A Fieldbus egy ipari, elosztott vezérlőrendszerekhez kifejlesztett digitális, kétirányú, soros kommunikációs hálózat.

* **Miért hozták létre?** Korábban minden egyes szenzort és beavatkozót külön analóg kábellel (pl. 4-20 mA áramhurok) kellett a központi PLC-be (vezérlőbe) kötni. A Fieldbus ezt egyetlen, az összes eszközt felfűző digitális kábelre cserélte, ami drasztikusan csökkentette a kábelezési költségeket és lehetővé tette az okos eszközök integrációját.

## 5. Miért nem jó az IT Ethernet az ipari / valós-idejű környezetben?

A szabványos (irodai) Ethernet TCP/IP és UDP hálózat kiváló webezésre, de alkalmatlan kritikus ipari gépek vezérlésére. Ennek okai:

* **Nincs Valós-idejűség ("Best Effort"):** A klasszikus Ethernet CSMA/CD (Carrier Sense Multiple Access with Collision Detection) protokollja vagy a modern Switchek pufferezése (Store-and-Forward) miatt **nincs garantált felső korlát az átviteli időre**. Nagy forgalom esetén a csomagok késnek (nagy jitter) vagy akár el is vesznek.
* **Hatalmas Overhead:** Egy egyszerű motorvezérlő (pl. "fordulatszám: 1500") adata csupán pár bájt. Ezt az IT hálózatokon egy 64-1500 bájtos Ethernet keretbe, plusz IP és TCP/UDP fejlécekbe csomagolják, ami sávszélesség-pazarló és lassú.

---

## 6. A CAN Busz (Controller Area Network) (KIEMELT FÓKUSZ)

A CAN az autóipar és az automatizálás legelterjedtebb hálózata. Ez egy aszinkron, multi-master, broadcast (mindenki hall mindenkit) busz topológia.

### Hogyan működik a csatornahozzáférés? (Bitwise Arbitration)

A CAN nem használ külön címet az eszközökhöz, hanem az üzenetek tartalmát azonosítja (Message ID). A hozzáférést a **CSMA/BA (Carrier Sense Multiple Access with Bitwise Arbitration)** szabályozza:

* A fizikai réteg (kábel) egy logikai "ÉS" (Wired-AND) kapuként működik: a logikai '0' a **Domináns**, a logikai '1' a **Recesszív** bit. Ha az egyik eszköz '0'-t, a másik '1'-et ad ki, a buszon '0' lesz.
* **Az Arbitráció folyamata:** Ha két eszköz egyszerre kezd el adni, mindkettő bitenként visszoolvassa a buszt. Ha egy eszköz recesszív ('1') bitet akar küldeni, de a buszon domináns ('0') bitet érzékel, azonnal tudja, hogy egy magasabb prioritású (kisebb ID-jű) üzenet van a vonalon. Ekkor **azonnal abbahagyja az adást**.
* *Előny:* Nincs ütközés miatti adatvesztés vagy ismétlés, a nyertes üzenet megszakítás nélkül halad tovább!.

### Frame felépítése

Egy standard CAN keret főbb részei:

* **SOF (Start of Frame):** 1 domináns bit.
* **Arbitration Field:** 11 bit (Standard) vagy 29 bit (Extended) Identifier (Azonosító), ami egyben a prioritás is, plusz az RTR (Remote Transmission Request) bit.
* **Control Field:** 6 bit, tartalmazza a DLC-t (Data Length Code), ami megadja az adatbájtok számát.
* **Data Field:** 0-8 bájt tényleges hasznos adat.
* **CRC Field:** 15 bites ellenőrzőösszeg a hibadetektáláshoz.
* **ACK Field:** Nyugtázó bit (a vevők egy domináns bittel felülírják, ha hibátlanul vették).
* **EOF (End of Frame):** 7 recesszív bit.

### Bit-stuffing (Bitbeszúrás) és Jitter

A CAN buszon nincs külön órajel vezeték, az eszközök a jelszintek változásából (élekből) szinkronizálják magukat.

* Ha 5 darab azonos logikai értékű bit (pl. öt '0') szerepelne a hasznos adatban, az adó hardvere **automatikusan beszúr egy ellentétes bitet** (egy '1'-est), amit a vevő automatikusan kivág.
* *Mi a gond vele?* A beszúrt bitek miatt a keret hossza (és így az átviteli idő) az adat tartalmától függően megváltozik (akár 15%-kal hosszabb lehet). **Ez időzítési bizonytalanságot, azaz Jittert okoz a hálózatban!**.

### Bit-sebesség korlát (A buszhossz limitációja)

A CAN sebessége és a busz maximális hossza fordítottan arányos. Az Arbitráció és az ACK nyugtázás csak akkor működik, ha egyetlen bitidő alatt a jel oda-vissza végigér a legmesszebb lévő csomópontig. Emiatt a fénysebesség fizikai korlátot jelent: **1 Mbps sebesség mellett a busz maximális hossza kb. 40 méter lehet**. Ha 1000 méteres kábelt akarunk, a sebességet le kell vinni 50 kbps-re.

### Hiba detektálás (Error Handling)

A CAN busz zsenialitása a nagyon robusztus hibakezelés. 5 szinten történik ellenőrzés:

1. **Bit monitoring:** Az adó visszoolvassa, amit a buszra tett. Ha '1'-et tesz ki, de '0'-t olvas vissza (és épp nincs arbitráció), bit hibát jelez.
2. **Bit stuffing hiba:** Ha egymás után 6 azonos bitet lát a buszon.
3. **CRC hiba:** A kapott adat CRC ellenőrzőösszege nem stimmel.
4. **Frame check hiba:** Hibás jelszintek a fix mezőkben (pl. EOF nem recesszív).
5. **ACK hiba:** Egyetlen vevő sem nyugtázta a keretet.
*(Ha hiba van, a felismerő azonnal Error Flag-et küld, ami leállítja az adást, és kikényszeríti az automatikus újraküldést.)*

---

## 7. EtherCAT (Ethernet for Control Automation Technology) (KIEMELT FÓKUSZ)

Mivel az IT Ethernet alkalmatlan volt ipari célokra, a Beckhoff kifejlesztette az EtherCAT-et, ami szabványos Ethernet (IEEE 802.3) fizikai réteget és kereteket használ, de brutális, mikroszekundumos valós-idejűséggel.

### Alapötlet és a "Processing on the fly"

Az EtherCAT logikailag egy gyűrű (ring) topológia, szigorú Master-Slave architektúrával.

* Hagyományos Ethernet esetén a keret bemegy a memóriába, a szoftveres stack feldolgozza, kinyeri az adatot, majd generál egy új keretet a válasznak. Ez nagyon lassú (milliszekundumok).
* **Processing on the fly (Röptében történő feldolgozás):** Az EtherCAT Master kiküld egyetlen hatalmas Ethernet keretet, amiben minden egyes Slave (motor, szenzor) adata benne van. Ahogy a keret elektromos jele áthalad a Slave eszközön lévő speciális hardveren (ASIC / FPGA), a Slave hardveresen, **nanoszekundumok alatt** kiolvassa a neki szóló adatbájtokat, és azon nyomban, ahogy a keret halad tovább, bele is írja (beszúrja) a saját mérési eredményeit a dedikált helyére. A keret folyamatosan halad tovább az állomásokon, késleltetés nélkül, majd a gyűrű végén visszatér a Masterhez.

### Miért extrém gyors és szinkronizált?

* **Hatékonyság (Payload):** Nincs IP és TCP fejléc. Egy Ethernet keretben (max 1500 bájt) több tucat eszköz adata küldhető el egyszerre, így a hasznos adat aránya (bandwidth utilization) 90% feletti.
* **Szinkronizáció (Distributed Clocks - DC):** Többtengelyes robotkaroknál a mikroszekundumos szinkronizáció elengedhetetlen. Az EtherCAT hálózatban az első DC-képes Slave lesz a referenciaóra (Reference Clock). A rendszer a hálózat felállásakor megméri az eszközök közötti vezetékeken a jel terjedési késleltetését (propagation delay). Ennek ismeretében a hardver kompenzálja a késéseket, így **a hálózat összes eszközének belső órája szigorúan < 1 mikroszekundum pontossággal** együtt fog járni.

---

Ezzel is megvagyunk! Ha minden világos a CAN busz arbitrációjával, a bitbeszúrás okozta jitterrel és az EtherCAT "repülés közbeni" feldolgozásával kapcsolatban, jöhet a 11-12. előadás a Time-Sensitive Networking (TSN) hálózatokról!