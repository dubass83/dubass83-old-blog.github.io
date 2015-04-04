---
layout: post
title: "Добавление нового пользователя в MySQl"
date: 2013-06-09 12:51
comments: true
categories: [MySQL, Debian] 
---

 С настройкой *Mysql-сервера* я сталкиваюсь нечасто, в основном во время того когда занимаюсь разроботкой и тестированием нового *WEB-приложения*. Поэтому иногда требуется добавить нового пользователя на базу. Устанавливать для этих целей web-интерфейс не целесообразно, а через командную строку запоминать лень. Поэтому для лентяев как я следующий код:

{% codeblock  lang:sql %}
mysql>GRANT ALL PRIVILEGES ON testbase.* TO testuser@localhost IDENTIFIED BY 'testpass';
mysql>flush privileges;
{% endcodeblock %}


Если есть время разобратся и выучить что то новое тогда лучше посетить [сайт с официальной документацией](http://www.mysql.ru/docs/man/Adding_users.html). 

<!--more-->
 {% codeblock [Добавление новых пользователей, используя команду GRANT] lang:sql  %}
shell> mysql --user=root mysql
mysql> GRANT ALL PRIVILEGES ON *.* TO monty@localhost
    ->     IDENTIFIED BY 'some_pass' WITH GRANT OPTION;
mysql> GRANT ALL PRIVILEGES ON *.* TO monty@"%"
        -> IDENTIFIED BY 'some_pass' WITH GRANT OPTION;
mysql> GRANT RELOAD,PROCESS ON *.* TO admin@localhost;
mysql> GRANT USAGE ON *.* TO dummy@localhost;

{% endcodeblock %}

###Эти команды GRANT создают трех новых пользователей:

**monty**  
Полноценный суперпользователь - он может подсоединяться к серверу откуда угодно, но должен использовать для этого пароль some_pass. Обратите внимание на то, что мы должны применить операторы GRANT как для monty@localhost, так и для monty@"%". Если не добавить запись с localhost, запись анонимного пользователя для localhost, которая создается при помощи mysql_install_db, будет иметь преимущество при подсоединении с локального компьютера, так как в ней указано более определенное значение для поля Host, и она расположена раньше в таблице user.  
**admin**  
Пользователь, который может подсоединяться с localhost без пароля; ему назначены административные привилегии RELOAD и PROCESS. Эти привилегии позволяют пользователю запускать команды mysqladmin reload, mysqladmin refresh и mysqladmin flush-*, а также mysqladmin processlist. Ему не назначено никаких привилегий, относящихся к базам данных (их можно назначить позже, дополнительно применив оператор GRANT).   
**dummy**  
Пользователь, который может подсоединяться к серверу без пароля, но только с локального компьютера. Все глобальные привилегии установлены в значение 'N'-тип привилегии USAGE, который позволяет создавать пользователей без привилегий. Предполагается, что относящиеся к базам данных привилегии будут назначены позже.  

{% codeblock [Предоставление привилегий пользователям при помощи оператора GRANT] lang:sql %}

shell> mysql --user=root mysql
mysql> GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP
    -> 	   ON bankaccount.*
    -> 	   TO custom@localhost
    ->     IDENTIFIED BY 'stupid';
mysql> GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP
    ->     ON expenses.*
    ->     TO custom@whitehouse.gov
    ->     IDENTIFIED BY 'stupid';
mysql> GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP
    ->     ON customer.*
    ->     TO custom@'%'
    ->     IDENTIFIED BY 'stupid';
{% endcodeblock %}

Привилегии для пользователя **custom** мы назначаем потому, что этот пользователь хочет получать доступ к *MySQL* как с локального компьютера через сокеты *Unix*, так и с удаленного компьютера whitehouse.gov через протокол TCP/IP. 