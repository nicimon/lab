# LAB-2

## OSPF в UNDELAY сети

---
### Схема связи и адресное пространство
Схема и адресное пространство взято из LAB-1

![img_2.png](screenshots/laba2.png)

<details>
<summary>План адресного пространства</summary>
Суммарный для Lo0 и Lo2 – 10.0.0.0/15

Loopack-s:
|             | Lo0 /32  |
|-------------|----------|
| Spine1      | 10.0.1.0 |
| Spine2      | 10.0.2.0 |

|             | Lo0 /32  |
|-------------|----------|
| Leaf1       | 10.0.1.1 |
| Leaf2       | 10.0.1.2 |
| Leaf3       | 10.0.1.3 |
| Leaf4       | 10.0.1.4 |
| BorderLeaf1 | 10.0.1.5 |
| BorderLeaf2 | 10.0.1.6 |
---
Суммарный для p2p и резерва – 10.2.0.0/15
| **Connection**   	| **Spine Address** 	| **Leaf Address** 	| **Subnet**  	|
|------------------	|-------------------	|------------------	|-------------	|
| Spine 1 → Leaf 1 	| 10.2.1.0          	| 10.2.1.1         	| 10.2.1.0/31 	|
| Spine 1 → Leaf 2 	| 10.2.1.2              | 10.2.1.3          | 10.2.1.2/31  	|
| Spine 1 → Leaf 3 	| 10.2.1.4              | 10.2.1.5          | 10.2.1.4/31  	|
| Spine 1 → Leaf 4 	| 10.2.1.6          	| 10.2.1.7         	| 10.2.1.6/31 	|
| Spine 1 → BorderLeaf 1 | 10.2.1.8         | 10.2.1.9          | 10.2.1.8/31  	|
| Spine 1 → BorderLeaf 2 | 10.2.1.10        | 10.2.1.11        	| 10.2.1.10/31 	|
|------------------	|----------------------	|------------------	|-------------	|
| Spine 2 → Leaf 1 	| 10.2.2.0              | 10.2.2.1          | 10.2.2.0/31  	|
| Spine 2 → Leaf 2 	| 10.2.2.2              | 10.2.2.3          | 10.2.1.2/31  	|
| Spine 2 → Leaf 3 	| 10.2.2.4              | 10.2.2.5          | 10.2.1.4/31  	|
| Spine 2 → Leaf 4 	| 10.2.2.6              | 10.2.2.7          | 10.2.1.6/31  	|
| Spine 2 → BorderLeaf 1 | 10.2.2.8         | 10.2.2.9         | 10.2.1.8/31  	|
| Spine 2 → BorderLeaf 2 | 10.2.2.10        | 10.2.2.11          | 10.2.1.10/31  |

---

### IP установлены следующим образом

root@Spine1> show interfaces terse | match "10.[0,1,2]"
```text
xe-0/0/1.0              up    up   inet     10.2.1.0/31     
xe-0/0/2.0              up    up   inet     10.2.1.2/31     
xe-0/0/3.0              up    up   inet     10.2.1.4/31     
xe-0/0/4.0              up    up   inet     10.2.1.6/31     
xe-0/0/5.0              up    up   inet     10.2.1.8/31     
xe-0/0/6.0              up    up   inet     10.2.1.10/31    
lo0.0                   up    up   inet     10.0.1.0            --> 0/0
```
root@Spine2> show interfaces terse | match "10.[0,1,2]"
```text
xe-0/0/1.0              up    up   inet     10.2.2.0/31     
xe-0/0/2.0              up    up   inet     10.2.2.2/31     
xe-0/0/3.0              up    up   inet     10.2.2.4/31     
xe-0/0/4.0              up    up   inet     10.2.2.6/31     
xe-0/0/5.0              up    up   inet     10.2.2.8/31     
xe-0/0/6.0              up    up   inet     10.2.2.10/31      
lo0.0                   up    up   inet     10.0.2.0            --> 0/0
```
root@Leaf1> show interfaces terse | match "10.[0,1,2]" 
```text
xe-0/0/1.0              up    up   inet     10.2.1.1/31     
xe-0/0/2.0              up    up   inet     10.2.2.1/31     
lo0.0                   up    up   inet     10.0.1.1            --> 0/0
```
root@Leaf2> show interfaces terse | match "10.[0,1,2]" 
```text
xe-0/0/1.0              up    up   inet     10.2.1.3/31     
xe-0/0/2.0              up    up   inet     10.2.2.3/31     
lo0.0                   up    up   inet     10.0.1.2            --> 0/0
```
root@Leaf3> show interfaces terse | match "10.[0,1,2]"  
  ```text
xe-0/0/1.0              up    up   inet     10.2.1.5/31     
xe-0/0/2.0              up    up   inet     10.2.2.5/31     
lo0.0                   up    up   inet     10.0.1.3            --> 0/0
```
root@Leaf4> show interfaces terse | match "10.[0,1,2]"    
```text
xe-0/0/1.0              up    up   inet     10.2.1.7/31     
xe-0/0/2.0              up    up   inet     10.2.2.7/31     
lo0.0                   up    up   inet     10.0.1.4            --> 0/0
```
root@BorderLeaf1> show interfaces terse | match "10.[0,1,2]" 
```text
xe-0/0/1.0              up    up   inet     10.2.1.9/31     
xe-0/0/2.0              up    up   inet     10.2.2.9/31     
lo0.0                   up    up   inet     10.0.1.5            --> 0/0
```
root@BorderLeaf2> show interfaces terse | match "10.[0,1,2]" 
```text
xe-0/0/1.0              up    up   inet     10.2.1.11/31    
xe-0/0/2.0              up    up   inet     10.2.2.11/31    
lo0.0                   up    up   inet     10.0.1.6            --> 0/0
```
</details>

Для настройки OSPF необходимо в секцию  protocols ospf указать зону и внести участвующие интерфейсы в процессе OSPF, а также указать router-id
<details>
<summary>Spine1</summary>

root@Spine1> show configuration protocols ospf
```text
area 0.0.0.0 {
    interface xe-0/0/1.0 {
        interface-type p2p;
    }
    interface xe-0/0/2.0 {
        interface-type p2p;
    }
    interface xe-0/0/3.0 {
        interface-type p2p;
    }
    interface xe-0/0/4.0 {
        interface-type p2p;
    }
    interface xe-0/0/5.0 {
        interface-type p2p;
    }
    interface xe-0/0/6.0 {
        interface-type p2p;
    }
    interface lo0.0;
}
reference-bandwidth 100g;

root@Spine1> show configuration routing-options 
router-id 10.0.1.0;
```
</details>

<details>
<summary>Spine2</summary>

root@Spine2> show configuration protocols ospf
```text
area 0.0.0.0 {
    interface xe-0/0/1.0 {
        interface-type p2p;
    }
    interface xe-0/0/2.0 {
        interface-type p2p;
    }
    interface xe-0/0/3.0 {
        interface-type p2p;
    }
    interface xe-0/0/4.0 {
        interface-type p2p;
    }
    interface xe-0/0/5.0 {
        interface-type p2p;
    }
    interface xe-0/0/6.0 {
        interface-type p2p;
    }
    interface lo0.0;
}
reference-bandwidth 100g;

{master:0}
root@Spine2> show configuration routing-options 
router-id 10.0.2.0;
```
</details>

<details>
<summary>Leaf1</summary>

root@Leaf1> show configuration protocols ospf 
```text
area 0.0.0.0 {
    interface xe-0/0/1.0 {
        interface-type p2p;
    }
    interface xe-0/0/2.0 {
        interface-type p2p;
    }
    interface lo0.0;
}
reference-bandwidth 100g;

{master:0}
root@Leaf1> show configuration routing-options 
router-id 10.0.1.1;
```
</details>

<details>
<summary>Leaf2</summary>

root@Leaf2> show configuration protocols ospf
```text
area 0.0.0.0 {
    interface xe-0/0/1.0 {
        interface-type p2p;
    }
    interface xe-0/0/2.0 {
        interface-type p2p;
    }
    interface lo0.0;
}
reference-bandwidth 100g;

{master:0}
root@Leaf2> show configuration routing-options 
router-id 10.0.1.2;
```
</details>
<details>

<summary>Leaf3</summary>

root@Leaf3> show configuration protocols ospf
```text
area 0.0.0.0 {
    interface xe-0/0/1.0 {
        interface-type p2p;
    }
    interface xe-0/0/2.0 {
        interface-type p2p;
    }
    interface lo0.0;
}
reference-bandwidth 100g;

{master:0}
root@Leaf3> show configuration routing-options 
router-id 10.0.1.3;
```
</details>

<details>
<summary>Leaf4</summary>

root@Leaf4> show configuration protocols ospf 
```text
area 0.0.0.0 {
    interface xe-0/0/1.0 {
        interface-type p2p;
    }
    interface xe-0/0/2.0 {
        interface-type p2p;
    }
    interface lo0.0;
}
reference-bandwidth 100g;

{master:0}
root@Leaf4> show configuration routing-options 
router-id 10.0.1.4;
```
</details>

<details>
<summary>BorderLeaf1</summary>

root@BorderLeaf1> show configuration protocols ospf
```text
area 0.0.0.0 {
    interface xe-0/0/1.0 {
        interface-type p2p;
    }
    interface xe-0/0/2.0 {
        interface-type p2p;
    }
    interface lo0.0;
}
reference-bandwidth 100g;

{master:0}
root@BorderLeaf1> show configuration routing-options 
router-id 10.0.1.5;
```
</details>

<details>
<summary>BorderLeaf2</summary>

root@BorderLeaf2> show configuration protocols ospf
```text
area 0.0.0.0 {
    interface xe-0/0/1.0 {
        interface-type p2p;
    }
    interface xe-0/0/2.0 {
        interface-type p2p;
    }
    interface lo0.0;
}
reference-bandwidth 100g
root@BorderLeaf2> show configuration routing-options 
router-id 10.0.1.6;
```
</details>

Проверяем соседство
<details>
<summary>Spine1</summary>

```text
root@Spine1> show ospf neighbor 
Address          Interface              State           ID               Pri  Dead
10.2.1.1         xe-0/0/1.0             Full            10.0.1.1         128    37
10.2.1.3         xe-0/0/2.0             Full            10.0.1.2         128    37
10.2.1.5         xe-0/0/3.0             Full            10.0.1.3         128    34
10.2.1.7         xe-0/0/4.0             Full            10.0.1.4         128    38
10.2.1.9         xe-0/0/5.0             Full            10.0.1.5         128    37
10.2.1.11        xe-0/0/6.0             Full            10.0.1.6         128    38
```
</details>
<details>
<summary>Spine2</summary>

```text
root@Spine2> show ospf neighbor 
Address          Interface              State           ID               Pri  Dead
10.2.2.1         xe-0/0/1.0             Full            10.0.1.1         128    32
10.2.2.3         xe-0/0/2.0             Full            10.0.1.2         128    34
10.2.2.5         xe-0/0/3.0             Full            10.0.1.3         128    33
10.2.2.7         xe-0/0/4.0             Full            10.0.1.4         128    33
10.2.2.9         xe-0/0/5.0             Full            10.0.1.5         128    36
10.2.2.11        xe-0/0/6.0             Full            10.0.1.6         128    31
```
</details>
<details>
<summary>Leaf1</summary>

```text
root@Leaf1> show ospf neighbor 
Address          Interface              State           ID               Pri  Dead
10.2.1.0         xe-0/0/1.0             Full            10.0.1.0         128    37
10.2.2.0         xe-0/0/2.0             Full            10.0.2.0         128    32
```
</details>
<details>
<summary>Leaf2</summary>

```text
root@Leaf2> show ospf neighbor 
Address          Interface              State           ID               Pri  Dead
10.2.1.2         xe-0/0/1.0             Full            10.0.1.0         128    38
10.2.2.2         xe-0/0/2.0             Full            10.0.2.0         128    35
```
</details>
<details>
<summary>Leaf3</summary>

```text
root@Leaf3> show ospf neighbor 
Address          Interface              State           ID               Pri  Dead
10.2.1.4         xe-0/0/1.0             Full            10.0.1.0         128    33
10.2.2.4         xe-0/0/2.0             Full            10.0.2.0         128    36
```
</details>
<details>
<summary>Leaf4</summary>

```text
root@Leaf4> show ospf neighbor 
Address          Interface              State           ID               Pri  Dead
10.2.1.6         xe-0/0/1.0             Full            10.0.1.0         128    39
10.2.2.6         xe-0/0/2.0             Full            10.0.2.0         128    39
```
</details>
<details>
<summary>BorderLeaf1</summary>

```text
root@BorderLeaf1> show ospf neighbor 
Address          Interface              State           ID               Pri  Dead
10.2.1.8         xe-0/0/1.0             Full            10.0.1.0         128    35
10.2.2.8         xe-0/0/2.0             Full            10.0.2.0         128    35
```
</details>
<details>
<summary>BorderLeaf2</summary>

```text
root@BorderLeaf2> show ospf neighbor 
Address          Interface              State           ID               Pri  Dead
10.2.1.10        xe-0/0/1.0             Full            10.0.1.0         128    31
10.2.2.10        xe-0/0/2.0             Full            10.0.2.0         128    34
```
</details>

Проверим базу ospf 
<details>
<summary>Spine1</summary>

```text
root@Spine1> show ospf database 

    OSPF database, Area 0.0.0.0
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router  *10.0.1.0         10.0.1.0         0x80000020  1798  0x22 0x6e2d 180
Router   10.0.1.1         10.0.1.1         0x80000023   446  0x22 0xb486  84
Router   10.0.1.2         10.0.1.2         0x80000022   954  0x22 0x22f   84
Router   10.0.1.3         10.0.1.3         0x8000001f   947  0x22 0x53d5  84
Router   10.0.1.4         10.0.1.4         0x8000001f   954  0x22 0x9e7f  84
Router   10.0.1.5         10.0.1.5         0x80000022  2347  0x22 0xe32c  84
Router   10.0.1.6         10.0.1.6         0x8000001a  2374  0x22 0x3fcd  84
Router   10.0.2.0         10.0.2.0         0x8000002d   447  0x22 0x7b04 180
```
</details>

<details>
<summary>Spine2</summary>

```text
root@Spine2> show ospf database 

    OSPF database, Area 0.0.0.0
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router   10.0.1.0         10.0.1.0         0x80000020  2121  0x22 0x6e2d 180
Router   10.0.1.1         10.0.1.1         0x80000023   767  0x22 0xb486  84
Router   10.0.1.2         10.0.1.2         0x80000022  1275  0x22 0x22f   84
Router   10.0.1.3         10.0.1.3         0x8000001f  1268  0x22 0x53d5  84
Router   10.0.1.4         10.0.1.4         0x8000001f  1275  0x22 0x9e7f  84
Router   10.0.1.5         10.0.1.5         0x80000022  2668  0x22 0xe32c  84
Router   10.0.1.6         10.0.1.6         0x8000001a  2695  0x22 0x3fcd  84
Router  *10.0.2.0         10.0.2.0         0x8000002d   766  0x22 0x7b04 180
```
</details>

<details>
<summary>Leaf1</summary>

```text
root@Leaf1> show ospf database 

    OSPF database, Area 0.0.0.0
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router   10.0.1.0         10.0.1.0         0x80000020  2144  0x22 0x6e2d 180
Router  *10.0.1.1         10.0.1.1         0x80000023   790  0x22 0xb486  84
Router   10.0.1.2         10.0.1.2         0x80000022  1300  0x22 0x22f   84
Router   10.0.1.3         10.0.1.3         0x8000001f  1293  0x22 0x53d5  84
Router   10.0.1.4         10.0.1.4         0x8000001f  1300  0x22 0x9e7f  84
Router   10.0.1.5         10.0.1.5         0x80000022  2693  0x22 0xe32c  84
Router   10.0.1.6         10.0.1.6         0x8000001a  2720  0x22 0x3fcd  84
Router   10.0.2.0         10.0.2.0         0x8000002d   791  0x22 0x7b04 180
```
</details>

<details>
<summary>Leaf2</summary>

```text
root@Leaf2> show ospf database 

    OSPF database, Area 0.0.0.0
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router   10.0.1.0         10.0.1.0         0x80000020  2191  0x22 0x6e2d 180
Router   10.0.1.1         10.0.1.1         0x80000023   839  0x22 0xb486  84
Router  *10.0.1.2         10.0.1.2         0x80000022  1345  0x22 0x22f   84
Router   10.0.1.3         10.0.1.3         0x8000001f  1340  0x22 0x53d5  84
Router   10.0.1.4         10.0.1.4         0x8000001f  1347  0x22 0x9e7f  84
Router   10.0.1.5         10.0.1.5         0x80000022  2740  0x22 0xe32c  84
Router   10.0.1.6         10.0.1.6         0x8000001a  2767  0x22 0x3fcd  84
Router   10.0.2.0         10.0.2.0         0x8000002d   838  0x22 0x7b04 180
```
</details>

<details>
<summary>Leaf3</summary>

```text
root@Leaf3> show ospf database 

    OSPF database, Area 0.0.0.0
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router   10.0.1.0         10.0.1.0         0x80000020  2212  0x22 0x6e2d 180
Router   10.0.1.1         10.0.1.1         0x80000023   861  0x22 0xb486  84
Router   10.0.1.2         10.0.1.2         0x80000022  1369  0x22 0x22f   84
Router  *10.0.1.3         10.0.1.3         0x8000001f  1360  0x22 0x53d5  84
Router   10.0.1.4         10.0.1.4         0x8000001f  1369  0x22 0x9e7f  84
Router   10.0.1.5         10.0.1.5         0x80000022  2762  0x22 0xe32c  84
Router   10.0.1.6         10.0.1.6         0x8000001a  2789  0x22 0x3fcd  84
Router   10.0.2.0         10.0.2.0         0x8000002d   860  0x22 0x7b04 180
```
</details>

<details>
<summary>Leaf4</summary>

```text
root@Leaf4> show ospf database 

    OSPF database, Area 0.0.0.0
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router   10.0.1.0         10.0.1.0         0x80000020  2233  0x22 0x6e2d 180
Router   10.0.1.1         10.0.1.1         0x80000023   881  0x22 0xb486  84
Router   10.0.1.2         10.0.1.2         0x80000022  1390  0x22 0x22f   84
Router   10.0.1.3         10.0.1.3         0x8000001f  1382  0x22 0x53d5  84
Router  *10.0.1.4         10.0.1.4         0x8000001f  1388  0x22 0x9e7f  84
Router   10.0.1.5         10.0.1.5         0x80000022  2783  0x22 0xe32c  84
Router   10.0.1.6         10.0.1.6         0x8000001a  2810  0x22 0x3fcd  84
Router   10.0.2.0         10.0.2.0         0x8000002d   880  0x22 0x7b04 180
```
</details>

<details>
<summary>BorderLeaf1</summary>

```text
root@BorderLeaf1> show ospf database 

    OSPF database, Area 0.0.0.0
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router   10.0.1.0         10.0.1.0         0x80000020  2252  0x22 0x6e2d 180
Router   10.0.1.1         10.0.1.1         0x80000023   900  0x22 0xb486  84
Router   10.0.1.2         10.0.1.2         0x80000022  1408  0x22 0x22f   84
Router   10.0.1.3         10.0.1.3         0x8000001f  1401  0x22 0x53d5  84
Router   10.0.1.4         10.0.1.4         0x8000001f  1408  0x22 0x9e7f  84
Router  *10.0.1.5         10.0.1.5         0x80000022  2799  0x22 0xe32c  84
Router   10.0.1.6         10.0.1.6         0x8000001a  2828  0x22 0x3fcd  84
Router   10.0.2.0         10.0.2.0         0x8000002d   899  0x22 0x7b04 180
```
</details>

<details>
<summary>BorderLeaf2</summary>

```text
root@BorderLeaf2> show ospf database 

    OSPF database, Area 0.0.0.0
 Type       ID               Adv Rtr           Seq      Age  Opt  Cksum  Len 
Router   10.0.1.0         10.0.1.0         0x80000020  2268  0x22 0x6e2d 180
Router   10.0.1.1         10.0.1.1         0x80000023   916  0x22 0xb486  84
Router   10.0.1.2         10.0.1.2         0x80000022  1425  0x22 0x22f   84
Router   10.0.1.3         10.0.1.3         0x8000001f  1417  0x22 0x53d5  84
Router   10.0.1.4         10.0.1.4         0x8000001f  1425  0x22 0x9e7f  84
Router   10.0.1.5         10.0.1.5         0x80000022  2817  0x22 0xe32c  84
Router  *10.0.1.6         10.0.1.6         0x8000001a  2843  0x22 0x3fcd  84
Router   10.0.2.0         10.0.2.0         0x8000002d   915  0x22 0x7b04 180
```
</details>