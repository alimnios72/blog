---
title: Funciones Aws lambda con Nodejs
tags:
  - aws
  - lambda
  - nodejs
date: 2021-07-03 22:43:49
---


AWS lambda es un servicio que permite ejecutar programas sin necesidad de un servidor (serverless), es decir, puede ejecutar código con tan solo subirlo a AWS y estar listo para ser invocado posteriormente. AWS cobra por número de peticiones a la función lambda y por el tiempo de ejecución de la misma. Entre los lenguajes soportados en la funciones lambda están: Java, Python, Go, Ruby, .Net y Nodejs. Es de éste último del cual voy a hablar en este artículo.
<!-- more -->

AWS lambda tiene diferentes casos de uso, algunos de los más populares son la creación de micro-servicios y la creación de habilidades de Alexa. Yo he estado utilizando el servicio, entre otras cosas, para ejecutar tareas muy básica a ciertas horas del día. Lo más maravilloso es que su uso me sale completamente gratis ya que el [free tier](https://aws.amazon.com/lambda/pricing/) ofrece el 1er millón de peticiones y los primeros 400,000 GB-segundos completamente gratis cada mes.

A continuación decribiré como crear una simple función en AWS lambda:

1. En la consola de AWS busca lambda dentro de los servicios y en la lista de funciones da click en "Create".
2. Elije un nombre para tu función y el lenguaje de programación de tu preferencia. Yo elegí Node.js 14.x
{% asset_img lambda_basic.png Función Lambda %}
3. En la sección de permisos es recomendado dejar la opción por defecto la cual creará un nuevo rol con permisos básicos.
{% asset_img lambda_permissions.png Permisos Lambda %}
4. Por último en la sección de configuración avanzada también puedes dejar los valores por defecto.

Una vez creada la función podrás ver un editor de texto que contiene un folder con un archivo `index.js` (si elejiste Nodejs) y un programa tipo _"hola-mundo"_. Para probar el funcionamiento de la función, puedes hacer las modificaciones que desees al programa, guardar los cambios y posteriormente dar click en "Deploy" (o su equivalente en español). Es importante dar click en "Deploy" cada que necesites probar tus cambios porque de otra forma no servirá al ejecutar la prueba.

{% asset_img index_file.png Editor %}

Con los cambios desplegados es posible crear un test (o prueba) desde el menu superior del editor. Si notaste la función principal en el archivo `index.js`, ésta recibe un sólo argumento llamado `event` y cuando configuras un test puedes elegir diferentes objetos que serán asignados a `event` al momento de correr la prueba. Las funciones lambda pueden integrarse con varios otros servicios disponibles en AWS, es por esta razón que existen diferentes tipos de templates para ejecutar las pruebas, yo elegí el template más básico _helo-world_ y edité los valores por defecto. Para guardar el test sólo ve al final de la ventana desplegada y da click en "Create".

{% asset_img test_config.png Test %}

Para finalmente ejecutar el test recién creado da click en "Test" y una nueva pestaña se abrirá mostrando los resultados de ejecución. Si configuraste más de un test entonces ve a la flecha a un lado de "Test" y, dentro de la lista desplegable, elige el nombre del test que desees. El resultado muestra algunas cosas como: el objeto que la función principal regresa, logs (si imprimiste algo con `console.log` o equivalentes), duración efectiva de la ejecución y cuanto se ha cargado por duración y memoria del programa.

{% asset_img exec_results.png Resultados %}

El editor de texto en la página de la función es sumamente práctico para funciones sumamente sencillas pero las cosas se empiezan a complicar cuando tu programa tiene varios archivos como son todas aquellas dependencias (`node_modules`) que tu programa pudiera necesitar. Es importante mencionar que el runtime de Nodejs en lambda no incluye ninguna librería externa a excepción del paquete `aws-sdk` (puedes acceder directamente a éste con `require('aws-sdk')` sin necesidad de tenerlo en tus `node_modules`). En este tipo de situaciones es preferible trabajar localmente, crear un archivo zip de tu programa y subirlo a AWS.

Para subir un archivo zip con tu función puedes hacerlo manualmente desde el editor web y elegir "Upload from" -> ".zip file" o si tienes `aws-cli` instalado en tu computadora ejecutar un simple comando que subirá el zip por ti. A continuación muestro como crear y subir el zip con `aws-cli`:

- Primero crea el archivo zip con la herramienta que prefieras, yo use la terminal y el programa `zip`. Adicionalmente me cercioré que archivos _git_ no sean includios en el zip.
```
zip  -r function . -x '*.git*
```
- Después ejecuta el siguiente comando desde el mismo directorio donde creaste el zip. No olvides reemplazar `<funcion>` con el nombre de tu función.
```
aws lambda update-function-code --function-name <funcion> --zip-file fileb://function.zip
```

Esto es todo para este pequeño tutorial, en un siguiente artículo enseñaré como invocar funciones lambda con eventos de CloudWatch y como subscribir funciones a temas de SNS.
