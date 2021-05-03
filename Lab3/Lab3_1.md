# Настройка DCHPv4


<details> <summary>Схема</summary>
  
![изображение](https://user-images.githubusercontent.com/80280218/116880159-58010d80-ac2a-11eb-8fc2-fb5dee483897.png)
  
</details>

<details> <summary>Таблица адресации и VLAN</summary>
  
Подсеть A: 192.168.1.0/26 255.255.255.192 62 хоста  
Подсеть B: 192.168.1.64/27 255.255.255.224 30 хостов  
Подсеть C: 192.168.1.96/28 255.255.255.240 14 хостов  

| Device | Interface | IP Address   | Subnet Mask     | Default Gateway |
| ------ | --------- | ------------ | --------------- | --------------- |
| R1     | E0/0      | 10.0.0.1     | 255.255.255.252 | N/A             |
| R1     | E0/1      | N/A          | N/A             | N/A             |
| R1     | E0/1.100  | 192.168.1.1  | 255.255.255.192 | N/A             |
| R1     | E0/1.200  | 192.168.1.65 | 255.255.255.224 | N/A             |
| R1     | E0/1.1000 |              |                 | N/A             |
| R2     | E0/0      | 10.0.0.2     | 255.255.255.252 | N/A             |
| R2     | E0/1      | 192.168.1.97 | 255.255.255.240 | N/A             |
| S1     | VLAN 200  | 192.168.1.66 | 255.255.255.224 | 192.168.1.65    |
| S2     | VLAN 1    | 192.168.1.98 | 255.255.255.240 | 192.168.1.97    |
| PC-A   | NIC       | DHCP         | DHCP            | DHCP            |
| PC-B   | NIC       | DHCP         | DHCP            | DHCP            |

| VLAN | Name        | Interface Assigned |
| ---- | ----------- | ------------------ |
| 1    | N/A         | S2: E0/1           |
| 100  | Clients     | S1: E0/1           |
| 200  | Management  | S1: VLAN 200       |
| 999  | Parking_Lot | S1: E0/2-3         |
| 1000 | Native      | N/A                |
  
</details>


## 1. Подготовка и базовая конфигурация


<details> <summary>Настроенный R1</summary>

```
R1#show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                10.0.0.1        YES manual up                    up
Ethernet0/1                unassigned      YES unset  up                    up
Ethernet0/1.100            192.168.1.1     YES manual up                    up
Ethernet0/1.200            192.168.1.65    YES manual up                    up
Ethernet0/2                unassigned      YES unset  administratively down down
Ethernet0/3                unassigned      YES unset  administratively down down
R1#ping 10.0.0.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

</details>

<details> <summary>Настроенный R2</summary>
  
```
R2#sh ip int br
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                10.0.0.2        YES manual up                    up
Ethernet0/1                192.168.1.97    YES manual up                    up
Ethernet0/2                unassigned      YES unset  administratively down down
Ethernet0/3                unassigned      YES unset  administratively down down
R2#ping 10.0.0.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```
</details>  


<details> <summary>Настроенный S1</summary>
  
```
S1#show vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active
100  Clients                          active    Et0/1
200  Management                       active
999  Parking Lot                      active    Et0/2, Et0/3
1000 Native                           active
...
S1#show ip int br
Interface              IP-Address      OK? Method Status                Protocol
Ethernet0/0            unassigned      YES unset  up                    up
Ethernet0/1            unassigned      YES unset  up                    up
Ethernet0/2            unassigned      YES unset  up                    up
Ethernet0/3            unassigned      YES unset  up                    up
Vlan200                192.168.1.66    YES manual up                    up
S1#ping 192.168.1.65
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.65, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

</details>

<details> <summary>Настроенный S2</summary>

```
S2#show vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/0, Et0/1, Et0/2, Et0/3

S2#show ip int br
Interface              IP-Address      OK? Method Status                Protocol
Ethernet0/0            unassigned      YES unset  up                    up
Ethernet0/1            unassigned      YES unset  up                    up
Ethernet0/2            unassigned      YES unset  up                    up
Ethernet0/3            unassigned      YES unset  up                    up
Vlan1                  192.168.1.98    YES manual up                    up

S2#ping 192.168.1.97
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.97, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
show 
```

</details>

