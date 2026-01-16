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

> ⚠️ Pozor: Po změně konfigurace je potřeba nastavení uložit tlačítkem 💾 (Save Config) v horní liště, jinak se po restartu OLT změny ztratí

### Volitelné: Připojení do CMS (Cloud Management System)

Pokud chcete OLT spravovat centrálně pomocí [platformy CMS](GPON/pojmy.md#cms), musíte povolit vzdálený přístup přes protokol **MQTT**.

1. V prohlížeči přejděte do `Maintenance -> Access Manager`
2. Zapněte přepínač `CMS`
3. Vyplňte údaje serveru, které zjistíte v administraci CMS v sekci Admin-Enterprise. (TODO: ověřit)
4. Po potvrzení by se OLT mělo objevit v seznamu zařízení (Device-OLT-OLT List) v platformě CMS.

