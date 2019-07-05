---
title: Incrementar historia de comandos en bash
date: 2014-12-31 03:10:45
categories:
  - linux
id: 11
tags:
  - bash
  - histsize
  - history
---

Si eres de los que utiliza **CTRL + R** para ejecutar un comando que ya habías ejecutado previamente o simplemente deseas buscar un comando en la historia de bash, quizá te resulte frustrante darte cuenta que el comando que querías ya no está en la memoria. Por defecto bash guarda un número limitado de comandos en la historia, en mi caso particular descubrí que OSX Yosemite conserva los últimos 500 comandos mientras que Ubuntu 12.04 preserva 1000. Por fortuna hay dos variables globales las cuales puedes editar para incrementar o disminuir el número de comandos almacenados en la historia: **HISTSIZE** y **HISTFILESIZE**.**HISTSIZE **indica cuantos comandos guarda bash en memoria durante la sesión activa. Por otro lado **HISTFILESIZE **indica cuantos comandos se pueden almacenar en el archivo **.bash\_history**. Para mayores detalles sobre estas variables puedes leer [este link.](http://stackoverflow.com/questions/19454837/bash-histsize-vs-histfilesize) 
<!-- more -->

Lo más conveniente es editar las variables y agregarlas a tu **.bash\_profile** (si no existe, créalo) para hacer válidos los cambios en sesiones de bash posteriores. Yo lo hice así:

```
#Increase bash history limit
HISTSIZE=2000
HISTFILESIZE=3000
```

Y para hacer efectivos los cambios en la sesión activa:

```
$ source ~.bash\_profile
```

Incluso puedes editar las variables para que se guarden los comandos de [forma infinita.](http://superuser.com/questions/479726/how-to-get-infinite-command-history-in-bash) Yo no quise arriesgarme porque uso mucho la consola y no quiero imaginar como crecerá esto, pero tú puedes arriesgarte ;)