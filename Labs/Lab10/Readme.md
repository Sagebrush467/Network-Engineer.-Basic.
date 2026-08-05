# Лабораторная работа №11. Настройка протокола OSPFv2 для одной области/

## Топология

![](topo.png)

## Таблица адресации.

|Устройство|Интерфейс|IP адрес|Маска подсети|
|----------|---------|--------|-------------|
|R1|G0/0/1|10.53.0.1|255.255.255.0|
| |Loopback1|172.16.1.1|255.255.255.0|
|R2|G0/0/1|10.53.0.2|255.255.255.0|
| |Loopback1|192.168.1.1|255.255.255.0|

## Цели
 
- Создание сети и настройка основных параметров устройства
 
- Настройка и проверка базовой работы протокола  OSPFv2 для одной области

- Оптимизация и проверка конфигурации OSPFv2 для одной области
 
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
exit
line vty 0 4
password cisco
login
exit
service password-encryption
banner motd #Unauthorized access is prohibited#
exit
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
exit
line vty 0 4
password cisco
login
exit
service password-encryption
banner motd #Unauthorized access is prohibited#
exit
copy running-config startup-config
```
 
 ### Базовая настройка коммутаторов
 
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
exit
line vty 0 15
password cisco
login
exit
service password-encryption
banner motd #Unauthorized access is prohibited#
exit
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
exit
line vty 0 15
password cisco
login
exit
service password-encryption
banner motd #Unauthorized access is prohibited#
exit
copy running-config startup-config
 ```
 
 ### Настройка и проверка базовой работы OSPFv2 для одной области

- R1

```
interface g0/0/1
ip address 10.53.0.1 255.255.255.0
no shutdown
exit
interface loopback 1
ip address 172.16.1.1 255.255.255.0
exit
router ospf 56
router-id 1.1.1.1
network 10.53.0.0 0.0.0.255 area 0
exit
```

- R2

```
interface g0/0/1
ip address 10.53.0.2 255.255.255.0
no shutdown
exit
interface loopback 1
ip address 192.168.1.1 255.255.255.0
exit
router ospf 56
router-id 2.2.2.2
network 10.53.0.0 0.0.0.255 area 0
network 192.168.1.0 0.0.0.255 area 0
```

- Проверка соседства

```
R1#show ip ospf neighbor


Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           1   FULL/BDR        00:00:39    10.53.0.2       GigabitEthernet0/0/1

R2#sho ip ospf neighbor 


Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1           1   FULL/DR         00:00:32    10.53.0.1       GigabitEthernet0/0/1

```

Из вывода команды видим что R2 является назначенным маршрутизатор (DR). R1 запасной назначенный маршрутизатор (BDR). 
При равных приоритетах (по умолчанию 1) DR становится маршрутизатор с наибольшим Router ID.

```
R1#show ip route ospf 
     192.168.1.0/32 is subnetted, 1 subnets
O       192.168.1.1 [110/2] via 10.53.0.2, 00:20:11, GigabitEthernet0/0/1

R1#ping 192.168.1.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/1 ms
```

### Оптимизация и проверка конфигурации OSPFv2 для одной области

- R1

```
interface g0/0/1
ip ospf priority 50
ip ospf hello-interval 30
exit
ip route 0.0.0.0 0.0.0.0 Loopback1
router ospf 56
default-information originate
exit
router ospf 56
auto-cost reference-bandwidth 10000
end
clear ip ospf process
```

- R2

```
interface g0/0/1
ip ospf hello-interval 30
exit
interface loopback 1
ip ospf network point-to-point
exit
router ospf 56
passive-interface loopback 1
auto-cost reference-bandwidth 10000
end
clear ip ospf process
```

- Проверка

- R1

```
GigabitEthernet0/0/1 is up, line protocol is up
  Internet address is 10.53.0.1/24, Area 0
  Process ID 56, Router ID 1.1.1.1, Network Type BROADCAST, Cost: 100
  Transmit Delay is 1 sec, State DR, Priority 50
  Designated Router (ID) 1.1.1.1, Interface address 10.53.0.1
  Backup Designated Router (ID) 2.2.2.2, Interface address 10.53.0.2
  Timer intervals configured, Hello 30, Dead 40, Wait 40, Retransmit 5
    Hello due in 00:00:09
  Index 1/1, flood queue length 0
  Next 0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 1
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 1, Adjacent neighbor count is 1
    Adjacent with neighbor 2.2.2.2  (Backup Designated Router)
  Suppress hello for 0 neighbor(s)
  
  R1#show ip route ospf
O    192.168.1.0 [110/101] via 10.53.0.2, 00:02:44, GigabitEthernet0/0/1

R2#show ip route ospf 
O*E2 0.0.0.0/0 [110/1] via 10.53.0.1, 00:15:05, GigabitEthernet0/0/1

R2#ping 172.16.1.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 172.16.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/2/10 ms
```

Маршрут по умолчанию - внешний типа E2 с фиксированной метрикой (1), а сеть 192.168.1.0/24 - внутренний OSPF-маршрут, чья метрика (101) равна сумме стоимостей интерфейсов по пути. Поэтому значения разные.
