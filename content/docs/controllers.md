+++
weight = 200
date = "2023-05-03T22:37:22+01:00"
draft = false
title = "Los controladores"
icon = "code"
toc = true
publishdate = "2023-05-03T22:37:22+01:00"
+++

## Introducción

Los controladores son responsables de manejar las solicitudes HTTP y devolver una respuesta. En Slim, los controladores
son clases PHP que contienen métodos de acción. Cada método de acción corresponde a una ruta y un verbo HTTP específico.

## Crear un controlador

Para crear un controlador, crea un archivo en el directorio `src/Controller` y pega el siguiente contenido:

{{< prism lang="php" line-numbers="true" >}}

    <?php
    
    namespace Cms\Controller;
    
    use Laminas\Diactoros\Response as Response;
    use Laminas\Diactoros\ServerRequest as Request;
    use Psr\Container\ContainerInterface;
    
    class HomeController {
        private ContainerInterface $container;
    
        public function __construct(ContainerInterface $container) {
    
            $this->container = $container;
        }
    
        public function index(Request $request, Response $response, $args): Response {
    
            $response->getBody()->write('Home Controller');
            return $response;
        }
    
        public function hello(Request $request, Response $response, $args): Response {
    
            $response->getBody()->write("¡Hola, {$args['name']}!");
            return $response;
        }
    }

{{< /prism >}}

## Usar un controlador

Para usar un controlador, necesitas definirlo en el contenedor de dependencias. Crea un archivo `config/container.php` y
pega el siguiente contenido:

{{< prism lang="php" line-numbers="true" >}}

    <?php
    
    use Cms\Controller\HomeController;
    use Psr\Container\ContainerInterface;
    use Slim\App;
    use Slim\Factory\AppFactory;
    
    return [
        //APP
        App::class => function (ContainerInterface $container) {
    
            $app = AppFactory::createFromContainer($container);
            (require_once __DIR__ . '/routes.php')($app);
            (require_once __DIR__ . '/middleware.php')($app);
            return $app;
        },
        HomeController::class => function (ContainerInterface $container) {
    
            return new HomeController($container);
        },
    ];

{{< /prism >}}

Como puedes ver, hemos definido un controlador `HomeController` en el contenedor de dependencias. Ahora puedes usar el
controlador en tus rutas.

## Usar un controlador en una ruta

Para usar un controlador en una ruta, crea un archivo `config/routes.php` y pega el siguiente contenido:

{{< prism lang="php" line-numbers="true" >}}

    <?php
    
    use Cms\Controller\HomeController;
    use Slim\App;
    
    return function (App $app) {
        $app->get('/', [HomeController::class, 'index']);
        $app->get('/hello/{name}', [HomeController::class, 'hello']);
    };

{{< /prism >}}

En este ejemplo, hemos definido dos rutas. La primera ruta llama al método `index` del controlador `HomeController`
cuando se accede a la raíz del sitio. La segunda ruta llama al método `hello` del controlador `HomeController` y pasa un
parámetro `name` en la URL.

## Resumen

En este tutorial, aprendiste cómo crear y usar controladores en Slim. Los controladores son una forma de organizar tu
código y mantenerlo limpio y mantenible. Puedes definir múltiples controladores y métodos de acción para manejar
diferentes rutas y verbos HTTP en tu aplicación Slim.

Ahora que sabes cómo usar controladores en Slim, puedes comenzar a construir aplicaciones web más complejas y
escalables. ¡Buena suerte!