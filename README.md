# homework7
```
1. Поднимаем ВМ с "generic/centos8s", версия "4.3.8"
2. Переходим на суперпользователя и переходит в директорию  /root
     sudo su
     cd
3. Устанавливаем ПО:
     yum install -y redhat-lsb-core
     yum install -y wget 
     yum install -y rpmdevtools 
     yum install -y rpm-build 
     yum install -y createrepo
     yum install -y yum-utils 
     yum install -y gcc
4. Выбираем ПО веб-сервер nginx и openssl
5. Скачиваем последний пакет SRPM nginx 
     wget https://nginx.org/packages/centos/8/SRPMS/nginx-1.24.0-1.el8.ngx.src.rpm
6. Создаем директорию /root/rpmbuild
     mkdir rpmbuild
7. Устанавливаем пакет из скачанного файла 
     rpm -i nginx-1.*
8. Скачиваем openssl
     wget https://github.com/openssl/openssl/archive/refs/heads/OpenSSL_1_1_1-stable.zip
9. Разархивируем архив /root/rpmbuild/SOURCES/nginx-1.24.0.tar.gz
     tar -xvf nginx-1.24.0.tar.gz
10. Загружаем зависимости 
     yum-builddep rpmbuild/SPECS/nginx.spec
11. Добавляем в ~/rpmbuild/SPECS/nginx.spec в раздел %build путь к openssl
     %build
     ./configure %{BASE_CONFIGURE_ARGS} \
     --with-cc-opt="%{WITH_CC_OPT}" \
     --with-ld-opt="%{WITH_LD_OPT}" \
     --with-openssl=/root/openssl-OpenSSL_1_1_1-stable
12. Начинаем сборку
     rpmbuild -bb rpmbuild/SPECS/nginx.spec
13. Проверяем результат визуально
     ll rpmbuild/RPMS/x86_64/
14. Устанавиваем собранный веб-сервер 
     yum localinstall -y rpmbuild/RPMS/x86_64/nginx-1.24.0-1.el8.ngx.x86_64.rpm
15. Запускаем и проверяем статус веб-сервера
     systemctl start nginx
     systemctl status nginx
16. Создаем локальный репозиторий. Создаем директорию repo в директории веб-сервера /usr/share/nginx/html
     mkdir /usr/share/nginx/html/repo
17. Копируем собранный rpm и загружаем rpm Percona-Server'а для установки репозитория
     cp rpmbuild/RPMS/x86_64/nginx-1.24.0-1.el8.ngx.x86_64.rpm /usr/share/nginx/html/repo/nginx-1.24.0-1.el8.ngx.x86_64.rpm
     wget https://downloads.percona.com/downloads/percona-distribution-mysql-ps/percona-distribution-mysql-ps-8.0.28/binary/redhat/8/x86_64/percona-orchestrator-3.2.6-2.el8.x86_64.rpm -O /usr/share/nginx/html/repo/percona-orchestrator-3.2.6-2.el8.x86_64.rpm
18. Инициализируем репозиторий
     createrepo /usr/share/nginx/html/repo/
19. Включаем просмотр каталогов в nginx. Добавляем в раздел location файла /etc/nginx/conf.d/default.conf добавляем директиву "autoindex on"
     server {
     listen       80;
     server_name  localhost;

     #access_log  /var/log/nginx/host.access.log  main;

     location / {
         root   /usr/share/nginx/html;
         index  index.html index.htm;
         autoindex on;
     }
20. Проверяем синтаксис nginx и перезагружаем nginx
     nginx -t
     nginx -s reload
21. Смотрим веб-страницу репозитория
     curl -a http://localhost/repo/ 
     
     <html>
     <head><title>Index of /repo/</title></head>
     <body>
     <h1>Index of /repo/</h1><hr><pre><a href="../">../</a>
     <a href="repodata/">repodata/</a>                                          18-Dec-2023 18:45                   -
     <a href="nginx-1.24.0-1.el8.ngx.x86_64.rpm">nginx-1.24.0-1.el8.ngx.x86_64.rpm</a>                  18-Dec-2023 18:43             2073052
     <a href="percona-orchestrator-3.2.6-2.el8.x86_64.rpm">percona-orchestrator-3.2.6-2.el8.x86_64.rpm</a>        16-Feb-2022 15:57             5222976
     </pre><hr></body>
     </html>
22. Добавляем в конфигурационный файл репозитария yum подключенние локальный репозиторий
     cat >> /etc/yum.repos.d/otus.repo << EOF
     [otus]
     name=otus-linux
     baseurl=http://localhost/repo
     gpgcheck=0
     enabled=1
     EOF
23. Проверяем наличие в репозитории yum наличие программ локального репозитория
     yum repolist enabled | grep otus
     yum list | grep otus
24. Проверяем работу репозтория установкой 
     yum install percona-orchestrator.x86_64 -y  
```
