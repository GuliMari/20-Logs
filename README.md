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
Feb 10 20:15:03 web nginx_access: 192.168.56.1 - - [10/Feb/2023:20:15:03 +0300] "GET /img/header-background.png HTTP/1.1" 200 82896 "http://192.168.56.10/" "Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/109.0"
Feb 10 20:15:03 web nginx_access: 192.168.56.1 - - [10/Feb/2023:20:15:03 +0300] "GET /favicon.ico HTTP/1.1" 404 3650 "http://192.168.56.10/" "Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/109.0"
...
[root@log ~]# cat /var/log/rsyslog/web/nginx_error.log
Feb 10 20:15:03 web nginx_error: 2023/02/10 20:15:03 [error] 4148#4148: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 192.168.56.1, server: _, request: "GET /favicon.ico HTTP/1.1", host: "192.168.56.10", referrer: "http://192.168.56.10/"
```
