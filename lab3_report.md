University: [ITMO University](https://itmo.ru/ru/) <br />
Faculty: [FICT](https://fict.itmo.ru) <br />
Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing) <br />
Year: 2022/2023 <br />
Group: K33202 <br />
Author: Donina Daria Dmitrievna <br />
Lab: Lab3 <br />
Date of create: 11.01.2023 <br />
Date of finished: 13.01.2023 <br />


# Цель работы
Изучить протоколы OSPF и MPLS, механизмы организации EoMPLS.

# Ход работы
### 1. Создание топологии сети в файле *.clab.yml

```network.clab.yml
name: lab3

mgmt:
  network: my_mgmt_network_3
  ipv4_subnet: 172.30.10.0/24

topology:
  nodes:
    R01.NY:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.30.10.11
    R01.LND:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.30.10.12
    R01.LBN:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9  
      mgmt_ipv4: 172.30.10.13
    R01.HKI:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9  
      mgmt_ipv4: 172.30.10.14
    R01.MSK:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9  
      mgmt_ipv4: 172.30.10.15
    R01.SPB:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9  
      mgmt_ipv4: 172.30.10.16
    SGI_Prism:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.30.10.17
    PC1:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.30.10.18
      

  links:
    - endpoints: ["R01.NY:eth3", "SGI_Prism:eth1"]
    - endpoints: ["R01.NY:eth1", "R01.LND:eth1"]
    - endpoints: ["R01.NY:eth2", "R01.LBN:eth1"]
    - endpoints: ["R01.LND:eth2", "R01.HKI:eth1"]
    - endpoints: ["R01.LBN:eth2", "R01.HKI:eth2"]
    - endpoints: ["R01.LBN:eth3", "R01.MSK:eth1"]
    - endpoints: ["R01.HKI:eth3", "R01.SPB:eth1"]
    - endpoints: ["R01.MSK:eth2", "R01.SPB:eth2"]
    - endpoints: ["R01.SPB:eth3", "PC1:eth1"]
```

### 2. Развертывание сети
С помощью команды ```sudo containerlab deploy``` разворачиваем сеть.

### 3. Создание схемы сети.

![lab3-topology-2](https://user-images.githubusercontent.com/43678323/211931149-80262add-1b6d-4dfd-8c9e-5406a398830a.png)

### 4. Настройка конфигурации сети.

#### RO1.SPB

Для начала создадим новый интерфейс ```loopback```, который необходим для настройки OSPF и MPLS, а также добавим IP-адреса всем интерфейсам, 
в соответствии с топологией сети.

```
/interface bridge
add name=loopback

/ip address
add address=10.0.5.2/30 interface=ether2 network=10.0.5.0
add address=10.0.6.2/30 interface=ether3 network=10.0.6.0
add address=10.10.10.6 interface=loopback network=10.10.10.6
```

Теперь включим MPLS на роутере на интерфейсах, смотрящих внутрь сети:
```
/mpls ldp
set enabled=yes transport-address=10.10.10.6
/mpls ldp interface
add interface=ether2
add interface=ether3
```

Для запуска динамической маршрутизации по OSPF вводим следующие команды:
```
/routing ospf instance
set 0 router-id=10.10.10.6
/routing ospf network
add area=backbone
```

И, наконец, настраиваем EoMPLS. Его необходимо настроить на двух крайних роутерах (NY и SPB) на интерфейсах, смотрящих наружу:
```
/interface bridge
add name=EoMPLS_bridge
/interface vpls
add cisco-style=yes cisco-style-id=222 disabled=no name=EoMPLS remote-peer=10.10.10.1
/interface bridge port
add bridge=EoMPLS_bridge interface=ether4
add bridge=EoMPLS_bridge interface=EoMPLS
```

Меняем имя устройства, если необходимо, и устанавливаем пользователю новый пароль:
```
/system-identity set name=R01.SPB
/user set 0 password=******** 
```

Итоговая конфигурация R01.SPB:
```
/interface bridge
add name=EoMPLS_bridge
add name=loopback
/interface vpls
add cisco-style=yes cisco-style-id=222 disabled=no l2mtu=1500 mac-address=02:81:F2:1D:97:CC name=EoMPLS remote-peer=\
    10.10.10.1
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.6
/interface bridge port
add bridge=EoMPLS_bridge interface=ether4
add bridge=EoMPLS_bridge interface=EoMPLS
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.0.5.2/30 interface=ether2 network=10.0.5.0
add address=10.0.6.2/30 interface=ether3 network=10.0.6.0
add address=10.10.10.6 interface=loopback network=10.10.10.6
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=10.10.10.6
/mpls ldp interface
add interface=ether2
add interface=ether3
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
add name=EoMPLS_bridge
add name=loopback
/interface vpls
add cisco-style=yes cisco-style-id=222 disabled=no l2mtu=1500 mac-address=02:73:F4:04:43:C5 name=EoMPLS remote-peer=\
    10.10.10.6
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.1
/interface bridge port
add bridge=EoMPLS_bridge interface=ether4
add bridge=EoMPLS_bridge interface=EoMPLS
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.0.0.1/30 interface=ether2 network=10.0.0.0
add address=10.0.1.1/30 interface=ether3 network=10.0.1.0
add address=10.10.10.1 interface=loopback network=10.10.10.1
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=10.10.10.1
/mpls ldp interface
add interface=ether2
add interface=ether3
/routing ospf network
add area=backbone
/system identity
set name=R01.NY
```

#### RO1.LND

На этом и последующих роутерах необходимо аналогично предыдущим настроить OSPF, MPLS, интерфейсы и IP-адреса.

Конфигурация RO1.LND:

```
/interface bridge
add name=loopback
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.2
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.0.0.2/30 interface=ether2 network=10.0.0.0
add address=10.0.2.1/30 interface=ether3 network=10.0.2.0
add address=10.10.10.2 interface=loopback network=10.10.10.2
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=10.10.10.2
/mpls ldp interface
add interface=ether2
add interface=ether3
/routing ospf network
add area=backbone
/system identity
set name=R01.LND
```

#### RO1.LBN

Конфигурация RO1.LBN:

```
/interface bridge
add name=loopback
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.3
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.0.1.2/30 interface=ether2 network=10.0.1.0
add address=10.0.3.1/30 interface=ether3 network=10.0.3.0
add address=10.0.4.1/30 interface=ether4 network=10.0.4.0
add address=10.10.10.3 interface=loopback network=10.10.10.3
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
set name=R01.LBN
```

#### RO1.HKI

Конфигурация RO1.HKI:

```
/interface bridge
add name=loopback
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.4
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.0.2.2/30 interface=ether2 network=10.0.2.0
add address=10.0.3.2/30 interface=ether3 network=10.0.3.0
add address=10.0.5.1/30 interface=ether4 network=10.0.5.0
add address=10.10.10.4 interface=loopback network=10.10.10.4
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
set name=R01.HKI
```

#### RO1.MSK

Конфигурация RO1.MSK:

```
/interface bridge
add name=loopback
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing ospf instance
set [ find default=yes ] router-id=10.10.10.5
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=10.0.4.2/30 interface=ether2 network=10.0.4.0
add address=10.0.6.1/30 interface=ether3 network=10.0.6.0
add address=10.10.10.5 interface=loopback network=10.10.10.5
/ip dhcp-client
add disabled=no interface=ether1
/mpls ldp
set enabled=yes transport-address=10.10.10.5
/mpls ldp interface
add interface=ether2
add interface=ether3
/routing ospf network
add area=backbone
/system identity
set name=R01.MSK
```

#### PC1, SGI Prism

Для этих устройств необходимо добавить статические IP-адреса.

Конфигурация PC1:
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

Конфигурация SGI Prism:
```
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=192.168.10.3/24 interface=ether2 network=192.168.10.0
/ip dhcp-client
add disabled=no interface=ether1
/system identity
set name=SGI_Prism
```

### 5. Проверка работоспрособности:

PC1 -> SGI_Prism:
![photo_2023-01-12_02-59-45](https://user-images.githubusercontent.com/43678323/211943860-10834c03-50e1-48e0-bc4c-a069ce35bbe4.jpg)

SGI_Prism -> PC1:
![photo_2023-01-12_02-59-43](https://user-images.githubusercontent.com/43678323/211943852-fa54b2f9-945e-463a-8317-00b1c5ac090a.jpg)

# Таблица маршрутизации роутера R01.SPB:

![photo_2023-01-12_03-57-58](https://user-images.githubusercontent.com/43678323/211950526-b8431a79-6f2a-4766-a2aa-16df43e7cada.jpg)

# Трассировка NY-SPB

![photo_2023-01-12_04-04-14](https://user-images.githubusercontent.com/43678323/211951325-7e0b8fb4-d732-45e4-bf3f-90e273dd3f1b.jpg)

# Вывод
В ходе данной работы я изучила протоколы OSPF и MPLS, а также принципы организации EoMPLS.
