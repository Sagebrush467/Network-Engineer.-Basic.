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

### Базовая настройка маршрутизаторов

- R1

```
enable
configure terminal
no ip domain-lookup
banner motd #Unauthorized access prohibited#
hostname R1
service password-encryption
ipv6 unicast-routing
enable secret class
line console 0
password cisco
login
line vty 0 4
password cisco
login
exit
end
copy running-config startup-config
```

- R2

```
enable
configure terminal
no ip domain-lookup
banner motd #Unauthorized access prohibited#
hostname R2
service password-encryption
ipv6 unicast-routing
enable secret class
line console 0
password cisco
login
line vty 0 4
password cisco
login
exit
end
copy running-config startup-config
```

### Настройка маршрутизации.

- R1

```
interface g0/0/0
ipv6 address 2001:db8:acad:2::1/64
ipv6 address fe80::1 link-local
no shutdown
exit
interface g0/0/1
ipv6 address 2001:db8:acad:1::1/64
ipv6 address fe80::1 link-local
no shutdown
exit
ipv6 route ::/0 2001:db8:acad:2::2
```

- R2

```
interface g0/0/0
ipv6 address 2001:db8:acad:2::2/64
ipv6 address fe80::2 link-local
no shutdown
exit
interface g0/0/1
ipv6 address 2001:db8:acad:3::1/64
ipv6 address fe80::1 link-local
no shutdown
exit
ipv6 route ::/0 2001:db8:acad:2::1
```

- Проверка

```
R2#ping 2001:db8:acad:2::1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:db8:acad:2::1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms

R2#ping 2001:db8:acad:1::1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:db8:acad:1::1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/1 ms

R1#ping 2001:db8:acad:2::2

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:db8:acad:2::2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms

R1#ping 2001:db8:acad:3::1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 2001:db8:acad:3::1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
```

