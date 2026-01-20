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

> ⚠️ Po dokončení konfigurace v CMS vždy klikněte na tlačítko 💾 (Save Config) v horní liště, aby se nastavení v OLT uložilo do trvalé paměti. Bez uložení by se při restartu OLT vrátilo k původnímu nastavení.

### Volitelné: Připojení do CMS (Cloud Management System)

Pokud chcete OLT spravovat centrálně pomocí [platformy CMS](GPON/pojmy.md#cms), musíte povolit vzdálený přístup přes protokol **MQTT**.

1. V prohlížeči přejděte do `Maintenance -> Access Manager`
2. Zapněte přepínač `CMS`
3. Vyplňte adresu **MQTT** serveru, kterou zjistíte v **CMS** v sekci `Admin -> Enterprise` 
4. Po potvrzení by se OLT mělo objevit v seznamu zařízení `Device -> OLT-MQTT -> OLT List` v platformě CMS

# Scénáře použití

## Transparentní režim

Tento návod slouží k nastavení sítě GPON tak, aby se celá optická distribuční síť (ODN) chovala jako transparentní prodloužení vaší lokální sítě (L2 bridge). V tomto režimu fungují jednotky ONT v podstatě jako „chytré optické převodníky", které pouze předávají data mezi vaším centrálním routerem a koncovým zařízením zákazníka bez jakékoliv další manipulace s IP adresami nebo routováním.

### Příprava VLAN a Uplink portu

Nejprve musíte definovat VLAN, která bude sloužit pro přenos dat z vaší lokální sítě, a nastavit uplinkový port tak, aby z něj data odcházela netagovaná (untagged).

1. V menu `Configuration -> VLAN -> Port VLAN` vyberte uplinkový port (např. `GE1`).
2. Nastavte Mode na Access a zadejte ID vaší servisní VLAN (např. `100`). V tomto režimu OLT při výstupu dat k vašemu routeru odstraní VLAN tag.

> ℹ️ Režim Access na straně OLT funguje tak, že při vstupu netagovaného rámce z vašeho routeru mu OLT přiřadí interní VLAN ID. Při výstupu dat směrem z OLT do routeru pak tento tag automaticky odstraní. Váš router vůbec netuší, že uvnitř optické sítě nějaká VLAN existuje.

### Vytvoření DBA profilu

[DBA profil](GPON/pojmy.md#dba-profil) určuje, jakou šířku pásma budou mít ONT k dispozici v upstreamu.

1. V `Deployment -> Profile -> DBA Profile` a klikněte na `Add`
2. Zvolte [typ profilu](GPON/pojmy.md#typy-šířky-pásma) např. `Max` a nastavte maximální šířku pásma (např. `1000000 kbps` pro 1 Gbps)

### Vytvoření Line profilu

Tento profil mapuje provoz z ONT do konkrétní VLAN skrze GEM porty.

1. V sekci `Deployment -> Profile -> Line Profile` klikněte na `Add`
2. Mapping-mode nastavte na `VLAN`
3. Přiřaďte Tcont1 dříve vytvořený DBA profil
4. V části Mapping aktivujte přepínač `VLAN-transparent`

### Vytvoření Service profilu (Servisní profil)

Zde definujete fyzické parametry portů na ONT.

1. V `Deployment -> Profile -> Service Profile` klikněte na `Add`
2. Počet portů (ETH, POTS atd.) nastavte na `Adaptive` (automatické rozpoznání podle typu ONT)
3. V kroku ONU Port (pro SFU jednotky) nastavte u vybraného portu (např. `ETH1`) Native VLAN na hodnotu vaší VLAN (např. `100`). Tím zajistíte, že netagovaná data z portu ONT dostanou správný tag pro průchod optickou sítí

### Vytvoření WAN profilu (Bridge)

Tento profil řekne ONT, že má fungovat jako bridge, nikoliv jako router.

1. V `Deployment -> Profile -> WAN Profile` klikněte na `Add`
2. V sekci `WAN Configuration` klikněte na `Add` Nastavte Mode na `Bridge`
4. Zaškrtněte porty (např. `LAN1`, `LAN2`), které mají být do tohoto bridge zahrnuty

### Vytvoření a aplikace politiky (Auth Policy, Apply Policy)

Aby se nastavení automaticky aplikovalo na připojená ONT, musíte vytvořit politiku.

1. V `Deployment -> Auth Policy` klikněte na `Create Policy`
2. V sekci Policy vyberte dříve vytvořený Line Profile, Service Profile a WAN Profile
3. Následně v `Deployment -> Apply Policy` klikněte na `Add`
4. Vyberte konkrétní PON port (v případě víceportové OLT můžete vytvořit více politik pro různé porty)
5. Přiřaďte dříve vytvořenou Auth Policy
6. Vyberte ONU authmode 
    * *TODO: doplnit co to vlastně dělá, nepodařilo se mi zjistit*
7. Vyberte prioritu (pokud na jeden port dopadá více politik, OLT upřednostní tu s vyšší prioritou)
8. Vyberte ONU matching-rules (Tento filtr určuje, pro která zařízení je politika určena)
    * `Any` (Jakékoli): Nejdůležitější volba pro hromadné nasazení. Pokud je zaškrtnuto, OLT aplikuje politiku na každou ONU, která se poprvé nahlásí na daném portu, bez ohledu na model nebo výrobce
    * Další možnosti (SN, Vendor ID, Software): Umožňují omezit automatizaci pouze na konkrétní kusy, výrobce (např. pouze C-DATA) nebo konkrétní verze firmwaru

> ✅ Jakmile se nyní jakékoliv ONT připojí k danému PON portu, OLT mu automaticky odešle konfiguraci, která z něj udělá bridge. Provoz z LAN portu ONT bude transparentně přenesen do vaší lokální sítě skrze uplink port OLT.

> ⚠️ Po dokončení konfigurace v CMS vždy klikněte na tlačítko 💾 (Save Config) v horní liště, aby se nastavení v OLT uložilo do trvalé paměti. Bez uložení by se při restartu OLT vrátilo k původnímu nastavení.

## ISP

Tento návod vás provede nastavením profilů pro typický scénář: zákazník si zakoupil **100 Mbps symetrickou přípojku**. 

### Omezení odesílání dat

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

### Mapování provozu

1. V sekci `Deployment -> Profile -> Line Profile` vytvořte nový profil a nastavte **Mapping-mode** na `VLAN`
2. Přiřaďte **Tcont1** váš **DBA Profil**
3. U **Gemport1** zapněte přepínač `Gemport car` a přiřaďte **Traffic Profil** pro downstream (je možné zvolit i traffic profile pro upstream jako dodatečné omezení přímo při vstupu do GEM portu, ale v tomto scénáři to není nutné) 
4. V sekci **Mapping** zvolte User VLAN `Tag` a zadejte ID vaší VLAN (např. ID 100)

### Schopnosti jednotky

1. Přejděte na `Deployment -> Profile -> Service Profile` a klikněte na `Add`
2. Ponechte porty (ETH, POTS, Wi-Fi) na `Adaptive`
3. Pokud používáte **jednotky typu Bridge** (SFU), nastavte v sekci `Port Config` u portu ETH1 hodnotu `Native VLAN` na ID vaší internetové VLANy (např. 100). Tím zajistíte, že zákaznický router dostane data bez tagu. *TODO: tady to nesedí, takhle tam to nastavení nevypadá, pravděpodobně s tam někde bude volit i NAT*

### WAN profile (pro jednotky typu HGU)

1. Přejděte na `Deployment -> Profile -> WAN Profile` a klikněte na tlačítko `Add`
2. Pojmenujte profil a klikněte na `Next`
3. V dalším okně klikněte na `Add` a zapněte volbu `VLAN`. Do pole VLAN ID zadejte číslo VLANy, kterou jste si připravili pro internet (např. 100)
4. Výběr protokolu (Mode): V poli Mode zvolte metodu, jakou zákazník získá IP adresu:
    * **IPoE**: Nejčastější volba, kdy jednotka dostane adresu automaticky přes DHCP.
    * **PPPoE**: Zvolte, pokud vyžadujete přihlašovací jméno a heslo.
5. Určení typu služby (Service Type): V poli Service Type nastavte hodnotu INTERNET. Tím jednotce řeknete, že tento profil je určen pro běžný datový provoz.
6. **MTU**: Pro IPoE (DHCP) ponechte 1500, pro PPPoE nastavte 1492.
7. **Port Binding**: Tento krok je nejčastějším zdrojem chyb. Musíte zaškrtnout fyzické porty a Wi-Fi sítě, na kterých má internet fungovat. Jako univerzální řešení můžete vybrat všechny dostupné LAN porty a Wi-Fi SSID.

### Automatizace a aktivace

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

> ⚠️ Po dokončení konfigurace v CMS vždy klikněte na tlačítko 💾 (Save Config) v horní liště, aby se nastavení v OLT uložilo do trvalé paměti. Bez uložení by se při restartu OLT vrátilo k původnímu nastavení.

## Konfigurace WiFi 

Vzdálená konfigurace WiFi na ONT jednotkách