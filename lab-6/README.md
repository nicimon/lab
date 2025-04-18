# LAB-6

## VxLAN. EVPN L3
---
### Схема связи и адресное пространство
Схема и адресное пространство взято из LAB-1

![img_6.png](screenshots/Lab-6.JPG)

Конфигурация MAC-VRF на Leaf1 для symmetric IRB
```text
set routing-instances macvrf-1 instance-type mac-vrf
set routing-instances macvrf-1 protocols evpn encapsulation vxlan
set routing-instances macvrf-1 protocols evpn default-gateway do-not-advertise
set routing-instances macvrf-1 protocols evpn extended-vni-list all
set routing-instances macvrf-1 protocols evpn multicast-mode ingress-replication
set routing-instances macvrf-1 vtep-source-interface lo0.0
set routing-instances macvrf-1 bridge-domains bd-v100 domain-type bridge
set routing-instances macvrf-1 bridge-domains bd-v100 vlan-id 100
set routing-instances macvrf-1 bridge-domains bd-v100 interface ge-0/0/3.0
set routing-instances macvrf-1 bridge-domains bd-v100 routing-interface irb.100
set routing-instances macvrf-1 bridge-domains bd-v100 vxlan vni 10100
set routing-instances macvrf-1 bridge-domains bd-v200 domain-type bridge
set routing-instances macvrf-1 bridge-domains bd-v200 vlan-id 200
set routing-instances macvrf-1 bridge-domains bd-v200 interface ge-0/0/4.0
set routing-instances macvrf-1 bridge-domains bd-v200 routing-interface irb.200
set routing-instances macvrf-1 bridge-domains bd-v200 vxlan vni 10200
set routing-instances macvrf-1 service-type vlan-aware
set routing-instances macvrf-1 route-distinguisher 10.0.1.1:100
set routing-instances macvrf-1 vrf-target target:3:3
```
Конфигурация MAC-VRF на BorderLeaf1 для symmetric IRB
```text
set routing-instances macvrf-1 instance-type mac-vrf
set routing-instances macvrf-1 protocols evpn encapsulation vxlan
set routing-instances macvrf-1 protocols evpn default-gateway do-not-advertise
set routing-instances macvrf-1 protocols evpn extended-vni-list all
set routing-instances macvrf-1 protocols evpn multicast-mode ingress-replication
set routing-instances macvrf-1 vtep-source-interface lo0.0
set routing-instances macvrf-1 bridge-domains bd-v100 vlan-id 100
set routing-instances macvrf-1 bridge-domains bd-v100 interface ge-0/0/3.0
set routing-instances macvrf-1 bridge-domains bd-v100 routing-interface irb.100
set routing-instances macvrf-1 bridge-domains bd-v100 vxlan vni 10100
set routing-instances macvrf-1 bridge-domains bd-v200 vlan-id 200
set routing-instances macvrf-1 bridge-domains bd-v200 interface ge-0/0/4.0
set routing-instances macvrf-1 bridge-domains bd-v200 routing-interface irb.200
set routing-instances macvrf-1 bridge-domains bd-v200 vxlan vni 10200
set routing-instances macvrf-1 service-type vlan-aware
set routing-instances macvrf-1 route-distinguisher 10.0.1.6:100
set routing-instances macvrf-1 vrf-target target:3:3
```
Конфигурация IP VRF на Leaf1 для symmetric IRB
```text
set routing-instances Tenant1 instance-type vrf
set routing-instances Tenant1 protocols evpn irb-symmetric-routing vni 10500
set routing-instances Tenant1 protocols evpn ip-prefix-routes advertise direct-nexthop
set routing-instances Tenant1 protocols evpn ip-prefix-routes encapsulation vxlan
set routing-instances Tenant1 protocols evpn ip-prefix-routes vni 10500
set routing-instances Tenant1 interface irb.100
set routing-instances Tenant1 interface irb.200
set routing-instances Tenant1 interface lo0.10500
set routing-instances Tenant1 route-distinguisher 10.0.1.1:500
set routing-instances Tenant1 vrf-target target:500:500
```
Конфигурация IP VRF на BorderLeaf1 для symmetric IRB
```text
set routing-instances Tenant1 instance-type vrf
set routing-instances Tenant1 protocols evpn irb-symmetric-routing vni 10500
set routing-instances Tenant1 protocols evpn ip-prefix-routes advertise direct-nexthop
set routing-instances Tenant1 protocols evpn ip-prefix-routes encapsulation vxlan
set routing-instances Tenant1 protocols evpn ip-prefix-routes vni 10500
set routing-instances Tenant1 interface irb.100
set routing-instances Tenant1 interface irb.200
set routing-instances Tenant1 interface lo0.10500
set routing-instances Tenant1 route-distinguisher 10.0.1.6:500
set routing-instances Tenant1 vrf-target target:500:500
```
Проверка
R37#ping 192.168.253.16      
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.253.16, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 3/4/8 ms

![img_6.1.png](screenshots/Lab-6.1.JPG)
