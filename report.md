### University: [ITMO University](https://itmo.ru/ru/)

#### Faculty: [FICT](https://fict.itmo.ru)

#### Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing)

##### Year: 2022/2023

##### Group: K33202

##### Author: Konovalenko Maxim Pavlovich

##### Lab: Lab3

##### Date of create: 20.12.2022

##### Date of finished: 4.01.2023

***

# Отчёт по лабораторной работе №3 "Эмуляция распределенной корпоративной сети связи, настройка OSPF и MPLS, организация первого EoMPLS"

**Цель работы:** научиться использовать containerlab, получение практических навыков в настройке сети с VLAN'ами и методами работы с ними.

**Результаты работы:**

#### 1. Схема настраеваемой сети

![schema](./assets/images/schema.jpg)

#### 2. Файл для развертывания тестовой сети yaml

```
name: netlab3
mgmt:
  network: statics
  ipv4_subnet: 172.20.20.0/24
topology:
  nodes:
    R01.SPB:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.2
    R01.MSK:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.3
    R01.HKI:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.4
    R01.LND:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.5
    R01.LBN:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.6
    R01.NY:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.7
    PC1:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.8
    SGI_Prism:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.9
      
  links: 
    - endpoints: ["R01.SPB:eth2", "R01.MSK:eth2"]
    - endpoints: ["R01.SPB:eth1", "R01.HKI:eth1"]
    - endpoints: ["R01.MSK:eth3", "R01.LBN:eth3"]
    - endpoints: ["R01.HKI:eth3", "R01.LND:eth3"]
    - endpoints: ["R01.HKI:eth2", "R01.LBN:eth2"]
    - endpoints: ["R01.LND:eth2", "R01.NY:eth2"]
    - endpoints: ["R01.LBN:eth1", "R01.NY:eth1"]
    - endpoints: ["R01.SPB:eth3", "PC1:eth3"]
    - endpoints: ["R01.NY:eth3", "SGI_Prism:eth3"]
```

#### 3. Тексты конфигурация для каждого сетевого устройства

**R01.SPB**

```
# jan/04/2023 17:38:56 by RouterOS 6.47.9
# software id = 
#
#
#
/interface bridge
add name=EoMPLS
add name=Lo0
/interface vpls
add cisco-style=yes cisco-style-id=666 disabled=no l2mtu=1500 mac-address=\
    02:BF:EE:4D:EF:F1 name=EoMPLS_VPLS remote-peer=10.10.10.6
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] name=ospf0 router-id=10.10.10.1
/interface bridge port
add bridge=EoMPLS interface=ether4
add bridge=EoMPLS interface=EoMPLS_VPLS
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.1.1/30 interface=ether3 network=10.10.1.0
add address=10.10.2.1/30 interface=ether2 network=10.10.2.0
add address=192.168.10.1/24 interface=ether4 network=192.168.10.0
add address=10.10.10.1 interface=Lo0 network=10.10.10.1
/ip dhcp-client
add disabled=no interface=ether1
/ip route
add distance=1 dst-address=192.168.10.0/24 gateway=ether4
add distance=1 dst-address=192.168.20.0/24 gateway=10.10.10.6
/mpls ldp
set enabled=yes transport-address=10.10.10.1
/mpls ldp interface
add interface=ether2
add interface=ether3
/routing ospf network
add area=backbone
/system identity
set name=R01.SPB
```

**R01.MSK**

```
# jan/04/2023 17:39:22 by RouterOS 6.47.9
# software id = 
#
#
#
/interface bridge
add name=Lo0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] name=ospf0 router-id=10.10.10.2
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.1.2/30 interface=ether3 network=10.10.1.0
add address=10.10.3.1/30 interface=ether4 network=10.10.3.0
add address=10.10.10.2 interface=Lo0 network=10.10.10.2
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=10.10.10.2
/mpls ldp interface
add interface=ether3
add interface=ether4
/routing ospf network
add area=backbone
/system identity
set name=R01.MSK
```

**R01.HKI**

```
# jan/04/2023 17:39:48 by RouterOS 6.47.9
# software id = 
#
#
#
/interface bridge
add name=Lo0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] name=ospf0 router-id=10.10.10.3
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.2.2/30 interface=ether2 network=10.10.2.0
add address=10.10.4.1/30 interface=ether3 network=10.10.4.0
add address=10.10.5.1/30 interface=ether4 network=10.10.5.0
add address=10.10.10.3 interface=Lo0 network=10.10.10.3
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=10.10.10.3
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
/routing ospf network
add area=backbone
/system identity
set name=R01.HKI
```

**R01.LND**

```
# jan/04/2023 17:40:25 by RouterOS 6.47.9
# software id = 
#
#
#
/interface bridge
add name=Lo0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] name=ospf0 router-id=10.10.10.5
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.6.1/30 interface=ether3 network=10.10.6.0
add address=10.10.5.2/30 interface=ether4 network=10.10.5.0
add address=10.10.10.5 interface=Lo0 network=10.10.10.5
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=10.10.10.5
/mpls ldp interface
add interface=ether3
add interface=ether4
/routing ospf network
add area=backbone
/system identity
set name=R01.LND
```

**R01.LBN**

```
# jan/04/2023 17:40:54 by RouterOS 6.47.9
# software id = 
#
#
#
/interface bridge
add name=Lo0
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] name=ospf0 router-id=10.10.10.4
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.7.1/30 interface=ether2 network=10.10.7.0
add address=10.10.4.2/30 interface=ether3 network=10.10.4.0
add address=10.10.3.2/30 interface=ether4 network=10.10.3.0
add address=10.10.10.4 interface=Lo0 network=10.10.10.4
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=10.10.10.4
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
/routing ospf network
add area=backbone
/system identity
set name=R01.LBN
```

**R01.NY**

```
# jan/04/2023 17:40:32 by RouterOS 6.47.9
# software id = 
#
#
#
/interface bridge
add name=EoMPLS
add name=Lo0
/interface vpls
add cisco-style=yes cisco-style-id=666 disabled=no l2mtu=1500 mac-address=\
    02:25:B9:5B:6B:A3 name=EoMPLS_VPLS remote-peer=10.10.10.1
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] name=ospf0 router-id=10.10.10.6
/interface bridge port
add bridge=EoMPLS interface=ether4
add bridge=EoMPLS interface=EoMPLS_VPLS
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.10.7.2/30 interface=ether2 network=10.10.7.0
add address=10.10.6.2/30 interface=ether3 network=10.10.6.0
add address=192.168.20.1/24 interface=ether4 network=192.168.20.0
add address=10.10.10.6 interface=Lo0 network=10.10.10.6
/ip dhcp-client
add disabled=no interface=ether1
/ip route
add distance=1 dst-address=192.168.10.0/24 gateway=10.10.10.1
add distance=1 dst-address=192.168.20.0/24 gateway=ether4
/mpls ldp
set enabled=yes transport-address=10.10.10.6
/mpls ldp interface
add interface=ether2
add interface=ether3
/routing ospf network
add area=backbone
/system identity
set name=R01.NY
```

**PC1**

```
# jan/04/2023 17:41:44 by RouterOS 6.47.9
# software id = 
#
#
#
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=192.168.10.250/24 interface=ether4 network=192.168.10.0
/ip dhcp-client
add disabled=no interface=ether1
/ip route
add distance=1 gateway=192.168.10.1
/system identity
set name=PC1
```

**SGI_Prism**

```
# jan/04/2023 17:41:14 by RouterOS 6.47.9
# software id = 
#
#
#
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=192.168.20.250/24 interface=ether4 network=192.168.20.0
/ip dhcp-client
add disabled=no interface=ether1
/ip route
add distance=1 gateway=192.168.20.1
/system identity
set name=SGI_Prism
```
#### 4. Выкладки с маршрутами на каждом устройстве.

![](./assets/images/routes1.jpg)
![](./assets/images/routes2.jpg)
![](./assets/images/routes3.jpg)
![](./assets/images/routes4.jpg)
![](./assets/images/routes5.jpg)
![](./assets/images/routes6.jpg)

#### 5. Результаты пингов

![](./assets/images/ping1.jpg)
![](./assets/images/ping2.jpg)
![](./assets/images/ping3.jpg)
![](./assets/images/ping4.jpg)

### Вывод

В ходе лабораторной работы №3 были изучены протоколы OSPF и MPLS, а также механизмы организации EoMPLS.