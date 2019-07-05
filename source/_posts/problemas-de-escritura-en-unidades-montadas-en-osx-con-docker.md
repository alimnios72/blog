---
title: Problemas de escritura en unidades montadas en OSX con Docker
date: 2016-03-31 02:00:34
categories:
  - docker
id: 15
tags:
  - docker
  - osx
  - brew
---

El día de hoy estuve lidiando con un problema que impedía a mi container docker escribir en un volumen montado desde mi Mac. Investigando, descubrí que fuera del usuario root (del container) otros usuarios no pueden escribir en el sistema host (OSX) como se indica [aquí](https://github.com/boot2docker/boot2docker/issues/581).En este caso particular el usuario _www-data_ de nginx intentaba escribir archivos de cache necesarios para las vistas volt del framework PHP phalcon (como en otros frameworks también). Algunas soluciones propuestas eran otorgar todos los permisos a esa carpeta, agregar al usuario creado al iniciar un container al grupo _www-data, _cambiar los permisos con los que Vbox monta los volúmenes compartidos, entre otras. Fuera de que algunas no me funcionaron, no quedé conforme hasta encontrar una solución más "elegante". 
<!-- more -->

Para mi sorpresa dicha solución está en los comentarios de ese mismo [issue](https://github.com/boot2docker/boot2docker/issues/581#issuecomment-153512609). Instalando este [script](https://github.com/adlogix/docker-machine-nfs) es posible montar todo el directorio /Users con los permisos correctos y el problema queda solucionado.

En resumen lo único que hice fue instalar el script usando brew 

```
brew install docker-machine-nfs

con la docker-machine corriendo corrí el script

docker-machine-nfs default
```

y por último agregué las variables de entorno a mi sesión

```
eval "$(docker-machine env default)"
```

Cabe mencionar que el nombre de mi máquina docker es **default. **Con estos sencillos pasos resolví un problema que llevaba horas atormentándome, espero les sea útil.