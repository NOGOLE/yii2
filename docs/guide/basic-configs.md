Configurations
==============

Configurations are widely used in Yii for creating new objects or initializing existing objects.
They usually include the class names of the objects being created and a list of initial values
that should be assigned to object [properties](basic-properties.md). They may also include a list of
handlers that should be attached to the object [events](basic-events.md), and/or a list of
[behaviors](basic-behaviors.md) that should be attached to the objects.

In the following, a configuration is used to create and initialize a DB connection:

```php
$config = [
    'class' => 'yii\db\Connection',
    'dsn' => 'mysql:host=127.0.0.1;dbname=demo',
    'username' => 'root',
    'password' => '',
    'charset' => 'utf8',
];

$db = Yii::createObject($config);
```

The [[Yii::createObject()]] method takes a configuration and creates an object based on the class name
specified in the configuration. When the object is being instantiated, the rest of the configuration
will be used to initialize the object properties, event handlers and/or behaviors.

If you already have an object, you may use [[Yii::configure()]] to initialize the object properties with
a configuration, like the following,

```php
Yii::configure($object, $config);
```

Note that in this case, the configuration should not contain the `class` element.


Configuration Format
--------------------

The format of a configuration can be formally described as follows,

```php
[
    'class' => 'ClassName',
    'propertyName' => 'propertyValue',
    'on eventName' => $eventHandler,
    'as behaviorName' => $behaviorConfig,
]
```

where

* The `class` element specifies a fully qualified class name for the object being created.
* The `propertyName` elements specify the property initial values. The keys are the property names, and the
  values are the corresponding initial values. Only public member variables and [properties](basic-properties.md)
  defined by getters/setters can be configured.
* The `on eventName` elements specify what handlers should be attached to the object [events](basic-events.md).
  Notice that the array keys are formed by prefixing event names with `on `. Please refer to
  the [Events](basic-events.md) chapter for supported event handler formats.
* And the `as behaviorName` elements specify what [behaviors](basic-behaviors.md) should be attached to the object.
  Notice that the array keys are formed by prefixing behavior names with `on `. `$behaviorConfig` represents
  the configuration for creating a behavior, like a normal configuration as we are describing here.

Below is an example showing a configuration with property initial values, event handlers and behaviors:

```php
[
    'class' => 'app\components\SearchEngine',
    'apiKey' => 'xxxxxxxx',
    'on search' => function ($event) {
        Yii::info("Keyword searched: " . $event->keyword);
    },
    'as indexer' => [
        'class' => 'app\components\IndexerBehavior',
        // ... property init values ...
    ],
]
```


Using Configurations
--------------------

Configurations are used in many places in Yii. At the beginning of this chapter, we have shown how to use
create an object according to a configuration by using [[Yii::createObject()]]. In this section, we will
describe application configurations and widget configurations - two major usages of configurations.


### Application Configurations

Configuration for an [application](structure-applications.md) is probably one of the most complex configurations.
This is because the [[yii\web\Application|application]] class has a lot of configurable properties and events.
More importantly, its [[yii\web\Application::components|components]] property can receive an array of configurations
for creating components that are registered through the application. The following is an abstract from the application
configuration file for the [basic application template](start-basic.md).

```php
$config = [
    'id' => 'basic',
    'basePath' => dirname(__DIR__),
    'extensions' => require(__DIR__ . '/../vendor/yiisoft/extensions.php'),
    'components' => [
        'cache' => [
            'class' => 'yii\caching\FileCache',
        ],
        'mail' => [
            'class' => 'yii\swiftmailer\Mailer',
        ],
        'log' => [
            'class' => 'yii\log\Dispatcher',
            'traceLevel' => YII_DEBUG ? 3 : 0,
            'targets' => [
                [
                    'class' => 'yii\log\FileTarget',
                ],
            ],
        ],
        'db' => [
            'class' => 'yii\db\Connection',
            'dsn' => 'mysql:host=localhost;dbname=stay2',
            'username' => 'root',
            'password' => '',
            'charset' => 'utf8',
        ],
    ],
];
```

The configuration does not have a `class` key. This is because it is used as follows in
an [entry script](structure-entry-scripts.md), where the class name is already given,

```php
(new yii\web\Application($config))->run();
```

For more details about configuring the `components` property of an application can be found
in the [Applications](structure-applications.md) chapter and the [Service Locator](basic-service-locator.md) chapter.


### Widget Configurations

When using [widgets](structure-widgets.md), you often need to use configurations to customize the widget properties.
Both of the [[yii\base\Widget::widget()]] and [[yii\base\Widget::beginWidget()]] methods can be used to create
a widget. They take a configuration array, like the following,

```php
use yii\widgets\Menu;

echo Menu::widget([
    'activateItems' => false,
    'items' => [
        ['label' => 'Home', 'url' => ['site/index']],
        ['label' => 'Products', 'url' => ['product/index']],
        ['label' => 'Login', 'url' => ['site/login'], 'visible' => Yii::$app->user->isGuest],
    ],
]);
```

The above code creates a `Menu` widget and initializes its `activeItems` property to be false.
The `items` property is also configured with menu items to be displayed.

Note that because the class name is already given, the configuration array should NOT have the `class` key.


Configuration Files
-------------------

When a configuration is very complex, a common practice is to store it in one or multiple PHP files, known as
*configuration files*. To use a configuration, simply "require" the corresponding configuration file.
For example, you may keep an application configuration in a file named `web.php`, like the following,

```php
return [
    'id' => 'basic',
    'basePath' => dirname(__DIR__),
    'extensions' => require(__DIR__ . '/../vendor/yiisoft/extensions.php'),
    'components' => require(__DIR__ . '/components.php'),
];
```

Because the `components` configuration is complex too, you store it in a separate file called `components.php`
and "require" this file in `web.php` as shown above. The content of `components.php` is as follows,

```php
return [
    'cache' => [
        'class' => 'yii\caching\FileCache',
    ],
    'mail' => [
        'class' => 'yii\swiftmailer\Mailer',
    ],
    'log' => [
        'class' => 'yii\log\Dispatcher',
        'traceLevel' => YII_DEBUG ? 3 : 0,
        'targets' => [
            [
                'class' => 'yii\log\FileTarget',
            ],
        ],
    ],
    'db' => [
        'class' => 'yii\db\Connection',
        'dsn' => 'mysql:host=localhost;dbname=stay2',
        'username' => 'root',
        'password' => '',
        'charset' => 'utf8',
    ],
];
```

And the code for starting an application becomes,

```php
$config = require('path/to/web.php');
(new yii\web\Application($config))->run();
```


Default Configurations
----------------------

The [[Yii::createObject()]] method is implemented based on a [dependency injection container](basic-di-container.md).
It allows you specify a set of the so-called *default configurations* which will be applied to ANY instances of
the specified classes when they are being created using [[Yii::createObject()]]. The default configurations
can be specified by calling `Yii::$container->set()` in the [bootstrapping](runtime-bootstrapping.md) code.

For example, if you want to customize [[yii\widgets\LinkPager]] so that ALL link pagers will show at most 5 page buttons
(the default value is 10), you may use the following code to achieve this goal,

```php
\Yii::$container->set('yii\widgets\LinkPager', [
    'maxButtonCount' => 5,
]);
```

Without using default configurations, you would have to configure `maxButtonCount` in every place where you use
link pagers.
