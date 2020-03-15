
<h1>Развертывание коммутируемой сети с резервными каналами</h1>

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_2/Topo_STP.PNG)

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_2/IP_Table.PNG)

<h3>Цели</h3>
 1. Создание сети и настройка основных параметров устройства</br>  
 2. Выбор корневого моста</br>  
 3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов</br>
 4. Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов</br>
 
<h3>1.Настройка основных параметров устройства и проверка связности</h3>

Пример  базовой настройки коммутатора: 

```
conf t
hostname S3    #Присваиваем имя устройству в соответствии с топологией.    

enable secret class  #Назначаем class в качестве зашифрованного пароля доступа к привилегированному режиму.

no ip domain-lookup #Отключаем поиск DNS.
         
interface Vlan1   
 ip address 192.168.1.3 255.255.255.0 #Задаем IP-адрес, указанный в таблице адресации для VLAN 1.
 no shutdown

banner motd ^C!!!!!!! Unauthorized access to this device is prohibited !!!!!!!^C #Настроим баннерное сообщение дня (MOTD) для
                                                                                 #предупреждения пользователей о запрете
                                                                                 #несанкционированного доступа.

line con 0            
 password cisco  #Назначаем cisco в качестве пароля для консоли и активируем вход для консоли.
 login
 logging synchronous #Настроим logging synchronous для консольного канала.

line vty 0 4  
 password cisco #Назначаем cisco в качестве пароля для VTY и активируем вход для VTY.
 login

end

copy running-config startup-config #Копируем текущую конфигурацию в файл загрузочной конфигурации.

```

Проверяем связность между коммутаторами

эхо-запрос от коммутатора S1 на коммутатор S2:
```
S1#ping 192.168.1.3                   

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/5/8 ms
```

эхо-запрос от коммутатора S1 на коммутатор S3:

```
S1#ping 192.168.1.2

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/4 ms
```

эхо-запрос от коммутатора S2 на коммутатор S3:

```
S2#ping 192.168.1.3                   

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/4 ms
```
Положительные пинги говорят, что связность есть. 

<h3>2.Определение корневого моста</h3>

Отключаем все порты на коммутаторах.
```
conf t
int range e0/0-3
 shutdown
``` 
Настроим подключенные порты в качестве транковых.
```
 conf t
  int range e0/0 - 3
  switchport trunk encapsulation dot1q
  switchport mode trunk
```
 

Включаем порты е0/1 и е0/3 на всех коммутаторах. 

```
conf t
 int e0/1
   no shut
 
 int e0/3
   no shut
```

Отображаем данные протокола spanning-tree.

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
             Aging Time 300

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
             Aging Time 300

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Root FWD 100       128.2    Shr 
Et0/3               Desg FWD 100       128.4    Shr 

S3#show spanning-tree 

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        4 (Ethernet0/3)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time 300

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Altn BLK 100       128.2    Shr 
Et0/3               Root FWD 100       128.4    Shr 
```
Схема топологии с STP. 
![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_2/Topo_STP2.PNG)
Таблица портов на S1:
| Eq  | Port | Role          | Sts                    | Comment (name)                  |
|-----|------|---------------|------------------------|---------------------------------|
| S1  | e0/0 | Not Used      | Admin Shutdown         | to S2-e0/0                      |
| S1  | e0/1 | Designated    | Forwarding             | to S2-e0/1                      |
| S1  | e0/2 | Not Used      | Admin Shutdown         | to S3-e0/2                      |
| S1  | e0/3 | Designated    | Forwarding             | to S3-e0/3                      |


Таблица портов на S2:
| Eq  | Port | Role          | Sts                    | Comment (name)                  |
|-----|------|---------------|------------------------|---------------------------------|
| S2  | e0/0 | Not Used      | Admin Shutdown         | to S1-e0/0                      |
| S2  | e0/1 | Root          | Forwarding             | to S1-e0/1                      |
| S2  | e0/2 | Not Used      | Admin Shutdown         | to S3-e0/0                      |
| S2  | e0/3 | Designated    | Forwarding             | to S3-e0/1                      |


Таблица портов на S3:
| Eq  | Port | Role          | Sts                    | Comment (name)                  |
|-----|------|---------------|------------------------|---------------------------------|
| S3  | e0/0 | Not Used      | Admin Shutdown         | to S3-e0/2                      |
| S3  | e0/1 | Alternate     | Blocking               | to S3-e0/3                      |
| S3  | e0/2 | Not Used      | Admin Shutdown         | to S1-e0/2                      |
| S3  | e0/3 | Root          | Forwarding             | to S1-e0/3                      |

