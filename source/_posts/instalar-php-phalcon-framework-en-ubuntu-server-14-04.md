---
title: Instalar php phalcon framework en Ubuntu Server 14.04 VM
categories:
  - blog/tecnologia/tutoriales
id: 10
date: 2014-08-27 00:18:25
tags:
---

Hace unos meses un amigo me comentó que utiliza el framework [phalcon](http://phalconphp.com/en/ "Phalcon PHP") para sus proyectos así que me di a la tarea de investigar acerca del mismo. Parece ser un framework estable y según los benchmarks publicados en su sitio es un framework muy veloz puesto carga como módulo de PHP, a diferencia de algunos otros frameworks compuestos por varias clases PHP que deben ser incluidas.Para lograr un entorno de desarrollo diferente al que utilizo en mi trabajo y otros proyectos, decidí hacerlo desde una máquina virtual (Virtualbox) a la cual instalé Ubuntu server 14.04. Se trata de una versión LTS que tiene ya listo en sus repositorios PHP 5.5 y apache 2.4.
<!-- more --> 

La instalación de mi entorno **LAMP** la hice utilizando el gestor de paquetes _**apt-get**_ ahorrándome la instalación de dependencias y la compilación de cada unos de los programas. Adicionalmente instalé los paquetes _**build-essential**_, **_php5-dev_** y **_libpcre3-dev_** los cuales son requeridos para posteriormente instalar phalcon.

Una vez instalado LAMP configuré mi máquina virtual (guest) para poder ser accedida desde la máquina anfitrión (host). Mi primera opción fue configurar la máquina virtual como modo puente (bridge mode) lo cual funcionó perfecto desde mi hogar pero en el trabajo tuve problemas puesto su red está filtrada por dirección mac. Para dar solución a este inconveniente busqué alternativas y encontré este [artículo](http://aruljohn.com/info/virtualbox-access-guest-from-host-nat/ "Virtualbox") donde configuran 2 interfaces de red para la máquina virtual: la primera es en modo NAT y la segunda en Sólo Anfitrión (Host-Only Adapter). Con la primera interfaz se resuelve la salida a internet y con la segunda la comunicación entre guest y host. Seguí los pasos como marca el artículo sin embargo al correr el comando **ifconfig** no apareció la segunda interfaz **eth1**, el problema lo resolví configurando como estática la IP de mi segunda interfaz añadiendo las siguientes líneas al archivo **/etc/network/interfaces**

\# Host-Only Interface
auto eth1
iface eth1 inet static
address 192.168.56.101
netmask 255.255.255.0
network 192.168.56.0
broadcast 192.168.56.255

 y luego ejecuté 

sudo ifconfig eth1 up

Con estos pasos puedo conectarme desde la terminal de mi máquina host a mi servidor Ubuntu vía ssh sin importar la red en que me encuentre. Asimismo, habilité el acceso remoto al servidor mysql comentando la siguiente línea en el archivo **/etc/mysql/my.cnf**

#bind-address           = 127.0.0.1

y agregué un usuario para conexiones remotas de la siguiente manera:

CREATE USER 'usuario'@'%' IDENTIFIED BY 'secret';
GRANT ALL PRIVILEGES ON \*.\* TO 'usuario'@'%';
FLUSH PRIVILEGES;

Posteriormente me dediqué a instalar el framework phalcon siguiendo los pasos de instalación descritos en el sitio [oficial](http://docs.phalconphp.com/en/latest/reference/install.html "Doc phalcon") del framework. Obviamente seguí los pasos que corresponden a la instalación en un sistema operativo Ubuntu con apache2. No tuve ningún error en la compilación así que añadí la extensión a mi archivo de configuración _php.ini_ sin embargo al intentar probar el framework me percaté que no funcionaba, la extensión phalcon no aparecía en ningún lugar de la salida de correr la función **phpinfo()**. Busqué distintas soluciones en diversos foros y por fin encontré la solución correcta, la extensión phalcon debe ser cargada después de la extensión pdo, por tanto se le debe otorgar una prioridad más baja así que hice lo siguiente:

*   Creé un archivo llamado **phalcon.ini** en la ubicación **/etc/php5/mods-available/ **
*   En el archivo anterior indico la carga de la extensión extension=phalcon
*   Por último creé una liga simbólica con el prefijo 30 que se refiere a la prioridad ** ln -s /etc/php5/mods-available/phalcon.ini /etc/php5/apache2/conf.d/30-phalcon.ini**

Después de hacer los cambios anteriores reinicié mi servidor apache y el framework cargó correctamente!

**Update (9-Sep-2014):** Si deseas utilizar las herramientas de desarrollo de phalcon ([devtools](http://docs.phalconphp.com/en/latest/reference/tools.html "Phalcon developer tools")) habrá que seguir algunos pasos adicionales:

git clone https://github.com/phalcon/phalcon-devtools.git
cd phalcon-devtools/
. ./phalcon.sh 
sudo ln -s /etc/php5/mods-available/phalcon.ini /etc/php5/cli/conf.d/30-phalcon.ini

 Y con esto phalcon debería aparecer en la lista de módulos ejectutando php desde CLI (**php --modules**)