
<h3>Настройка базового протокола OSPFv2 для одной области</h3>

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_5/TOPO_OSPF.PNG)
![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_5/IP_TABLE.PNG)

<h4>Задачи</h4> 

1. Создание сети и настройка основных параметров устройства<br>
2. Настройка и проверка маршрутизации OSPF<br> 
3. Изменение назначений идентификаторов маршрутизаторов<br> 
4. Настройка пассивных интерфейсов OSPF<br> 
5. Изменение метрик OSPF<br> 



<h4>2.Настройка и проверка маршрутизации OSPF</h4>

Настроим протокол OSPF на маршрутизаторе R1.

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_5/R1_NETS.PNG)

аналогично повторим данную процедуру для R2 и R3 
После на маршрутизаторе R1  появятся сообщения об отношениях смежности.

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_5/R1_ADJ.PNG)

Проверим информацию о соседних устройствах и маршрутизации OSPFкомандами show ip ospf neighbor и show ip route для примера на R1

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_5/R1_OSPF_SHOW.PNG)

Проверим параметры протокола OSPF использовав команду show ip protocols

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_5/OSPF_PROTO.PNG)

Командой show ip ospf проверим данные процесса OSPF

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_5/OSPF_PROC.PNG)

Для проверки параметров интерфейса OSPF используем  show ip ospf interface brief(для отображения сводки интерфейсов с поддержкой протокола OSPF.) 
и show ip ospf interface(для более детальной информации)

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_5/OSPF_INT1.PNG)

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_5/OSPF_INT2.PNG)

Теперь пропингуем для примера с PC-A, и видим что связность между хостами есть. Все работает.

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_5/PING.PNG)


<h4>3.Изменение назначенных идентификаторов маршрутизаторов </h4>

Идентификатор OSPF-маршрутизатора используется для уникальной идентификации домена маршрутизации OSPF. Маршрутизаторам компании Cisco идентификатор назначается одним из трех способов и в следующем порядке: 

-IP-адрес, настроенный с помощью команды OSPF router-id (при наличии)<br> 
-Наибольший IP-адрес любого из loopback-адресов маршрутизатора (при наличии)<br> 
-Наибольший активный IP-адрес любого из физических интерфейсов маршрутизатора<br> 

Поскольку ни на одном из трех маршрутизаторов не настроены ID маршрутизатора или интерфейсы loopback, ID каждого маршрутизатора определяется наивысшим IP-адресом любого активного интерфейса. 

В данной части  изменим назначение идентификатора OSPF-маршрутизатора с помощью loopback-адресов. Также мы воспользуемся командой router-id для смены идентификатора маршрутизатора.

Изменим идентификаторы маршрутизатора с помощью loopback-адресов. 

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_5/R1_LOOP.PNG)

Назначим IP-адреса loopback-интерфейсам 0 для маршрутизаторов R2 и R3. Используем IP-адрес 2.2.2.2/32 для R2 и 3.3.3.3/32 для R3.
Затем выполним команду reload на всех трех маршрутизаторах дя использования в качестве  идентификатора маршрутизатора  loopback-адреса.
После того как маршрутизатор завершит процесс перезагрузки, введем команду show ip protocols, чтобы просмотреть новый идентификатор маршрутизатора и show ip ospf neighbor, чтобы просмотреть изменения идентификаторов соседних маршрутизаторов.

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_5/R1_LOOP2.PNG)


Теперь изменим идентификатор с помощью команды router-id сначала R1

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_5/R1_ROUTER_ID.PNG)

Информационное сообщение говорит о том, что необходимо либо перезагрузить маршрутизатор, либо воспользоваться командой clear ip ospf process для вступления этого изменения в силу. Так что воспользуемся командой clear ip ospf process на всех трех маршрутизаторах. Введи 

Для маршрутизатор R2 настройте идентификатор 22.22.22.22, а для маршрутизатора R3 настройте идентификатор 33.33.33.33. Затем используйте команду clear ip ospf process для сброса процесса маршрутизации ospf. 

Введите команду show ip protocols, чтобы проверить, изменился ли идентификатор на маршрутизаторе и 
show ip ospf neighbor на маршрутизаторе R1, чтобы убедиться, что новые идентификаторы для маршрутизаторов R2 и R3 
содержатся в списке

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_5/R1_IP_PROTO.PNG)



