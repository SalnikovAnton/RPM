# Размещаем свой RPM в своем репозитории
1) Создать свой RPM пакет; 
2) Создать свой репозиторий и разместить там ранее собраннýй RPM; 

#### 1 Устанавливаем пакеты:
```
[root@otuslinux ~]# yum install -y \
redhat-lsb-core \
wget \
rpmdevtools \
rpm-build \
createrepo \
yum-utils \
gcc
```
Для примера возьмем пакет NGINX, скачиваем и устанавливаем
```
[root@otuslinux ~]# wget https://nginx.org/packages/centos/8/SRPMS/nginx-1.20.2-1.el8.ngx.src.rpm
[root@otuslinux ~]# rpm -i nginx-1.20.2-1.el8.ngx.src.rpm
```
Также скачиваем и разархивируем исходники для openssl - он потребуется при сборке
```
[root@otuslinux vagrant]# wget https://www.openssl.org/source/openssl-1.1.1s.tar.gz
[root@otuslinux vagrant]# tar -xvf openssl-1.1.1s.tar.gz
```
Дополнительно установим все зависимости
```
[root@otuslinux ~]# yum-builddep rpmbuild/SPECS/nginx.spec
```
Правим файл nginx.spec , чтобы NGINX собирался с необходимыми нам опциями: добавляем строчку 
```
--with-openssl=/home/vagrant/openssl-1.1.1s             < путь до каталога распаковки openssl
```
Запускаем сборку RPM пакета
```
[root@otuslinux ~]# rpmbuild -bb rpmbuild/SPECS/nginx.spec
---
Wrote: /root/rpmbuild/RPMS/x86_64/nginx-1.20.2-1.el8.ngx.x86_64.rpm
Wrote: /root/rpmbuild/RPMS/x86_64/nginx-debuginfo-1.20.2-1.el8.ngx.x86_64.rpm
Executing(%clean): /bin/sh -e /var/tmp/rpm-tmp.ADsYgD
+ umask 022
+ cd /root/rpmbuild/BUILD
+ cd nginx-1.20.2
+ /usr/bin/rm -rf /root/rpmbuild/BUILDROOT/nginx-1.20.2-1.el8.ngx.x86_64
+ exit 0
---
```
Убедимся что пакеты созданны
```
[root@otuslinux ~]# ll rpmbuild/RPMS/x86_64/
total 4456
-rw-r--r--. 1 root root 2060740 Feb  5 18:03 nginx-1.20.2-1.el8.ngx.x86_64.rpm
-rw-r--r--. 1 root root 2495832 Feb  5 18:03 nginx-debuginfo-1.20.2-1.el8.ngx.x86_64.rpm
```
Теперь установим наш пакет и проверим работу
```
[root@otuslinux ~]# yum localinstall -y rpmbuild/RPMS/x86_64/nginx-1.20.2-1.el8.ngx.x86_64.rpm
---
Verifying        : nginx-1:1.20.2-1.el8.ngx.x86_64                                                1/1 

Installed:
  nginx-1:1.20.2-1.el8.ngx.x86_64                                                                       

Complete!

[root@otuslinux ~]# systemctl start nginx
[root@otuslinux ~]# systemctl status nginx
● nginx.service - nginx - high performance web server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Sun 2023-02-05 18:42:46 UTC; 10s ago
     Docs: http://nginx.org/en/docs/
  Process: 51598 ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf (code=exited, status=0/SUCCESS)
 Main PID: 51599 (nginx)
    Tasks: 2 (limit: 5975)
   Memory: 1.9M
   CGroup: /system.slice/nginx.service
           ├─51599 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx.conf
           └─51600 nginx: worker process

Feb 05 18:42:46 otuslinux systemd[1]: Starting nginx - high performance web server...
Feb 05 18:42:46 otuslinux systemd[1]: nginx.service: Can't open PID file /var/run/nginx.pid (yet?) afte>
Feb 05 18:42:46 otuslinux systemd[1]: Started nginx - high performance web server.
```

#### 2 Теперь приступим к созданию своего репозитория.

Создаем каталог /repo в директории для статики в nginx
```
[root@otuslinux ~]# mkdir /usr/share/nginx/html/repo
```
Копируем туда наш собранный RPM пакет nginx и дополнительно RPM для установки репозитория Percona-Server
```
[root@otuslinux ~]# cp rpmbuild/RPMS/x86_64/nginx-1.20.2-1.el8.ngx.x86_64.rpm /usr/share/nginx/html/repo/
[root@otuslinux ~]# wget https://downloads.percona.com/downloads/percona-distribution-mysql-ps/percona-distribution-mysql-ps-8.0.28/binary/redhat/8/x86_64/percona-orchestrator-3.2.6-2.el8.x86_64.rpm -O /usr/share/nginx/html/repo/percona-orchestrator-3.2.6-2.el8.x86_64.rpm
```
получается
```
[root@otuslinux ~]# ll /usr/share/nginx/html/repo/
total 7120
-rw-r--r--. 1 root root 2060740 Feb  5 18:51 nginx-1.20.2-1.el8.ngx.x86_64.rpm
-rw-r--r--. 1 root root 5222976 Feb 16  2022 percona-orchestrator-3.2.6-2.el8.x86_64.rpm
```
Инициализируем репозиторий командой
```
[root@otuslinux ~]# createrepo /usr/share/nginx/html/repo/
Directory walk started
Directory walk done - 2 packages
Temporary output repo path: /usr/share/nginx/html/repo/.repodata/
Preparing sqlite DBs
Pool started (with 5 workers)
Pool finished
```
настроим в NGINX доступ к листингу каталога, для этого в файле /etc/nginx/conf.d/default.conf добавим директиву autoindex on. В результате location будет выглядеть так
```
server {
    listen       80;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        autoindex on;                      < добавили эту директиву
    }
```
Проверяем синтаксис и перезапускаем NGINX
```
[root@otuslinux ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@otuslinux ~]# nginx -s reload
```
Проверим с помощью curl:
```
[root@otuslinux ~]# curl -a http://localhost/repo/
<html>
<head><title>Index of /repo/</title></head>
<body>
<h1>Index of /repo/</h1><hr><pre><a href="../">../</a>
<a href="repodata/">repodata/</a>                                          05-Feb-2023 18:59                   -
<a href="nginx-1.20.2-1.el8.ngx.x86_64.rpm">nginx-1.20.2-1.el8.ngx.x86_64.rpm</a>                  05-Feb-2023 18:51             2060740
<a href="percona-orchestrator-3.2.6-2.el8.x86_64.rpm">percona-orchestrator-3.2.6-2.el8.x86_64.rpm</a>        16-Feb-2022 15:57             5222976
</pre><hr></body>
</html>
```
Теперь добавим репазиторий в /etc/yum.repos.d
```
[root@otuslinux ~]# cat >> /etc/yum.repos.d/otus.repo << EOF
> [otus]
> name=otus-linux
> baseurl=http://localhost/repo
> gpgcheck=0
> enabled=1
> EOF
```
Проверяем подключение репозиторий и смотрим что в нем есть:
```
[root@otuslinux ~]# yum repolist enabled | grep otus
otus                           otus-linux

[root@otuslinux ~]# yum list | grep otus
otus-linux                                      183 kB/s | 2.8 kB     00:00    
percona-orchestrator.x86_64                            2:3.2.6-2.el8                                              otus            
```
установим репозиторий percona-release
```
[root@otuslinux ~]# yum install percona-orchestrator.x86_64 -y

----
Installed:
  jq-1.6-6.el8.x86_64     oniguruma-6.8.2-2.el8.x86_64     percona-orchestrator-2:3.2.6-2.el8.x86_64    

Complete!
```
P.S. Команда для обновления репозиторий (необходимо при каждом добавлении файлов)
```
createrepo /usr/share/nginx/html/repo/
```
