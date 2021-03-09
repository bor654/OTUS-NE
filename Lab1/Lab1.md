# Configure Router-on-a-Stick Inter-VLAN Routing


## Задание:
1. Построить сеть и выполнить базовую настройку
2. Создать VLAN и назначить на порты коммутатора
3. Настроить 802.1q транк между коммутаторами
4. Настроить машрутизацию между VLAN на роутере
5. Проверить связность между VLAN 

### Схема
![Scheme](https://user-images.githubusercontent.com/80280218/110362348-3b918c00-8052-11eb-9711-264b50c26cc1.PNG)

### Адресация

| Устройство | Interface | IP Address   | Subnet Mask   | Default  Gateway |
| ---------- | --------- | ------------ | ------------- | ---------------- |
| R1         | G0/0/1.3  | 192.168.3.1  | 255.255.255.0 | N/A              |
| R1         | G0/0/1.4  | 192.168.4.1  | 255.255.255.0 | N/A              |
| R1         | G0/0/1.8  | N/A          | N/A           | N/A              |
| S1         | VLAN 3    | 192.168.3.11 | 255.255.255.0 | 192.168.3.1      |
| S2         | VLAN 3    | 192.168.3.12 | 255.255.255.0 | 192.168.3.1      |
| PC-A       | NIC       | 192.168.3.3  | 255.255.255.0 | 192.168.3.1      |
| PC-B       | NIC       | 192.168.4.3  | 255.255.255.0 | 192.168.4.1      |

### Таблица VLAN

| VLAN | Имя        | Интерфейсы                                                  |
| ---- | ---------- | ----------------------------------------------------------- |
| 3    | Management | S1: VLAN 3  S2: VLAN 3  S1: F0/6                            |
| 4    | Operations | S2: F0/18                                                   |
| 7    | ParkingLot | S1: F0/2-4, F0/7-24, G0/1-2   S2: F0/2-17, F0/19-24, G0/1-2 |
| 8    | Native     | N/A                                                         |

## Выполнение

#### 1. Построить сеть и выполнить базовую настройку

<details>
<summary>Выполнена базовая настройка роутера.</summary>

```
Router>en
Router#configure terminal 
Router(config)#hostname R1
R1(config)#no ip domain-lookup 
R1(config)#enable secret class
R1(config)#line console 0
R1(config-line)#password admin
R1(config-line)#login
R1(config-line)#password cisco
R1(config-line)#login
R1(config)#line vty 0 4
R1(config-line)#password cisco
R1(config-line)#login
R1(config)#service password-encryption 
R1(config)# banner login *Authorized access only.*
R1#clock set 22:26:00 08 March 2021
R1#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]
```
   
   </details>

<details>
   <summary>Выполнена базовая настройка коммутаторов.</summary>
   
   ```
На примере S1:
   
Switch>en
Switch#conf t
Switch(config)#hostname S1
S1(config)#no ip domain-lookup 
S1(config)#enable password class
S1(config)#line vty 0 4
S1(config-line)#password cisco
S1(config-line)#login
S1(config)#line console 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config)#service password-encryption 
S1(config)#banner motd *Authorized access only.*
S1#clock set 22:44:00 08 March 2021
S1#copy running-config startup-config
   ```
   
   </details>
   
   Выполнена настройка IP адресов на ПК.

#### 2. Создать VLAN и назначить на порты коммутатора
<details>
<summary> S1: </summary>
   
   ```
   S1#show vlan brief 

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/5
3    mgmt                             active    Fa0/6
4    Operations                       active
7    ParkingLot                       active    Fa0/2, Fa0/3, Fa0/4, Fa0/7
                                                Fa0/8, Fa0/9, Fa0/10, Fa0/11
                                                Fa0/12, Fa0/13, Fa0/14, Fa0/15
                                                Fa0/16, Fa0/17, Fa0/18, Fa0/19
                                                Fa0/20, Fa0/21, Fa0/22, Fa0/23
                                                Fa0/24, Gig0/1, Gig0/2
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active
   ```
   </details>
   
   <details>
<summary> S2: </summary>
   
   ```
S2(config)#do sh vlan br

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/19, Fa0/20, Fa0/21
                                                Fa0/22, Fa0/23, Fa0/24
3    mgmt                             active    
4    Operations                       active    Fa0/18
7    ParkingLot                       active    Fa0/2, Fa0/3, Fa0/4, Fa0/5
                                                Fa0/6, Fa0/7, Fa0/8, Fa0/9
                                                Fa0/10, Fa0/11, Fa0/12, Fa0/13
                                                Fa0/14, Fa0/15, Fa0/16, Fa0/17
                                                Gig0/1, Gig0/2
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active 
   ```

   </details>
   
#### 3. Настроить 802.1q транк между коммутаторами
 <details>
<summary> S1: </summary>
   
   ```
S1#sh run | sec 0/1
interface FastEthernet0/1
 switchport trunk native vlan 8
 switchport trunk allowed vlan 3-4,8
 switchport mode trunk
S1#sh run | sec 0/5
interface FastEthernet0/5
 switchport trunk native vlan 8
 switchport trunk allowed vlan 3-4,8
 switchport mode trunk
   ```
   
   </details>
   
   <details>
<summary> S2: </summary>
   
   ```
   S2# sh run | sec 0/1
interface FastEthernet0/1
 switchport trunk native vlan 8
 switchport trunk allowed vlan 3-4,8
 switchport mode trunk
 ```
   
   </details>
   
#### 4. Настроить машрутизацию между VLAN на роутере
   
     <details>
<summary> R1: </summary>
   
   ```
   R1#show run | sec 0/0/1
interface GigabitEthernet0/0/1
 no ip address
 duplex auto
 speed auto
interface GigabitEthernet0/0/1.3
 description to S1 VLAN 3
 encapsulation dot1Q 3
 ip address 192.168.3.1 255.255.255.0
interface GigabitEthernet0/0/1.4
 description to S1 VLAN 4
 encapsulation dot1Q 4
 ip address 192.168.4.1 255.255.255.0
interface GigabitEthernet0/0/1.8
 description to S1 VLAN 8
 encapsulation dot1Q 8
 no ip address
 
 R1#sh ip int br
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0/0   unassigned      YES unset  administratively down down 
GigabitEthernet0/0/1   unassigned      YES unset  up                    up 
GigabitEthernet0/0/1.3 192.168.3.1     YES manual up                    up 
GigabitEthernet0/0/1.4 192.168.4.1     YES manual up                    up 
GigabitEthernet0/0/1.8 unassigned      YES unset  up                    up 
Vlan1                  unassigned      YES unset  administratively down down
 ```
   
   </details>
   
#### 5. Проверить связность между VLAN

   Пинг с PC-A до шлюза
   
   ![изображение](https://user-images.githubusercontent.com/80280218/110523187-d22d7e00-8122-11eb-8611-933f38d2ea37.png)
   
   Пинг с PC-A до PC-B
   
   ![изображение](https://user-images.githubusercontent.com/80280218/110522546-11a79a80-8122-11eb-8a78-d4c553f5ff83.png)
   
   Пинг с PC-A до S2
   
   ![изображение](https://user-images.githubusercontent.com/80280218/110523248-e40f2100-8122-11eb-9fb1-a2727519e113.png)

   Пинг с PC-B до шлюза
   
   ![изображение](https://user-images.githubusercontent.com/80280218/110523616-6992d100-8123-11eb-8159-4448996640d0.png)

   Пинг с PC-B до PC-A
   
   ![изображение](https://user-images.githubusercontent.com/80280218/110522610-1ff5b680-8122-11eb-8063-38930ce4166f.png)
   
   Пинг с PC-B до S2
   
   ![изображение](https://user-images.githubusercontent.com/80280218/110523861-bb3b5b80-8123-11eb-875f-69b480afd2dd.png)


