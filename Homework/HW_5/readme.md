
<h3>Настройка базового протокола OSPFv2 для одной области</h3>

![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_5/TOPO_OSPF.PNG)
![](https://github.com/rayakhin/OTUS_Neteng/blob/master/Homework/HW_5/IP_TABLE.PNG)

<h4>Задачи</h4> 

1. Создание сети и настройка основных параметров устройства<br>
2. Настройка и проверка маршрутизации OSPF<br> 
3. Изменение назначений идентификаторов маршрутизаторов<br> 
4. Настройка пассивных интерфейсов OSPF<br> 
5. Изменение метрик OSPF<br> 



<h4>Настройка и проверка маршрутизации OSPF</h4>

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

