Örömmel segítek a vizsgafelkészülésben! Az alábbiakban a megadott diasor alapján, szigorúan az általad kért módszertant és sorrendet követve kidolgoztam a kiemelt fogalmakat.



\### 1. Beágyazott rendszer definíciója

\*\*A 9. és 10. dia alapján:\*\*

\* \[cite\_start]\*\*Definíció:\*\* Olyan speciális számítógépes rendszerek, amelyeket egy jól meghatározott feladatra találtak ki\[cite: 1452]. \[cite\_start]A teljes rendszernek csak egy részét képezik\[cite: 1446].

\* \[cite\_start]\*\*Kapcsolat a külvilággal:\*\* A feladat ellátása érdekében intenzív információs kapcsolatban állnak a fizikai környezetükkel: érzékelik annak paramétereit, és gyakran be is avatkoznak abba\[cite: 1453, 1454].

\* \[cite\_start]\*\*Interfészek:\*\* A gép és a beágyazó fizikai környezet közötti kapcsolatot \*\*szenzorok, beavatkozók és kommunikációs felületek\*\* biztosítják, emellett jelen lehet humán operátori (felhasználói) felület is\[cite: 1455].

\* \[cite\_start]\*\*Hardver és Operációs Rendszer:\*\* Egy beágyazott rendszerben akár PC-t, standard Windowst vagy Linuxot is lehet használni\[cite: 1456, 1458]. \[cite\_start]A kulcs, hogy ilyenkor is legalább részben dedikált feladata van a rendszernek egy korlátozott alkalmazási körben (még ha ezen operációs rendszerek eredetileg nem is erre lettek kitalálva)\[cite: 1457, 1459, 1460].



\### 2. Valós-idejű rendszer definíciója

\*\*A 11., 12. és 13. dia alapján:\*\*

\* \[cite\_start]\*\*Köznyelvi vs. Szakmai jelentés:\*\* A köznyelvben a valós-idejűség azt jelenti, hogy valami "azonnal" megtörténik (ezeket pontosabb \*\*reszponzív\*\* rendszernek nevezni)\[cite: 1466, 1468, 1469, 1470]. \[cite\_start]A biztonságkritikus szakmai definíció azonban sokkal szigorúbb: \*\*Garantálni KELL a határidőket\*\*\[cite: 1480, 1482].

\* \*\*Jellemzők:\*\* A határidők járművek esetén pl. \[cite\_start]20-50 ms-osak, az iparban még kisebbek is lehetnek\[cite: 1483, 1484]. \[cite\_start]Egy elosztott rendszerben a valós határidők megengedett ingadozása (jittere) mikro-szekundumos ($\\mu s$) tartományban mozog\[cite: 1503, 1504]. \[cite\_start]A határidő lekésése megnöveli a reakcióidőt, és növeli a hibás döntések valószínűségét (az oktató analógiája: befolyásolt állapotban történő vezetés)\[cite: 1486, 1487, 1488].

\* \[cite\_start]\*\*Kategóriák\[cite: 1515]:\*\*

&#x20;   \* \[cite\_start]\*\*Hard Real-Time:\*\* Mindig teljesülnie kell a határidőknek\[cite: 1505]. \[cite\_start]Ha nem teljesül, a rendszer hibás, ami katasztrofális következményekkel jár (pl. szervokormány, atomerőmű)\[cite: 1505, 1508]. \[cite\_start]Megvalósítása drága\[cite: 1506].

&#x20;   \* \[cite\_start]\*\*Soft Real-Time:\*\* A határidők statisztikailag (pl. 99%, 99.9%) teljesülnek\[cite: 1509]. \[cite\_start]Olcsóbb megvalósítani, a határidő lekésése nem okoz katasztrófát (pl. telekommunikáció)\[cite: 1511, 1512].



\### 3. Valós-idejű operációs rendszer definíciója

\*\*A 14. dia alapján:\*\*

\* \[cite\_start]\*\*Definíció:\*\* Olyan operációs rendszer (OS), amelynek felépítéséből következően a megadott szolgáltatásai képesek valós időben működni\[cite: 1528]. \[cite\_start]Például garantálja, hogy egy HW megszakítás után a hozzá tartozó kód adott időn belül (pár $\\mu s$) lefut\[cite: 1529, 1530].

\* \[cite\_start]\*\*Korlátai:\*\* Maga a valós-idejű OS csak \*lehetővé teszi\* a valós idejű működést, de a rajta futtatott alkalmazásnak is valós-idejűnek kell lennie\[cite: 1531, 1532].

\* \[cite\_start]\*\*Asztali rendszerek (Linux/Windows):\*\* Alapértelmezetten nem adnak ilyen garanciát\[cite: 1533]. \[cite\_start]Azonban léteznek kiterjesztések (patch-ek), például a Real-Time Linux vagy Windowshoz az IntervalZero RTX64, amik alkalmassá teszik őket erre\[cite: 1534, 1535, 1536]. \[cite\_start]Fontos: A "Beágyazott Linux" nem egyenlő a Real-Time Linux-szal, sok IoT/szórakoztatóelektronikai alkalmazásnál elegendő a sima "best effort" működés is\[cite: 1537, 1538, 1540, 1541].



\### 4. Kiber-fizikai rendszer (CPS) definíciója

\*\*A 26. dia alapján:\*\*

\* \[cite\_start]\*\*Definíció:\*\* Olyan elosztott, kiterjedt informatikai és a hozzá kapcsolódó fizikai rendszer, amelyben az informatikai rendszerrész a begyűjtött információk alapján a fizikai rendszer működésébe \*\*beavatkozik\*\*\[cite: 1797, 1798]. 

\* \[cite\_start]\*\*Kulcsfogalmak:\*\* A fizikai rendszert "fizikai beágyazó környezetnek" is hívják\[cite: 1799]. \[cite\_start]Angolul a 3C szabály írja le a legjobban: \*\*Communicate, Compute, Control\*\*\[cite: 1807].

\* \[cite\_start]\*\*Célja:\*\* A rendszer működésének automatikus optimalizálása (emberi felügyelet nélkül működjön jobban, olcsóbban), erőforrás-gazdálkodás, valamint a "nem várt" eseményekre történő reagálás képessége (Emergent system)\[cite: 1802, 1803, 1804, 1806].



\### 5. Miben tér el az IoT/IIoT és a CPS?

\*\*A 26. dia alapján:\*\*

\* \*\*IoT/IIoT esetén:\*\* Bár a rendszerek gyűjtenek adatot, az IoT/IIoT alkalmazásokban általában kerüljük az automatikus beavatkozást, mert félünk a nem várt viselkedéstől. \[cite\_start]Ezeknél a \*\*szabályzó hurkot az ember zárja\*\* (az ember dönt a kapott adatok alapján)\[cite: 1800].

\* \*\*CPS esetén:\*\* Van merszünk a rendszerre bízni az irányítást. \[cite\_start]A CPS automatikusan beavatkozik és optimalizál, akár emberi felügyelet nélkül (pl. zárt visszacsatolásos hurkok alkalmazásával)\[cite: 1798, 1802, 1803]. 



\### 6. CPS rendszerek architektúrája és az egyes szintek alapvető funkciói

\*\*A 29-33. diák alapján:\*\*

\[cite\_start]A CPS rendszerek egy öt szintű hierarchiára bonthatók\[cite: 1895, 1896, 1897, 1898, 1899]:



1\.  \[cite\_start]\*\*Intelligens összeköttetés szint (Smart Interconnect Level - 29. dia):\*\* Szenzorok és beavatkozók hálózata, amely a fizikai környezetből méréseket végez és beavatkozik\[cite: 1880, 1881, 1882, 1883]. Ide tartozik az IT hálózat is. \[cite\_start]A rendszer saját működéséről is gyűjt adatokat (komponensek kihasználtsága, hálózati hibák, energiafogyasztás)\[cite: 1884, 1885, 1889, 1894].

2\.  \*\*Adat-Információ konverziós szint (Data to Information Level - 30. dia):\*\* Intelligens analízis. \[cite\_start]Az adatokból kinyerik az információt alkalmazás specifikusan\[cite: 1907, 1908]. \[cite\_start]Egyetlen szenzorból (pl. kamera) többféle adatot generálnak (vonatsebesség, gyalogos a sínnél)\[cite: 1909, 1910, 1912]. \[cite\_start]Itt történik az adatok tömörítése (pl. RRD - round-robin adatbázis) és a szenzorfúzió (pl. GNSS + gyorsulásmérő a jobb pozícióért) a megbízhatóság növelése érdekében\[cite: 1917, 1918, 1924].

3\.  \[cite\_start]\*\*Kiber-fizikai szint (Cyber-Physical Level - 31. dia):\*\* A fizikai környezet és a rendszer együttes modelljének (Digitális Iker / Digital Twin) felépítése és ellenőrzése az információk alapján\[cite: 1937, 1938, 1939, 1942]. \[cite\_start]Itt zajlik a "rendszeridentifikáció" (paraméterek, gráf modellek meghatározása) klasszikus és mesterséges intelligencia (MI) algoritmusokkal\[cite: 1940, 1941, 1943, 1944].

4\.  \*\*Kognitív szint (Cognition Level - 32. dia):\*\* Helyzetértékelés. \[cite\_start]A rendszer "megérti" a működését a modellek alapján, felismeri az anomáliákat és azok okait, és megvizsgálja, mit lehetne tenni az optimálisabb működésért (kitalálja a változás okait)\[cite: 1964, 1965, 1967, 1968].

5\.  \*\*Automatizmusok (Self-X Level - 33. dia):\*\* Végrehajtás. \[cite\_start]Ide tartozik az önkonfigurálás, az adaptivitás és az önoptimalizálás\[cite: 1975, 1977, 1978, 1979]. \[cite\_start]A rendszer döntéseket hoz és hajt végre (Orchestration - konfiguráció, koordináció automatizálása)\[cite: 1980, 1981, 1982].



\### 7. Internet technológiák problémái CPS környezetben, valós-idejűség, megbízhatóság, IT biztonság

\*\*A 35. és 36. dia alapján:\*\*

\* \[cite\_start]\*\*Valós-idejűség és Megbízhatóság:\*\* A jelenlegi Ethernet és TCP/IP technológiák "best effort" elven működnek, azaz nem valós-idejűek\[cite: 2052, 2053]. \[cite\_start]Bár hibatűrők, a hálózati útvonalválasztás javítása (másodpercek) túl lassú a biztonságkritikus CPS-eknek (ahol ms-os reakció kell)\[cite: 2054, 2055].

\* \[cite\_start]\*\*Vezeték nélküli technológiák:\*\* A vezeték nélküli rendszerek fokozott zavarérzékenységük miatt nem felelnek meg a szigorú biztonságkritikus követelményeknek, ezért oda vezetékes hálózat kell\[cite: 2061, 2062]. \[cite\_start]Jellemző probléma a GNSS (pl. GPS) zavarás (jamming) és spoofing\[cite: 2065, 2072].

\* \[cite\_start]\*\*Jövő (TSN):\*\* A megoldás a TSN (Time-Sensitive Networking), ami valós idejű és hibatűrő Ethernet protokollokat biztosít\[cite: 2059, 2060].

\* \[cite\_start]\*\*IT Biztonság:\*\* Jelenleg protokollokkal elérhető a biztonság (VPN, SSL), de ezek ideiglenesek, folyamatos frissítést igényelnek\[cite: 2056, 2057].



\### 8. IT biztonság és a CPS viszonya, élettartam, minősítés problémái

\*\*A 37., 38. és 39. dia alapján:\*\*

\* \[cite\_start]\*\*Élettartam probléma:\*\* A CPS rendszerek élettartama évtizedekben mérhető (10-30-50 év)\[cite: 2162]. \[cite\_start]Az IT biztonsági protokollok azonban gyorsan (néhány évente) elavulnak\[cite: 2163, 2164]. \[cite\_start]A meglévő régi hardver gyakran nem bírja futtatni az új titkosítási algoritmusokat (nincs elég teljesítmény)\[cite: 2168, 2169].

\* \[cite\_start]\*\*Minősítés problémája:\*\* Biztonságkritikus ágazatokban (vasút, repülés) szigorú külső minősítés szükséges\[cite: 2170, 2171, 2172]. \[cite\_start]Ha egy biztonsági protokollt frissíteni kell egy sérülékenység miatt, a teljes rendszert újra kell minősíteni, ami rendkívül drága és időigényes\[cite: 2173, 2174, 2175].

\* \[cite\_start]\*\*Biztonsági megközelítés (CIA triász CPS fókusszal):\*\* Míg a klasszikus IT az adatok titkosságára fókuszál, a CPS-ben sokszor az adat nem titkos (pl. aktuális hőmérséklet)\[cite: 2186, 2196, 2198, 2199]. Itt sokkal kritikusabb az \*\*Adat-integritás\*\* és az \*\*Adat-elérhetőség\*\* (valós-idejűség). \[cite\_start]\*Ha egy rendszer nem real-time, akkor nem is biztonságos!\* \[cite: 2188, 2189, 2191] \[cite\_start]Egy DoS támadás vagy egy autóbuszon manipulált adat (integritás sérülés) emberéletekbe kerülhet\[cite: 2194, 2195].



\### 9. Domén/alkalmazás szakértők szerepe a CPS-ben

\*\*A 29. és 40. dia alapján:\*\*

\* \[cite\_start]\*\*Szerepkör:\*\* Mivel a szenzorok a fizikai, kémiai, és biológiai világra csatlakoznak, interdiszciplináris tudás szükséges\[cite: 2207, 2208, 2209]. \[cite\_start]A CPS-t nem tudják tisztán informatikusok megtervezni; villamosmérnökök, gépészmérnökök, orvosok, vegyészek szaktudása is kell\[cite: 2211, 2212].

\* \[cite\_start]\*\*Kihívás:\*\* A domén szakértők az "Intelligens összeköttetés" szinten kapcsolódnak be a tervezésbe\[cite: 1880, 1886]. \[cite\_start]A fő kihívás, hogy "más nyelvet beszélnek, más a világképük", mint az IT szakembereknek\[cite: 1888].

\* \[cite\_start]\*\*Egyéb szakértők:\*\* Szükség van továbbá humán területek szakértőire (pszichológus, UX designer) és politikai/adminisztratív szakemberekre is (jogi és számviteli megfelelés, GDPR kérdések)\[cite: 2216, 2217, 2218, 2222, 2224].



\### 10. ANSI/IEC 60529-2004 szabvány alapjai és alkalmazási köre

\*\*A 42. dia alapján:\*\*

\* \[cite\_start]\*\*Alapok:\*\* Ipari környezeti szabvány, amely az eszközök burkolata által biztosított fizikai védelmi szinteket (IP Code - Ingress Protection) írja le\[cite: 2256].

\* \*\*Formátum:\*\* \*\*IPxyzv\*\*, ahol az egyes karakterek jelentése:

&#x20;   \* \[cite\_start]\*\*x (Szilárd test elleni védelem):\*\* 0-6 közötti érték (pl. az IP2X érték 12.5 mm-nél nagyobb tárgyaktól, például egy ujj behatolásától véd)\[cite: 2257, 2258].

&#x20;   \* \[cite\_start]\*\*y (Folyadék behatolása elleni védelem):\*\* 0-8 közötti érték (Létezik egy 9K német kiegészítés nagynyomású, forró vizes mosás ellen: 100 bar, 80°C)\[cite: 2259, 2260, 2261].

&#x20;   \* \[cite\_start]\*\*z (Mechanikai ütés elleni védelem):\*\* 0-9 közötti érték\[cite: 2262, 2263].

&#x20;   \* \[cite\_start]\*\*v (Egyéb tulajdonságok):\*\* Betűkódok, pl. az "f" olajtól védett kialakítást jelent\[cite: 2264].



\### 11. MIL-STD-810 szabvány alapjai és alkalmazási köre

\*\*A 43. dia alapján:\*\*

\* \[cite\_start]\*\*Alapok:\*\* Katonai szabvány ("Environmental Engineering Considerations and Laboratory Tests"), melyet a NATO (így hazánk is) használ, bár olykor ipari/fogyasztói eszközöket is minősítenek a tesztjeivel\[cite: 2303, 2304, 2305].

\* \[cite\_start]\*\*Cél:\*\* Azt garantálja, hogy az eszköz a Föld bármely pontján, bármikor üzemszerűen használható a specifikált alkalmazási környezetben\[cite: 2307, 2308].

\* \[cite\_start]\*\*Működése (alkalmazási köre):\*\* Több mint 30 speciális laboratóriumi tesztet ír le (pl. kis légnyomás repülőn, tengeri sós környezet okozta korrózió, robbanások/telitalálat tesztelése, gombák, sugárzás, vibráció)\[cite: 2310]. \[cite\_start]Fontos, hogy nem az összes tesztet kell teljesíteni, hanem alkalmazás-specifikusan választják ki a szükségeseket\[cite: 2311]. \[cite\_start]Megfeleléshez sokszor speciális minősített dobozok, csatlakozók kellenek\[cite: 2312].



\### 12. Szenzor definíciója és alapvető működése és tulajdonságai

\*\*A 46. dia alapján:\*\*

\* \[cite\_start]\*\*Definíció:\*\* Fizikai mennyiség elektromos mérésére szolgáló eszköz, azaz egy \*jelátalakító\*\[cite: 2373].

\* \[cite\_start]\*\*Működése:\*\* \* \*\*Bemenet:\*\* A beágyazó környezet egy fizikai mennyisége (pl. ellenállás, hő) alapján működik\[cite: 2373, 2374, 2376]. \[cite\_start]Mérése során bizonyos mértékben befolyásolja a környezetet (például energiát von el tőle a működéshez)\[cite: 2377].

&#x20;   \* \[cite\_start]\*\*Kimenet:\*\* A kimenete egy analóg elektromos jel (feszültség vagy áram)\[cite: 2383].

&#x20;   \* \*\*Digitalizálás:\*\* Mivel a számítógép digitális adatokkal dolgozik, a kimenő jelet digitálissá kell alakítani (A/D konverzió). \[cite\_start]Ezt \*\*mintavételezéssel\*\* és \*\*kvantálással\*\* érik el, melynek eredménye tipikusan egy előjel nélküli egész szám lesz minden mintavételi időpillanatban\[cite: 2385, 2388, 2389].

\* \[cite\_start]\*\*Tulajdonságai és hibái:\*\* A kimenetet terhelheti offszet, nemlinearitás, illetve befolyásolhatja a frekvenciamenet\[cite: 2383]. Két nagyon fontos paraméterét meg kell különböztetni: a \*\*felbontást\*\* (legkisebb mérhető egység) és a \*\*pontosságot\*\* (a mért és valós érték közötti eltérés). (Példa: egy percmutatós óra felbontása 1 perc, de pontossága akár több perc is lehet, ha siet) \[cite\_start]\[cite: 2394, 2397].



\### 13. Beavatkozó definíciója és alapvető működése és tulajdonságai

\*\*A 47. dia alapján:\*\*

\* \[cite\_start]\*\*Definíció:\*\* Elektromos jeleket fizikai mennyiséggé átalakító eszköz\[cite: 2404]. \[cite\_start]Szintén jelátalakító, de ez a kapott jelet (parancsot) rákényszeríti a beágyazó fizikai környezetre\[cite: 2404].

\* \[cite\_start]\*\*Működése:\*\* Mivel a környezet manipulálásához sokszor jelentős erő kell, a beavatkozó jellemzően \*\*erősítő is egyben\*\* (mert a vezérlő elektronika önmagában nem bír akkora teljesítményt leadni)\[cite: 2404, 2405]. \[cite\_start]A számítógépes vezérlőjelet vissza kell alakítani (D/A konverzió - digitálisból analóg), ami fordítottan végzi a mintavételezést és kvantálást\[cite: 2406, 2407].

\* \[cite\_start]\*\*Típusai (tulajdonságok):\*\* Kialakításuk szerint lehetnek hidraulikus, pneumatikus, vagy elektromos (mágneses pl. relé, motor) rendszerek\[cite: 2421, 2422, 2423]. \[cite\_start]A repülőgépek szárnya is ilyen beavatkozókkal (Slats, Flap, Spoiler) van felszerelve\[cite: 2410, 2411, 2412].

