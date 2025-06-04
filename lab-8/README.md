# LAB-8

## Выход из фабрики и связи двух VRF, через внешниее устройство

---
### Схема связи и адресное пространство
Схема и адресное пространство взято из LAB-1.
![lab8-1.PNG](screenshots/lab8-1.PNG)
Фабрика будет построена на EVPN Pure Type-5 (IP Prefix Routes) без использования Type-2 (MAC/IP) для полного отказа от L2-анонсов.

Tenant1 подключен к Leaf1 с настройками
```text
interface Loopback1
 ip address 10.100.20.10 255.255.255.255
!
interface Ethernet0/0
 ip address 198.51.100.21 255.255.255.254
!
router bgp 65001
 bgp log-neighbor-changes
 network 10.100.20.0 mask 255.255.255.0
 redistribute connected
 redistribute static
 neighbor 198.51.100.20 remote-as 4200000011
 neighbor 198.51.100.20 route-map EXPORT-TO-Tenant1 out
!
ip route 10.100.20.0 255.255.255.0 Null0
!
ip prefix-list EVPN-PREFIXES seq 5 permit 0.0.0.0/0 le 32
!
ip prefix-list Tenant1-PREFIXES seq 5 permit 10.100.20.0/24 le 32
!
route-map EXPORT-TO-Tenant1 permit 10
 match ip address prefix-list EVPN-PREFIXES
!
route-map EXPORT-TO-Tenant1 deny 20
!
route-map Tenant1 permit 10
 match ip address prefix-list Tenant1-PREFIXES
```
Tenant2 подключен к Leaf3 с настройками
```text
interface Loopback1
 ip address 10.100.21.10 255.255.255.255
!
interface Ethernet0/0
 ip address 198.52.100.21 255.255.255.254
!
router bgp 65002
 bgp log-neighbor-changes
 network 10.100.21.0 mask 255.255.255.0
 redistribute connected
 redistribute static
 neighbor 198.52.100.20 remote-as 4200000013
 neighbor 198.52.100.20 route-map EXPORT-TO-Tenant2 out
 !
 ip route 10.100.21.0 255.255.255.0 Null0
!
!
ip prefix-list EVPN-PREFIXES seq 5 permit 0.0.0.0/0 le 32
!
ip prefix-list Tenant2-PREFIXES seq 5 permit 10.100.21.0/24 le 32
!
route-map EXPORT-TO-Tenant2 permit 10
 match ip address prefix-list EVPN-PREFIXES
!
route-map EXPORT-TO-Tenant2 deny 20
!
route-map Tenant2 permit 10
 match ip address prefix-list Tenant2-PREFIXES
```
Роутер ExternalRouter
```text
interface Ethernet0/0
 ip address 198.51.200.21 255.255.255.254
!
interface Ethernet0/1
 ip address 198.51.10.21 255.255.255.254
!
router bgp 65003
 bgp router-id 198.51.200.0
 bgp log-neighbor-changes
 neighbor 198.51.10.20 remote-as 4200000016
 neighbor 198.51.200.20 remote-as 4200000015
 !
 address-family ipv4
  redistribute connected
  neighbor 198.51.10.20 activate
  neighbor 198.51.200.20 activate
 exit-address-family
 ```
 ### Настройка Pure Type-5
 ```text
 root@Leaf1> show configuration routing-instances Tenant1 | display set 
set routing-instances Tenant1 instance-type vrf
set routing-instances Tenant1 protocols bgp group Tenant1 type external
set routing-instances Tenant1 protocols bgp group Tenant1 family inet unicast
set routing-instances Tenant1 protocols bgp group Tenant1 export export-to-Tenant1
set routing-instances Tenant1 protocols bgp group Tenant1 peer-as 65001
set routing-instances Tenant1 protocols bgp group Tenant1 neighbor 198.51.100.21
set routing-instances Tenant1 protocols evpn ip-prefix-routes advertise direct-nexthop
set routing-instances Tenant1 protocols evpn ip-prefix-routes encapsulation vxlan
set routing-instances Tenant1 protocols evpn ip-prefix-routes vni 10500
set routing-instances Tenant1 protocols evpn ip-prefix-routes export Tenant1
set routing-instances Tenant1 interface ge-0/0/3.0
set routing-instances Tenant1 route-distinguisher 10.0.1.1:500
set routing-instances Tenant1 vrf-target target:500:500

root@Leaf1> ...atement export-to-Tenant1 | display set                        
set policy-options policy-statement export-to-Tenant1 term 1 from protocol evpn
set policy-options policy-statement export-to-Tenant1 term 1 then accept
set policy-options policy-statement export-to-Tenant1 term 2 then reject

root@Leaf1> ...-options policy-statement Tenant1 | display set              
set policy-options policy-statement Tenant1 term 2 from route-filter 198.51.0.0/16 orlonger
set policy-options policy-statement Tenant1 term 2 then accept
set policy-options policy-statement Tenant1 term 1 from route-filter 10.100.0.0/16 orlonger
set policy-options policy-statement Tenant1 term 1 then accept
set policy-options policy-statement Tenant1 term 3 from route-filter 198.52.0.0/16 orlonger
set policy-options policy-statement Tenant1 term 3 then accept
 ```
 ```text
 root@Leaf3> show configuration routing-instances Tenant2 | display set 
set routing-instances Tenant2 instance-type vrf
set routing-instances Tenant2 protocols bgp group Tenant2 type external
set routing-instances Tenant2 protocols bgp group Tenant2 family inet unicast
set routing-instances Tenant2 protocols bgp group Tenant2 export export-to-Tenant2
set routing-instances Tenant2 protocols bgp group Tenant2 peer-as 65002
set routing-instances Tenant2 protocols bgp group Tenant2 neighbor 198.52.100.21
set routing-instances Tenant2 protocols evpn irb-symmetric-routing vni 10600
set routing-instances Tenant2 protocols evpn ip-prefix-routes advertise direct-nexthop
set routing-instances Tenant2 protocols evpn ip-prefix-routes encapsulation vxlan
set routing-instances Tenant2 protocols evpn ip-prefix-routes vni 10600
set routing-instances Tenant2 protocols evpn ip-prefix-routes export Tenant2
set routing-instances Tenant2 interface ge-0/0/4.0
set routing-instances Tenant2 route-distinguisher 10.0.1.3:600
set routing-instances Tenant2 vrf-target target:600:600

root@Leaf3> ...atement export-to-Tenant2 | display set                        
set policy-options policy-statement export-to-Tenant2 term 1 from protocol evpn
set policy-options policy-statement export-to-Tenant2 term 1 then accept
set policy-options policy-statement export-to-Tenant2 term 2 then reject

root@Leaf3> ...-options policy-statement Tenant2 | display set              
set policy-options policy-statement Tenant2 term 2 from route-filter 198.51.0.0/16 orlonger
set policy-options policy-statement Tenant2 term 2 then accept
set policy-options policy-statement Tenant2 term 1 from route-filter 10.100.0.0/16 orlonger
set policy-options policy-statement Tenant2 term 1 then accept
set policy-options policy-statement Tenant2 term 3 from route-filter 198.52.0.0/16 orlonger
set policy-options policy-statement Tenant2 term 3 then accept
 ```
 ### Проверка пинга между Tenant1 - Tenant2
 ```text
 Tenant1#ping 10.100.21.10
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.100.21.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 6/7/11 ms
```
```text
Tenant1#traceroute 10.100.21.10
Type escape sequence to abort.
Tracing the route to 10.100.21.10
VRF info: (vrf in name/id, vrf out name/id)
  1 198.51.100.20 1 msec 1 msec 1 msec
  2 198.51.200.20 [AS 4200000015] 4 msec 2 msec 4 msec
  3 198.51.200.21 [AS 4200000015] 3 msec 3 msec 4 msec
  4 198.51.10.20 [AS 65003] 5 msec 5 msec 4 msec
  5 198.52.100.20 [AS 4200000013] 7 msec 7 msec 7 msec
  6 198.52.100.21 [AS 4200000013] 6 msec 10 msec 10 msec
  ```
  ```text
  Tenant2#ping 10.100.20.10
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.100.20.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 6/7/10 ms
  ```
  ```text
  Tenant2#traceroute 10.100.20.10
Type escape sequence to abort.
Tracing the route to 10.100.20.10
VRF info: (vrf in name/id, vrf out name/id)
  1 198.52.100.20 1 msec 1 msec 0 msec
  2 198.51.10.20 [AS 4200000016] 4 msec 3 msec 4 msec
  3 198.51.10.21 [AS 4200000016] 3 msec 4 msec 3 msec
  4 198.51.200.20 [AS 65003] 5 msec 4 msec 5 msec
  5 198.51.100.20 [AS 4200000011] 6 msec 6 msec 6 msec
  6 198.51.100.21 [AS 4200000011] 7 msec 7 msec 6 msec
  ```