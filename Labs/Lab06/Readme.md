# Лабораторная работа №7. Развертывание коммутируемой сети с резервными каналами

## Топология

![](topo.png)

## Таблица адресации
|Устройство|Интерфейс|IP-адрес|Маска подсети|
|----------|---------|--------|-------------|
|S1|VLAN1|192.168.1.1|255.255.255.0|
|S2|VLAN1|192.168.1.2|255.255.255.0|
|S3|VLAN1|192.168.1.3|255.255.255.0|

## Задачи
- Часть 1. Создание сети и настройка основных параметров устройства
- Часть 2. Выбор корневого моста
- Часть 3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов
- Часть 4. Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов

## Выполнение

- Создадим cеть и настроим основные параметры.

![](01.png)

S1

```
Switch>en
Switch#conf t

Switch(config)#no ip domain-lookup
Switch(config)#hostname S1
S1(config)#enable secret class

S1(config)#line con 0
S1(config-line)#password cisco
S1(config-line)#logging synchronous
S1(config-line)#login
S1(config-line)#exit

S1(config)#line vty 0 15
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit

S1(config)#banner motd c
Enter TEXT message.  End with the character 'c'.
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!UNAUTHORIZED ACCESS PROHIBITED!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!UNAUTHORIZED ACCESS PROHIBITED!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!UNAUTHORIZED ACCESS PROHIBITED!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!UNAUTHORIZED ACCESS PROHIBITED!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!UNAUTHORIZED ACCESS PROHIBITED!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!! c

S1(config)#service password-encryption
S1(config)#interface vlan 1
S1(config-if)#ip address 192.168.1.1 255.255.255.0
S1(config-if)#no shutdown
S1(config-if)#end

S1#copy running-config startup-config
Destination filename [startup-config]? 
Building configuration...
[OK]
```

S2

```
Switch>en
Switch#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#no ip domain-lookup
Switch(config)#hostname S2
S2(config)#enable secret class
S2(config)#line con 0
S2(config-line)#password cisco
S2(config-line)#logging synchronous
S2(config-line)#login
S2(config-line)#exit
S2(config)#line vty 0 15
S2(config-line)#password cisco
S2(config-line)#login
S2(config-line)#exit
S2(config)#banner motd c
Enter TEXT message.  End with the character 'c'.
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!UNAUTHORIZED ACCESS PROHIBITED!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!UNAUTHORIZED ACCESS PROHIBITED!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!UNAUTHORIZED ACCESS PROHIBITED!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!UNAUTHORIZED ACCESS PROHIBITED!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!UNAUTHORIZED ACCESS PROHIBITED!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!! c

S2(config)#service password-encryption
S2(config)#interface vlan 1
S2(config-if)#ip address 192.168.1.2 255.255.255.0
S2(config-if)#no shutdown
S2(config-if)#end
S2#copy running-config startup-config
```
S3

```
Switch>en
Switch#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#no ip domain-lookup
Switch(config)#hostname S3
S3(config)#enable secret class
S3(config)#line con 0
S3(config-line)#password cisco
S3(config-line)#logging synchronous
S3(config-line)#login
S3(config-line)#exit
S3(config)#line vty 0 15
S3(config-line)#password cisco
S3(config-line)#login
S3(config-line)#exit
S3(config)#banner motd c
Enter TEXT message.  End with the character 'c'.
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!UNAUTHORIZED ACCESS PROHIBITED!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!UNAUTHORIZED ACCESS PROHIBITED!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!UNAUTHORIZED ACCESS PROHIBITED!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!UNAUTHORIZED ACCESS PROHIBITED!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!UNAUTHORIZED ACCESS PROHIBITED!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!! c

S3(config)#service password-encryption
S3(config)#interface vlan 1
S3(config-if)#ip address 192.168.1.3 255.255.255.0
S3(config-if)#no shutdown
S3(config-if)#end
S3#copy running-config startup-config
```
проверим связность

```
S1#ping 192.168.1.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
..!!!
Success rate is 60 percent (3/5), round-trip min/avg/max = 0/0/0 ms
S1#ping 192.168.1.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
..!!!
Success rate is 60 percent (3/5), round-trip min/avg/max = 0/0/0 ms

S2#ping 192.168.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
S2#ping 192.168.1.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
..!!!
Success rate is 60 percent (3/5), round-trip min/avg/max = 0/0/0 ms

S3#ping 192.168.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
S3#ping 192.168.1.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms
```
