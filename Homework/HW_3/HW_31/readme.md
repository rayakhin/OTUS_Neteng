<h1>Развертывание коммутируемой сети с резервными каналами</h1>

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_3/HW_31/TOPO_31.PNG)

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_3/HW_31/IP_TABLE_31.PNG)

<h3>Цели</h3>
 1. Настройка базовых параметров коммутатора</br>  
 2. Настройка PAgP </br>  
 3. Настройка LACP </br>
 
 
<h3>1.Настройка основных параметров коммутатора </h3>
Пример настройки для коммутатора S3:

```
conf t
hostname S3    #Присваиваем имя устройству в соответствии с топологией.    

enable secret class  #Назначаем class в качестве зашифрованного пароля доступа к привилегированному режиму.

no ip domain-lookup #Отключаем поиск DNS.
         
interface Vlan99   
 ip address 192.168.99.13 255.255.255.0 #Задаем IP-адрес, указанный в таблице адресации для VLAN 99.
 no shutdown

banner motd ^C!!!!!!! Unauthorized access to this device is prohibited !!!!!!!^C #Настроим баннерное сообщение дня (MOTD) для
                                                                                 #предупреждения пользователей о запрете
                                                                                 #несанкционированного доступа.
																				 
int range e0/0 - 3  # Отключаем все порты коммутатора, кроме портов, подключенных к компьютерам.
   shutdown
int range e1/1 - 3
  shutdown
int e1/0
 description  Link to PC-A
 switchport access vlan 10
 switchport mode access
 
 
Vlan 99 #Настроим сеть VLAN 99 и присвоим имя Management.
name Management
Vlan10 #Настроим сеть VLAN 10 и присвоим имя Staff.
name Staff   
  

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

Аналогично настраиваем остальные коммутаторы S1 и S2. Также назначаем адресацию на PC в
соответствии с таблицей IP.

<h3>2.Настройка PAgP </h3>

<h4>Настроим PAgP на S1 и S3</h4>

Для создания канала между S1 и S3 настройте порты на S1 с использованием рекомендуемого режима (desirable), а порты на S3 — с использованием автоматического режима (auto). Включите порты после настройки режимов PAgP.

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_3/HW_31/S1_PAgP.PNG)

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_3/HW_31/S3_PAgP.PNG)

<h4>Проверка конфигурации на портах.</h4> 
В настоящее время интерфейсы e0/2, e0/3 и Po1 (Port-channel1) на коммутаторах S1 и S3 находятся в режиме доступе, а режим управления установлен на динамический автоматический режим (dynamic auto). Проверяем конфигурацию с помощью соответствующих команд show run interface идентификатор-интерфейса и show interfaces идентификатор-интерфейса switchport. Для интерфейса e0/3 на S1 отображаются следующие выходные данные конфигурации:

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_3/HW_31/S1_int_config.PNG)


<h4>Убеждаемся, что порты объединены.</h4>

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_3/HW_31/S1_Sum.PNG)

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_3/HW_31/S3_Sum.PNG)


В данном случае флаг SU говорит о том, что  агрегирование второго уровня выполнено и  этот интерфейс используется. А флаг P на портах указывает, что интерфейсы  собраны в port-channel.

<h4>Настройка транковых портов.</h4>
После агрегирования портов команды, применённые на интерфейсе Port Channel, влияют на все объединённые в группу каналы. Вручную настраиваем порты Po1 на S1 и S3 в качестве транковых и назначьте их сети native VLAN 99.

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_3/HW_31/Trunk_S1.PNG)

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_3/HW_31/Trunk_S3.PNG)

<h4>Проверяем,что порты настроены в качестве транковых.</h4>

Выполняем команду show run interface идентификатор-интерфейса на S1 и S3.

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_3/HW_31/Check_Trunk_S1.PNG)

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_3/HW_31/Check_Trunk_S3.PNG)

Из вывода команда может судитьто, что настройки применяемые на Po1, также применились на физических интерфейсах.
Единственно что отсутствует на Po1, это команды касаемые агрегации видны только на физических портах. 

<h3>3.Настройка протокола LACP</h3> 

Протокол LACP является открытым протоколом агрегирования каналов, разработанным на базе стандарта IEEE. В части 3 необходимо выполняем настройку канала между S1 и S2 и канала между S2 и S3 с помощью протокола LACP. Кроме того, отдельные каналы необходимо настроить в качестве транковых, прежде чем они будут объединены в каналы EtherChannel.

<h4>Настройте LACP между S1 и S2.</h4>

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_3/HW_31/LACP_S1.PNG)

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_3/HW_31/LACP_S2.PNG)

Проверяем что порты собраны в Port-channel, начнем со стороны S2

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_3/HW_31/Fail_S2.PNG)


Но к сожалению Po2 не в работе, так как судя по флагам D на физических портах - они просто не подняты.
Исправим данное упущение, благо порты не подняты толькосо стороны S2.

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_3/HW_31/UP_S2.PNG)

Результат на лицо, физические порты поднялись и Po2 в работе о чем свидетельствуют флаги SU и P.

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_3/HW_31/LACP_Sum_S2.PNG)



<h4>Настраиваем LACP между S2 и S3.</h4>
Сначала настраиваем S2

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_3/HW_31/LACP_S2-3.PNG)

Затем S3

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_3/HW_31/LACP_S3-2.PNG)

Убеждаемся, что Po3 поднялся 

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_3/HW_31/Po3_S2.PNG)









