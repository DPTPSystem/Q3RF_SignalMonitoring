# Wand Text v2.10
* DPTP System - Wand Text v2.10
* Original version date: 2014-06-27.
* New version date: 2023-02-25.
* [neo-geo.hu](http://neo-geo.hu)
* PIC16F684, Thermal Fuse (acceleration switch assembly), MPLAB v8.91

![Wand Text v2.10](https://github.com/DPTPSystem/WandText/blob/master/images/wandtext_1.PNG "Wand Text v2.10")

# Program működése
- A program működése rendkívül egyszerű. Egy Timer0-ás megszakítás fut, amely másodpercenként okoz megszakítást, ezért óra mérés is készíthető
a programban. Ez a másodperces megszakítás emel egy tétlenségi változót, majd, ha eléri a 30-at vagy is a 30mp tétlenséget, akkor
SLEEP üzemmódra vált vagy is aludni megy a kontroller. Gombnyomásra felébred az eszköz és folytatja a működését. A gomb megnyomására vált a szövegek közt,
az eszközt a megfelelő ütemben jobbra és ballra mozgatva kirajzolódik a LED játékából és annak a frissítéséből egy szöveg, amelyet 
előre meghatároztunk. Ezek a szövegek tetszőlegesen változtathatóak, de minden változáshoz ismételt programozásra van szükség, amely
az ICSP felületen megtörténhet egy PicKit2 vagy megfelelő programozó segítségével.

# Definiciók és változók
- definiciók:
- - `#define	LED1		RA0		// Értelem szerűen a LED-ek protjai`
- - `#define	LED2		RC0`
- - `#define	LED3		RC1`
- - `#define	LED4		RC2`
- - `#define	LED5		RC5`
- - `#define	LED6		RC3`
- - `#define	LED7		RC4		// Led7`
- - `#define OSZIDO		80		// Oszlopok frissítési ideje`
- - `#define	SZUNET		120		// Betűk közti szünet`
- - `#define Tetlen_mp	30		// 30mp tétlenség, SLEEP`
- - `#define GSZENZ		RA1		// Gyorsulási kapcsoló`
- - `#define GOMB		RA2		// Menü gomb, ébresztés`
- Változók:
- - `unsigned short int timer_count = 0;		// Timer számláló`
- - `unsigned char mp_count = 0;			// Másodperc számláló`

# Thermal Fuse
- Hő biztosítékból készítünk gyorsulás érzékelős kapcsolót. (helyettesíthető, ha van más ötleted, pl. xbox360 függőleges kapcsoló)
- videó a készítésről: [Kattints ide a videó megtekintéséhez!](https://youtu.be/Vj7QPg4JtCE)
- a hőbiztosítékot az egyik végénél, (jellemzően a csúcsosabb gyantás végénél) le kell vágni vagy a lábát levágva lecsiszolni, 
majd a belső részeket ki kell szedni. Ez után egy kisebb rugót és egy csapágygolyót a belsejébe kell helyezni, amelynek mozgása 
fogja a kapcsolást a biztosíték fala és a polus közt megteremteni. A levágott végénél ésszerűen egy nyákdarabot addig csiszolni,
ameddig az pontosan nem illik be a hengerbe, majd ennek közepére a lábat befúrni és a belső oldalán rögzíteni, hogy ne jöjjön ki.
A rögzítés nitteléssel vagy forrasztással is lehetséges. A lényeg, hogy az egyik lába a biztosítéknak VCC (ez logika kérdése is)
lesz, a másik lába viszont független (10K Ohm-os ellenállással le van húzva a terven) kell legyen, vagy is ez a lába fogja a kapcsolást 
érzékelni.

# További videók
- [DPTP System - WandClock 1.0](https://youtu.be/EMNUdDG5QfQ)
- [DPTP System - WandClock 2](https://youtu.be/C4ieYFhOSFA)
- [DPTP System - WandClock 3](https://youtu.be/fKSn2c8CGdg)
- [DPTP System - HandText 2.0](https://youtu.be/XQEPS1zDwPo)

# Újratervezés
-  A programot először 2014-ben majd ebben az évben történő fejlesztések, továbbfejlesztések történtek meg.
- 2023-ban újra elővettem és normális terveket készítettem az eszközhöz és publikáltam, mert kiváló tanuló projekt lehet belőle.
- Az előző változathoz képest csak a LED portok változtak az optimális tervezhetőség miatt.

# Javítások
- 2023-02-26. ICSP csatlakozó felületen hibásan volt bekötve a PGD (data) vonal, ezt LED7-es jelőlte. Utolsó módosítással ezt helyére
tettem, így LED1-re változott a jelőlés. 
- 2023-03-07. Polarítás váltás a GOMB esetében, negatív esetben érvényesül a program.
- - `if(!GOMB){`
- - `while(!GOMB);`

![Wand Text v2.10](https://github.com/DPTPSystem/WandText/blob/master/images/wandtext_8.jpg "Wand Text v2.10")