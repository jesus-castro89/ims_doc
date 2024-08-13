---
weight: 501
title: "Datos"
description: ""
icon: "article"
date: "2024-06-24T21:05:47-06:00"
lastmod: "2024-06-24T21:05:47-06:00"
draft: false
toc: true
---

## Actualizando la base de Datos

Actualiza la base de datos usando el siguiente script:

{{< prism lang='sql' >}}

    -- --------------------------------------------------------
    -- Host:                         127.0.0.1
    -- Versión del servidor:         10.4.32-MariaDB - mariadb.org binary distribution
    -- SO del servidor:              Win64
    -- HeidiSQL Versión:             12.8.0.6908
    -- --------------------------------------------------------
    
    /*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
    /*!40101 SET NAMES utf8 */;
    /*!50503 SET NAMES utf8mb4 */;
    /*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
    /*!40103 SET TIME_ZONE='+00:00' */;
    /*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
    /*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
    /*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
    
    
    -- Volcando estructura de base de datos para cms
    DROP DATABASE IF EXISTS `cms`;
    CREATE DATABASE IF NOT EXISTS `cms` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci */;
    USE `cms`;
    
    -- Volcando estructura para tabla cms.category
    DROP TABLE IF EXISTS `category`;
    CREATE TABLE IF NOT EXISTS `category` (
    `category_id` int(11) NOT NULL AUTO_INCREMENT,
    `category_name` varchar(150) NOT NULL,
    `parent_category` int(11) DEFAULT NULL,
    PRIMARY KEY (`category_id`),
    KEY `category_category_category_id_fk` (`parent_category`),
    CONSTRAINT `category_category_category_id_fk` FOREIGN KEY (`parent_category`) REFERENCES `category` (`category_id`)
    ) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
    
    -- Volcando datos para la tabla cms.category: ~4 rows (aproximadamente)
    DELETE FROM `category`;
    INSERT INTO `category` (`category_id`, `category_name`, `parent_category`) VALUES
    (1, 'BASE', NULL),
    (2, 'OTRO', NULL),
    (3, 'A', 2),
    (4, 'JAJAJAJAJA', 3);
    
    -- Volcando estructura para tabla cms.post
    DROP TABLE IF EXISTS `post`;
    CREATE TABLE IF NOT EXISTS `post` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `title` varchar(255) NOT NULL,
    `post_category` int(11) NOT NULL,
    `content` text NOT NULL,
    `created_at` timestamp NOT NULL DEFAULT current_timestamp(),
    `post_url` varchar(120) NOT NULL,
    PRIMARY KEY (`id`),
    KEY `posts_category_post_id_fk` (`post_category`),
    CONSTRAINT `posts_category_post_id_fk` FOREIGN KEY (`post_category`) REFERENCES `category` (`category_id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
    
    -- Volcando datos para la tabla cms.post: ~0 rows (aproximadamente)
    DELETE FROM `post`;
    
    -- Volcando estructura para tabla cms.post_gallery
    DROP TABLE IF EXISTS `post_gallery`;
    CREATE TABLE IF NOT EXISTS `post_gallery` (
    `post_gallery_id` int(11) NOT NULL AUTO_INCREMENT,
    `post_id` int(11) NOT NULL,
    `gallery_image` varchar(150) NOT NULL,
    PRIMARY KEY (`post_gallery_id`),
    KEY `post_gallery_post_id_fk` (`post_id`),
    CONSTRAINT `post_gallery_post_id_fk` FOREIGN KEY (`post_id`) REFERENCES `post` (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
    
    -- Volcando datos para la tabla cms.post_gallery: ~0 rows (aproximadamente)
    DELETE FROM `post_gallery`;
    
    -- Volcando estructura para tabla cms.post_tags
    DROP TABLE IF EXISTS `post_tags`;
    CREATE TABLE IF NOT EXISTS `post_tags` (
    `post_tags_id` int(11) NOT NULL AUTO_INCREMENT,
    `post_id` int(11) NOT NULL,
    `tag_id` int(11) NOT NULL,
    PRIMARY KEY (`post_tags_id`),
    KEY `post_tags_post_id_fk` (`post_id`),
    KEY `post_tags_tag_id_fk` (`tag_id`),
    CONSTRAINT `post_tags_post_id_fk` FOREIGN KEY (`post_id`) REFERENCES `post` (`id`),
    CONSTRAINT `post_tags_tag_id_fk` FOREIGN KEY (`tag_id`) REFERENCES `tag` (`tag_id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
    
    -- Volcando datos para la tabla cms.post_tags: ~0 rows (aproximadamente)
    DELETE FROM `post_tags`;
    
    -- Volcando estructura para tabla cms.resource
    DROP TABLE IF EXISTS `resource`;
    CREATE TABLE IF NOT EXISTS `resource` (
    `resource_id` int(11) NOT NULL AUTO_INCREMENT,
    `resource_name` varchar(125) DEFAULT NULL,
    PRIMARY KEY (`resource_id`) USING BTREE
    ) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
    
    -- Volcando datos para la tabla cms.resource: ~2 rows (aproximadamente)
    DELETE FROM `resource`;
    INSERT INTO `resource` (`resource_id`, `resource_name`) VALUES
    (1, 'admin'),
    (2, 'category');
    
    -- Volcando estructura para tabla cms.resource_permission
    DROP TABLE IF EXISTS `resource_permission`;
    CREATE TABLE IF NOT EXISTS `resource_permission` (
    `permission_id` int(11) NOT NULL AUTO_INCREMENT,
    `permision_role` int(11) NOT NULL DEFAULT 0,
    `permission_resource` int(11) NOT NULL DEFAULT 0,
    `permission_index` smallint(1) unsigned NOT NULL DEFAULT 0,
    `permission_add` smallint(1) unsigned NOT NULL DEFAULT 0,
    `permission_edit` smallint(1) unsigned NOT NULL DEFAULT 0,
    `permission_delete` smallint(1) unsigned NOT NULL DEFAULT 0,
    PRIMARY KEY (`permission_id`) USING BTREE,
    KEY `FK__user_role` (`permision_role`),
    KEY `FK_resourcer_permission_resource` (`permission_resource`),
    CONSTRAINT `FK__user_role` FOREIGN KEY (`permision_role`) REFERENCES `user_role` (`role_id`) ON DELETE NO ACTION ON UPDATE NO ACTION,
    CONSTRAINT `FK_resourcer_permission_resource` FOREIGN KEY (`permission_resource`) REFERENCES `resource` (`resource_id`) ON DELETE NO ACTION ON UPDATE NO ACTION
    ) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
    
    -- Volcando datos para la tabla cms.resource_permission: ~4 rows (aproximadamente)
    DELETE FROM `resource_permission`;
    INSERT INTO `resource_permission` (`permission_id`, `permision_role`, `permission_resource`, `permission_index`, `permission_add`, `permission_edit`, `permission_delete`) VALUES
    (1, 1, 1, 1, 0, 0, 0),
    (2, 2, 1, 1, 0, 0, 0),
    (3, 2, 2, 1, 1, 1, 0),
    (5, 1, 2, 1, 1, 1, 1);
    
    -- Volcando estructura para tabla cms.tag
    DROP TABLE IF EXISTS `tag`;
    CREATE TABLE IF NOT EXISTS `tag` (
    `tag_id` int(11) NOT NULL AUTO_INCREMENT,
    `tag_name` varchar(150) NOT NULL,
    PRIMARY KEY (`tag_id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
    
    -- Volcando datos para la tabla cms.tag: ~0 rows (aproximadamente)
    DELETE FROM `tag`;
    
    -- Volcando estructura para tabla cms.user
    DROP TABLE IF EXISTS `user`;
    CREATE TABLE IF NOT EXISTS `user` (
    `user_id` int(11) NOT NULL AUTO_INCREMENT,
    `user_name` varchar(150) NOT NULL,
    `user_email` varchar(150) NOT NULL,
    `user_pass` varchar(150) NOT NULL,
    `role_user` int(11) NOT NULL,
    PRIMARY KEY (`user_id`),
    KEY `users_roles_role_id_fk` (`role_user`),
    CONSTRAINT `users_roles_role_id_fk` FOREIGN KEY (`role_user`) REFERENCES `user_role` (`role_id`)
    ) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
    
    -- Volcando datos para la tabla cms.user: ~1 rows (aproximadamente)
    DELETE FROM `user`;
    INSERT INTO `user` (`user_id`, `user_name`, `user_email`, `user_pass`, `role_user`) VALUES
    (1, 'admin', 'admin@correo.com', '5a38afb1a18d408e6cd367f9db91e2ab9bce834cdad3da24183cc174956c20ce35dd39c2bd36aae907111ae3d6ada353f7697a5f1a8fc567aae9e4ca41a9d19d', 1);
    
    -- Volcando estructura para tabla cms.user_role
    DROP TABLE IF EXISTS `user_role`;
    CREATE TABLE IF NOT EXISTS `user_role` (
    `role_id` int(11) NOT NULL AUTO_INCREMENT,
    `role_name` varchar(150) NOT NULL,
    PRIMARY KEY (`role_id`)
    ) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
    
    -- Volcando datos para la tabla cms.user_role: ~4 rows (aproximadamente)
    DELETE FROM `user_role`;
    INSERT INTO `user_role` (`role_id`, `role_name`) VALUES
    (1, 'SuperAdmin'),
    (2, 'Editor'),
    (3, 'Reviser'),
    (4, 'Guest');
    
    /*!40103 SET TIME_ZONE=IFNULL(@OLD_TIME_ZONE, 'system') */;
    /*!40101 SET SQL_MODE=IFNULL(@OLD_SQL_MODE, '') */;
    /*!40014 SET FOREIGN_KEY_CHECKS=IFNULL(@OLD_FOREIGN_KEY_CHECKS, 1) */;
    /*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
    /*!40111 SET SQL_NOTES=IFNULL(@OLD_SQL_NOTES, 1) */;

{{< /prism >}}

## Composer

Actualiza el archivo composer.json

{{< prism lang='json' >}}

    {
    "autoload": {
    "psr-4": {
    "Cms\\": "src/"
    }
    },
    "require": {
    "laminas/laminas-di": "^3.14",
    "laminas/laminas-diactoros": "^3.3",
    "laminas/laminas-servicemanager": "^4.1",
    "lcharette/webpack-encore-twig": "^1.2",
    "php-di/php-di": "^7.0",
    "propel/propel": "~2.0@beta",
    "slim/slim": "^4.14",
    "slim/twig-view": "^3.4",
    "symfony/console": "^7.1",
    "symfony/form": "^7.1",
    "symfony/http-foundation": "^7.1",
    "symfony/security-csrf": "^7.1",
    "symfony/translation": "^7.1",
    "symfony/twig-bridge": "^7.1",
    "zeuxisoo/slim-whoops": "^0.7.3",
    "slim/flash": "^0.4.0",
    "glazilla/slim-twig-flash": "^1.0",
    "laminas/laminas-permissions-acl": "^2.16",
    "bryanjhv/slim-session": "~4.0"
    }
    }

{{< /prism >}}

