# LAB-7

## Построение EVPN active/active multihoming
---
### Схема связи и адресное пространство
Схема и адресное пространство взято из LAB-1.
![img_7.png](screenshots/Lab-7.JPG)

Подключаем коммутатор к Leaf2 и Leaf3 и поднимаем агрегированный канал.

Конфигурация агрегированного канала на коммутаторе
```text
Switch#sh run int po1
interface Port-channel1
 switchport trunk allowed vlan 100,200
 switchport trunk encapsulation dot1q
 switchport mode trunk
end

Switch#sh run int e0/0 
interface Ethernet0/0
 switchport trunk allowed vlan 100,200
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-protocol lacp
 channel-group 1 mode active
end

Switch#sh run int e0/1
interface Ethernet0/1
 switchport trunk allowed vlan 100,200
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-protocol lacp
 channel-group 1 mode active
end

Switch#sh run int vl100
interface Vlan100
 ip address 192.168.254.100 255.255.255.0
end

Switch#sh run int vl200
interface Vlan200
 ip address 192.168.253.200 255.255.255.0
 shutdown
end
```
Настройка EVPN active/active multihoming на Leaf2, Leaf3 
```text
root@Leaf2> show configuration | display set 
set chassis aggregated-devices ethernet device-count 1

set interfaces ge-0/0/3 gigether-options 802.3ad ae0

set interfaces ae0 vlan-tagging
set interfaces ae0 encapsulation flexible-ethernet-services
set interfaces ae0 esi 00:12:12:12:12:12:12:12:12:12
set interfaces ae0 esi all-active
set interfaces ae0 aggregated-ether-options lacp active
set interfaces ae0 aggregated-ether-options lacp system-id 12:12:12:12:12:12
set interfaces ae0 unit 100 encapsulation vlan-bridge
set interfaces ae0 unit 100 vlan-id 100
set interfaces ae0 unit 200 encapsulation vlan-bridge
set interfaces ae0 unit 200 vlan-id 200

root@Leaf3> show configuration | display set
set chassis aggregated-devices ethernet device-count 1

set interfaces ge-0/0/3 gigether-options 802.3ad ae0
set interfaces ae0 vlan-tagging
set interfaces ae0 encapsulation flexible-ethernet-services
set interfaces ae0 esi 00:12:12:12:12:12:12:12:12:12
set interfaces ae0 esi all-active
set interfaces ae0 aggregated-ether-options lacp active
set interfaces ae0 aggregated-ether-options lacp system-id 12:12:12:12:12:12
set interfaces ae0 unit 100 encapsulation vlan-bridge
set interfaces ae0 unit 100 vlan-id 100
set interfaces ae0 unit 200 encapsulation vlan-bridge
set interfaces ae0 unit 200 vlan-id 200
```
Проверка агрегации
```text

На коммутаторе

Switch#sh etherchannel 1 summary 
Number of channel-groups in use: 1
Number of aggregators:           1

Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)         LACP      Et0/0(P)    Et0/1(P)    

На Leaf2 и Leaf3

root@Leaf2> show lacp interfaces extensive 
Aggregated interface: ae0
    LACP state:       Role   Exp   Def  Dist  Col  Syn  Aggr  Timeout  Activity
      ge-0/0/3       Actor    No    No   Yes  Yes  Yes   Yes     Fast    Active
      ge-0/0/3     Partner    No    No   Yes  Yes  Yes   Yes     Slow    Active
    LACP protocol:        Receive State  Transmit State          Mux State 
      ge-0/0/3                  Current   Slow periodic Collecting distributing
    LACP info:        Role     System             System       Port     Port    Port 
                             priority         identifier   priority   number     key 
      ge-0/0/3       Actor        127  12:12:12:12:12:12        127        1       1
      ge-0/0/3     Partner      32768  aa:bb:cc:80:05:00      32768        1       1

root@Leaf3> show lacp interfaces extensive 
Aggregated interface: ae0
    LACP state:       Role   Exp   Def  Dist  Col  Syn  Aggr  Timeout  Activity
      ge-0/0/3       Actor    No    No   Yes  Yes  Yes   Yes     Fast    Active
      ge-0/0/3     Partner    No    No   Yes  Yes  Yes   Yes     Slow    Active
    LACP protocol:        Receive State  Transmit State          Mux State 
      ge-0/0/3                  Current   Slow periodic Collecting distributing
    LACP info:        Role     System             System       Port     Port    Port 
                             priority         identifier   priority   number     key 
      ge-0/0/3       Actor        127  12:12:12:12:12:12        127        1       1
      ge-0/0/3     Partner      32768  aa:bb:cc:80:05:00      32768        2       1
```