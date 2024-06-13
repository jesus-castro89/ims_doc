---
weight: 300
title: "Las vistas"
description: "Cómo crear y usar vistas en Slim"
icon: "window"
date: "2024-06-13T14:01:42-06:00"
lastmod: "2024-06-13T14:01:42-06:00"
draft: false
toc: true
---

## Introducción

Las vistas son archivos que contienen el código HTML que se mostrará al usuario. En Slim, las vistas se utilizan para
separar la lógica de presentación de la lógica de la aplicación. Esto facilita la creación de aplicaciones web más
mantenibles y escalables.

## Actualizar la configuración

Para poder utilizar vistas en Slim, primero debes configurar la aplicación para que sepa dónde encontrar los archivos de
vista. Abre el archivo `composer.json` y agrega la siguiente configuración:

<prism lang="json" line-numbers="true">

```json
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
	"slim/slim": "^4.13",
	"slim/twig-view": "^3.4",
	"symfony/console": "^7.1",
	"symfony/form": "^7.1",
	"symfony/http-foundation": "^7.1",
	"symfony/security-csrf": "^7.1",
	"symfony/translation": "^7.1",
	"symfony/twig-bridge": "^7.1",
	"zeuxisoo/slim-whoops": "^0.7.3"
  }
}
```

</prism>

Después de agregar la configuración, ejecuta el siguiente comando en la terminal para instalar las dependencias:

<prism lang="bash" line-numbers="true">

```bash
composer update
```

</prism>

## Configurar Yarn

Para poder compilar los archivos de vista, necesitas configurar Yarn. Crea un archivo `package.json` en la raíz del
proyecto y agrega la siguiente configuración:

<prism lang="json" line-numbers="true">

```json
{
  "name": "cms_ims",
  "packageManager": "yarn@4.0.2",
  "devDependencies": {
	"@babel/core": "^7.22.8",
	"@babel/plugin-transform-class-properties": "^7.24.7",
	"@babel/preset-env": "^7.22.7",
	"@symfony/webpack-encore": "^4.3.0",
	"babel-loader": "^9.1.3",
	"mini-css-extract-plugin": "^2.7.6",
	"pnp-webpack-plugin": "^1.7.0",
	"webpack": "^5.88.1",
	"webpack-cli": "^5.1.4",
	"webpack-manifest-plugin": "^5.0.0",
	"webpack-notifier": "1.15.0"
  },
  "scripts": {
	"dev-server": "encore dev-server",
	"dev": "encore dev",
	"watch": "encore dev --watch",
	"build": "encore production --progress"
  },
  "dependencies": {
	"@popperjs/core": "^2.11.8",
	"bootstrap": "5.3.3",
	"font-awesome": "^4.7.0",
	"jquery": "^3.7.1"
  }
}
```

</prism>

Después de agregar la configuración, ejecuta el siguiente comando en la terminal para instalar las dependencias:

<prism lang="bash" line-numbers="true">

```bash
yarn install
```

</prism>

## Configurar Webpack Encore

Crea un archivo `webpack.config.js` en la raíz del proyecto y agrega la siguiente configuración:

<prism lang="javascript" line-numbers="true">

```javascript
const Encore = require('@symfony/webpack-encore');

// Manually configure the runtime environment if not already configured yet by the "encore" command.
// It's useful when you use tools that rely on webpack.config.js file.
if (!Encore.isRuntimeEnvironmentConfigured()) {
	Encore.configureRuntimeEnvironment(process.env.NODE_ENV || 'dev');
}

Encore
	// directory where compiled assets will be stored
	.setOutputPath('public/build/')
	// public path used by the web server to access the output path
	.setPublicPath('http://localhost/cms_ims/build')
	.configureFilenames({
		js: '[name].js',
		css: '[name].css',
		assets: 'assets/[name].[ext]',
	})
	// only needed for CDN's or subdirectory deploy
	.setManifestKeyPrefix('build/')

	/*
     * ENTRY CONFIG
     *
     * Each entry will result in one JavaScript file (e.g. app.js)
     * and one CSS file (e.g. app.css) if your JavaScript imports CSS.
     */
	.addEntry('app', './assets/app.js')

	// When enabled, Webpack "splits" your files into smaller pieces for greater optimization.
	.splitEntryChunks()

	// will require an extra script tag for runtime.js
	// but, you probably want this, unless you're building a single-page app
	.enableSingleRuntimeChunk()

	/*
     * FEATURE CONFIG
     *
     * Enable & configure other features below. For a full
     * list of features, see:
     * https://symfony.com/doc/current/frontend.html#adding-more-features
     */
	.cleanupOutputBeforeBuild()
	.enableBuildNotifications()
	.enableSourceMaps(!Encore.isProduction())
	// enables hashed filenames (e.g. app.abc123.css)
	.enableVersioning(Encore.isProduction())

	.configureBabel((config) => {
		config.plugins.push('@babel/plugin-transform-class-properties');
	})

	// enables @babel/preset-env polyfills
	.configureBabelPresetEnv((config) => {
		config.useBuiltIns = 'usage';
		config.corejs = 3;
	})

	// enables Sass/SCSS support
	//.enableSassLoader()

	// uncomment if you use TypeScript
	//.enableTypeScriptLoader()

	// uncomment if you use React
	//.enableReactPreset()

	// uncomment to get integrity="..." attributes on your script & link tags
	// requires WebpackEncoreBundle 1.4 or higher
	//.enableIntegrityHashes(Encore.isProduction())

	// uncomment if you're having problems with a jQuery plugin
	.autoProvidejQuery()
;

module.exports = Encore.getWebpackConfig();
```

</prism>

### Crear el archivo de assets

Crea un archivo `assets/app.js` en la raíz del proyecto y agrega el siguiente contenido:

<prism lang="javascript" line-numbers="true">

```javascript
//JS
import 'bootstrap/dist/js/bootstrap';
//CSS
import 'bootstrap/dist/css/bootstrap.css';
import 'font-awesome/css/font-awesome.css';
import './styles/app.css';
```

</prism>

Crea un archivo `assets/styles/app.css` en la raíz del proyecto y agrega el siguiente contenido:

<prism lang="css" line-numbers="true">

```css
/* assets/styles/app.css */
body {
    background-color: #2678c9;
    font-size: 18px;
    color: #fff;
}
```

</prism>

### Compilar los assets

Ejecuta el siguiente comando en la terminal para compilar los assets:

<prism lang="bash" line-numbers="true">

```bash
yarn build
```

</prism>

## Crear una vista

Crea un archivo `templates/home/base.html.twig` en la raíz del proyecto y agrega el siguiente contenido:

<prism lang="twig" line-numbers="true">

```twig
<!DOCTYPE html>
<html lang="es">
<head>
    <base href="./"/>
    <meta charset="UTF-8"/>
    <meta http-equiv="X-UA-Compatible" content="IE=edge"/>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <title>Home</title>
    {% block javascripts %}
        {{ encore_entry_script_tags('app', {
            defer: true
        }) }}
    {% endblock %}
    {% block stylesheets %}
        {{ encore_entry_link_tags('app') }}
    {% endblock %}
</head>
<body class="container">
<h1>Mi Página</h1>
<main class="container">
    {% block content %}
        {{ data }}
    {% endblock %}
    <hr/>
    <i class="fa fa-address-card"></i>
    <i class="fa fa-address-card-o"></i>
    <i class="fa fa-amazon"></i>
    <i class="fa fa-ambulance"></i>
</main>
<hr />
<footer class="container">
    <p>&copy; 2024 - All rights reserved</p>
</footer>
</body>
</html>
```

</prism>

Crea un archivo `templates/index.html.twig` en la raíz del proyecto y agrega el siguiente contenido:

<prism lang="twig" line-numbers="true">

```twig
{% extends 'base.html.twig' %}

{% block content %}
    <h2>Home</h2>
    <p>Bienvenido a mi página</p>
{% endblock %}
```

</prism>

## Usar una vista

Como usaremos diversos controladores en nuestro proyecto, necesitamos configurar un controlador abstracto que nos
permita renderizar las vistas. Crea un archivo `src/Controller/AbstractController.php` y agrega el siguiente contenido:

<prism lang="php" line-numbers="true">

```php
<?php

namespace Cms\Controller;

use Laminas\Diactoros\Response;
use Psr\Container\ContainerInterface;

abstract class AbstractController
{
    protected ContainerInterface $container;

    public function __construct(ContainerInterface $container)
    {
        $this->container = $container;
    }

    public function render($response, $template, $data = []): Response
    {
        return $this->container->get('view')->render($response, $template, $data);
    }
}
```

</prism>

Después de crear el controlador abstracto, necesitamos configurar el contenedor de dependencias para que pueda inyectar
el controlador en los controladores hijos. Abre el archivo `config/container.php` y agrega el siguiente contenido:

<prism lang="php" line-numbers="true">

```php
<?php

use Cms\Controller\AbstractController;use Cms\Controller\HomeController;use Lcharette\WebpackEncoreTwig\EntrypointsTwigExtension;use Lcharette\WebpackEncoreTwig\TagRenderer;use Psr\Container\ContainerInterface;use Slim\App;use Slim\Factory\AppFactory;use Slim\Views\Twig;use Symfony\Bridge\Twig\Extension\FormExtension;use Symfony\Bridge\Twig\Extension\TranslationExtension;use Symfony\Bridge\Twig\Form\TwigRendererEngine;use Symfony\Component\Form\FormRenderer;use Symfony\Component\Security\Csrf\CsrfTokenManager;use Symfony\Component\Security\Csrf\TokenGenerator\UriSafeTokenGenerator;use Symfony\Component\Security\Csrf\TokenStorage\NativeSessionTokenStorage;use Symfony\Component\Translation\Translator;use Symfony\WebpackEncoreBundle\Asset\EntrypointLookup;use Twig\RuntimeLoader\FactoryRuntimeLoader;

return [
	//APP
	App::class => function (ContainerInterface $container) {

		$app = AppFactory::createFromContainer($container);
		(require_once __DIR__ . '/middleware.php')($app);
		(require_once __DIR__ . '/routes.php')($app);
		return $app;
	},
	//Controladores
	HomeController::class => function (ContainerInterface $container) {

		return new HomeController($container);
	},
	//Twig
	'view' => function (ContainerInterface $container) {

		$csrfGenerator = new UriSafeTokenGenerator();
		$csrfStorage = new NativeSessionTokenStorage();
		$csrfManager = new CsrfTokenManager($csrfGenerator, $csrfStorage);
		$defaultFormTheme = 'bootstrap_5_layout.html.twig';
		$appVariableReflection = new \ReflectionClass('\Symfony\Bridge\Twig\AppVariable');
		$vendorTwigBridgeDir = dirname($appVariableReflection->getFileName());
		$twig = Twig::create([
			__DIR__ . '/../templates',
			$vendorTwigBridgeDir . '/Resources/views/Form'
		], ['cache' => false]);
		$formEngine = new TwigRendererEngine([
			$defaultFormTheme,
		], $twig->getEnvironment());
		$twig->addRuntimeLoader(new FactoryRuntimeLoader([
			FormRenderer::class => function () use ($formEngine, $csrfManager) {

				return new FormRenderer($formEngine, $csrfManager);
			}
		]));
		$entryPoints = new EntrypointLookup(__DIR__ . '/../public/build/entrypoints.json');
		$tagRenderer = new TagRenderer($entryPoints);
		$extension = new EntrypointsTwigExtension($entryPoints, $tagRenderer);
		$twig->addExtension($extension);
		$twig->addExtension(new FormExtension());
		$twig->addExtension(new TranslationExtension(new Translator('en')));
		return $twig;
	},
];
```

</ prism>

### Modifica el HomeController

Abre el archivo `src/Controller/HomeController.php` y modifica el contenido de la clase `HomeController` de la siguiente
manera:

<prism lang="php" line-numbers="true">

```php
<?php

namespace Cms\Controller;

use Cms\Book;use Cms\BookQuery;use Laminas\Diactoros\Response as Response;use Laminas\Diactoros\ServerRequest as Request;use Propel\Runtime\Exception\PropelException;use Psr\Container\ContainerExceptionInterface;use Psr\Container\ContainerInterface;use Psr\Container\NotFoundExceptionInterface;

class HomeController extends AbstractController
{
    public function index(Request $request, Response $response, $args): Response
    {

        return $this->render($response, 'index.html.twig', [
            'data' => 'Hello, World!'
        ]);
    }

    public function hello(Request $request, Response $response, $args): Response
    {

        return $this->render($response, 'hello.html.twig', [
            'data' => 'Hello, World!',
            'name' => $args['name']
        ]);
    }
}
```

</ prism>

### Modifica el archivo de rutas

Abre el archivo `config/routes.php` y modifica el contenido de la función anónima de la siguiente manera:

<prism lang="php" line-numbers="true">

```php
<?php

use Cms\Controller\HomeController;
use Slim\App;

return function (App $app) {
    $app->get('/[index]', [HomeController::class, 'index']);
    $app->get('/hello/{name}', [HomeController::class, 'hello']);
};
```

</ prism>

De esta manera, hemos configurado Slim para que pueda renderizar vistas en nuestra aplicación. Ahora puedes crear tantas
vistas como necesites y usarlas en tus controladores.

## Conclusión

En este tutorial, aprendiste cómo crear y usar vistas en Slim. Las vistas son una parte esencial de cualquier
aplicación web, ya que permiten separar la lógica de presentación de la lógica de la aplicación. Al seguir este
tutorial, podrás crear aplicaciones web más mantenibles y escalables con Slim. ¡Buena suerte!