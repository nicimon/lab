# LAB-5

## VxLAN. L2 VNI
---
### Схема связи и адресное пространство
Схема и адресное пространство взято из LAB-1

![img_5.png](screenshots/Lab-5.JPG)

---
Для Overlay используем eBGP

Конфигурация на Leaf
```text
root@Leaf1# show protocols bgp group OVERLAY | display set 
set protocols bgp group OVERLAY type external
set protocols bgp group OVERLAY multihop
set protocols bgp group OVERLAY local-address 10.0.1.1
set protocols bgp group OVERLAY family evpn signaling
set protocols bgp group OVERLAY peer-as 4200000001
set protocols bgp group OVERLAY multipath
set protocols bgp group OVERLAY neighbor 10.0.1.0 description "Spine1 loopback"
set protocols bgp group OVERLAY neighbor 10.0.2.0 description "Spine2 loopback"
```
Конфигурация на Spine
```text
root@Spine1# show protocols bgp group OVERLAY | display set 
set protocols bgp group OVERLAY type external
set protocols bgp group OVERLAY multihop no-nexthop-change
set protocols bgp group OVERLAY local-address 10.0.1.0
set protocols bgp group OVERLAY family evpn signaling
set protocols bgp group OVERLAY multipath
set protocols bgp group OVERLAY neighbor 10.0.1.1 description "Leaf1 loopback"
set protocols bgp group OVERLAY neighbor 10.0.1.1 peer-as 4200000011
set protocols bgp group OVERLAY neighbor 10.0.1.2 description "Leaf2 loopback"
set protocols bgp group OVERLAY neighbor 10.0.1.2 peer-as 4200000012
set protocols bgp group OVERLAY neighbor 10.0.1.3 description "Leaf3 loopback"
set protocols bgp group OVERLAY neighbor 10.0.1.3 peer-as 4200000013
set protocols bgp group OVERLAY neighbor 10.0.1.4 description "Leaf4 loopback"
set protocols bgp group OVERLAY neighbor 10.0.1.4 peer-as 4200000014
set protocols bgp group OVERLAY neighbor 10.0.1.5 description "BorderLeaf1 loopback"
set protocols bgp group OVERLAY neighbor 10.0.1.5 peer-as 4200000015
set protocols bgp group OVERLAY neighbor 10.0.1.6 description "BorderLeaf2 loopback"
set protocols bgp group OVERLAY neighbor 10.0.1.6 peer-as 4200000016
```
Вывод BGP Overlay
```text
root@Spine1# run show bgp summary group OVERLAY                  
Threading mode: BGP I/O
Default eBGP mode: advertise - accept, receive - accept
Groups: 2 Peers: 12 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                       6          6          0          0          0          0
bgp.evpn.0           
                       6          6          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
10.0.1.1         4200000011       2566       2532       0       2    19:22:02 Establ
  bgp.evpn.0: 3/3/3/0
10.0.1.2         4200000012       2562       2533       0       0    19:21:23 Establ
  bgp.evpn.0: 0/0/0/0
10.0.1.3         4200000013        217        217       0       0     1:37:37 Establ
  bgp.evpn.0: 0/0/0/0
10.0.1.4         4200000014        216        216       0       0     1:37:33 Establ
  bgp.evpn.0: 0/0/0/0
10.0.1.5         4200000015        216        216       0       0     1:37:29 Establ
  bgp.evpn.0: 0/0/0/0
10.0.1.6         4200000016        219        214       0       0     1:37:36 Establ
```
```text
root@Spine2# run show bgp summary group OVERLAY 
Threading mode: BGP I/O
Default eBGP mode: advertise - accept, receive - accept
Groups: 2 Peers: 12 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                       6          6          0          0          0          0
bgp.evpn.0           
                       6          6          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
10.0.1.1         4200000011        210        207       0       0     1:34:00 Establ
  bgp.evpn.0: 3/3/3/0
10.0.1.2         4200000012        207        208       0       0     1:33:56 Establ
  bgp.evpn.0: 0/0/0/0
10.0.1.3         4200000013        208        208       0       0     1:33:52 Establ
  bgp.evpn.0: 0/0/0/0
10.0.1.4         4200000014        208        208       0       0     1:33:49 Establ
  bgp.evpn.0: 0/0/0/0
10.0.1.5         4200000015        208        208       0       0     1:33:44 Establ
  bgp.evpn.0: 0/0/0/0
10.0.1.6         4200000016        210        206       0       0     1:33:40 Establ
  bgp.evpn.0: 3/3/3/0
```
Настройка RD, RT
```text
root@Leaf1# show switch-options | display set 
set switch-options vtep-source-interface lo0.0
set switch-options route-distinguisher 10.0.1.1:3
set switch-options vrf-target target:3:3
```
Указание evpn в качестве инкапсуляции
```text
root@Leaf1# show protocols evpn | display set 
set protocols evpn encapsulation vxlan
set protocols evpn extended-vni-list all
```
Добавление vlan
```text
root@Leaf1# show vlans v100 | display set 
set vlans v100 vlan-id 100
set vlans v100 vxlan vni 10100
```
Настройка интерфейса
```text
root@Leaf1# show interfaces xe-0/0/3 | display set 
set interfaces xe-0/0/3 unit 0 family ethernet-switching interface-mode access
set interfaces xe-0/0/3 unit 0 family ethernet-switching vlan members v100
```
Проверка связанности между серверами, подключенные между Leaf1 и BorderLeaf2
```text
root@Docker19:/home# ping 192.168.254.16 -c 5
PING 192.168.254.16 (192.168.254.16) 56(84) bytes of data.
64 bytes from 192.168.254.16: icmp_seq=1 ttl=64 time=114 ms
64 bytes from 192.168.254.16: icmp_seq=2 ttl=64 time=113 ms
64 bytes from 192.168.254.16: icmp_seq=3 ttl=64 time=113 ms
64 bytes from 192.168.254.16: icmp_seq=4 ttl=64 time=116 ms
64 bytes from 192.168.254.16: icmp_seq=5 ttl=64 time=113 ms

--- 192.168.254.16 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4005ms
rtt min/avg/max/mdev = 113.091/114.037/116.084/1.132 ms
```
```text
root@Docker20:/home# ping 192.168.254.10 -c 5
PING 192.168.254.10 (192.168.254.10) 56(84) bytes of data.
64 bytes from 192.168.254.10: icmp_seq=1 ttl=64 time=114 ms
64 bytes from 192.168.254.10: icmp_seq=2 ttl=64 time=115 ms
64 bytes from 192.168.254.10: icmp_seq=3 ttl=64 time=114 ms
64 bytes from 192.168.254.10: icmp_seq=4 ttl=64 time=227 ms
64 bytes from 192.168.254.10: icmp_seq=5 ttl=64 time=207 ms

--- 192.168.254.10 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 113.550/155.411/226.650/50.571 ms
```
Вывод mac address table
```text
root@Leaf1> show ethernet-switching table 

MAC flags (S - static MAC, D - dynamic MAC, L - locally learned, P - Persistent static
           SE - statistics enabled, NM - non configured MAC, R - remote PE MAC, O - ovsdb MAC)


Ethernet switching table : 2 entries, 2 learned
Routing instance : default-switch
   Vlan                MAC                 MAC      Logical                SVLBNH/      Active
   name                address             flags    interface              VENH Index   source
   v100                50:00:00:15:00:01   D        xe-0/0/3.0           
   v100                50:00:00:16:00:01   D        vtep.32769                          10.0.1.6
```
```text
root@Leaf1> show evpn database                
Instance: default-switch
VLAN  DomainId  MAC address        Active source                  Timestamp        IP address
     10100      50:00:00:15:00:01  xe-0/0/3.0                     Apr 09 10:10:18  192.168.254.10
     10100      50:00:00:16:00:01  10.0.1.6                       Apr 09 10:07:34  192.168.254.16
```
```text
root@Leaf1# show | display set 
set version 20.3R1.8
set system host-name Leaf1
set system root-authentication encrypted-password "$6$iOHCqldH$4Bv5iM.SYPDCPExs15aDwLgKfad9fLWfdv53fovFYghTXlJ9rQVFxA9yoIUOf58hSJrXKANdZ.3L2ZWXBPs1T0"
set system root-authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key"
set system login user vagrant uid 2000
set system login user vagrant class super-user
set system login user vagrant authentication ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key"
set system services ssh root-login allow
set system services netconf ssh
set system services rest http port 8080
set system services rest enable-explorer
set system syslog user * any emergency
set system syslog file messages any notice
set system syslog file messages authorization info
set system syslog file interactive-commands interactive-commands any
set system extensions providers juniper license-type juniper deployment-scope commercial
set system extensions providers chef license-type juniper deployment-scope commercial
set interfaces xe-0/0/1 unit 0 description "--- Leaf1 - Spine1  ---"
set interfaces xe-0/0/1 unit 0 family inet address 10.2.1.1/31
set interfaces xe-0/0/2 unit 0 description "--- Leaf1 - Spine2  ---"
set interfaces xe-0/0/2 unit 0 family inet address 10.2.2.1/31
set interfaces xe-0/0/3 unit 0 family ethernet-switching interface-mode access
set interfaces xe-0/0/3 unit 0 family ethernet-switching vlan members v100
set interfaces em0 unit 0 family inet dhcp
set interfaces em1 unit 0 family inet address 169.254.0.2/24
set interfaces lo0 unit 0 family inet address 10.0.1.1/32
set forwarding-options storm-control-profiles default all
set policy-options policy-statement BGP_LOOPBACK0 term TERM1 from protocol direct
set policy-options policy-statement BGP_LOOPBACK0 term TERM1 from route-filter 10.0.1.1/32 exact
set policy-options policy-statement BGP_LOOPBACK0 term TERM1 then accept
set policy-options policy-statement PFE-ECMP then load-balance per-packet
set policy-options policy-statement allow-loopback from interface lo0.0
set policy-options policy-statement allow-loopback then accept
set routing-options forwarding-table export PFE-ECMP
set routing-options router-id 10.0.1.1
set routing-options autonomous-system 4200000011
set protocols bgp group UNDERLAY type external
set protocols bgp group UNDERLAY family inet unicast
set protocols bgp group UNDERLAY export BGP_LOOPBACK0
set protocols bgp group UNDERLAY peer-as 4200000001
set protocols bgp group UNDERLAY multipath
set protocols bgp group UNDERLAY neighbor 10.2.1.0
set protocols bgp group UNDERLAY neighbor 10.2.2.0
set protocols bgp group OVERLAY type external
set protocols bgp group OVERLAY multihop
set protocols bgp group OVERLAY local-address 10.0.1.1
set protocols bgp group OVERLAY family evpn signaling
set protocols bgp group OVERLAY peer-as 4200000001
set protocols bgp group OVERLAY multipath
set protocols bgp group OVERLAY neighbor 10.0.1.0 description "Spine1 loopback"
set protocols bgp group OVERLAY neighbor 10.0.2.0 description "Spine2 loopback"
set protocols evpn encapsulation vxlan
set protocols evpn extended-vni-list all
set protocols igmp-snooping vlan default
set switch-options vtep-source-interface lo0.0
set switch-options route-distinguisher 10.0.1.1:3
set switch-options vrf-target target:3:3
set vlans default vlan-id 1
set vlans v100 vlan-id 100
set vlans v100 vxlan vni 10100
```