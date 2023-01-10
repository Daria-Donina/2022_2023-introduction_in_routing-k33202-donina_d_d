University: [ITMO University](https://itmo.ru/ru/) <br />
Faculty: [FICT](https://fict.itmo.ru) <br />
Course: [Introduction in routing](https://github.com/itmo-ict-faculty/introduction-in-routing) <br />
Year: 2022/2023 <br />
Group: K33202 <br />
Author: Donina Daria Dmitrievna <br />
Lab: Lab2 <br />
Date of create: 10.01.2023 <br />
Date of finished: 13.01.2023 <br />


# Цель работы
Ознакомиться с принципами планирования IP адресов, настройкой статической маршрутизации и сетевыми функциями устройств.

# Ход работы
### 1. Создание топологии сети в файле *.clab.yml

```network.clab.yml
name: lab2

mgmt:
  network: my_mgmt_network_2
  ipv4_subnet: 172.18.1.0/24

topology:
  nodes:
    R01.MSK:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.18.1.2
    R01.BRL:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.18.1.5
    R01.FRT:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9  
      mgmt_ipv4: 172.18.1.8
    PC1:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.18.1.9
    PC2:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.18.1.6
    PC3:
      kind: vr-ros
      image: vrnetlab/vr-routeros:6.47.9
      mgmt_ipv4: 172.18.1.10
      

  links:
    - endpoints: ["R01.MSK:eth3", "PC1:eth1"]
    - endpoints: ["R01.MSK:eth2", "R01.FRT:eth2"]
    - endpoints: ["R01.MSK:eth1", "R01.BRL:eth1"]
    - endpoints: ["R01.BRL:eth3", "PC3:eth1"]
    - endpoints: ["R01.BRL:eth2", "R01.FRT:eth1"]
    - endpoints: ["R01.FRT:eth3", "PC2:eth1"]
```

### 2. Развертывание сети
С помощью команды ```sudo containerlab deploy``` разворачиваем сеть.

### 3. Создание схемы сети.


### 4. Настройка конфигурации сети.

#### RO1.MSK

Настраиваем DHCP-сервер на ether4, как в 1 лабораторной:
```
/ip pool
add name=pool1 ranges=192.168.10.2-192.168.10.254

/ip dhcp-server
add address-pool=pool1 disabled=no interface=ether4 name=dhcp10

/ip dhcp-server network
add address=192.168.10.0/24 gateway=192.168.10.1
```

На интерфейсах, которые смотрят на другие роутеры, надо настроить статические ip-адреса для маршрутизации. Адреса выбираются в соответствии со схемой:
```
/ip address
add address=10.10.1.1/30 interface=ether2 network=10.10.1.0
add address=10.10.2.2/30 interface=ether3 network=10.10.2.0
```

И, наконец, настраиваем маршруты в соответствии со схемой:
```
/ip route
add dst-address=192.168.20.0/24 gateway=10.10.2.1
add dst-address=192.168.30.0/24 gateway=10.10.1.2
```

Меняем имя устройства, если необходимо, и устанавливаем пользователю новый пароль:
```
/system-identity set name=R01
/user set 0 password=******** 
```

Итоговая конфигурация R01.MSK:
```
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip pool
add name=pool1 ranges=192.168.10.2-192.168.10.254
/ip dhcp-server
add address-pool=pool1 disabled=no interface=ether4 name=dhcp10
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=192.168.10.1/24 interface=ether4 network=192.168.10.0
add address=10.10.1.1/30 interface=ether2 network=10.10.1.0
add address=10.10.2.2/30 interface=ether3 network=10.10.2.0
/ip dhcp-client
add disabled=no interface=ether1
/ip dhcp-server network
add address=192.168.10.0/24 gateway=192.168.10.1
/ip route
add distance=1 dst-address=192.168.20.0/24 gateway=10.10.2.1
add distance=1 dst-address=192.168.30.0/24 gateway=10.10.1.2
/system identity
set name=R01.MSK
```
#### RO1.FRT
Настройка аналогична роутеру R01.MSK.

Конфигурация RO1.FRT:

```
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip pool
add name=pool20 ranges=192.168.20.2-192.168.20.254
/ip dhcp-server
add address-pool=pool20 disabled=no interface=ether4 name=dhcp20
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=192.168.20.1/24 interface=ether4 network=192.168.20.0
add address=10.10.2.1/30 interface=ether3 network=10.10.2.0
add address=10.10.3.2/30 interface=ether2 network=10.10.3.0
/ip dhcp-client
add disabled=no interface=ether1
/ip dhcp-server network
add address=192.168.20.0/24 gateway=192.168.20.1
/ip route
add distance=1 dst-address=192.168.10.0/24 gateway=10.10.2.2
add distance=1 dst-address=192.168.30.0/24 gateway=10.10.3.1
/system identity
set name=R01.FRT
```

#### RO1.BRL

```
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip pool
add name=pool30 ranges=192.168.30.2-192.168.30.254
/ip dhcp-server
add address-pool=pool30 disabled=no interface=ether4 name=dhcp30
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
add address=192.168.30.1/24 interface=ether4 network=192.168.30.0
add address=10.10.1.2/30 interface=ether2 network=10.10.1.0
add address=10.10.3.1/30 interface=ether3 network=10.10.3.0
/ip dhcp-client
add disabled=no interface=ether1
/ip dhcp-server network
add address=192.168.30.0/24 gateway=192.168.30.1
/ip route
add distance=1 dst-address=192.168.10.0/24 gateway=10.10.1.1
add distance=1 dst-address=192.168.20.0/24 gateway=10.10.3.2
/system identity
set name=R01.BRL
```

#### PC1-3

Здесь на ether2 надо включить DHCP-клиент:

```
/ip dhcp-client 
add disabled=no interface=ether2
```

Конфигурация:
```
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip address
add address=172.31.255.30/30 interface=ether1 network=172.31.255.28
/ip dhcp-client
add disabled=no interface=ether1
add disabled=no interface=ether2
/system identity
set name=PC1
```

### 5. Проверка работоспрособности:

PC1 -> PC2:

![photo_2023-01-11_01-12-57](https://user-images.githubusercontent.com/43678323/211673426-b89ff89d-c988-4f7e-8323-7bb059b5ccf3.jpg)

PC1 -> PC3:

![photo_2023-01-11_01-13-34](https://user-images.githubusercontent.com/43678323/211673487-675b54d3-0aaf-4b7b-a1cc-41d2e1876e47.jpg)

PC2 -> PC3:

![photo_2023-01-11_01-13-58](https://user-images.githubusercontent.com/43678323/211673601-476c31f8-cd40-42c6-af61-8d0daf87b703.jpg)

# Вывод
В ходе данной работы я ознакомилась с принципами планирования IP адресов, настройкой статической маршрутизации и сетевыми функциями устройств.

