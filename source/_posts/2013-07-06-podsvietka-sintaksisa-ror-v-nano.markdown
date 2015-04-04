---
layout: post
title: "Подсветка синтаксиса RoR в nano"
date: 2013-07-06 21:08
comments: true
categories: [ rails, nano ]
---
Если ты используеш один из *Linux* дистрибутивов и твой выбор редактора пал на текстовый редактор *nano* тогда для изучения и разработки *rails* проэктов будет полезно установить подсветку синтаксиса *Embeded Rybu .erb*.  

##INSTALLATION  
Необходимо [скачать](https://github.com/geomic/ERB-And-More-Code-Highlighting-for-nano/blob/master/erb_for_nano0.7.tar.gz) и разархивировать файл в домашний католог пользователя.   

	tar xzvf erb_for_nanoX.X.tar.gz

после этой команды уже можно пользоватся редактором с подсветкой синтаксиса *Embeded Rybu .erb*.
Но я рекомендовал бы переместить файл `erb.nanorc` из домашнего католога в `/usr/share/nano/`. Для этой операции вам потребуются права *root*!!! Итак преступим вам следует открыть текстовый редактор *nano* и изменить следующий текст  

	include "~/erb.nanorc"

на

	include "/usr/share/nano/erb.nanorc"


На этом всё, комфортного вам кодинга!