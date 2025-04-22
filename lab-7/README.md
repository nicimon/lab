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