
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



<h3>3.Наблюдение за процессом выбора протоколом STP порта,
исходя из стоимости портов</h3>

Определим коммутатор с заблокированным портом.
```
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

```
Изменим стоимость порта.

```
S3(config-if)#int e0/3                
S3(config-if)#spanning-tree cost 99   
S3(config-if)#end
```
Смотрим изменения протокола spanning-tree.
```
S3#show spanning-tree                

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        99
             Port        4 (Ethernet0/3)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time 15 

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Desg LRN 100       128.2    Shr 
Et0/3               Root FWD 99        128.4    Shr 

S2#sh  spanning-tree                 

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
Et0/3               Altn BLK 100       128.4    Shr 

```
Почему протокол spanning-tree заменяет ранее заблокированный порт на назначенный порт
и блокирует порт, который был назначенным портом на другом коммутаторе? 
Так как стоимость пути  стала более предпочительней.

Удаляем изменения стоимости порта.
```
S3(config-if)#int e0/3                
S3(config-if)#no spanning-tree cost 99   
S3(config-if)#end
```
<h3>4.Наблюдение за процессом выбора протоколом STP порта,
исходя из приоритета портов</h3>
Включаем порты e0/0 и e0/2 на всех коммутаторах.
```
 conf t
  int  e0/0
  no shut
  int  e0/2
  no shut
 end 
```




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
             Aging Time 300

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100       128.1    Shr 
Et0/1               Altn BLK 100       128.2    Shr 
Et0/2               Desg FWD 100       128.3    Shr 
Et0/3               Desg FWD 100       128.4    Shr 
S3#sh spanning-tree 

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        3 (Ethernet0/2)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time 300

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Altn BLK 100       128.1    Shr 
Et0/1               Altn BLK 100       128.2    Shr 
Et0/2               Root FWD 100       128.3    Shr 
Et0/3               Altn BLK 100       128.4    Shr 

```
Какой порт выбран протоколом STP в качестве порта корневого моста на каждом коммутаторе
некорневого моста?
На S2 в качестве корневого порта выбран e0/0, а на S3 это e0/2.

Почему протокол STP выбрал эти порты в качестве портов корневого моста на этих коммутаторах?
Порты включеные в корневой коммутатор:
```
S3#sh cdp nei | i S1
S1               Eth 0/2           167              R S   Linux Uni Eth 0/2
S1               Eth 0/3           155              R S   Linux Uni Eth 0/3
```
В данном случае отработал приоретет портов, то есть с меньшим значением станновиться корневым
``` 
SS3#sh spanning-tree | i  Et0/[23]
Et0/2               Root FWD 100       128.3    Shr 
Et0/3               Altn BLK 100       128.4    Shr 

```


