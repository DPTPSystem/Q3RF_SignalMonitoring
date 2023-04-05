# COMPUTHERM Q3RF
* DPTP System - COMPUTHERM Q3RF 
* Date: 2023-03-17.
* STM32F072, CC1101

![CC1101](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/p2.jpg "Padlófátés és hőfok/páratartalom figyelő")

# Az eszköz működése
* Az adóban egy SH79F3283P MCU-t használnak, majd erre egy ismeretlen, de a leírásából adódóan 868.35MHz-es rádió kapott helyet. 
Az MCU adatlapjából az derül ki, hogy egy analog vagy éppen digitális lábról van szó. PWM-et elvileg nem tud. Az adatlapot a documents
mappában megtalálható. A készüléken egy kapcsolóval lehet nappali és esti hőmérsékleti értéket beállítani, majd ezzel a kapcsolóval 
bármikor egyikről a másik állapotra átállítani a készüléket. Az érzékenységét lehet állítani, de nálam a legmagasabb értéken van, ami
0.2 fok. Tehát ha a hőmérséklet alá vagy főlé megy 0.2 fokkal a beállított értéknek, akkor ki vagy be kapcsólja a kazánt.

![Q3RF](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/q3rf_4.jpg "Q3RF felépítése")

MCU képe, 20. láb az érdekes nekünk, azon a lábon érkezik a jel a rádióba:

![Q3RF](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/sh793283p.PNG "Q3RF felépítése")

# Rádió megfigyelése
- Ez a szál nem egy full kész projk megosztása céljából jön létre, hanem mert magamnak mint, amolyan jegyzet és történet
vezetés képpen, hogy vissza tudjam követni, hogy még is miket csináltam már meg és milyen eredményei voltak. Ettől független
publikálom az adatokat, hátha másnak is hasznosak lesznek és hátha bele botlok valakibe, aki majd tud segíteni, hogy a végső
célom elérjem.

# Célkitűzés
- Q3RF kommunikáció megfigyelése és az adatok feldolgozása, majd egy külön kijelzőn jelezni, hogy a fűtés rendszer aktív avagy sem.
- További célkitúzés lehet, egy optimálisabb vezérlés kialakítása, amely a lakás több helyiségének átlaghömérsékletének alapján 
vezérlné a fűtésrendszert, figyelembe véve a külső hőmérsékleteket és a helyiségekben mért páratartalmat.

# Eddigi megfigyeléseim
Szeretném jelezni már az elején, hogy azért az alábbi módon kezdtem el a megfigyelést mert nincs birtokomban jelenleg olyan eszköz,
amely segítségével direkt meg tudnám figyelni az éterben kiküldött rádiójeleket.
- A meglévő adatokat az eszközön lévő rádió és MCU közti kommunikációra használt vezetékeire digitális analizátort kötöttem és lehallgattam
a köztük áramlott kommunikációs adatot, amely egyirányú volt. Csak az MCU küldött adatot 1 vezetéken a rádiónak. Megfigyeléseim arra engednek
következtetni, főként abból, hogy csak 1 vezetéket használ, hogy valamilyen sima időzített szintek csoportjaiban történik adat továbbítás.
Ezek lehetnek szoftveres PWM jelek, vagy valamilyen időzített billegtetés. A megfigyelt jelek a következők voltak:

# START
- WakeUp vagy init jel 45 db egymást követő 50%-os kitöltésű jel:
![Init](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/init2.PNG "init jel")
Ez a jel egyébként minden START vagy éppen a STOP jelzés elején ott van. Ebből arra következtetek, hogy ez a jel jelzi a vevőnek, 
hogy adatok fognak érkezni. A jel után egy 4mS-os szünet érkezik.

- Következő képen az Init és az első parancs, amelyet a továbbiakban CMD1 néven nevezek.
![InitandCMD1](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/cmd1.PNG "Init és CMD1")

- CMD1 parancs 39-40 egymást követő jel alkotja:
![CMD1](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/cmd1_kozelebrol.PNG "CMD1")
Azárt nem tudom, hogy 39 vagy 40 mert a végén van egy jel, amelyet nem tudok megfelelően értelmezni egyelőre. Az vagy egy levágott jel,
amely nulla vagy a következő ismétlés közti szünet egy része.

- CMD1 megfejtése, binárisan ábrázólva
![CMD1](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/cm1_binaris.PNG "CMD1")

- Két CMD1 és majd a CCMD1 parancsok közti szünetről, amelyet egyelőre nem tudok értékelni, csak találgatok:
![CMD1](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/nem_duom_minek_a_resze.PNG "CMD1")
A képen a 0-től 1-ig történő időközről van szó. Az 1-től 2-ig tartott rész egyértelműen egy szünet. Ezt majd később még biztosan 
ki kell értékelni. Arra is gondoltam, hogy a szünet kezdete összefolyik az utolsó jel végével és így annak megfelelően, hogy mennyi
ideig van magas szinten dönti el, hogy 0 vagy 1-es a jel értéke. Ezt majd később látni is fogjuk.

Tehát a fentebbi jelsorozat vissza fejtve a következő képpen néz ki: 

` CMD1 5 byte`

` CMD1 Binárisa: 0111 1010 0011 0011 1000 0000 1000 0000 0100 1001`

` CMD1 Hexa: 0x7A33808049`

Látható, hogy automatikusan kiegészült 1 bittel a sorozat, hogy értelmezhető byte alakot kapjunk. Jelenleg a sorozat elejére került
egy nulla, amely az értékén nem változtat, de a jel elcsúszhat 1 bit-et jobbra vagy adott esetben ballra.
* Fontos megjegyezni, hogy a CMD1-es parancs 4 szer egymás után érkezik egy pici szünettel elválsztva, mind a 4 ugyan olyan jel, de
a szünetek miatt még kérdéses az a bizonyos 1 bit, mert nem világos hol kezdődik a szünet. Továbbá véleményem szerint ez a parancs
tartalmazhatja az azonosítót is, bár a STOP jelnél más a sorozat.

- A következő parancs, ami még a START részét képezi azt CCMD1-nek neveztem el, amely már érdekesebb is mint az előző.
![CMD1](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/ccmd1.PNG "CMD1")
Ez a sorozat is 4szer egymát követően érkezik pici, de határozott időkkel elválasztva. Emiatt talán jobban kiértékelhetőbb és élesebb határokkal 
rendelkezik a jelsorozat mint a korábbiak, így könnyebb dolgom volt vele.

Tehát a fentebbi jelsorozat vissza fejtve a következő képpen néz ki:

` CCMD1 7 byte` 

` CCMD1 Bináris: 0101 1000 1100 1100 0111 0000 0000 0101 1000 1100 1100 0111 0000 0000` 

` CCMD1 Hexa: 0x58CC70058CC700` 

Ami rögtön feltűnt mikor hexába átforgattam, hogy egy csomagon bellül is ismétli önmagát. Ami érdekes még, hogy ha felbonjuk kerek byte-okra
a sorozatot, akkor középpen és a legvégén marad 4 bit (0000), amely valószínűleg a csomagon belüli 2 csomagot választja el egymástól. Vagy esetleg
lehetséges, hogy a sorozat elejéről hiányzik 4 0000-ás bit. (ezt utóbit csak találgatom)

# STOP
- WakeUp vagy init jel 45 db egymást követő 50%-os kitöltésű jel, kép fentebb látható, ugyan olyan.

* CMD2 jelsorozat visszafejtése

![CMD2](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/cmd2.PNG "CMD2")

Kinagyítva a CMD2-es.
![CMD2N](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/cmd2_kozelrol.PNG "CMD2N")

` CMD2 5 byte`

` CMD2 Binárisa: 0111 1010 0011 0011 1000 0000 0000 0000 1100 1001`

` CMD2 Hexa: 0x7A338000C9`

Itt látható, hogy a START jelhez képest más a jelsorozat bár kimutatható, de nem vészesen sok a külömbség.

* CCMD2 jelsorozat visszafejtése

![CCMD2](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/ccmd2.PNG "CCMD2")

` CCMD2 7 byte` 

` CCMD2 Bináris: 0101 1000 1100 1100 0111 1111 0000 0101 1000 1100 1100 0111 1111 0000`
 
` CCMD2 Hexa: 0x58CC7F058CC7F0`

Itt viszont megint jól látszik, hogy egy csomagban ismétli önmagát a sorozat, ugyan azokkal a jellemzőkkel mint ahogy a CCMD1-nél leírtam.

# Adatok összevetése
* - Megjegyzés: Ezen gondolataim leírása közben ujra áttekintettem a digitális mintákat és kiderült, hogy rosszúl értelmeztem ezeket javítottam
feljebb is, hogy a pontos adatok legyenek leírva. A képeket nem módosítottam.

* START és STOP (CMD1, CMD2) jel egymás alatt

` CMD1 Binárisa: 0111 1010 0011 0011 1000 0000 1000 0000 0100 1001 || CMD1 Hexa: 0x7A33808049`

` CMD2 Binárisa: 0111 1010 0011 0011 1000 0000 0000 0000 1100 1001 || CMD2 Hexa: 0x7A338000C9`

* START és STOP (CCMD1, CCMD2) jel egymás alatt

` CCMD1 Bináris: 0101 1000 1100 1100 0111 0000 0000 0101 1000 1100 1100 0111 0000 0000 || CCMD1 Hexa: 0x58CC70058CC700`

` CCMD2 Bináris: 0101 1000 1100 1100 0111 1111 0000 0101 1000 1100 1100 0111 1111 0000 || CCMD2 Hexa: 0x58CC7F058CC7F0`

- Újra értelmezve a jeleket, már sokkal szebb és értelmesebb adatokat látok mint korábban, egyértelműen kiolvashatók a külömbségek és
értelmesnek és helyesnek tűnnek az adatok.
További felbontás ezuttal csak hexában:

* START és STOP (CMD1, CMD2) jel egymás alatt

` CMD1 Hexa: 0x 7A 33 80 80 49`

` CMD2 Hexa: 0x 7A 33 80 00 C9` - Utolsó 2 byte a külömbség

* START és STOP (CCMD1, CCMD2) jel egymás alatt

` CCMD1 Hexa: 0x 58 CC 70 - 0 - 58 CC 70 - 0`

` CCMD2 Hexa: 0x 58 CC 7F - 0 - 58 CC 7F - 0` - Ketté osztható a csomag és a felekben csak az utolsó 4 bit változik.

# Konklúzió
- Első 45 jel továbbra is úgy gondolom, hogy egy ébresztő vagy más jelzés a vevőnek, hogy adatok fognak érkezni.
- CMD1, CMD2 azonosító adat lehet, Bár nem értem az utolsó 2 byte jelentőségét egyelőre. Ha csak nem itt is jelzi mondjuk a START illetve
a STOP jelet. Pl.: 0x80 = START, 0x00 = STOP.
Kérdés továbbá itt, hogy az utolsó 1 byte mit közölhet. Ennek majd még utána nézek, pl elem merülés?
- CCMD1, CCMD2 itt sokkal jobb a helyzet mind két esetben csak az utolsó 4 bit változik. 0x0 bekapcsolás, 0xF kikapcsolás. 
Az első 3 byte és a közvetlen mögötte lévő vagy az utolsó byte felső 4bitje még hozzá tartozik az azonosítához.
Mind 2 csomag első része valamilyen azonosító lehet, ezek nem változnak sorozaton belül, újra őárosítást még nem próbáltam, hogy az 
hatással lehet e rájuk, de majd fűtési szezont követően lehet kipróbálom, ha nem oldódik meg nélküle a figyelés.
- Ami még talán fontos, hogy a jelsorozatok egy amolyan sajátos szoftveres PWM vezérléshez hasónlítanak, az ingadozások pont amiatt lehetnek,
hogy eggyes esetekben az MCU nem tudja folyamatosan ~uS alatt tartani a kivezérlésének sebességét. (idejét) A jelsorozatókból jól látszik, 
hogy vannak olyan esetek, amelyeknél az MCU lasabb kiszolgálása miatt a sorozatok közti szünetek el, összecsúsznak és ezért van olyan érzése
a megfigyelőnek, hogy valami nem teljesen stimmel a visszafejtésnél. 

# Ismétlés a tudás annya
Újabb megigyelt csomagot vizsgáltam meg ellenőrzés képpen, az eredmények ugyan azok lettek, tehát kijelenthető, 
hogy helyesek az adatok illetve a visszafejtések. Önmagában a meglévő csomagok tovább vizsgálása már újabb eredményt nem hoznának.

START jel

`CMD1 : 0b01111010 00110011 10000000 10000000 01001001 - 0x7A33808049`

`CCMD1 : 0b01011000 11001100 01110000 00000101 10001100 11000111 00000000 - 0x58CC70058CC700`

STOP jel

`CMD2 : 0b01111010 00110011 10000000 00000000 11001001 - 0x7A338000C9`

`CCMD2 : 0b01011000 11001100 01111111 00000101 10001100 11000111 11110000 - 0x58CC7F058CC7F0`

# Merülő elem teszt
- Következő teszt a merülő elem teszt, hogy milyen csomagok mennek. (ezt csak akkor, ha nem lesz más ötletem)

# Hibák
2023-03-17. Újra átnéztem az összes digitális jelet és már kicsit jobb rálátással újra értelmeztem őket, a leírásban javítottam tehát
már a jó bináris és hexa államonyokat olvashatjuk. Képeket nem cseréltem, azok továbbra is hibásan ábrázolják a vissza fejtést.
2023-03-25. Az eszköz működésésénél írtam, hogy nem PWM a vezérlés, vagy legalább is erre utaltam benne mert az MCU adatlapjában látható,
a felhasznált port nem tud hardveres PWM vezérlést. Annyival egészíteném ki ezt a részt, hogy ettől még szoftveres PWM kimenet megoldható,
bármelyik portra.

# Rádió eszköz
2023-03-19. Banggod-ról rendeltem egy 100KHz-1,7GHz teljes spektrumú UV HF RTL-SDR USB-s eszközt. Megjön és éllesben is be fogom a
rádijelet és vissza fejtem, hogy megegyezik e ténylegesen a fentebb felvett digitális jellel. Illetve megtudhatom, hogy pontosan
milyen frekin kommunikál a két ezsköz.

![USB RTL-SDR](https://imgaz2.staticbg.com/thumb/large/oaupload/banggood/images/DE/8E/b0d1a97c-3cc3-4ad0-b930-153054d5e486.jpg "USB RTL-SDR")

A termék linkje: https://hu.banggood.com/100KHz-1_7GHz-Full-Band-UV-HF-RTL-SDR-USB-Tuner-Receiver-USB-Dongle-with-RTL2832U-R820T2-Ham-Radio-RTL-SDR-p-1942393.html 

# RTL.SDR adás
2023-03.30. Ma megérkezett a rádó vevő és izgatottan rá is vetettem magam. Elég bonyolult elsőre a dolog, de annyit már megtudtam, hogy
az adó eszköz (Q3RF) nem mindig ad jelet vagy is nem mindig adja ki rádión a jelet, csak abban az esetben, ha az álltala mért hőmrséklet
valóban a beállított érték küszöbökön túl van. Tehát a digitális jel, amit fentebb mértem, azok nem mindig mennek ki. Az eszközzel és a
felhasznált programokkal azt már megállapítottam, hogy hol  legerősebb a jel. A HDSDR programmal, sikerült belőni, hogy az én adóm a 
868.283.500Hz-en adja a jelet. Vagy is a mérések alapján itt a legerősebb a jel.

![USB RTL-SDR](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/adas_1.PNG "USB RTL-SDR")

# HDSDR / Audacity
Programok használata következik a legjobb vételi frekit kell megtalálnom. Annak ellenére, hogy a HDSDR szerint a legerősebb jel 868.283MHz-en
mutatkozik a vizesés modelezés szerint, még is a jobb jeleket, amelyeket ellenőrzésként és visszafejtés miatt a Audacity programban ellenőriztem,
inkább a 868.35-868.6M-ig tartományban kaptam. Az így kapott értékeket már elemeztem is és nem meglepő módon teljesen öszhangban vannak az 
előzőleg a digitális analizátorral felvett jelekkel. Szerintem ez már egyértelműen meghatározza, hogy milyen jeleket kell várjak, vagy küldjek.

Visszafejtés, a Audacity program képéből: - init jel
![USB RTL-SDR](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/44_init_jel.PNG "USB RTL-SDR init adás")
44 külön álló ébresztő jel, majd ezt követően jönnek az érvényes adatok. Ez amúgy a diitálisnál 45 ször van, szerintem a rádió és a szoftveres
PWM közti külömbség itt megmutatkozik. Ettől még ez véleményem szerint az ébresztó Init jel.

További jelek visszafejtve kép nélkül:

START jel:

`0b0111101000110011100000001000000001001001 - 0x7A33808049 - START`

`0b0111101000110011100000001000000001001001 - 0x7A33808049 - START ismétlés 4x`

`0b01011000110011000111000000000101100011001100011100000000 - 0x58CC70058CC700 - START CCMD`

`0b01011000110011000111000000000101100011001100011100000000 - 0x58CC70058CC700 - ismétlés 4x`

STOP jel:

`0b0111101000110011100000000000000011001001 - 0x7A338000C9 - STOP`

`0b0111101000110011100000000000000011001001 - 0x7A338000C9 - STOP insmétlés 4x`

CCMD STOP jelből egy részlet:
![USB RTL-SDR](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/CCMD_stop.PNG "USB RTL-SDR CCMD adás")

`0b01011000110011000111111100000101100011001100011111110000 - 0x58CC7F058CC7F0 - CCMD STOP`

`0b01011000110011000111111100000101100011001100011111110000 - 0x58CC7F058CC7F0 - imsétlés 4x`

Jelalakok, 868,100M-től 868,6M tartományban:
![USB RTL-SDR](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/jelalak.PNG "USB RTL-SDR Jelalakok")

Az érdekesség és a miértje a fentebbi képnek, hogy azt vettem észre, hogy nem feltétlen az a jó jelerőség, amelyet a HDSDR program mutat
mert, hogy tesztelést követően illetve Audacity programot használva jól látszik, hogy egyes jelek milyen éllesek avagy mennyire kaotikus
ábrázolást muatnak. 50KHz-es lépésekben vettem fel jeleket, a képen ez látható. Azon frekiket ki is vettem, amelyek nem mutattak semmit.
A legszebb és legélesebb jelalakokat a 868,450M és 868,600M-es frekint tudtam rögzíteni. Még azt nem tudom eldönteni, hogy ha a jel 
alak ezeken a frekvenciákon a legtisztábbak, akkor ugyan ezen a frekire kellene állítanom az CC1101-es adóvevőmet, vagy azt más akar az 
adatok alapján, amit a készüléken is feltüntettek a 868,350M-es frekire lőjem be? Ez a kérdés már szerinem az éles teszteléssel fog majd eldőlni.

# Hacker
Sajnos még a programot ki kell ismernem, de nem kevés idő után, azért egy ilyen állapotot már elértem..
A jel felbontása tökéletes ahhoz, hogy felismerjük a szinteket és leolvasva, vissza tudjuk fejteni az értékét.

![USB RTL-SDR](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/hack.PNG "USB RTL-SDR Jelalakok")
A programot még nem tudom megfelelően használni, így az értékeket automatikusan nem tudja megfejteni. Ez még várat magára.

# Adás tétlenség
Megfigyeléseim alapján az adó 5 percenként küld egy jelet a vevőnek, amelyet még nem fejtettem vissza, de a megfigyeléseim
már megkezdőtek e tekintetben is.

# Új írány vagy újra kezdés
2023-04-03: Sajnos a korábban Ali-ról rendelt CC1101-es modult nem tudtam rá venni, hogy küldjön ki jeleket, így egy másik modullal progbálkoztam, 
amely gyárilag forrasztási hibás volt, próbáltam megjavítani, de sajnos teljesen tönkre ment legalább is a kommunikáció teljesen ellehetetlenűlt. 
Az eltelt idő és a rengeteg féle képpen lekódolt adatküldő rutin vagy éppen az inicializáló program illletve az a rengeteg áttanulmányozott forrás
és a CC1101-es adatlap után azt kell feltételeznem, hogy a megvásárólt modulok alapból hibásak lehetnek, mert olyan nincs, hogy egyetlen
eszszer sem passzol semmi legalább annyira, hogy jelet küldjön az eszköz. Utolsó lehetőségként rendeltem 2 újabb modult ezuttal 
németországból, 1-2 héten belül megérkezik és adok egy utolsó esélyt ennek a projektnek, ha nem jutok előrébb az újabb modullal sem,
akkor ezt a projektet félre fogom tenni, és más eszközzel kell befognom a Q3RF jeleit.

# Program hiba
2023-04-04: Nem hagyott nyugodni, hogy látszólag mindne rendben van az eszközzel és jó értékeket is kapok vissza egy-egy lekérdezés vagy 
konfigurálás kapcsán, ezért átnéztem ismét a teljes kódomat. Meglepődve tapasztaltam, hogy néhány módosítás és tesztelés után észre vettem
valamit a vizesésben, ami olyan volt mint, amilyenhez hasonlót vártam. És az is volt.. ;)

![USB RTL-SDR](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/korekciozott_jel.PNG "USB RTL-SDR Jelalakok")

A képen már egy korrekciózott jelet lehet látni, de még csak egy 40 byte random csomaggal, ez után jön majd, annak a beállítása, hogy
azokat az adatokat küldje ki, amelyeket én akarok és pont ugyan úgy, mint az eredeti adó is teszi. A korrekcióval jelenleg 868.355MHz-en
küldi ki a jelet, pont ott, ahol az eredeti adó is küldi, vagy is a CC1101-es modulom, jelentősen csal, pontosan a következő mértékben:
Skálán beállított érték: 868,286MHz
Valós érték: 868,355MHz
A kettő külömbségét adtam meg a programnak (PPM), amellyel korigálta CC1101 csúszását. Ennek megfelelően a programot is módosítottam és az
új értékkel használom.

2023-04-05. Kis tétlengedés és gondolkodás után elkezdtem fogglalkozni a CC1101-es rádióval, hogy lemásolva ugyan azt a jelet adja ki, mint
az eredeti rádió. Ennek eredményeként néhány képet megosztok:

A felső ábrán az eredeti rádió álltal kiadott jel látható, alatta pedig a CC1101-es rádió álltal kiadott jel.
![USB RTL-SDR](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/origi_and_new.PNG "USB RTL-SDR Jelalakok")

Messziről majdnem teljesen egyezést mutat, de sajna van két pontosabban 3 kisebb eltérés az eredetihez képest. A következő képen 
az első jel látszik, hogy kétszer akkora mint az eredeti. Sajnos fél bit-et nem tudok küldeni, szóval ez ilyen és remélem ennyi eltérés
ha a köztes részek egyformák nem lehet probléma. Amúgy is arra gondolok, hogy az eredeti rendszernek ez egy hibája lehet, nem szabad, hogy
sokat számítson.
![USB RTL-SDR](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/origi_and_new_reszlet.PNG "USB RTL-SDR Jelalakok")

Egy köztes jel kinagyítva, azt hiszem ezzel probléma nem lehet..
![USB RTL-SDR](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/origi_and_new_reszlet_2.PNG "USB RTL-SDR Jelalakok")

És a jelsorozat végén van egy pici eltérés, amelynél az eredeti kicsit később kezdődik és kicsivel rövidebb mint a CC1101-es változat, 
de talán ez sem lehet nagy gond révén, hogy ez valamilyen ébresztő jel lehet.
![USB RTL-SDR](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/origi_and_new_reszlet_3.PNG "USB RTL-SDR Jelalakok")

Ez az Init vagy WakeUp jel, 45db 50%-os kitúltésű jel, amely a vevőnek jelez, hogy adatok fognak érkezi. Ez az adasor az Init esetében 
4.3 kBaud sebességgel érkezik.

2023-04-06. CMD STAR jelet sikerrel létrehoztam, mondhatni tökéletesen sikerült. Sajnos programból CC1101 modem sebességét parancsonként
állítani kell nem egészen értem miért így írta meg az eredeti szoftvert a gyártó, de lehet pont azért, hogy minél nehezebb legyen 
visszafejteni vagy a készülékéhez kompatibilis eszközt gyártani. Míg a WakeUp jel, 4.3 kBaud sebességgel megy, a CMD START jel már csak 
2.48 kBaud sebességel. 

A képen felül az eredeti jel, alatta a CC1101-es modul álltal adott CMD START jel.
![USB RTL-SDR](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/origi_and_new_cdmstart.PNG "USB RTL-SDR Jelalakok")