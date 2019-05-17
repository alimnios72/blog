---
title: Router Keygen para Android
categories:
  - blog/tecnologia/apps
id: 2
date: 2012-02-17 19:08:33
tags:
---

Me di a la tarea de instalar aircrack-ng en mi dispositivo android, mi búsqueda no duró mucho debido a la poca información al respecto existente en la web. Sinceramente no tengo mucho tiempo para profundizar en la investigación sobre como lograr correr aircrack en android, lo que de alguna forma debería ser posible porque android está basado en Linux ,por otro lado no creo que algún  celular o tablet  tenga una tarjeta inalámbrica preparada para la inyección de paquetes. Es entonces cuando gracias a un compañero del tabajo, conocí la aplicación Router Keygen de la cual voy a platicarles.

Puede resultar frustrante, para aquellos como yo quienes no contamos con un plan de datos, querer navegar en internet desde su dispositivo estando en alguna ubicación donde  haya muchos AP's pero con algún tipo de seguridad activada. Y es que hace algunos años los modems de Telmex venían sin seguridad alguna y uno podía libremente "tomar prestado el internet del vecino", al día de hoy son mínimos aquellos modems sin seguridad. Ahora por defecto telmex distribuye routers con por lo menos WEP habilitada evitando, de esta manera, al usuario promedio conectarse gratuitamente.

Router keygen es una aplicación para android que no crackea WEP, sin embargo se basa en diccionarios de claves hexadecimales e intenta "adivinar" la clave WEP de los routers. La aplicación no pretende ser la panacea, de hecho no  funciona con WPA  o derivados y en mi experiencia no es capaz de descifrar claves de routers cuyo SSID no es el que viene por defecto.  Pese a lo anterior vale la pena probarla porque en la Ciudad de México hay gran cantidad de routers configurados con los valores de fabrica , tal como los entrega el proveedor de servicio, y por ende son presa fácil.

Router keygen no está en el android market (las malas lenguas cuentan que alguna vez lo estuvo) pero puede descargarse el apk desde su servidor de archivos/blog/foro favorito. Yo hice la descarga en este [sitio](http://androidzone.org/2011/10/descarga-router-keygen-2-8-2-en-tu-android-descrifra-claves-wifi-y-navega-gratis-apk/).

El manejo de la aplicación es muy intuitivo, los AP's marcados en verde son las posibles víctimas mientras los marcados en rojo seguramente utilicen WPA o por alguna razón no es posible descifrar su clave. Seleccionar alguna de las redes en verde nos despliega una clave como lo muestra la imagen. Esta clave deberemos introducirla en el correspondiente AP des de la configuración de redes inalámbricas de nuestro dispositivo android.

![Router keygen android](images/blog/tecnologia/apps/router-keygen-3-263x468.jpg)

Si presionamos el botón de opción dentro de la aplicación encontramos la configuración de la aplicación, las opciones son por sí solas explicativas. La opción de utilizar redes 3G para los diccionarios proporciona una mayor probabilidad de éxito puesto que utiliza dichos diccionarios para intentar generar las claves. También es posible descargar estos diccionarios por separado y configurar la ruta donde se encuentren los mismos.  
Espero la aplicación les sea de gran utilidad como lo ha sido para mí, describan sus experiencias y/o sugerencias sobre aplicaciones similares en la sección de comentarios.