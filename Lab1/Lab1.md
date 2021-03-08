# Configure Router-on-a-Stick Inter-VLAN Routing


## Задание:
1. Построить сеть и выполнить базовую настройку
2. Создать VLAN и назначить на порты коммутатора
3. Настроить 802.1q транк между коммутаторами
4. Настроить машрутизацию между VLAN на роутере
5. Проверить связность между VLAN 

### Схема

   ![Scheme](C:\Users\mbor6\Documents\Labs Cisco\Lab1\Scheme.PNG)

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

<details><summary>Выполнена базовая настройка роутера</summary>
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
```
</details>

