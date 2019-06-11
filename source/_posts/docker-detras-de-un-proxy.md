---
title: Docker detrás de un proxy
categories:
  - blog/tecnologia/tips
id: 16
date: 2016-03-31 04:50:14
tags:
---

En el lugar donde trabajo hay instalado un proxy para bloquear cierto tráfico de internet y fuera de ser una pesadilla para navegar libremente, me dio problemas para comunicarme con el docker daemon en mi Mac. Curiosamente al correr 

docker ps -a

el daemon escuchaba sin problema, el problema surgió al intentar correr un archivo composedocker-compose up -d   
ERROR: Couldn't connect to Docker daemon - you might need to run \`docker-machine start default\`.
<!-- more -->

Intenté regenerar los certificados como sugerían ciertos comandos de docker pero no tuve éxito. El problema se debe a que mi host intenta conectarse al docker daemon pero la ip otorgada a la VM pasa primero por el proxy y por tanto no resuelve. Indagando en distintos sitios muchos sugerían quitar el proxy, lo cual en mi caso no es posible.

La solución es agregar la variable **NO\_PROXY** de entorno a la sesión con la ip de la docker machine.

export NO\_PROXY=$(docker-machine ip default)

 Listo, con eso mis problemas quedaron resuelto.

**Nota:** por alguna razón a veces es necesario reiniciar la Mac, prender la docker machine y exportar la variable.