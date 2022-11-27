### University: [ITMO University](https://itmo.ru/ru/)

#### Faculty: [FICT](https://fict.itmo.ru)

#### Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing)

##### Year: 2022/2023

##### Group: K33202

##### Author: Konovalenko Maxim Pavlovich

##### Lab: Lab1

##### Date of create: 27.11.2022

##### Date of finished: 27.11.2022

***

# Отчёт по лабораторной работе №2 "Эмуляция распределенной корпоративной сети связи, настройка статической маршрутизации между филиалами"

**Цель работы:** ознакомиться с принципами планирования IP адресов, настройке статической маршрутизации и сетевыми функциями устройств.

**Результаты работы:**

#### 1. Схема настраеваемой сети

![schema](./assets/images/lab2Drawio.png)

#### 2. Файл для развертывания тестовой сети yaml

```
name: lab2
mgmt:
  network: statics
  ipv4_subnet: 172.20.20.0/24
topology:
  nodes:
    R01.MSK:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.22
    R01.BRL:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.23
    R01.FRT:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.24
    PC1:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.25
    PC2:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.26
    PC3:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.20.20.27
  links: 
    - endpoints: ["R01.MSK:eth2", "R01.FRT:eth2"]
    - endpoints: ["R01.MSK:eth1", "R01.BRL:eth1"]
    - endpoints: ["R01.BRL:eth2", "R01.FRT:eth1"]
    - endpoints: ["R01.MSK:eth3", "PC1:eth3"]
    - endpoints: ["R01.FRT:eth3", "PC2:eth3"]
    - endpoints: ["R01.BRL:eth3", "PC3:eth3"]
```

#### 3. Тексты конфигурация для каждого сетевого устройства

**R01.MSK**

```
/interface wireless security-profiles 
set [ find default=yes ] supplicant-identity=MikroTik 
/ip pool 
add name=pool1 ranges=192.168.10.10-192.168.10.254 
/ip dhcp-server 
add address-pool=pool1 disabled=no interface=ether4 name=dhcp1 
/ip address 
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28 
add address=10.10.1.1/30 interface=ether2 network=10.10.1.0 
add address=10.10.2.1/30 interface=ether3 network=10.10.2.0 
add address=192.168.10.1/24 interface=ether4 network=192.168.10.0 
/ip dhcp-client 
add disabled=no interface=ether1 
/ip route 
add distance=1 dst-address=192.168.20.0/24 gateway=10.10.2.2 
add distance=1 dst-address=192.168.30.0/24 gateway=10.10.1.2 
/system identity 
set name=R01.MSK
```


**R01.FRT**

```
/interface wireless security-profiles 
set [ find default=yes ] supplicant-identity=MikroTik 
/ip pool 
add name=pool2 ranges=192.168.20.10-192.168.20.254 
/ip dhcp-server 
add address-pool=pool2 disabled=no interface=ether4 name=dhcp2 
/ip address 
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28 
add address=10.10.2.2/30 interface=ether3 network=10.10.2.0 
add address=10.10.3.1/30 interface=ether2 network=10.10.3.0 
add address=192.168.20.1/24 interface=ether4 network=192.168.20.0 
/ip dhcp-client 
add disabled=no interface=ether1 
/ip route 
add distance=1 dst-address=192.168.10.0/24 gateway=10.10.2.1 
add distance=1 dst-address=192.168.30.0/24 gateway=10.10.3.2 
/system identity 
set name=R01.FRT
```

**R01.BRL**

```
/interface wireless security-profiles 
set [ find default=yes ] supplicant-identity=MikroTik 
/ip pool 
add name=pool3 ranges=192.168.30.10-192.168.30.254 
/ip dhcp-server 
add address-pool=pool3 disabled=no interface=ether4 name=dhcp3 
/ip address 
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28 
add address=10.10.1.2/30 interface=ether2 network=10.10.1.0 
add address=10.10.3.2/30 interface=ether3 network=10.10.3.0 
add address=192.168.30.1/24 interface=ether4 network=192.168.30.0 
/ip dhcp-client 
add disabled=no interface=ether2 
/ip route 
add distance=1 dst-address=192.168.10.0/24 gateway=10.10.1.1 
add distance=1 dst-address=192.168.20.0/24 gateway=10.10.3.1 
/system identity 
set name=R01.BRL 
```

**PC1**

```
/interface wireless security-profiles 
set [ find default=yes ] supplicant-identity=MikroTik 
/ip address 
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28 
/ip dhcp-client 
add disabled=no interface=ether1 
add disabled=no interface=ether4 
/ip route 
add distance=1 dst-address=10.10.1.0/30 gateway=192.168.10.1 
add distance=1 dst-address=10.10.2.0/30 gateway=192.168.10.1 
add distance=1 dst-address=192.168.20.0/24 gateway=192.168.10.1 
add distance=1 dst-address=192.168.30.0/24 gateway=192.168.10.1 
/system identity 
set name=PC1 
```

**PC2**

```
/interface wireless security-profiles 
set [ find default=yes ] supplicant-identity=MikroTik 
/ip address 
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28 
/ip dhcp-client 
add disabled=no interface=ether1 
add disabled=no interface=ether4 
/ip route 
add distance=1 dst-address=10.10.2.0/30 gateway=192.168.20.1 
add distance=1 dst-address=10.10.3.0/30 gateway=192.168.20.1 
add distance=1 dst-address=192.168.10.0/24 gateway=192.168.20.1 
add distance=1 dst-address=192.168.30.0/24 gateway=192.168.20.1 
/system identity 
set name=PC2  
```

**PC3**

```
/interface wireless security-profiles 
set [ find default=yes ] supplicant-identity=MikroTik 
/ip address 
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28 
/ip dhcp-client 
add disabled=no interface=ether1 
add disabled=no interface=ether4 
/ip route 
add distance=1 dst-address=10.10.1.0/30 gateway=192.168.30.1 
add distance=1 dst-address=10.10.3.0/30 gateway=192.168.30.1 
add distance=1 dst-address=192.168.10.0/24 gateway=192.168.30.1 
add distance=1 dst-address=192.168.20.0/24 gateway=192.168.30.1 
/system identity 
set name=PC3 
```

#### 4. Результаты пингов

![ping1](./assets/images/lab2Ping1.png)
![ping2](./assets/images/lab2Ping2.png)

### Вывод

В ходе лабораторной работы №2 я ознакомился с принципами планирования IP адресов, настройке статической маршрутизации и сетевыми функциями устройств.