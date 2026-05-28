# Реализация маршрутизации между VLAN 
<img width="783" height="297" alt="image" src="https://github.com/user-attachments/assets/7260e6b9-2e02-422c-b006-8da53a190f86" />

## Таблица адресации

| Устройство | Интерфейс     | IP‑адрес      | Маска подсети | Шлюз по умолчанию |
|------------|---------------|---------------|---------------|-------------------|
| R1         | G0/0/1.10     | 192.168.10.1  | 255.255.255.0 | —                 |
| R1         | G0/0/1.20     | 192.168.20.1  | 255.255.255.0 | —                 |
| R1         | G0/0/1.30     | 192.168.30.1  | 255.255.255.0 | —                 |
| R1         | G0/0/1.1000   | —             | —             | —                 |
| S1         | VLAN 10       | 192.168.10.11 | 255.255.255.0 | 192.168.10.1      |
| S2         | VLAN 10       | 192.168.10.12 | 255.255.255.0 | 192.168.10.1      |
| PC‑A       | NIC           | 192.168.20.3  | 255.255.255.0 | 192.168.20.1      |
| PC‑B       | NIC           | 192.168.30.3  | 255.255.255.0 | 192.168.30.1      |

## Таблица VLAN

| VLAN | Имя         | Назначенные интерфейсы                        |
|------|-------------|-----------------------------------------------|
| 10   | Management  | S1: VLAN 10, S2: VLAN 10                     |
| 20   | Sales       | S1: F0/6                                      |
| 30   | Operations  | S2: F0/18                                     |
| 999  | Parking_Lot | S1: F0/2‑4, F0/7‑24, G0/1‑2; S2: F0/2‑17, F0/19‑24, G0/1‑2 |
| 1000 | Native      | —                                             |

---

## Необходимые ресурсы
- 1 маршрутизатор Cisco 4221 (или аналогичный)
- 2 коммутатора Cisco 2960
- 2 ПК с Windows (программа эмуляции терминала)
- Консольные кабели, кабели Ethernet (прямые)

---

## Часть 1. Создание сети и настройка основных параметров устройств

### 1.1 Базовая настройка маршрутизатора R1

```cisco
Router> enable
Router# configure terminal
Router(config)# hostname R1
R1(config)# no ip domain-lookup
R1(config)# enable secret class
R1(config)# line console 0
R1(config-line)# password cisco
R1(config-line)# login
R1(config-line)# logging synchronous
R1(config-line)# exec-timeout 5 0
R1(config-line)# exit
R1(config)# line vty 0 4
R1(config-line)# password cisco
R1(config-line)# login
R1(config-line)# exec-timeout 5 0
R1(config-line)# exit
R1(config)# service password-encryption
R1(config)# banner motd ^CGo OUT!^C
R1(config)# clock timezone MSK 3
R1(config)# end
R1#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]
```
### 1.2 Базовая настройка коммутатора S1
```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname S1
S1(config)# no ip domain-lookup
S1(config)# enable secret class
S1(config)# line console 0
S1(config-line)# password cisco
S1(config-line)# login
S1(config-line)# logging synchronous
S1(config-line)# exec-timeout 5 0
S1(config-line)# exit
S1(config)# line vty 0 15
S1(config-line)# password cisco
S1(config-line)# login
S1(config-line)# exec-timeout 5 0
S1(config-line)# exit
S1(config)# service password-encryption
S1(config)# banner motd ^CGO OUT!^C
S1(config)# clock timezone UTC +3
S1(config)# end
S1#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]
```
### 1.3 Базовая настройка коммутатора S2 
```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname S2
S2(config)# no ip domain-lookup
S2(config)# enable secret class
S2(config)# line console 0
S2(config-line)# password cisco
S2(config-line)# login
S2(config-line)# logging synchronous
S2(config-line)# exec-timeout 5 0
S2(config-line)# exit
S2(config)# line vty 0 15
S2(config-line)# password cisco
S2(config-line)# login
S2(config-line)# exec-timeout 5 0
S2(config-line)# exit
S2(config)# service password-encryption
S2(config)# banner motd ^CUnauthorized access prohibited^C
S2(config)# clock timezone UTC +3
S2(config)# end
S2#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]
```
### 1.4 Настройка ПК
#### PC-A:  
```cmd
C:\>ipconfig  
FastEthernet0 Connection:(default port)  
  
   Connection-specific DNS Suffix..:   
   Link-local IPv6 Address.........: FE80::202:16FF:FE95:C08D  
   IPv6 Address....................: ::  
   IPv4 Address....................: 192.168.20.3  
   Subnet Mask.....................: 255.255.255.0  
   Default Gateway.................: ::  
                                     192.168.20.1  
  ```
#### PC-B:  
```cmd
  C:\>ipconfig  

FastEthernet0 Connection:(default port)  
  
   Connection-specific DNS Suffix..:   
   Link-local IPv6 Address.........: FE80::2D0:D3FF:FE3A:107E  
   IPv6 Address....................: ::  
   IPv4 Address....................: 192.168.30.3  
   Subnet Mask.....................: 255.255.255.0  
   Default Gateway.................: ::  
                                     192.168.30.1  
  ```
## Часть 2. Создание сетей VLAN и назначение портов коммутатора

### 2.1 Создание VLAN на S1
```cisco
S1> enable
S1# configure terminal
S1(config)# vlan 10
S1(config-vlan)# name Management
S1(config-vlan)# exit
S1(config)# vlan 20
S1(config-vlan)# name Sales
S1(config-vlan)# exit
S1(config)# vlan 30
S1(config-vlan)# name Operations
S1(config-vlan)# exit
S1(config)# vlan 999
S1(config-vlan)# name Parking_Lot
S1(config-vlan)# exit
S1(config)# vlan 1000
S1(config-vlan)# name Native
S1(config-vlan)# exit
```
### 2.2 Создание VLAN на S2 

#### Создаем VLAN аналогично S1  
```cisco
S2> enable
S2# configure terminal
S2(config)# vlan 10
S2(config-vlan)# name Management
S2(config-vlan)# exit
S2(config)# vlan 20
S2(config-vlan)# name Sales
S2(config-vlan)# exit
S2(config)# vlan 30
S2(config-vlan)# name Operations
S2(config-vlan)# exit
S2(config)# vlan 999
S2(config-vlan)# name Parking_Lot
S2(config-vlan)# exit
S2(config)# vlan 1000
S2(config-vlan)# name Native
S2(config-vlan)# exit
```
### 2.3 Настройка интерфейса управления и шлюза по умолчанию

#### S1
```cisco
S1(config)# interface vlan 10
S1(config-if)#
%LINK-5-CHANGED: Interface Vlan10, changed state to up
S1(config-if)# ip address 192.168.10.11 255.255.255.0
S1(config-if)# no shutdown
S1(config-if)# exit
S1(config)# ip default-gateway 192.168.10.1
```

#### S2
```cisco
S2(config)# interface vlan 10
S2(config-if)#
%LINK-5-CHANGED: Interface Vlan10, changed state to up
S2(config-if)# ip address 192.168.10.12 255.255.255.0
S2(config-if)# no shutdown
S2(config-if)# exit
S2(config)# ip default-gateway 192.168.10.1
```
### 2.4 Деактивация неиспользуемых портов
#### S1
```cisco
S1(config)# interface range f0/2-4, f0/7-24, g0/1-2
S1(config-if-range)# switchport mode access
S1(config-if-range)# switchport access vlan 999
S1(config-if-range)# shutdown
%LINK-5-CHANGED: Interface FastEthernet0/2, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/3, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/4, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/7, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/8, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/9, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/10, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/11, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/12, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/13, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/14, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/15, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/16, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/17, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/18, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/19, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/20, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/21, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/22, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/23, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/24, changed state to administratively down

%LINK-5-CHANGED: Interface GigabitEthernet0/1, changed state to administratively down

%LINK-5-CHANGED: Interface GigabitEthernet0/2, changed state to administratively down
S1(config-if-range)# exit
```
#### S2
```cisco
S2(config)# interface range f0/2-17, f0/19-24, g0/1-2
S2(config-if-range)# switchport mode access
S2(config-if-range)# switchport access vlan 999
S2(config-if-range)# shutdown
%LINK-5-CHANGED: Interface FastEthernet0/2, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/3, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/4, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/5, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/6, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/7, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/8, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/9, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/10, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/11, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/12, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/13, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/14, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/15, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/16, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/17, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/19, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/20, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/21, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/22, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/23, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/24, changed state to administratively down

%LINK-5-CHANGED: Interface GigabitEthernet0/1, changed state to administratively down

%LINK-5-CHANGED: Interface GigabitEthernet0/2, changed state to administratively down
S2(config-if-range)# exit
```
### 2.5 Назначение используемых портов в соответствующие VLAN
#### S1 (порт F0/6 в VLAN 20):
```cisco
S1(config)# interface f0/6
S1(config-if)# switchport mode access
S1(config-if)# switchport access vlan 20
S1(config-if)# no shutdown
S1(config-if)# exit
```
#### S2 (порт F0/18 в VLAN 30):
```cisco
S2(config)# interface f0/18
S2(config-if)# switchport mode access
S2(config-if)# switchport access vlan 30
S2(config-if)# no shutdown
S2(config-if)# exit
```
### 2.6 Проверка
#### S1:
```cisco
S1#show vlan brief 

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/5
10   Management                       active    
20   Sales                            active    Fa0/6
30   Operations                       active    
999  Parking_Lot                      active    Fa0/2, Fa0/3, Fa0/4, Fa0/7
                                                Fa0/8, Fa0/9, Fa0/10, Fa0/11
                                                Fa0/12, Fa0/13, Fa0/14, Fa0/15
                                                Fa0/16, Fa0/17, Fa0/18, Fa0/19
                                                Fa0/20, Fa0/21, Fa0/22, Fa0/23
                                                Fa0/24, Gig0/1, Gig0/2
1000 Native                           active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    
S1#
S1#show interfaces status 
Port      Name               Status       Vlan       Duplex  Speed Type
Fa0/1                        connected    1          a-full  a-100 10/100BaseTX
Fa0/2                        disabled 999        auto    auto  10/100BaseTX
Fa0/3                        disabled 999        auto    auto  10/100BaseTX
Fa0/4                        disabled 999        auto    auto  10/100BaseTX
Fa0/5                        notconnect   1          auto    auto  10/100BaseTX
Fa0/6                        connected    20         a-full  a-100 10/100BaseTX
Fa0/7                        disabled 999        auto    auto  10/100BaseTX
Fa0/8                        disabled 999        auto    auto  10/100BaseTX
Fa0/9                        disabled 999        auto    auto  10/100BaseTX
Fa0/10                       disabled 999        auto    auto  10/100BaseTX
Fa0/11                       disabled 999        auto    auto  10/100BaseTX
Fa0/12                       disabled 999        auto    auto  10/100BaseTX
Fa0/13                       disabled 999        auto    auto  10/100BaseTX
Fa0/14                       disabled 999        auto    auto  10/100BaseTX
Fa0/15                       disabled 999        auto    auto  10/100BaseTX
Fa0/16                       disabled 999        auto    auto  10/100BaseTX
Fa0/17                       disabled 999        auto    auto  10/100BaseTX
Fa0/18                       disabled 999        auto    auto  10/100BaseTX
Fa0/19                       disabled 999        auto    auto  10/100BaseTX
Fa0/20                       disabled 999        auto    auto  10/100BaseTX
Fa0/21                       disabled 999        auto    auto  10/100BaseTX
Fa0/22                       disabled 999        auto    auto  10/100BaseTX
Fa0/23                       disabled 999        auto    auto  10/100BaseTX
Fa0/24                       disabled 999        auto    auto  10/100BaseTX
Gig0/1                       disabled 999        auto    auto  10/100/1000BaseTX
Gig0/2                       disabled 999        auto    auto  10/100/1000BaseTX
```
## Часть 3. Настройка транка 802.1Q между коммутаторами

### 3.1 Настройка транка на интерфейсе F0/1 
#### S1
```cisco
S1(config)# interface f0/1
S1(config-if)# switchport mode trunk
S1(config-if)# switchport trunk native vlan 1000
S1(config-if)# switchport trunk allowed vlan 10,20,30,1000
S1(config-if)# no shutdown
S1(config-if)# exit
S1(config)#
%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on FastEthernet0/1 (1000), with S2 FastEthernet0/1 (1).
```
#### S2:
```cisco
S2(config)# interface f0/1
S2(config-if)# switchport mode trunk
S2(config-if)# switchport trunk native vlan 1000
S2(config-if)# switchport trunk allowed vlan 10,20,30,1000
S2(config-if)# no shutdown
S2(config-if)# exit
```
### 3.2 Настройка транка на S1 F0/5
```cisco
S1(config)# interface f0/5
S1(config-if)# switchport mode trunk
S1(config-if)# switchport trunk native vlan 1000
S1(config-if)# switchport trunk allowed vlan 10,20,30,1000
S1(config-if)# no shutdown
S1(config-if)# exit
S1(config)# end
S1#
%SYS-5-CONFIG_I: Configured from console by console
S1# copy running-config startup-config
Destination filename [startup-config]? 
Building configuration...
[OK]
```
### 3.3 Проверка
```cisco
S1#show interfaces trunk 
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      1000

Port        Vlans allowed on trunk
Fa0/1       10,20,30,1000

Port        Vlans allowed and active in management domain
Fa0/1       10,20,30,1000

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       10,20,30,1000

S2#show interfaces trunk 
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      1000

Port        Vlans allowed on trunk
Fa0/1       10,20,30,1000

Port        Vlans allowed and active in management domain
Fa0/1       10,20,30,1000

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       10,20,30,1000
```
## Часть 4. Настройка маршрутизации между VLAN на R1
### 4.1 Активация физического интерфейса G0/0/1 и создание Subinterface
```cisco
R1> enable
R1# configure terminal
R1(config)# interface g0/0/1
R1(config-if)# no shutdown
R1(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1, changed state to up
R1(config-if)# exit
R1(config)# interface g0/0/1.10
R1(config-subif)#
%LINK-3-UPDOWN: Interface GigabitEthernet0/0/1.10, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.10, changed state to up
R1(config-subif)# encapsulation dot1Q 10
R1(config-subif)# ip address 192.168.10.1 255.255.255.0
R1(config-subif)# description Management
R1(config-subif)# exit

R1(config)# interface g0/0/1.20
R1(config-subif)#
%LINK-3-UPDOWN: Interface GigabitEthernet0/0/1.20, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.20, changed state to up
R1(config-subif)# encapsulation dot1Q 20
R1(config-subif)# ip address 192.168.20.1 255.255.255.0
R1(config-subif)# description Sales
R1(config-subif)# exit

R1(config)# interface g0/0/1.30
R1(config-subif)#
%LINK-3-UPDOWN: Interface GigabitEthernet0/0/1.30, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.30, changed state to up
R1(config-subif)# encapsulation dot1Q 30
R1(config-subif)# ip address 192.168.30.1 255.255.255.0
R1(config-subif)# description Operations
R1(config-subif)# exit

R1(config)# interface g0/0/1.1000
R1(config-subif)#
%LINK-3-UPDOWN: Interface GigabitEthernet0/0/1.1000, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.1000, changed state to up

R1(config-subif)# encapsulation dot1Q 1000 native
R1(config-subif)# description Native_VLAN
R1(config-subif)# exit
```
### 4.2 Проверка подинтерфейсов
```cisco
R1#show ip interface brief 
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0/0   unassigned      YES unset  administratively down down 
GigabitEthernet0/0/1   unassigned      YES unset  up                    up 
GigabitEthernet0/0/1.10192.168.10.1    YES manual up                    up 
GigabitEthernet0/0/1.20192.168.20.1    YES manual up                    up 
GigabitEthernet0/0/1.30192.168.30.1    YES manual up                    up 
GigabitEthernet0/0/1.1000unassigned      YES unset  up                    up 
GigabitEthernet0/0/2   unassigned      YES unset  administratively down down 
Vlan1                  unassigned      YES unset  administratively down down
R1#show ip route 
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is not set

     192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.10.0/24 is directly connected, GigabitEthernet0/0/1.10
L       192.168.10.1/32 is directly connected, GigabitEthernet0/0/1.10
     192.168.20.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.20.0/24 is directly connected, GigabitEthernet0/0/1.20
L       192.168.20.1/32 is directly connected, GigabitEthernet0/0/1.20
     192.168.30.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.30.0/24 is directly connected, GigabitEthernet0/0/1.30
L       192.168.30.1/32 is directly connected, GigabitEthernet0/0/1.30
```
## Часть 5. Проверка работы маршрутизации между VLAN
### 5.1 Тесты с PC‑A
```cmd
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

C:\>ping 192.168.30.3

Pinging 192.168.30.3 with 32 bytes of data:

Request timed out.
Reply from 192.168.30.3: bytes=32 time<1ms TTL=127
Reply from 192.168.30.3: bytes=32 time<1ms TTL=127
Reply from 192.168.30.3: bytes=32 time<1ms TTL=127

Ping statistics for 192.168.30.3:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

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

C:\>
```
### 5.2 Тест с PC‑B – трассировка до PC‑A
```cmd
C:\>tracert 192.168.20.3

Tracing route to 192.168.20.3 over a maximum of 30 hops: 

  1   0 ms      0 ms      0 ms      192.168.30.1
  2   0 ms      0 ms      0 ms      192.168.20.3

Trace complete.

C:\>
```
Пакет идёт с PC-B к шлюзу 192.168.30.1 на подинтерфейс R1, там маршрутизируется на подинтерфейс 192.168.20.1, а затем к хосту 192.168.20.3. В таблице мы видим лишь адрес шлюза (192.168.30.1) и затем сразу конечный хост, т.к. R1 выполняет маршрутизацию напрямую между подсетями без дополнительных хопов.  

## Вопросы
#### Что произойдет, если G0/0/1 на R1 будет отключен?
 Будет нарушена маршрутизация между VLAN. S1 и S2 не потеряются, т.к. продолжат общаться между собой по транку (VLAN 10,20,30), но пакеты не смогут покинуть локальную сеть, так как шлюз станет недоступен. PC‑A и PC‑B потеряют связь друг с другом и с управлениями SVI коммутаторов. 
