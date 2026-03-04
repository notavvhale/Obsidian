Задачи:
1) Привязать комп к заббикс серверу.
2) Настроить проксю для заббикса.

==Сначала надо сделать прокси а потом подключать комп к серверу через прокси потому что в настройках агента надо указывать прокси.==

# Привязать комп к заббикс серверу.

1. Заходим на серв ищем версию
	   ![[Pasted image 20260202093740.png]]
	   ![[Pasted image 20260202093911.png]]
2. Заходим на https://www.zabbix.com/download_agents качаем agent ![[Pasted image 20260202094039.png]]
3. Устанавливаем и настраиваем agent ![[Pasted image 20260202094128.png]]![[Pasted image 20260202094206.png]]![[Pasted image 20260202094225.png]]!![[Pasted image 20260203131906.png]]Тут HOSTNAME пишем так как потом будем писать на сервере. В том же регистре. 10050 стандартный порт. Ставим галку add agent to PATH чтобы можно было из cmd проверить подключение. 
   При наличии прокси (как в нашем случае) ==ВЕЗДЕ== указываем ip прокси.
   Если что конфиг агента лежит по дефолту здесь - C:\Program Files\Zabbix Agent\zabbix_agentd.conf![[Pasted image 20260203100734.png]]![[Pasted image 20260203100747.png]]

# Создание прокси для заббикса

брал гайд отсюда - https://serverspace.ru/support/help/nastrojka-zabbix-dlya-raspredelennogo-monitoringa/?utm
1. Нужен серв linux. Я создал на ESXI виртуалку ubuntu server 22.04.3.![[Pasted image 20260202111148.png]]


2. Переходим по [ссылке](https://www.zabbix.com/download?zabbix=6.4&os_distribution=ubuntu&os_version=20.04&components=proxy&db=mysql&ws=) и по гайду копипастим команды.![[Pasted image 20260202111245.png]]Чтобы нормально копипастились команды можно подключиться по SSH.
   ip a - узнать ip в ubuntu
   в cmd - ssh username@ip_address
   Сейчас у меня так - ssh administrator@192.168.2.146![[Pasted image 20260202112230.png]]У меня была ошибка 
   "E: The repository 'file:/cdrom jammy Release' no longer has a Release file" 
   красная 
   и ещё два предупреждения
   https://repo.zabbix.com/zabbix/7.4/release/ubuntu focal
https://repo.zabbix.com/zabbix/7.4/stable/ubuntu focal

Я так понял что focal это название OS ubuntu. И при выборе версии 22.04 команды в гайде не менялись.

Сначала чистим то что накачал - `apt remove zabbix-release -y`

Потом делаем заново
`sudo -s`
`Zxc%6739`
`wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-2+ubuntu22.04_all.deb`
`dpkg -i zabbix-release_7.0-2+ubuntu22.04_all.deb`
`apt update`
 
 Дальше...
 Zabbix форсит установку MySQL и создания базы данных. Это можно обойти через SQLite.
 `apt install zabbix-proxy-sqlite3 zabbix-agent2 -y`
 Вроде как-то можно, но у меня не получилось.
 
 ==В итоге делал через MySQL.==
 
 ==Качаем устанавливаем MariaDB, потому что без этого не работает.==
 `apt install mariadb-server mariadb-client -y`
`systemctl enable mariadb`
`systemctl start mariadb`

==Создаём БД и настраиваем базу.==

`mysql`
`DROP DATABASE IF EXISTS zabbix_proxy;`
`DROP USER IF EXISTS 'zabbix'@'localhost';`
`CREATE DATABASE zabbix_proxy`
`CHARACTER SET utf8mb4`
`COLLATE utf8mb4_bin;`
`CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'ZabbixStrongPass123';`
`GRANT ALL PRIVILEGES ON zabbix_proxy.* TO 'zabbix'@'localhost';`
`FLUSH PRIVILEGES;`
`EXIT;`

==Устанавливаем заббикс прокси, агент и sql скрипты.==

`apt install zabbix-proxy-mysql zabbix-agent2 zabbix-sql-scripts -y`

==Импорт схемы бд от заббикса.==

systemctl stop zabbix-proxy
mysql -u zabbix -p zabbix_proxy < /usr/share/zabbix-sql-scripts/mysql/proxy.sql

==Запускаем службу и проверяем статус==
`systemctl start zabbix-proxy`
`systemctl status zabbix-proxy`

==Если всё ок -> меняем конфиг, ставим адрес сервера и название хоста.==

==Если не всё ок -> чатгпт сказал что какая-то багулина с этой версией ubuntu и надо дополнительно прописывать права.==

==Версия ubuntu 22.04==
`systemctl edit zabbix-proxy`

==Вставить:==
[Service]
Type=simple
ExecStart=
ExecStart=/usr/sbin/zabbix_proxy -f -c /etc/zabbix/zabbix_proxy.conf
RuntimeDirectory=zabbix
RuntimeDirectoryMode=0755

==Запуск PROXY==
`systemctl daemon-reexec`
`systemctl daemon-reload`
`systemctl enable zabbix-proxy`
`systemctl start zabbix-proxy`

==Проверяем статус==
`systemctl status zabbix-proxy`

==Редактируем конфиг==
nano /etc/zabbix/zabbix_proxy.conf

Hostname - имя прокси, то что надо писать на сервере с учётом регистра
Server - только ip сервера
DBName - имя базы данных
DBUser - имя пользователя в базе данных
DBPassword - пароль от базы данных

Zabbix-proxy служба с active checks выглядит так если всё работает
![[Pasted image 20260203132757.png]]Логи выглядят так
![[Pasted image 20260203132847.png]]
Во второй раз когда переустанавливал всё потому что случайно поломал скачав пакет postgresql всё заработало как надо. Без того что чатгпт сказал писать с этой версией то что типа багулина.