# Лабораторная работа №9.  Настройка DHCPv6

## Топология

![](topo.png)

## Таблица адресации

|Устройство|Интерфейс|IPv6-адрес|
|----------|---------|----------|
|R1|G0/0/0|2001:db8:acad:2::1/64|
| | |fe80::1|
| |G0/0/1|2001:db8:acad:1::1/64|
| | |fe80::1|
|R2|G0/0/0|2001:db8:acad:2::2/64|
| | |fe80::2|
| |G0/0/1|2001:db8:acad:3::1/64|
| | |fe80::1|
|PC-A|NIC|DHCP|
|PC-B|NIC|DHCP|

## Задачи
- Создание сети и настройка основных параметров устройства
- Проверка назначения адреса SLAAC от R1
- Настройка и проверка сервера DHCPv6 без гражданства на R1
- Настройка и проверка состояния DHCPv6 сервера на R1
- Настройка и проверка DHCPv6 Relay на R2

### Базовая настройка коммутаторов

- S1

```
enable
configure terminal
no ip domain-lookup
hostname S1
banner motd #Unauthorized access prohibited#
service password-encryption
enable secret class
line console 0
password cisco
login
line vty 0 15
password cisco
login
exit
interface range f0/1-4,f0/7-24,g0/1-2
shutdown
end
copy running-config startup-config
```

- S2

```
enable
configure terminal
no ip domain-lookup
hostname S2
banner motd #Unauthorized access prohibited#
service password-encryption
enable secret class
line console 0
password cisco
login
line vty 0 15
password cisco
login
exit
interface range f0/1-4,f0/6-17,f0/19-24,g0/1-2
shutdown
end
copy running-config startup-config
```
