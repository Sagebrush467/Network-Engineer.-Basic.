# Лабораторная работа №8. Реализация DHCPv4.

## Топология

![](topo.png)

## Таблица адресации

|устройство|интерфейс|ip-адрес|маска подсети|шлюз по умолчанию|
|----------|---------|--------|-------------|-----------------|
|R1|G0/0/0|	10.0.0.1|255.255.255.252|-|
| |G0/0/1|-|-|-|
| |G0/0/1.100|192.168.1.65|255.255.255.192|-|
| |G0/0/1.200|192.168.1.1|255.255.255.192|-|
| |G0/0/1.1000|-|-|-|
|R2|G0/0/0|10.0.0.2|255.255.255.252|-|
| |G0/0/1|192.168.1.129|255.255.255.240|-|
|S1|VLAN 200|192.168.1.2|255.255.255.192|192.168.1.1|
|S2|VLAN 1|192.168.1.130|255.255.255.240|192.168.1.129|
|PC-A|DHCP|DHCP|DHCP|DHCP|
|PC-B|DHCP|DHCP|DHCP|DHCP|

## Таблица VLAN

|VLAN|Имя|Назначенный интерфейс|
|----|---|---------------------|
|1|Нет|S2:F0/18|
|100|Клиенты|S1:F0/6|
|200|Управление|S1:VLAN 200|
|999|Parking_Lot|S1:F0/1-4,F0/7-24,G0/1-2|
|1000|Собственная|-|

## Задачи

- Создание сети и настройка основных параметров устройства

- Настройка и проверка двух серверов DHCPv4 на R1

- Настройка и проверка DHCP-ретрансляции на R2

### Базовая настройка маршрутизаторов

- R1

```
enable
configure terminal
hostname R1
no ip domain-lookup
enable secret class
line console 0
 password cisco
 login
line vty 0 4
 password cisco
 login
service password-encryption
banner motd #Unauthorized access prohibited#
exit
clock set 13:41:00 27 Jun 2026
copy running-config startup-config
```

- R2

```
enable
configure terminal
hostname R2
no ip domain-lookup
enable secret class
line console 0
 password cisco
 login
line vty 0 4
 password cisco
 login
service password-encryption
banner motd #Unauthorized access prohibited#
exit
clock set 13:44 27 Jun 2026
copy running-config startup-config
```
