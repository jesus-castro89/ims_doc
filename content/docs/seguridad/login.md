---
weight: 502
title: "Login"
description: ""
icon: "article"
date: "2024-06-24T21:05:47-06:00"
lastmod: "2024-06-24T21:05:47-06:00"
draft: false
toc: true
---

## Introducción

Crea la clase SecurityAcl.php en el directorio src/Security y pega el siguiente contenido:

{{< prism lang="php" line-numbers="true" >}}

    <?php
    
    namespace Cms\Security;
    
    use Cms\ResourcePermissionQuery;
    use Cms\ResourceQuery;
    use Cms\UserRoleQuery;
    use Laminas\Permissions\Acl\Acl;
    use Propel\Runtime\Exception\PropelException;
    
    class SecurityAcl
    {
    
        private Acl $acl;
    
        /**
         * @throws PropelException
         */
        public function __construct()
        {
            $this->acl = new Acl();
            $this->addRoles();
            $this->addResources();
            $this->addPrivileges();
        }
    
        public function addRoles(): void
        {
    
            $query = new UserRoleQuery();
            $roles = $query->find();
            foreach ($roles as $role) {
                $this->acl->addRole($role->getRoleName());
            }
        }
    
        public function addResources(): void
        {
            $query = new ResourceQuery();
            $resources = $query->find();
            foreach ($resources as $resource) {
                $this->acl->addResource($resource->getResourceName());
            }
        }
    
        /**
         * @throws PropelException
         */
        public function addPrivileges(): void
        {
            $query = new ResourcePermissionQuery();
            $query->joinWithResource();
            $query->joinWithUserRole();
            $permissions = $query->find();
            foreach ($permissions as $permission) {
    
                $this->acl->allow($permission->getUserRole()->getRoleName(), $permission->getResource()->getResourceName(),
                    $permission->getPermission());
            }
        }
    
        public function isAllowed($role, $resource, $privilege): bool
        {
            return $this->acl->isAllowed($role, $resource, $privilege);
        }
    }

{{< /prism >}}

## Cotainer

Actualiza el archivo container.php

{{< prism lang="php" line-numbers="true" >}}

    <?php
    
    use Cms\Controller\CategoryController;
    use Cms\Controller\HomeController;
    use Cms\Extensions\RouterExtension;
    use Cms\Security\SecurityAcl;
    use Lcharette\WebpackEncoreTwig\EntrypointsTwigExtension;
    use Lcharette\WebpackEncoreTwig\TagRenderer;
    use Psr\Container\ContainerInterface;
    use Slim\App;
    use Slim\Factory\AppFactory;
    use Slim\Flash\Messages;
    use Slim\Views\Twig;
    use SlimSession\Helper;
    use Symfony\Bridge\Twig\Extension\FormExtension;
    use Symfony\Bridge\Twig\Extension\TranslationExtension;
    use Symfony\Bridge\Twig\Form\TwigRendererEngine;
    use Symfony\Component\Form\Extension\Csrf\CsrfExtension;
    use Symfony\Component\Form\Extension\HttpFoundation\HttpFoundationExtension;
    use Symfony\Component\Form\Extension\Validator\ValidatorExtension;
    use Symfony\Component\Form\FormRenderer;
    use Symfony\Component\Form\Forms;
    use Symfony\Component\Security\Csrf\CsrfTokenManager;
    use Symfony\Component\Security\Csrf\TokenGenerator\UriSafeTokenGenerator;
    use Symfony\Component\Security\Csrf\TokenStorage\NativeSessionTokenStorage;
    use Symfony\Component\Translation\Translator;
    use Symfony\Component\Validator\Validation;
    use Symfony\WebpackEncoreBundle\Asset\EntrypointLookup;
    use Twig\RuntimeLoader\FactoryRuntimeLoader;
    
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
        CategoryController::class => function (ContainerInterface $container) {
    
            return new CategoryController($container);
        },
        //Router
        'router' => function (ContainerInterface $container) {
    
            return $container->get(App::class)->getRouteCollector()->getRouteParser();
        },
        //Acl
        'acl' => function (ContainerInterface $container) {
    
            return new SecurityAcl();
        },
        //Session
        'session' => function ($container) {
    
            return new Helper();
        },
        //Flash
        'flash' => function (ContainerInterface $container) {
    
            $storage = [];
            return new Messages($storage);
        },
        //Formularios
        'form.factory' => function (ContainerInterface $container) {
    
            $csrfGenerator = new UriSafeTokenGenerator();
            $csrfStorage = new NativeSessionTokenStorage();
            $csrfManager = new CsrfTokenManager($csrfGenerator, $csrfStorage);
            return Forms::createFormFactoryBuilder()->addExtensions([
                new HttpFoundationExtension(),
                new CsrfExtension($csrfManager),
                new ValidatorExtension($container->get('validator'))
            ])->getFormFactory();
        },
        'validator' => function (ContainerInterface $container) {
    
            return Validation::createValidator();
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
            $twig->addExtension(new RouterExtension($container));
            $twig->addExtension(new Glazilla\Slim\Views\TwigMessages(
                $container->get('flash')
            ));
            return $twig;
        },
    ];

{{< /prism >}}

## Middleware

Actualiza el archivo middleware.php

{{< prism lang="php" line-numbers="true" >}}

    <?php
    
    use Slim\App;
    use Slim\Middleware\Session;
    
    return function (App $app) {
    
        // Configurar el middleware de sesiones
        $app->add(new Session([
            'name' => 'slim_session',
            'autorefresh' => true,
            'lifetime' => '1 hour'
        ]));
        // Parse json, form data and xml
        $app->addBodyParsingMiddleware();
        // Add the Slim built-in routing middleware
        $app->addRoutingMiddleware();
        //Whops!
            $app->add(new Zeuxisoo\Whoops\Slim\WhoopsMiddleware([
            'enable' => true,
            'editor' => 'sublime',
            'title' => 'Parece que hay un error',
        ]));
        //Flash Messages
        $app->add(
            function ($request, $next) {
                // Start PHP session
                if (session_status() !== PHP_SESSION_ACTIVE) {
                    session_start();
                }
    
                // Change flash message storage
                $this->get('flash')->__construct($_SESSION);
    
                return $next->handle($request);
            }
        );
        //Add Base Path
        $app->setBasePath('/cms_ims');
    };

{{< /prism >}}}


## Home & Login

Agrega las siguientes funciones a la clase HomeController

{{< prism lang="php" line-numbers="true" >}}

    public function login(Request $request, Response $response, $args): Response
    {
        if ($this->container->get('session')->get('user')) {

            return $response->withHeader('Location',
                $this->container->get('router')->urlFor('category'))
                ->withStatus(302);
        }
        $factory = $this->container->get('form.factory');
        return $this->render($response, 'login.html.twig', [
            'form' => $factory->createBuilder(LoginForm::class)->getForm()->createView(),
            'flashm' => sizeof($this->container->get('flash')->getMessages())
        ]);
    }

    public function logout(Request $request, Response $response, $args): Response
    {
        $this->container->get('session')->delete('user');
        return $response->withHeader('Location',
            $this->container->get('router')->urlFor('login'))
            ->withStatus(302);
    }

    public function validateLogin(Request $request, Response $response, $args): Response
    {
        $req = $request->getParsedBody();
        $userName = $req['login_form']['userName'];
        $password = $req['login_form']['password'];
        $user = UserQuery::create()->findOneByUserName($userName);
        if ($user && hash("sha3-512", $password) == $user->getUserPass()) {

            $this->container->get('session')->set('user', $user->getUserRole()->getRoleName());
            return $response->withHeader('Location',
                $this->container->get('router')->urlFor('category'))
                ->withStatus(302);
        } else {
            $this->flash('error', 'Usuario o contraseña incorrectos');
            return $response->withHeader('Location',
                $this->container->get('router')->urlFor('login'))
                ->withStatus(302);
        }
    }

{{< /prism >}}