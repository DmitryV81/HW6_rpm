Выполнение домашнего задания №6

Управление пакетами. Дистрибьюция софта.

Цель задания:
1. Создать свой RPM пакет
2. Создать свой репозиторий и разместить там ранее собранный RPM

Пошаговое описание.

Реализация домашнего задания производится в Vagrant, создаем пакет nginx+ssl, размещаем его в локальном репозитории на apache.

Все команды выполняются в Vagrant из подключаемого файла rpm.sh

1. Устанавливаем необходимые для сборки пакеты:

```
yum install -y redhat-lsb-core wget rpmdevtools rpm-build createrepo yum-utils gcc
```
2. Загружаем SRPM пакет NGINX:

```
wget https://nginx.org/packages/centos/7/SRPMS/nginx-1.14.1-1.el7_4.ngx.src.rpm
```
3. Скачиваем и разархивируем исходники для openssl:

```
wget --no-check-certificate https://www.openssl.org/source/openssl-1.1.1s.tar.gz
tar -xvf openssl-1.1.1s.tar.gz
```

4. Ставим все зависимости чтобы в процессе сборки не было ошибок:

```
yum-builddep -y rpmbuild/SPECS/nginx.spec
```

5. Редкатируем spec-файл. Добавляем опцию --with-openssl. Данный файл находится в каталоге с Vagrantfile. Отредактированный файл копируем в директорию: 

```
\cp /vagrant/nginx.spec rpmbuild/SPECS/nginx.spec
```

6. Приступаем к сборке пакета:

```
 rpmbuild -bb rpmbuild/SPECS/nginx.spec
```

7. Устанавливаем и стартуем apache. Создаем каталог для будущего репозитория:

```
yum install -y httpd
systemctl start httpd
systemctl status httpd
mkdir /var/www/html/repo
```
8. Копируем в созданный каталог собранный пакет nginx:

```
cp rpmbuild/RPMS/x86_64/nginx-1.14.1-1.el7_4.ngx.x86_64.rpm /var/www/html/repo/
```
9. Инициализируем репозиторий:

```
 createrepo /usr/share/nginx/html/repo/
```

10. Добавляем репозиторий в /etc/yum.repos.d

```
cat >> /etc/yum.repos.d/otus.repo << EOF
[otus]
name=otus-linux
baseurl=http://localhost/repo
gpgcheck=0
enabled=1
EOF
```

11. Проверяем, что репозиторий подключен:

```
[vagrant@rpm ~]$ yum repolist enabled | grep otus
otus                             otus-linux                                    2
```

12. Смотрим содержимое:

```
[vagrant@rpm ~]$ yum list | grep otus
percona-release.noarch                      0.1-6                      @otus    
nginx.x86_64                                1:1.14.1-1.el7_4.ngx       otus
```
