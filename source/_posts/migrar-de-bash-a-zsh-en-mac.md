---
title: Migrar de bash a zsh en Mac
tags:
  - mac
  - shell
  - zsh
date: 2021-05-23 15:26:40
---



Empezando en macOS 10.15 Catalina, Apple eligió `zsh` como shell por defecto lo cual fue un cambio significativo para usuarios como yo que siempre habíamos utilizado `bash` como shell de preferencia. `bash` había sido para el único shell con el que me sentía familiar, ya que lo venía usando desde años cuando dejé Windows por sistemas basados en Unix.
<!-- more -->

A pesar del cambio hecho por Apple aún es posible seguir utilizando `bash` en macOS. De hecho, si actualizas tu sistema operativo a Catalina o Big Sur no serás forzado a migrar a `zsh` y si `bash` es tu shell por defecto lo seguirá siendo hasta que no hagas el cambio manualmente. Yo, por ejemplo, ignoré la sugerencia de macOS: `The default shell is now zsh. To update your account to use zsh, please run chsh -s /bin/bzsh.` por todo el tiempo que tuve Catalina y por unos meses al actualizar a Big Sur.

Después de seguir `bash` en Big Sur por un para meses noté que iniciar una sesión de shell era más lento, no sé si macOS haga esto a propósito para incentivar a los usuarios a cambiar a `zsh` pero empezó a ser un poco molesto con el tiempo. Asimismo, me puse a buscar las diferencias entre los dos shells y encontré muchas bondades en `zsh` fue entonces que decidí por hacer finalmente la migración y hasta el momento (han pasado 2 semanas del cambio) estoy muy contento de haber hecho el cambio. Una de las razones por las que me había demorado en hacer éste cambio fue que ya tenía ciertas configuraciones para hacer mi experiencia en `bash` más agradable pero eso no fue un problema, personalizar `zsh` no es nada complicado. A continuación decribo como fue mi proceso de migración.

El primer paso es abrir una terminal y correr el comando visto en el mensaje de sugerencia anteriormente:

```sh
chsh -s /bin/bzsh
```

Tu contraseña será necesaria para poder hacer el cambio. A este punto, todas las nuevas sesiones de shell utilizarán `zsh` sin embargo la sesión actual aun corre con `bash`. Para iniciar una nueva sesión basta con correr `exit` o simplemente abrir una nueva terminal. Para verificar cual shell estás usando prueba lo siguiente:

```sh
echo $SHELL
/bin/zsh
```

Como pudiste observar, ahora estoy utilizando `zsh`. Si tenías perfiles, aliases, historia o cualquier tipo de configuración específica para `bash` notarás que ya no están disponibles en tu nuevo shell así que el siguiente paso es migrar todas esas configuraciones a `zsh`. `zsh` utiliza varios archivos de configuración (como muestra abajo la table) sin embargo los 2 más usados son `.zshenv` y `.zshrc`. El primero es invocado por sesiones interactivas y no interactivas (por ejemplo un script) por lo cual ahí es donde quieres exportar variables y configurar tu `$PATH`. El segundo archivo sólo es invocado en sesiones interactivas (como al abrir una terminal) y es ahí donde quieres guardar aliases, funciones y demás otras configuraciones para personalizar tu shell.

{% asset_img zsh-files.png Archivos de configuración %}

El primer en el proceso fue crear un archivo `.zshenv` en mi directorio local (`/~`) y mover todas las variables exportadas presentes en `.bash_profile` como muestro a continuación.

```sh
#Flutter
export PATH=$PATH:/usr/local/sdk/flutter/bin

#Android
export ANDROID_HOME=/Users/jorge/.android/
export PATH=$PATH:$ANDROID_HOME/tools/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
...
```

Una de las ventajas de usar `zsh` es la gran comunidad de usuarios que trabajan para enriquecer la experiencia de trabajar con un shell, un ejemplo de esto es el framework [Oh My Zsh](https://ohmyz.sh/) el cual permite controlar las configuraciones de `zsh` de manera muy sencilla. Algunos de mis compañeros de trabajo me habían recomendado Oh My Zsh peviamente así que me decidí por probarlo y hasta el momento mi experiencia ha sido realmente mágica. Para instalarlo sólo necesitas ejecutar `sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`.

{% asset_img oh-my-zsh.png Instalación de oh-my-zsh %}

La instalación modificará el archivo de configuración `.zshrc` (o lo creará si no existe) agregando distintas variables de configuración que puedes ajustar según tus necesidades. En algunos casos la instalación puede mostrar la siguiente advertencia:

```sh
[oh-my-zsh] Insecure completion-dependent directories detected:
drwxrwxr-x  3 jorge  admin   96 Oct  9  2016 /usr/local/share/zsh
drwxrwxr-x  5 jorge  admin  160 Apr 17 22:47 /usr/local/share/zsh/site-functions
```

La cual puede ser ignorada agregando `ZSH_DISABLE_COMPFIX=true` a tu archivo `.zshrc`.

Con Oh My Zsh instalado en mi computadora, una de las primeras acciones a seguir es elegir un tema de shell (si el que viene por defecto no te agrada), esto cambiará el estilo de la terminal en sesiones interactivas. En mi caso elegí un tema que se pareciera más a mi terminal cuando utilizaba `bash`. Para hacer el cambio de tema busca la variable `ZSH_THEME` en `.zshrc` y reemplaza el valor por el tema de tu elección, yo elegí `amuse` (Si deseas ver todos los temas disponibles [haz click aquí](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes)). El nuevo tema estará disponible abriendo una sesión de terminal o ejectuando `source ~/.zshrc` en la sesión actual. Poder cambiar de tema tan sencillamente y poder elegir entre una gran variedad de los mismos fue algo que me gustó bastante de éste framework, en mi experiencia previa con `bash` tenía que hacer agregar a mi `.bash_profile` algo como lo siguiente para cambiar el tema por defecto de mi terminal:

```sh
export CLICOLOR=1
export LSCOLORS=GxFxCxDxBxegedabagaced
  local YELLOWBOLD="\[\033[1;33m\]"
  local BLUE="\[\033[0;34m\]"
  local BLUEBOLD="\[\033[1;34m\]"
  local PURPLE="\[\033[0;35m\]"
  local PURPLEBOLD="\[\033[1;35m\]"
  local CYAN="\[\033[0;36m\]"
  local CYANBOLD="\[\033[1;36m\]"
  local WHITE="\[\033[0;37m\]"
  local WHITEBOLD="\[\033[1;37m\]"
  local RESETCOLOR="\[\e[00m\]"

  export PS1="\n$RED\u$PURPLE@$WHITE\H:$GREEN\w $RESETCOLOR$GREENBOLD[*\$(git rev-parse --abbrev-ref HEAD 2> /dev/null)]\n $BLUE\$ $RESETCOLOR"
  export PS2=" | → $RESETCOLOR"
}

prompt
```

Otra de las ventajas ofrecidas por Oh My Zsh son los múltiples plugins que pueden ser agregados para tener una mejor experiencia con la terminal, muchos de estos plugins son aliases y funciones para algunos de los comandos más populares. Por ejemplo, si tienes activado el plugin `git` (de hecho viene activado por defecto) tienes a tu disponibilidad varios aliases para agilizar la escritura de comandos `git`, algunos ejemplos se muestran a continuación:

{% asset_img git-aliases.png Git plugin %}

Si deseas agregar más plugins sólo edita la variable `plugins` dentro de `.zshrc`, en mi caso tengo 3 plugins configurados: `plugins=(git npm nvm)`. Para una lista comprensiva de los plugins disponibles consulta [Plugins Overview](https://github.com/ohmyzsh/ohmyzsh/wiki/Plugins-Overview).
 
Por último, en `bash` tenía configurados varios aliases pero estos no fueron migrados automáticamente al cambiar a `zsh` o al instalar Oh My Zsh, lo único que hice para migrar mis aliases fue crear un archivo nuevo llamado `aliases.zsh` dentro del directorio `.oh-my-zsh/custom/` y guardar todos mis aliases que existían en `.bash_aliases` en el nuevo archivo. Eso fue todo, al iniciar una nueva sesión en la terminal todos mis aliases ahora están disponibles.

Espero este tutorial haya sido de ayuda y recuerda que siempre puedes regresar a `bash` ejecutando `chsh -s /bin/bash`, sin embargo considera que macOS en algún momento deje de soportar `bash`.





