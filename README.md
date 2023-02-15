# Сбор и анализ логов
## Домашнее задание:
1. в вагранте поднимаем 2 машины web и log
2. на web поднимаем nginx
3. на log настраиваем центральный лог сервер на любой системе на выбор
    journald;
    rsyslog;
    elk.
4. настраиваем аудит, следящий за изменением конфигов нжинкса

## Выполнение

С помощью `vagrant up` разворачиваем сервер с nginx и лог сервер с уже необходимыми для логирования настройками.

```bash
[root@web ~]# ss -ntlp | grep 80
LISTEN     0      128          *:80                       *:*                   users:(("nginx",pid=4151,fd=5),("nginx",pid=4149,fd=5))
LISTEN     0      128       [::]:80                    [::]:*                   users:(("nginx",pid=4151,fd=6),("nginx",pid=4149,fd=6))
```

```bash
[root@log ~]# ss -ntul | grep 514
udp    UNCONN     0      0         *:514                   *:*                  
udp    UNCONN     0      0      [::]:514                [::]:*                  
tcp    LISTEN     0      25        *:514                   *:*                  
tcp    LISTEN     0      25     [::]:514                [::]:*           
```

Чтобы проверить, что логи ошибок также улетают на удаленный сервер, удалить картинку, к
которой будет обращаться nginx во время открытия веб-сраницы, и смотрим логи на log-сервере:

```bash
[root@log ~]# cat /var/log/rsyslog/web/nginx_access.log
Feb 15 22:15:54 web nginx_access: 192.168.56.1 - - [15/Feb/2023:22:15:54 +0300] "GET / HTTP/1.1" 200 4833 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/109.0"
Feb 15 22:15:54 web nginx_access: 192.168.56.1 - - [15/Feb/2023:22:15:54 +0300] "GET /img/centos-logo.png HTTP/1.1" 200 3030 "http://192.168.56.10/" "Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/109.0"
Feb 15 22:15:54 web nginx_access: 192.168.56.1 - - [15/Feb/2023:22:15:54 +0300] "GET /img/header-background.png HTTP/1.1" 200 82896 "http://192.168.56.10/" "Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/109.0"
Feb 15 22:15:54 web nginx_access: 192.168.56.1 - - [15/Feb/2023:22:15:54 +0300]
...
[root@log ~]# cat /var/log/rsyslog/web/nginx_error.log
Feb 15 22:15:54 web nginx_error: 2023/02/15 22:15:54 [error] 4616#4616: *2 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 192.168.56.1, server: _, request: "GET /favicon.ico HTTP/1.1", host: "192.168.56.10", referrer: "http://192.168.56.10/"
```
Поменяем атрибут у файла `/etc/nginx/nginx.conf` и провериv на log-сервере, что пришла информация об изменении атрибута:
```bash
[root@log ~]# grep web /var/log/audit/audit.log
node=web type=SYSCALL msg=audit(1676488925.302:1953): arch=c000003e syscall=268 success=yes exit=0 a0=ffffffffffffff9c a1=1e30420 a2=1ed a3=7ffc3e39cee0 items=1 ppid=5428 pid=24152 auid=1000 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=7 comm="chmod" exe="/usr/bin/chmod" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="nginx_conf"
node=web type=CWD msg=audit(1676488925.302:1953):  cwd="/root"
node=web type=PATH msg=audit(1676488925.302:1953): item=0 name="/etc/nginx/nginx.conf" inode=558226 dev=08:01 mode=0100644 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 objtype=NORMAL cap_fp=0000000000000000 cap_fi=0000000000000000 cap_fe=0 cap_fver=0
node=web type=PROCTITLE msg=audit(1676488925.302:1953): proctitle=63686D6F64002B78002F6574632F6E67696E782F6E67696E782E636F6E66
```
