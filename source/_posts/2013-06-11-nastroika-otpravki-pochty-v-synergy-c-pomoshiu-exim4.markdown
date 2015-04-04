---
layout: post
title: "Настройка отправки почты в Synergy c помошью Exim4"
date: 2013-06-11 23:16
comments: true
categories: [rails, exim4, estore]
---

Для того что бы свеже установленое преложение на рельсах, в моём случае это адаптированый вариант интернет магазина *Spree*, начало отправлть почту добавляем следующий код.
{% codeblock lang:ruby config/initializers/action_mailer.rb %}
ActionMailer::Base.delivery_method = :sendmail
ActionMailer::Base.sendmail_settings = {
  :arguments => '-r no-reply@spree-mail-example.com'
}
{% endcodeblock %}
после чего запустить приложение на рельсах следующей командой:
	rails s
	