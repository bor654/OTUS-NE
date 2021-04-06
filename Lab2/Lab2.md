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
