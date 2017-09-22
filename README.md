# Краткое руководство по первоначальной настройке OLT ELTEX LTP-8x.
## 1. Подключение и авторизация
К OLT с заводскими настройками можно подключиться двумя способами:
по TELNET/SSH и через последовательный serial port. Подключение через TELNET/SSH возможно на любом front-port. Для подключения через TELNET/SSH используются предустановленные настройки:

    Default IP : 192.168.1.2
    Default mask : 255.255.255.0
    Login: admin
    Password: password
    
Для подключения через com-port необходимо использовать следующие настройки:

    Скорость порта 115200 бит/с;
    Биты данных 8 бит.

Логин и пароль те же, что и для подключения через TELNET/SSH.

## 2. Настройка управления
Просмотр текущей конфигурации параметров управления производится командой __"show management"__.

    km46-gpon-sw# show management 
        Network:
            Hostname:                                      'km46-gpon-sw'
            Ipaddr:                                        172.16.68.113
            Netmask:                                       255.255.240.0
            Vlan management:                               904
            Gateway:                                       172.16.64.1
            Vlan prio:                                     7
            Dscp:                                          63
    km46-gpon-sw# 

Для настройки этих параметров необходимо перейти в режим конфигурирования:

    km46-gpon-sw# configure terminal 
    km46-gpon-sw(config)#management ip 172.16.68.113
    km46-gpon-sw(config)#management mask 255.255.240.0
    km46-gpon-sw(config)#management gateway 172.16.64.1
    km46-gpon-sw(config)#management vid 904

После чего необходимо убедиться в правильности настроек:

    km46-gpon-sw(config)# do show management 
        Network:
            Hostname:                                      'km46-gpon-sw'
            Ipaddr:                                        172.16.68.113
            Netmask:                                       255.255.240.0
            Vlan management:                               904
            Gateway:                                       172.16.64.1
            Vlan prio:                                     7
            Dscp:                                          63

Выйти из режима конфигурирования применить и сохранить настройки:
    
    km46-gpon-sw(config)#exit
    km46-gpon-sw#commit
    km46-gpon-sw#save

Для правильной работы управления необходимо создать и настроить нужный в VLAN в модуле SWITCH.

## 3. Добавление пользователей
Просмотр списка пользователей производится командой __show users__:

    km46-gpon-sw# show users 
        ##                Name      Privilege
         1                root           15
         2               admin           15
         3              remote           15

Добавление новых пользователей производится командой __user__ в режиме конфигурирования:

    km46-gpon-sw(config)# user vic

При создании пользователя ему назначается минимальный набор прав:

    km46-gpon-sw(config)# do show users 
        ##                Name      Privilege
         1                root           15
         2               admin           15
         3              remote           15
         4                 vic            0

Назначение пароля для пользователя:

    km46-gpon-sw(config)# user vic password newawesomepassword

Минимальная длина пароля - 8 символов, максимальная - 31.

Назначение привелегий.

    km46-gpon-sw(config)# user vic privilege 15
    km46-gpon-sw(config)# do show users
        ##                Name      Privilege
         1                root           15
         2               admin           15
         3              remote           15
         4                 vic           15
    km46-gpon-sw(config)# do commit 
        Changes successfully commited (1 chunk)
    km46-gpon-sw(config)# do save
        ........
        Configuration successfully saved to file
    km46-gpon-sw(config)# 

## 4. Настройка модуля switch
Настройка VLAN производится в модуле swtich;

    km46-gpon-sw# switch 
    km46-gpon-olt(switch)#
    km46-gpon-olt(switch)# configure 
    km46-gpon-olt(switch)(config)# vlan 2085
    km46-gpon-olt(switch)(config-vlan)#name vlan2085
    km46-gpon-olt(switch)(config-vlan)#tagged 10-G-front-port 0
    km46-gpon-olt(switch)(config-vlan)#commit

По этому принципу настраиваются все необходимые VLAN на необходимых интерфейсах (front-port 0 - 7, 10G-G-front-port 0 - 1, pon-port 0 - 7).

## 5. Настройка профилей:
### 5.1 Типы и назначение профилей
Для реализации различных моделей предоставления услуг вводится понятие сервисной модели. Сервисаня модель предоставления услуг орпеделяет
общие принципы построения каналов передачи через GEM-порт внутри OLT. В нашем случае используется модель 3 VLAN на сервис.
Просмотреть текущую конфигурацию можно используя команду __"show gpon olt configuration"__:

        km46-gpon-sw# show gpon olt configuration 
            Block duplicated mac:                          enabled
            Ont block time:                                5
            Dhcpra shaper:                                 100
            Profile pppoe-ia:                              pppoe-ia-00       OLT Profile PPPoE Intermediate Agent 0
            Profile dhcp-ra:                               dhcp-ra-00        OLT Profile DHCP Relay Agent 0
            Profile dhcpv6-ra:                               dhcpv6-ra-00      OLT Profile DHCP Relay Agent 0
            Profile dhcp-ra per VLAN:                       <list is empty>
            Profile dhcpv6-ra per VLAN:                       <list is empty>
            Datapath:
                Model:                                     model3
                Broadcast gem port:                        4095
                Multicast gem port:                        4094
            Encryption:
                Enable:                                    false
                Key update interval:                       1
            Unactivated timeout:                           60
            ONT authentication mode:                       both
            Auto reconfigure ONT:                          true
            Auto reconfigure GPON-port:                    true
            Auto reconfigure OLT:                          true
            PLOAM password in alarm:                       false

В текущей конфигурации нам потребуется два типа профилей. Cross-connect, определяющий vlan-преобразования на OLT и ONT.
DBA - определяющий разграничивание полосы пропускания.
   
### 5.2 Cross-connect type management
Для автоматической настройки абонентского устройства нам понадобится профиль cross-connect с типом management:

        km46-gpon-sw(config)#profile cross-connect ACS
        km46-gpon-sw(config-cross-connect)("ACS")#outer vid 200
        km46-gpon-sw(config-cross-connect)("ACS")#user vid untagged
        km46-gpon-sw(config-cross-connect)("ACS")#type management
        km46-gpon-sw(config-cross-connect)("ACS")#exit
    
Проверим корректность конфигурации:

        km46-gpon-sw(config)# do show profile cross-connect ACS
            Name:                                              'ACS'
            Description:                                       'ONT Profile Cross Connect 2'
            Model:                                             ont-rg
            Bridge group:                                      -
            Tag mode:                                          single-tagged
            Outer vid:                                         200
            Outer cos:                                         unused
            Inner vid:                                         -
            U vid:                                             untagged
            U cos:                                             unused
            Mac table entry limit:                             unlimited
            Type:                                              management
            IP host index:                                     0
            Priority queue:                                    0

### 5.3 Cross-connect service
Настроим профиль cross-connect для сервиса PPPoE:

        km46-gpon-sw(config)# profile cross-connect PPPOE_TEST 
        km46-gpon-sw(config-cross-connect)("PPPOE_TEST")#
        km46-gpon-sw(config-cross-connect)("PPPOE_TEST")#outer vid 2085
        km46-gpon-sw(config-cross-connect)("PPPOE_TEST")#user vid 2085
        km46-gpon-sw(config-cross-connect)("PPPOE_TEST")#exit

Проверим корректность конфигурации:

        km46-gpon-sw(config)# do show profile cross-connect PPPOE_TEST 
            Name:                                              'PPPOE_TEST'
            Description:                                       'ONT Profile Cross Connect 1'
            Model:                                             ont-rg
            Bridge group:                                      -
            Tag mode:                                          single-tagged
            Outer vid:                                         2085
            Outer cos:                                         unused
            Inner vid:                                         -
            U vid:                                             2085
            U cos:                                             unused
            Mac table entry limit:                             unlimited
            Type:                                              general
            IP host index:                                     0
            Priority queue:                                    0

### 5.4 DBA
Настроим DBA-профиль для распределения ширины канала между всеми абонентскими устройствами.
Параметрами fixed-bandwidth, guaranteed-bandwidth, besteffort-bandwidth задаются соответственно фиксированная,гарантированная и максимальная полосы.
        
        km46-gpon-sw(config)# profile dba 100mb 
        km46-gpon-sw(config-dba)("100mb")# bandwidth guaranteed 20000
        km46-gpon-sw(config-dba)("100mb")# bandwidth fixed 100000
        km46-gpon-sw(config-dba)("100mb")# exit

Проверим корректность конфигурации:
        
        km46-gpon-sw(config)# do show profile dba 100mb 
            Name:                                              '100mb'
            Description:                                       'ONT Profile DBA 1'
            Dba:
                Sla data:
                    Service class:                             type5
                    Status reporting:                          nsr
                    Alloc size:                                0
                    Alloc period:                              0
                    Fixed bandwidth:                           100000
                    Guaranteed bandwidth:                      20000
                    Besteffort bandwidth:                      1244000
            T-CONT allocation scheme:                          share T-CONT with same profile

Не забудьте применить и сохранить внесённые изменения командами __commit__ и __save__.

## 6. Добавление ONT
При подключении нового абонентского устройства оно будет отображаться в неактивированном состоянии.
Такие устройства можно просмотреть командой __show interface 0-7 unactivated__:
    
    km46-gpon-sw# show interface ont 0-7 unactivated
    -----------------------------------
    GPON-port 0 ONT unactivated list
    -----------------------------------
    ## Serial ONT ID GPON-port Status RSSI[dBm] Version EquipmentID Description
     1 ELTX6900553C n/a 0 UNACTIVATED n/a n/a n/a n/a

Необходимо добавить ONT и назначить необходимые сервисы:
    
    km46-gpon-sw# configure terminal
    km46-gpon-sw(config)# interface ont 0/1
    km46-gpon-sw(config)(if-ont-0/1)# serial ELTX6900553C
    km46-gpon-sw(config)(if-ont-0/1)# service 0 profile cross-connect ACS
    km46-gpon-sw(config)(if-ont-0/1)# service 0 profile dba dba-00
    km46-gpon-sw(config)(if-ont-0/1)# service 1 profile cross-connect PPPOE_TEST
    km46-gpon-sw(config)(if-ont-0/1)# service 1 profile dba 100mb
    km46-gpon-sw(config)(if-ont-0/1)# do commit
    km46-gpon-sw(config)(if-ont-0/1)# do save
    km46-gpon-sw(config)(if-ont-0/1)# exit

Убедимся, что все профили назначились на ONT корректно:

    km46-gpon-sw(config)# do show interface ont 2/1 configuration 

        -----------------------------------
        [ONT2/1] configuration
        -----------------------------------

            Description:                                       ''
            Enabled:                                           true
            Serial:                                            ELTX6900553C
            Password:                                          '00000000'
            Fec up:                                            false
            Easy mode:                                         false
            Downstream broadcast:                              true
            Ber interval:                                      none
            Ber update period:                                 60
            Rf port state:                                     disabled
            Omci error tolerant:                               false
            Service [0]:
                Profile cross connect:                         ACS               ONT Profile Cross Connect 2
                Profile dba:                                   dba-00            ONT Profile DBA 0
                Custom cross connect:                          disabled
            Service [1]:
                Profile cross connect:                         PPPOE_TEST        ONT Profile Cross Connect 1
                Profile dba:                                   100mb             ONT Profile DBA 1
                Custom cross connect:                          disabled
            Profile shaping:                                   shaping-00        ONT Profile Shaping 0
            Profile ports:                                     ports-00          ONT Profile Ports 0
            Profile management:                                unassigned
            Profile scripting:                                 unassigned
            Custom model:                                      none
            Template:                                          unassigned
            Pppoe sessions unlimited:                          false

## 7. Настройка шаблонов для ONT
С ростом числа абонентских устройст настраивать каждое устройство набором профилей и сервисом становится не очень удобно.
Для облегчения настройки однотипных абонентских устройств созданы шаблоны настройки.
Просмотреть список имеющихся шаблонов можно с помощью команды __show template__:
        
        km46-gpon-sw# show template 
            ##                Name    Description
             1         template-00    ONT Template 0
             2           PPPOE-100    ONT Template 1
    
Настроим один шаблон:

        km46-gpon-sw(config)# template PPPOE-100
        km46-gpon-sw(ont-template)("PPPOE-100")# service 0 profile cross-connect ACS
        km46-gpon-sw(ont-template)("PPPOE-100")# service 0 profile dba dba-00
        km46-gpon-sw(ont-template)("PPPOE-100")# service 1 profile cross-connect PPPOE_TEST
        km46-gpon-sw(ont-template)("PPPOE-100")# service 1 profile dba 100mb
        km46-gpon-sw(ont-template)("PPPOE-100")# do commit
        km46-gpon-sw(ont-template)("PPPOE-100")# do save

После создания профиля назначим его на абонентский терминал ELTX69005370. Процесс конфигурации ONT ELTX69005370 идентичен конфигурации ONT ELTX6900553C,
только вместо профилей на сервисы назначается шаблон:
        
        km46-gpon-sw# configure terminal
        km46-gpon-sw(config)# interface ont 2/1
        km46-gpon-sw(config)(if-ont-2/1)# serial ELTX69005370
        km46-gpon-sw(config)(if-ont-2/1)# template PPPOE-100
        km46-gpon-sw(config)(if-ont-2/1)# do commit
        km46-gpon-sw(config)(if-ont-2/1)# do save

После проведенных действий, абонентский терминал сможет подключиться к OLT и получить настройки через ACS сервер.

## 8. Назначение профиля автоконфигурации ONT в ACS.
Станционный терминал OLT-8X rev.c имеет встроенный сервер ACS для автоматической настройки абонентского оборудования по протоколу TR-069.
Сервер ACS работает совместно с внутренним сервером DHCP. Настроим ACS сервер.

    km46-gpon-sw# configure terminal 
    km46-gpon-sw(config)# ip acs server enable
    km46-gpon-sw(config)# ip acs server vid 200
    km46-gpon-sw(config)# ip acs server ip 192.168.200.1
    km46-gpon-sw(config)# ip acs server mask 255.255.258.0
    km46-gpon-sw(config)# ip acs server scheme http

Так как для работы ACS необходим работающий DHCP, то настроим и его:

    km46-gpon-sw(config)# ip dhcp server enable
    km46-gpon-sw(config)# ip dhcp server option-43
    km46-gpon-sw(config)# ip dhcp server range 192.168.200.2 192.168.200.254

При настройке диапазона выдачи ip адресов dhcp сервером необходимо иметь ввиду максимально возможное количество абонентских устройств.
Так как максимально возможный коэффициент разветвления на данном OLT 1:128, то максимально возможное количество абонентских устройст 
будет считаться как 8*128=1024 (8 gpon портов, по 128 ONT на порту).

Настройка профиля автоконфигурации производится внутри встроенного сервера ACS:

    km46-gpon-sw# acs
    (acs) profile
    (acs-profile) add profile PPPOETEST
    (acs-profile) profile PPPOETEST
    (acs-profile-name='PPPOETEST')

В этом меню нужно сконфигурировать профиль с помощью команды __set property__.
Подробную инструкцию по возможным свойствам, а так же готовые профили можно найти по адресу: http://kcs.eltex.nsk.ru/articles/1121
В данный момент для имеющихся ONT NTU-2W используется профиль Ntu-2w----01-pppoe inner. После внесения изменений необходимо сохранить профиль.

    (acs-profile-name='PPPOETEST') commit

Посмотреть список доступных профилей можно с помощью команды __show list__:

    (acs-profile)show list 
    Listing of device profiles:


       ##   Name                 Inform interval Script name          Base profile        

        1:  0                    3600                                                     
        2:  PPPOETEST            3600                                                     

Далее, необходимо назначить созданный профиль конкретному пользователю в меню сервера ACS. Добаление ONT происходит по серийному номеру, но формат серийного номера отличается от того, что используется для добавления ONT на OLT.
Посмотрим список устройств, обратившихся за конфигурацией на сервер ACS

    km46-gpon-sw# acs
    (acs) ont
    (acs-ont)show list all 

       ##   Serial               Profile      Hardware name  Firmware     Last contact        

        1:  454C54586900553C     PPPOETEST    NTU-2W         3.25.2.1561  2017-09-20 14:22:27 
        2:  454C545869005370     PPPOETEST    NTU-2W         3.25.3.101   2017-09-21 10:35:47 

Как видно, серийны номер содержит 16 символов в отличие от серийного номера в меню OLT, там он состоит из 12 символов.
Добавим пользователя и пропишем его приватные параметры, такие как логин и пароль для доступа в интернет.

    km46-gpon-sw# acs
    (acs) user
    (acs-user) add user test
    (acs-user) user test
    (acs-user-subscriber='test') set pon_serial 454C54586900553C
    (acs-user-subscriber='test') set profile PPPOETEST
    (acs-user-subscriber='test') set ppp_login test
    (acs-user-subscriber='test') set ppp_password 123456
    (acs-user-subscriber='test') exit
    (acs-user) commit

На этом настройка пользовательского ONT закончена.

## 9. Дополнительное чтение:
http://eltex-co.ru/upload/iblock/775/kratkoe-rukovodstvo-po-nastroyke-ltp_x-v.3.26.1.pdf

http://eltex-co.ru/upload/iblock/57d/olt-ltp_3.26.1_issue8_rus.pdf

http://eltex-co.ru/upload/iblock/1a5/ltp_x_cli_3.26.1.pdf

