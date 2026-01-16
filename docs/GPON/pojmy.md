## GEM Port

**GEM Port** (GPON Encapsulation Method) je základní virtuální transportní kanál sloužící k přenosu dat mezi OLT a ONT v sítích GPON. Jeho hlavním účelem je zapouzdření (enkapsulace) uživatelských datových rámců, nejčastěji Ethernetových, do fragmentů proměnlivé délky, které jsou následně posílány skrze optickou síť.

### Identifikace a filtrování dat

V **downstreamu** (od OLT k uživateli) jsou data vysílána formou broadcastu všem připojeným jednotkám ONT

**GEM port ID** slouží jako unikátní identifikátor, díky kterému konkrétní ONT pozná, která data patří právě jemu. ONT filtruje příchozí provoz a zpracovává pouze ty GEM PDUs (Protocol Data Units), jejichž ID se shoduje s porty nakonfigurovanými na daném zařízení.

### Agregace a mapování služeb

V **upstreamu** (od uživatele k OLT) probíhá proces následovně:

1. Enkapsulace: Ethernetové rámce z uživatelského zařízení jsou v ONT zapouzdřeny do GEM paketů
2. Mapování na T-CONT: GEM porty jsou následně mapovány do front v rámci T-CONT
3. Odeslání: Data jsou odeslána v přesně vymezeném časovém slotu, který OLT přidělilo danému T-CONTu

### Oddělení různých typů provozu

GEM porty umožňují logické oddělení různých služeb v rámci jednoho optického vlákna. Výrobce doporučuje přiřazovat různé GEM porty pro různé služby (např. jeden pro internet, jiný pro VoIP nebo IPTV)

To umožňuje nastavit specifické parametry kvality služeb (QoS) a priority pro každý typ provozu zvlášť.

## T-CONT

T-CONT (Transmission Container) je základní logická jednotka v sítích GPON, která slouží k řízení a alokaci šířky pásma v upstreamu. Zatímco [GEM porty](#gem-port) slouží k identifikaci a separaci konkrétních služeb, T-CONT funguje jako virtuální „přepravní kontejner“, kterému OLT přiděluje konkrétní časové sloty pro odesílání dat.

### Řízení upstream provozu

V GPON síti sdílejí všechna ONT jedno optické vlákno (a jednu vlnovou délku) pro odesílání dat. Aby nedošlo ke kolizím, používá se metoda TDMA (Time Division Multiple Access). OLT vysílá v každém rámci instrukci zvanou **BWmap** (Bandwidth Map), která přesně určuje, který T-CONT má v jaký okamžik dovoleno vysílat svá data.

### Spolupráce s DBA

T-CONT je klíčovým prvkem pro mechanismus dynamického přidělování šířky pásma DBA (Dynamic Bandwidth Allocation).

1. ONT neustále reportuje stav svých T-CONT zásobníků do OLT.
2. OLT na základě těchto reportů a přiřazeného **DBA profilu** (který definuje pevnou, garantovanou nebo maximální rychlost) dynamicky mění velikost časových oken pro daný T-CONT

### Agregace GEM portů

T-CONT slouží jako vyšší vrstva pro GEM porty. Proces přenosu dat vypadá následovně:

1. Uživatelská data (Ethernet rámce) vstoupí do ONT
2. Data jsou zapouzdřena do GEM portů
3. GEM porty jsou namapovány do konkrétního T-CONTu
4. T-CONT čeká na svůj časový slot přidělený od OLT a následně odešle všechna data ze svých GEM portů do optické sítě

### Typy T-CONT

V praxi se používají různé typy T-CONT (označované jako Type 1 až Type 5), které umožňují prioritizaci provozu (QoS):

* Fixní šířka pásma: Pro služby citlivé na zpoždění (např. VoIP).
* Garantovaná šířka pásma: Pro stabilní služby (např. video).
* Best-effort (maximální): Pro běžný internetový provoz, kde rychlost kolísá podle vytížení sítě.

## OMCI

OMCI (Optical Network Unit Management and Control Interface) je standartizovaný protokol (definovaný doporučením ITU-T G.984.4), který slouží k vzdálené správě a konfiguraci ONT přímo z OLT. Pracuje na 2. a 3. vrstvě a funguje jako most mezi OLT a ONT, který umožňuje poskytovatelům služeb spravovat koncová zařízení bez nutnosti manuálního zásahu u zákazníka.

K přenosu OMCI zpráv využívá OLT dedikovaný logický kanál zvaný OMCC (ONT Management and Control Channel). Tento kanál běží nad specifickým [GEM portem](#gem-port), který je vytvořen automaticky během inicializace ONT.

### Správa konfigurace

OLT přes OMCI definuje, jak se má ONT chovat, což zahrnuje vytváření T-CONTů, mapování GEM portů, nastavování pravidel pro VLAN tagování a definování parametrů kvality služeb (QoS).
* **Správa poruch**: Protokol umožňuje OLT detekovat a reagovat na chyby v reálném čase, například generovat alarmy při degradaci signálu nebo selhání hardwaru.
* **Sledování výkonu**: ONT skrze OMCI neustále hlásí statistiky provozu, jako je propustnost, chybovost paketů, latence nebo síla optického signálu.
* **Údržba a upgrady**: OMCI usnadňuje vzdálenou aktualizaci firmwaru ONT a diagnostiku spojení.

### Interoperabilita (Vendor-neutrality)

Díky tomu, že je OMCI mezinárodním standardem, zajišťuje, že OLT od jednoho výrobce (např. C-DATA, Bdcom) může komunikovat a spravovat ONT od jiných výrobců (např. Huawei, TP-Link), pokud obě zařízení standard OMCI plně podporují. To dává operátorům flexibilitu při výběru koncových zařízení bez závislosti na jednom dodavateli.

## BWmap 

Primární funkcí BWmap (Bandwidth Map) je přidělování vysílacích časů pro **upstream** jednotlivým jednotkám ONT.

### Řízení přístupu ke sdílenému médiu

V GPON síti sdílejí všechna připojená zařízení ONT jedno optické vlákno pro odesílání dat k OLT. Aby nedocházelo ke kolizím a konfliktům dat, musí OLT přesně určovat, kdy a jak dlouho smí každé ONT vysílat.

### Definice časových slotů

BWmap obsahuje instrukce, které specifikují přesný začátek a konec časového okna pro konkrétní kontejnery T-CONT každé jednotky ONT. ONT pak vysílá svá data pouze v těchto přidělených úsecích v rámci TDMA

### Rozhodování na základě DBA

O přidělení šířky pásma rozhoduje modul DBA uvnitř OLT. Ten v reálném čase sbírá zprávy o stavu front v ONT, provádí výpočty podle nastavených priorit a výsledné instrukce odesílá právě skrze BWmap v každém sestupném rámci, tedy každých 125 μs.

Díky BWmap může OLT dynamicky měnit velikost přiděleného času pro každé zařízení podle aktuální potřeby, což zvyšuje celkovou propustnost sítě a umožňuje obsloužit více uživatelů na jednom portu.

## DBA profil

Hlavním účelem DBA profilu je optimalizace využití přenosové kapacity optického vlákna v **upstreamu**. Namísto toho, aby měl každý uživatel pevně vyhrazenou rychlost, kterou nevyužívá neustále, umožňuje DBA profil přidělit nevyužitou kapacitu jiným aktivním uživatelům, čímž zvyšuje celkovou propustnost.

### Typy šířky pásma

V rámci DBA profilu se definují různé typy přenosových kapacit, které odpovídají požadavkům různých služeb:

* **Fixed Bandwidth** (Fix – Type 1): Pevně vyhrazená kapacita, která je uživateli k dispozici neustále, bez ohledu na to, zda ji využívá. Je ideální pro služby extrémně citlivé na zpoždění a jitter, jako je VoIP nebo správa sítě.
* **Assured Bandwidth** (Assure – Type 2/3): Garantovaná šířka pásma, kterou OLT poskytne ONT kdykoliv o ni požádá. Pokud ONT data neposílá, může být tato kapacita dočasně uvolněna pro jiný provoz.
* **Maximum Bandwidth** (Max – Type 4): Horní limit (tzv. "best-effort"), který může ONT využít, pokud je v síti aktuálně volná kapacita. Tato rychlost není garantována a závisí na celkovém vytížení portu.
* **Kombinované typy** (Assure & Max / Fix & Assure & Max): Umožňují kombinovat pevnou (Fix) garantovanou (Assure) a maximální (Max) rychlost, což je nejčastější nastavení pro běžné internetové tarify.

[GEM porty](#gem-port) v rámci [T-CONT](#t-cont) sdílí tuto přidělenou šířku pásma.

## VLANIF

V kotextu OLT představuje VLANIF (VLAN Interface) virtuální rozhraní na 3. síťové vrstvě, které slouží k přiřazení IP adresy konkrétní VLANě. VLANIF umožňuje, aby OLT vystupovalo v síti jako samostatný koncový bod s vlastní IP adresou.

### Rozhraní pro „In-band“ správu

VLANIF je klíčové pro nastavení tzv. in-band managementu. To znamená, že OLT můžete spravovat skrze jeho běžné datové uplink porty namísto použití dedikovaného fyzického MGMT portu / Management WiFi. Pokud chcete OLT konfigurovat z počítače připojeného do stejné datové sítě, musíte vytvořit VLANIF rozhraní pro VLANu, kterou používáte pro správu.

Bez konfigurace VLANIF by OLT pouze přepínalo pakety, ale nemělo by adresu, na kterou se lze připojit přes webový prohlížeč (HTTP/HTTPS) nebo Telnet/SSH. Vytvoření tohoto rozhraní je také nezbytným předpokladem pro:

* **Nastavení Gateway**: Aby OLT mohlo komunikovat se zařízeními mimo svou lokální podstíť, například se serverem CMS, musí mít VLANIF rozhraní definovanou bránu.
* **Routování**: VLANIF umožňuje OLT využívat funkce 3. vrstvy, jako je statické směrování nebo DHCP-relay.

## CMS

 CMS (Cloud Managed System) je platforma pro správu sítě. Slouží jako integrované řešení pro provoz a údržbu. Platforma umožňuje **centralizovanou správu**, vizuální monitoring a inteligentní údržbu různých síťových prvků, jako jsou jednotky ONT, OLT, switche a routery.

### Flexibilní správa různých zařízení

Systém je navržen tak, aby zvládal zařízení různých výrobců a různé komunikační protokoly:

* **OLT Management:** Zařízení C-DATA jsou spravována přes protokol **MQTT**, zatímco OLT třetích stran (např. Huawei, ZTE nebo V-SOL) lze spravovat přes **SNMP**

* **ONT Management:** CMS umožňuje přímou správu ONT přes **TR-069** nebo nepřímou správu skrze OLT pomocí protokolu [**OMCI**](#omci)

### Vizuální monitoring a diagnostika

CMS poskytuje **grafické rozhraní** pro sledování stavu sítě v reálném čase:
*  Nabízí přehledné **karty zařízení**, kde barvy (zelená, žlutá, červená) signalizují online stav a závažnost případných alarmů
*  Umožňuje sledovat statistiky alarmů, trendy a provádět **automatickou diagnostiku poruch** u koncových uživatelů
*  Poskytuje detailní pohled na výkonnostní metriky, jako je síla optického signálu (RX/TX), vytížení CPU nebo paměti

### Škálovatelnost a dostupnost

Platforma podporuje soukromé nasazení na fyzických strojích i cloudových hostitelích. Je navržena pro horizontální expanzi, což znamená, že dokáže obsloužit prakticky neomezený počet připojených zařízení v závislosti na výkonu serveru.



