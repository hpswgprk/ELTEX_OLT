
В последнее время возникают обращения по поводу несоответсвия скорости тарифному плану у клиентов
подключенных через PON. В абсолютном большинстве случаев они связаны с неправильным профилем конфигурации ONT.
Для проверки таких клиентов необходимо выполнить следующие шаги.

Подключиться к gpon-станции и выяснить серийный номер абонентского терминала. Абонентский терминал
привязывается к клиенту в контексте acs-user на станционном терминале. В общем случае абонент на станции равен 
логину абонента. Например:

    km46-gpon-olt# 
    km46-gpon-olt# acs
    km46-gpon-olt(acs)# user 
    km46-gpon-olt(acs-user)# show user 747394566
    Subscriber '747394566' not found!
    km46-gpon-olt(acs-user)# 

Как видно из вывода, на этой станции абонент не сконфигурирован. Очевидно, он на другой станции.

    sov115-gpon-sw# acs
    sov115-gpon-sw(acs)# user 
    sov115-gpon-sw(acs-user)# show user 747394566
    Information about subscriber '747394566':


        Subscriber ID = "747394566"
           PON serial = "454C545869005610"
              Profile = "PPPOETEST"

      voice1_enable: -
      voice1_number: -
      voice1_password: -
      voice2_enable: -
      voice2_number: -
      voice2_password: -
      sip_proxy: -
      ppp_login: "747394566"
      ppp_password: "308406"
      user_password: -
      admin_password: -
      wifi_enable: -
      wifi_ssid: -
      wifi_encoding: -
      wifi_password: -

Прежде чем смотреть конфигурацию абонентского устройства необходимо выйти в стандартный контекст командой __exit__
до приглашения следющего вида:

    sov115-gpon-sw#

Из предыдущего вывода нам интересен __PON serial__.
По нему мы посмотрим конфигурацию абонентского устройства:

    sov115-gpon-sw# show interface ont 454C545869005610 configuration 

    -----------------------------------
    [ONT3/13] configuration
    -----------------------------------

        Description:                                       'suv83'
        Enabled:                                           true
        Serial:                                            ELTX69005610
        Password:                                          'user'
    [T] Fec up:                                            false
    [T] Easy mode:                                         false
    [T] Downstream broadcast:                              true
    [T] Ber interval:                                      none
    [T] Ber update period:                                 60
    [T] Rf port state:                                     disabled
    [T] Omci error tolerant:                               false
        Service [0]:
    [T]     Profile cross connect:                         ACS               ONT Profile Cross  Connect 2
    [T]     Profile dba:                                   dba-00            ONT Profile DBA 0
            Custom cross connect:                          disabled
        Service [1]:
    [T]     Profile cross connect:                         PPPOE_TEST        ONT Profile Cross  Connect 1
    [T]     Profile dba:                                   300mb             ONT Profile DBA 3
            Custom cross connect:                          disabled
    [T] Profile shaping:                                   shaping-00        ONT Profile Shaping 0
    [T] Profile ports:                                     ports-00          ONT Profile Ports 0
    [T] Profile management:                                unassigned
        Template:                                          PPPOE-100         ONT Template 2


В последней строке этого вывода легко заметить, что клиенту назначен профиль __PPPOE-100__, однако у клиента тариф на 300 мегабит.
Давайте исправим это:

    sov115-gpon-sw# configure terminal
    sov115-gpon-sw(config)# interface ont 3/13
    sov115-gpon-sw(config)(if-ont-3/13)# 

Может возникнуть вопрос почему __ont 3/13__. Это должно быть очевидно из вывода команды __show interface ont 454C545869005610 configuration__.
Каждое абонентское устройство имеет свой идентификатор вида __PON port/ONT id__ т.е. устройство
нашего клиента имеет id 13 на 3 порту.

Назначаем клиенту правильный профиль:

    sov115-gpon-sw(config)(if-ont-3/13)# template PPPOE-300 

И сохраняем настройки.

    sov115-gpon-sw(config)(if-ont-3/13)# do co
    commit copy   
    sov115-gpon-sw(config)(if-ont-3/13)# do commit 
        Changes successfully commited (1 chunk)
    sov115-gpon-sw(config)(if-ont-3/13)# do save
        ........
        Configuration successfully saved to file
    sov115-gpon-sw(config)(if-ont-3/13)# exit
    sov115-gpon-sw(config)# exit

После всех проведенных манипуляций желательно перезагрузить клиентское устройство:

    sov115-gpon-sw# send omci reset interface ont 3/13
