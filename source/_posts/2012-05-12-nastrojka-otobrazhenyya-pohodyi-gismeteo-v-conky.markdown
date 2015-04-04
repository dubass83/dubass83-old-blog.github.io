---
comments: false
date: 2012-05-12 23:18:36
layout: post
slug: nastrojka-otobrazhenyya-pohodyi-gismeteo-v-conky
title: Настройка отображения погоды Gismeteo в Conky
wordpress_id: 410
categories:
- GNU/Linux
- Ubuntu desktop
tags:
- bash script
- conky
- linux
---

### Настройка вывода информации о погоде в conky.

Я не стал заморачиваться с RSS (он у меня как-то криво работал) и немного погуглив остановился на следуещем варианте. **Скрипт для отображения погоды в Conky с gismeteo.**
  
  
  


Итак, научим Conky выдирать данные о погоде из этого XML - http://informer.gismeteo.ru/getcode/xml.php?id=27612 . Вам нужно получить ссылку для своего города. Для Москвы это http://informer.gismeteo.ru/xml/27612_1.xml .
  
<!-- more -->  

Теперь создайте файл conky-weather.sh с таким содержимым:

```
#!/bin/sh

time=$(date +%H)

URL="http://informer.gismeteo.ru/xml/33614_1.xml"
EXEC="/usr/bin/curl -s"

a=`$EXEC $URL`

#температура
echo $a | tr "/>" "\n" | grep HEAT | sed -n 1p | sed -e 's/<HEAT //' |\
sed -e 's/"//g' | tr -d "min=" | tr -d "max=" | sed -e 's/ //' |\
sed -e 's/ /../' | gawk '{ print "Темп.",$1,"°C" }'
#ветер
echo $a | tr "/>" "\n" | grep WIND | sed -n 1p | sed -e 's/<WIND //' |\
sed -e 's/"//g' | tr -d "min=" | tr -d "max=" | sed -e 's/ //' |\
gawk '{ print "Ветер",($1+$2)/2,"м/с"}'
#давление
echo $a | tr "/>" "\n" | grep PRESSURE | sed -n 1p | sed -e 's/<PRESSURE //' |\
sed -e 's/"//g' | tr -d "min=" | tr -d "max=" | sed -e 's/ //' |\
gawk '{ print "Давл.",($1+$2)/2,"мм"}'



```
    




URL="http://informer.gismeteo.ru/xml/27719_1.xml" замените на ссылку для вашего города. Проверить работоспособность скрипта можно, выполнив его в терминале:

```       
$ ./conky-weather.sh 
Темп. 14..16 °C
Ветер 3 м/с
Давл. 754 мм

```    


Если все работает, то добавьте в ваш ~/.conkyrc следующие строки:

```    
${color a1ccea}Weather:
${color ffffff}${execi 3600 /путь/до/скрипта/conky-weather.sh}

```    


Вот и все. 


* * *


Этот скрипт встретился мне на странице сайта [http://zenux.ru](http://zenux.ru/articles/8/)
