---
title: No mostrar archivos ocultos en Filezilla
categories:
  - blog/tecnologia/redes
id: 8
date: 2014-07-18 16:08:51
tags:
---

Cuando se trata de subir o descargar archivos vía FTP de mi servidor siempre utilizo Filezilla para hacerlo. Ya sea que esté utilizando Windows o Linux, siempre tengo instalado el cliente FTP de Filezilla.Algunas veces resulta conveniente no mostrar los archivos o directorios ocultos de linux (archivos que comienzan en "."), puesto son archivos de configuración que normalmente no sincronizo con mi servidor y me dificulta encontrar el archivo deseado para subir/descargar. Si no mal recuerdo, versiones anteriores de Filezilla tenían una opción para lograr este cometido desde el menú ver, pero en la versión que tengo (3.5.3) no aparece dicha opción. Sin embargo sigue siendo posible ocultar dichos archivos siguiendo los siguientes pasos:

1.  Desde el menú "Ver", elige la opción "Filtros de nombre de archivo...".
2.  En el cuadro de diálogo que se abre, da click sobre el botón "Editar reglas de filtro..." (parte inferior izquierda).
3.  Crea un nuevo filtro desde el botón "Nuevo", puede nombrarlo "Archivos ocultos".
4.  En el panel de la derecha selecciona "empieza con", desde la lista de opciones, y en el cuadro de texto introduce el caracter "." (sin las comillas).
5.  Guarda los cambios dando click en "Ok", luego en la ventana anterior en "Aplicar" y "Ok" para cerrar.

Este filtro nuevo puedes utilizarlo en tu sistema de archivos tanto local como en tu servidor, simplemente palomea la opción que satisfaga tus necesidades.