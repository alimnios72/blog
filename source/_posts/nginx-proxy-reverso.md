---
title: Proxy reverso con Nginx
date: 2022-04-22 13:58:42
tags:
  - servers
  - nginx
  - linux
---

Para un proyecto de trabajo necesitaba configurar un servidor que pudiera aceptar peticiones http y websockets al mismo tiempo, así que me di a la tarea de investigar como hacerlo y a continuación describo el proceso que llevé a cabo para lograr dicha tarea.
<!-- more -->

Primero, creé una nueva instancia de un servidor EC2 con Ubuntu instalado en AWS, sin embargo cualquier sistema operativo que soporte nginx debería funcionar para lo mismo. Una vez que instalé `nginx` en Ubuntu (para más detalles de la instalación puedes seguir este [tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04)) deshabilité el sitio por defecto y agregué un nuevo archivo de configuración en el directorio `/etc/nginx/sites-available/` (esta ruta puede ser diferente en otros sistemas operativos):

```
sudo rm /etc/nginx/sites/enabled/default
sudo vim /etc/nginx/sites-available/http-web-socket-proxy
```

A continuación, agregué la siguiente configuración al archivo `http-web-socket-proxy`:

```
server {
    location /ws/ {
        proxy_pass http://<servidor-websocket>:puerto;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_set_header Host $host;
    }

    location / {
        proxy_pass http://<servidor-http>:puerto;
        proxy_set_header Host $host;
    }
}
```

Algunos detalles sobre la configuración son:

- `location <ruta>`: todas las configuraciones dentro de este bloque aplican para la ruta especificada. Por ejemplo, si el servidor tiene ip 54.562.485.4, todas las conexiones a http://54.562.485.4/ws son interpretadas por el primer bloque de configuración mientras que http://54.562.485.4/ van al segundo bloque.
- `proxy_pass <servidor>`: envia las peticiones al servidor especificado en la directiva. Por ejemplo, si tengo un servidor de websockets corriendo en el mismo servidor Ubuntu quedaría algo como `proxy_pass: http://localhost:3000`
- `proxy_http_version 1.1`: indica la versión de http a utilizar para el proxy reverso.
- `proxy_set_header <variable> <valor>`: establece un valor para el header indicado. Por ejemplo, `proxy_set_header Host $host;` envia la dirección del host al servidor establecido en `proxy_pass`. En el caso del servidor de websocket es necesario agregar los header "upgrade" indicados arriba, de esta forma un cliente de websockets sabe que el servidor soporta websockets y debe cambiar el protocolo al momento de la conexión.

Finalmente, es necesario agregar el nuevo vhost (creando un link simbólico) y reiniciar `nginx`:

```
sudo ln -s /etc/nginx/sites-available/http-web-socket-proxy /etc/nginx/sites-enabled/
sudo systemctl reload nginx
```

Si todo salió bien, tu servidor debería poder responder a peticiones http y websockets. En otro post explicaré cómo crear un servidor http y otro websocket en nodejs para que puedas probar tu proxy reverso.

