University: [ITMO University](https://itmo.ru/ru/) <br />
Faculty: [FICT](https://fict.itmo.ru) <br />
Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing) <br />
Year: 2022/2023 <br />
Group: K33202 <br />
Author: Donina Daria Dmitrievna <br />
Lab: Lab4 <br />
Date of create: 12.01.2023 <br />
Date of finished: 13.01.2023 <br />


# Цель работы
Изучить протоколы BGP, MPLS и правила организации L3VPN и VPLS.

# Ход работы
### 1. Создание топологии сети в файле *.clab.yml

```network.clab.yml
name: lab4

mgmt:
  network: my_mgmt_network_4
  ipv4_subnet: 172.18.1.0/24

topology:
  nodes:
    R01.NY:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.18.1.2
    R01.LND:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.18.1.5
    R01.LBN:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9  
      mgmt_ipv4: 172.18.1.8
    R01.HKI:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9  
      mgmt_ipv4: 172.18.1.9
    R01.SLV:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9  
      mgmt_ipv4: 172.18.1.6
    R01.SPB:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9  
      mgmt_ipv4: 172.18.1.10
    PC2:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.18.1.11
    PC1:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.18.1.12
    PC3:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.18.1.13
      

  links:
    - endpoints: ["R01.NY:eth2", "PC2:eth1"]
    - endpoints: ["R01.NY:eth1", "R01.LND:eth1"]
    - endpoints: ["R01.LND:eth2", "R01.HKI:eth1"]
    - endpoints: ["R01.LND:eth3", "R01.LBN:eth1"]
    - endpoints: ["R01.HKI:eth2", "R01.LBN:eth2"]
    - endpoints: ["R01.LBN:eth3", "R01.SVL:eth1"]
    - endpoints: ["R01.SVL:eth2", "PC3:eth1"]
    - endpoints: ["R01.HKI:eth3", "R01.SPB:eth1"]
    - endpoints: ["R01.SPB:eth2", "PC1:eth1"]
```

### 2. Развертывание сети
С помощью команды ```sudo containerlab deploy``` разворачиваем сеть.

# Первая часть

### 3. Создание схемы сети.

![lab4-1](https://user-images.githubusercontent.com/43678323/212186691-5ea8d9cf-fd6f-4f18-ac1b-ddcf0c72e3e9.png)

### 4. Настройка конфигурации сети.

#### RO1.SPB

На данном роутере необходимо настроить IP-адреса и интерфейсы в соответствии с топологией, MPLS, OSPF и iBGP маршрутизацию. На интерфейсe, смотрящем наружу,
надо настроить VRF, RT и RD.

Конфигурация R01.SPB:
```
/interface bridge
add name=loopback
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=10.10.10.6
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.6
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.0.6.2/30 interface=ether2 network=10.0.6.0
add address=192.168.10.1/24 interface=ether3 network=192.168.10.0
add address=10.10.10.6 interface=loopback network=10.10.10.6
/ip dhcp-client
add disabled=no interface=ether1
/ip route vrf
add export-route-targets=65530:777 import-route-targets=65530:777 interfaces=ether3 \
    route-distinguisher=65530:777 routing-mark=VRF_DEVOPS
/mpls ldp
set enabled=yes transport-address=10.10.10.6
/mpls ldp interface
add interface=ether2
/routing bgp instance vrf
add redistribute-connected=yes routing-mark=VRF_DEVOPS
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer6 remote-address=10.10.10.4 remote-as=\
    65530 update-source=loopback
/routing ospf network
add area=backbone
/system identity
set name=R01.SPB
```
#### RO1.NY
Настройка аналогична роутеру R01.SPB.

Конфигурация RO1.NY:

```
/interface bridge
add name=loopback
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=10.10.10.1
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.1
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.0.1.1/30 interface=ether2 network=10.0.1.0
add address=192.168.20.1/24 interface=ether3 network=192.168.20.0
add address=10.10.10.1 interface=loopback network=10.10.10.1
/ip dhcp-client
add disabled=no interface=ether1
/ip route vrf
add export-route-targets=65530:777 import-route-targets=65530:777 interfaces=ether3 \
    route-distinguisher=65530:777 routing-mark=VRF_DEVOPS
/mpls ldp
set enabled=yes transport-address=10.10.10.1
/mpls ldp interface
add interface=ether2
/routing bgp instance vrf
add redistribute-connected=yes routing-mark=VRF_DEVOPS
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=10.10.10.2 remote-as=\
    65530 update-source=loopback
/routing ospf network
add area=backbone
/system identity
set name=R01.NY
```

#### RO1.SVL
Настройка аналогична роутеру R01.SPB.

Конфигурация RO1.SVL:

```
/interface bridge
add name=loopback
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=10.10.10.5
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.5
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.0.5.2/30 interface=ether2 network=10.0.5.0
add address=192.168.30.1/24 interface=ether3 network=192.168.30.0
add address=10.10.10.5 interface=loopback network=10.10.10.5
/ip dhcp-client
add disabled=no interface=ether1
/ip route vrf
add export-route-targets=65530:777 import-route-targets=65530:777 interfaces=ether3 \
    route-distinguisher=65530:777 routing-mark=VRF_DEVOPS
/mpls ldp
set enabled=yes transport-address=10.10.10.5
/mpls ldp interface
add interface=ether2
/routing bgp instance vrf
add redistribute-connected=yes routing-mark=VRF_DEVOPS
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer5 remote-address=10.10.10.3 remote-as=\
    65530 update-source=loopback
/routing ospf network
add area=backbone
/system identity
set name=R01.SVL
```

#### RO1.LND

На этом и оставшихся внутренних роутерах необходимо также настроить IP-адреса и интерфейсы в соответствии с топологией, MPLS и OSPF маршрутизацию. 
В отличие от внешних роутеров, iBGP маршрутизация здесь будет с RR кластером. 

Конфигурация RO1.LND:

```
/interface bridge
add name=loopback
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=10.10.10.2
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.2
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.0.1.2/30 interface=ether2 network=10.0.1.0
add address=10.0.3.1/30 interface=ether3 network=10.0.3.0
add address=10.0.2.1/30 interface=ether4 network=10.0.2.0
add address=10.10.10.2 interface=loopback network=10.10.10.2
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=10.10.10.2
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer1 remote-address=10.10.10.1 remote-as=\
    65530 update-source=loopback
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer2 remote-address=10.10.10.3 remote-as=\
    65530 route-reflect=yes update-source=loopback
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer4 remote-address=10.10.10.4 remote-as=\
    65530 route-reflect=yes update-source=loopback
/routing ospf network
add area=backbone
/system identity
set name=R01.LND
```

#### RO1.LBN
Настройка аналогична роутеру R01.LND.

Конфигурация RO1.LBN:

```
/interface bridge
add name=loopback
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=10.10.10.3
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.3
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.0.2.2/30 interface=ether2 network=10.0.2.0
add address=10.0.4.1/30 interface=ether3 network=10.0.4.0
add address=10.0.5.1/30 interface=ether4 network=10.0.5.0
add address=10.10.10.3 interface=loopback network=10.10.10.3
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=10.10.10.3
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer3 remote-address=10.10.10.4 remote-as=\
    65530 route-reflect=yes update-source=loopback
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer2 remote-address=10.10.10.2 remote-as=\
    65530 route-reflect=yes update-source=loopback
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer5 remote-address=10.10.10.5 remote-as=\
    65530 update-source=loopback
/routing ospf network
add area=backbone
/system identity
set name=R01.LBN
```

#### RO1.HKI
Настройка аналогична роутеру R01.LND.

Конфигурация RO1.HKI:

```
/interface bridge
add name=loopback
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing bgp instance
set default router-id=10.10.10.4
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.4
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.0.3.2/30 interface=ether2 network=10.0.3.0
add address=10.0.4.2/30 interface=ether3 network=10.0.4.0
add address=10.0.6.1/30 interface=ether4 network=10.0.6.0
add address=10.10.10.4 interface=loopback network=10.10.10.4
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=10.10.10.4
/mpls ldp interface
add interface=ether2
add interface=ether3
add interface=ether4
/routing bgp peer
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer4 remote-address=10.10.10.2 remote-as=\
    65530 route-reflect=yes update-source=loopback
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer3 remote-address=10.10.10.3 remote-as=\
    65530 route-reflect=yes update-source=loopback
add address-families=ip,l2vpn,l2vpn-cisco,vpnv4 name=peer6 remote-address=10.10.10.6 remote-as=\
    65530 update-source=loopback
/routing ospf network
add area=backbone
/system identity
set name=R01.HKI
```

#### PC1-3

Для этих устройств необходимо добавить статические IP-адреса.

Конфигурация PC1-3:
```
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=192.168.10.2/24 interface=ether2 network=192.168.10.0
/ip dhcp-client
add disabled=no interface=ether1
/system identity
set name=PC1
```

### 5. Проверка связности VRF:

#### Маршруты BGP для R01.LND, все в состоянии Established:

![photo_2023-01-13_02-28-53](https://user-images.githubusercontent.com/43678323/212203013-1308527b-101f-41e8-a3e8-191d9baaaef3.jpg)

#### Проверка VRF_DEVOPS таблиц:
SPB -> NY:

![photo_2023-01-13_02-56-20](https://user-images.githubusercontent.com/43678323/212205924-349b63d3-f72d-4b17-ac74-9b15b74511e1.jpg)

NY -> SVL:

![photo_2023-01-13_02-56-23](https://user-images.githubusercontent.com/43678323/212205939-34df508c-e6ed-40ea-bb2e-e0e3fb97eb53.jpg)

SVL -> SPB:

![photo_2023-01-13_03-59-00](https://user-images.githubusercontent.com/43678323/212212807-b59745c2-40fc-4261-bcd8-4980093225b4.jpg)

# Вторая часть

### 3. Создание схемы сети.

![lab4-2](https://user-images.githubusercontent.com/43678323/212186747-f3ed23c8-fb41-4d6d-b62b-493fb89a8969.png)

### 4. Настройка конфигурации сети.

В этой части необходимо разобрать VRF и настроить VPLS на крайних роутeрах (SPB, NY, SVL) на внешние интерфейсы. Ниже приведены изменения в конфигурации:

#### R01.SPB

```
/ip route vrf remove 0
/routing bgp instance vrf remove 0
/ip address remove 2
/interface bridge
add name=VPLS_bridge
/interface vpls
add disabled=no l2mtu=1500 mac-address=02:EA:1B:7C:94:E8 name=VPLS1 \
    remote-peer=10.10.10.1 vpls-id=10:0
add disabled=no l2mtu=1500 mac-address=02:31:00:35:1F:2C name=VPLS2 \
    remote-peer=10.10.10.5 vpls-id=10:0
/interface bridge port
add bridge=VPLS_bridge interface=ether3
add bridge=VPLS_bridge interface=VPLS1
add bridge=VPLS_bridge interface=VPLS2
```

#### R01.NY

```
/ip route vrf remove 0
/routing bgp instance vrf remove 0
/ip address remove 2
/interface bridge
add name=VPLS_bridge
/interface vpls
add disabled=no l2mtu=1500 mac-address=02:0E:CB:29:A4:41 name=VPLS1 \
    remote-peer=10.10.10.5 vpls-id=10:0
add disabled=no l2mtu=1500 mac-address=02:00:AD:9A:FD:EA name=VPLS3 \
    remote-peer=10.10.10.6 vpls-id=10:0
/interface bridge port
add bridge=VPLS_bridge interface=ether3
add bridge=VPLS_bridge interface=VPLS1
add bridge=VPLS_bridge interface=VPLS3
```

#### R01.SVL

```
/ip route vrf remove 0
/routing bgp instance vrf remove 0
/ip address remove 2
/interface bridge
add name=VPLS_bridge
/interface vpls
add disabled=no l2mtu=1500 mac-address=02:E6:6D:0B:5F:AC name=VPLS2 \
    remote-peer=10.10.10.1 vpls-id=10:0
add disabled=no l2mtu=1500 mac-address=02:9F:AA:24:46:06 name=VPLS3 \
    remote-peer=10.10.10.6 vpls-id=10:0
/interface bridge port
add bridge=VPLS_bridge interface=ether3
add bridge=VPLS_bridge interface=VPLS2
add bridge=VPLS_bridge interface=VPLS3
```
#### PC1-3
На компьютерах устанавливаем IP-адреса в соответствии со второй схемой. Все компьютеры находятся в одной сети.

```
/interface wireless security-profiles 
set [ find default=yes ] supplicant-identity=MikroTik
/ip address 
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28 
add address=192.168.100.4/24 interface=ether2 network=192.168.100.0
/ip dhcp-client 
add disabled=no interface=ether1
/system identity 
set name=PC1
```
```
/interface wireless security-profiles 
set [ find default=yes ] supplicant-identity=MikroTik
/ip address 
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28 
add address=192.168.100.2/24 interface=ether2 network=192.168.100.0
/ip dhcp-client 
add disabled=no interface=ether1
/system identity 
set name=PC2
```
```
/interface wireless security-profiles 
set [ find default=yes ] supplicant-identity=MikroTik
/ip address 
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28 
add address=192.168.100.3/24 interface=ether2 network=192.168.100.0
/ip dhcp-client 
add disabled=no interface=ether1
/system identity 
set name=PC3
```

### 5. Проверка работоспособности VPLS:

PC1 -> PC2:

![photo_2023-01-13_03-59-50](https://user-images.githubusercontent.com/43678323/212212854-f22d36ff-623a-4dc4-a651-2d47f68679d9.jpg)

PC1 -> PC3:

![photo_2023-01-13_04-00-26](https://user-images.githubusercontent.com/43678323/212212916-0217b311-f8eb-46a3-ae7e-d259f62b4cd8.jpg)

PC3 -> PC2:

![photo_2023-01-13_04-00-52](https://user-images.githubusercontent.com/43678323/212212955-f12f29e6-13dd-4e66-9c6d-56b17d696363.jpg)

# Вывод
В ходе данной работы я изучила протоколы BGP, MPLS, а также принципы организации L3VPN и VPLS.
