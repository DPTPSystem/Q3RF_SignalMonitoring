# COMPUTHERM Q3RF
* DPTP System - COMPUTHERM Q3RF 
* Date: 2023-03-17.
* STM32F072, CC1101

![CC1101](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/tree/master/p1.jpg "Padlófátés és hőfok/páratartalom figyelő")

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

- WakeUp vagy init jel 45 db egymást követő 50%-os kitöltésű jel:
![Init](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/init2.PNG "init jel")
Ez a jel egyébként minden START vagyéppen a STOP jelzés elején ott van. Ebből arra következtetek, hogy ez a jel jelzi a vevőnek, 
hogy adatok fognak érkezni. A jel után egy 4mS-os szünet érkezik.

- Következő képen az Init és az első parancs, amelyet a továbbiakban CMD1 néven nevezek.
![InitandCMD1](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/cmd1.PNG "Init és CMD1")

- CMD1 parancs 39-40 egymást követő jel alkotja:
![CMD1](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/cmd1_kozelebrol.PNG "CMD1")
Azárt nem tudom, hogy 39 vagy 40 mert a végén van egy jel, amelyet nem tudok megfelelően értelmezni egyelőre. Az vagy egy levágott jel,
amely nulla, vagy a következő ismétlés közti szünet egy része.

- CMD1 megfejtése, binárisan ábrázólva
![CMD1](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/cm1_binaris.PNG "CMD1")

- Két CMD1 és majd a CCMD1 parancsok közti szünetről, amelyet egyelőre nem tudok értékelni, csak találgatok:
![CMD1](https://github.com/DPTPSystem/Q3RF_SignalMonitoring/blob/master/images/nem_duom_minek_a_resze.PNG "CMD1")
A képen a 0-től 1-ig történő időközről van szó. Az 1-től 2-ig tartott rész egyértelműen egy szünet.
