---
weight: 402
title: "ORM"
description: ""
icon: "article"
date: "2024-06-24T21:05:47-06:00"
lastmod: "2024-06-24T21:05:47-06:00"
draft: false
toc: true
---

## ORM

Un ORM (Object-Relational Mapping) es una técnica de programación que permite convertir datos entre sistemas
incompatibles mediante el uso de objetos de programación orientada a objetos. En este tutorial, usaremos PropelORM, un
ORM de alto rendimiento para PHP.

## Instalar PropelORM

Para instalar PropelORM, ejecuta el siguiente comando en la terminal:

```bash
composer require propel/propel
```

Una vez que Composer haya terminado, se creará el directorio `vendor` en tu proyecto.

## Configurar PropelORM

Para configurar PropelORM, necesitas ejecutar el siguiente comando en la terminal:

```bash
php vendor/bin/propel init
```

Aparecerá un mensaje que te pedirá que elijas el gestor de bases de datos que deseas usar. En este caso, selecciona
MySQL.

Después de seleccionar MySQL, PropelORM nos solicitará la información de la conexión a la base de datos. Proporciona la
información requerida y PropelORM detectará automáticamente la base de datos y creará un archivo de configuración.

Seguidamente, el sistema te solicitará el directorio en donde deseas guardar los archivos generados. En este caso, elige
`src`. Así mismo te solicitará el nombre del espacio de nombres. En este caso, elige `Cms`. Finalmente, te solicitará el
tipo de archivo de configuración. En este caso, elige `xml`.

Una vez que hayas completado estos pasos, PropelORM generará los archivos necesarios en el directorio `src`.

Recuerda mover los archivos generados de la carpeta `Cms` a `src`. Así como los archivos de configuración de la carpeta
`generated_conf` a `config`.

## Actualizar el archivo `composer.json`

Para que PropelORM funcione correctamente, necesitas actualizar el archivo `composer.json` con el siguiente contenido:

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

Es importante para que este proceso y el siguiente funcionen correctamente, que tengas instalado Composer en tu sistema.

Si no lo tienes, puedes descargarlo desde [getcomposer.org](https://getcomposer.org/). De lo contrario, no podrás
continuar con el tutorial.

Así mismo es necesario descargar Yarn, puedes descargarlo
desde [yarnpkg.com](https://yarnpkg.com/getting-started/install).

## Instalar dependencias

Para instalar las dependencias, ejecuta el siguiente comando en la terminal:

```bash
composer install
```

Una vez que Composer haya terminado, se creará el directorio `vendor` en tu proyecto.

## Actualizar el archivo `config/container.php`

Para usar PropelORM en tu proyecto, necesitas actualizar el archivo `config/container.php` con el siguiente contenido:

{{< prism lang="php" line-numbers="true" >}}

    <?php
    
    use Cms\Controller\CategoryController;
    use Cms\Controller\HomeController;
    use Cms\Extensions\RouterExtension;
    use Lcharette\WebpackEncoreTwig\EntrypointsTwigExtension;
    use Lcharette\WebpackEncoreTwig\TagRenderer;
    use Psr\Container\ContainerInterface;
    use Slim\App;
    use Slim\Factory\AppFactory;
    use Slim\Views\Twig;
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
        //Router
        'router' => function (ContainerInterface $container) {
    
            return $container->get(App::class)->getRouteCollector()->getRouteParser();
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
            return $twig;
        },
    ];
{{< /prism >}}

## Actualizar el archivo `config/app.php`

Para usar PropelORM en tu proyecto, necesitas actualizar el archivo `config/app.php` con el siguiente contenido:

{{< prism lang="php" line-numbers="true" >}}

    <?php
    
    use DI\ContainerBuilder;
    use Slim\App;
    
    require_once __DIR__ . '/../vendor/autoload.php';
    require_once __DIR__ . '/config.php';
    $containerBuilder = new ContainerBuilder();
    // Add DI container definitions
    $containerBuilder->addDefinitions(__DIR__ . '/container.php');
    // Create DI container instance
    try {
        $container = $containerBuilder->build();
        $app = $container->get(App::class);
        $app->run();
    } catch (Exception $e) {
    }

{{< /prism >}}

## Crear los Modelos

Crea la clase `AbstractModel` en la carpeta src/Models y pega el siguiente contenido:

{{< prism lang="php" line-numbers="true" >}}

    <?php
    
    namespace Cms\Models;
    
    use Propel\Runtime\ActiveQuery\ModelCriteria;
    
    abstract class AbstractModel
    {
    
        protected $query;
    
        public function __construct($class)
        {
    
            $this->query = new ModelCriteria(null, $class);
        }
    
        public abstract function getAll();
    
        public abstract function getById($id);
    
        public abstract function deleteById($id);
    
        public abstract function save($data);
    
        public abstract function update($data);
    }

{{< /prism >}}

Como ejemplo veremos el modelo de la tabla `category`.

Crea la clase `CategoryModel` en la carpeta src/Models y pega el siguiente contenido:

{{< prism lang="php" line-numbers="true" >}}

    <?php
    
    namespace Cms\Models;
    
    use Cms\Map\CategoryTableMap;
    use Laminas\Diactoros\Response;
    use Laminas\Diactoros\ServerRequest as Request;
    use Propel\Runtime\ActiveQuery\Criteria;
    use Propel\Runtime\Exception\PropelException;
    use Slim\Routing\RouteContext;
    
    class CategoryModel extends AbstractModel
    {
        public function __construct()
        {
    
            parent::__construct(CategoryTableMap::OM_CLASS);
        }
    
        /**
         * @return mixed|\Propel\Runtime\Collection\Collection $categories
         */
        public function getAll(): mixed
        {
    
            return $this->query->find();
        }
    
        public function tableLoader(Request $request, Response $response, array $args): Response
        {
            $search = $args['search'] ?? null;
            $limit = $args['limit'] ?? null;
            $offset = $args['offset'] ?? null;
            if (is_null($limit) || is_null($offset)) {
    
                return $response;
            }
            $router = RouteContext::fromRequest($request)->getRouteParser();
            $next = $router->fullUrlFor($request->getUri(), "category_table") . "/$limit/" . ($offset + $limit);
            $previous = $offset == 0 ? null : $router->fullUrlFor($request->getUri(), "category_table") . "/$limit/" . ($offset - $limit);
            if (!is_null($search)) {
                $count = $this->query->filterByCategoryName("%" . $search . "%", Criteria::LIKE)->count();
                $data = $this->query->filterByCategoryName( "%" . $search . "%", Criteria::LIKE)->limit($limit)->offset($offset);
            } else {
                $count = $this->query->find()->count();
                $data = $this->query->limit($limit)->offset($offset);
            }
            $output = [
                'count' => $count,
                'next' => $next,
                'previous' => $previous
            ];
            foreach ($data as $category) {
    
                $output['results'][] = [
                    'edit' => "<a class='btn btn-primary edit' entity='" . $category->getIdCategory() . "'>" .
                        "<i class='cil-pencil pe-2'></i>Editar</a>",
                    'name' => $category->getCategoryName(),
                    'father' => !is_null($category->getParentCategory()) ? $category->getCategoryRelatedByParentCategory()->getCategoryName() : "",
                ];
            }
            $response->getBody()->write(json_encode($output));
            return $response->withHeader("Content-Type", "application/json");
        }
    
        public function getById($id)
        {
    
            return $this->query->findPk($id);
        }
    
        public function deleteById($id): bool
        {
    
            $category = $this->query->findPk($id);
            try {
                $category->delete();
            } catch (PropelException $e) {
                return false;
            }
            return true;
        }
    
        public function save($data): bool
        {
    
            try {
                $data->setNew(true);
                $data->save();
            } catch (PropelException $e) {
                return false;
            }
            return true;
        }
    
        public function update($data): bool
        {
            try {
                $data->setNew(false);
                $data->save();
            } catch (PropelException $e) {
                return false;
            }
            return true;
        }
    }

{{< /prism >}}

## Modificar el archivo `Category`

Crea la clase `Category` en la carpeta src y pega el siguiente contenido:

{{< prism lang="php" line-numbers="true" >}}

    <?php
    
    namespace Cms;
    
    use Cms\Base\Category as BaseCategory;
    
    /**
     * Skeleton subclass for representing a row from the 'category' table.
     *
     *
     *
     * You should add additional methods to this class to meet the
     * application requirements.  This class will only be generated as
     * long as it does not already exist in the output directory.
     */
    class Category extends BaseCategory {
        public function setParentCategory($v) {
    
            if ($v instanceof Category) {
                parent::setParentCategory($v->getCategoryId());
            } else {
                parent::setParentCategory($v);
            }
        }
    
        public function getParentCategoryName(): string {
    
            return CategoryQuery::create()->findPk($this->getParentCategory())->getCategoryName();
        }
    }

{{< /prism >}}

## Crear los Formularios

Crea la clase `CategoryForm` en la carpeta src/Forms y pega el siguiente contenido:

{{< prism lang="php" line-numbers="true" >}}

    <?php
    
    namespace Cms\Forms;
    
    use Cms\CategoryQuery;
    use Cms\Models\CategoryModel;
    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\Extension\Core\Type\ButtonType;
    use Symfony\Component\Form\Extension\Core\Type\ChoiceType;
    use Symfony\Component\Form\Extension\Core\Type\SubmitType;
    use Symfony\Component\Form\Extension\Core\Type\TextType;
    use Symfony\Component\Form\FormBuilderInterface;
    use Symfony\Component\OptionsResolver\OptionsResolver;
    
    class CategoryForm extends AbstractType
    {
        private CategoryModel $model;
    
        public function __construct()
        {
    
            $this->model = new CategoryModel();
        }
    
        public function buildForm(FormBuilderInterface $builder, array $options): void
        {
    
            $builder->setAction($options['attr']['action']);
            $builder->add('categoryName', TextType::class, [
                'label' => 'Nombre de la Categoría',
                'required' => true,
            ])->add('parentCategory', ChoiceType::class, [
                'label' => 'Categoria Padre',
                'required' => false,
                'choices' => $this->getCategoryList(),
                'choice_label' => function ($choice, $key, $value) {
    
                    return $choice->getCategoryName();
                },
                'choice_value' => function ($choice) {
    
                    if (is_int($choice)) {
    
                        $query = new CategoryQuery();
                        $choice = $query->create()
                            ->findOneByIdCategory($choice);
                    }
                    return $choice ? $choice->getCategoryId() : '';
                },
            ])->add('save', SubmitType::class, [
                'label' => 'Save',
            ])->add('cancel', ButtonType::class, [
                'label' => 'Cancel',
                'attr' => [
                    'onclick' => 'window.location.href = "' . $options['attr']['url'] . '"',
                ],
            ]);
        }
    
        public function configureOptions(OptionsResolver $resolver): void
        {
    
            $resolver->setDefaults([
                'data_class' => 'Cms\Category',
            ]);
        }
    
        private function getCategoryList(): array
        {
    
            $categories = $this->model->getAll();
            $list = [];
            foreach ($categories as $category) {
                $list[$category->getCategoryId()] = $category;
            }
            return $list;
        }
    }

{{< /prism >}}

## Actualizar la clase `AbstractController`

Para usar PropelORM en tu proyecto, necesitas actualizar la clase `AbstractController` con el siguiente contenido:

{{< prism lang="php" line-numbers="true" >}}

    <?php
    
    namespace Cms\Controller;
    
    use Laminas\Diactoros\Response;
    use Psr\Container\ContainerExceptionInterface;
    use Psr\Container\ContainerInterface;
    use Psr\Container\NotFoundExceptionInterface;
    use Symfony\Component\Form\FormInterface;
    
    abstract class AbstractController
    {
        protected ContainerInterface $container;
    
        public function __construct(ContainerInterface $container)
        {
    
            $this->container = $container;
        }
    
        public function render(Response $response, $template, $data = []): Response
        {
    
            try {
                return $this->container->get('view')->render($response, $template, $data);
            } catch (NotFoundExceptionInterface|ContainerExceptionInterface $e) {
                return $response;
            }
        }
    
        public function redirect($response, $url): Response
        {
    
            return $response->withHeader('Location', $url)->withStatus(302);
        }
    
        public function createForm($entity, $class, $id, $method): ?FormInterface
        {
    
            try {
                $factory = $this->container->get('form.factory');
                return $factory->createBuilder($class, $entity, [
                    'attr' => [
                        'id' => $id,
                        'url' => $this->container->get('router')->urlFor($id),
                        'action' => $this->container->get('router')->urlFor($id . "_" . $method)
                    ]
                ])->getForm();
            } catch (NotFoundExceptionInterface|ContainerExceptionInterface $e) {
                return null;
            }
        }
    }

{{< /prism >}}

## Crear los Controladores

Crea la clase `CategoryController` en la carpeta src/Controller y pega el siguiente contenido:

{{< prism lang="php" line-numbers="true" >}}

    <?php
    
    namespace Cms\Controller;
    
    use Cms\Category;
    use Cms\Forms\CategoryForm;
    use Cms\Models\CategoryModel;
    use Laminas\Diactoros\Response as Response;
    use Laminas\Diactoros\ServerRequest as Request;
    use Psr\Container\ContainerInterface;
    use Symfony\Component\HttpFoundation\Request as SymfonyRequest;
    
    class CategoryController extends AbstractController
    {
        private CategoryModel $model;
    
        public function __construct(ContainerInterface $container)
        {
    
            parent::__construct($container);
            $this->model = new CategoryModel();
        }
    
        public function index(Request $request, Response $response, $args): Response
        {
    
            $categories = $this->model->getAll();
            return $this->render($response, 'category/index.html.twig', [
                'categories' => $categories,
            ]);
        }
    
        public function add(Request $request, Response $response, $args): Response
        {
    
            $req = SymfonyRequest::createFromGlobals();
            $category = new Category();
            $form = $this->createForm($category, CategoryForm::class, 'category', 'add');
            $form->handleRequest($req);
            if ($form->isSubmitted() && $form->isValid()) {
    
                $this->model->save($category);
                return $this->redirect($response, $this->container->get('router')->urlFor('category'));
            }
            return $this->render($response, 'forms.html.twig', [
                'form' => $form->createView(),
                'form_title' => 'Añadir Categoría'
            ]);
        }
    
        public function table(Request $request, Response $response, $args): Response
        {
            return $this->model->tableLoader($request, $response, $args);
        }
    }

{{< /prism >}}

## Actualizar el archivo `config/routes.php`

Para usar PropelORM en tu proyecto, necesitas actualizar el archivo `config/routes.php` con el siguiente contenido:

{{< prism lang="php" line-numbers="true" >}}

    <?php
    
    use Cms\Controller\CategoryController;
    use Cms\Controller\HomeController;
    use Slim\App;
    
    return function (App $app) {
    
        $app->get('/[index]', [
            HomeController::class,
            'index'
        ]);
        $app->get('/hello/{name}', [
            HomeController::class,
            'hello'
        ]);
        $app->group('/category', function ($group) {
    
            $group->get('', [
                CategoryController::class,
                'index'
            ])->setName('category');
            $group->map(['GET', 'POST'], '/add', [
                CategoryController::class,
                'add'
            ])->setName('category_add');
            $group->get('/edit/{id}', [
                CategoryController::class,
                'edit'
            ])->setName('category_edit');
            $group->get('/delete/{id}', [
                CategoryController::class,
                'delete'
            ])->setName('category_delete');
            $group->get("/tableData[/{limit}/{offset}]", [
                CategoryController::class,
                "table"
            ])->setName("category_table");
            $group->get("/tableData/{search}/{limit}/{offset}", [
                CategoryController::class,
                "table"
            ])->setName("category_search");
        });
    };

{{< /prism >}}

## Las Vistas

Crea la vista `category/index.html.twig` en la carpeta templates y pega el siguiente contenido:

{{< prism lang="twig" line-numbers="true" >}}

    {% extends 'base.html.twig' %}
    {% block javascripts %}
    {{ parent() }}
    {{ encore_entry_script_tags('category', {
    defer: true
    }) }}
    {% endblock %}
    
    {% block stylesheets %}
    {{ parent() }}
    {{ encore_entry_link_tags('category') }}
    {% endblock %}
    {% block content %}
    <h1>Lista de Categorías</h1>
    <div>
    <a class="btn btn-primary" href="{{ path('category_add') }}">Agregar Categoría</a>
    </div>
    <hr/>
    <div id="table-loader" loader="{{ path('category_table') }}"></div>
    <div id="wrapper"></div>
    <form id="entityForm" method="post" action="">
    <input type="hidden" name="id" id="idEntity" value=""/>
    <input type="hidden" id="editLink" value="{{ path("category_add") }}"/>
    </form>
    {% endblock %}

{{< /prism >}}

Crea la vista `forms.html.twig` en la carpeta templates y pega el siguiente contenido:

{{< prism lang="twig" line-numbers="true" >}}

    {% extends 'base.html.twig' %}
    {% block content %}
    <h1>{{ form_title }}</h1>
    <form method="post" action="{{ form.vars.attr.url }}">
    {{ form_start(form) }}
    {{ form_widget(form) }}
    {{ form_end(form) }}
    </form>
    {% endblock %}

{{< /prism >}}