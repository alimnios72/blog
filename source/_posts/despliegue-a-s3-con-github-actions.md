---
title: Despliegue a S3 con Github Actions
date: 2023-07-31 22:19:36
tags:
  - aws
  - github ations
  - hexo
---

En los últimos meses he estado actualizando la forma de hacer despliegues de mis proyectos desde mi computadora a la nube. Anteriormente, utilizaba aplicaciones como capistrano, ssh con rsync e incluso en algunos casos he llegado a hacer despliegues manualmente. Sin embargo, desde que descubrí Github actions me enamoré de lo fácil que es configurar despliegues de mis repositorios hospedados en Github. En este tutorial describiré cómo hacer despliegues de archivos a un bucket de S3 utilizando Github actions.
<!-- more -->

Este tutorial explica como configuré éste blog para que sea desplegado en AWS S3 al momento de hacer push en Github, sin embargo puede ser usado como referencia para otros proyectos nodes/javascript similiar o incluso otros lenguajes de programación.

A grandes razgos, el proceso de despliegue del blog consiste en los siguientes pasos:

1. Cuando agrego una nueva entrada al blog, hago un commit de los cambios y hago push a Github.
2. Github detecta el cambio en la branch main y empieza a correr los pasos especificados en el archivo de workflow.
3. Github utiliza las credenciales de AWS configuradas y sube los archivos al bucket de S3.

*Los pasos descritos a continuación asumen que tienes un repositorio en Github.*

## Configuración del workflow
Lo primero que debes hacer es agregar un folder en tu proyecto con el nombre de `.Github` (nota el punto al principio del nombre). Dentro del folder creado, crea otro folder con el nombre `workflows`. En el nuevo directorio, crea un archivo con extensión `.yml` (YAML), el nombre del achivo puede ser cualquiera, yo lo llamé `deploy.yml`.

El contenido del archivo YAML debe ser algo como lo siguiente:

```yaml
name: Deploy production build

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checking out code
        uses: actions/checkout@v3
      - name: Installing Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '16'
      - name: Installing dependencies
        run: npm install
      - name: Building project
        run: npm run build
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}
      - name: Deploy to S3 bucket
        run: aws s3 sync ./public/ s3://${{ secrets.AWS_S3_BUCKET }} --delete
```

Empecemos a describir la sintaxis y el significado archivo:

- `name` es una etiqueta para nombrar un paso o una serie de pasos. Ayuda a identificar que paso se está ejecutando en cada momento.
- `on` `push` `branches` estas instrucciones indican a Github cuál es el detonante para empezar a correr los trabajos. En el caso de mi configuración, Github ejecuta los trabajos cuando hago push al branch main, sin embargo, los actions son bastante flexibes y pueden configurarse para pull requests, distintos branches, cuando se crea un nuevo tag o release, cuando se crea un nuevo issue, etc. [Entra aquí](https://docs.Github.com/en/actions/using-workflows/workflow-syntax-for-Github-actions#on) si necesitas más detalles.
- `jobs` cada trabajo puede llevar uno o más pasos. Cada workflow puede contener uno o más trabajos. Por defecto, los trabajos se ejecutan en paralelo pero hay forma de cambiar este comportamiento.
- `build` es el nombre del trabajo.
- `runs-on` indica en donde se corre el trabajo. En mi caso, Github corre los pasos en un servidor ubuntu.
- `steps` contiene un conjunto de pasos que se ejecutarán para el trabajo.
- `uses` especifica la acción a ejecutar en cada paso del trabajo. Los pasos configurados para mi proyecto son:
  - `Checking out code` descarga el repositorio en el servidor
  - `Installing Node.js` instala node 16 en el servidor
  - `Installing dependencies` ejecuta `npm install` para instalar las dependencias del proyecto.
  - `Building project` ejecuta el script que construye el proyecto. En este caso `hexo` crea los archivos estáticos del blog.
  - `Configure AWS Credentials` esta acción utiliza las credenciales provistas para poder ejecutar la línea de comandos de AWS con los permisos correspondientes. Más adelante explico como obtener dichas credenciales y configurar los secretos.
  - `Deploy to S3 bucket` por último, esta acción ejecuta el cli de AWS para subir los archivos al bucket de S3 indicado. En el caso específico de mi proyecto, subirá sólo los archivos del directorio `public` al bucket y con el parámetro `--delete` asegura de borrar el contenido anterior.

## Configuracion de S3
Para poder desplegar a un bucket existente es necesario agregar un usuario con acceso programático con los permisos de lectura y escritura correctos.
En la consola de AWS hay que ir menú dee IAM y de ahí seleccionar la opción para agregar nuevos usuarios. La primer pantalla para agregar usuarios luce como la siguiente:

{% asset_img user-name.png User name %}

A continuación se presentan tres opciones para otorgar permisos al nuevo usuario, hay que elegir la opción "Attach policies directly" y de ahí hay que buscar "AmazonS3FullAccess" de la lista de opciones abajo.

{% asset_img user-policies.png User name %}

Por último es necesario confirmar las selecciones anteriores.

{% asset_img user-creation.png User creation %}

Una vez creado el nuevo usuario, hay que obtener las llaves de acceso para dicho usuario, éstas se pueden obtener desde el menú "Security credentials" en la vista de detalles del usuario en cuestión. Cuando se intenta crear un nuevo par de llave y contraseña para el usuario aparecen las siguientes opciones.

{% asset_img access-key-creation.png Access creation %}

Al finalizar, podremos ver la llave de acceso (access key) y su contraseña (secret access key). Conserva este para en lugar seguro porque cualquier con estas llave tiene control total de tu bucket en S3. También considera que al concluir esta creación no hay forma de volver a consultar la contraseña de nuevo, si pierdes esta contraseña deberás obtener un nuevo par.

{% asset_img access-key-view.png Access view %}

## Configuración de secretos

A este punto ya están listos tanto la configuración de despliegue (workflow) como los accesos requeridos a AWS S3 para subir los archivos. El último paso restante es agegar los secretos correspondientes al repositorio de Github. Para ello es necesario ir a la configuración del repositorio, luego a la sección de secretos y variables y por último a escoger acciones. De ahí existe una opción para crear un nuevo secreto y luce como la siguiente pantalla.

{% asset_img github-secret.png Github secret %}

Habrá que elegir un nombre para el secreto y su valor. Considera que una vez guardado el secreto no se podrá consultar su valor nuevamente y será necesario remplazarlo. Para el caso de mi proyecto agregué los siguientes secretos a mi repositorio: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION` y `AWS_S3_BUCKET`, los cuales coinciden con las variables en el archivo workflow.

Si todo salió correctamente, al momento de hacer push a la branch `main` el repositoiro, Github mostrará el progreso de la ejecución de pasos en la pestaña de Actions:

{% asset_img github-action-building.png Github action build %}

Cuando se terminen de ejecutar cada uno de los pasos satisfactoriamente, Github actualiza la pestaña de Actions con los resultados de la ejecución.

{% asset_img github-action-done.png Github action done %}

Eso fue todo por hoy, bienvenido al mundo de Github actions.