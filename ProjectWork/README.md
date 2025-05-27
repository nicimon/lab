# Проектная работа.
# "Настройка и тестирование EVPN/VXLAN для обеспечения бесшовной миграции виртуальных машин на оборудовании Juniper"

### Цели:
* Реализация сетевой фабрики с использованием Overlay технологии VxLAN
* Сети Overlay и Underlay используется eBGP

### Описание/Пошаговая инструкция выполнения проектной работы:
Построения сетевой фабрики на основе протокола VxLAN:
#### 1) Необходимо настроить подключение нод Proxmox сразу к двум коммутаторам с использованием технологии ESI;
#### 2) Настроить маршрутизацию между сетями;
#### 3) Зафиксировать в документации - план работы, адресное пространство, схему сети, настройки сетевого оборудования.

# Выполнение:

## План выполнения проектной работы:

1) Зафиксировать параметры сети;
2) Привести схему сети;
3) Опубликовать список реализованных функций;
4) Провести проверку работы сети;
7) Опубликовать листинг команд для проверки корректной работы сети;
8) Привести конфигурации устройств.

## Параметры сети.

IP адреса Loopack интерфейсов:
Loopack-s:
<div align="center">

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
</div>

IP адреса P-t-P сетей

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
| Spine 2 → BorderLeaf 2 | 10.2.2.10        | 10.2.2.11          | 10.2.1.10/31 |

Таблица номеров автономных систем (AS)
<div align="center">

| Switch      |     AS     |
|-------------|------------|
| Spine1      | 4200000001 |
| Spine2      | 4200000001 |
| Leaf1       | 4200000011 |
| Leaf2       | 4200000012 |
| Leaf3       | 4200000013 |
| Leaf4       | 4200000014 |
| BorderLeaf1 | 4200000015 |
| BorderLeaf2 | 4200000016 |
</div>

#### Прочее.
В сзязи с реализацией схемы сети в виртуальной среде, необходимо в реальной сети настроить протокол BFD и выставить таймеры в протоколе BGP

### Схема связи

![Proj-1.PNG](screenshots/Proj-1.PNG)
#### Описание сети 
Необходимо реализовать схему связи предоставленную на рисунке 1. 

Cеть состоит из:

* трех Spine (Spine3 нужен для лучшей резервирования сети);
* семи Leaf из них два BorderLeaf;
* двух серверов СХД.

В процессе настройки сети в виртуальной среде потребовалось внести корректировки в исходную схему, которая предоставлена на рисунке 2 
![Proj-2.PNG](screenshots/Proj-2.PNG)

Для проверки работоспособности сети был настроена одна сеть с параментами 
| Назначение | Сервисная модель | VLAN ID/VRF name | VNI | RD | RT |
| ------------ |:---------------:|:---------------:|:---------------:|:---------------:|:---------------:|
| MAC-VRF 1 | VLAN-Aware-Bundle | 100 | 10100 | (Router ID):100 | 3:3 |
| IP-VRF 1 | Symmetric IRB | Tenant1 | 10500 | (Router ID):500 | 500:500 |

### Параметры ESI для серверов:

#### Агрегация между Leaf2, Leaf3 и proxmoxnode3.
| Интерфейс | ESI ID | lacp system-id | Site |
|:---------------:|:---------------:|:---------------:|:---------------:|
| ae0 | 00:12:12:12:12:12:12:12:12:12 | 12:12:12:12:12:12 | 1 |

### Параметры на proxmoxnode3:
| Назначение | IP-шлюза | MAC | VLAN ID | VRF name | Site |
| ------------ |:---------------:|:---------------:|:---------------:|:---------------:|:---------------:|
| Bond0 | 172.20.31.120/24 | 6a:26:9a:42:60:e1 | 100 | Tenant1 | 1 |

## Список реализованных функций:

* Протокол маршрутизации для Overlay/Underlay-сети - eBGP;
* Протоколы Overlay-сети - VXLAN и EVPN;
* Проверена L2-связность L3-связность между нодами proxmox;
* Схема подключения нод proxmox - Multi-Homing (ESI);
* Трафик внутри и меджу сайтами балансируется по ECMP.

## Проверка работы сети:

#### Связь между серверами proxmoxnode1 и proxmoxnode2
![Proj-4.PNG](screenshots/Proj-4.PNG)

#### Связь между серверами proxmoxnode1 и proxmoxnode3
![Proj-5.PNG](screenshots/Proj-5.PNG)

## Листинг команд с примерами вывода:
#### Листинг:
```
show evpn database
show mac-vrf routing database
show evpn instance extensive
show route  
```




show evpn database
<details>
<summary>Leaf2</summary>

``` text
root@Leaf2> show evpn database 
Instance: macvrf-1
VLAN  DomainId  MAC address        Active source                  Timestamp        IP address
     10100      00:00:5e:00:01:01  05:fa:56:ea:10:00:00:27:74:00  May 27 11:05:23  172.20.31.1
     10100      00:50:56:80:26:6c  10.0.1.4                       May 27 11:05:23  172.20.31.68
     10100      00:50:56:80:b0:e9  10.0.1.1                       May 27 11:05:23  172.20.31.98
     10100      00:50:56:80:bb:b7  10.0.1.1                       May 27 11:05:23  172.20.31.75
     10100      00:50:56:80:f7:71  10.0.1.1                       May 27 11:13:51  172.20.31.81
     10100      50:00:00:5e:00:01  ge-0/0/4.0                     May 27 11:38:55  172.20.31.80
     10100      6a:26:9a:42:60:e1  00:12:12:12:12:12:12:12:12:12  May 27 14:54:31  172.20.31.102
   ```
</details>

<details>
<summary>Leaf3</summary>

``` text
root@Leaf3> show evpn database 
Instance: macvrf-1
VLAN  DomainId  MAC address        Active source                  Timestamp        IP address
     10100      00:00:5e:00:01:01  05:fa:56:ea:10:00:00:27:74:00  May 27 12:08:00  172.20.31.1
     10100      00:50:56:80:26:6c  10.0.1.4                       May 27 12:08:00  172.20.31.68
     10100      00:50:56:80:b0:e9  10.0.1.1                       May 27 12:08:00  172.20.31.98
     10100      00:50:56:80:bb:b7  10.0.1.1                       May 27 12:08:00  172.20.31.75
     10100      00:50:56:80:f7:71  10.0.1.1                       May 27 12:08:00  172.20.31.81
     10100      50:00:00:5e:00:01  10.0.1.2                       May 27 12:08:00  172.20.31.80
     10100      6a:26:9a:42:60:e1  00:12:12:12:12:12:12:12:12:12  May 27 14:54:31  172.20.31.102
```
</details>
show mac-vrf routing database
<details>
<summary>Leaf2</summary>

``` text
root@Leaf2> show mac-vrf routing database 
Instance: macvrf-1
VLAN  DomainId  MAC address        Active source                  Timestamp        IP address
     10100      00:00:5e:00:01:01  05:fa:56:ea:10:00:00:27:74:00  May 27 11:05:23  172.20.31.1
     10100      00:50:56:80:26:6c  10.0.1.4                       May 27 11:05:23  172.20.31.68
     10100      00:50:56:80:b0:e9  10.0.1.1                       May 27 11:05:23  172.20.31.98
     10100      00:50:56:80:bb:b7  10.0.1.1                       May 27 11:05:23  172.20.31.75
     10100      00:50:56:80:f7:71  10.0.1.1                       May 27 15:29:34  172.20.31.81
     10100      50:00:00:5e:00:01  ge-0/0/4.0                     May 27 11:38:55  172.20.31.80
     10100      6a:26:9a:42:60:e1  00:12:12:12:12:12:12:12:12:12  May 27 14:54:31  172.20.31.102
   ```
</details>
<details>
<summary>Leaf3</summary>

``` text

root@Leaf3> show mac-vrf routing database    
Instance: macvrf-1
VLAN  DomainId  MAC address        Active source                  Timestamp        IP address
     10100      00:00:5e:00:01:01  05:fa:56:ea:10:00:00:27:74:00  May 27 12:08:00  172.20.31.1
     10100      00:50:56:80:26:6c  10.0.1.4                       May 27 12:08:00  172.20.31.68
     10100      00:50:56:80:b0:e9  10.0.1.1                       May 27 12:08:00  172.20.31.98
     10100      00:50:56:80:bb:b7  10.0.1.1                       May 27 12:08:00  172.20.31.75
     10100      00:50:56:80:f7:71  10.0.1.1                       May 27 15:29:34  172.20.31.81
     10100      50:00:00:5e:00:01  10.0.1.2                       May 27 12:08:00  172.20.31.80
     10100      6a:26:9a:42:60:e1  00:12:12:12:12:12:12:12:12:12  May 27 14:54:31  172.20.31.102
```
</details>

show evpn instance extensive

<details>
<summary>Leaf2</summary>

``` text

root@Leaf2> show evpn instance extensive 
Instance: __default_evpn__
  Route Distinguisher: 10.0.1.2:0
  Number of bridge domains: 0
  Number of neighbors: 1
    Address               MAC    MAC+IP        AD        IM        ES Leaf-label Remote-DCI-Peer Flow-label
    10.0.1.3                0         0         0         0         1                            NO  

Instance: macvrf-1
  Route Distinguisher: 10.0.1.2:100
  Encapsulation type: VXLAN
  Control word enabled
  Duplicate MAC detection threshold: 5
  Duplicate MAC detection window: 180
  MAC database status                     Local  Remote
    MAC advertisements:                       2       6
    MAC+IP advertisements:                    2       6
    Default gateway MAC advertisements:       0       0
  Number of local interfaces: 3 (3 up)
    Interface name  ESI                            Mode             Status     AC-Role
    .local..9       00:00:00:00:00:00:00:00:00:00  single-homed     Up         Root 
    ae0.100         00:12:12:12:12:12:12:12:12:12  all-active       Up         Root 
    ge-0/0/4.0      00:00:00:00:00:00:00:00:00:00  single-homed     Up         Root 
  Number of IRB interfaces: 1 (0 up)
    Interface name  VLAN   VNI    Status  L3 context
    irb.100                10100   Down   Not assigned                     
  Number of protect interfaces: 0
  Number of bridge domains: 1
    VLAN  Domain-ID Intfs/up   IRB-intf  Mode            MAC-sync v4-SG-sync v6-SG-sync
    100   10100        2  2    irb.100   Extended        Enabled  Disabled   Disabled  
  Number of neighbors: 4
    Address               MAC    MAC+IP        AD        IM        ES Leaf-label Remote-DCI-Peer Flow-label
    10.0.1.1                3         3         0         1         0                            NO  
    10.0.1.3                1         1         2         1         0                            NO  
    10.0.1.4                1         1         0         1         0                            NO  
    10.0.1.6                1         1         1         1         0                            NO  
  Number of ethernet segments: 3
    ESI: 00:12:12:12:12:12:12:12:12:12
      Status: Resolved by IFL ae0.100
      Local interface: ae0.100, Status: Up/Forwarding
      Number of remote PEs connected: 1
        Remote-PE        MAC-label  Aliasing-label  Mode
        10.0.1.3         10100      0               all-active   
      DF Election Algorithm: MOD based
      Designated forwarder: 10.0.1.2
      Backup forwarder: 10.0.1.3
      Last designated forwarder update: May 27 12:08:24
    ESI: 05:fa:56:ea:0c:00:00:27:74:00
      Status: Resolved
      Local interface: irb.100, Status: Down
    ESI: 05:fa:56:ea:10:00:00:27:74:00
      Status: Resolved
      Number of remote PEs connected: 1
        Remote-PE        MAC-label  Aliasing-label  Mode
        10.0.1.6         10100      0               all-active   
  Router-ID: 10.0.1.2                   
  Source VTEP interface IP: 10.0.1.2
  SMET Forwarding: Disabled

   ```
</details>

<details>
<summary>Leaf3</summary>

``` text
root@Leaf3> show evpn instance extensive 
Instance: __default_evpn__
  Route Distinguisher: 10.0.1.3:0
  Number of bridge domains: 0
  Number of neighbors: 1
    Address               MAC    MAC+IP        AD        IM        ES Leaf-label Remote-DCI-Peer Flow-label
    10.0.1.2                0         0         0         0         1                            NO  

Instance: macvrf-1
  Route Distinguisher: 10.0.1.3:100
  Encapsulation type: VXLAN
  Control word enabled
  Duplicate MAC detection threshold: 5
  Duplicate MAC detection window: 180
  MAC database status                     Local  Remote
    MAC advertisements:                       1       7
    MAC+IP advertisements:                    1       7
    Default gateway MAC advertisements:       0       0
  Number of local interfaces: 2 (2 up)
    Interface name  ESI                            Mode             Status     AC-Role
    .local..9       00:00:00:00:00:00:00:00:00:00  single-homed     Up         Root 
    ae0.100         00:12:12:12:12:12:12:12:12:12  all-active       Up         Root 
  Number of IRB interfaces: 1 (0 up)
    Interface name  VLAN   VNI    Status  L3 context
    irb.100                10100   Down   Not assigned                     
  Number of protect interfaces: 0
  Number of bridge domains: 1
    VLAN  Domain-ID Intfs/up   IRB-intf  Mode            MAC-sync v4-SG-sync v6-SG-sync
    100   10100        1  1    irb.100   Extended        Enabled  Disabled   Disabled  
  Number of neighbors: 4
    Address               MAC    MAC+IP        AD        IM        ES Leaf-label Remote-DCI-Peer Flow-label
    10.0.1.1                3         3         0         1         0                            NO  
    10.0.1.2                2         2         2         1         0                            NO  
    10.0.1.4                1         1         0         1         0                            NO  
    10.0.1.6                1         1         1         1         0                            NO  
  Number of ethernet segments: 3
    ESI: 00:12:12:12:12:12:12:12:12:12
      Status: Resolved by IFL ae0.100
      Local interface: ae0.100, Status: Up/Forwarding
      Number of remote PEs connected: 1
        Remote-PE        MAC-label  Aliasing-label  Mode
        10.0.1.2         10100      0               all-active   
      DF Election Algorithm: MOD based
      Designated forwarder: 10.0.1.2
      Backup forwarder: 10.0.1.3
      Last designated forwarder update: May 27 12:08:23
    ESI: 05:fa:56:ea:0d:00:00:27:74:00
      Status: Resolved
      Local interface: irb.100, Status: Down
    ESI: 05:fa:56:ea:10:00:00:27:74:00
      Status: Resolved
      Number of remote PEs connected: 1
        Remote-PE        MAC-label  Aliasing-label  Mode
        10.0.1.6         10100      0               all-active   
  Router-ID: 10.0.1.3
  Source VTEP interface IP: 10.0.1.3
  SMET Forwarding: Disabled

   ```

</details>

show route

<details>

<summary>Leaf2</summary>

root@Leaf2> show route  

inet.0: 11 destinations, 15 routes (11 active, 0 holddown, 0 hidden)

* = Active Route, - = Last Active, * = Both

10.0.1.0/32        *[BGP/170] 04:39:14, localpref 100
                      AS path: 4200000001 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
10.0.1.1/32        *[BGP/170] 04:36:15, localpref 100, from 10.2.1.2
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                       to 10.2.1.2 via ge-0/0/1.0
                    >  to 10.2.2.2 via ge-0/0/2.0
                    [BGP/170] 04:36:15, localpref 100
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                    >  to 10.2.2.2 via ge-0/0/2.0
10.0.1.2/32        *[Direct/0] 6d 02:11:49
                    >  via lo0.0
10.0.1.3/32        *[BGP/170] 03:37:47, localpref 100, from 10.2.1.2
                      AS path: 4200000001 4200000013 I, validation-state: unverified
                       to 10.2.1.2 via ge-0/0/1.0
                    >  to 10.2.2.2 via ge-0/0/2.0
                    [BGP/170] 03:37:47, localpref 100
                      AS path: 4200000001 4200000013 I, validation-state: unverified
                    >  to 10.2.2.2 via ge-0/0/2.0
10.0.1.4/32        *[BGP/170] 04:36:15, localpref 100, from 10.2.1.2
                      AS path: 4200000001 4200000014 I, validation-state: unverified
                       to 10.2.1.2 via ge-0/0/1.0
                    >  to 10.2.2.2 via ge-0/0/2.0
                    [BGP/170] 04:36:15, localpref 100
                      AS path: 4200000001 4200000014 I, validation-state: unverified
                    >  to 10.2.2.2 via ge-0/0/2.0
10.0.1.6/32        *[BGP/170] 04:36:15, localpref 100, from 10.2.1.2
                      AS path: 4200000001 4200000016 I, validation-state: unverified
                       to 10.2.1.2 via ge-0/0/1.0
                    >  to 10.2.2.2 via ge-0/0/2.0
                    [BGP/170] 04:36:15, localpref 100
                      AS path: 4200000001 4200000016 I, validation-state: unverified
                    >  to 10.2.2.2 via ge-0/0/2.0
10.0.2.0/32        *[BGP/170] 04:36:15, localpref 100
                      AS path: 4200000001 I, validation-state: unverified
                    >  to 10.2.2.2 via ge-0/0/2.0
10.2.1.2/31        *[Direct/0] 04:40:24
                    >  via ge-0/0/1.0
10.2.1.3/32        *[Local/0] 04:40:24
                       Local via ge-0/0/1.0
10.2.2.2/31        *[Direct/0] 04:40:24
                    >  via ge-0/0/2.0
10.2.2.3/32        *[Local/0] 04:40:24
                       Local via ge-0/0/2.0

Tenant1.inet.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

172.20.31.0/24     *[EVPN/170] 04:40:22
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0

inet6.0: 2 destinations, 2 routes (2 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

fe80::52a9:3ff:fe00:6400/128
                   *[Local/0] 6d 02:55:07
                       Reject           
ff02::2/128        *[INET6/0] 6d 02:55:08
                       MultiRecv

Tenant1.inet6.0: 1 destinations, 1 routes (1 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

ff02::2/128        *[INET6/0] 6d 02:11:49
                       MultiRecv

bgp.evpn.0: 29 destinations, 50 routes (29 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

1:10.0.1.2:0::121212121212121212::FFFF:FFFF/192 AD/ESI        
                   *[EVPN/170] 04:06:42
                       Indirect
1:10.0.1.2:100::121212121212121212::0/192 AD/EVI        
                   *[EVPN/170] 04:06:43
                       Indirect
1:10.0.1.3:0::121212121212121212::FFFF:FFFF/192 AD/ESI        
                   *[BGP/170] 03:37:20, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000013 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
                    [BGP/170] 03:37:20, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000013 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
1:10.0.1.3:100::121212121212121212::0/192 AD/EVI        
                   *[BGP/170] 03:37:21, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000013 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
                    [BGP/170] 03:37:20, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000013 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
1:10.0.1.6:0::05fa56ea100000277400::FFFF:FFFF/192 AD/ESI        
                   *[BGP/170] 04:40:22, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000016 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
                    [BGP/170] 04:40:22, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000016 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
2:10.0.1.1:100::10100::00:50:56:80:b0:e9/304 MAC/IP        
                   *[BGP/170] 04:40:22, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
                    [BGP/170] 04:40:22, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
2:10.0.1.1:100::10100::00:50:56:80:bb:b7/304 MAC/IP        
                   *[BGP/170] 04:40:22, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
                    [BGP/170] 04:40:22, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
2:10.0.1.1:100::10100::00:50:56:80:f7:71/304 MAC/IP        
                   *[BGP/170] 00:03:21, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                       to 10.2.1.2 via ge-0/0/1.0, Push 631
                    >  to 10.2.2.2 via ge-0/0/2.0, Push 631
                    [BGP/170] 00:03:21, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                       to 10.2.1.2 via ge-0/0/1.0, Push 631
                    >  to 10.2.2.2 via ge-0/0/2.0, Push 631
2:10.0.1.2:100::10100::50:00:00:5e:00:01/304 MAC/IP        
                   *[EVPN/170] 04:17:55
                       Indirect
2:10.0.1.2:100::10100::6a:26:9a:42:60:e1/304 MAC/IP        
                   *[EVPN/170] 03:23:33
                       Indirect
2:10.0.1.3:100::10100::6a:26:9a:42:60:e1/304 MAC/IP        
                   *[BGP/170] 03:37:21, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000013 I, validation-state: unverified
                       to 10.2.1.2 via ge-0/0/1.0, Push 631
                    >  to 10.2.2.2 via ge-0/0/2.0, Push 631
                    [BGP/170] 03:37:20, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000013 I, validation-state: unverified
                       to 10.2.1.2 via ge-0/0/1.0, Push 631
                    >  to 10.2.2.2 via ge-0/0/2.0, Push 631
2:10.0.1.4:100::10100::00:50:56:80:26:6c/304 MAC/IP        
                   *[BGP/170] 04:40:22, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000014 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
                    [BGP/170] 04:40:22, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000014 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
2:10.0.1.6:100::10100::00:00:5e:00:01:01/304 MAC/IP        
                   *[BGP/170] 04:40:22, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000016 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
                    [BGP/170] 04:40:22, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000016 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
2:10.0.1.1:100::10100::00:50:56:80:b0:e9::172.20.31.98/304 MAC/IP        
                   *[BGP/170] 04:40:22, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                       to 10.2.1.2 via ge-0/0/1.0, Push 631
                    >  to 10.2.2.2 via ge-0/0/2.0, Push 631
                    [BGP/170] 04:40:22, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                       to 10.2.1.2 via ge-0/0/1.0, Push 631
                    >  to 10.2.2.2 via ge-0/0/2.0, Push 631
2:10.0.1.1:100::10100::00:50:56:80:bb:b7::172.20.31.75/304 MAC/IP        
                   *[BGP/170] 04:40:22, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
                    [BGP/170] 04:40:22, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
2:10.0.1.1:100::10100::00:50:56:80:f7:71::172.20.31.81/304 MAC/IP        
                   *[BGP/170] 00:03:21, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
                    [BGP/170] 00:03:21, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
2:10.0.1.2:100::10100::50:00:00:5e:00:01::172.20.31.80/304 MAC/IP        
                   *[EVPN/170] 04:17:47
                       Indirect
2:10.0.1.2:100::10100::6a:26:9a:42:60:e1::172.20.31.102/304 MAC/IP        
                   *[EVPN/170] 02:54:31
                       Indirect
2:10.0.1.3:100::10100::6a:26:9a:42:60:e1::172.20.31.102/304 MAC/IP        
                   *[BGP/170] 00:51:13, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000013 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
                    [BGP/170] 00:51:13, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000013 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
2:10.0.1.4:100::10100::00:50:56:80:26:6c::172.20.31.68/304 MAC/IP        
                   *[BGP/170] 04:40:22, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000014 I, validation-state: unverified
                       to 10.2.1.2 via ge-0/0/1.0, Push 631
                    >  to 10.2.2.2 via ge-0/0/2.0, Push 631
                    [BGP/170] 04:40:22, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000014 I, validation-state: unverified
                       to 10.2.1.2 via ge-0/0/1.0, Push 631
                    >  to 10.2.2.2 via ge-0/0/2.0, Push 631
2:10.0.1.6:100::10100::00:00:5e:00:01:01::172.20.31.1/304 MAC/IP        
                   *[BGP/170] 04:40:22, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000016 I, validation-state: unverified
                       to 10.2.1.2 via ge-0/0/1.0, Push 631
                    >  to 10.2.2.2 via ge-0/0/2.0, Push 631
                    [BGP/170] 04:40:22, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000016 I, validation-state: unverified
                       to 10.2.1.2 via ge-0/0/1.0, Push 631
                    >  to 10.2.2.2 via ge-0/0/2.0, Push 631
3:10.0.1.1:100::10100::10.0.1.1/248 IM            
                   *[BGP/170] 04:40:22, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
                    [BGP/170] 04:40:22, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
3:10.0.1.2:100::10100::10.0.1.2/248 IM            
                   *[EVPN/170] 6d 02:11:48
                       Indirect
3:10.0.1.3:100::10100::10.0.1.3/248 IM            
                   *[BGP/170] 03:37:44, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000013 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
                    [BGP/170] 03:37:44, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000013 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
3:10.0.1.4:100::10100::10.0.1.4/248 IM            
                   *[BGP/170] 04:40:22, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000014 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
                    [BGP/170] 04:40:22, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000014 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
3:10.0.1.6:100::10100::10.0.1.6/248 IM            
                   *[BGP/170] 04:40:22, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000016 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
                    [BGP/170] 04:40:22, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000016 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
4:10.0.1.2:0::121212121212121212:10.0.1.2/296 ES            
                   *[EVPN/170] 04:06:43
                       Indirect
4:10.0.1.3:0::121212121212121212:10.0.1.3/296 ES            
                   *[BGP/170] 03:37:21, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000013 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
                    [BGP/170] 03:37:20, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000013 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
5:10.0.1.6:500::0::172.20.31.0::24/248               
                   *[BGP/170] 04:40:22, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000016 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 656
                       to 10.2.2.2 via ge-0/0/2.0, Push 656
                    [BGP/170] 04:40:22, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000016 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 656
                       to 10.2.2.2 via ge-0/0/2.0, Push 656

Tenant1.evpn.0: 1 destinations, 2 routes (1 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

5:10.0.1.6:500::0::172.20.31.0::24/248               
                   *[BGP/170] 04:40:22, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000016 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 656
                       to 10.2.2.2 via ge-0/0/2.0, Push 656
                    [BGP/170] 04:40:22, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000016 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 656
                       to 10.2.2.2 via ge-0/0/2.0, Push 656

macvrf-1.evpn.0: 25 destinations, 44 routes (25 active, 0 holddown, 0 hidden)
+ = Active Route, - = Last Active, * = Both

1:10.0.1.2:100::121212121212121212::0/192 AD/EVI        
                   *[EVPN/170] 04:06:43
                       Indirect
1:10.0.1.3:0::121212121212121212::FFFF:FFFF/192 AD/ESI        
                   *[BGP/170] 03:37:20, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000013 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
                    [BGP/170] 03:37:20, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000013 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
1:10.0.1.3:100::121212121212121212::0/192 AD/EVI        
                   *[BGP/170] 03:37:21, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000013 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
                    [BGP/170] 03:37:20, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000013 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
1:10.0.1.6:0::05fa56ea100000277400::FFFF:FFFF/192 AD/ESI        
                   *[BGP/170] 04:40:22, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000016 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
                    [BGP/170] 04:40:22, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000016 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
2:10.0.1.1:100::10100::00:50:56:80:b0:e9/304 MAC/IP        
                   *[BGP/170] 04:40:22, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
                    [BGP/170] 04:40:22, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
2:10.0.1.1:100::10100::00:50:56:80:bb:b7/304 MAC/IP        
                   *[BGP/170] 04:40:22, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
                    [BGP/170] 04:40:22, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
2:10.0.1.1:100::10100::00:50:56:80:f7:71/304 MAC/IP        
                   *[BGP/170] 00:03:21, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                       to 10.2.1.2 via ge-0/0/1.0, Push 631
                    >  to 10.2.2.2 via ge-0/0/2.0, Push 631
                    [BGP/170] 00:03:21, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                       to 10.2.1.2 via ge-0/0/1.0, Push 631
                    >  to 10.2.2.2 via ge-0/0/2.0, Push 631
2:10.0.1.2:100::10100::50:00:00:5e:00:01/304 MAC/IP        
                   *[EVPN/170] 04:17:55
                       Indirect
2:10.0.1.2:100::10100::6a:26:9a:42:60:e1/304 MAC/IP        
                   *[EVPN/170] 03:23:33
                       Indirect
2:10.0.1.3:100::10100::6a:26:9a:42:60:e1/304 MAC/IP        
                   *[BGP/170] 03:37:21, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000013 I, validation-state: unverified
                       to 10.2.1.2 via ge-0/0/1.0, Push 631
                    >  to 10.2.2.2 via ge-0/0/2.0, Push 631
                    [BGP/170] 03:37:20, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000013 I, validation-state: unverified
                       to 10.2.1.2 via ge-0/0/1.0, Push 631
                    >  to 10.2.2.2 via ge-0/0/2.0, Push 631
2:10.0.1.4:100::10100::00:50:56:80:26:6c/304 MAC/IP        
                   *[BGP/170] 04:40:22, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000014 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
                    [BGP/170] 04:40:22, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000014 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
2:10.0.1.6:100::10100::00:00:5e:00:01:01/304 MAC/IP        
                   *[BGP/170] 04:40:22, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000016 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
                    [BGP/170] 04:40:22, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000016 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
2:10.0.1.1:100::10100::00:50:56:80:b0:e9::172.20.31.98/304 MAC/IP        
                   *[BGP/170] 04:40:22, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                       to 10.2.1.2 via ge-0/0/1.0, Push 631
                    >  to 10.2.2.2 via ge-0/0/2.0, Push 631
                    [BGP/170] 04:40:22, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                       to 10.2.1.2 via ge-0/0/1.0, Push 631
                    >  to 10.2.2.2 via ge-0/0/2.0, Push 631
2:10.0.1.1:100::10100::00:50:56:80:bb:b7::172.20.31.75/304 MAC/IP        
                   *[BGP/170] 04:40:22, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
                    [BGP/170] 04:40:22, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
2:10.0.1.1:100::10100::00:50:56:80:f7:71::172.20.31.81/304 MAC/IP        
                   *[BGP/170] 00:03:21, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
                    [BGP/170] 00:03:21, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
2:10.0.1.2:100::10100::50:00:00:5e:00:01::172.20.31.80/304 MAC/IP        
                   *[EVPN/170] 04:17:47
                       Indirect
2:10.0.1.2:100::10100::6a:26:9a:42:60:e1::172.20.31.102/304 MAC/IP        
                   *[EVPN/170] 02:54:31
                       Indirect
2:10.0.1.3:100::10100::6a:26:9a:42:60:e1::172.20.31.102/304 MAC/IP        
                   *[BGP/170] 00:51:13, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000013 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
                    [BGP/170] 00:51:13, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000013 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0, Push 631
                       to 10.2.2.2 via ge-0/0/2.0, Push 631
2:10.0.1.4:100::10100::00:50:56:80:26:6c::172.20.31.68/304 MAC/IP        
                   *[BGP/170] 04:40:22, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000014 I, validation-state: unverified
                       to 10.2.1.2 via ge-0/0/1.0, Push 631
                    >  to 10.2.2.2 via ge-0/0/2.0, Push 631
                    [BGP/170] 04:40:22, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000014 I, validation-state: unverified
                       to 10.2.1.2 via ge-0/0/1.0, Push 631
                    >  to 10.2.2.2 via ge-0/0/2.0, Push 631
2:10.0.1.6:100::10100::00:00:5e:00:01:01::172.20.31.1/304 MAC/IP        
                   *[BGP/170] 04:40:22, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000016 I, validation-state: unverified
                       to 10.2.1.2 via ge-0/0/1.0, Push 631
                    >  to 10.2.2.2 via ge-0/0/2.0, Push 631
                    [BGP/170] 04:40:22, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000016 I, validation-state: unverified
                       to 10.2.1.2 via ge-0/0/1.0, Push 631
                    >  to 10.2.2.2 via ge-0/0/2.0, Push 631
3:10.0.1.1:100::10100::10.0.1.1/248 IM            
                   *[BGP/170] 04:40:22, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
                    [BGP/170] 04:40:22, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000011 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
3:10.0.1.2:100::10100::10.0.1.2/248 IM            
                   *[EVPN/170] 6d 02:11:48
                       Indirect
3:10.0.1.3:100::10100::10.0.1.3/248 IM            
                   *[BGP/170] 03:37:44, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000013 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
                    [BGP/170] 03:37:44, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000013 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
3:10.0.1.4:100::10100::10.0.1.4/248 IM            
                   *[BGP/170] 04:40:22, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000014 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
                    [BGP/170] 04:40:22, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000014 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
3:10.0.1.6:100::10100::10.0.1.6/248 IM            
                   *[BGP/170] 04:40:22, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000016 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
                    [BGP/170] 04:40:22, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000016 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0

  __default_evpn__.evpn.0: 3 destinations, 4 routes (3 active, 0 holddown, 0 hidden)
  + = Active Route, - = Last Active, * = Both

1:10.0.1.2:0::121212121212121212::FFFF:FFFF/192 AD/ESI        
                   *[EVPN/170] 04:06:42
                       Indirect
4:10.0.1.2:0::121212121212121212:10.0.1.2/296 ES            
                   *[EVPN/170] 04:06:43
                       Indirect
4:10.0.1.3:0::121212121212121212:10.0.1.3/296 ES            
                   *[BGP/170] 03:37:21, localpref 100, from 10.0.2.0
                      AS path: 4200000001 4200000013 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
                    [BGP/170] 03:37:20, localpref 100, from 10.0.1.0
                      AS path: 4200000001 4200000013 I, validation-state: unverified
                    >  to 10.2.1.2 via ge-0/0/1.0
                       to 10.2.2.2 via ge-0/0/2.0
    ```

</details>

![Proj-3.PNG](screenshots/Proj-3.PNG)
