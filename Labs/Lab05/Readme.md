# Лабораторная работа №6.

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

Создадим сеть согласно топологии и гастроим базовые параметры маршрутизатора.

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
Настроим базовые параметры коммутаторов. Настройки будут одинаковыми за исключением имени.

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

настроим время на часах коммутаторов и сохраним конфигурацию

```
S1#clock set 10:12:57 20 May 2026

S1#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]

```
