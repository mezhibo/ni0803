**Лабораторная работа "Построение корпоративной сети с использованием технологии GRE"**

Топология сети для выполнения лабораторной работы представлена на картинке ниже:

![Image alt](https://github.com/mezhibo/ni0803/blob/b005f8796a4a7c0b279b2109ed40e35c6b4bbc62/IMG/1.jpg)


Описание: есть центральный офис с маршрутизаторами, на каждом из которых настроен выход в интернет. Есть удаленный офис со своим выходом в интеренет. Необходимо объединить корпоративную сеть посредством GRE туннеля и динамической маршрутизации.



**Задание 1**

Разработать план IP-адресов для схемы сети выше.


**Решение 1**


![Image alt](https://github.com/mezhibo/ni0803/blob/b005f8796a4a7c0b279b2109ed40e35c6b4bbc62/IMG/4.jpg)


![Image alt](https://github.com/mezhibo/ni0803/blob/b005f8796a4a7c0b279b2109ed40e35c6b4bbc62/IMG/2.jpg)



**Задание 2.**

Настроить протокол FHRP для резервирования default gateway.


**Решение 2**

В рамках задания для резервирования настраивается протокол HSRP (проприетарный Cisco). Протокол HSRP рассчитан на 2 роутера. Путь по умолчанию - через роутер R0, при пропадании внешнего линка маршрут перестраивается на R1.

Команды настройки маршрутизаторов:

```
-- Настройка R0 --
R0(config)#int gi0/0/1
R0(config-if)#standby 1 ip 192.168.0.10
R0(config-if)#standby version 2
R0(config-if)#standby 1 priority 105
R0(config-if)#standby 1 track gi0/0/0
R0(config-if)#standby 1 preempt

-- Настройка R1 --
R1(config)#int gi0/0/1
R1(config-if)#standby 1 ip 192.168.0.10
R1(config-if)#standby version 2
R1(config-if)#standby 1 priority 100
R1(config-if)#standby 1 preempt
```

[PKT-файл](https://github.com/mezhibo/ni0803/blob/b005f8796a4a7c0b279b2109ed40e35c6b4bbc62/IMG/0803-02-01.pkt)



**Задание 3.**

Поднять туннели GRE, используя все каналы выхода в интернет.



**Решение 3**

Команды настройки туннелей между парами маршрутизаторов:

```
--- R0 - INTERNET - R0 ---
INTERNET(config)#int tunnel 0
INTERNET(config-if)#ip address 10.0.0.1 255.255.255.0
INTERNET(config-if)#tunnel source gi0/1
INTERNET(config-if)#tunnel destination 100.0.0.1

R0(config)#int tunnel 0
R0(config-if)#ip address 10.0.0.2 255.255.255.0
R0(config-if)#tunnel source gi0/0/0
R0(config-if)#tunnel destination 100.0.0.2

--- R1 - INTERNET - R1 ---
INTERNET(config)#int tunnel 1
INTERNET(config-if)#ip address 10.0.1.1 255.255.255.0
INTERNET(config-if)#tunnel source gi0/0
INTERNET(config-if)#tunnel destination 100.0.1.1

R1(config)#int tunnel 1
R1(config-if)#ip address 10.0.1.2 255.255.255.0
R1(config-if)#tunnel source gi0/0/0
R1(config-if)#tunnel destination 100.0.1.2

--- R2 - INTERNET - R2 ---
INTERNET(config)#int tunnel 2
INTERNET(config-if)#ip address 10.0.2.1 255.255.255.0
INTERNET(config-if)#tunnel source gi0/2
INTERNET(config-if)#tunnel destination 100.0.2.1

R2(config)#int tunnel 2
R2(config-if)#ip address 10.0.2.2 255.255.255.0
R2(config-if)#tunnel source gi0/0/0
R2(config-if)#tunnel destination 100.0.2.2
```

[PKT-файл](https://github.com/mezhibo/ni0803/blob/b005f8796a4a7c0b279b2109ed40e35c6b4bbc62/IMG/0803-03-01.pkt)



**Задание 4.**

Настроить динамическую маршрутизацию для работы GRE туннелей.



**Решение 4**

Для организации динамической маршрутизации в данной топологии выбран протокол OSPF.

Команды настройки динамической маршрутизации:


```
--- INTERNET ---
INTERNET(config)#router ospf 1
INTERNET(config-router)#network 0.0.0.0 255.255.255.255 area 0

--- R0 ---
R0(config)#router ospf 1
R0(config-router)#network 0.0.0.0 255.255.255.255 area 0
-
R0(config)#ip route 172.16.0.0 255.255.255.0 10.0.0.1

--- R1 ---
R1(config)#router ospf 1
R1(config-router)#network 0.0.0.0 255.255.255.255 area 0
--
R1(config)#ip route 172.16.0.0 255.255.255.0 10.0.1.1

--- R2 ---
R2(config)#router ospf 1
R2(config-router)#network 0.0.0.0 255.255.255.255 area 0
--
R2(config)#ip route 192.168.0.0 255.255.255.0 10.0.2.1
```

Результат прохождения трафика через тоннели GRE:


![Image alt](https://github.com/mezhibo/ni0803/blob/b005f8796a4a7c0b279b2109ed40e35c6b4bbc62/IMG/3.jpg)


[PKT-файл](https://github.com/mezhibo/ni0803/blob/b005f8796a4a7c0b279b2109ed40e35c6b4bbc62/IMG/0803-04-01_ospf.pkt)
