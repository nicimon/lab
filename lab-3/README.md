# LAB-3

## IS-IS в UNDELAY сети
---
### Схема связи и адресное пространство
Схема и адресное пространство взято из LAB-1

![img_3.png](screenshots/laba3.png)

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
1. В интерфейсе участвующий в процессе IS-IS указываем **family iso**. 
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
``` text
root@Leaf1> show configuration interfaces lo0 
unit 0 {
    family inet {
        address 10.0.1.1/32;
    }
    family iso {
        address 49.0078.0100.0000.1001.00;
    }
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
 Вывод show route protocol isis 
<details>
<summary>Spine1</summary>

``` text
root@Spine1> show route protocol isis 

inet.0: 28 destinations, 28 routes (28 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.0.1.1/32        *[IS-IS/18] 18:26:34, metric 10
                    >  to 10.2.1.1 via xe-0/0/1.0
10.0.1.2/32        *[IS-IS/18] 18:17:06, metric 10
                    >  to 10.2.1.3 via xe-0/0/2.0
10.0.1.3/32        *[IS-IS/18] 18:16:07, metric 10
                    >  to 10.2.1.5 via xe-0/0/3.0
10.0.1.4/32        *[IS-IS/18] 18:14:15, metric 10
                    >  to 10.2.1.7 via xe-0/0/4.0
10.0.1.5/32        *[IS-IS/18] 18:13:16, metric 10
                    >  to 10.2.1.9 via xe-0/0/5.0
10.0.1.6/32        *[IS-IS/18] 18:12:29, metric 10
                    >  to 10.2.1.11 via xe-0/0/6.0
10.0.2.0/32        *[IS-IS/18] 18:12:03, metric 20
                       to 10.2.1.1 via xe-0/0/1.0
                       to 10.2.1.3 via xe-0/0/2.0
                       to 10.2.1.5 via xe-0/0/3.0
                    >  to 10.2.1.7 via xe-0/0/4.0
                       to 10.2.1.9 via xe-0/0/5.0
                       to 10.2.1.11 via xe-0/0/6.0
10.2.2.0/31        *[IS-IS/18] 18:30:36, metric 20
                    >  to 10.2.1.1 via xe-0/0/1.0
10.2.2.2/31        *[IS-IS/18] 18:17:06, metric 20
                    >  to 10.2.1.3 via xe-0/0/2.0
10.2.2.4/31        *[IS-IS/18] 18:16:07, metric 20
                    >  to 10.2.1.5 via xe-0/0/3.0
10.2.2.6/31        *[IS-IS/18] 18:14:15, metric 20
                    >  to 10.2.1.7 via xe-0/0/4.0
10.2.2.8/31        *[IS-IS/18] 18:13:16, metric 20
                    >  to 10.2.1.9 via xe-0/0/5.0
10.2.2.10/31       *[IS-IS/18] 18:12:29, metric 20
                    >  to 10.2.1.11 via xe-0/0/6.0
```
</details>

<details>
<summary>Spine2</summary>

``` text
root@Spine2> show route protocol isis 

inet.0: 28 destinations, 28 routes (28 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.0.1.0/32        *[IS-IS/18] 18:14:33, metric 20
                       to 10.2.2.1 via xe-0/0/1.0
                       to 10.2.2.3 via xe-0/0/2.0
                       to 10.2.2.5 via xe-0/0/3.0
                       to 10.2.2.7 via xe-0/0/4.0
                    >  to 10.2.2.9 via xe-0/0/5.0
                       to 10.2.2.11 via xe-0/0/6.0
10.0.1.1/32        *[IS-IS/18] 18:29:04, metric 10
                    >  to 10.2.2.1 via xe-0/0/1.0
10.0.1.2/32        *[IS-IS/18] 18:19:05, metric 10
                    >  to 10.2.2.3 via xe-0/0/2.0
10.0.1.3/32        *[IS-IS/18] 18:18:11, metric 10
                    >  to 10.2.2.5 via xe-0/0/3.0
10.0.1.4/32        *[IS-IS/18] 18:16:18, metric 10
                    >  to 10.2.2.7 via xe-0/0/4.0
10.0.1.5/32        *[IS-IS/18] 18:15:20, metric 10
                    >  to 10.2.2.9 via xe-0/0/5.0
10.0.1.6/32        *[IS-IS/18] 18:14:33, metric 10
                    >  to 10.2.2.11 via xe-0/0/6.0
10.2.1.0/31        *[IS-IS/18] 18:32:40, metric 20
                    >  to 10.2.2.1 via xe-0/0/1.0
10.2.1.2/31        *[IS-IS/18] 18:19:05, metric 20
                    >  to 10.2.2.3 via xe-0/0/2.0
10.2.1.4/31        *[IS-IS/18] 18:18:11, metric 20
                    >  to 10.2.2.5 via xe-0/0/3.0
10.2.1.6/31        *[IS-IS/18] 18:16:18, metric 20
                    >  to 10.2.2.7 via xe-0/0/4.0
10.2.1.8/31        *[IS-IS/18] 18:15:20, metric 20
                    >  to 10.2.2.9 via xe-0/0/5.0
10.2.1.10/31       *[IS-IS/18] 18:14:33, metric 20
                    >  to 10.2.2.11 via xe-0/0/6.0
```
</details>

<details>
<summary>Leaf1</summary>

``` text
root@Leaf1> show route protocol isis 

inet.0: 24 destinations, 24 routes (24 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.0.1.0/32        *[IS-IS/18] 18:31:28, metric 10
                    >  to 10.2.1.0 via xe-0/0/1.0
10.0.1.2/32        *[IS-IS/18] 18:20:52, metric 20
                       to 10.2.1.0 via xe-0/0/1.0
                    >  to 10.2.2.0 via xe-0/0/2.0
10.0.1.3/32        *[IS-IS/18] 18:19:57, metric 20
                       to 10.2.1.0 via xe-0/0/1.0
                    >  to 10.2.2.0 via xe-0/0/2.0
10.0.1.4/32        *[IS-IS/18] 18:18:04, metric 20
                       to 10.2.1.0 via xe-0/0/1.0
                    >  to 10.2.2.0 via xe-0/0/2.0
10.0.1.5/32        *[IS-IS/18] 18:17:07, metric 20
                       to 10.2.1.0 via xe-0/0/1.0
                    >  to 10.2.2.0 via xe-0/0/2.0
10.0.1.6/32        *[IS-IS/18] 18:16:20, metric 20
                       to 10.2.1.0 via xe-0/0/1.0
                    >  to 10.2.2.0 via xe-0/0/2.0
10.0.2.0/32        *[IS-IS/18] 18:31:06, metric 10
                    >  to 10.2.2.0 via xe-0/0/2.0
10.2.1.2/31        *[IS-IS/18] 18:34:52, metric 20
                    >  to 10.2.1.0 via xe-0/0/1.0
10.2.1.4/31        *[IS-IS/18] 18:34:52, metric 20
                    >  to 10.2.1.0 via xe-0/0/1.0
10.2.1.6/31        *[IS-IS/18] 18:34:52, metric 20
                    >  to 10.2.1.0 via xe-0/0/1.0
10.2.1.8/31        *[IS-IS/18] 18:34:52, metric 20
                    >  to 10.2.1.0 via xe-0/0/1.0
10.2.1.10/31       *[IS-IS/18] 18:34:52, metric 20
                    >  to 10.2.1.0 via xe-0/0/1.0
10.2.2.2/31        *[IS-IS/18] 18:34:27, metric 20
                    >  to 10.2.2.0 via xe-0/0/2.0
10.2.2.4/31        *[IS-IS/18] 18:34:27, metric 20
                    >  to 10.2.2.0 via xe-0/0/2.0
10.2.2.6/31        *[IS-IS/18] 18:34:27, metric 20
                    >  to 10.2.2.0 via xe-0/0/2.0
10.2.2.8/31        *[IS-IS/18] 18:34:27, metric 20
                    >  to 10.2.2.0 via xe-0/0/2.0
10.2.2.10/31       *[IS-IS/18] 18:34:27, metric 20
                    >  to 10.2.2.0 via xe-0/0/2.0
```
</details>

<details>
<summary>Leaf2</summary>

``` text

root@Leaf2> show route protocol isis 

inet.0: 24 destinations, 24 routes (24 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.0.1.0/32        *[IS-IS/18] 18:22:05, metric 10
                    >  to 10.2.1.2 via xe-0/0/1.0
10.0.1.1/32        *[IS-IS/18] 18:21:35, metric 20
                       to 10.2.1.2 via xe-0/0/1.0
                    >  to 10.2.2.2 via xe-0/0/2.0
10.0.1.3/32        *[IS-IS/18] 18:20:41, metric 20
                       to 10.2.1.2 via xe-0/0/1.0
                    >  to 10.2.2.2 via xe-0/0/2.0
10.0.1.4/32        *[IS-IS/18] 18:18:48, metric 20
                       to 10.2.1.2 via xe-0/0/1.0
                    >  to 10.2.2.2 via xe-0/0/2.0
10.0.1.5/32        *[IS-IS/18] 18:17:50, metric 20
                       to 10.2.1.2 via xe-0/0/1.0
                    >  to 10.2.2.2 via xe-0/0/2.0
10.0.1.6/32        *[IS-IS/18] 18:17:03, metric 20
                       to 10.2.1.2 via xe-0/0/1.0
                    >  to 10.2.2.2 via xe-0/0/2.0
10.0.2.0/32        *[IS-IS/18] 18:21:35, metric 10
                    >  to 10.2.2.2 via xe-0/0/2.0
10.2.1.0/31        *[IS-IS/18] 18:22:05, metric 20
                    >  to 10.2.1.2 via xe-0/0/1.0
10.2.1.4/31        *[IS-IS/18] 18:22:05, metric 20
                    >  to 10.2.1.2 via xe-0/0/1.0
10.2.1.6/31        *[IS-IS/18] 18:22:05, metric 20
                    >  to 10.2.1.2 via xe-0/0/1.0
10.2.1.8/31        *[IS-IS/18] 18:22:05, metric 20
                    >  to 10.2.1.2 via xe-0/0/1.0
10.2.1.10/31       *[IS-IS/18] 18:22:05, metric 20
                    >  to 10.2.1.2 via xe-0/0/1.0
10.2.2.0/31        *[IS-IS/18] 18:21:35, metric 20
                    >  to 10.2.2.2 via xe-0/0/2.0
10.2.2.4/31        *[IS-IS/18] 18:21:35, metric 20
                    >  to 10.2.2.2 via xe-0/0/2.0
10.2.2.6/31        *[IS-IS/18] 18:21:35, metric 20
                    >  to 10.2.2.2 via xe-0/0/2.0
10.2.2.8/31        *[IS-IS/18] 18:21:35, metric 20
                    >  to 10.2.2.2 via xe-0/0/2.0
10.2.2.10/31       *[IS-IS/18] 18:21:35, metric 20
                    >  to 10.2.2.2 via xe-0/0/2.0
```
</details>

<details>
<summary>Leaf3</summary>

``` text
root@Leaf3> show route protocol isis 

inet.0: 24 destinations, 24 routes (24 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.0.1.0/32        *[IS-IS/18] 18:21:39, metric 10
                    >  to 10.2.1.4 via xe-0/0/1.0
10.0.1.1/32        *[IS-IS/18] 18:21:14, metric 20
                       to 10.2.1.4 via xe-0/0/1.0
                    >  to 10.2.2.4 via xe-0/0/2.0
10.0.1.2/32        *[IS-IS/18] 18:21:14, metric 20
                       to 10.2.1.4 via xe-0/0/1.0
                    >  to 10.2.2.4 via xe-0/0/2.0
10.0.1.4/32        *[IS-IS/18] 18:19:20, metric 20
                       to 10.2.1.4 via xe-0/0/1.0
                    >  to 10.2.2.4 via xe-0/0/2.0
10.0.1.5/32        *[IS-IS/18] 18:18:23, metric 20
                       to 10.2.1.4 via xe-0/0/1.0
                    >  to 10.2.2.4 via xe-0/0/2.0
10.0.1.6/32        *[IS-IS/18] 18:17:36, metric 20
                       to 10.2.1.4 via xe-0/0/1.0
                    >  to 10.2.2.4 via xe-0/0/2.0
10.0.2.0/32        *[IS-IS/18] 18:21:14, metric 10
                    >  to 10.2.2.4 via xe-0/0/2.0
10.2.1.0/31        *[IS-IS/18] 18:21:39, metric 20
                    >  to 10.2.1.4 via xe-0/0/1.0
10.2.1.2/31        *[IS-IS/18] 18:21:39, metric 20
                    >  to 10.2.1.4 via xe-0/0/1.0
10.2.1.6/31        *[IS-IS/18] 18:21:39, metric 20
                    >  to 10.2.1.4 via xe-0/0/1.0
10.2.1.8/31        *[IS-IS/18] 18:21:39, metric 20
                    >  to 10.2.1.4 via xe-0/0/1.0
10.2.1.10/31       *[IS-IS/18] 18:21:39, metric 20
                    >  to 10.2.1.4 via xe-0/0/1.0
10.2.2.0/31        *[IS-IS/18] 18:21:14, metric 20
                    >  to 10.2.2.4 via xe-0/0/2.0
10.2.2.2/31        *[IS-IS/18] 18:21:14, metric 20
                    >  to 10.2.2.4 via xe-0/0/2.0
10.2.2.6/31        *[IS-IS/18] 18:21:14, metric 20
                    >  to 10.2.2.4 via xe-0/0/2.0
10.2.2.8/31        *[IS-IS/18] 18:21:14, metric 20
                    >  to 10.2.2.4 via xe-0/0/2.0
10.2.2.10/31       *[IS-IS/18] 18:21:14, metric 20
                    >  to 10.2.2.4 via xe-0/0/2.0
```
</details>

<details>
<summary>Leaf4</summary>

``` text
root@Leaf4> show route protocol isis 

inet.0: 24 destinations, 24 routes (24 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.0.1.0/32        *[IS-IS/18] 18:20:14, metric 10
                    >  to 10.2.1.6 via xe-0/0/1.0
10.0.1.1/32        *[IS-IS/18] 18:19:47, metric 20
                       to 10.2.1.6 via xe-0/0/1.0
                    >  to 10.2.2.6 via xe-0/0/2.0
10.0.1.2/32        *[IS-IS/18] 18:19:47, metric 20
                       to 10.2.1.6 via xe-0/0/1.0
                    >  to 10.2.2.6 via xe-0/0/2.0
10.0.1.3/32        *[IS-IS/18] 18:19:47, metric 20
                       to 10.2.1.6 via xe-0/0/1.0
                    >  to 10.2.2.6 via xe-0/0/2.0
10.0.1.5/32        *[IS-IS/18] 18:18:50, metric 20
                       to 10.2.1.6 via xe-0/0/1.0
                    >  to 10.2.2.6 via xe-0/0/2.0
10.0.1.6/32        *[IS-IS/18] 18:18:03, metric 20
                       to 10.2.1.6 via xe-0/0/1.0
                    >  to 10.2.2.6 via xe-0/0/2.0
10.0.2.0/32        *[IS-IS/18] 18:19:47, metric 10
                    >  to 10.2.2.6 via xe-0/0/2.0
10.2.1.0/31        *[IS-IS/18] 18:20:14, metric 20
                    >  to 10.2.1.6 via xe-0/0/1.0
10.2.1.2/31        *[IS-IS/18] 18:20:14, metric 20
                    >  to 10.2.1.6 via xe-0/0/1.0
10.2.1.4/31        *[IS-IS/18] 18:20:14, metric 20
                    >  to 10.2.1.6 via xe-0/0/1.0
10.2.1.8/31        *[IS-IS/18] 18:20:14, metric 20
                    >  to 10.2.1.6 via xe-0/0/1.0
10.2.1.10/31       *[IS-IS/18] 18:20:14, metric 20
                    >  to 10.2.1.6 via xe-0/0/1.0
10.2.2.0/31        *[IS-IS/18] 18:19:47, metric 20
                    >  to 10.2.2.6 via xe-0/0/2.0
10.2.2.2/31        *[IS-IS/18] 18:19:47, metric 20
                    >  to 10.2.2.6 via xe-0/0/2.0
10.2.2.4/31        *[IS-IS/18] 18:19:47, metric 20
                    >  to 10.2.2.6 via xe-0/0/2.0
10.2.2.8/31        *[IS-IS/18] 18:19:47, metric 20
                    >  to 10.2.2.6 via xe-0/0/2.0
10.2.2.10/31       *[IS-IS/18] 18:19:47, metric 20
                    >  to 10.2.2.6 via xe-0/0/2.0
```
</details>

<details>
<summary>BorderLeaf1</summary>

``` text
root@BorderLeaf1> show route protocol isis 

inet.0: 24 destinations, 24 routes (24 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.0.1.0/32        *[IS-IS/18] 18:19:40, metric 10
                    >  to 10.2.1.8 via xe-0/0/1.0
10.0.1.1/32        *[IS-IS/18] 18:19:15, metric 20
                       to 10.2.1.8 via xe-0/0/1.0
                    >  to 10.2.2.8 via xe-0/0/2.0
10.0.1.2/32        *[IS-IS/18] 18:19:15, metric 20
                       to 10.2.1.8 via xe-0/0/1.0
                    >  to 10.2.2.8 via xe-0/0/2.0
10.0.1.3/32        *[IS-IS/18] 18:19:15, metric 20
                       to 10.2.1.8 via xe-0/0/1.0
                    >  to 10.2.2.8 via xe-0/0/2.0
10.0.1.4/32        *[IS-IS/18] 18:19:15, metric 20
                       to 10.2.1.8 via xe-0/0/1.0
                    >  to 10.2.2.8 via xe-0/0/2.0
10.0.1.6/32        *[IS-IS/18] 18:18:28, metric 20
                       to 10.2.1.8 via xe-0/0/1.0
                    >  to 10.2.2.8 via xe-0/0/2.0
10.0.2.0/32        *[IS-IS/18] 18:19:15, metric 10
                    >  to 10.2.2.8 via xe-0/0/2.0
10.2.1.0/31        *[IS-IS/18] 18:19:40, metric 20
                    >  to 10.2.1.8 via xe-0/0/1.0
10.2.1.2/31        *[IS-IS/18] 18:19:40, metric 20
                    >  to 10.2.1.8 via xe-0/0/1.0
10.2.1.4/31        *[IS-IS/18] 18:19:40, metric 20
                    >  to 10.2.1.8 via xe-0/0/1.0
10.2.1.6/31        *[IS-IS/18] 18:19:40, metric 20
                    >  to 10.2.1.8 via xe-0/0/1.0
10.2.1.10/31       *[IS-IS/18] 18:19:40, metric 20
                    >  to 10.2.1.8 via xe-0/0/1.0
10.2.2.0/31        *[IS-IS/18] 18:19:15, metric 20
                    >  to 10.2.2.8 via xe-0/0/2.0
10.2.2.2/31        *[IS-IS/18] 18:19:15, metric 20
                    >  to 10.2.2.8 via xe-0/0/2.0
10.2.2.4/31        *[IS-IS/18] 18:19:15, metric 20
                    >  to 10.2.2.8 via xe-0/0/2.0
10.2.2.6/31        *[IS-IS/18] 18:19:15, metric 20
                    >  to 10.2.2.8 via xe-0/0/2.0
10.2.2.10/31       *[IS-IS/18] 18:19:15, metric 20
                    >  to 10.2.2.8 via xe-0/0/2.0
```
</details>

<details>
<summary>BorderLeaf2</summary>

``` text
root@BorderLeaf2> show route protocol isis 

inet.0: 24 destinations, 24 routes (24 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

10.0.1.0/32        *[IS-IS/18] 18:19:21, metric 10
                    >  to 10.2.1.10 via xe-0/0/1.0
10.0.1.1/32        *[IS-IS/18] 18:18:55, metric 20
                       to 10.2.1.10 via xe-0/0/1.0
                    >  to 10.2.2.10 via xe-0/0/2.0
10.0.1.2/32        *[IS-IS/18] 18:18:55, metric 20
                       to 10.2.1.10 via xe-0/0/1.0
                    >  to 10.2.2.10 via xe-0/0/2.0
10.0.1.3/32        *[IS-IS/18] 18:18:55, metric 20
                       to 10.2.1.10 via xe-0/0/1.0
                    >  to 10.2.2.10 via xe-0/0/2.0
10.0.1.4/32        *[IS-IS/18] 18:18:55, metric 20
                       to 10.2.1.10 via xe-0/0/1.0
                    >  to 10.2.2.10 via xe-0/0/2.0
10.0.1.5/32        *[IS-IS/18] 18:18:55, metric 20
                       to 10.2.1.10 via xe-0/0/1.0
                    >  to 10.2.2.10 via xe-0/0/2.0
10.0.2.0/32        *[IS-IS/18] 18:18:55, metric 10
                    >  to 10.2.2.10 via xe-0/0/2.0
10.2.1.0/31        *[IS-IS/18] 18:19:21, metric 20
                    >  to 10.2.1.10 via xe-0/0/1.0
10.2.1.2/31        *[IS-IS/18] 18:19:21, metric 20
                    >  to 10.2.1.10 via xe-0/0/1.0
10.2.1.4/31        *[IS-IS/18] 18:19:21, metric 20
                    >  to 10.2.1.10 via xe-0/0/1.0
10.2.1.6/31        *[IS-IS/18] 18:19:21, metric 20
                    >  to 10.2.1.10 via xe-0/0/1.0
10.2.1.8/31        *[IS-IS/18] 18:19:21, metric 20
                    >  to 10.2.1.10 via xe-0/0/1.0
10.2.2.0/31        *[IS-IS/18] 18:18:55, metric 20
                    >  to 10.2.2.10 via xe-0/0/2.0
10.2.2.2/31        *[IS-IS/18] 18:18:55, metric 20
                    >  to 10.2.2.10 via xe-0/0/2.0
10.2.2.4/31        *[IS-IS/18] 18:18:55, metric 20
                    >  to 10.2.2.10 via xe-0/0/2.0
10.2.2.6/31        *[IS-IS/18] 18:18:55, metric 20
                    >  to 10.2.2.10 via xe-0/0/2.0
10.2.2.8/31        *[IS-IS/18] 18:18:55, metric 20
                    >  to 10.2.2.10 via xe-0/0/2.0
```
</details>






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
---
**Конфигурация оборудования**

Вывод display set
<details>
<summary>Spine1</summary>

``` text
root@Spine1> show configuration | display set 
set version 20.3R1.8
set system host-name Spine1
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
set interfaces xe-0/0/1 unit 0 description "--- Spine1 - Leaf1 ---"
set interfaces xe-0/0/1 unit 0 family inet address 10.2.1.0/31
set interfaces xe-0/0/1 unit 0 family iso
set interfaces xe-0/0/2 unit 0 description "--- Spine1 - Leaf2 ---"
set interfaces xe-0/0/2 unit 0 family inet address 10.2.1.2/31
set interfaces xe-0/0/2 unit 0 family iso
set interfaces xe-0/0/3 unit 0 description "--- Spine1 - Leaf3 ---"
set interfaces xe-0/0/3 unit 0 family inet address 10.2.1.4/31
set interfaces xe-0/0/3 unit 0 family iso
set interfaces xe-0/0/4 unit 0 description "--- Spine1 - Leaf4 ---"
set interfaces xe-0/0/4 unit 0 family inet address 10.2.1.6/31
set interfaces xe-0/0/4 unit 0 family iso
set interfaces xe-0/0/5 unit 0 description "--- Spine1 - BorderLeaf1 ---"
set interfaces xe-0/0/5 unit 0 family inet address 10.2.1.8/31
set interfaces xe-0/0/5 unit 0 family iso
set interfaces xe-0/0/6 unit 0 description "--- Spine1 - BorderLeaf2 ---"
set interfaces xe-0/0/6 unit 0 family inet address 10.2.1.10/31
set interfaces xe-0/0/6 unit 0 family iso
set interfaces em0 unit 0 family inet dhcp
set interfaces em1 unit 0 family inet address 169.254.0.2/24
set interfaces lo0 unit 0 family inet address 10.0.1.0/32
set interfaces lo0 unit 0 family iso address 49.0078.0100.0000.1000.00
set forwarding-options storm-control-profiles default all
set routing-options router-id 10.0.1.0
set protocols isis interface xe-0/0/1.0 level 1 disable
set protocols isis interface xe-0/0/2.0 level 1 disable
set protocols isis interface xe-0/0/3.0 level 1 disable
set protocols isis interface xe-0/0/4.0 level 1 disable
set protocols isis interface xe-0/0/5.0 level 1 disable
set protocols isis interface xe-0/0/6.0 level 1 disable
set protocols isis interface lo0.0 level 1 disable
set protocols isis level 2 wide-metrics-only
set protocols igmp-snooping vlan default
set vlans default vlan-id 1
```
</details>
<details>
<summary>Spine2</summary>

``` text
root@Spine2# show | display set 
set version 20.3R1.8
set system host-name Spine2
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
set interfaces xe-0/0/1 unit 0 description "--- Spine2 - Leaf1 ---"
set interfaces xe-0/0/1 unit 0 family inet address 10.2.2.0/31
set interfaces xe-0/0/1 unit 0 family iso
set interfaces xe-0/0/2 unit 0 description "--- Spine2 - Leaf2 ---"
set interfaces xe-0/0/2 unit 0 family inet address 10.2.2.2/31
set interfaces xe-0/0/2 unit 0 family iso
set interfaces xe-0/0/3 unit 0 description "--- Spine2 - Leaf3 ---"
set interfaces xe-0/0/3 unit 0 family inet address 10.2.2.4/31
set interfaces xe-0/0/3 unit 0 family iso
set interfaces xe-0/0/4 unit 0 description "--- Spine2 - Leaf4 ---"
set interfaces xe-0/0/4 unit 0 family inet address 10.2.2.6/31
set interfaces xe-0/0/4 unit 0 family iso
set interfaces xe-0/0/5 unit 0 description "--- Spine2 - BorderLeaf1 ---"
set interfaces xe-0/0/5 unit 0 family inet address 10.2.2.8/31
set interfaces xe-0/0/5 unit 0 family iso
set interfaces xe-0/0/6 unit 0 description "--- Spine2 - BorderLeaf2 ---"
set interfaces xe-0/0/6 unit 0 family inet address 10.2.2.10/31
set interfaces xe-0/0/6 unit 0 family iso
set interfaces em0 unit 0 family inet dhcp
set interfaces em1 unit 0 family inet address 169.254.0.2/24
set interfaces lo0 unit 0 family inet address 10.0.2.0/32
set interfaces lo0 unit 0 family iso address 49.0078.0100.0000.2000.00
set forwarding-options storm-control-profiles default all
set routing-options router-id 10.0.2.0
set protocols isis interface xe-0/0/1.0 level 1 disable
set protocols isis interface xe-0/0/2.0 level 1 disable
set protocols isis interface xe-0/0/3.0 level 1 disable
set protocols isis interface xe-0/0/4.0 level 1 disable
set protocols isis interface xe-0/0/5.0 level 1 disable
set protocols isis interface xe-0/0/6.0 level 1 disable
set protocols isis interface lo0.0 level 1 disable
set protocols isis level 2 wide-metrics-only
set protocols igmp-snooping vlan default
set vlans default vlan-id 1
```
</details>
<details>
<summary>Leaf1</summary>

``` text
root@Leaf1> show configuration | display set 
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
set interfaces xe-0/0/1 unit 0 family iso
set interfaces xe-0/0/2 unit 0 description "--- Leaf1 - Spine2  ---"
set interfaces xe-0/0/2 unit 0 family inet address 10.2.2.1/31
set interfaces xe-0/0/2 unit 0 family iso
set interfaces em0 unit 0 family inet dhcp
set interfaces em1 unit 0 family inet address 169.254.0.2/24
set interfaces lo0 unit 0 family inet address 10.0.1.1/32
set interfaces lo0 unit 0 family iso address 49.0078.0100.0000.1001.00
set forwarding-options storm-control-profiles default all
set routing-options router-id 10.0.1.1
set protocols isis interface xe-0/0/1.0 level 1 disable
set protocols isis interface xe-0/0/2.0 level 1 disable
set protocols isis interface lo0.0 level 1 disable
set protocols isis level 2 wide-metrics-only
set protocols igmp-snooping vlan default
set vlans default vlan-id 1
```
</details>
<details>
<summary>Leaf2</summary>

``` text
root@Leaf2> show configuration | display set 
set version 20.3R1.8
set system host-name Leaf2
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
set interfaces xe-0/0/1 unit 0 description "--- Leaf2 - Spine1  ---"
set interfaces xe-0/0/1 unit 0 family inet address 10.2.1.3/31
set interfaces xe-0/0/1 unit 0 family iso
set interfaces xe-0/0/2 unit 0 description "--- Leaf2 - Spine2  ---"
set interfaces xe-0/0/2 unit 0 family inet address 10.2.2.3/31
set interfaces xe-0/0/2 unit 0 family iso
set interfaces em0 unit 0 family inet dhcp
set interfaces em1 unit 0 family inet address 169.254.0.2/24
set interfaces lo0 unit 0 family inet address 10.0.1.2/32
set interfaces lo0 unit 0 family iso address 49.0078.0100.0000.1002.00
set forwarding-options storm-control-profiles default all
set routing-options router-id 10.0.1.2
set protocols isis interface xe-0/0/1.0 level 1 disable
set protocols isis interface xe-0/0/2.0 level 1 disable
set protocols isis interface lo0.0 level 1 disable
set protocols isis level 2 wide-metrics-only
set protocols igmp-snooping vlan default
set vlans default vlan-id 1
```
</details>
<details>
<summary>Leaf3</summary>

``` text
root@Leaf3> show configuration | display set 
set version 20.3R1.8
set system host-name Leaf3
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
set interfaces xe-0/0/1 unit 0 description "--- Leaf3 - Spine1  ---"
set interfaces xe-0/0/1 unit 0 family inet address 10.2.1.5/31
set interfaces xe-0/0/1 unit 0 family iso
set interfaces xe-0/0/2 unit 0 description "--- Leaf3 - Spine2  ---"
set interfaces xe-0/0/2 unit 0 family inet address 10.2.2.5/31
set interfaces xe-0/0/2 unit 0 family iso
set interfaces em0 unit 0 family inet dhcp
set interfaces em1 unit 0 family inet address 169.254.0.2/24
set interfaces lo0 unit 0 family inet address 10.0.1.3/32
set interfaces lo0 unit 0 family iso address 49.0078.0100.0000.1003.00
set forwarding-options storm-control-profiles default all
set routing-options router-id 10.0.1.3
set protocols isis interface xe-0/0/1.0 level 1 disable
set protocols isis interface xe-0/0/2.0 level 1 disable
set protocols isis interface lo0.0 level 1 disable
set protocols isis level 2 wide-metrics-only
set protocols igmp-snooping vlan default
set vlans default vlan-id 1
```
</details>
<details>
<summary>Leaf4</summary>

``` text
root@Leaf4> show configuration | display set 
set version 20.3R1.8
set system host-name Leaf4
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
set interfaces xe-0/0/1 unit 0 description "--- Leaf4 - Spine1  ---"
set interfaces xe-0/0/1 unit 0 family inet address 10.2.1.7/31
set interfaces xe-0/0/1 unit 0 family iso
set interfaces xe-0/0/2 unit 0 description "--- Leaf4 - Spine2  ---"
set interfaces xe-0/0/2 unit 0 family inet address 10.2.2.7/31
set interfaces xe-0/0/2 unit 0 family iso
set interfaces em0 unit 0 family inet dhcp
set interfaces em1 unit 0 family inet address 169.254.0.2/24
set interfaces lo0 unit 0 family inet address 10.0.1.4/32
set interfaces lo0 unit 0 family iso address 49.0078.0100.0000.1004.00
set forwarding-options storm-control-profiles default all
set routing-options router-id 10.0.1.4
set protocols isis interface xe-0/0/1.0 level 1 disable
set protocols isis interface xe-0/0/2.0 level 1 disable
set protocols isis interface lo0.0 level 1 disable
set protocols isis level 2 wide-metrics-only
set protocols igmp-snooping vlan default
set vlans default vlan-id 1
```
</details>
<details>
<summary>BorderLeaf1</summary>

``` text
root@BorderLeaf1> show configuration | display set 
set version 20.3R1.8
set system host-name BorderLeaf1
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
set interfaces xe-0/0/1 unit 0 description "--- BorderLeaf1 - Spine1  ---"
set interfaces xe-0/0/1 unit 0 family inet address 10.2.1.9/31
set interfaces xe-0/0/1 unit 0 family iso
set interfaces xe-0/0/2 unit 0 description "--- BorderLeaf1 - Spine2  ---"
set interfaces xe-0/0/2 unit 0 family inet address 10.2.2.9/31
set interfaces xe-0/0/2 unit 0 family iso
set interfaces em0 unit 0 family inet dhcp
set interfaces em1 unit 0 family inet address 169.254.0.2/24
set interfaces lo0 unit 0 family inet address 10.0.1.5/32
set interfaces lo0 unit 0 family iso address 49.0078.0100.0000.1005.00
set forwarding-options storm-control-profiles default all
set routing-options router-id 10.0.1.5
set protocols isis interface xe-0/0/1.0 level 1 disable
set protocols isis interface xe-0/0/2.0 level 1 disable
set protocols isis interface lo0.0 level 1 disable
set protocols isis level 2 wide-metrics-only
set protocols igmp-snooping vlan default
set vlans default vlan-id 1
```
</details>
<details>
<summary>BorderLeaf2</summary>

``` text
root@BorderLeaf2# show | display set 
set version 20.3R1.8
set system host-name BorderLeaf2
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
set interfaces xe-0/0/1 unit 0 family inet address 10.2.1.11/31
set interfaces xe-0/0/1 unit 0 family iso
set interfaces xe-0/0/2 unit 0 family inet address 10.2.2.11/31
set interfaces xe-0/0/2 unit 0 family iso
set interfaces em0 unit 0 family inet dhcp
set interfaces em1 unit 0 family inet address 169.254.0.2/24
set interfaces lo0 unit 0 family inet address 10.0.1.6/32
set interfaces lo0 unit 0 family iso address 49.0078.0100.0000.1006.00
set forwarding-options storm-control-profiles default all
set routing-options router-id 10.0.1.6
set protocols isis interface xe-0/0/1.0 level 1 disable
set protocols isis interface xe-0/0/2.0 level 1 disable
set protocols isis interface lo0.0 level 1 disable
set protocols isis level 2 wide-metrics-only
set protocols igmp-snooping vlan default
set vlans default vlan-id 1
```
</details>
Конфигурация без display set
<details>
<summary>Spine1</summary>

```text
root@Spine1> show configuration 
## Last commit: 2025-03-21 12:04:48 UTC by root
version 20.3R1.8;
system {
    host-name Spine1;
    root-authentication {
        encrypted-password "$6$iOHCqldH$4Bv5iM.SYPDCPExs15aDwLgKfad9fLWfdv53fovFYghTXlJ9rQVFxA9yoIUOf58hSJrXKANdZ.3L2ZWXBPs1T0"; ## SECRET-DATA
        ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key"; ## SECRET-DATA
    }
    login {
        user vagrant {
            uid 2000;
            class super-user;
            authentication {
                ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key"; ## SECRET-DATA
            }
        }
    }
    services {
        ssh {
            root-login allow;
        }
        netconf {
            ssh;
        }
        rest {
            http {
                port 8080;
            }
            enable-explorer;
        }
    }
    syslog {                            
        user * {
            any emergency;
        }
        file messages {
            any notice;
            authorization info;
        }
        file interactive-commands {
            interactive-commands any;
        }
    }
    extensions {
        providers {
            juniper {
                license-type juniper deployment-scope commercial;
            }
            chef {
                license-type juniper deployment-scope commercial;
            }
        }
    }
}
interfaces {                            
    xe-0/0/1 {
        unit 0 {
            description "--- Spine1 - Leaf1 ---";
            family inet {
                address 10.2.1.0/31;
            }
            family iso;
        }
    }
    xe-0/0/2 {
        unit 0 {
            description "--- Spine1 - Leaf2 ---";
            family inet {
                address 10.2.1.2/31;
            }
            family iso;
        }
    }
    xe-0/0/3 {
        unit 0 {
            description "--- Spine1 - Leaf3 ---";
            family inet {
                address 10.2.1.4/31;    
            }
            family iso;
        }
    }
    xe-0/0/4 {
        unit 0 {
            description "--- Spine1 - Leaf4 ---";
            family inet {
                address 10.2.1.6/31;
            }
            family iso;
        }
    }
    xe-0/0/5 {
        unit 0 {
            description "--- Spine1 - BorderLeaf1 ---";
            family inet {
                address 10.2.1.8/31;
            }
            family iso;
        }
    }
    xe-0/0/6 {                          
        unit 0 {
            description "--- Spine1 - BorderLeaf2 ---";
            family inet {
                address 10.2.1.10/31;
            }
            family iso;
        }
    }
    em0 {
        unit 0 {
            family inet {
                dhcp;
            }
        }
    }
    em1 {
        unit 0 {
            family inet {
                address 169.254.0.2/24;
            }
        }
    }
    lo0 {                               
        unit 0 {
            family inet {
                address 10.0.1.0/32;
            }
            family iso {
                address 49.0078.0100.0000.1000.00;
            }
        }
    }
}
forwarding-options {
    storm-control-profiles default {
        all;
    }
}
routing-options {
    router-id 10.0.1.0;
}
protocols {
    isis {
        interface xe-0/0/1.0 {
            level 1 disable;
        }                               
        interface xe-0/0/2.0 {
            level 1 disable;
        }
        interface xe-0/0/3.0 {
            level 1 disable;
        }
        interface xe-0/0/4.0 {
            level 1 disable;
        }
        interface xe-0/0/5.0 {
            level 1 disable;
        }
        interface xe-0/0/6.0 {
            level 1 disable;
        }
        interface lo0.0 {
            level 1 disable;
        }
        level 2 wide-metrics-only;
    }
    igmp-snooping {
        vlan default;
    }                                   
}
vlans {
    default {
        vlan-id 1;
    }
}

{master:0}

```
</details>

<details>
<summary>Spine2</summary>

```text

root@Spine2# show 
## Last changed: 2025-03-21 12:09:21 UTC
version 20.3R1.8;
system {
    host-name Spine2;
    root-authentication {
        encrypted-password "$6$iOHCqldH$4Bv5iM.SYPDCPExs15aDwLgKfad9fLWfdv53fovFYghTXlJ9rQVFxA9yoIUOf58hSJrXKANdZ.3L2ZWXBPs1T0"; ## SECRET-DATA
        ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key"; ## SECRET-DATA
    }
    login {
        user vagrant {
            uid 2000;
            class super-user;
            authentication {
                ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key"; ## SECRET-DATA
            }
        }
    }
    services {
        ssh {
            root-login allow;
        }
        netconf {
            ssh;
        }
        rest {
            http {
                port 8080;
            }
            enable-explorer;
        }
    }
    syslog {                            
        user * {
            any emergency;
        }
        file messages {
            any notice;
            authorization info;
        }
        file interactive-commands {
            interactive-commands any;
        }
    }
    extensions {
        providers {
            juniper {
                license-type juniper deployment-scope commercial;
            }
            chef {
                license-type juniper deployment-scope commercial;
            }
        }
    }
}
interfaces {                            
    xe-0/0/1 {
        unit 0 {
            description "--- Spine2 - Leaf1 ---";
            family inet {
                address 10.2.2.0/31;
            }
            family iso;
        }
    }
    xe-0/0/2 {
        unit 0 {
            description "--- Spine2 - Leaf2 ---";
            family inet {
                address 10.2.2.2/31;
            }
            family iso;
        }
    }
    xe-0/0/3 {
        unit 0 {
            description "--- Spine2 - Leaf3 ---";
            family inet {
                address 10.2.2.4/31;    
            }
            family iso;
        }
    }
    xe-0/0/4 {
        unit 0 {
            description "--- Spine2 - Leaf4 ---";
            family inet {
                address 10.2.2.6/31;
            }
            family iso;
        }
    }
    xe-0/0/5 {
        unit 0 {
            description "--- Spine2 - BorderLeaf1 ---";
            family inet {
                address 10.2.2.8/31;
            }
            family iso;
        }
    }
    xe-0/0/6 {                          
        unit 0 {
            description "--- Spine2 - BorderLeaf2 ---";
            family inet {
                address 10.2.2.10/31;
            }
            family iso;
        }
    }
    em0 {
        unit 0 {
            family inet {
                dhcp;
            }
        }
    }
    em1 {
        unit 0 {
            family inet {
                address 169.254.0.2/24;
            }
        }
    }
    lo0 {                               
        unit 0 {
            family inet {
                address 10.0.2.0/32;
            }
            family iso {
                address 49.0078.0100.0000.2000.00;
            }
        }
    }
}
forwarding-options {
    storm-control-profiles default {
        all;
    }
}
routing-options {
    router-id 10.0.2.0;
}
protocols {
    isis {
        interface xe-0/0/1.0 {
            level 1 disable;
        }                               
        interface xe-0/0/2.0 {
            level 1 disable;
        }
        interface xe-0/0/3.0 {
            level 1 disable;
        }
        interface xe-0/0/4.0 {
            level 1 disable;
        }
        interface xe-0/0/5.0 {
            level 1 disable;
        }
        interface xe-0/0/6.0 {
            level 1 disable;
        }
        interface lo0.0 {
            level 1 disable;
        }
        level 2 wide-metrics-only;
    }
    igmp-snooping {
        vlan default;
    }                                   
}
vlans {
    default {
        vlan-id 1;
    }
}
```
</details>

<details>
<summary>Leaf1</summary>

```text
root@Leaf1> show configuration 
## Last commit: 2025-03-21 12:14:11 UTC by root
version 20.3R1.8;
system {
    host-name Leaf1;
    root-authentication {
        encrypted-password "$6$iOHCqldH$4Bv5iM.SYPDCPExs15aDwLgKfad9fLWfdv53fovFYghTXlJ9rQVFxA9yoIUOf58hSJrXKANdZ.3L2ZWXBPs1T0"; ## SECRET-DATA
        ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key"; ## SECRET-DATA
    }
    login {
        user vagrant {
            uid 2000;
            class super-user;
            authentication {
                ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key"; ## SECRET-DATA
            }
        }
    }
    services {
        ssh {
            root-login allow;
        }
        netconf {
            ssh;
        }
        rest {
            http {
                port 8080;
            }
            enable-explorer;
        }
    }
    syslog {                            
        user * {
            any emergency;
        }
        file messages {
            any notice;
            authorization info;
        }
        file interactive-commands {
            interactive-commands any;
        }
    }
    extensions {
        providers {
            juniper {
                license-type juniper deployment-scope commercial;
            }
            chef {
                license-type juniper deployment-scope commercial;
            }
        }
    }
}
interfaces {                            
    xe-0/0/1 {
        unit 0 {
            description "--- Leaf1 - Spine1  ---";
            family inet {
                address 10.2.1.1/31;
            }
            family iso;
        }
    }
    xe-0/0/2 {
        unit 0 {
            description "--- Leaf1 - Spine2  ---";
            family inet {
                address 10.2.2.1/31;
            }
            family iso;
        }
    }
    em0 {
        unit 0 {
            family inet {
                dhcp;
            }                           
        }
    }
    em1 {
        unit 0 {
            family inet {
                address 169.254.0.2/24;
            }
        }
    }
    lo0 {
        unit 0 {
            family inet {
                address 10.0.1.1/32;
            }
            family iso {
                address 49.0078.0100.0000.1001.00;
            }
        }
    }
}
forwarding-options {
    storm-control-profiles default {
        all;                            
    }
}
routing-options {
    router-id 10.0.1.1;
}
protocols {
    isis {
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
    }
    igmp-snooping {
        vlan default;
    }
}
vlans {                                 
    default {
        vlan-id 1;
    }
}

{master:0}
```
</details>

<details>
<summary>Leaf2</summary>

```text
root@Leaf2> show configuration 
## Last commit: 2025-03-21 12:17:51 UTC by root
version 20.3R1.8;
system {
    host-name Leaf2;
    root-authentication {
        encrypted-password "$6$iOHCqldH$4Bv5iM.SYPDCPExs15aDwLgKfad9fLWfdv53fovFYghTXlJ9rQVFxA9yoIUOf58hSJrXKANdZ.3L2ZWXBPs1T0"; ## SECRET-DATA
        ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key"; ## SECRET-DATA
    }
    login {
        user vagrant {
            uid 2000;
            class super-user;
            authentication {
                ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key"; ## SECRET-DATA
            }
        }
    }
    services {
        ssh {
            root-login allow;
        }
        netconf {
            ssh;
        }
        rest {
            http {
                port 8080;
            }
            enable-explorer;
        }
    }
    syslog {                            
        user * {
            any emergency;
        }
        file messages {
            any notice;
            authorization info;
        }
        file interactive-commands {
            interactive-commands any;
        }
    }
    extensions {
        providers {
            juniper {
                license-type juniper deployment-scope commercial;
            }
            chef {
                license-type juniper deployment-scope commercial;
            }
        }
    }
}
interfaces {                            
    xe-0/0/1 {
        unit 0 {
            description "--- Leaf2 - Spine1  ---";
            family inet {
                address 10.2.1.3/31;
            }
            family iso;
        }
    }
    xe-0/0/2 {
        unit 0 {
            description "--- Leaf2 - Spine2  ---";
            family inet {
                address 10.2.2.3/31;
            }
            family iso;
        }
    }
    em0 {
        unit 0 {
            family inet {
                dhcp;
            }                           
        }
    }
    em1 {
        unit 0 {
            family inet {
                address 169.254.0.2/24;
            }
        }
    }
    lo0 {
        unit 0 {
            family inet {
                address 10.0.1.2/32;
            }
            family iso {
                address 49.0078.0100.0000.1002.00;
            }
        }
    }
}
forwarding-options {
    storm-control-profiles default {
        all;                            
    }
}
routing-options {
    router-id 10.0.1.2;
}
protocols {
    isis {
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
    }
    igmp-snooping {
        vlan default;
    }
}
vlans {                                 
    default {
        vlan-id 1;
    }
}

{master:0}
```
</details>

<details>
<summary>Leaf3</summary>

```text
root@Leaf3> show configuration 
## Last commit: 2025-03-21 12:17:55 UTC by root
version 20.3R1.8;
system {
    host-name Leaf3;
    root-authentication {
        encrypted-password "$6$iOHCqldH$4Bv5iM.SYPDCPExs15aDwLgKfad9fLWfdv53fovFYghTXlJ9rQVFxA9yoIUOf58hSJrXKANdZ.3L2ZWXBPs1T0"; ## SECRET-DATA
        ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key"; ## SECRET-DATA
    }
    login {
        user vagrant {
            uid 2000;
            class super-user;
            authentication {
                ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key"; ## SECRET-DATA
            }
        }
    }
    services {
        ssh {
            root-login allow;
        }
        netconf {
            ssh;
        }
        rest {
            http {
                port 8080;
            }
            enable-explorer;
        }
    }
    syslog {                            
        user * {
            any emergency;
        }
        file messages {
            any notice;
            authorization info;
        }
        file interactive-commands {
            interactive-commands any;
        }
    }
    extensions {
        providers {
            juniper {
                license-type juniper deployment-scope commercial;
            }
            chef {
                license-type juniper deployment-scope commercial;
            }
        }
    }
}
interfaces {                            
    xe-0/0/1 {
        unit 0 {
            description "--- Leaf3 - Spine1  ---";
            family inet {
                address 10.2.1.5/31;
            }
            family iso;
        }
    }
    xe-0/0/2 {
        unit 0 {
            description "--- Leaf3 - Spine2  ---";
            family inet {
                address 10.2.2.5/31;
            }
            family iso;
        }
    }
    em0 {
        unit 0 {
            family inet {
                dhcp;
            }                           
        }
    }
    em1 {
        unit 0 {
            family inet {
                address 169.254.0.2/24;
            }
        }
    }
    lo0 {
        unit 0 {
            family inet {
                address 10.0.1.3/32;
            }
            family iso {
                address 49.0078.0100.0000.1003.00;
            }
        }
    }
}
forwarding-options {
    storm-control-profiles default {
        all;                            
    }
}
routing-options {
    router-id 10.0.1.3;
}
protocols {
    isis {
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
    }
    igmp-snooping {
        vlan default;
    }
}
vlans {                                 
    default {
        vlan-id 1;
    }
}

{master:0}
```
</details>

<details>
<summary>Leaf4</summary>

```text
root@Leaf4> show configuration 
## Last commit: 2025-03-21 12:17:55 UTC by root
version 20.3R1.8;
system {
    host-name Leaf4;
    root-authentication {
        encrypted-password "$6$iOHCqldH$4Bv5iM.SYPDCPExs15aDwLgKfad9fLWfdv53fovFYghTXlJ9rQVFxA9yoIUOf58hSJrXKANdZ.3L2ZWXBPs1T0"; ## SECRET-DATA
        ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key"; ## SECRET-DATA
    }
    login {
        user vagrant {
            uid 2000;
            class super-user;
            authentication {
                ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key"; ## SECRET-DATA
            }
        }
    }
    services {
        ssh {
            root-login allow;
        }
        netconf {
            ssh;
        }
        rest {
            http {
                port 8080;
            }
            enable-explorer;
        }
    }
    syslog {                            
        user * {
            any emergency;
        }
        file messages {
            any notice;
            authorization info;
        }
        file interactive-commands {
            interactive-commands any;
        }
    }
    extensions {
        providers {
            juniper {
                license-type juniper deployment-scope commercial;
            }
            chef {
                license-type juniper deployment-scope commercial;
            }
        }
    }
}
interfaces {                            
    xe-0/0/1 {
        unit 0 {
            description "--- Leaf4 - Spine1  ---";
            family inet {
                address 10.2.1.7/31;
            }
            family iso;
        }
    }
    xe-0/0/2 {
        unit 0 {
            description "--- Leaf4 - Spine2  ---";
            family inet {
                address 10.2.2.7/31;
            }
            family iso;
        }
    }
    em0 {
        unit 0 {
            family inet {
                dhcp;
            }                           
        }
    }
    em1 {
        unit 0 {
            family inet {
                address 169.254.0.2/24;
            }
        }
    }
    lo0 {
        unit 0 {
            family inet {
                address 10.0.1.4/32;
            }
            family iso {
                address 49.0078.0100.0000.1004.00;
            }
        }
    }
}
forwarding-options {
    storm-control-profiles default {
        all;                            
    }
}
routing-options {
    router-id 10.0.1.4;
}
protocols {
    isis {
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
    }
    igmp-snooping {
        vlan default;
    }
}
vlans {                                 
    default {
        vlan-id 1;
    }
}

{master:0}
```
</details>

<details>
<summary>BorderLeaf1</summary>

```text
root@BorderLeaf1> show configuration 
## Last commit: 2025-03-21 12:17:55 UTC by root
version 20.3R1.8;
system {
    host-name BorderLeaf1;
    root-authentication {
        encrypted-password "$6$iOHCqldH$4Bv5iM.SYPDCPExs15aDwLgKfad9fLWfdv53fovFYghTXlJ9rQVFxA9yoIUOf58hSJrXKANdZ.3L2ZWXBPs1T0"; ## SECRET-DATA
        ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key"; ## SECRET-DATA
    }
    login {
        user vagrant {
            uid 2000;
            class super-user;
            authentication {
                ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key"; ## SECRET-DATA
            }
        }
    }
    services {
        ssh {
            root-login allow;
        }
        netconf {
            ssh;
        }
        rest {
            http {
                port 8080;
            }
            enable-explorer;
        }
    }
    syslog {                            
        user * {
            any emergency;
        }
        file messages {
            any notice;
            authorization info;
        }
        file interactive-commands {
            interactive-commands any;
        }
    }
    extensions {
        providers {
            juniper {
                license-type juniper deployment-scope commercial;
            }
            chef {
                license-type juniper deployment-scope commercial;
            }
        }
    }
}
interfaces {                            
    xe-0/0/1 {
        unit 0 {
            description "--- BorderLeaf1 - Spine1  ---";
            family inet {
                address 10.2.1.9/31;
            }
            family iso;
        }
    }
    xe-0/0/2 {
        unit 0 {
            description "--- BorderLeaf1 - Spine2  ---";
            family inet {
                address 10.2.2.9/31;
            }
            family iso;
        }
    }
    em0 {
        unit 0 {
            family inet {
                dhcp;
            }                           
        }
    }
    em1 {
        unit 0 {
            family inet {
                address 169.254.0.2/24;
            }
        }
    }
    lo0 {
        unit 0 {
            family inet {
                address 10.0.1.5/32;
            }
            family iso {
                address 49.0078.0100.0000.1005.00;
            }
        }
    }
}
forwarding-options {
    storm-control-profiles default {
        all;                            
    }
}
routing-options {
    router-id 10.0.1.5;
}
protocols {
    isis {
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
    }
    igmp-snooping {
        vlan default;
    }
}
vlans {                                 
    default {
        vlan-id 1;
    }
}

{master:0}
```
</details>

<details>
<summary>BorderLeaf2</summary>

```text
root@BorderLeaf2# show 
## Last changed: 2025-03-21 11:58:48 UTC
version 20.3R1.8;
system {
    host-name BorderLeaf2;
    root-authentication {
        encrypted-password "$6$iOHCqldH$4Bv5iM.SYPDCPExs15aDwLgKfad9fLWfdv53fovFYghTXlJ9rQVFxA9yoIUOf58hSJrXKANdZ.3L2ZWXBPs1T0"; ## SECRET-DATA
        ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key"; ## SECRET-DATA
    }
    login {
        user vagrant {
            uid 2000;
            class super-user;
            authentication {
                ssh-rsa "ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key"; ## SECRET-DATA
            }
        }
    }
    services {
        ssh {
            root-login allow;
        }
        netconf {
            ssh;
        }
        rest {
            http {
                port 8080;
            }
            enable-explorer;
        }
    }
    syslog {                            
        user * {
            any emergency;
        }
        file messages {
            any notice;
            authorization info;
        }
        file interactive-commands {
            interactive-commands any;
        }
    }
    extensions {
        providers {
            juniper {
                license-type juniper deployment-scope commercial;
            }
            chef {
                license-type juniper deployment-scope commercial;
            }
        }
    }
}
interfaces {                            
    xe-0/0/1 {
        unit 0 {
            family inet {
                address 10.2.1.11/31;
            }
            family iso;
        }
    }
    xe-0/0/2 {
        unit 0 {
            family inet {
                address 10.2.2.11/31;
            }
            family iso;
        }
    }
    em0 {
        unit 0 {
            family inet {
                dhcp;
            }
        }
    }                                   
    em1 {
        unit 0 {
            family inet {
                address 169.254.0.2/24;
            }
        }
    }
    lo0 {
        unit 0 {
            family inet {
                address 10.0.1.6/32;
            }
            family iso {
                address 49.0078.0100.0000.1006.00;
            }
        }
    }
}
forwarding-options {
    storm-control-profiles default {
        all;
    }
}                                       
routing-options {
    router-id 10.0.1.6;
}
protocols {
    isis {
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
    }
    igmp-snooping {
        vlan default;
    }
}
vlans {
    default {
        vlan-id 1;                      
    }
}
```
</details>