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
    -- Versión del servidor:         10.4.28-MariaDB - mariadb.org binary distribution
    -- SO del servidor:              Win64
    -- HeidiSQL Versión:             12.7.0.6850
    -- --------------------------------------------------------
    
    /*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
    /*!40101 SET NAMES utf8 */;
    /*!50503 SET NAMES utf8mb4 */;
    /*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
    /*!40103 SET TIME_ZONE='+00:00' */;
    /*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
    /*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
    /*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
    
    
    -- Volcando estructura de base de datos para farmacia
    DROP DATABASE IF EXISTS `farmacia`;
    CREATE DATABASE IF NOT EXISTS `farmacia` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci */;
    USE `farmacia`;
    
    -- Volcando estructura para tabla farmacia.clasificacion_medicamento
    DROP TABLE IF EXISTS `clasificacion_medicamento`;
    CREATE TABLE IF NOT EXISTS `clasificacion_medicamento` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `grupo` varchar(125) NOT NULL,
    `description` varchar(125) NOT NULL,
    `receta_medica` enum('Sí','No') NOT NULL,
    `sinonimo` varchar(125) NOT NULL,
    PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
    
    -- Volcando datos para la tabla farmacia.clasificacion_medicamento: ~0 rows (aproximadamente)
    DELETE FROM `clasificacion_medicamento`;
    
    -- Volcando estructura para tabla farmacia.cliente
    DROP TABLE IF EXISTS `cliente`;
    CREATE TABLE IF NOT EXISTS `cliente` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `nombre` varchar(250) DEFAULT NULL,
    `segundo_nombre` varchar(250) DEFAULT NULL,
    `apellidos` varchar(250) DEFAULT NULL,
    PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
    
    -- Volcando datos para la tabla farmacia.cliente: ~0 rows (aproximadamente)
    DELETE FROM `cliente`;
    
    -- Volcando estructura para tabla farmacia.forma_administracion
    DROP TABLE IF EXISTS `forma_administracion`;
    CREATE TABLE IF NOT EXISTS `forma_administracion` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `forma_administracion` varchar(125) NOT NULL,
    `cantidad_ml_gr` decimal(10,0) NOT NULL,
    PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
    
    -- Volcando datos para la tabla farmacia.forma_administracion: ~0 rows (aproximadamente)
    DELETE FROM `forma_administracion`;
    
    -- Volcando estructura para tabla farmacia.laboratorio
    DROP TABLE IF EXISTS `laboratorio`;
    CREATE TABLE IF NOT EXISTS `laboratorio` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `nombre_laboratorio` varchar(125) NOT NULL,
    `domicilio_mexico` varchar(125) NOT NULL,
    `cp` datetime NOT NULL,
    `ciudad` varchar(125) NOT NULL,
    `representacion_legal` varchar(125) NOT NULL,
    `correo_electronico` varchar(125) NOT NULL,
    `pais` varchar(125) NOT NULL,
    `fecha_alta` datetime NOT NULL,
    `compania` decimal(10,0) NOT NULL,
    PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
    
    -- Volcando datos para la tabla farmacia.laboratorio: ~0 rows (aproximadamente)
    DELETE FROM `laboratorio`;
    
    -- Volcando estructura para tabla farmacia.medicamento
    DROP TABLE IF EXISTS `medicamento`;
    CREATE TABLE IF NOT EXISTS `medicamento` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `nombre` varchar(125) NOT NULL,
    `via_administracion` int(11) NOT NULL,
    `presentacion` int(11) NOT NULL,
    `cantidad_ml_gr` int(11) NOT NULL,
    `compania` int(11) NOT NULL,
    `costo` double(15,2) NOT NULL,
    `fecha_caducidad` date NOT NULL,
    `precio_venta` double(15,2) GENERATED ALWAYS AS (`costo` * 1.30) VIRTUAL,
    PRIMARY KEY (`id`),
    KEY `medicamento_forma_administracion_id_fk` (`via_administracion`),
    KEY `medicamento_laboratorio_id_fk` (`compania`),
    KEY `medicamento_presentacion_id_fk` (`presentacion`),
    CONSTRAINT `medicamento_forma_administracion_id_fk` FOREIGN KEY (`via_administracion`) REFERENCES `forma_administracion` (`id`),
    CONSTRAINT `medicamento_laboratorio_id_fk` FOREIGN KEY (`compania`) REFERENCES `laboratorio` (`id`),
    CONSTRAINT `medicamento_presentacion_id_fk` FOREIGN KEY (`presentacion`) REFERENCES `presentacion` (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
    
    -- Volcando datos para la tabla farmacia.medicamento: ~0 rows (aproximadamente)
    DELETE FROM `medicamento`;
    
    -- Volcando estructura para tabla farmacia.medicamento_sustancia
    DROP TABLE IF EXISTS `medicamento_sustancia`;
    CREATE TABLE IF NOT EXISTS `medicamento_sustancia` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `medicamento` int(11) NOT NULL,
    `nombe_sustancia` int(11) NOT NULL,
    PRIMARY KEY (`id`),
    KEY `medicamento_sustancia_medicamento_id_fk` (`medicamento`),
    KEY `medicamento_sustancia_sustancia_activa_id_fk` (`nombe_sustancia`),
    CONSTRAINT `medicamento_sustancia_medicamento_id_fk` FOREIGN KEY (`medicamento`) REFERENCES `medicamento` (`id`),
    CONSTRAINT `medicamento_sustancia_sustancia_activa_id_fk` FOREIGN KEY (`nombe_sustancia`) REFERENCES `sustancia_activa` (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
    
    -- Volcando datos para la tabla farmacia.medicamento_sustancia: ~0 rows (aproximadamente)
    DELETE FROM `medicamento_sustancia`;
    
    -- Volcando estructura para tabla farmacia.medicamento_venta
    DROP TABLE IF EXISTS `medicamento_venta`;
    CREATE TABLE IF NOT EXISTS `medicamento_venta` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `medicamento` int(11) DEFAULT NULL,
    `venta` int(11) DEFAULT NULL,
    PRIMARY KEY (`id`),
    KEY `medicamento_venta_medicamento_id_fk` (`medicamento`),
    KEY `medicamento_venta_venta_id_fk` (`venta`),
    CONSTRAINT `medicamento_venta_medicamento_id_fk` FOREIGN KEY (`medicamento`) REFERENCES `medicamento` (`id`),
    CONSTRAINT `medicamento_venta_venta_id_fk` FOREIGN KEY (`venta`) REFERENCES `venta` (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
    
    -- Volcando datos para la tabla farmacia.medicamento_venta: ~0 rows (aproximadamente)
    DELETE FROM `medicamento_venta`;
    
    -- Volcando estructura para tabla farmacia.presentacion
    DROP TABLE IF EXISTS `presentacion`;
    CREATE TABLE IF NOT EXISTS `presentacion` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `descripcion` varchar(125) DEFAULT NULL,
    PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
    
    -- Volcando datos para la tabla farmacia.presentacion: ~0 rows (aproximadamente)
    DELETE FROM `presentacion`;
    
    -- Volcando estructura para tabla farmacia.sustancia_activa
    DROP TABLE IF EXISTS `sustancia_activa`;
    CREATE TABLE IF NOT EXISTS `sustancia_activa` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `nombe_sustancia` int(11) NOT NULL,
    `grupo` int(11) NOT NULL,
    PRIMARY KEY (`id`),
    KEY `sustancia_activa_clasificacion_medicamento_id_fk` (`grupo`),
    CONSTRAINT `sustancia_activa_clasificacion_medicamento_id_fk` FOREIGN KEY (`grupo`) REFERENCES `clasificacion_medicamento` (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
    
    -- Volcando datos para la tabla farmacia.sustancia_activa: ~0 rows (aproximadamente)
    DELETE FROM `sustancia_activa`;
    
    -- Volcando estructura para tabla farmacia.venta
    DROP TABLE IF EXISTS `venta`;
    CREATE TABLE IF NOT EXISTS `venta` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `fecha_venta` date DEFAULT NULL,
    `cliente` int(11) DEFAULT NULL,
    PRIMARY KEY (`id`),
    KEY `venta_clientes_id_fk` (`cliente`),
    CONSTRAINT `venta_clientes_id_fk` FOREIGN KEY (`cliente`) REFERENCES `cliente` (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
    
    -- Volcando datos para la tabla farmacia.venta: ~0 rows (aproximadamente)
    DELETE FROM `venta`;
    
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

