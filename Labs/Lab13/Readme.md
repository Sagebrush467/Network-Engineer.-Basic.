# Лабораторная работа №14 Настройка протоколов CDP, LLDP и NTP

## Топология

![](topo.png)

## Таблица адресации

|Устройство|Интерфейс|IP-Адрес|Маска подсети|Шлюз по умолчанию|
|----------|---------|--------|-------------|-----------------|
|R1|Loopback|172.16.1.1|255.255.255.0|-|
| |G0/0/1|10.22.0.1|255.255.255.0|-|
|S1|SVI VLAN 1|10.22.0.2|255.255.255.0|10.22.0.1|
|S2|SVI VLAN 1|10.22.0.2|255.255.255.0|10.22.0.1|

### Базовые параметры

- R1

```
hostname R1
no ip domain-lookup
enable secret class
line console 0
password cisco
login
exit 
line vty 0 4
password cisco
login
exit
service password-encryption
banner motd cUnauthorized access prohibitedc
interface loopback 1
ip address 172.16.1.1 255.255.255.0
exit
interface gi0/0/1
ip address 10.22.0.1 255.255.255.0
no shutdown
end
copy running-config startup-config
```

- S1

```
hostname S1
no ip domain-lookup
enable secret class
line console 0
password cisco
login
exit
line vty 0 4
password cisco
login
exit
service password-encryption
banner motd cOnly authorized users!c
interface vlan 1
ip address 10.22.0.2 255.255.255.0
no shutdown
exit
ip default-gateway 10.22.0.1
interface range fa0/2-4, fa0/6-24, g0/1-2
shutdown
end
copy running-config startup-config
```

- S2

```
hostname S2
no ip domain-lookup
enable secret class
line console 0
password cisco
login
exit
line vty 0 4
password cisco
login
exit
service password-encryption
banner motd cOnly authorized users!c
interface vlan 1
ip address 10.22.0.3 255.255.255.0
no shutdown
exit
ip default-gateway 10.22.0.1
interface range fa0/2-24, g0/1-2
shutdown
end
copy running-config startup-config
```

## Обнаружение сетевых устройств с помощью CDP.

- R1

```
show cdp interface

Vlan1 is administratively down, line protocol is down
  Sending CDP packets every 60 seconds
  Holdtime is 180 seconds
GigabitEthernet0/0/0 is administratively down, line protocol is down
  Sending CDP packets every 60 seconds
  Holdtime is 180 seconds
GigabitEthernet0/0/1 is up, line protocol is up
  Sending CDP packets every 60 seconds
  Holdtime is 180 seconds
```

Из вывода видно что в CDP объявлениях учавствуют три интерфейса. Активен только интерфейс GigabitEthernet0/0/1.

```
cdp entry *

Device ID: S1
Entry address(es): 
  IP address : 10.22.0.2
Platform: cisco 2960, Capabilities: Switch
Interface: GigabitEthernet0/0/1, Port ID (outgoing port): FastEthernet0/5
Holdtime: 165

Version :
Cisco IOS Software, C2960 Software (C2960-LANBASEK9-M), Version 15.0(2)SE4, RELEASE SOFTWARE (fc1)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2013 by Cisco Systems, Inc.
Compiled Wed 26-Jun-13 02:49 by mnguyen

advertisement version: 2
Duplex: full
```

Из вывода видно что что на S1 используется весия IOS 15.0(2)SE4

```
show cdp ?
  entry      Information for specific neighbor entry
  interface  CDP interface status and configuration
  neighbors  CDP neighbor entries
  <cr>
```

Из вывода видно что команда `show cdp traffic` в cisco packet tracer не поддерживается. Если бы она поддерживалась то можно было бы увидеть вывод следующего содержания:

```
S1# show cdp traffic
CDP counters : 
        Total packets output: 179, Input: 148 
        Hdr syntax: 0, Chksum error: 0, Encaps failed: 0 
        No memory: 0, Invalid packet: 0, 
        CDP version 1 advertisements output: 0, Input: 0 
        CDP version 2 advertisements output: 179, Input: 148
```

согласно выводу видно что было отправлено 179 пакетов.

После настройки SVI на коммутаторах дополнительной информации в cisco packet tracer при использовании команды `show cdp entry` не отображается. Bначе можно было бы наблюдать следующий вывод:

```
R1 # show cdp entry  S1 
-------------------------
Device ID: S1
Entry address(es):
  IP address: 10.22.0.2
Platform: cisco WS-C2960+24LC-L, Capabilities: Switch IGMP 
Interface: GigabitEthernet0/0/1, Port ID (outgoing port): FastEthernet0/5
Holdtime : 133 sec

Version :
Cisco IOS Software, C2960 Software (C2960-LANBASEK9-M), Version 15.2(4)E8, RELEASE SOFTWARE (fc3) 
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2019 by Cisco Systems, Inc.
Compiled Fri 15-Mar-19 17:28 by prod_rel_team 

advertisement version: 2
VTP Management Domain: ''
Native VLAN: 1
Duplex: full
Management address(es):
  IP address: 10.22.0.2 
```

Из вывода видим ip управления S1, Native VLAN, задан ли VTP домен на коммутаторе.

Отключим CDP на всех устройствах используя команду `no cdp run`

## Обнаружение сетевых ресурсов с помощью протокола LLDP

Включим на устройствах обнаружение по LLDP, используя команду `lldp run`

Cisco packet tracer не поддерживает на коммутаторах команду `show lldp entry`
Иначе был бы следующий вывод:

```
S1# show lldp entry S2

Capability codes:
    (R) Router, (B) Bridge, (T) Telephone, (C) DOCSIS Cable Device
    (W) WLAN Access Point, (P) Repeater, (S) Station, (O) Other
------------------------------------------------
Local Intf: Fa0/1  
Chassis id: c025.5cd7.ef00 
Port id: Fa0/1 
Port Description: FastEthernet0/1
System Name: S2

System Description:
Cisco IOS Software, C2960 Software (C2960-LANBASEK9-M), Version 15.2(4)E8, RELEASE SOFTWARE (fc3) 
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2019 by Cisco Systems, Inc.
Compiled Fri 15-Mar-19 17:28 by prod_rel_team 

Time remaining: 109 seconds 
System Capabilities: B
Enabled Capabilities: B
Management Addresses:
    IP: 10.22.0.3 
Auto Negotiation - supported, enabled
Physical media capabilities:
    100base-TX(FD)
    100base-TX(HD)
    10base-T(FD)
    10base-T(HD)
Media Attachment Unit type: 16
Vlan ID: 1


Total entries displayed: 1
```

в качестве Chassis id используется мак адрес устройства.

изучить топологию можно на устройствах с помощью команды `show lldp neighbors`

## настройка NTP

проверим время на R1

```
show clock detail
*4:5:41.384 UTC Mon Mar 1 1993
Time source is hardware calendar
```

|Дата|Время|Часовой пояс|Источник|
|----|-----|------------|--------|
|Пон 1 Мар 1993|4:05:41.384|UTC +0|системный календарь|

установим время и настроим R1 в качестве хозяина NTP  с уровнем слоя 4

```
clock set 10:00:00 1 Jan 2000
confingure terminal
ntp master 4
```

проверим время на S1 и S2

- S1

```
show clock detail
*4:13:53.984 UTC Mon Mar 1 1993
Time source is hardware calendar
```

|Дата|Время|Часовой пояс|Источник|
|----|-----|------------|--------|
|Пон 1 Мар 1993|4:13:53.984|UTC +0|системный календарь|

- S2
```
S2#sho clock det
*4:13:52.708 UTC Mon Mar 1 1993
Time source is hardware calendar
```

|Дата|Время|Часовой пояс|Источник|
|----|-----|------------|--------|
|Пон 1 Мар 1993|4:13:52.708|UTC +0|системный календарь|

введём на обоих устройствах команду `ntp server 10.22.0.1` и через несколько минут проверим результат

- S1

```
show clock detail 
10:3:45.522 UTC Sat Jan 1 2000
Time source is NTP
```

- S2

```
sho clock det
10:4:1.692 UTC Sat Jan 1 2000
Time source is NTP
```

Для повторения:
Необходимо отключать CDP и LLDP на интерфейсах подключающихся к внешним сетям для обеспечения безопасности чтобы злоумышленник не мог использовать информацию о подключенном устройстве для получения несанкционированного доступа. Так как вывод команд `show lldp` и `show cdp` даёт достаточно информации для планирования атаки.
