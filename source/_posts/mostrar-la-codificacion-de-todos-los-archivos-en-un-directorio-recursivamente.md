---
title: Mostrar la codificación de todos los archivos en un directorio recursivamente
date: 2014-07-18 19:36:06
categories:
  - linux
id: 9
tags:
  - linux
  - codificacion
  - find
---

Si por alguna razón tienes la necesidad de mostrar la codificación (encoding) de todos los archivos dentro de un directorio y subdirectorios, el comando **find** de Linux resulta de gran ayuda. Supongamos que deseas mostrar la codificación de todos los archivos _.php_ dentro del directorio actual: `$ find . -type f -name \\\*.php -exec file -i {} \\;`

 A continuación explico cada uno de los parámetros utilizados:
<!-- more -->

*   El punto "." hace referencia al directorio actual, si quisieras correr el comando sobre otro directorio tendrías que cambiar el punto por el directorio que desees.
*   **\-type f** limita la búsqueda a sólo archivos, no directorios.
*   **\-name \\\*.php** devuelve todos los archivos que terminan en _.php_
*   **\-exec file -i {} \\; **la opción _\-exec_ de **find** ejecuta el comando que se encuentra a continuación, en este caso es el comando **file**. _\-i_ es el parámetro de **file** que nos muestra tanto el tipo (mime type) del archivo como su codificación (encoding) y los corchetes _{}_ hacen referencia a cada uno de los archivos encontrados. Por último _\\;_ sirve para que el comando se ejecute para cada uno de los archivos uno por uno.

Como vimos con este ejemplo, **find** tiene un alto potencial para ejecutar tareas que requieran navegar recursivamente en un directorio. Así como en este caso se utilizó **file** para mostrar los datos del archivo, la opción _\-exec_ puede servirnos para muchas otras tareas.