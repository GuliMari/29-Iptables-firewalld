# 29-Iptables-firewalld:

1. реализовать knocking port: centralRouter может попасть на ssh inetrRouter через knock скрипт
    пример в материалах.
2. добавить inetRouter2, который виден(маршрутизируется (host-only тип сети для виртуалки)) с хоста или форвардится порт через локалхост.
3. запустить nginx на centralServer.
4. пробросить 80й порт на inetRouter2 8080.
5. дефолт в инет оставить через inetRouter.
*  реализовать проход на 80й порт без маскарадинга


## 1. Knocking port

На `inetRouter` добавлены правила iptables для knocking port и разрешен вход через ssh по логину и паролю;  
на `centralRouter` добавлен скрипт, с помощью которого будем "стучаться" на `inetRouter`.

```bash
[root@centralRouter vagrant]# ssh vagrant@192.168.255.1
^C
[root@centralRouter vagrant]# ./knock.sh 192.168.255.1 8881 7777 9991

Starting Nmap 6.40 ( http://nmap.org ) at 2023-04-12 16:46 UTC
Warning: 192.168.255.1 giving up on port because retransmission cap hit (0).
Nmap scan report for 192.168.255.1
Host is up (0.00043s latency).
PORT     STATE    SERVICE
8881/tcp filtered unknown
MAC Address: 08:00:27:18:63:A6 (Cadmus Computer Systems)

Nmap done: 1 IP address (1 host up) scanned in 0.38 seconds

Starting Nmap 6.40 ( http://nmap.org ) at 2023-04-12 16:46 UTC
Warning: 192.168.255.1 giving up on port because retransmission cap hit (0).
Nmap scan report for 192.168.255.1
Host is up (0.00046s latency).
PORT     STATE    SERVICE
7777/tcp filtered cbt
MAC Address: 08:00:27:18:63:A6 (Cadmus Computer Systems)

Nmap done: 1 IP address (1 host up) scanned in 0.37 seconds

Starting Nmap 6.40 ( http://nmap.org ) at 2023-04-12 16:46 UTC
Warning: 192.168.255.1 giving up on port because retransmission cap hit (0).
Nmap scan report for 192.168.255.1
Host is up (0.00037s latency).
PORT     STATE    SERVICE
9991/tcp filtered issa
MAC Address: 08:00:27:18:63:A6 (Cadmus Computer Systems)

Nmap done: 1 IP address (1 host up) scanned in 0.38 seconds
[root@centralRouter vagrant]# ssh vagrant@192.168.255.1
vagrant@192.168.255.1's password: 
Last login: Wed Apr 12 16:33:04 2023 from 10.0.2.2
[vagrant@inetRouter ~]$ 
```

## 2. Проброс портов
Из задания не совсем понятно, в какой сети находится inetRouter2. Я соединила все роутеры в единую сеть `router-net`.
Проверяем работу `nginx` на хостовой машине:
```bash
tw4@tw4-mint:~/Desktop/Linux/Pro/24-Iptables/29-Iptables-firewalld$ curl -I localhost:8082
HTTP/1.1 200 OK
Server: nginx/1.20.1
Date: Wed, 19 Apr 2023 08:11:19 GMT
Content-Type: text/html
Content-Length: 4833
Last-Modified: Fri, 16 May 2014 15:12:48 GMT
Connection: keep-alive
ETag: "53762af0-12e1"
Accept-Ranges: bytes
```
P.S. В плейбуке почему-то не срабатывал `restart network` ни через `ansible.builtin.systemd`, ни через `ansible.builtin.service`. При входе на сервер и ручном вводе команды в терминале, все срабатывало. В предыдущих плейбуках ansible отрабатывал без ошибок. В связи с чем в плейбук добавила `reboot` серверов, заодно проверив сохраняются ли настройки.
