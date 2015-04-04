---
comments: false
date: 2012-05-12 15:36:14
layout: post
slug: conky-for-ubuntu-12-04-lts
title: Conky for Ubuntu 12.04 LTS
wordpress_id: 364
categories:
- GNU/Linux
- Ubuntu desktop
tags:
- conky
- linux
- Ubuntu 12.04
---


### Conky это монитор для X Windows систем.

Он отображает различную системную информацию на рабочем столе. [Conky](../podrobnaya-nastrojka-conky) настраивается при помощи редактирования пользовательского конфигурационного файла `~/.conkyrc`. Рассмотрим один из таких файлов **Conky-Lua**, который украсит ваш рабочий стол и позволит контролировать некоторые параметры системы.

Если у вас не установлен Conky то установим его выполнив команду:



	
  1. `sudo apt-get install conky-all`

	
  2. Теперь скачаем сам файл конфигурации [тут](http://gnome-look.org/content/download.php?content=139024&id=1&tan=46022226).

	
  3. Распаковываем архив, а в нем выберем нужный для вашего дистрибутива архив и тоже распакуем его.

	
  4. Переходим в распакованный каталог. Переименовываем _«conkyrc»_ в _«.conkyrc»_ и перемещаем его в домашний каталог.

	
  5. Включает отображение скрытых файолов в Nautilus и создаем папку с именем _«.conky»_. Копируем из распакованного каталога картинку и файл _clock_rings.lua_ в _«.conky»_.

	
  6. Теперь открываем текстовым редактором файл _«.conkyrc»_ и заменяем строчку:
`lua_load /~.lua/scripts/clock_rings.lua`
на
`lua_load ~/.conky/clock_rings.lua`

	
  7. Запускаем conky. Для этого жмем Alt+F2 и вводим команду: conky

<!-- more -->
Для того что бы conky стартовала вместе с системой необходимо прописать в автозагрузку слдующию строку `sh -c "sleep 15 && conky" --pause 30`

Если вы подключены к интернету через wifi, то вам необходимо поменять в конфигурационом файле _~.conkyrc_ следующие строки:

```
${color FFFFFF}${goto 125}${voffset 25}${downspeed eth0}
${color FFFFFF}${goto 125}${upspeed eth0}
${color FF6600}${goto 125}Net

```
на
```
${color FFFFFF}${goto 125}${voffset 25}${downspeed wlan0}
${color FFFFFF}${goto 125}${upspeed wlan0}
${color FF6600}${goto 125}Net wifi

```



После, открываем следующий файл _~.conky/clock_rings.lua_ и в нём изменяем следующие строки: 

```
name='downspeedf',
arg='eth0',
```
и
```
name='upspeedf',
arg='eth0',
```
на
```
name='downspeedf',
arg='wlan0',
```
и
```
name='upspeedf',
arg='wlan0',
```


Это сработает если беспроводной сетевой интерфейс на компютере определяется как wlan0. Поэтому перед тем как вносить изменения в конфигурационые файлы потрудись посмотреть на вывод команды _ iwconfig_.


* * *


В следующей статье я раскажу [как настроить отображение погоды для твоего города](../nastrojka-otobrazhenyya-pohodyi-gismeteo-v-conky).
