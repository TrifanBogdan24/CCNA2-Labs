# Sheet Packet Tracer

*Cuprins*:
- [Sheet Packet Tracer](#sheet-packet-tracer)
  - [Rutare](#rutare)
  - [VLAN](#vlan)
    - [Voice VLAN](#voice-vlan)
      - [Voice VLAN (Alone)](#voice-vlan-alone)
      - [Voice VLAN \& Data VLANs](#voice-vlan--data-vlans)
  - [Inter-VLAN routing](#inter-vlan-routing)
    - [Legacy](#legacy)
    - [RoaS (Router-on-a-stick)](#roas-router-on-a-stick)
    - [MultiLayer (Layer 3) Switch](#multilayer-layer-3-switch)
  - [STP (Spanning Tree Protocol)](#stp-spanning-tree-protocol)
  - [EtherChannel](#etherchannel)
    - [PAgP (**Port Aggregation Protocol**)](#pagp-port-aggregation-protocol)
    - [LACP (**Link Aggregation Control Protocol**)](#lacp-link-aggregation-control-protocol)
  - [DHCP (**Dynamic Host Configuration Protocol**)](#dhcp-dynamic-host-configuration-protocol)
    - [DHCPv4](#dhcpv4)
    - [DHCPv6](#dhcpv6)
  - [FHRP (**First-hop redundancy protocol**)](#fhrp-first-hop-redundancy-protocol)
    - [HSRP (**Hot Standby Router Protocol**)](#hsrp-hot-standby-router-protocol)
    - [GLBP (**Gateway Load Balancing Protocol**)](#glbp-gateway-load-balancing-protocol)

## Rutare

- Rutare IPv4:
```sh
Router(config)# ip route network-address subnet-mask { ip-address | exit-intf [ip address]} <administrative-distance>
```

- Rutare IPv6:
```sh
Router(config)#ipv6 route network-address/prefix-length{ ipv6-address | exit-intf [ipv6 address]} <administrative-distance>
```

## VLAN

```sh
Switch(config)# vlan 10
Switch(config-vlan)# name Vanzari
Switch(config-vlan)# exit
! Nu este nevoie de `write`
```

Tipuri de port-uri:
- **Access**:
```sh
Switch(config)# interface fa0/2
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
```

- **Trunk**:
```sh
Switch(config)# interface fa0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk allowed vlan 10,20,30
! ATENTIE: fara spatii dupa virgule
```


- **Dynamic** - foloseste **DTP** (Dynamic Trunking Protocol)
    pentru determinarea tipului de interfata
```sh
Switch(config)# interface fa0/1
Switch(config-if)# switchport mode dynamic <desirable|auto>
```


### Voice VLAN


VLAN-ul de voce poate fi configurat alaturi de mai multe VLAN-uri de date.

#### Voice VLAN (Alone)


- Template:
```sh
Switch(config)# vlan <VOICE_VLAN_ID>
Switch(config-vlan)# name <VOICE_VLAN_NAME>
Switch(config-vlan)# exit

Switch(config)# interface <INTERFACE>
Switch(config-if)# switchport mode access
Switch(config-if)# mls qos trust cos
Switch(config-if)# switchport voice vlan <VOICE_VLAN_ID>
```

- Example:
```sh
Switch(config)# vlan 150
Switch(config-vlan)# name voice
Switch(config-vlan)# exit

Switch(config)# interface Fa0/1
Switch(config-if)# switchport mode access
Switch(config-if)# mls qos trust cos
Switch(config-if)# switchport voice vlan 150
```

#### Voice VLAN & Data VLANs

Pe un port configurat cu:
- `switchport mode access`
- si `switchport voice vlan`
nu va exista **niciodata** mai multe de un singur VLAN de date (`access vlan`).

> Pe un port cu VLAN de voce, putem configura maxim un singur VLAN de date.

- Template:
```sh
! VLAN-ul de date:
Switch(config)# vlan <DATA_VLAN_ID>
Switch(config-vlan)# name <DATA_VLAN_NAME>
Switch(config-vlan)# exit

! VLAN-ul de voce:
Switch(config)# vlan <VOICE_VLAN_ID>
Switch(config-vlan)# name <VOICE_VLAN_NAME>
Switch(config-vlan)# exit

Switch(config)# interface <INTERFACE>
Switch(config-if)# switchport mode access
! VLAN-ul de date:
Switch(config-if)# switchport access vlan <DATA_VLAN_ID>
! VLAN-ul de voce:
Switch(config-if)# mls qos trust cos
Switch(config-if)# switchport voice vlan <VOICE_VLAN_ID>
```

- Example:
```sh
! VLAN-ul de date:
Switch(config)# vlan 20
Switch(config-vlan)# name student
Switch(config-vlan)# exit

! VLAN-ul de voce:
Switch(config)# vlan 150
Switch(config-vlan)# name voice
Switch(config-vlan)# exit

Switch(config)# interface Fa0/1
Switch(config-if)# switchport mode access
! VLAN-ul de date:
Switch(config-if)# switchport access vlan 20
! VLAN-ul de voce:
Switch(config-if)# mls qos trust cos
Switch(config-if)# switchport voice vlan 150
```

---

Abrevieri:
- **MLS**: Multi-Layer Switching
- **QoS**: Qualtiy of Service
- **CoS**: Cost of Service (marcheaza prioritatea cadrului)


---

`mls qos` este comanda folosita pentru a activa functionalitata QoS pe un **switch**.


## Inter-VLAN routing

### Legacy

> Cate o interfata pentru fiecare VLAN.

- Template:
```sh
Router(config)# interface <INTERFACE_1>
Router(config-if)# ip addr <IP_ADDR_1> <DECIMAL_MASK_1>
Router(config-if)# no shut
Router(config-if)# interface <INTERFACE_2>
Router(config-if)# ip addr <IP_ADDR_2> <DECIMAL_MASK_2>
Router(config-if)# no shut
Router(config-if)# interface <INTERFACE_3>
Router(config-if)# ip addr <IP_ADDR_3> <DECIMAL_MASK_3>
Router(config-if)#no shut
```

- Example:
```sh
Router(config)# interface gig0/0
Router(config-if)# ip addr 10.10.0.1 255.255.255.0
Router(config-if)# no shut
Router(config-if)# interface gig1/0
Router(config-if)# ip addr 20.20.20.1 255.255.255.0
Router(config-if)# no shut
Router(config-if)# interface gig2/0
Router(config-if)# ip addr 30.30.30.1 255.255.255.0
Router(config-if)#no shut
```


### RoaS (Router-on-a-stick)

> Good practice:
> se recomandă ca ID-ul sub-interfeței logice 
> să corespundă cu ID-ul VLAN-ului asociat.

Comanda `no shut` (de pornire a interfetei)
se aplica la nivelul **interfetei fizice**.


- Template:
```sh
Router(config)# interface <INTERFACE>.<SUB_INTERFACE_ID_1>
Router(config-subif)# encapsulation dot1q <VLAN_ID>
Router(config-subif)# ip addr <IP_ADDR_1> <DECIMAL_MASK_1>

Router(config-subif)# interface <INTERFACE>.<SUB_INTERFACE_ID_2>
Router(config-subif)# encapsulation dot1q <VLAN_ID>
Router(config-subif)# ip addr <IP_ADDR_2> <DECIMAL_MASK_2>

Router(config-subif)# interface <INTERFACE>
Router(config-if)# no shut
```

- Example:
```sh
Router(config)# interface gig0/0.10
Router(config-subif)# encapsulation dot1q 10
Router(config-subif)# ip addr 10.10.10.1 255.255.255.0

Router(config-subif)# interface gig0/0.30
Router(config-subif)# encapsulation dot1q 30
Router(config-subif)# ip addr 30.30.30.1 255.255.255.0

Router(config-subif)# interface gig0/0
Router(config-if)# no shut
```


### MultiLayer (Layer 3) Switch

- **Routed port**:
  - Template:
```sh
Sw-L3(config)# ip routing
Sw-L3(config)# interface <INTERFACE>
Sw-L3(config-if)# no switchport
Sw-L3(config-if)# ip address <IP_ADDR> <DECIMAL_MASK>
Sw-L3(config-if)# no sh
```
  - Example: 
```sh
Sw-L3(config)# ip routing
Sw-L3(config)# interface fa0/0
Sw-L3(config-if)# no switchport
Sw-L3(config-if)# ip address 192.168.0.1 255.255.255.0
Sw-L3(config-if)# no sh
```

- **SVI** (Switch Virtual Interface):
  - Template:
```sh
Sw-L3(config)# vlan <VLAN_ID>
Sw-L3(config-vlan)# interface vlan <VLAN_ID>
Sw-L3(config-if)# ip addr <IP_ADDR> <DECIMAL_MASK>
Sw-L3(config-if)# no sh
Sw-L3(config-if)# exit

Sw-L3(config)# int <INTERFACE>
Sw-L3(config-if)# sw mode access
Sw-L3(config-if)# sw access vlan <VLAN_ID>
```
  - Example: 
```sh
Sw-L3(config)# vlan 10
Sw-L3(config-vlan)# interface vlan 10
Sw-L3(config-if)# ip addr 10.10.10.1 255.255.255.0
Sw-L3(config-if)# no sh
Sw-L3(config-if)# exit

Sw-L3(config)# int fa0/1
Sw-L3(config-if)# sw mode access
Sw-L3(config-if)# sw access vlan 10



Sw-L3(config)# vlan 30
Sw-L3(config-vlan)# interface vlan 30
Sw-L3(config-if)# ip addr 30.30.30.1 255.255.255.0
Sw-L3(config-if)# no sh
Sw-L3(config-if)# exit

Sw-L3(config)# int fa0/3
Sw-L3(config-if)# sw mode access
Sw-L3(config-if)# sw access vlan 30
```



## STP (Spanning Tree Protocol)

```sh
Sw(config)# spanning-tree mode <pvst|rapid-pvst>

! Setare prioritate manual:
Sw(config)# spanning-tree vlan <VLAN_ID> priority <PRIORITY>

! Setare prioritate automat:
Sw(config)# spanning-tree vlan <VLAN_ID> root <primary|secondary>
```


```sh
! Modificare cost:
Sw(config-if)# spanning-tree cost <COST>

! Trecere directa in starea forwarding (doar pentru legaturi access)
Sw(config-if)# spanning-tree portfast

! BPDU guard (nu se trimit update-uri pe link-uri access):
Sw(config)# spanning-tree bpduguard enable
```


## EtherChannel

Comenzi de afisare/debug:
```sh
Sw# show etherchannel summary

! Template:
SW1# show interfaces port-channel <CHANNEL_GROUP_ID>
! Example:
SW1#show interfaces port-channel 1


SW1# show etherchannel summary
SW1# show etherchannel port-channel

! Template:
SW1# show interfaces <INTERFACE> etherchannel
! Example:
SW1#show interfaces fa0/15 etherchannel
```

### PAgP (**Port Aggregation Protocol**)

- Template:
```sh
SW1(config)# interface <INTERFACE_1>
SW1(config-if)# switchport mode trunk
SW1(config-if)# channel-group <CHANNEL_GROUP_ID> mode <desirable|auto>

SW1(config-if)# interface <INTERFACE_2>
SW1(config-if)# switchport mode trunk
SW1(config-if)# channel-group <CHANNEL_GROUP_ID> mode <desirable|auto>
```

- Example:
```sh
SW1(config)# interface FastEthernet0/13
SW1(config-if)# switchport mode trunk
SW1(config-if)# channel-group 1 mode <desirable|auto>

SW1(config-if)# interface FastEthernet0/14
SW1(config-if)# switchport mode trunk
SW1(config-if)# channel-group 1 mode <desirable|auto>
```



### LACP (**Link Aggregation Control Protocol**)

- Template:
```sh
SW1(config)# interface <INTERFACE_1>
SW1(config-if)# switchport mode trunk
SW1(config-if)# channel-group <CHANNEL_GROUP_ID> mode <active|passive>

SW1(config-if)# interface <INTERFACE_2>
SW1(config-if)# switchport mode trunk
SW1(config-if)# channel-group <CHANNEL_GROUP_ID> mode <active|passive>
```

- Example:
```sh
SW1(config)# interface FastEthernet0/15
SW1(config-if)# switchport mode trunk
SW1(config-if)# channel-group 1 mode <active|passive>

SW1(config-if)# interface FastEthernet0/16
SW1(config-if)# switchport mode trunk
SW1(config-if)# channel-group 1 mode <active|passive>
```



## DHCP (**Dynamic Host Configuration Protocol**)

### DHCPv4

- Configurare Server DHCPv4:
  - Template:
```sh
! Exclude un interval de adrese IP:
Router1(config)# ip dhcp excluded-address <FIRST_IP_TO_EXCLUDE> <LAST_IP_TO_EXCLUDE>

! Exclude o adresa IP specifica:
Router1(config)# ip dhcp excluded-address <IP_TO_EXCLUDE>

Router1(config)# ip dhcp pool <DHCP_POOL_NAME>
Router1(dhcp-config)# network <IP_NETWORK> <DECIMAL_MASK>
Router1(dhcp-config)# default-router <IP_DEFAULT_GATEWAY>
Router1(dhcp-config)# dns-server <IP_DNS_SERVER>
Router1(dhcp-config)# domain-name <DOMAIN_NAME>
Router1(dhcp-config)# exit
``` 
  - Example: 
```sh
Router1(config)# ip dhcp excluded-address 192.168.10.1 192.168.10.9
Router1(config)# ip dhcp excluded-address 192.168.10.254
Router1(config)# ip dhcp pool LAN-POOL-1
Router1(dhcp-config)# network 192.168.10.0 255.255.255.0
Router1(dhcp-config)# default-router 192.168.0.1
Router1(dhcp-config)# dns-server 192.168.11.5
Router1(dhcp-config)# domain-name example.com
Router1(dhcp-config)# exit
```

- Verificare Server DHCPv4:
```sh
Router1# show ip dhcp binding
```

- Configurare DHCPv4 Relay:
  - Template:
```sh
Router1(config)# interface <INTERFACE>
Router1(config)# ip helper-address <IP_DHCP_SERVER>
Router1(config-if)# exit
``` 
  - Example: 
```sh
Router1(config)# interface g0/0/0
Router1(config)# ip helper-address 192.168.11.6
Router1(config-if)# exit
```

### DHCPv6

- Stateless DHCPv6 - Server
```sh
Router1(config)# ipv6 unicast-routing
Router1(config)# ipv6 dhcp pool IPV6-STATELESS
Router1(config-dhcpv6)# dns-server 2001:db8:acad:1::254
Router1(config-dhcpv6)# domain-name example.com
Router1(config-dhcpv6)# exit
Router1(config)# interface g0/0/1
Router1(config-if)# ipv6 dhcp server IPV6-STATELESS
Router1(config-if)# ipv6 nd other-config-flag
```

- Stateful DHCPv6 - Server
```sh
Router1(config)# ipv6 unicast-routing
Router1(config)# ipv6 dhcp pool IPV6-STATEFUL
Router1(config-dhcpv6)# address prefix 2001:db8:acad:1::/64
Router1(config-dhcpv6)# dns-server 2001:4860:4860::8888
Router1(config-dhcpv6)# domain-name example.com
Router1(config-dhcpv6)# exit
Router1(config)# interface g0/0/1
Router1(config-if)# ipv6 dhcp server IPV6-STATEFUL
Router1(config-if)# ipv6 nd managed-config-flag
Router1(config-if)# ipv6 dhcp nd prefix default no-autoconfig
```

- Stateful DHCPv6 - Client
```sh
Router3(config)# ipv6 unicast-routing
Router3(config)# interface g0/0/1
Router3(config-if)# ipv6 enable
Router3(config-if)# ipv6 address dhcp
Router3(config-if)# do show ipv6 interface brief
```


- Stateful DHCPv5 - Relay
```sh
Router1(config)#interface g0/0/1
Router1(config-if)#ipv6 dhcp relay destination 2001:db8:acad:1::2 g0/0/0
```

- Comenzi de show/debug:
```sh
Router# show ipv6 dhcp binding
Router# show ipv6 dhcp interface
```


## FHRP (**First-hop redundancy protocol**)



### HSRP (**Hot Standby Router Protocol**)

- Template:
```sh
R1(config)# interface <INTERFACE>
R1(config-if)# ip address <PHYSICAL_IP_1> <DECIMAL_MASK>
R1(config-if)# standby <ID_Grupare> ip <VIRTUAL_IP>
R1(config-if)# standby <ID_Grupare> preempt

R2(config)# interface <INTERFACE>
R2(config-if)# ip address <PHYSICAL_IP_2> <DECIMAL_MASK>
R2(config-if)# standby <ID_Grupare> priority <PRIORITY>
R2(config-if)# standby <ID_Grupare> ip <VIRTUAL_IP>
```


- Example:
```sh
R1(config)# interface fa0/0
R1(config-if)# ip address 10.1.1.1 255.255.255.0
R1(config-if)# standby 123 ip 10.1.1.100
R1(config-if)# standby 123 preempt

R2(config)# interface fa0/0
R2(config-if)# ip address 10.1.1.2 255.255.255.0
R2(config-if)# standby 123 priority 90
R2(config-if)# standby 123 ip 10.1.1.100
```


- Comenzi de show/debug:
```sh
Router# show standby brief
```



### GLBP (**Gateway Load Balancing Protocol**)

- Template:
```sh
R1(config)# interface <INTERFACE>
R1(config-if)# ip address <PHYSICAL_IP_1> <DECIMAL_MASK>
R1(config-if)# glbp <ID_Grupare> ip <VIRTUAL_IP>
R1(config-if)# glbp <ID_Grupare> priority <PRIORITY>
R1(config-if)# glbp <ID_Grupare> preempt

R2(config)# interface <INTERFACE>
R2(config-if)# ip address <PHYSICAL_IP_2> <DECIMAL_MASK>
R2(config-if)# glbp <ID_Grupare> priority 110
R2(config-if)# glbp <ID_Grupare> ip <VIRTUAL_IP>
R2(config-if)# glbp <ID_Grupare> preempt

R3(config-if)# int <INTERFACE>
R3(config-if)# glbp <ID_Grupare> ip <VIRTUAL_IP>
```

- Example:
```sh
R1(config)# interface fa 0/0
R1(config-if)# ip address 10.1.1.1 255.255.255.0
R1(config-if)# glbp 123 ip 10.1.1.254
R1(config-if)# glbp 123 priority 120
R1(config-if)# glbp 123 preempt

R2(config)# interface fa 0/0
R2(config-if)# ip address 10.1.1.2 255.255.255.0
R2(config-if)# glbp 123 priority 110
R2(config-if)# glbp 123 ip 10.1.1.254
R2(config-if)# glbp 123 preempt

R3(config-if)# int fa0/0
R3(config-if)# glbp 123 ip 10.1.1.254
```


- Comenzi de show/debug:
```sh
R1# show glbp br
```

