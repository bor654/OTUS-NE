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

</details>

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
