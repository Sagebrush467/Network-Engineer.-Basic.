# Лабораторная работа №6. Внедрение маршрутизации между виртуальными локальными сетями.

## Топология

![](topo.png)

## Таблица адресации

|Устройство|Интерфейс|IP-адрес|маска подсети|Шлюз по умолчанию|
|----------|---------|--------|-------------|-----------------|
|R1|G0/0/1.10|192.168.10.1|255.255.255.0|-|
| |G0/0/1.20|192.168.20.1|255.255.255.0|-|
| |G0/0/1.30|192.168.30.1|255.255.255.0|-|
| |G0/0/1.1000|-|-|-|
|S1|VLAN 10|192.168.10.11|255.255.255.0|192.168.10.1|
|S2|VLAN 10|192.168.10.12|255.255.255.0|192.168.10.1|
|PC-A|NIC|192.168.20.3|255.255.255.0|192.168.20.1|
|PC-B|NIC|192.168.30.3|255.255.255.0|192.168.30.1|

## Таблица VLAN

|VLAN|Имя|Назначенный интерфейс|
|----|---|---------------------|
|10|Управление|S1: VLAN 10|
| | |S:2 VLAN 10|
|20|Sales|S1: F0/6|
|30|Operations|S2: F0/18|
|999|Parking_Lot|C:1 F0/2-4, F0/7-24, G0/1-2|
| | |C2: F0/2-17, F0/19-24, G0/1-2|
|1000|Собственная|-|

## Задачи

- Создание сети и настройка основных параметров устройства

- Создание сетей VLAN и назначение портов коммутатора

- Настройка транка 802.1Q между коммутаторами

- Настройка межвлановой маршрутизации

- Проверка межвлановой маршрутизации

## Выполнение

- Создадим сеть согласно топологии и гастроим базовые параметры маршрутизатора.

```
Router>en

Router#conf t

Router(config)#hostname R1

R1(config)#no ip domain-lookup

R1(config)#enable secret class

R1(config)#line con 0

R1(config-line)#password cisco

R1(config-line)#login

R1(config-line)#exit

R1(config-line)#exit

R1(config)#line vty 0 15

R1(config-line)#password cisco

R1(config-line)#login

R1(config-line)#exit

R1(config)#service password-encryption

R1(config)#banner motd c Nesanktsionirovanniy dostup zapreschyon c

R1(config)#ex

R1#clock set 10:00:00 20 May 2026

R1#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]

```
- Настроим базовые параметры коммутаторов. Настройки будут одинаковыми за исключением имени и в большинстве случаев времени если не используется ntp сервер.

```
Switch>en

Switch#conf t

Switch(config)#hostname S1

S1(config)#no ip domain-lookup

S1(config)#enable secret class

S1(config)#line con 0

S1(config-line)#password class

S1(config-line)#login

S1(config-line)#exit

S1(config)#line vty 0 15

S1(config-line)#password cisco

S1(config-line)#login

S1(config-line)#exit

S1(config)#service password-encryption

S1(config)#banner motd c Nesanktsionirovanniy dostup zapreschyon c

```

не имея часов можно проверить время на коммутаторе с настроенными часами

```
R1#show clock

10:12:57.597 UTC Wed May 20 2026

```

- настроим время на часах коммутаторов и сохраним конфигурацию

```
S1#clock set 10:12:57 20 May 2026

S1#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]

```

если бы использовался ntp сервер, то в простейшем выиде это навстройка бы так

задаётся адрес ntp сервера, часовой пояс

```

S1(config)#ntp server 192.168.100.100

S1(config)# clock timezone MSK 3

S1(config)# end

S1# copy running-config startup-config

```

- После настройки коммутаторов, зададим настройки ip адреса для PC-A и PC-B.

![](01.png)

![](02.png)

- Создадим и назовём необходимые VLAN на каждом коммутаторе.

S1

```
S1(config)#vlan 10
S1(config-vlan)#name Upravlenie
S1(config-vlan)#ex

S1(config)#vlan 20
S1(config-vlan)#name Sales
S1(config-vlan)#ex

S1(config)#vlan 30
S1(config-vlan)#name Operations
S1(config-vlan)#ex

S1(config)#vlan 999
S1(config-vlan)#name Parking_Lot
S1(config-vlan)#ex

S1(config)#Vlan 1000
S1(config-vlan)#name Sobstvennaya
S1(config)#exit
```

S2

```
S2(config)#vlan 10
S2(config-vlan)#name Upravlenie
S2(config-vlan)#ex

S2(config)#vlan 20
S2(config-vlan)#name Sales
S2(config-vlan)#ex

S2(config)#vlan 30
S2(config-vlan)#name Operations
S2(config-vlan)#ex

S2(config)#vlan 999
S2(config-vlan)#name Parking_Lot
S2(config-vlan)#ex

S2(config)#Vlan 1000
S2(config-vlan)#name Sobstvennaya
S2(config)#ex
```

- Настроим интерфейс управления и шлюз по умолчанию на каждом коммутаторе

S1

```
S1(config)#int vl 10
S1(config-if)#
%LINK-5-CHANGED: Interface Vlan10, changed state to up
S1(config-if)#ip address 192.168.10.11 255.255.255.0
S1(config-if)#ex
S1(config)#ip default-gateway 192.168.10.1
```
S2

```
S2(config)#int vl 10
S2(config-if)#
%LINK-5-CHANGED: Interface Vlan10, changed state to up
S2(config-if)#ip address 192.168.10.12 255.255.255.0
S2(config-if)#ex
S2(config)#ip default-gateway 192.168.10.1
```

- Назначим все неиспользуемые порты коммутатора VLAN Parking_Lot, настроим их для статического режима доступа и административно деактивируем их.

S1

```
S1(config)#interface range fastEthernet 0/2-4
S1(config-if-range)#shutdown 
S1(config-if-range)#switchport access vlan 999
S1(config-if-range)#switchport mode access 
S1(config-if-range)# ex

S1(config)#interface range fastEthernet 0/7-24
S1(config-if-range)#shutdown
S1(config-if-range)#switchport access vlan 999
S1(config-if-range)#switchport mode access 
S1(config-if-range)#ex

S1(config)#interface range gi0/1-2
S1(config-if-range)#shutdown 
S1(config-if-range)#switchport access vlan 999
S1(config-if-range)#switchport mode access
```

S2

```
S2(config)#interface range fastEthernet 0/2-17
S2(config-if-range)#shutdown
S2(config-if-range)#switchport access vlan 999
S2(config-if-range)#switchport mode access
S2(config-if-range)#ex

S2(config)#interface range fastEthernet 0/19-24
S2(config-if-range)#shutdown
S2(config-if-range)#switchport access vlan 999
S2(config-if-range)#switchport mode access
S2(config-if-range)#ex

S2(config)#interface range gigabitEthernet 0/1-2
S2(config-if-range)#shutdown
S2(config-if-range)#switchport access vlan 999
S2(config-if-range)#switchport mode access
```

 - Назначим используемые порты соответствующей VLAN и настроим их для режима статического доступа и проверим корректность назначения интерфейсов.

S1

```
S1(config)# interface fastEthernet 0/6
S1(config-if)#description Sales
S1(config-if)#switchport mode access 
S1(config-if)#switchport access vlan 20
```

```
S1(config-if)#do sho vl br

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/5
10   Upravlenie                       active
20   Sales                            active    Fa0/6
30   Operations                       active    Fa0/18
999  Parking_Lot                      active    Fa0/2, Fa0/3, Fa0/4, Fa0/7
                                                Fa0/8, Fa0/9, Fa0/10, Fa0/11
                                                Fa0/12, Fa0/13, Fa0/14, Fa0/15
                                                Fa0/16, Fa0/17, Fa0/18, Fa0/19
                                                Fa0/20, Fa0/21, Fa0/22, Fa0/23
                                                Fa0/24, Gig0/1, Gig0/2
1000 Sobstvennaya                     active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active 
```

S2

```
S2(config)#interface fastEthernet 0/18
S2(config-if)#description Operations
S2(config-if)#switchport mode access
S2(config-if)#switchport acc vl 30
```

```
S2(config-if)#do sho vl br

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1
10   Upravlenie                       active
20   Sales                            active    Fa0/6
30   Operations                       active    Fa0/18
999  Parking_Lot                      active    Fa0/2, Fa0/3, Fa0/4, Fa0/5
                                                Fa0/6, Fa0/7, Fa0/8, Fa0/9
                                                Fa0/10, Fa0/11, Fa0/12, Fa0/13
                                                Fa0/14, Fa0/15, Fa0/16, Fa0/17
                                                Fa0/19, Fa0/20, Fa0/21, Fa0/22
                                                Fa0/23, Fa0/24, Gig0/1, Gig0/2
1000 Sobstvennaya                     active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    
```

- Настроим транкинг на интерфейсе F0/1 коммутаторов и проверим настройки

S1

```
S1(config)#int fa0/1

S1(config-if)#descr Uplink_to_S2

S1(config-if)#switchport mode trunk

S1(config-if)#switchport trunk native vlan 1000
S1(config-if)#%SPANTREE-2-RECV_PVID_ERR: Received BPDU with inconsistent peer vlan id 1 on FastEthernet0/1 VLAN1000.

%SPANTREE-2-BLOCK_PVID_LOCAL: Blocking FastEthernet0/1 on VLAN1000. Inconsistent local vlan.

S1(config-if)#switchport trunk allowed vlan 1000
S1(config-if)#switchport trunk allowed vlan add 10
S1(config-if)#switchport trunk allowed vlan add 20
S1(config-if)#switchport trunk allowed vlan add 30

S1#sho run
!
interface FastEthernet0/1
 description Uplink_to_S2
 switchport trunk native vlan 1000
 switchport trunk allowed vlan 10,20,30,1000
 switchport mode trunk
!
```
Для S2 настройки вланов идентичны.

- настроим интерфейс F0/5 на S1 и проверим настройки

```
S1(config)#interface fastEthernet 0/5

S1(config-if)#description Uplink_to_R1

S1(config-if)#switchport mode trunk

S1(config-if)#switchport trunk native vlan 1000

S1(config-if)#switchport trunk allowed vlan 1000
S1(config-if)#switchport trunk allowed vlan add 10
S1(config-if)#switchport trunk allowed vlan add 20
S1(config-if)#switchport trunk allowed vlan add 30

S1(config-if)#end

S1#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]

S1#sho run
!
interface FastEthernet0/5
 description Uplink_to_R1
 switchport trunk native vlan 1000
 switchport trunk allowed vlan 10,20,30,1000
 switchport mode trunk
!
```

Интерфейс G0/0/1 на R1 по умолчанию отключен, поэтому сейчас линка нет, даже если интерфейс будет настроен марщрутизация проходить не будет.

- Настроим интерфейс G0/0/1 и его сабинтерфейсы на R1 и проверим настройки

```
R1#conf t

R1(config)#int g0/0/1

R1(config-if)#no shut

R1(config-if)#

%LINK-5-CHANGED: Interface GigabitEthernet0/0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1, changed state to up

R1(config-if)#ex



R1(config)#int g0/0/1.10

R1(config-subif)#

%LINK-3-UPDOWN: Interface GigabitEthernet0/0/1.10, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.10, changed state to up

R1(config-subif)#encapsulation dot1Q 10

R1(config-subif)#ip add 192.168.10.1 255.255.255.0

R1(config-subif)#description Upravlenie

R1(config-if)#ex



R1(config)#int gi 0/0/1.20

R1(config-subif)#

%LINK-3-UPDOWN: Interface GigabitEthernet0/0/1.20, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.20, changed state to up

R1(config-subif)#encapsulation dot1Q 20

R1(config-subif)#ip add 192.168.20.1 255.255.255.0

R1(config-subif)#description Sales

R1(config-subif)#ex



R1(config)#int g0/0/1.30

R1(config-subif)#

%LINK-3-UPDOWN: Interface GigabitEthernet0/0/1.30, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.30, changed state to up

R1(config-subif)#encapsulation dot1Q 30

R1(config-subif)#ip add 192.168.30.1 255.255.255.0

R1(config-subif)#description Upravlenie

R1(config-subif)#ex



R1(config)#int g0/0/1.1000

R1(config-subif)#

%LINK-3-UPDOWN: Interface GigabitEthernet0/0/1.1000, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.1000, changed state to up

R1(config-subif)#encapsulation dot1Q 1000 native 



R1(config-subif)#do sho run

!
interface GigabitEthernet0/0/1
 no ip address
 duplex auto
 speed auto
!
interface GigabitEthernet0/0/1.10
 description Upravlenie
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
!
interface GigabitEthernet0/0/1.20
 description Sales
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
!
interface GigabitEthernet0/0/1.30
 description Operations
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
!
interface GigabitEthernet0/0/1.1000
 encapsulation dot1Q 1000 native
 no ip address
!



R1(config-subif)#do sho int

GigabitEthernet0/0/1 is up, line protocol is up (connected)
  Hardware is Lance, address is 0060.5c7a.0602 (bia 0060.5c7a.0602)
  MTU 1500 bytes, BW 100000 Kbit, DLY 100 usec,
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, loopback not set
  Full-duplex, 100Mb/s, media type is RJ45
  ARP type: ARPA, ARP Timeout 04:00:00, 
  Last input 00:00:08, output 00:00:05, output hang never
  Last clearing of "show interface" counters never
  Input queue: 0/75/0 (size/max/drops); Total output drops: 0
  Queueing strategy: fifo
  Output queue :0/40 (size/max)
  5 minute input rate 0 bits/sec, 0 packets/sec
  5 minute output rate 0 bits/sec, 0 packets/sec
     0 packets input, 0 bytes, 0 no buffer
     Received 0 broadcasts, 0 runts, 0 giants, 0 throttles
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored, 0 abort
     0 input packets with dribble condition detected
     0 packets output, 0 bytes, 0 underruns
     0 output errors, 0 collisions, 2 interface resets
     0 babbles, 0 late collision, 0 deferred
     0 lost carrier, 0 no carrier
     0 output buffer failures, 0 output buffers swapped out


GigabitEthernet0/0/1.10 is up, line protocol is up (connected)
  Hardware is PQUICC_FEC, address is 0060.5c7a.0602 (bia 0060.5c7a.0602)
  Internet address is 192.168.10.1/24
  MTU 1500 bytes, BW 100000 Kbit, DLY 100 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation 802.1Q Virtual LAN, Vlan ID 10
  ARP type: ARPA, ARP Timeout 04:00:00, 
  Last clearing of "show interface" counters never


GigabitEthernet0/0/1.20 is up, line protocol is up (connected)
  Hardware is PQUICC_FEC, address is 0060.5c7a.0602 (bia 0060.5c7a.0602)
  Internet address is 192.168.20.1/24
  MTU 1500 bytes, BW 100000 Kbit, DLY 100 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation 802.1Q Virtual LAN, Vlan ID 20
  ARP type: ARPA, ARP Timeout 04:00:00, 
  Last clearing of "show interface" counters never


GigabitEthernet0/0/1.30 is up, line protocol is up (connected)
  Hardware is PQUICC_FEC, address is 0060.5c7a.0602 (bia 0060.5c7a.0602)
  Internet address is 192.168.30.1/24
  MTU 1500 bytes, BW 100000 Kbit, DLY 100 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation 802.1Q Virtual LAN, Vlan ID 30
  ARP type: ARPA, ARP Timeout 04:00:00, 
  Last clearing of "show interface" counters never


GigabitEthernet0/0/1.1000 is up, line protocol is up (connected)
  Hardware is PQUICC_FEC, address is 0060.5c7a.0602 (bia 0060.5c7a.0602)
  MTU 1500 bytes, BW 100000 Kbit, DLY 100 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation 802.1Q Virtual LAN, Vlan ID 1000
  ARP type: ARPA, ARP Timeout 04:00:00, 
  Last clearing of "show interface" counters never
Vlan1 is administratively down, line protocol is down
  Hardware is CPU Interface, address is 0001.4327.8d30 (bia 0001.4327.8d30)
  MTU 1500 bytes, BW 100000 Kbit, DLY 1000000 usec,
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, loopback not set
  ARP type: ARPA, ARP Timeout 04:00:00
  Last input 21:40:21, output never, output hang never
  Last clearing of "show interface" counters never
  Input queue: 0/75/0/0 (size/max/drops/flushes); Total output drops: 0
  Queueing strategy: fifo
  Output queue: 0/40 (size/max)
  5 minute input rate 0 bits/sec, 0 packets/sec
  5 minute output rate 0 bits/sec, 0 packets/sec
     1682 packets input, 530955 bytes, 0 no buffer
     Received 0 broadcasts (0 IP multicast)
     0 runts, 0 giants, 0 throttles
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     563859 packets output, 0 bytes, 0 underruns
     0 output errors, 23 interface resets
     0 output buffer failures, 0 output buffers swapped out
```
- Отправим эхо-запрос с PC-A на шлюз по умолчанию, эхо-запрос с PC-A на PC-B, с компьютера PC-A на коммутатор S2.

с PC-A на шлюз

```
C:\>ping 192.168.20.1

Pinging 192.168.20.1 with 32 bytes of data:

Reply from 192.168.20.1: bytes=32 time<1ms TTL=255
Reply from 192.168.20.1: bytes=32 time<1ms TTL=255
Reply from 192.168.20.1: bytes=32 time<1ms TTL=255
Reply from 192.168.20.1: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.20.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

с PC-A на PC-B

```
C:\>ping 192.168.30.3

Pinging 192.168.30.3 with 32 bytes of data:

Reply from 192.168.30.3: bytes=32 time<1ms TTL=127
Reply from 192.168.30.3: bytes=32 time<1ms TTL=127
Reply from 192.168.30.3: bytes=32 time<1ms TTL=127
Reply from 192.168.30.3: bytes=32 time<1ms TTL=127

Ping statistics for 192.168.30.3:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

c PC-A на S2

```
C:\>ping 192.168.10.12

Pinging 192.168.10.12 with 32 bytes of data:

Request timed out.
Request timed out.
Reply from 192.168.10.12: bytes=32 time<1ms TTL=254
Reply from 192.168.10.12: bytes=32 time<1ms TTL=254

Ping statistics for 192.168.10.12:
    Packets: Sent = 4, Received = 2, Lost = 2 (50% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

C:\>ping 192.168.10.12

Pinging 192.168.10.12 with 32 bytes of data:

Reply from 192.168.10.12: bytes=32 time<1ms TTL=254
Reply from 192.168.10.12: bytes=32 time<1ms TTL=254
Reply from 192.168.10.12: bytes=32 time<1ms TTL=254
Reply from 192.168.10.12: bytes=32 time<1ms TTL=254

Ping statistics for 192.168.10.12:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

- Выполним трассировку с PC-B до PC-A

```
C:\>tracert 192.168.20.3

Tracing route to 192.168.20.3 over a maximum of 30 hops: 

  1   0 ms      0 ms      0 ms      192.168.30.1
  2   1 ms      0 ms      0 ms      192.168.20.3

Trace complete.

C:\>
```

По трассировке видим что PC-B изначально обращается к своему шлюзу `192.168.30.1` после чего пакет попадает к PC-B `192.168.20.3`, находящимся во Vlan 20.
