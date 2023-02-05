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







#### 2 
```

```
