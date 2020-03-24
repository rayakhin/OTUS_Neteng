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
В настоящее время интерфейсы e0/2, e0/3 и Po1 (Port-channel1) на коммутаторах S1 и S3 находятся в режиме доступе, а режим управления установлен на динамический автоматический режим (dynamic auto). Проверьте конфигурацию с помощью соответствующих команд show run interface идентификатор-интерфейса и show interfaces идентификатор-интерфейса switchport. Для интерфейса e0/3 на S1 отображаются следующие выходные данные конфигурации:
![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_3/HW_31/S1_int_config.PNG)
