# LAB-3

## IS-IS в UNDELAY сети

---

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

Настраиваем интерфейсы
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
```text
root@Leaf1> show isis adjacency 
Interface             System         L State        Hold (secs) SNPA
xe-0/0/1.0            Spine1         2  Up                    7  2:5:86:71:d6:7
xe-0/0/2.0            Spine2         2  Up                    8  2:5:86:71:71:7
```
```text
root@Leaf2> show isis adjacency 
Interface             System         L State        Hold (secs) SNPA
xe-0/0/1.0            Spine1         2  Up                   18  2:5:86:71:d6:b
xe-0/0/2.0            Spine2         2  Up                   22  2:5:86:71:71:b
```
```text
root@Leaf3> show isis adjacency 
Interface             System         L State        Hold (secs) SNPA
xe-0/0/1.0            Spine1         2  Up                    6  2:5:86:71:d6:f
xe-0/0/2.0            Spine2         2  Up                    7  2:5:86:71:71:f
```
```text
root@Leaf4> show isis adjacency 
Interface             System         L State        Hold (secs) SNPA
xe-0/0/1.0            Spine1         2  Up                   23  2:5:86:71:d6:13
xe-0/0/2.0            Spine2         2  Up                   22  2:5:86:71:71:13
```
```text
root@BorderLeaf1> show isis adjacency 
Interface             System         L State        Hold (secs) SNPA
xe-0/0/1.0            Spine1         2  Up                    8  2:5:86:71:d6:17
xe-0/0/2.0            Spine2         2  Up                    7  2:5:86:71:71:17
```
```text
root@BorderLeaf2> show isis adjacency 
Interface             System         L State        Hold (secs) SNPA
xe-0/0/1.0            Spine1         2  Up                    7  2:5:86:71:d6:1b
xe-0/0/2.0            Spine2         2  Up                   24  2:5:86:71:71:1b
```
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
```text
root@Leaf1> show isis interface 
IS-IS interface database:
Interface             L CirID Level 1 DR        Level 2 DR        L1/L2 Metric
lo0.0                 3   0x1 Disabled          Passive                 0/0
xe-0/0/1.0            2   0x1 Disabled          Spine1.02              10/10
xe-0/0/2.0            2   0x1 Disabled          Spine2.02              10/10
```
```text
root@Leaf2> show isis interface 
IS-IS interface database:
Interface             L CirID Level 1 DR        Level 2 DR        L1/L2 Metric
lo0.0                 3   0x1 Disabled          Passive                 0/0
xe-0/0/1.0            2   0x2 Disabled          Leaf2.02               10/10
xe-0/0/2.0            2   0x3 Disabled          Leaf2.03               10/10
```
```text
root@Leaf3> show isis interface 
IS-IS interface database:
Interface             L CirID Level 1 DR        Level 2 DR        L1/L2 Metric
lo0.0                 3   0x1 Disabled          Passive                 0/0
xe-0/0/1.0            2   0x1 Disabled          Spine1.03              10/10
xe-0/0/2.0            2   0x1 Disabled          Spine2.03              10/10
```
```text
root@Leaf4> show isis interface 
IS-IS interface database:
Interface             L CirID Level 1 DR        Level 2 DR        L1/L2 Metric
lo0.0                 3   0x1 Disabled          Passive                 0/0
xe-0/0/1.0            2   0x2 Disabled          Leaf4.02               10/10
xe-0/0/2.0            2   0x3 Disabled          Leaf4.03               10/10
```
```text
root@BorderLeaf1> show isis interface 
IS-IS interface database:
Interface             L CirID Level 1 DR        Level 2 DR        L1/L2 Metric
lo0.0                 3   0x1 Disabled          Passive                 0/0
xe-0/0/1.0            2   0x1 Disabled          Spine1.04              10/10
xe-0/0/2.0            2   0x1 Disabled          Spine2.04              10/10
```
```text
root@BorderLeaf2> show isis interface 
IS-IS interface database:
Interface             L CirID Level 1 DR        Level 2 DR        L1/L2 Metric
lo0.0                 3   0x1 Disabled          Passive                 0/0
xe-0/0/1.0            2   0x1 Disabled          Spine1.05              10/10
xe-0/0/2.0            2   0x2 Disabled          BorderLeaf2.02         10/10
```
