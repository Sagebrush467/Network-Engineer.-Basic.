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

Проверим на PC-A назначение адреса SLAAC от R1

```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 0003.E41C.C7AD
   Link-local IPv6 Address.........: FE80::203:E4FF:FE1C:C7AD
   IPv6 Address....................: 2001:DB8:ACAD:1:203:E4FF:FE1C:C7AD
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-84-9D-ED-A6-00-03-E4-1C-C7-AD
   DNS Servers.....................: ::
                                     0.0.0.0
```

При использовании SLAAC PC-A самостоятельно сгенерировал идентификатор хоста на снове своего MAC-адреса. Добавив FF:FE в середину мак-адреса и инвертировав седьмой бит.

### Настройка пула на R1

```
ipv6 dhcp pool R1-STATELESS
dns-server 2001:db8:acad::254
domain-name STATELESS.com
exit
interface g0/0/1
ipv6 nd other-config-flag
ipv6 dhcp server R1-STATELESS
end
copy running-config startup-config
```

- Проверка

```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: STATELESS.com 
   Physical Address................: 0003.E41C.C7AD
   Link-local IPv6 Address.........: FE80::203:E4FF:FE1C:C7AD
   IPv6 Address....................: 2001:DB8:ACAD:1:203:E4FF:FE1C:C7AD
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 1316044128
   DHCPv6 Client DUID..............: 00-01-00-01-84-9D-ED-A6-00-03-E4-1C-C7-AD
   DNS Servers.....................: 2001:DB8:ACAD::254
                                     0.0.0.0

Bluetooth Connection:

   Connection-specific DNS Suffix..: STATELESS.com 
   Physical Address................: 00E0.F960.6871
   Link-local IPv6 Address.........: ::
   IPv6 Address....................: ::
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: ::
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 1316044128
   DHCPv6 Client DUID..............: 00-01-00-01-84-9D-ED-A6-00-03-E4-1C-C7-AD
   DNS Servers.....................: ::
                                     0.0.0.0


