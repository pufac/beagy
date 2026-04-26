Örömmel segítek a vizsgafelkészülésben! Az alábbiakban a megadott diasor alapján, szigorúan az általad kért módszertant és sorrendet követve kidolgoztam a kiemelt fogalmakat.

### 1. Beágyazott rendszer definíciója
**A 9. és 10. dia alapján:**
* **Definíció:** Olyan speciális számítógépes rendszerek, amelyeket egy jól meghatározott feladatra találtak ki. A teljes rendszernek csak egy részét képezik.
* **Kapcsolat a külvilággal:** A feladat ellátása érdekében intenzív információs kapcsolatban állnak a fizikai környezetükkel: érzékelik annak paramétereit, és gyakran be is avatkoznak abba.
* **Interfészek:** A gép és a beágyazó fizikai környezet közötti kapcsolatot **szenzorok, beavatkozók és kommunikációs felületek** biztosítják, emellett jelen lehet humán operátori (felhasználói) felület is.
* **Hardver és Operációs Rendszer:** Egy beágyazott rendszerben akár PC-t, standard Windowst vagy Linuxot is lehet használni. A kulcs, hogy ilyenkor is legalább részben dedikált feladata van a rendszernek egy korlátozott alkalmazási körben (még ha ezen operációs rendszerek eredetileg nem is erre lettek kitalálva).

### 2. Valós-idejű rendszer definíciója
**A 11., 12. és 13. dia alapján:**
* **Köznyelvi vs. Szakmai jelentés:** A köznyelvben a valós-idejűség azt jelenti, hogy valami "azonnal" megtörténik (ezeket pontosabb **reszponzív** rendszernek nevezni). A biztonságkritikus szakmai definíció azonban sokkal szigorúbb: **Garantálni KELL a határidőket**.
* **Jellemzők:** A határidők járművek esetén pl. 20-50 ms-osak, az iparban még kisebbek is lehetnek. Egy elosztott rendszerben a valós határidők megengedett ingadozása (jittere) mikro-szekundumos ($\mu s$) tartományban mozog. A határidő lekésése megnöveli a reakcióidőt, és növeli a hibás döntések valószínűségét (az oktató analógiája: befolyásolt állapotban történő vezetés).
* **Kategóriák:**
    * **Hard Real-Time:** Mindig teljesülnie kell a határidőknek. Ha nem teljesül, a rendszer hibás, ami katasztrofális következményekkel jár (pl. szervokormány, atomerőmű). Megvalósítása drága.
    * **Soft Real-Time:** A határidők statisztikailag (pl. 99%, 99.9%) teljesülnek. Olcsóbb megvalósítani, a határidő lekésése nem okoz katasztrófát (pl. telekommunikáció).

### 3. Valós-idejű operációs rendszer definíciója
**A 14. dia alapján:**
* **Definíció:** Olyan operációs rendszer (OS), amelynek felépítéséből következően a megadott szolgáltatásai képesek valós időben működni. Például garantálja, hogy egy HW megszakítás után a hozzá tartozó kód adott időn belül (pár $\mu s$) lefut.
* **Korlátai:** Maga a valós-idejű OS csak *lehetővé teszi* a valós idejű működést, de a rajta futtatott alkalmazásnak is valós-idejűnek kell lennie.
* **Asztali rendszerek (Linux/Windows):** Alapértelmezetten nem adnak ilyen garanciát. Azonban léteznek kiterjesztések (patch-ek), például a Real-Time Linux vagy Windowshoz az IntervalZero RTX64, amik alkalmassá teszik őket erre. Fontos: A "Beágyazott Linux" nem egyenlő a Real-Time Linux-szal, sok IoT/szórakoztatóelektronikai alkalmazásnál elegendő a sima "best effort" működés is.

### 4. Kiber-fizikai rendszer (CPS) definíciója
**A 26. dia alapján:**
* **Definíció:** Olyan elosztott, kiterjedt informatikai és a hozzá kapcsolódó fizikai rendszer, amelyben az informatikai rendszerrész a begyűjtött információk alapján a fizikai rendszer működésébe **beavatkozik**. 
* **Kulcsfogalmak:** A fizikai rendszert "fizikai beágyazó környezetnek" is hívják. Angolul a 3C szabály írja le a legjobban: **Communicate, Compute, Control**.
* **Célja:** A rendszer működésének automatikus optimalizálása (emberi felügyelet nélkül működjön jobban, olcsóbban), erőforrás-gazdálkodás, valamint a "nem várt" eseményekre történő reagálás képessége (Emergent system).

### 5. Miben tér el az IoT/IIoT és a CPS?
**A 26. dia alapján:**
* **IoT/IIoT esetén:** Bár a rendszerek gyűjtenek adatot, az IoT/IIoT alkalmazásokban általában kerüljük az automatikus beavatkozást, mert félünk a nem várt viselkedéstől. Ezeknél a **szabályzó hurkot az ember zárja** (az ember dönt a kapott adatok alapján).
* **CPS esetén:** Van merszünk a rendszerre bízni az irányítást. A CPS automatikusan beavatkozik és optimalizál, akár emberi felügyelet nélkül (pl. zárt visszacsatolásos hurkok alkalmazásával). 

### 6. CPS rendszerek architektúrája és az egyes szintek alapvető funkciói
**A 29-33. diák alapján:**
A CPS rendszerek egy öt szintű hierarchiára bonthatók:

1.  **Intelligens összeköttetés szint (Smart Interconnect Level - 29. dia):** Szenzorok és beavatkozók hálózata, amely a fizikai környezetből méréseket végez és beavatkozik. Ide tartozik az IT hálózat is. A rendszer saját működéséről is gyűjt adatokat (komponensek kihasználtsága, hálózati hibák, energiafogyasztás).
2.  **Adat-Információ konverziós szint (Data to Information Level - 30. dia):** Intelligens analízis. Az adatokból kinyerik az információt alkalmazás specifikusan. Egyetlen szenzorból (pl. kamera) többféle adatot generálnak (vonatsebesség, gyalogos a sínnél). Itt történik az adatok tömörítése (pl. RRD - round-robin adatbázis) és a szenzorfúzió (pl. GNSS + gyorsulásmérő a jobb pozícióért) a megbízhatóság növelése érdekében.
3.  **Kiber-fizikai szint (Cyber-Physical Level - 31. dia):** A fizikai környezet és a rendszer együttes modelljének (Digitális Iker / Digital Twin) felépítése és ellenőrzése az információk alapján. Itt zajlik a "rendszeridentifikáció" (paraméterek, gráf modellek meghatározása) klasszikus és mesterséges intelligencia (MI) algoritmusokkal.
4.  **Kognitív szint (Cognition Level - 32. dia):** Helyzetértékelés. A rendszer "megérti" a működését a modellek alapján, felismeri az anomáliákat és azok okait, és megvizsgálja, mit lehetne tenni az optimálisabb működésért (kitalálja a változás okait).
5.  **Automatizmusok (Self-X Level - 33. dia):** Végrehajtás. Ide tartozik az önkonfigurálás, az adaptivitás és az önoptimalizálás. A rendszer döntéseket hoz és hajt végre (Orchestration - konfiguráció, koordináció automatizálása).

### 7. Internet technológiák problémái CPS környezetben, valós-idejűség, megbízhatóság, IT biztonság
**A 35. és 36. dia alapján:**
* **Valós-idejűség és Megbízhatóság:** A jelenlegi Ethernet és TCP/IP technológiák "best effort" elven működnek, azaz nem valós-idejűek. Bár hibatűrők, a hálózati útvonalválasztás javítása (másodpercek) túl lassú a biztonságkritikus CPS-eknek (ahol ms-os reakció kell).
* **Vezeték nélküli technológiák:** A vezeték nélküli rendszerek fokozott zavarérzékenységük miatt nem felelnek meg a szigorú biztonságkritikus követelményeknek, ezért oda vezetékes hálózat kell. Jellemző probléma a GNSS (pl. GPS) zavarás (jamming) és spoofing.
* **Jövő (TSN):** A megoldás a TSN (Time-Sensitive Networking), ami valós idejű és hibatűrő Ethernet protokollokat biztosít.
* **IT Biztonság:** Jelenleg protokollokkal elérhető a biztonság (VPN, SSL), de ezek ideiglenesek, folyamatos frissítést igényelnek.

### 8. IT biztonság és a CPS viszonya, élettartam, minősítés problémái
**A 37., 38. és 39. dia alapján:**
* **Élettartam probléma:** A CPS rendszerek élettartama évtizedekben mérhető (10-30-50 év). Az IT biztonsági protokollok azonban gyorsan (néhány évente) elavulnak. A meglévő régi hardver gyakran nem bírja futtatni az új titkosítási algoritmusokat (nincs elég teljesítmény).
* **Minősítés problémája:** Biztonságkritikus ágazatokban (vasút, repülés) szigorú külső minősítés szükséges. Ha egy biztonsági protokollt frissíteni kell egy sérülékenység miatt, a teljes rendszert újra kell minősíteni, ami rendkívül drága és időigényes.
* **Biztonsági megközelítés (CIA triász CPS fókusszal):** Míg a klasszikus IT az adatok titkosságára fókuszál, a CPS-ben sokszor az adat nem titkos (pl. aktuális hőmérséklet). Itt sokkal kritikusabb az **Adat-integritás** és az **Adat-elérhetőség** (valós-idejűség). *Ha egy rendszer nem real-time, akkor nem is biztonságos!* Egy DoS támadás vagy egy autóbuszon manipulált adat (integritás sérülés) emberéletekbe kerülhet.

### 9. Domén/alkalmazás szakértők szerepe a CPS-ben
**A 29. és 40. dia alapján:**
* **Szerepkör:** Mivel a szenzorok a fizikai, kémiai, és biológiai világra csatlakoznak, interdiszciplináris tudás szükséges. A CPS-t nem tudják tisztán informatikusok megtervezni; villamosmérnökök, gépészmérnökök, orvosok, vegyészek szaktudása is kell.
* **Kihívás:** A domén szakértők az "Intelligens összeköttetés" szinten kapcsolódnak be a tervezésbe. A fő kihívás, hogy "más nyelvet beszélnek, más a világképük", mint az IT szakembereknek.
* **Egyéb szakértők:** Szükség van továbbá humán területek szakértőire (pszichológus, UX designer) és politikai/adminisztratív szakemberekre is (jogi és számviteli megfelelés, GDPR kérdések).

### 10. ANSI/IEC 60529-2004 szabvány alapjai és alkalmazási köre
**A 42. dia alapján:**
* **Alapok:** Ipari környezeti szabvány, amely az eszközök burkolata által biztosított fizikai védelmi szinteket (IP Code - Ingress Protection) írja le.
* **Formátum:** **IPxyzv**, ahol az egyes karakterek jelentése:
    * **x (Szilárd test elleni védelem):** 0-6 közötti érték (pl. az IP2X érték 12.5 mm-nél nagyobb tárgyaktól, például egy ujj behatolásától véd).
    * **y (Folyadék behatolása elleni védelem):** 0-8 közötti érték (Létezik egy 9K német kiegészítés nagynyomású, forró vizes mosás ellen: 100 bar, 80°C).
    * **z (Mechanikai ütés elleni védelem):** 0-9 közötti érték.
    * **v (Egyéb tulajdonságok):** Betűkódok, pl. az "f" olajtól védett kialakítást jelent.

### 11. MIL-STD-810 szabvány alapjai és alkalmazási köre
**A 43. dia alapján:**
* **Alapok:** Katonai szabvány ("Environmental Engineering Considerations and Laboratory Tests"), melyet a NATO (így hazánk is) használ, bár olykor ipari/fogyasztói eszközöket is minősítenek a tesztjeivel.
* **Cél:** Azt garantálja, hogy az eszköz a Föld bármely pontján, bármikor üzemszerűen használható a specifikált alkalmazási környezetben.
* **Működése (alkalmazási köre):** Több mint 30 speciális laboratóriumi tesztet ír le (pl. kis légnyomás repülőn, tengeri sós környezet okozta korrózió, robbanások/telitalálat tesztelése, gombák, sugárzás, vibráció). Fontos, hogy nem az összes tesztet kell teljesíteni, hanem alkalmazás-specifikusan választják ki a szükségeseket. Megfeleléshez sokszor speciális minősített dobozok, csatlakozók kellenek.

### 12. Szenzor definíciója és alapvető működése és tulajdonságai
**A 46. dia alapján:**
* **Definíció:** Fizikai mennyiség elektromos mérésére szolgáló eszköz, azaz egy *jelátalakító*.
* **Működése:** * **Bemenet:** A beágyazó környezet egy fizikai mennyisége (pl. ellenállás, hő) alapján működik. Mérése során bizonyos mértékben befolyásolja a környezetet (például energiát von el tőle a működéshez).
    * **Kimenet:** A kimenete egy analóg elektromos jel (feszültség vagy áram).
    * **Digitalizálás:** Mivel a számítógép digitális adatokkal dolgozik, a kimenő jelet digitálissá kell alakítani (A/D konverzió). Ezt **mintavételezéssel** és **kvantálással** érik el, melynek eredménye tipikusan egy előjel nélküli egész szám lesz minden mintavételi időpillanatban.
* **Tulajdonságai és hibái:** A kimenetet terhelheti offszet, nemlinearitás, illetve befolyásolhatja a frekvenciamenet. Két nagyon fontos paraméterét meg kell különböztetni: a **felbontást** (legkisebb mérhető egység) és a **pontosságot** (a mért és valós érték közötti eltérés). (Példa: egy percmutatós óra felbontása 1 perc, de pontossága akár több perc is lehet, ha siet) .

### 13. Beavatkozó definíciója és alapvető működése és tulajdonságai
**A 47. dia alapján:**
* **Definíció:** Elektromos jeleket fizikai mennyiséggé átalakító eszköz. Szintén jelátalakító, de ez a kapott jelet (parancsot) rákényszeríti a beágyazó fizikai környezetre.
* **Működése:** Mivel a környezet manipulálásához sokszor jelentős erő kell, a beavatkozó jellemzően **erősítő is egyben** (mert a vezérlő elektronika önmagában nem bír akkora teljesítményt leadni). A számítógépes vezérlőjelet vissza kell alakítani (D/A konverzió - digitálisból analóg), ami fordítottan végzi a mintavételezést és kvantálást.
* **Típusai (tulajdonságok):** Kialakításuk szerint lehetnek hidraulikus, pneumatikus, vagy elektromos (mágneses pl. relé, motor) rendszerek. A repülőgépek szárnya is ilyen beavatkozókkal (Slats, Flap, Spoiler) van felszerelve.