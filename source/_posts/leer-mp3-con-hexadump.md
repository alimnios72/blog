---
title: Leer mp3 con hexdump
date: 2014-07-09 02:27:24
categories:
  - linux
id: 4
tags:
  - hexdump
  - bash
  - linux
  - mp3
---

Con el fin de comprender a mayor detalle la estructura interna de un archivo mp3 (técnicamente me refiero al estándar MPEG-1 capa 3), me di a la tarea de investigar una forma sencilla de abrir el fichero para ver su contenido.Lo primero que vino a mi mente fue utilizar un editor de textos tradicional (gedit, notepad, notepad++, emacs, etc), algunos de ellos lograrán mostrar el contenido binario del archivo pero demora mucho tiempo y no es eficiente hacerlo de esta manera. Fue entonces cuando _google_ vino al rescate y los resultados sugirieron utilizar el programa **hexdump** para sistemas operativos linux, así que lo instalé y me dediqué a hacer pruebas para entender el funcionamiento del mismo.
<!-- more -->

**hexdump** es sumamente fácil de utilizar, tan sólo basta con escribir el nombre del programa y el fichero que se desea leer:

$ hexdump song.mp3

El programa mostrará todo el contenido del fichero en formato hexadecimal, la primera columna corresponde al _offset_ (posición donde comienzan los valores) y las 8 columnas restantes son los valores representados de forma hexadecimal. Cada 2 caracteres corresponden a un byte, por tanto cada línea muestra 16 bytes del archivo.

0000000 4449 0333 0000 0000 6614 4154 424c 0000
0000010 2100 0000 ff01 48fe 7500 6d00 6100 6e00
0000020 2000 4100 6600 7400 6500 7200 2000 4100
0000030 6c00 6c00 5400 4550 0031 0000 0015 0100

El símbolo **\*** indica que los mismos valores se repiten hasta la posición marcada por el offset.

0000310 3044 0000 0000 0000 0000 0000 0000 0000
0000320 0000 0000 0000 0000 0000 0000 0000 0000
\*
00003e8

Para mostrar un resultado más amigable al humano basta con utilizar la opción **\-C** en el programa con lo cual, además de ver separados byte por byte (16 columnas), observamos una última columna donde se representa el valor con texto.

$ hexdump -C song.mp3

00000000  49 44 33 03 00 00 00 00  14 66 54 41 4c 42 00 00  |ID3......fTALB..|
00000010  00 21 00 00 01 ff fe 48  00 75 00 6d 00 61 00 6e  |.!.....H.u.m.a.n|
00000020  00 20 00 41 00 66 00 74  00 65 00 72 00 20 00 41  |. .A.f.t.e.r. .A|

Otras dos opciones que son de gran utilidad son **-n** y **\-s.** La primera indica el número de bytes que serán mostrados y la segunda indica a partir de que byte (contando desde el inicio) se mostrará el contenido. Es decir, si deseo mostrar sólo 10 bytes del archivo comenzando desde el byte 20 el comando sería el siguiente:

$ hexdump -s 20 -n 10 -C song.mp3

00000014  01 ff fe 48 00 75 00 6d  00 61                    |...H.u.m.a|
0000001e

Si tenemos algunos conocimientos acerca de la estructura interna de un archivo mp3 podemos observar que al inicio se encuentra la etiqueta **ID3** del archivo, en este caso se trata de una etiqueta ID3 v2.x puesto inicia con los caracteres "ID3" (para mayor información consulte las especificaciones técnicas de las etiquetas ID3 en este [sitio](http://id3.org/ "ID3")). Para aquellos que desconocen las etiquetas, éstas son los metadatos que puede llevar un archivo mp3 y almacenan información como el nombre del artista, nombre del álbum, nombre de la canción, género, compositor, año, entre muchos otros. La versión 1 de ID3 estaba limitada a tan sólo 128 bytes de información con cada uno de sus campos fijos y para la versión 2 se realizaron varias modificaciones utilizando campos dinámicos y permitiendo almacenar muchos más contenido. Como dato cultural: el contenido de las etiquetas mp3 permiten a los reproductores los datos del archivo mp3 a través del display.

El encabezado de una etiqueta ID3 v2.x (2.2, 2.3 y 2.4) consta de 10 bytes como lo muestra la siguiente tabla:

![ID3 header](images/blog/tecnologia/linux/id3_header.png)

Podemos conocer el tamaño total de la etiqueta leyendo y decoficando los últimos 4 bytes del encabezado. Si deseas conocer como decodificar el tamaño de la etiqueta puedes referirte a [este sitio](http://phoxis.org/2010/05/08/what-are-id3-tags-all-about/#unsyncsafe). Inmediatamente enseguida del encabezado del ID3, encontraremos los distintos _frames_ que corresponden a los metadatos de la canción, el orden y el tamaño de los mismos varia de canción en canción, sin embargo el encabezado de cada frame es fijo para cada versión de etiqueta ID3 v2.x.

![ID3 frame header](images/blog/tecnologia/linux/id3_frame_header.png)

Para el caso particular del archivo mp3 mostrado como ejemplo en este artículo, el tamaño de su etiqueta ID3 es de 2,672 bytes. Este valor se obtuvo a partir de la interpretación y decodificación del encabezado ID3 como se mencionó anteriormente. Conociendo el tamaño de la etiqueta podemos utilizar **hexdump** para leer la información pertinente a la estructura del mp3 y ya no a sus metadatos (etiqueta ID3). El comando para leer los primeros 48 bytes correspondientes al primer frame del fichero es: 

$ hexdump -s 2672 -n 48 -C song.mp3

00000a70  ff fb e4 64 00 0f f0 00  00 66 00 00 00 08 00 00  |...d.....f......|
00000a80  0c c0 00 00 01 00 00 01  98 00 00 00 20 00 00 33  |............ ..3|
00000a90  00 00 00 04 4c 41 4d 45  33 2e 39 39 20 28 61 6c  |....LAME3.99 (al|

Si tomamos los primeros 4 bytes del resultado anterior y los convertimos a su representación binaria (32 bits) podemos interpretar el encabezado del primer frame del archivo mp3 tal como lo muestra este [artículo](http://www.codeproject.com/Articles/8295/MPEG-Audio-Frame-Header) y corroborar que se trata de un frame de audio mp3 como se muestra a continuación:

11111111111 11 01 11110010001100100

Los primeros 11 bits están encendidos para indicar el comiendo de un frame mp3, los siguientes 2 bits corresponden a la versión de audio que en este caso es MPEG versión 1 y los siguientes 2 bits consecutivos indican que se trata de un archivo capa 3. Con esto comprobamos que efectivamente estamos lidiando con un archivo mp3. Dejo al lector profundizar más en la decodificación de cada frame mp3 si así lo desea.

Este artículo pretende dar una breve introducción de la estructura interna de un archivo mp3 así como apreciar el uso de la herramienta hexdump para este tipo de tareas, en este caso trabajamos con un tipo específico de archivo pero la herramienta sirve para leer diferentes archivos binarios.