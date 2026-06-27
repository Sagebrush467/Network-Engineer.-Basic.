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
- Проверим работу маршрутизации

```
R1#ping 192.168.1.129

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.129, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 0/0/0 ms
```


### Базовая Настройка коммутаторов

- S1

```
enable
configure terminal
hostname S1
no ip domain-lookup
enable secret class
line console 0
password cisco
login
line vty 0 15
password cisco
login
exit
service password-encryption
banner motd #Unauthorized access prohibited#
vlan 100
name Clients
vlan 200
name Management
vlan 999
name Parking_Lot
vlan 1000
name Native
exit
ip default-gateway 192.168.1.1
interface vlan 200
ip address 192.168.1.2 255.255.255.192
no shutdown
interface range f0/1-4, f0/7-24, g0/1-2
switchport mode access
switchport access vlan 999
shutdown
interface f0/6
switchport mode access
switchport access vlan 100
interface f0/5
switchport mode trunk
switchport trunk native vlan 1000
switchport trunk allowed vlan 100,200,1000
end
clock set 14:23:00 27 jun 2026
copy running-config startup-config
```

- S2
```
enable
configure terminal
hostname S2
no ip domain-lookup
enable secret class
line console 0
password cisco
login
line vty 0 15
password cisco
login
exit
service password-encryption
banner motd #Unauthorized access prohibited#
ip default-gateway 192.168.1.129
interface vlan 1
ip address 192.168.1.130 255.255.255.240
no shutdown
interface f0/5
switchport mode access
switchport access vlan 1
no shutdown
interface f0/18
switchport mode access
switchport access vlan 1
no shutdown
interface range f0/1-4, f0/6-17, f0/19-24, g0/1-2
shutdown
end
clock set 14:31 jun 27 2026
copy running-config startup-config
```

### Настройка DHCP серверов на R1

- Исключаем первые 5 адресов в каждой подсети, где будет работать DHCP

```
ip dhcp excluded-address 192.168.1.65 192.168.1.69
ip dhcp excluded-address 192.168.1.129 192.168.1.133
```

- Пул для подсети A

```
ip dhcp pool R1_Client_LAN
network 192.168.1.64 255.255.255.192
default-router 192.168.1.65
domain-name CCNA-lab.com
````

В Cisco packet tracer lease time не поддерживается

```
R1(dhcp-config)#lease ?
% Unrecognized command

R1(dhcp-config)#option 51 ip 192.168.1.70
%This version of PT does not support options other than 43 and 150
```
### DHCP ретрансляция на R2

```
interface g0/0/1
ip helper-address 10.0.0.1
end
copy running-config startup-config
```


### Проверка

```
R1#sho ip dhcp binding 
IP address       Client-ID/              Lease expiration        Type
                 Hardware address
192.168.1.70     0000.0C98.8DD3           --                     Automatic
192.168.1.134    000B.BEC2.76C1           --                     Automatic

PC-B
C:\>ipconfig /all
FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: CCNA-lab.com
   Physical Address................: 000B.BEC2.76C1
   Link-local IPv6 Address.........: FE80::20B:BEFF:FEC2:76C1
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.1.134
   Subnet Mask.....................: 255.255.255.240
   Default Gateway.................: ::
                                     192.168.1.129
   DHCP Servers....................: 10.0.0.1
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-88-CA-46-80-00-0B-BE-C2-76-C1
   DNS Servers.....................: ::
                                     0.0.0.0
PC-A
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: CCNA-lab.com
   Physical Address................: 0000.0C98.8DD3
   Link-local IPv6 Address.........: FE80::200:CFF:FE98:8DD3
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.1.70
   Subnet Mask.....................: 255.255.255.192
   Default Gateway.................: ::
                                     192.168.1.65
   DHCP Servers....................: 192.168.1.65
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-2B-9D-4A-EE-00-00-0C-98-8D-D3
   DNS Servers.....................: ::
                                     0.0.0.0
```
