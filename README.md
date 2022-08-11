# Описание файлов в директории
logFileFull.log - полный лог выполнения  
homeWork1 - первое домашнее задание
homeWork2 - второе домашнее задание

# homeWork1 Между двумя виртуалками поднять vpn в режимах
Vagrant_folder - все что понадобится для поднятия ВМ и краткое описание файлов в ней  
Vagrantfile - вагрант файл  
provisioning - директория с файлами провижинга  
```
├── playbook.yml
└── roles
    ├── client
    │   ├── handlers
    │   │   └── main.yml
    │   ├── tasks
    │   │   └── main.yml
    │   └── templates
    │       └── server.conf
    ├── common
    │   └── tasks
    │       └── main.yml
    └── server
        ├── handlers
        │   └── main.yml
        ├── tasks
        │   └── main.yml
        └── templates
            └── server.conf
```
## Описание как запустить виртуальную машину (кратко)
```
vagrant up
vagrant ssh server
iperf3 -s &
vagrant ssh client
iperf3 -c 10.10.10.1 -t 40 -i 5
```
Далее меняем переменную work_mode: tap, на tun в provisioning/playbook.yml. И выполняем провижинг
```
vagrant provision
vagrant ssh client
iperf3 -c 10.10.10.1 -t 40 -i 5
```

## ansible playbook только важное
Роль common нужна для установки на обе ВМ пакетов openvpn iperf3, а так же настройки selinux.  
На ВМ server генерируем статический ключ и переносим его на главную машину
```
- name: create static key
  shell:
    cmd: openvpn --genkey --secret /etc/openvpn/static.key

- name: Copy sert
  fetch:
    src: /etc/openvpn/static.key
    dest: ../
    flat: yes
```
На ВМ cient копируем ключи и переносим в /etc/openvpn/static.key. Запускаем openVPN
## Логи
Измеряем пропускную способность через виртуальный туннель tap, с использованием iperf3
```
[root@server ~]# -----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------
Accepted connection from 10.10.10.2, port 55414
[  5] local 10.10.10.1 port 5201 connected to 10.10.10.2 port 55416
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-1.00   sec  4.67 MBytes  39.1 Mbits/sec
[  5]   1.00-2.00   sec  5.71 MBytes  47.9 Mbits/sec
[  5]   2.00-3.00   sec  5.45 MBytes  45.7 Mbits/sec
[  5]   3.00-4.00   sec  5.23 MBytes  43.9 Mbits/sec
[  5]   4.00-5.00   sec  6.72 MBytes  56.3 Mbits/sec
[  5]   5.00-6.00   sec  6.83 MBytes  57.4 Mbits/sec
[  5]   6.00-7.00   sec  6.61 MBytes  55.4 Mbits/sec
[  5]   7.00-8.00   sec  6.24 MBytes  52.3 Mbits/sec
[  5]   8.00-9.00   sec  6.03 MBytes  50.6 Mbits/sec
[  5]   9.00-10.00  sec  5.76 MBytes  48.3 Mbits/sec
[  5]  10.00-11.00  sec  6.84 MBytes  57.4 Mbits/sec
[  5]  10.00-11.00  sec  6.84 MBytes  57.4 Mbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-11.00  sec  0.00 Bytes  0.00 bits/sec                  sender
[  5]   0.00-11.00  sec  70.7 MBytes  53.9 Mbits/sec                  receiver
iperf3: the client has terminated
-----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------

[root@client ~]# iperf3 -c 10.10.10.1 -t 40 -i 5
Connecting to host 10.10.10.1, port 5201
[  4] local 10.10.10.2 port 55416 connected to 10.10.10.1 port 5201
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-5.00   sec  29.7 MBytes  49.8 Mbits/sec    9    363 KBytes
[  4]   5.00-10.01  sec  31.5 MBytes  52.7 Mbits/sec    0    400 KBytes
^C[  4]  10.01-11.59  sec  10.8 MBytes  57.5 Mbits/sec    0    419 KBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-11.59  sec  72.0 MBytes  52.1 Mbits/sec    9             sender
[  4]   0.00-11.59  sec  0.00 Bytes  0.00 bits/sec                  receiver
iperf3: interrupt - the client has terminated
```
Измеряем пропускную способность через виртуальный туннель tun.
```
[root@server ~]# Accepted connection from 10.10.10.2, port 55418
[  5] local 10.10.10.1 port 5201 connected to 10.10.10.2 port 55420
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-1.00   sec  5.74 MBytes  48.1 Mbits/sec
[  5]   1.00-2.00   sec  5.72 MBytes  48.0 Mbits/sec
[  5]   2.00-3.00   sec  5.99 MBytes  50.3 Mbits/sec
[  5]   3.00-4.00   sec  6.12 MBytes  51.3 Mbits/sec
[  5]   4.00-5.01   sec  5.44 MBytes  45.3 Mbits/sec
[  5]   5.01-6.01   sec  5.46 MBytes  45.9 Mbits/sec
[  5]   6.01-7.00   sec  5.25 MBytes  44.2 Mbits/sec
[  5]   7.00-8.01   sec  6.55 MBytes  54.5 Mbits/sec
[  5]   8.01-9.01   sec  5.40 MBytes  45.4 Mbits/sec
[  5]   9.01-10.00  sec  5.83 MBytes  49.1 Mbits/sec
[  5]  10.00-11.00  sec  5.56 MBytes  46.6 Mbits/sec
[  5]  11.00-12.00  sec  5.36 MBytes  45.0 Mbits/sec
[  5]  12.00-13.00  sec  5.92 MBytes  49.7 Mbits/sec
[  5]  13.00-14.01  sec  5.80 MBytes  48.4 Mbits/sec
[  5]  14.01-15.00  sec  6.10 MBytes  51.3 Mbits/sec
[  5]  14.01-15.00  sec  6.10 MBytes  51.3 Mbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-15.00  sec  0.00 Bytes  0.00 bits/sec                  sender
[  5]   0.00-15.00  sec  91.5 MBytes  51.2 Mbits/sec                  receiver
iperf3: the client has terminated
-----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------


[root@client ~]# iperf3 -c 10.10.10.1 -t 40 -i 5
Connecting to host 10.10.10.1, port 5201
[  4] local 10.10.10.2 port 55420 connected to 10.10.10.1 port 5201
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-5.00   sec  32.1 MBytes  53.8 Mbits/sec    0   1.00 MBytes
[  4]   5.00-10.00  sec  28.6 MBytes  48.0 Mbits/sec  180    498 KBytes
[  4]  10.00-15.01  sec  28.6 MBytes  47.9 Mbits/sec    6    446 KBytes
^C[  4]  15.01-15.77  sec  5.00 MBytes  54.8 Mbits/sec    0    452 KBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-15.77  sec  94.2 MBytes  50.1 Mbits/sec  186             sender
[  4]   0.00-15.77  sec  0.00 Bytes  0.00 bits/sec                  receiver
iperf3: interrupt - the client has terminated
```

##Отличия режимов
TUN наиболее часто используемый интерфейс для построения VPN. TUN работает на L3 уровне, и используется когда необходимо объеденить разные сайты с разными подсетями, а также в случае RAS.  
TAP интерфейс работает на L2 уровне, т.е. виден ARP траффик если слушать TAP интерфейс. TAP интерфейс совместно с бридж-интерфейсом могут использоваться когда надо объединить два и более филиалов с одинаковой подсетью.  

TAP эмулирует Ethernet устройство и работает на канальном уровне модели OSI, оперируя кадрами Ethernet.  
TUN (сетевой туннель) работает на сетевом уровне модели OSI, оперируя IP пакетами.  
TAP используется для создания сетевого моста, тогда как TUN для маршрутизации.  

### TAP
Преимущества:
```
ведёт себя как настоящий сетевой адаптер (за исключением того, что он виртуальный);
может осуществлять транспорт любого сетевого протокола (IPv4, IPv6, IPX и прочих);
работает на 2 уровне, поэтому может передавать Ethernet-кадры внутри тоннеля;
позволяет использовать мосты.
```
Недостатки:
```
в тоннель попадает broadcast-трафик, что иногда не требуется;
добавляет свои заголовки поверх заголовков Ethernet на все пакеты, которые следуют через тоннель;
в целом, менее масштабируем из-за предыдущих двух пунктов;
не поддерживается устройствами Android и iOS (по информации с сайта OpenVPN).
```

### TUN
Преимущества:
```
передает только пакеты протокола IP (3й уровень);
сравнительно (отн. TAP) меньшие накладные расходы и, фактически, ходит только тот IP-трафик, который предназначен конкретному клиенту.
```
Недостатки:
```
broadcast-трафик обычно не передаётся;
нельзя использовать мосты.
```

# homeWork2 Поднять RAS на базе OpenVPN с клиентскими сертификатами, подключиться с локальной машины на виртуалку
Vagrant_folder - все что понадобится для поднятия ВМ и краткое описание файлов в ней  
Vagrantfile - вагрант файл  
provisioning - директория с файлами провижинга  
```
provisioning/
├── playbook.yml
└── templates
    └── server.conf
```

## Описание как запустить виртуальную машину (кратко)
```
vagrant up
openvpn --config client.conf &
ping 10.10.10.1
```

## ansible playbook только важное
Создаем сертификаты сервера и клиента
```
  - name: init pki
    shell:
      cmd: /usr/share/easy-rsa/3/easyrsa init-pki
    args:
      chdir: /etc/openvpn/

  - name: create cert for server
    shell: |
      echo 'rasvpn' | /usr/share/easy-rsa/3/easyrsa build-ca nopass
      echo 'rasvpn' | /usr/share/easy-rsa/3/easyrsa gen-req server nopass
      echo 'yes' | /usr/share/easy-rsa/3/easyrsa sign-req server server
      /usr/share/easy-rsa/3/easyrsa gen-dh
      openvpn --genkey --secret ta.key
    args:
      chdir: /etc/openvpn/

  - name : create cert for client
    shell: |
      echo 'client' | /usr/share/easy-rsa/3/easyrsa gen-req client nopass
      echo 'yes' | /usr/share/easy-rsa/3/easyrsa sign-req client client
    args:
      chdir: /etc/openvpn/
```
Копируем сертификата на главную машину
```
  - name: Copy sert
    fetch:
      src: "{{ item }}"
      dest: ../
      flat: yes
    with_items:
      - /etc/openvpn/pki/ca.crt
      - /etc/openvpn/pki/issued/client.crt
      - /etc/openvpn/pki/private/client.key
```

## Логи
Запускаем в фоновом режиме openVPN с использованием клиентского конфигурационного файла. И отправляем пинг на ВМ.
```
root@otssTestServer:/home/lx/git/openvpn2# openvpn --config client.conf &
[1] 17470
root@otssTestServer:/home/lx/git/openvpn2# Wed Aug 10 19:54:41 2022 WARNING: file './client.key' is group or others accessible
Wed Aug 10 19:54:41 2022 OpenVPN 2.4.4 x86_64-pc-linux-gnu [SSL (OpenSSL)] [LZO] [LZ4] [EPOLL] [PKCS11] [MH/PKTINFO] [AEAD] built on Mar 22 2022
Wed Aug 10 19:54:41 2022 library versions: OpenSSL 1.1.1  11 Sep 2018, LZO 2.08
Wed Aug 10 19:54:41 2022 WARNING: No server certificate verification method has been enabled.  See http://openvpn.net/howto.html#mitm for more info.
Wed Aug 10 19:54:41 2022 TCP/UDP: Preserving recently used remote address: [AF_INET]192.168.10.10:1207
Wed Aug 10 19:54:41 2022 Socket Buffers: R=[212992->212992] S=[212992->212992]
Wed Aug 10 19:54:41 2022 UDP link local (bound): [AF_INET][undef]:1194
Wed Aug 10 19:54:41 2022 UDP link remote: [AF_INET]192.168.10.10:1207
Wed Aug 10 19:54:41 2022 TLS: Initial packet from [AF_INET]192.168.10.10:1207, sid=0d361514 0666f734
Wed Aug 10 19:54:41 2022 VERIFY OK: depth=1, CN=rasvpn
Wed Aug 10 19:54:41 2022 VERIFY OK: depth=0, CN=rasvpn
Wed Aug 10 19:54:41 2022 Control Channel: TLSv1.2, cipher TLSv1.2 ECDHE-RSA-AES256-GCM-SHA384, 2048 bit RSA
Wed Aug 10 19:54:41 2022 [rasvpn] Peer Connection Initiated with [AF_INET]192.168.10.10:1207
Wed Aug 10 19:54:42 2022 SENT CONTROL [rasvpn]: 'PUSH_REQUEST' (status=1)
Wed Aug 10 19:54:42 2022 PUSH: Received control message: 'PUSH_REPLY,route 10.10.10.0 255.255.255.0,topology net30,ping 10,ping-restart 120,ifconfig 10.10.10.6 10.10.10.5,peer-id 0,cipher AES-256-GCM'
Wed Aug 10 19:54:42 2022 OPTIONS IMPORT: timers and/or timeouts modified
Wed Aug 10 19:54:42 2022 OPTIONS IMPORT: --ifconfig/up options modified
Wed Aug 10 19:54:42 2022 OPTIONS IMPORT: route options modified
Wed Aug 10 19:54:42 2022 OPTIONS IMPORT: peer-id set
Wed Aug 10 19:54:42 2022 OPTIONS IMPORT: adjusting link_mtu to 1625
Wed Aug 10 19:54:42 2022 OPTIONS IMPORT: data channel crypto options modified
Wed Aug 10 19:54:42 2022 Data Channel: using negotiated cipher 'AES-256-GCM'
Wed Aug 10 19:54:42 2022 Outgoing Data Channel: Cipher 'AES-256-GCM' initialized with 256 bit key
Wed Aug 10 19:54:42 2022 Incoming Data Channel: Cipher 'AES-256-GCM' initialized with 256 bit key
Wed Aug 10 19:54:42 2022 ROUTE_GATEWAY 10.32.80.254/255.255.255.0 IFACE=ens192 HWADDR=00:50:56:b3:cd:af
Wed Aug 10 19:54:42 2022 TUN/TAP device tun0 opened
Wed Aug 10 19:54:42 2022 TUN/TAP TX queue length set to 100
Wed Aug 10 19:54:42 2022 do_ifconfig, tt->did_ifconfig_ipv6_setup=0
Wed Aug 10 19:54:42 2022 /sbin/ip link set dev tun0 up mtu 1500
Wed Aug 10 19:54:42 2022 /sbin/ip addr add dev tun0 local 10.10.10.6 peer 10.10.10.5
Wed Aug 10 19:54:42 2022 /sbin/ip route add 10.10.10.0/24 via 10.10.10.5
Wed Aug 10 19:54:42 2022 WARNING: this configuration may cache passwords in memory -- use the auth-nocache option to prevent this
Wed Aug 10 19:54:42 2022 Initialization Sequence Completed
ping 10.10.10.1
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=0.929 ms
64 bytes from 10.10.10.1: icmp_seq=2 ttl=64 time=0.923 ms
^C
--- 10.10.10.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.923/0.926/0.929/0.003 ms
```
📚Домашнее задание/проектная работа разработано(-на) для курса ["Administrator Linux. Professional"](https://otus.ru/lessons/linux-professional/)