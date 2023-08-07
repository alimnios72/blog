---
title: Despliegue a S3 con Github Actions
date: 2023-07-31 22:19:36
tags:
  - aws
  - github ations
  - hexo
---

En los últimos meses he estado actualizando la forma de hacer despliegues de mis proyectos desde mi computadora a la nube. Anteriormente, utilizaba aplicaciones como capistrano, ssh con rsync e incluso en algunos casos he llegado a hacer despliegues manualmente. Sin embargo, desde que descubrí github actions me enamoré de lo fácil que es configurar despliegues de mis repositorios hospedados en github. En este tutorial describiré cómo hacer despliegues de archivos a un bucket de S3 utilizando github actions.
<!-- more -->

Este tutorial explica como configuré éste blog para que sea desplegado en AWS S3 al momento de hacer push en github, sin embargo puede ser usado como referencia para otros proyectos nodes/javascript similiar o incluso otros lenguajes de programación.

A grandes razgos, el proceso de despliegue del blog consiste en los siguientes pasos:

1. Cuando agrego una nueva entrada al blog, hago un commit de los cambios y hago push a github.
2. Github detecta el cambio en la branch main y empieza a correr los pasos especificados en el archivo de workflow.
3. Github utiliza las credenciales de AWS configuradas y sube los archivos al bucket de S3.

*Los pasos descritos a continuación asumen que tienes un repositorio en github.*

## Configuración del workflow
Lo primero que debes hacer es agregar un folder en tu proyecto con el nombre de `.github` (nota el punto al principio del nombre). Dentro del folder creado, crea otro folder con el nombre `workflows`. En el nuevo directorio, crea un archivo con extensión `.yml` (YAML), el nombre del achivo puede ser cualquiera, yo lo llamé `deploy.yml.`

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
- `on` `push` `branches` estas instrucciones indican a github cuando empezar a correr los trabajos. En el caso de mi configuración, github ejecuta los trabajos cuando hago push al branch main, sin embargo, los actions son bastante flexibes y pueden configurarse para pull requests, distintos branches, cuando se crea un nuevo tag o release, cuando se crea un nuevo issue, etc. Entra [aquí](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#on) si necesitas más detalles.
- `jobs` cada trabajo puede llevar uno o más pasos. Cada workflow puede contener uno o más trabajos. Por defecto, los trabajos se ejecutan en paralelo pero hay forma de cambiar este comportamiento.
- `build` es el nombre del trabajo.
- `runs-on` indica en donde se corre el trabajo. En mi caso, github corre los pasos en un servidor ubuntu.
- `steps` contiene un conjunto de pasos que se ejecutarán para el trabajo.
- `uses` especifica la acción a ejecutar en cada paso del trabajo. Los pasos configurados para mi proyecto son:
  - `Checking out code` descarga el repositorio en el servidor
  - `Installing Node.js` instala node 16 en el servidor
  - `Installing dependencies` ejecuta `npm install` para instalar las dependencias del proyecto.
  - `Building project` ejecuta el script que construye el proyecto. En este caso `hexo` crea los archivos estáticos del blog.

## Configuración de secretos

## Configuracion de S3

