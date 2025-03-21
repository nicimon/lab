# LAB-3

## IS-IS в UNDELAY сети

1. Для упрощения и совместимости принимаем, что все устройства работают с IS-IS 2 Level
2. NSAP Address
   - AFI - 49
   - IDI  - 0078  (00 по умолчанию, 78 -регион)
   - System ID используем видоизмененный адрес Loopback 10.0.1.0 представляем виде 0100.0000.1000, в итоге получаем  49.0078.0100.0000.1000.00
   - NSEL - 00 (по умолчанию)


---
Таблица соотвествия ip адреса Loopback интерфейса с NSAP адресом
| Switch      | Lo0 /32  |           NSAP            |
|-------------|----------|---------------------------|
| Spine1      | 10.0.1.0 | 49.0078.0100.0000.1000.00 |
| Spine2      | 10.0.2.0 | 49.0078.0100.0000.2000.00 |
| Leaf1       | 10.0.1.1 | 49.0078.0100.0000.1001.00 |
| Leaf2       | 10.0.1.2 | 49.0078.0100.0000.1002.00 |
| Leaf3       | 10.0.1.3 | 49.0078.0100.0000.1003.00 |
| Leaf4       | 10.0.1.4 | 49.0078.0100.0000.1004.00 |
| BorderLeaf1 | 10.0.1.5 | 49.0078.0100.0000.1005.00 |
| BorderLeaf2 | 10.0.1.6 | 49.0078.0100.0000.1006.00 |
---
Для настройки IS-IS необходимо:
1. В интерфейсе участвующий в процессе IS-IS указать **family iso**. 
2. В секцию protocols isis указать интерфейсы, запретить Level1, а также указать на Level2 wide metrics

Пример настройки интерфейса
```text
root@Leaf1> show configuration interfaces xe-0/0/1  
unit 0 {
    description "--- Leaf1 - Spine1  ---";
    family inet {
        address 10.2.1.1/31;
    }
    family iso;
}
```
Пример настройки в секции протокола IS-IS
```text
root@Leaf1> show configuration protocols isis 
interface xe-0/0/1.0 {
    level 1 disable;
}
interface xe-0/0/2.0 {
    level 1 disable;
}
interface lo0.0 {
    level 1 disable;
}
level 2 wide-metrics-only;
```
Вывод IS-IS соседства
<details>
<summary>Spine1</summary>

```text
root@Spine1> show isis adjacency 
Interface             System         L State        Hold (secs) SNPA
xe-0/0/1.0            Leaf1          2  Up                   24  2:5:86:71:51:7
xe-0/0/2.0            Leaf2          2  Up                    8  2:5:86:71:e6:7
xe-0/0/3.0            Leaf3          2  Up                   22  2:5:86:71:3c:7
xe-0/0/4.0            Leaf4          2  Up                    6  2:5:86:71:e6:7
xe-0/0/5.0            BorderLeaf1    2  Up                   22  2:5:86:71:2e:7
xe-0/0/6.0            BorderLeaf2    2  Up                   25  2:5:86:71:ac:7
```
</details>

<details>
<summary>Spine2</summary>

```text
root@Spine2> show isis adjacency 
Interface             System         L State        Hold (secs) SNPA
xe-0/0/1.0            Leaf1          2  Up                   21  2:5:86:71:51:b
xe-0/0/2.0            Leaf2          2  Up                    7  2:5:86:71:e6:b
xe-0/0/3.0            Leaf3          2  Up                   24  2:5:86:71:3c:b
xe-0/0/4.0            Leaf4          2  Up                    8  2:5:86:71:e6:b
xe-0/0/5.0            BorderLeaf1    2  Up                   23  2:5:86:71:2e:b
xe-0/0/6.0            BorderLeaf2    2  Up                    6  2:5:86:71:ac:b
```
</details>

<details>
<summary>Leaf1</summary>

```text
root@Leaf1> show isis adjacency 
Interface             System         L State        Hold (secs) SNPA
xe-0/0/1.0            Spine1         2  Up                    7  2:5:86:71:d6:7
xe-0/0/2.0            Spine2         2  Up                    8  2:5:86:71:71:7
```
</details>

<details>
<summary>Leaf2</summary>

```text
root@Leaf2> show isis adjacency 
Interface             System         L State        Hold (secs) SNPA
xe-0/0/1.0            Spine1         2  Up                   18  2:5:86:71:d6:b
xe-0/0/2.0            Spine2         2  Up                   22  2:5:86:71:71:b
```
</details>

<details>
<summary>Leaf3</summary>

```text
root@Leaf3> show isis adjacency 
Interface             System         L State        Hold (secs) SNPA
xe-0/0/1.0            Spine1         2  Up                    6  2:5:86:71:d6:f
xe-0/0/2.0            Spine2         2  Up                    7  2:5:86:71:71:f
```
</details>

<details>
<summary>Leaf4</summary>

```text
root@Leaf4> show isis adjacency 
Interface             System         L State        Hold (secs) SNPA
xe-0/0/1.0            Spine1         2  Up                   23  2:5:86:71:d6:13
xe-0/0/2.0            Spine2         2  Up                   22  2:5:86:71:71:13
```
</details>

<details>
<summary>BorderLeaf1</summary>

```text
root@BorderLeaf1> show isis adjacency 
Interface             System         L State        Hold (secs) SNPA
xe-0/0/1.0            Spine1         2  Up                    8  2:5:86:71:d6:17
xe-0/0/2.0            Spine2         2  Up                    7  2:5:86:71:71:17
```
</details>

<details>
<summary>BorderLeaf2</summary>

```text
root@BorderLeaf2> show isis adjacency 
Interface             System         L State        Hold (secs) SNPA
xe-0/0/1.0            Spine1         2  Up                    7  2:5:86:71:d6:1b
xe-0/0/2.0            Spine2         2  Up                   24  2:5:86:71:71:1b
```
</details>
Вывод информации интерфейсов 
<details>
<summary>Spine1</summary>

```text
root@Spine1> show isis interface 
IS-IS interface database:
Interface             L CirID Level 1 DR        Level 2 DR        L1/L2 Metric
lo0.0                 3   0x1 Disabled          Passive                 0/0
xe-0/0/1.0            2   0x2 Disabled          Spine1.02              10/10
xe-0/0/2.0            2   0x1 Disabled          Leaf2.02               10/10
xe-0/0/3.0            2   0x3 Disabled          Spine1.03              10/10
xe-0/0/4.0            2   0x1 Disabled          Leaf4.02               10/10
xe-0/0/5.0            2   0x4 Disabled          Spine1.04              10/10
xe-0/0/6.0            2   0x5 Disabled          Spine1.05              10/10
```
</details>

<details>
<summary>Spine2</summary>

```text
root@Spine2> show isis interface 
IS-IS interface database:
Interface             L CirID Level 1 DR        Level 2 DR        L1/L2 Metric
lo0.0                 3   0x1 Disabled          Passive                 0/0
xe-0/0/1.0            2   0x2 Disabled          Spine2.02              10/10
xe-0/0/2.0            2   0x1 Disabled          Leaf2.03               10/10
xe-0/0/3.0            2   0x3 Disabled          Spine2.03              10/10
xe-0/0/4.0            2   0x1 Disabled          Leaf4.03               10/10
xe-0/0/5.0            2   0x4 Disabled          Spine2.04              10/10
xe-0/0/6.0            2   0x1 Disabled          BorderLeaf2.02         10/10
```
</details>

<details>
<summary>Leaf1</summary>

```text
root@Leaf1> show isis interface 
IS-IS interface database:
Interface             L CirID Level 1 DR        Level 2 DR        L1/L2 Metric
lo0.0                 3   0x1 Disabled          Passive                 0/0
xe-0/0/1.0            2   0x1 Disabled          Spine1.02              10/10
xe-0/0/2.0            2   0x1 Disabled          Spine2.02              10/10
```
</details>

<details>
<summary>Leaf2</summary>

```text
root@Leaf2> show isis interface 
IS-IS interface database:
Interface             L CirID Level 1 DR        Level 2 DR        L1/L2 Metric
lo0.0                 3   0x1 Disabled          Passive                 0/0
xe-0/0/1.0            2   0x2 Disabled          Leaf2.02               10/10
xe-0/0/2.0            2   0x3 Disabled          Leaf2.03               10/10
```
</details>

<details>
<summary>Leaf3</summary>

```text
root@Leaf3> show isis interface 
IS-IS interface database:
Interface             L CirID Level 1 DR        Level 2 DR        L1/L2 Metric
lo0.0                 3   0x1 Disabled          Passive                 0/0
xe-0/0/1.0            2   0x1 Disabled          Spine1.03              10/10
xe-0/0/2.0            2   0x1 Disabled          Spine2.03              10/10
```
</details>

<details>
<summary>Leaf4</summary>

```text
root@Leaf4> show isis interface 
IS-IS interface database:
Interface             L CirID Level 1 DR        Level 2 DR        L1/L2 Metric
lo0.0                 3   0x1 Disabled          Passive                 0/0
xe-0/0/1.0            2   0x2 Disabled          Leaf4.02               10/10
xe-0/0/2.0            2   0x3 Disabled          Leaf4.03               10/10
```
</details>

<details>
<summary>BorderLeaf1</summary>

```text
root@BorderLeaf1> show isis interface 
IS-IS interface database:
Interface             L CirID Level 1 DR        Level 2 DR        L1/L2 Metric
lo0.0                 3   0x1 Disabled          Passive                 0/0
xe-0/0/1.0            2   0x1 Disabled          Spine1.04              10/10
xe-0/0/2.0            2   0x1 Disabled          Spine2.04              10/10
```
</details>

<details>
<summary>BorderLeaf2</summary>

```text
root@BorderLeaf2> show isis interface 
IS-IS interface database:
Interface             L CirID Level 1 DR        Level 2 DR        L1/L2 Metric
lo0.0                 3   0x1 Disabled          Passive                 0/0
xe-0/0/1.0            2   0x1 Disabled          Spine1.05              10/10
xe-0/0/2.0            2   0x2 Disabled          BorderLeaf2.02         10/10
```
</details>

Вывод isis database

```text
root@Leaf1> show isis database 
IS-IS level 1 link-state database:
LSP ID                      Sequence Checksum Lifetime Attributes
Leaf1.00-00                     0x56   0xf3a5      638 L1 L2
  1 LSPs

IS-IS level 2 link-state database:
LSP ID                      Sequence Checksum Lifetime Attributes
Spine1.00-00                    0x5a   0xe7ea      769 L1 L2
Spine1.02-00                    0x53   0xf74f      642 L1 L2
Spine1.03-00                    0x53   0x192b      928 L1 L2
Spine1.04-00                    0x52   0x3c06      642 L1 L2
Spine1.05-00                    0x52   0x49f6      770 L1 L2
Leaf1.00-00                     0x58   0x8a71      418 L1 L2
Leaf2.00-00                     0x57   0x9269      859 L1 L2
Leaf2.02-00                     0x53   0xe55e      631 L1 L2
Leaf2.03-00                     0x53   0x1023      859 L1 L2
Leaf3.00-00                     0x56   0x449d      960 L1 L2
Leaf4.00-00                     0x57   0x9447      985 L1 L2
Leaf4.02-00                     0x53   0xe758      872 L1 L2
Leaf4.03-00                     0x52   0x141c      672 L1 L2
BorderLeaf1.00-00               0x56   0x1051      481 L1 L2
BorderLeaf2.00-00               0x56   0xc598     1168 L1 L2
BorderLeaf2.02-00               0x53   0x1b11      688 L1 L2
Spine2.00-00                    0x5d   0xd6a2     1124 L1 L2
Spine2.02-00                    0x54   0xe540      585 L1 L2
Spine2.03-00                    0x53    0x91b      755 L1 L2
Spine2.04-00                    0x53   0x2af6      851 L1 L2
  20 LSPs
  ```
Вывод ping и traceroute от Leaf1 - BorderLeaf2  
```text
root@Leaf1> ping 10.0.1.6 
PING 10.0.1.6 (10.0.1.6): 56 data bytes
64 bytes from 10.0.1.6: icmp_seq=0 ttl=63 time=263.625 ms
64 bytes from 10.0.1.6: icmp_seq=1 ttl=63 time=128.886 ms
64 bytes from 10.0.1.6: icmp_seq=2 ttl=63 time=138.006 ms
64 bytes from 10.0.1.6: icmp_seq=3 ttl=63 time=178.101 ms
^C
--- 10.0.1.6 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max/stddev = 128.886/177.154/263.625/53.246 ms

{master:0}
root@Leaf1> traceroute 10.0.1.6 
traceroute to 10.0.1.6 (10.0.1.6), 30 hops max, 40 byte packets
 1  10.2.2.0 (10.2.2.0)  183.560 ms  364.774 ms  303.265 ms
 2  * * *
 3  10.0.1.6 (10.0.1.6)  189.421 ms  400.694 ms  305.938 ms
```