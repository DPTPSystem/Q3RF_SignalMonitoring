# COMPUTHERM Q3RF
* DPTP System - COMPUTHERM Q3RF 
* Date: 2023-03-17.
* STM32F072, CC1101

![CC1101](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/p2.jpg "Padlófátés és hőfok/páratartalom figyelő")

# Rádió megfigyelése
- Ez a szál nem egy full kész projk megosztása céljából jön létre, hanem mert magamnak mint, amolyan jegyzet és történet
vezetés képpen, hogy vissza tudjam követni, hogy még is miket csináltam már meg és milyen eredményei voltak. Ettől független
publikálom az adatokat, hátha másnak is hasznosak lesznek és hátha bele botlok valakibe, aki majd tud segíteni, hogy a végső
célom elérjem.

# Célkitúzés
- Q3RF kommunikáció megfigyelése és az adatok feldolgozása, majd egy külön kijelzőn jelezni, hogy a fűtés rendszer aktív avagy sem.
- További célkitúzés lehet, egy optimálisabb vezérlés kialakítása, amely a lakás több helyiségének átlaghömérsékletének alapján 
vezérlné a fűtésrendszert.

# Eddigi megfigyeléseim
Szeretném jelezni már az elején, hogy azért az alábbi módon kezdtem el a megfigyelést mert nincs birtokomban jelenleg olyan eszköz,
amely segítségével direkt meg tudnám figyelni az éterben kiküldött rádiójelet.
- A meglévő adatokat az eszközön lévő rádió és MCU közti kommunikációra használt vezetékeire digitális analizátort kötöttem és lehallgattam
a köztük áramlott kommunikációs adatot, amely egyirányú volt. Csak az MCU küldött adatot 1 vezetéken a rádiónak. Megfigyeléseim arra engednek
következtetni, főként abból, hogy csak 1 vezetéket használ, hogy valamilyen sima időzített szintek csoportjaiban történik adat továbbítás.
Ezek lehetnek PWM jelek, vagy valamilyen időzített billegtetés. A megfigyelt jelek voltak a következők:

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

Tehát a fentebbi jesorozat vissza fejtve a következő képpen néz ki: 
` CMD1 5 byte`
` CMD1 Binárisa: 0011 1101 0001 1001 1100 0000 0000 0000 0110 0100`
` CMD1 Hexa: 0x  	3	 D	  1	   9	C	 0	  0	   0	6	 4`
Látható, hogy automatikusan kiegészült 1 bittel a sorozat, hogy értelmezhető byte alakot kapjunk. Jelenleg a sorozat elejére került
egy nulla, amely az értékén nem változtat, de a jel elcsúszhat 1 bit-et jobbra vagy adott esetben ballra.
* Fontos megjegyezni, hogy a CMD1-es parancs 4 szer egymás után érkezik egy pici szünettel elválsztva, mind a 4 ugyan olyan jel, de
a szünetek miatt még kérdéses az a bizonyos 1 bit, mert nem világos hol kezdődik a szünet. Továbbá véleményem szerint ez a parancs
tartalmazhatja az azonosítót is, bár a STOP jelnél más a sorozat.

- A következő parancs, ami még a START részét képezi azt CCMD1-nek neveztem el, amely már érdekesebb is mint az előző.
![CMD1](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/ccmd1.PNG "CMD1")
Ez a sorozat is 4szer egymát követően érkezik pici időkkel elválasztva. Emiatt talán jobban kiértékelhetőbb és élesebb határokkal 
rendelkezik a jelsorozat, mint a korábbiak, így könnyebb dolgom volt vele.

Tehát a fentebbi jesorozat vissza fejtve a következő képpen néz ki:
` CCMD1 7 byte` 
` CCMD1 Bináris: 0101 1000 1100 1100 0111 0000 0000 0101 1000 1100 1100 0111 0000 0000` 
` CCMD1 Hexa: 0x 	5	 8    C	   C 	7	 0    0	   5 	8	 C	  C	   7 	0	 0` 
Ami rögtön feltűnt mikor hexába átforgattam, hogy egy csomagban ismétli önmagát. Ami érdekes még, hogy ha felbonjuk kerek byte-okra
a sorozatot, akkor középpen és a legvégén marad 4 bit (0000), amely valószínűleg a két csomagot választja el egymástól. Vagy esetleg
lehetséges, hogy a sorozat elejéről hiányzik 4 0000-ás bit. (ezt utóbit csak találgatom)

# STOP
- WakeUp vagy init jel 45 db egymást követő 50%-os kitöltésű jel, kép fentebb látható, ugyan olyan.
* CMD2 jelsorozat visszafejtése
` CMD2 5 byte` 
` CMD2 Binárisa: 0111 1010 0011 0011 1000 0000 0000 0000 1100 1001` 
` CMD2 Hexa: 0x	 	7	 A	  3    3	8	 0	  0	   0	C	 9` 
Itt látható, hogy a START jelhez képest más a jelsorozat, bár kimutatható, de nem vészesen sok a külömbség.

* CCMD2 jelsorozat visszafejtése
` CCMD2 7 byte` 
` CCMD2 Bináris: 01011 0001 1001 1000 1111 1110 0000 1011 0001 1001 1000 1111 1110 0000` 
` CCMD2 Hexa: 0x	 B	  1	   9	8	 F	  E	   0	B	 1	  9	   8	F	 E	  0`
Itt viszont megint jól látszik, hogy egy csomagban ismétli önmagát a sorozat, ugyan azokkal a jellemzőkkel mint ahogy a CCMD1-nél leírtam.