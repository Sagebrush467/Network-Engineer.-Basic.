# Лабораторная работа №10. Конфигурация безопасности коммутатора.

## Топология

![](topo.png)

## Таблица адресации

|Устройство|Interface/Vlan|IP-адрес|Маска подсети|
|----------|--------------|--------|-------------|
|R1|G0/0/1|192.168.10.1|255.255.255.0|
| |Loopback 0 |10.10.1.1|255.255.255.0|
|S1|VLAN 10|192.168.10.201|255.255.255.0|
|S2|VLAN 10|192.168.10.202|255.255.255.0|
|PC-A|NIC|DHCP|255.255.255.0|
|PC-B|NIV|DHCP|255.255.255.0|

## Задачи
- Реализация магистральных соединений 802.1Q.
- Настройка портов доступа
- Безопасность неиспользуемых портов коммутатора
- Документирование и реализация функций безопасности порта.
- Реализовать безопасность DHCP snooping .
- Реализация PortFast и BPDU Guard
- Проверка сквозной связанности.

### Настройка маршрутизатора

```
enable
configure terminal
hostname R1
no ip domain lookup
ip dhcp excluded-address 192.168.10.1 192.168.10.9
ip dhcp excluded-address 192.168.10.201 192.168.10.202
ip dhcp relay informationtrust-all
ip dhcp pool Students
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1
domain-name CCNA2.Lab-11.6.1
interface Loopback0
ip address 10.10.1.1 255.255.255.0
interface GigabitEthernet0/0/1
description Link to S1
ip address 192.168.10.1 255.255.255.0
no shutdown
line con 0
logging synchronous
exec-timeout 0 0
end
copy running-config startup-config
```

- Проверка конфигурации

```
R1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0/0   unassigned      YES NVRAM  administratively down down 
GigabitEthernet0/0/1   192.168.10.1    YES manual up                    up 
GigabitEthernet0/0/2   unassigned      YES NVRAM  administratively down down 
Loopback0              10.10.1.1       YES manual up                    up 
Vlan1                  unassigned      YES NVRAM  administratively down down
```

### Базовая настройка коммутаторов

- S1

```
enable
configure terminal
hostname S1
no ip domain-lookup
interface fastEthernet 0/5
description Link to R1
interface fastEthernet 0/6
description Link to PC-A
interface fastEthernet 0/1
description Trunk to S2
ip default-gateway 192.168.10.1
vlan 10
name Management
vlan 333
name Native
vlan 999
name ParkingLot
interface vlan 10
ip address 192.168.10.201 255.255.255.0
no shutdown
description Management SVI
end
copy running-config startup-config
```

- S2

```
enable
configure terminal
hostname S2
no ip domain-lookup
interface fastEthernet 0/1
description Trunk to S1
interface fastEthernet 0/18
description Link to PC-B
ip default-gateway 192.168.10.1
vlan 10
name Management
vlan 333
name Native
vlan 999
name ParkingLot
interface vlan 10
ip address 192.168.10.202 255.255.255.0
no shutdown
description Management SVI
end
copy running-config startup-config
```
