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
Длā примера возьмем пакет NGINX, скачиваем и устанавливаем
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










#### 2 
```

```
