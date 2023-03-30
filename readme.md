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
- 2023-03-19. Banggod-ról rendeltem egy 100KHz-1,7GHz teljes spektrumú UV HF RTL-SDR USB-s eszközt. Megjön és éllesben is be fogom a
rádijelet és vissza fejtem, hogy megegyezik e ténylegesen a fentebb felvett digitális jellel. Illetve megtudhatom, hogy pontosan
milyen frekin kommunikál a két ezsköz.

![USB RTL-SDR](https://imgaz2.staticbg.com/thumb/large/oaupload/banggood/images/DE/8E/b0d1a97c-3cc3-4ad0-b930-153054d5e486.jpg "USB RTL-SDR")

A termék linkje: https://hu.banggood.com/100KHz-1_7GHz-Full-Band-UV-HF-RTL-SDR-USB-Tuner-Receiver-USB-Dongle-with-RTL2832U-R820T2-Ham-Radio-RTL-SDR-p-1942393.html 

# RTL.SDR adás
- 2023-03.30. Ma megérkezett a rádó vevő és izgatottan rá is vetettem magam. Elég bonyolult elsőre a dolog, de annyit már megtudtam, hogy
az adó eszköz (Q3RF) nem mindig ad jelet vagy is nem mindig adja ki rádión a jelet, csak abban az esetben, ha az álltala mért hőmrséklet
valóban a beállított érték küszöbökön túl van. Tehát a digitális jel, amit fentebb mértem, azok nem mindig mennek ki. Az eszközzel és a
felhasznált programokkal azt már megállapítottam, hogy hol  legerősebb a jel. A HDSDR programmal, sikerült belőni, hogy az én adóm a 
868.283.500Hz-en adja a jelet. Vagy is a mérések alapján itt a legerősebb a jel.

![USB RTL-SDR](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/adas_1.PNG "USB RTL-SDR")

# Universal Radio Hacker
- Program használata következik, amely eltarthat egy darabig. Kicsit bonyolultnak tűnik elsőre.