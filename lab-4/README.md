# LAB-4

## Построение Underlay сети(eBGP)
---
### Схема связи и адресное пространство
Схема и адресное пространство взято из LAB-1

![img_4.png](screenshots/Lab-4.JPG)

---
Таблица соотвествия ip адреса Loopback интерфейса и AS
| Switch      | Lo0 /32  |     AS     |
|-------------|----------|------------|
| Spine1      | 10.0.1.0 | 4200000001 |
| Spine2      | 10.0.2.0 | 4200000001 |
| Leaf1       | 10.0.1.1 | 4200000011 |
| Leaf2       | 10.0.1.2 | 4200000012 |
| Leaf3       | 10.0.1.3 | 4200000013 |
| Leaf4       | 10.0.1.4 | 4200000014 |
| BorderLeaf1 | 10.0.1.5 | 4200000015 |
| BorderLeaf2 | 10.0.1.6 | 4200000016 |
---
Настройка eBGP на Spine и Leaf

Spine1
```text
root@Spine1# show protocols bgp | display set 
set protocols bgp group UNDERLAY type external
set protocols bgp group UNDERLAY family inet unicast
set protocols bgp group UNDERLAY export BGP_LOOPBACK0
set protocols bgp group UNDERLAY multipath
set protocols bgp group UNDERLAY neighbor 10.2.1.1 peer-as 4200000011
set protocols bgp group UNDERLAY neighbor 10.2.1.3 peer-as 4200000012
set protocols bgp group UNDERLAY neighbor 10.2.1.5 peer-as 4200000013
set protocols bgp group UNDERLAY neighbor 10.2.1.7 peer-as 4200000014
set protocols bgp group UNDERLAY neighbor 10.2.1.9 peer-as 4200000015
set protocols bgp group UNDERLAY neighbor 10.2.1.11 peer-as 4200000016

root@Spine1# show routing-options | display set 
set routing-options forwarding-table export PFE-ECMP
set routing-options router-id 10.0.1.0
set routing-options autonomous-system 4200000001
```
Spine2
```text
root@Spine2# show protocols bgp | display set 
set protocols bgp group UNDERLAY type external
set protocols bgp group UNDERLAY family inet unicast
set protocols bgp group UNDERLAY export BGP_LOOPBACK0
set protocols bgp group UNDERLAY multipath
set protocols bgp group UNDERLAY neighbor 10.2.2.1 peer-as 4200000011
set protocols bgp group UNDERLAY neighbor 10.2.2.3 peer-as 4200000012
set protocols bgp group UNDERLAY neighbor 10.2.2.5 peer-as 4200000013
set protocols bgp group UNDERLAY neighbor 10.2.2.7 peer-as 4200000014
set protocols bgp group UNDERLAY neighbor 10.2.2.9 peer-as 4200000015
set protocols bgp group UNDERLAY neighbor 10.2.2.11 peer-as 4200000016

root@Spine2# show routing-options | display set 
set routing-options forwarding-table export PFE-ECMP
set routing-options router-id 10.0.2.0
set routing-options autonomous-system 4200000001
```
Leaf1
```text
root@Leaf1# show protocols bgp | display set 
set protocols bgp group UNDERLAY type external
set protocols bgp group UNDERLAY family inet unicast
set protocols bgp group UNDERLAY export BGP_LOOPBACK0
set protocols bgp group UNDERLAY peer-as 4200000001
set protocols bgp group UNDERLAY multipath
set protocols bgp group UNDERLAY neighbor 10.2.1.0
set protocols bgp group UNDERLAY neighbor 10.2.2.0

root@Leaf1# show routing-options | display set 
set routing-options forwarding-table export PFE-ECMP
set routing-options router-id 10.0.1.1
set routing-options autonomous-system 4200000011
```
Leaf2
```text
root@Leaf2# show protocols bgp | display set 
set protocols bgp group UNDERLAY type external
set protocols bgp group UNDERLAY family inet unicast
set protocols bgp group UNDERLAY export BGP_LOOPBACK0
set protocols bgp group UNDERLAY peer-as 4200000001
set protocols bgp group UNDERLAY multipath
set protocols bgp group UNDERLAY neighbor 10.2.1.2
set protocols bgp group UNDERLAY neighbor 10.2.2.2

root@Leaf2# show routing-options | display set 
set routing-options forwarding-table export PFE-ECMP
set routing-options router-id 10.0.1.2
set routing-options autonomous-system 4200000012

```
leaf3
```text
root@Leaf3# show protocols bgp | display set 
set protocols bgp group UNDERLAY type external
set protocols bgp group UNDERLAY family inet unicast
set protocols bgp group UNDERLAY export BGP_LOOPBACK0
set protocols bgp group UNDERLAY peer-as 4200000001
set protocols bgp group UNDERLAY multipath
set protocols bgp group UNDERLAY neighbor 10.2.1.4
set protocols bgp group UNDERLAY neighbor 10.2.2.4

root@Leaf3# show routing-options | display set 
set routing-options forwarding-table export PFE-ECMP
set routing-options router-id 10.0.1.3
set routing-options autonomous-system 4200000013
```
leaf4
```text
root@Leaf4# show protocols bgp | display set 
set protocols bgp group UNDERLAY type external
set protocols bgp group UNDERLAY family inet unicast
set protocols bgp group UNDERLAY export BGP_LOOPBACK0
set protocols bgp group UNDERLAY peer-as 4200000001
set protocols bgp group UNDERLAY multipath multiple-as
set protocols bgp group UNDERLAY neighbor 10.2.1.6
set protocols bgp group UNDERLAY neighbor 10.2.2.6

root@Leaf4# show routing-options | display set 
set routing-options forwarding-table export PFE-ECMP
set routing-options router-id 10.0.1.4
set routing-options autonomous-system 4200000014
```
BorderLeaf1
```text
root@BorderLeaf1# show protocols bgp | display set 
set protocols bgp group UNDERLAY type external
set protocols bgp group UNDERLAY family inet unicast
set protocols bgp group UNDERLAY export BGP_LOOPBACK0
set protocols bgp group UNDERLAY peer-as 4200000001
set protocols bgp group UNDERLAY multipath multiple-as
set protocols bgp group UNDERLAY neighbor 10.2.1.8
set protocols bgp group UNDERLAY neighbor 10.2.2.8

root@BorderLeaf1# show routing-options | display set 
set routing-options forwarding-table export PFE-ECMP
set routing-options router-id 10.0.1.5
set routing-options autonomous-system 4200000015
```
BorderLeaf2
```text
root@BorderLeaf2# show protocols bgp | display set 
set protocols bgp group UNDERLAY type external
set protocols bgp group UNDERLAY export BGP_LOOPBACK0
set protocols bgp group UNDERLAY peer-as 4200000001
set protocols bgp group UNDERLAY multipath
set protocols bgp group UNDERLAY neighbor 10.2.1.10
set protocols bgp group UNDERLAY neighbor 10.2.2.10

root@BorderLeaf2# show routing-options | display set 
set routing-options forwarding-table export PFE-ECMP
set routing-options router-id 10.0.1.6
set routing-options autonomous-system 4200000016
```
Настройка политики балансировки ECMP и политики экспорта Lo0 адресов в BGP  
```text
show policy-options policy-statement PFE-ECMP | display set 
set policy-options policy-statement PFE-ECMP then load-balance per-packet

show policy-options policy-statement BGP_LOOPBACK0 | display s...
set policy-options policy-statement BGP_LOOPBACK0 term TERM1 from protocol direct
set policy-options policy-statement BGP_LOOPBACK0 term TERM1 from route-filter 10.0.1.0/32 exact
set policy-options policy-statement BGP_LOOPBACK0 term TERM1 then accept
```
---
Проверка
```text
root@Spine1> show bgp summary 
Threading mode: BGP I/O
Default eBGP mode: advertise - accept, receive - accept
Groups: 1 Peers: 6 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                       6          6          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
10.2.1.1         4200000011       2186       2160       0       0    16:24:26 Establ
  inet.0: 1/1/1/0
10.2.1.3         4200000012       2182       2158       0       0    16:24:21 Establ
  inet.0: 1/1/1/0
10.2.1.5         4200000013       2184       2159       0       0    16:24:26 Establ
  inet.0: 1/1/1/0
10.2.1.7         4200000014       2183       2159       0       0    16:24:26 Establ
  inet.0: 1/1/1/0
10.2.1.9         4200000015       2183       2158       0       0    16:24:26 Establ
  inet.0: 1/1/1/0
10.2.1.11        4200000016       2186       2159       0       0    16:24:25 Establ
  inet.0: 1/1/1/0
  ```
  ```text
  root@Spine2> show bgp summary 
Threading mode: BGP I/O
Default eBGP mode: advertise - accept, receive - accept
Groups: 1 Peers: 6 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                       6          6          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
10.2.2.1         4200000011       2206       2177       0       3    16:33:28 Establ
  inet.0: 1/1/1/0
10.2.2.3         4200000012       2202       2177       0       3    16:33:28 Establ
  inet.0: 1/1/1/0
10.2.2.5         4200000013       2203       2176       0       3    16:33:28 Establ
  inet.0: 1/1/1/0
10.2.2.7         4200000014       2203       2176       0       3    16:33:28 Establ
  inet.0: 1/1/1/0
10.2.2.9         4200000015       2203       2177       0       3    16:33:28 Establ
  inet.0: 1/1/1/0
10.2.2.11        4200000016       2207       2177       0       3    16:33:28 Establ
  inet.0: 1/1/1/0
```
 ```text
 root@Leaf1> show bgp summary 
Threading mode: BGP I/O
Default eBGP mode: advertise - accept, receive - accept
Groups: 1 Peers: 2 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                      12         12          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
10.2.1.0         4200000001       2166       2190       0       4    16:27:00 Establ
  inet.0: 6/6/6/0
10.2.2.0         4200000001       2181       2208       0       3    16:35:00 Establ
  inet.0: 6/6/6/0
 ```
 ```text
 root@Leaf2> show bgp summary 
Threading mode: BGP I/O
Default eBGP mode: advertise - accept, receive - accept
Groups: 1 Peers: 2 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                      12         12          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
10.2.1.2         4200000001       2166       2188       0       3    16:26:54 Establ
  inet.0: 6/6/6/0
10.2.2.2         4200000001       2182       2205       0       2    16:34:59 Establ
  inet.0: 6/6/6/0
 ```
 ```text
 root@Leaf3> show bgp summary 
Threading mode: BGP I/O
Default eBGP mode: advertise - accept, receive - accept
Groups: 1 Peers: 2 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                      12         12          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
10.2.1.4         4200000001       2165       2187       0       2    16:27:00 Establ
  inet.0: 6/6/6/0
10.2.2.4         4200000001       2180       2205       0       1    16:35:00 Establ
  inet.0: 6/6/6/0
 ```
 ```text
 root@Leaf4> show bgp summary 
Threading mode: BGP I/O
Default eBGP mode: advertise - accept, receive - accept
Groups: 1 Peers: 2 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                      12         12          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
10.2.1.6         4200000001       2166       2188       0       2    16:27:00 Establ
  inet.0: 6/6/6/0
10.2.2.6         4200000001       2180       2205       0       1    16:35:00 Establ
  inet.0: 6/6/6/0
  ```
  ```text
  root@BorderLeaf1> show bgp summary 
Threading mode: BGP I/O
Default eBGP mode: advertise - accept, receive - accept
Groups: 1 Peers: 2 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                      12         12          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
10.2.1.8         4200000001       2165       2188       0       2    16:26:59 Establ
  inet.0: 6/6/6/0
10.2.2.8         4200000001       2181       2205       0       1    16:34:59 Establ
  inet.0: 6/6/6/0
   ```
```text
root@BorderLeaf2> show bgp summary 
Threading mode: BGP I/O
Default eBGP mode: advertise - accept, receive - accept
Groups: 1 Peers: 2 Down peers: 0
Table          Tot Paths  Act Paths Suppressed    History Damp State    Pending
inet.0               
                      12         12          0          0          0          0
Peer                     AS      InPkt     OutPkt    OutQ   Flaps Last Up/Dwn State|#Active/Received/Accepted/Damped...
10.2.1.10        4200000001       2165       2190       0       2    16:26:58 Establ
  inet.0: 6/6/6/0
10.2.2.10        4200000001       2181       2209       0       1    16:35:00 Establ
  inet.0: 6/6/6/0
```

Проверка связанности 
 ```text
root@Leaf1> ping 10.0.1.6 count 3
PING 10.0.1.6 (10.0.1.6): 56 data bytes
64 bytes from 10.0.1.6: icmp_seq=0 ttl=63 time=120.962 ms
64 bytes from 10.0.1.6: icmp_seq=1 ttl=63 time=126.532 ms
64 bytes from 10.0.1.6: icmp_seq=2 ttl=63 time=207.981 ms

--- 10.0.1.6 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max/stddev = 120.962/151.825/207.981/39.773 ms
```
 ```text
root@Leaf1> traceroute 10.0.1.6 
traceroute to 10.0.1.6 (10.0.1.6), 30 hops max, 40 byte packets
 1  10.2.1.0 (10.2.1.0)  124.781 ms *  113.512 ms
 2  10.0.1.6 (10.0.1.6)  310.235 ms  206.774 ms  213.640 ms
```