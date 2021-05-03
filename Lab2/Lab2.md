# Развертывание коммутируемой сети с резервными каналами

<details> <summary>Топология</summary>

![Scheme](https://user-images.githubusercontent.com/80280218/113770437-cbe2ef80-972a-11eb-9f04-b88ee83e59ef.PNG)

</details>


<details> <summary>Таблица адресации</summary>

| Устройство | Интерфейс | IP-адрес    | Маска подсети |
| ---------- | --------- | ----------- | ------------- |
| S1         | VLAN 1    | 192.168.1.1 | 255.255.255.0 |
| S2         | VLAN 1    | 192.168.1.2 | 255.255.255.0 |
| S3         | VLAN 1    | 192.168.1.3 | 255.255.255.0 |

</details>

# 1. Создание сети и настройка основных параметров устройства

<details> <summary>ping S1->S2</summary>

S1#ping 192.168.1.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/1/1 ms

</details>

<details> <summary>ping S1->S3</summary>

S1#ping 192.168.1.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/1/1 ms

</details>

<details> <summary>ping S2->S3</summary>

S3#ping 192.168.1.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/1/1 ms


</details>


# 2.	Определение корневого моста


<details> <summary>Данные протокола spanning-tree</summary>
  


```


S1#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.1000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Desg FWD 100       128.2    Shr
Et0/3               Desg FWD 100       128.4    Shr


S2#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.2000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Root FWD 100       128.2    Shr
Et0/3               Desg FWD 100       128.4    Shr


S3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        4 (Ethernet0/3)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Altn BLK 100       128.2    Shr Peer(STP)
Et0/3               Root FWD 100       128.4    Shr Peer(STP)


```

</details>


<details> <summary>Заполненная схема</summary>


![изображение](https://user-images.githubusercontent.com/80280218/116862348-4b21f100-ac0d-11eb-818e-4e5491498836.png)


</details>


<details> <summary>Ответы на вопросы</summary>


Какой коммутатор является корневым мостом? **S1**

Почему этот коммутатор был выбран протоколом spanning-tree в качестве корневого моста? **Из-за наименьшего значения BID(наименьший MAC-адрес).**

Какие порты на коммутаторе являются корневыми портами? **S2: E0/1 S3: E0/3**

Какие порты на коммутаторе являются назначенными портами? **S1: E0/1, E0/3 S2: E0/3**

Какой порт отображается в качестве альтернативного и в настоящее время заблокирован? **S3: E0/1**

Почему протокол spanning-tree выбрал этот порт в качестве невыделенного (заблокированного) порта? **Наименьший BID у S2, поэтому порт на S2 выбран назначенным, а порт на S3 альтернативным.**



</details>


# 3.	Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов

Заблокированный порт S3 E0/1.

Изменение стоимости порта S3 E0/3:

```
S3(config)#interface Ethernet 0/3
S3(config-if)#spanning-tree cost 18
```


<details> <summary>Изменившееся дерево</summary>
  
```
S2#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.2000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  15  sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Root FWD 100       128.2    Shr
Et0/3               Altn BLK 100       128.4    Shr


S3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        18
             Port        4 (Ethernet0/3)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Desg BLK 100       128.2    Shr Peer(STP)
Et0/3               Root FWD 18        128.4    Shr Peer(STP)
```
  
</details>  


# 4.  Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов

<details> <summary>Данные протокола spanning-tree после включения всех портов</summary>
  


```
S1#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.1000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr
Et0/1               Desg FWD 100       128.2    Shr
Et0/2               Desg FWD 100       128.3    Shr
Et0/3               Desg FWD 100       128.4    Shr


S2#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        1 (Ethernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.2000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100       128.1    Shr
Et0/1               Altn BLK 100       128.2    Shr
Et0/2               Desg FWD 100       128.3    Shr
Et0/3               Desg FWD 100       128.4    Shr


S3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        3 (Ethernet0/2)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Altn BLK 100       128.1    Shr Peer(STP)
Et0/1               Altn BLK 100       128.2    Shr Peer(STP)
Et0/2               Root FWD 100       128.3    Shr Peer(STP)
Et0/3               Altn BLK 100       128.4    Shr Peer(STP)
```
</details>

Какой порт выбран протоколом STP в качестве порта корневого моста на каждом коммутаторе некорневого моста? **S2: E0/0 S3: E0/2**

Почему протокол STP выбрал эти порты в качестве портов корневого моста на этих коммутаторах? **По наименьшему значению приоритета(само значение приоритета одинаковый, но номер порта меньше).**



# 5.  Вопросы для повторения

1.	Какое значение протокол STP использует первым после выбора корневого моста, чтобы определить выбор порта? **Стоимость порта**

2.	Если первое значение на двух портах одинаково, какое следующее значение будет использовать протокол STP при выборе порта? **BID**

3.	Если оба значения на двух портах равны, каким будет следующее значение, которое использует протокол STP при выборе порта? **Приоритет порта**


[S1 config](https://github.com/bor654/OTUS-NE/blob/9c49c4c566bb509e8365648f50d6a8ebbf15ef22/Lab2/S1_cfg)
[S2 config](https://github.com/bor654/OTUS-NE/blob/9c49c4c566bb509e8365648f50d6a8ebbf15ef22/Lab2/S2_cfg)
[S3 config](https://github.com/bor654/OTUS-NE/blob/9c49c4c566bb509e8365648f50d6a8ebbf15ef22/Lab2/S3_cfg)
