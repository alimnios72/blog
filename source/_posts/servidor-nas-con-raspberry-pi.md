---
title: Servidor NAS con Raspberry Pi
date: 2021-05-18 18:46:37
categories:
  - raspberry
tags:
  - raspberry
  - tutoriales
keywords:
  - ntfs
  - nas
  - raspberry
  - respaldos
  - dropbox
  - onedrive
  - compartir archivos
  - red local
  - tutorial
  - mac
---

Desde hace unos años tengo una rapberry pi (modelo 3B) que había utilizado mayormente para jugar emuladores de juegos retro con [recalbox](https://www.recalbox.com), sin embargo, últimamente le he sacado un provecho más técnico utilizándola para bloquear anuncios en mi red local con [pi-hole](https://pi-hole.net) (más adelante haré un tutorial acerca de esto) y también como servidor NAS. 

NAS (Network Attached Storage, por sus siglas en inglés) es básicamente una unidad de almacenamiento que está disponible por medio de una red. En términos generales, un NAS pudiera compararse con algún servicio de almacenamiento en la nube como Dropbox o Onedrive, donde puedes guardar respaldos de tus datos y compartir archivos con otras personas, y su instalación/configuración puede ser tan compleja o sencilla dependiendo del caso de uso. En mi caso específico de uso, los únicos requerimientos son tener un lugar donde respaldar mis archivos y poder compartir los mismos dentro de la red local de mi hogar.
<!-- more -->

Para configurar el NAS en mi red local sólo me bastó con mi rasberry pi, un disco duro externo que no usaba y acceso físico a mi router de internet. La instalación física es sumamente sencilla, basta con conectar la raspberry pi al router, de preferencia vía ethernet, y la unidad de disco duro a la raspberry pi con un cable USB. La configuración e instalación del software necesario es un poco más complicado pero nada del otro mundo, a continuación describo los pasos que seguí.

### Instalar raspbian en la rapsberry pi

Con algunos conocimientos básicos sobre sistemas operativos es sumamente fácil instalar _raspbian_ en la raspberry pi. En esté artículo no pretendo hablar del proceso de instalación de rapsbian pero dejo algunos artículos que detallan dicho proceso:

- https://geekland.eu/instalar-raspbian-con-raspberry-pi-imager/
- https://raspberryparanovatos.com/tutoriales/instalar-raspbian-en-raspberry-pi-noobs/

### Montar la unidad de disco

Una vez conectado el disco duro a la raspberry pi es necesario acceder a raspbian via ssh, el usario y la ip varian dependiendo de los pasos seguidos en la instalación pero el comando para conectarse es algo como lo siguiente:

```sh
$ ssh pi@192.168.1.3
```

El siguiente paso es identificar la ubicación del disco duro otorgada por el sistema operativo, para esto se puede utilizar el siguiente comando:

```sh
sudo fdisk -l
```

Lo cual, entre otras cosas, muestra algo como lo siguiente:

```sh
Disk /dev/sda: 931.5 GiB, 1000204886016 bytes, 1953525168 sectors
Disk model: xternal HD
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xcbce2081

Device     Boot Start        End    Sectors   Size Id Type
/dev/sda1          63 1953520064 1953520002 931.5G  7 HPFS/NTFS/exFAT
```

En el ejemplo anterior pude identificar mi disco porque sé que el sistema de archivos es NTFS y la capacidad es un 1TB (equivalente a unos 931.5 gb como se muestra). La parte importante a identificar es la cadena `/dev/sda1` que será utilizada posteriormente para montar la unidad.

Cabe mencionar que el sistema de archivos de mi disco externo es NTFS (la mayoría de los discos externos vienen con ese formato de fábrica) y que raspbian necesita instalar paquetes adicionales para poder escribir sobre el disco, si no instalan los paquetes correspondientes sólo se podrán leer archivos del disco pero no guardar nuevos archivos. Para solucionar éste inconveniente es necesario instalar la librería `ntfs-3g` y se puede hacer de la siguiente forma:

```sh
sudo apt-get install ntfs-3g
```

A continuación es necesario crear el directorio en donde se montará la unidad de disco. Algunos sistemas basados en linux designan el directorio `/media` para montar unidades externas mientras que otros lo hacen en `/mnt`, por lo que leí en algunos foros la preferencia para sistemas raspbian es utilizar el directorio `/mnt` y en mi caso eso fue lo que hice. Con el siguiente comando creé un nuevo directorio:

```sh
sudo mkdir -p /mnt/MiDisco
```

Y posteriormente monté la unidad de disco en dicho directorio con:

```sh
sudo mount /dev/sda1 /mnt/MiDisco/
```

A este punto, si todo salió bien, deberías poder listar los archivos/directorios de tu disco duro.

```sh
ls -la /mnt/MiDisco
```

Ahora, si pretendes conectar y desconectar temporalmente el disco duro de la raspberry pi (por ejemplo, si es un disco portátil que llevas de un lado a otro) es sumamente recomendado desmontar la unidad antes de una desconexión física, para esto puedes correr el siguiente comando:

```sh
sudo umount /dev/sda1
```

Cada vez que necesites montar y desmontar la unidad puedes utilizar los comandos anteriores.

Si la idea es dejar el disco duro permanentemente conectado a la raspberry es recomendable editar `/etc/fstab` agregando los datos del disco y esto montará el disco de forma automática incluso después de un reinicio en el sistema operativo. Con tu editor de texto favorito, edita el archivo mencionado anteriormente:

```sh
sudo vim /etc/fstab
```

Y agrega la siguiente línea al final del archivo.

```
/dev/sda1 /mnt/MiDisco ntfs-3g user,noauto 0 0
```

Asegúrate de reemplazar lo anterior con las ubicaciones correctas para tu caso específico.

### Instalar y configurar samba

Samba es un software que re-implementa el protocolo de redes SMB. Samba permite compartir archivos e impresoras entre computadoras que corran Windows o ciertas variantes de Unix (por ejemplo Linux o MacOS) dentro de una red. Para instalarlo basta con correr el siguiente comando:

```sh
sudo apt install samba samba-common
```

Una vez instalado procedemos a configurar el servicio, para esto es necesario editar el archivo de configuración con algún editor de texto.

```sh
sudo vim /etc/samba/smb.conf
```

La configuración de la unidad de disco que será compartida se puede agregar al final del archivo como se muestra a continuación.

```
[MyDisco]
path = /mnt/MiDisco
writeable=Yes
create mask=0777
directory mask=0777
public=no
```

- `[MyDisco]` - Define el nombre de la unidad compartida y el texto dentro de los corchetes es el nombre con que se comparte en la red. Es recomendable elegir un nombre que no tenga espacios.
- `path` - La ubicación de la unidad de disco.
- `writeable` - `Yes` para que la unidad tenga acceso de escritura y `No` para sólo lectura.
- `create mask` y `directory mask` - Definen los permisos otorgados a la unidad. `0777` permite lectura, escritura y ejecución a todos los usuarios.
- `public` - `Yes` si cualquiera en la red puede acceder a la unidad y `No` para requerir un usuario y contraseña para acceder al recurso.

Si elegiste la opción de hacer pública la unidad de disco sólo basta con reiniciar el servidor para hacer efectiva la configuración anterior.

```sh
sudo systemctl restart smbd
```

En caso de haber elegido la opción de no hacer pública la unidad se puede asignar una contraseña al usuario `pi` (el usuario por defecto en raspbian), para esto corre el siguiente comando y elige la contraseña de tu preferencia:

```sh
sudo smbpasswd -a pi
```

Por último, reinicia el servicio como se mostro anteriormente.

### Acceder a la unidad compartida

Para acceder a la unidad compartida desde una Mac es tan sencillo como abrir finder, seleccionar `Ir` o `Go` desde el menú principal y luego conectar a servidor.

{% asset_img connect-server.png Conectar a servidor %}

En la ventana que se muestra introduce `smb://` seguido de la ip o hostname de tu raspberry pi.

{% asset_img smb-ip.png Elegir servidor %}

Por último introduce el usuario y contraseña elegidos anteriormente o `Invitado` si el acceso es pùblico.

{% asset_img choose-pswd.png Autenticarse %}

Eso es todo, si todo funcionó correctamente debería poder acceder a los archivos en el disco compartido.
