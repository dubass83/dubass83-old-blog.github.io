---
layout: post
title: "Установка и настройка Postgres на Debian 7"
date: 2013-07-07 10:54
comments: true
categories: [ postgres, debian, install ]
---
##Введение
{% pullquote %}
*PostgreSQL* является полнофункциональной объектно-реляционной системой управления базами данных. Она {" поддерживает большую часть стандартного SQL "} и предназначен для пользовательских расширений для многих аспектах. Некоторые из особенностей: **ACID-транзакции, внешние ключи, представления, последовательности, подзапросы, триггеры, пользовательские типы и функции, внешние объединения, контроль многоверсионность параллелизма**. Графические пользовательские интерфейсы и привязки для многих языков программирования также доступны.  
{% endpullquote %}
<!--more-->
##Установка базы данных PostgreSQL в Debian  

Если вы хотите установить базу даных *PostgreSQL*, а также консольный клиент для управления базами данных, тогда выполните следующую команду из оболочки.  
	#apt-get install postgresql postgresql-client

Эта команда установить пакет и все его зависимости. Если вы хотите проверить официальную документацию нажмите [здесь](http://www.postgresql.org/docs/).  

Несколько вопросов будут задавать сценарий настройки по ходу установки пакета. C большинством из них можно просто согласился.  

Каталог по умолчанию для базы данных находится по адресу  `/var/lib/postgres/`.   

Конфигурационый файл находится в `/etc/postgresql/postgresql.conf`.

##Создание пользователей для базы данных

Единственный (по умолчанию) способ войти во вновь созданную базу данных системы, это войти с правами администратора, затем войти как пользователь *Postgres*.  

	#su
	#su postgres
	#psql template1
	#template1=# \q
	#exit

Тем не менее, вы, вероятно, хотели, чтобы иметь возможность войти в систему как ваш пользователь *UNIX*, или любым другим пользователем на ваш выбор. Есть несколько способов настроить эти доступы, для этого надо отредактировать файл `/etc/postgresql/pg_hba.conf` файл (полная документация этого файл доступен [здесь](http://www.postgresql.org/docs/7.4/static/client-authentication.html)).  

{% codeblock pg_hba.conf lang:sh %}
local all postgres ident sameuser
#
# All other connections by UNIX sockets
local all all ident sameuser
#
# All IPv4 connections from localhost
host all all 127.0.0.1 255.255.255.255 md5
# All IPv6 localhost connections
host all all ::1 ffff:ffff:ffff:ffff:ffff:ffff:ffff:ffff ident sameuser
host all all ::ffff:127.0.0.1/128 ident sameuser
#
# reject all other connection attempts
host all all 0.0.0.0 0.0.0.0 reject
{% endcodeblock %}

Для редоктирования этого файла необходимо обладать правами супер пользователя, позже мы еще вернёмся к редактированию этого файла. Сейчас мы сказали *PostgreSQL* то, что все пользователи на локальной машине, должны иметь возможность доступа ко всем базам данных, используя свой ​​логин и *UNIX* пароли. Вам необходимо перезагрузить вашу базу данных, чтобы изменения вступили в силу.  

	#/etc/init.d/postgresql restart

Теперь проблема в том, что авторизованные пользователи могут войти в базу данных, если они также существуют в базе данных. Так что вам нужно создать учетные записи с их именах пользователей. Это делается путем ввода с терминала команды `CreateUser` от имени пользователя *Postgres*.  

	#su postgres
	#createuser firstuser
	#exit

В ходе выполнения команды `CreateUser` будет задан вопрос, если пользователь должен иметь возможность для создания баз данных или создания новых пользователей.  

Теперь у вас есть новый пользователь в базе данных, и вы должны иметь возможность подключиться от имени этого пользователя (назовем его *firstuser*) следующим образом:  

	#su - firstuser
	#firstuser@localhost:~$ psql -W template1
	#template1=# \q
	#exit

**-W** параметр запрос на ввод пароля. Вы должны ввести пароль вашей системы.
Так вот оно что, мы закончили первую часть этой установки. Теперь давайте приготовим это для Интернета.

Если некоторые из ваших веб-скриптов понадобится подключение к базе данных, вам необходимо предоставить им доступ через некоторое нового пользователя, зарезервированных для этого веб-приложения.

Давайте сначала создать нового пользователя (от имени пользователя *Postgres*). Назовем его *webuser*.  

	#su postgres
	#createuser webuser
	#exit 

Теперь дадим ему доступ к одному или нескольким базам данных (скажем, вы позволите ему получить доступ к базам данных, которые зовут *Web*). Чтобы сделать это, вам нужно отредактировать еще раз файл *pg_hba.conf*. Получение первой строке ниже и записи второй раз под первым.

	host all all 127.0.0.1 255.255.255.255 md5
	host web webuser 127.0.0.1 255.255.255.255 md5

Если вы хотели бы вы предоставить доступ этому пользователю с любого компьютера в подсети 192.168.0.xxx, вам придется добавить следующую строку.

	host web webuser 192.168.0.1 255.255.255.0 md5

Вы должны предоставить ему доступ к хосту, вероятно, вы будете использовать протокол *TCP/IP*, чтобы получить подключение к базе данных. Но, так как вы дали ему тип аутентификации MD5, вы должны дать ему пароль, а также. Для того, чтобы сделать это, вам необходимо подключиться к базе данных и выдает специальную команду, все как пользователь Postgres:  

	#su postgres
	#postgres@localhost:~$ psql template1
	#template1=# alter user webuser password 'some_password';
	#ALTER USER
	#template1=# \q
	#exit

Теперь пользователь должен задать хороший пароль *'some_password'*, которые можно использовать после перезапуска PostgreSQL, чтобы сахранить и запомнить изменения в *pg_hba.conf* нужно перезапустить сервер.  

	#/etc/init.d/postgresql restart

После этого вы должны быть в состоянии создать свою собственную базу данных, если вы дали ему на это разрешение.  

	#createdb -h localhost -U webuser -W web

Подключиться к этой вновь созданной базы данных можно с помощью следующей команды.  

	#psql -h localhost -U webuser -W web

Как вы могли заметить, мы используем параметр *-h localhost* здесь. Это должно заставить PostgreSQL клиента использовать *TCP/IP*, а не *UNIX* сокетов. Если вы не хотите есть, вам нужно добавить новую строку в *"local"* категории *pg_hba.conf* для пользователей *webuser* для подключения через *sockets*. 

	local web webuser md5

##Использование SQL

Важные команды которые вам может потребоваться знать,

	#psql -h localhost -U webuser -W -l


Выдаст вам список доступных баз данных.  

Теперь подключитесь к базе данных *PostgreSQL* с помощью клиента *psql*, и давайте создадим небольшую таблицу, сразу после нескольких тестов.  

{% codeblock lang:sql %}
web=> SELECT version();
web=> SELECT current_date;
web=> SELECT 2+2;
web=> \h
web=> \d
web=> \dt
web=> CREATE TABLE weather(
web=> city varchar(80),
web=> temp_lo int, -- low temperature
web=> temp_hi int, -- high temperature
web=> prcp real, -- precipitation
web=> date date
web=> );
web=> INSERT INTO weather (city, temp_lo, temp_hi, prcp, date)
web=> VALUES ('San Francisco', 43, 57, 0.0, '1994-11-29');
web=> INSERT INTO weather (date, city, temp_hi, temp_lo)
web=> VALUES ('1994-11-29', 'Hayward', 54, 37);
web=> SELECT city, temp_lo as 'Lowest Temperature', temp_hi as 'Highest Temperature' FROM weather;
web=> SELECT city FROM weather WHERE temp_lo < 40;
web=> SELECT max(temp_lo) FROM weather;
web=> UPDATE weather SET temp_lo = 41 WHERE city = 'Hayward';
web=> SELECT * FROM weather;
web=> DELETE FROM weather WHERE prcp IS NULL;
web=> SELECT * FROM weather;
web=> \q

{% endcodeblock %}

Теперь у вас был хороший обзор из нескольких операторов *SQL* и как использовать их в *PostgreSQL*.  
[Оригинальная статья на англиском](http://www.debianhelp.co.uk/postgresql.htm).