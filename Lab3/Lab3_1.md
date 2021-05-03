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

## 2. Настройка DHCP сервера на R1

<details> <summary>Конфиг DHCP R1</summary>
  
```
R1(config)#do sh run | sec dhcp
ip dhcp excluded-address 192.168.1.1 192.168.1.5
ip dhcp excluded-address 192.168.1.97 192.168.1.98
ip dhcp pool Clients_Pool_1
 network 192.168.1.0 255.255.255.192
 domain-name ccna-lab.com
 default-router 192.168.1.1
 lease 2 12 30
ip dhcp pool R2_Client_LAN
 network 192.168.1.96 255.255.255.240
 domain-name ccna-lab.com
 default-router 192.168.1.97
 lease 2 12 30
 R1#show ip dhcp pool

Pool Clients_Pool_1 :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0
 Total addresses                : 62
 Leased addresses               : 0
 Pending event                  : none
 1 subnet is currently in the pool :
 Current index        IP address range                    Leased addresses
 192.168.1.1          192.168.1.1      - 192.168.1.62      0

Pool R2_Client_LAN :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0
 Total addresses                : 14
 Leased addresses               : 0
 Pending event                  : none
 1 subnet is currently in the pool :
 Current index        IP address range                    Leased addresses
 192.168.1.97         192.168.1.97     - 192.168.1.110     0

R1#show ip dhcp binding
Bindings from all pools not associated with VRF:
IP address          Client-ID/              Lease expiration        Type
                    Hardware address/
                    User name
R1#show ip dhcp server statistics
Memory usage         25176
Address pools        2
Database agents      0
Automatic bindings   0
Manual bindings      0
Expired bindings     0
Malformed messages   0
Secure arp entries   0
Message              Received
BOOTREQUEST          0
DHCPDISCOVER         0
DHCPREQUEST          0
DHCPDECLINE          0
DHCPRELEASE          0
DHCPINFORM           0
Message              Sent
BOOTREPLY            0
DHCPOFFER            0
DHCPACK              0
DHCPNAK              0

```

</details>

<details> <summary>Получение ip на PC-A</summary>

```
VPCS> dhcp -r
DDORA IP 192.168.1.6/26 GW 192.168.1.1
VPCS> show ip

NAME        : VPCS[1]
IP/MASK     : 192.168.1.6/26
GATEWAY     : 192.168.1.1
DNS         :
DHCP SERVER : 192.168.1.1
DHCP LEASE  : 217769, 217800/108900/190575
DOMAIN NAME : ccna-lab.com
MAC         : 00:50:79:66:68:05
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

VPCS> ping 192.168.1.1
84 bytes from 192.168.1.1 icmp_seq=1 ttl=255 time=0.334 ms
84 bytes from 192.168.1.1 icmp_seq=2 ttl=255 time=0.537 ms
84 bytes from 192.168.1.1 icmp_seq=3 ttl=255 time=0.540 ms
84 bytes from 192.168.1.1 icmp_seq=4 ttl=255 time=0.502 ms
84 bytes from 192.168.1.1 icmp_seq=5 ttl=255 time=0.551 ms
```

</details>


## 3. Настройка DHCP-relay на R2

<details> <summary>Конфиг DHCP R2</summary>

```
R2(config-if)#do show run int eth0/1
Building configuration...

Current configuration : 125 bytes
!
interface Ethernet0/1
 description link to S2
 ip address 192.168.1.97 255.255.255.240
 ip helper-address 192.168.1.1
end
```

</details>
  
<details> <summary>Получение ip на PC-B</summary>

```
VPCS> dhcp -r
DDORA IP 192.168.1.99/28 GW 192.168.1.97

VPCS> show ip

NAME        : VPCS[1]
IP/MASK     : 192.168.1.99/28
GATEWAY     : 192.168.1.97
DNS         :
DHCP SERVER : 10.0.0.1
DHCP LEASE  : 217788, 217800/108900/190575
DOMAIN NAME : ccna-lab.com
MAC         : 00:50:79:66:68:06
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

VPCS> ping 192.168.1.1
84 bytes from 192.168.1.1 icmp_seq=1 ttl=254 time=0.529 ms
84 bytes from 192.168.1.1 icmp_seq=2 ttl=254 time=0.608 ms
84 bytes from 192.168.1.1 icmp_seq=3 ttl=254 time=0.689 ms
84 bytes from 192.168.1.1 icmp_seq=4 ttl=254 time=0.632 ms
84 bytes from 192.168.1.1 icmp_seq=5 ttl=254 time=0.746 ms

```

</details>


<details> <summary>Логи с R1 и R2</summary>

```
R1#show ip dhcp binding
Bindings from all pools not associated with VRF:
IP address          Client-ID/              Lease expiration        Type
                    Hardware address/
                    User name
192.168.1.6         0100.5079.6668.05       May 06 2021 05:10 AM    Automatic
192.168.1.99        0100.5079.6668.06       May 06 2021 05:15 AM    Automatic
R1#show ip dhcp server statistics
Memory usage         42094
Address pools        2
Database agents      0
Automatic bindings   2
Manual bindings      0
Expired bindings     0
Malformed messages   0
Secure arp entries   0

Message              Received
BOOTREQUEST          0
DHCPDISCOVER         4
DHCPREQUEST          2
DHCPDECLINE          0
DHCPRELEASE          0
DHCPINFORM           0

Message              Sent
BOOTREPLY            0
DHCPOFFER            2
DHCPACK              2
DHCPNAK              0
```
***

```
R2#show ip dhcp server statistics
Memory usage         22565
Address pools        0
Database agents      0
Automatic bindings   0
Manual bindings      0
Expired bindings     0
Malformed messages   0
Secure arp entries   0

Message              Received
BOOTREQUEST          0
DHCPDISCOVER         0
DHCPREQUEST          0
DHCPDECLINE          0
DHCPRELEASE          0
DHCPINFORM           0

Message              Sent
BOOTREPLY            0
DHCPOFFER            0
DHCPACK              0

```

</details>


[S1 config](https://github.com/bor654/OTUS-NE/blob/57d5e38647ee2e5c5808aaf25cd74f29db47282c/Lab3/S1_cfg)  
[S2 config](https://github.com/bor654/OTUS-NE/blob/ecb964c663ea26b4eb9b079bccc7cd04860b17ef/Lab3/S2_cfg)  
[R1 config](https://github.com/bor654/OTUS-NE/blob/241cf164e2605e76206bae8efd291848daecf119/Lab3/R1_cfg)  
[R1 config](https://github.com/bor654/OTUS-NE/blob/18acab581ad653b96940945cf53b02372bc6b577/Lab3/R2_cfg)  
