## Připojení k OLT

Pro prvotní konfiguraci, kdy zařízení ještě **nemá nastavenou IP adresu** ve vaší síti, můžete využít dvě hlavní metody: bezdrátové připojení (WiFi) nebo konzolový port. Tento návod se zaměřuje na konfiguraci pomocí WiFi a webového GUI.

### Připojení přes Management WiFi

Řada FD1601 je vybavena dedikovaným WiFi rozhraním pro správu.

1. Zapněte OLT a vyhledejte WiFi síť vysílanou zařízením (WiFi se zapne po probuzení displeje stisknutím libovolného tlačítka)
2. Přihlašte se k Wifi, **SSID a heslo naleznete na štítku**
3. Po připojení otevřete prohlížeč a zadejte výchozí IP adresu: `192.168.1.100`
4. Zadejte výchozí uživatelské jméno: `root` a heslo: `admin`

### Nastavení správy přes Uplink port (In-band Management)

Aby bylo možné OLT spravovat z počítače připojeného do stejné datové sítě jako uplinkové porty, musíte vytvořit [virtuální rozhraní VLANIF](GPON/pojmy.md#vlanif).

1. Přejděte do `Configuration -> VLANIF` a u `Vlanif1` klikněte zvolte `Edit`
2. IP mode nastavte na `DHCP`, případně `Static IP` a zadejte požadovanou adresu a masku podstítě
3. V případě statické IP adresy je potřeba nastavit i **Gateway**:
   1. V sekci VLANIF klikněte na tlačítko `Gateway`
   2. Vyberte rozhraní `Vlanif1` a zadejte IP adresu výchozí brány vašeho routeru

> ⚠️ Po dokončení konfigurace vždy klikněte na tlačítko 💾 (Save Config) v horní liště, aby se nastavení v OLT uložilo do trvalé paměti. Bez uložení by se při restartu OLT vrátilo k původnímu nastavení.

### Volitelné: Připojení do CMS (Cloud Management System)

Pokud chcete OLT spravovat centrálně pomocí [platformy CMS](GPON/pojmy.md#cms), musíte povolit vzdálený přístup přes protokol **MQTT**.

1. V prohlížeči přejděte do `Maintenance -> Access Manager`
2. Zapněte přepínač `CMS`
3. Vyplňte adresu **MQTT** serveru, kterou zjistíte v **CMS** v sekci `Admin -> Enterprise` 
4. Po potvrzení by se OLT mělo objevit v seznamu zařízení `Device -> OLT-MQTT -> OLT List` v platformě CMS

# Scénáře použití

## Transparentní režim

Tento návod slouží k nastavení sítě GPON tak, aby se celá optická distribuční síť (ODN) chovala jako transparentní prodloužení vaší lokální sítě. V tomto režimu fungují jednotky ONT v podstatě jako „chytré optické převodníky", které pouze předávají data mezi vaším centrálním routerem a koncovým zařízením bez jakékoliv další manipulace s IP adresami nebo routováním.

### Příprava VLAN a Uplink portu

Nejprve musíte definovat VLAN, která bude sloužit pro přenos dat z vaší lokální sítě, a nastavit uplinkový port tak, aby z něj data odcházela netagovaná (untagged).

1. V menu `Configuration -> VLAN -> Port VLAN` vyberte uplinkový port (např. `GE1`).
2. Nastavte Mode na Access a zadejte ID VLAN ktrou chcete používat pro provoz mezi OLT a ONT (můžete ponechat výchozí `1`). 

> ℹ️ Režim Access na straně OLT funguje tak, že při vstupu netagovaného rámce z vašeho routeru mu OLT přiřadí interní VLAN ID. Při výstupu dat směrem z OLT do routeru pak tento tag automaticky odstraní. Váš router vůbec netuší, že uvnitř optické sítě nějaká VLAN existuje.

### Vytvoření DBA profilu

[DBA profil](GPON/pojmy.md#dba-profil) určuje, jakou šířku pásma budou mít ONT k dispozici v upstreamu.

1. V `Deployment -> Profile -> DBA Profile` a klikněte na `Add`
2. Zvolte [typ profilu](GPON/pojmy.md#typy-šířky-pásma) např. `Max` a nastavte maximální šířku pásma (např. `1 000 000 kbps` pro 1 Gbps)

### Vytvoření Line profilu

Tento profil mapuje provoz z ONT do konkrétní VLAN skrze GEM porty.

1. V sekci `Deployment -> Profile -> Line Profile` klikněte na `Add`
2. Mapping-mode nastavte na `VLAN`
3. Přiřaďte Tcont1 dříve vytvořený DBA profil
4. V části Mapping aktivujte přepínač `VLAN-transparent`

### Vytvoření Service profilu

Zde definujete fyzické parametry portů na ONT.

1. V `Deployment -> Profile -> Service Profile` klikněte na `Add`
2. Počet portů (ETH, POTS atd.) nastavte na `Adaptive` (automatické rozpoznání podle typu ONT)
3. V kroku ONU Port (pro [SFU jednotky](GPON/pojmy.md#hgu-vs-sfu)) nastavte u vybraného portu (např. `ETH1`) Native VLAN na hodnotu vaší VLAN (např. `100`). Tím zajistíte, že netagovaná data z portu ONT dostanou správný tag pro průchod optickou sítí

### Vytvoření WAN profilu

Tento profil řekne ONT, že má fungovat jako bridge, nikoliv jako router.

1. V `Deployment -> Profile -> WAN Profile` klikněte na `Add`
2. V sekci `WAN Configuration` klikněte na `Add` Nastavte Mode na `Bridge`
4. Zaškrtněte porty (např. `LAN1`, `LAN2`), které mají být do tohoto bridge zahrnuty

### Vytvoření Auth Policy a Apply Policy

Aby se nastavení automaticky aplikovalo na připojená ONT, musíte vytvořit Auth Policy a Apply Policy.

1. V `Deployment -> Auth Policy` klikněte na `Create Policy`
2. V sekci Policy vyberte dříve vytvořený Line Profile, Service Profile a WAN Profile
3. Následně v `Deployment -> Apply Policy` klikněte na `Add`
4. Vyberte konkrétní PON port (v případě víceportové OLT můžete vytvořit více politik pro různé porty)
5. Přiřaďte dříve vytvořenou Auth Policy
6. Vyberte ONU authmode `SN`
7. Vyberte prioritu (pokud na jeden port dopadá více politik, OLT upřednostní tu s vyšší prioritou)
8. Vyberte ONU matching-rules (Tento filtr určuje, pro která zařízení je politika určena)
    * `Any` (Jakékoli): Nejdůležitější volba pro hromadné nasazení. Pokud je zaškrtnuto, OLT aplikuje politiku na každou ONU, která se poprvé nahlásí na daném portu, bez ohledu na model nebo výrobce
    * Další možnosti (SN, Vendor ID, Software): Umožňují omezit automatizaci pouze na konkrétní kusy, výrobce (např. pouze C-DATA) nebo konkrétní verze firmwaru

> ✅ Jakmile se nyní jakékoliv ONT připojí k danému PON portu, OLT mu automaticky odešle konfiguraci, která z něj udělá bridge. Provoz z LAN portu ONT bude transparentně přenesen do vaší lokální sítě skrze uplink port OLT.

> ⚠️ Po dokončení konfigurace vždy klikněte na tlačítko 💾 (Save Config) v horní liště, aby se nastavení v OLT uložilo do trvalé paměti. Bez uložení by se při restartu OLT vrátilo k původnímu nastavení.

## ISP

Tento návod vás provede nastavením profilů pro typický scénář: zákazník si zakoupil **100 Mbps symetrickou přípojku**. 

### Vytvoření DBA profilu

[**DBA profil**](GPON/pojmy.md#dba-profil) určuje, jak OLT přiděluje šířku pásma v upstreamu.

1. Přejděte do `Deployment -> Profile -> DBA Profile` a klikněte na `Add`
* **Profile Type**: Zvolte `Max (Type 4)`. Tento typ je ideální pro internetové služby, protože umožňuje dynamicky využívat pásmo až do nastaveného maxima
* **Max Bandwidth**: Zadejte hodnotu `100 000 kbps` (odpovídá 100 Mbps)

### Omezení stahování dat

Zatímco DBA hlídá upstream, [**Traffic profil**](GPON/pojmy.md#traffic-profil) je klíčový pro downstream k zákazníkovi. Zde nastavujeme rychlostní strop a parametry pro plynulost.

1. V menu přejděte na `Deployment -> Profile -> Traffic Profile` a zvolte `Add`
    * **CIR (Committed Information Rate)**: Pro běžný internet můžete nechat na minimum, čímž umožníte agregaci linky
    * **PIR (Peak Information Rate)**: Nastavte na `100 000 kbps`. Toto je maximální rychlost, které může zákazník dosáhnout, pokud je v síti volná kapacita
    * **CBS (Committed Burst Size)**: Určuje objem dat pro **CIR**. Běžně se používá kolem `4 KB` až `16 KB` na každých `64 kbps` **CIR**
    * **PBS (Peak Burst Size)** určuje velikost datového „nárazu“, který systém povolí plnou rychlostí PIR, než začne pakety striktně omezovat. Pro 100 Mbit se doporučuje hodnota v rozmezí `16 000 KB` až `32 000 KB`. Pokud ji nastavíte příliš malou, uživatel pocítí "drhnutí" internetu i při volné lince

### Vytvoření Line profilu

1. V sekci `Deployment -> Profile -> Line Profile` vytvořte nový profil a nastavte **Mapping-mode** na `VLAN`
2. Přiřaďte **Tcont1** váš **DBA Profil**
3. U **Gemport1** zapněte přepínač `Gemport car` a přiřaďte **Traffic Profil** pro downstream (je možné zvolit i traffic profile pro upstream jako dodatečné omezení přímo při vstupu do GEM portu, ale v tomto scénáři to není nutné) 
4. V sekci **Mapping** zvolte User VLAN `Tag` a zadejte ID vaší VLAN (např. ID 100)

### Vytvoření Service profilu

1. Přejděte na `Deployment -> Profile -> Service Profile` a klikněte na `Add`
2. Ponechte porty (ETH, POTS, Wi-Fi) na `Adaptive`
3.  * Pokud používáte [**jednotky typu SFU**](GPON/pojmy.md#hgu-vs-sfu) zde určíte, jak se mají chovat jednotlivé ethernetové zásuvky na jednotce
        1. Přepněte volbu na `Concern`
        2. V tabulce portů vyberte konkrétní port (např. Port 1) a klikněte na `Edit`
        3. Mode: zvolte `SFU`
        4. Native VLAN: Zde zadejte ID vaší internetové VLANy (např. 100). Toto nastavení zajistí, že data, která přijdou ze zákazníkova routeru, dostanou v ONU správný tag.
3.  * Pokud používáte [**jednotky typu HGU**](GPON/pojmy.md#hgu-vs-sfu), přepněte volbu na `Unconcern`

### Vytvoření WAN profilu

Pokud používáte [**ONT typu HGU**](GPON/pojmy.md#hgu-vs-sfu) (s integrovaným routerem), **musíte vytvořit WAN profil**, který určí, jakým způsobem bude jednotka získávat IP adresu pro připojení k internetu. V případě [**SFU**](GPON/pojmy.md#hgu-vs-sfu) se WAN profil **nevytváří**, potřebné nastavení se provede na routeru zákazníka.

1. Přejděte na `Deployment -> Profile -> WAN Profile` a klikněte na tlačítko `Add`
2. Pojmenujte profil a klikněte na `Next`
3. V dalším okně klikněte na `Add` a zapněte volbu `VLAN`. Do pole VLAN ID zadejte číslo VLANy, kterou jste si připravili pro internet (např. 100)
4. Mode: V poli Mode zvolte metodu, jakou zákazník získá IP adresu:
    * **IPoE**: Nejčastější volba, kdy jednotka dostane adresu automaticky přes DHCP
    * **PPPoE**: Potřeba zadat přihlašovací jméno a heslo
5. V poli Service Type nastavte hodnotu `INTERNET`. Tím jednotce řeknete, že tento profil je určen pro běžný datový provoz.
6. **MTU**: Pro IPoE (DHCP) ponechte `1500`, pro PPPoE nastavte `1492`.
7. **Port Binding**: Tento krok je nejčastějším zdrojem chyb. Musíte zaškrtnout fyzické porty a Wi-Fi sítě, na kterých má internet fungovat. Jako univerzální řešení můžete vybrat všechny dostupné LAN porty a Wi-Fi SSID.

### Vytvoření Auth Policy a Apply Policy

1. V `Deployment -> Auth Policy` klikněte na `Create Policy`
2. V sekci Policy vyberte dříve vytvořený Line Profile, Service Profile a WAN Profile
3. Následně v `Deployment -> Apply Policy` klikněte na `Add`
4. Vyberte konkrétní PON port
5. Přiřaďte dříve vytvořenou Auth Policy
6. Vyberte ONU authmode `SN`
7. Vyberte prioritu (pokud na jeden port dopadá více politik, OLT upřednostní tu s vyšší prioritou)
8. Vyberte ONU matching-rules (Tento filtr určuje, pro která zařízení je politika určena)
    * V tomto případě je vhodné použít `SN` a zadat seriové číšlo ONT zákazníka
9. Potvrďte tlačítkem `Confirm`

> ✅ Nyní, jakmile se připojí ONT se schodujícím se seriovým číslem k danému portu, OLT do ní automaticky nahraje nastavení pro 100 Mbps přípojku.

> ⚠️ Po dokončení konfigurace vždy klikněte na tlačítko 💾 (Save Config) v horní liště, aby se nastavení v OLT uložilo do trvalé paměti. Bez uložení by se při restartu OLT vrátilo k původnímu nastavení.

## Konfigurace WiFi 

*TODO: To bude trochu složitější, protože OMCI na to nebude úplně dělané. Takové věci se asi budou muset řešit přes TR-069 aby to bylo alespoň do nějaké míry standardizované. Jinak leda přes web UI na samotné ONT. Zeptat se Číňanů.*

# Deployment profily

## DBA profil

Hlavním účelem DBA profilu je optimalizace využití přenosové kapacity optického vlákna v **upstreamu**. Namísto toho, aby měl každý uživatel pevně vyhrazenou rychlost, kterou nevyužívá neustále, umožňuje DBA profil přidělit nevyužitou kapacitu jiným aktivním uživatelům, čímž zvyšuje celkovou propustnost.

### Typy šířky pásma

V rámci DBA profilu se definují různé typy přenosových kapacit, které odpovídají požadavkům různých služeb:

* **Fixed Bandwidth** (Fix – Type 1): Pevně vyhrazená kapacita, která je uživateli k dispozici neustále, bez ohledu na to, zda ji využívá. Je ideální pro služby extrémně citlivé na zpoždění a jitter, jako je VoIP nebo správa sítě.
* **Assured Bandwidth** (Assure – Type 2/3): Garantovaná šířka pásma, kterou OLT poskytne ONT kdykoliv o ni požádá. Pokud ONT data neposílá, může být tato kapacita dočasně uvolněna pro jiný provoz.
* **Maximum Bandwidth** (Max – Type 4): Horní limit (tzv. "best-effort"), který může ONT využít, pokud je v síti aktuálně volná kapacita. Tato rychlost není garantována a závisí na celkovém vytížení portu.
* **Kombinované typy** (Assure & Max / Fix & Assure & Max): Umožňují kombinovat pevnou (Fix) garantovanou (Assure) a maximální (Max) rychlost, což je nejčastější nastavení pro běžné internetové tarify.

[GEM porty](#gem-port) v rámci [T-CONT](#t-cont) sdílí tuto přidělenou šířku pásma.

## Traffic profil

Nastavení Traffic profilu je klíčovým krokem pro řízení šířky pásma v **downstreamu**. Zatímco [DBA profil](#dba-profil) se stará o upstream, Traffic profil definuje pravidla pro stahování dat.

### Parametry

* **CIR** (Committed Information Rate): Garantovaná přenosová rychlost
  * Určuje minimální šířku pásma, kterou má zákazník (nebo služba) vždy k dispozici. OLT se snaží zajistit, aby tento objem dat prošel sítí i v případě vysokého vytížení linky.
  * Ideální pro kritické služby jako VoIP (hlas) nebo IPTV, kde by kolísání rychlosti způsobilo výpadky.
* **PIR** (Peak Information Rate): Maximální (špičková) přenosová rychlost
  * Definuje absolutní strop, který nesmí datový tok překročit. Je to součet garantované rychlosti (CIR) a "nadbytečné" rychlosti, kterou může OLT přidělit, pokud je v síti zrovna volná kapacita
* **CBS** (Committed Burst Size): Garantovaná velikost dávky dat
  * Určuje objem dat, který může být přenesen rychlostí vyšší než CIR po velmi krátkou dobu, aniž by došlo k zahazování paketů. Pomáhá vyhlazovat drobné výkyvy v provozu.
* **PBS** (Peak Burst Size): Maximální velikost dávky dat.
  * Podobné jako CBS, ale vztahuje se k limitu PIR. Určuje, kolik dat může "proletět" špičkovou rychlostí v jednom okamžiku (burst). Jakmile je tento limit vyčerpán, OLT začne pakety nad rámec PIR nekompromisně zahazovat nebo označovat nižší prioritou.

GPON využívá sdílené médium. OLT musí přesně vědět, kolik dat může do kterého GEM portu „pustit“, aby jeden stahující zákazník nezahltil celou větev pro ostatních 127 sousedů na stejném portu. Díky Traffic profilu na straně downstreamu a DBA profilu na straně uplinku máte plnou kontrolu nad obousměrným provozem v síti.

## Line profil

### Global Setting

* **Profile Name:** Název profilu 
* **Type:** Typ technologie (`gpon`)
* **Mapping-mode:** Určuje, podle čeho bude OLT řadit data do kanálů. Nejrozšířenějším standardem je režim `VLAN`

### Tcont

T-CONT je virtuální kontejner, který reprezentuje přidělenou kapacitu pro nahrávání dat (upstream).
* **Tcont1**: Výchozí předvytvořený kontejner. Je možné vytvořit další kliknutím na `+`
* **DBA**: Ke každému T-CONTu je potřeba přiřadit **DBA profil**

### Gemport

* **Gemport1:** Výchozí předvytvořený gemport. Je možné vytvořit další kliknutím na `+`
* **Gemport Car:** Možnost zapnout funkci omezování rychlosti na úrovni GEM portu (shaping/policing). Zde je možné přiřadit **Traffic profil** pro downstream i upstream

### Mapping

Zde se definují pravidla, která říkají: „Data z této VLANy patří do tohoto GEM portu“.

* **Mapping1**: Výchozí mapping pro výchozí Gemport1. Pro další *Gemporty* je možné vytvořit další *Mapping* kliknutím na `+`. *Mapping1* odpovídá *Gemport1*, *Mapping2* odpovídá *Gemport2* atd.
* **VLAN-transparent:** Pokud je zapnuté, OLT nebude VLAN značky kontrolovat a propustí vše tak, jak to přichází do odpovídajícího GEM portu
* **Tag / Untag:** Volba, zda mají data do ONU přicházet s VLAN značkou (Tag) nebo mají být beze značky (Untag)
* **User VLAN:** Nasavení jaké VLAN ID, má být mapováno na odpovídající GEM port.

## Service profil

### Basic Information

* **Profile Name:** Název profilu
* **Loopback Detection:** Zapnutí detekce smyček přímo na ONT. Pokud zákazník způsobí zasmyčkování provozu, jednotka port zablokuje
* **ONU Capability Planning:**
    * Definice počtu portů: je monžé nastavit pevný počet portů podle vašeho modelu ONT, nebo **zvolit Adaptive**, OLT tak automaticky detekuje počet portů podle připojené jednotky

### IP Host Configuration

Slouží k definici IP rozhraní přímo pro management daného ONT, pokud k němu chcete přistupovat jiným způsobem než přes OMCI/TR-069

* **VLAN**: identifikační číslo VLANy, ve které má ONU komunikovat se správcovským serverem ACS
* **Priority**: 802.1p priorita tohoto rozhraní
* **Mode**:
    * **DHCP**: Získání iP adresy automaticky přes DHCP server ve vaší síti
    * **Static IP**: Zadání pevné IP adresy, masky a výchozí brány

### ONU Port

Tato část je nejdůležitější pro jednotky typu **SFU (Bridge)**.

* **Port Config**
    * **Unconcern:** Nastavujeme u **HGU** jednotek. OLT pak VLANy na portech ignoruje, budou řešeny ve WAN profilu
    * **Concern:** Nastavujeme u **SFU** jednotek. Tím se aktivuje možnost editovat konkrétní porty

* **Port Config -> `Edit`**
    * **Mode:** 
        * **Unconcern** *TODO: nepodařilo se mi zjistit co tohle dělá*
        * **HGU**
        * **SFU**
    * **Native VLAN:** VLAN ID, kterým se automaticky označí neoznačená data přicházející na tento port
    * **Native VLAN Priority:** Nastavení 802.1p priority (0–7)
    * **Bandwidth Control:** Zde můžeš zapnout **Ingress** (vstupní) a **Egress** (výstupní) limitaci rychlosti přímo na daném fyzickém LAN portu

* **Port VLAN Rules -> `Edit` -> VLAN Mode**
    * **Transparent**  *TODO: doplnit*
    * **Trunk**
    * **Translation**
    * **QinQ**

### ONU Multicast

Pokud vaše síť šíří televizi přes multicast, zde najdete pořebné nastavení.

*   **ONU Multicast:** Zapnutí/vypnutí multicastu pro daný profil.
*   **Multicast Mode:** Na výběr je **Snooping** (ONU sleduje IGMP zprávy), **Proxy** nebo **Unconcern**. *TODO: vysvětlit co to znamená*
*   **Fast-leave:** Při zapnutém ONU (*TODO: ONU nebo ONT?*) okamžitě přestane posílat data kanálu, jakmile zákazník přepne na jiný. To šetří kapacitu optické linky.
*   **Multicast Rules Configuration -> `Add`:**
    *   **Port:** Na kterém LAN portu má IPTV fungovat
    *   **Multicast VLAN ID:** Číslo VLAN, ve které teče TV stream.
    *   **Multicast IP type:** **TODO**
    *   **IGMP-Forward / Multicast-Forward:** **TODO**

## TR-069 profil

## WAN profil



