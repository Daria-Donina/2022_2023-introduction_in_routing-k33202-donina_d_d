University: [ITMO University](https://itmo.ru/ru/) <br />
Faculty: [FICT](https://fict.itmo.ru) <br />
Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing) <br />
Year: 2022/2023 <br />
Group: K33202 <br />
Author: Donina Daria Dmitrievna <br />
Lab: Lab1 <br />
Date of create: 08.01.2023 <br />
Date of finished: 13.01.2023 <br />


# Цель работы
Ознакомиться с инструментом ContainerLab и методами работы с ним, изучить способы настройки VLAN, IP-адресации и DHCP.

# Ход работы
### 1. Создание топологии сети в файле *.clab.yml

```network1.clab.yml
name: lab1

mgmt:
  network: my_mgmt_network
  ipv4_subnet: 172.18.1.0/24

topology:
  nodes:
    R01.TEST:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.18.1.2
    SW01.01.TEST:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.18.1.8
    SW02.01.TEST:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9  
      mgmt_ipv4: 172.18.1.9
    SW02.02.TEST:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.18.1.5
    PC1:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.18.1.6
    PC2:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.18.1.10
      

  links:
    - endpoints: ["R01.TEST:eth1", "SW01.01.TEST:eth1"]
    - endpoints: ["SW01.01.TEST:eth2", "SW02.01.TEST:eth1"]
    - endpoints: ["SW01.01.TEST:eth3", "SW02.02.TEST:eth1"]
    - endpoints: ["SW02.01.TEST:eth2", "PC1:eth1"]
    - endpoints: ["SW02.02.TEST:eth2", "PC2:eth1"]
```

### 2. Развертывание сети
С помощью команды ```sudo containerlab deploy``` разворачиваем сеть, выводится следующая информация:

![photo_2023-01-09_22-47-04](https://user-images.githubusercontent.com/43678323/211395151-3289a5bf-ad0b-4436-85ab-92eb588095a5.jpg)

### 3. Настройка конфигурации сети.
Подключение ко всем устройствам происходит по ssh, команда ```ssh admin@[ip-address]```.

#### RO1
Сначала производим настройку VLAN-ов:
```
/interface vlan
add interface=ether2 name=vlan10 vlan-id=10
add interface=ether2 name=vlan20 vlan-id=20

/ip address
add address=192.168.10.1/24 interface=vlan10 network=192.168.10.0
add address=192.168.20.1/24 interface=vlan20 network=192.168.20.0
```

Затем настраиваем DHCP-сервер:
```
/ip pool
add name=pool10 ranges=192.168.10.2-192.168.10.254
add name=pool20 ranges=192.168.20.2-192.168.20.254

/ip dhcp-server
add address-pool=pool10 disabled=no interface=vlan10 name=dhcp10
add address-pool=pool20 disabled=no interface-vlan20 name=dhcp20

/ip dhcp-server network
add address=192.168.10.0/24 gateway=192.168.10.1
add address=192.168.20.0/24 gateway=192.168.20.1
```
