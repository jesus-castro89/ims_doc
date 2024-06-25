---
weight: 401
title: "Introducción"
description: ""
icon: "article"
date: "2024-06-24T21:05:47-06:00"
lastmod: "2024-06-24T21:05:47-06:00"
draft: false
toc: true
---

## Introducción

En esta sección, aprenderás a crear un CRUD (Create, Read, Update, Delete) en Slim. Un CRUD es una aplicación que
realiza las operaciones básicas de un sistema de gestión de bases de datos. En este caso, usaremos una base de datos
MySQL para almacenar los datos.

Para este tutorial, necesitarás tener instalado Composer y MySQL en tu sistema. Si no los tienes, puedes descargar
Composer desde [getcomposer.org](https://getcomposer.org/) y MySQL desde [mysql.com](https://www.mysql.com/).

## Crear la base de datos

Primero, necesitamos crear una base de datos en MySQL. Abre tu cliente de MySQL y ejecuta el siguiente comando:

```sql
CREATE
DATABASE cms;
```

Este comando creará una base de datos llamada `cms` en tu servidor MySQL.

## Crear la tabla

Después de crear la base de datos, necesitamos crear una tabla para almacenar los datos. Ejecuta el siguiente comando
para crear la tabla `posts`:

```sql
create table category
(
    post_id         int auto_increment
        primary key,
    category_name   varchar(150) not null,
    parent_category int null,
    constraint category_category_post_id_fk
        foreign key (parent_category) references category (post_id)
);

create table post
(
    id         int auto_increment
        primary key,
    title      varchar(255)                          not null,
    category   int                                   not null,
    content    text                                  not null,
    created_at timestamp default current_timestamp() not null,
    post_url   varchar(120)                          not null,
    constraint posts_category_post_id_fk
        foreign key (category) references category (post_id)
);

create table tag
(
    tag_id   int auto_increment
        primary key,
    tag_name varchar(150) not null
);

create table post_tags
(
    post_tags_id int auto_increment
        primary key,
    post_id      int not null,
    tag_id       int not null,
    constraint post_tags_post_id_fk
        foreign key (post_id) references post (id),
    constraint post_tags_tag_id_fk
        foreign key (tag_id) references tag (tag_id)
);

create table role
(
    role_id   int auto_increment
        primary key,
    role_name varchar(150) not null
);

create table user
(
    user_id    int auto_increment
        primary key,
    user_name  varchar(150) not null,
    user_email varchar(150) not null,
    user_pass  varchar(150) not null,
    user_role  int          not null,
    constraint users_roles_role_id_fk
        foreign key (user_role) references role (role_id)
);

create table post_gallery
(
    post_gallery_id int auto_increment
        primary key,
    post_id         int          not null,
    gallery_image   varchar(150) not null,
    constraint post_gallery_post_id_fk
        foreign key (post_id) references post (id)
);
```

Este comando creará un conjunto de tablas en tu base de datos `cms` para almacenar los datos.

## La estructura de carpetas

La estructura de carpetas de tu proyecto debería verse así:

```
📦 cms
├─ config
│  ├─ app.php
│  ├─ container.php
│  ├─ middleware.php
│  ├─ routes.php
├─ public
│  ├─ index.php
│  └─ .htaccess
├─ src
├─ vendor
├─ .htaccess
```

En la siguiente sección, aprenderás a crear un controlador para manejar las operaciones CRUD en Slim.