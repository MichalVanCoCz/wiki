Technologie **GPON** (Gigabit Passive Optical Network) je dnes nejrozšířenějším standardem pro optické sítě typu FTTH (Fiber to the Home). Její hlavní výhodou je **pasivní charakter** - mezi centrálním bodem a zákazníkem nejsou žádné aktivní prvky vyžadující napájení, pouze optické splittery.

## Architektura a princip fungování

GPON využívá topologii bod-více bodů (P2MP). Centrální prvek **OLT** (Optical Line Terminal) komunikuje s koncovými jednotkami **ONT** (Optical Network Terminal, někdy také označované ONU) přes pasivní optickou distribuční síť (ODN).

* **Downstream (OLT -> ONT):** Data jsou vysílána všem ONT současně na vlnové délce **1490 nm** . Každé ONT si na základě ID vybere pouze data určená pro něj.
* **Upstream (ONT -> OLT):** Aby nedošlo ke kolizi dat z různých ONT, používá se metoda **TDMA** (Time Division Multiple Access). OLT přiděluje každému ONT přesný časový slot, kdy může vysílat na vlnové délce **1310 nm**.

![GPON topologie](https://upload.wikimedia.org/wikipedia/commons/b/ba/GPON_topology.png)
