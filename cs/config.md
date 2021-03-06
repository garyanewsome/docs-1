<div class='article-menu'>
  <ul>
    <li>
      <a href="#overview">Čtení konfigurace</a> <ul>
        <li>
          <a href="#factory">Factory</a>
        </li>
        <li>
          <a href="#native-arrays">Nativní pole (Array)</a>
        </li>
        <li>
          <a href="#file-adapter">Adaptéry pro soubory</a>
        </li>
        <li>
          <a href="#ini-files">Čtení INI souborů</a>
        </li>
        <li>
          <a href="#merging">Slučování konfigurace</a>
        </li>
        <li>
          <a href="#nested-configuration">Vnořené konfigurace</a>
        </li>
        <li>
          <a href="#injecting-into-di">Konfigurace jako služba</a>
        </li>
      </ul>
    </li>
  </ul>
</div>

<a name='overview'></a>

# Reading Configurations

Komponenta `Phalcon\Config` se používá pro konvertování různých formátů konfiguračních souborů (s použitím adaptérů) do PHP objektů pro pouřití v aplikaci.

<a name='factory'></a>

## Factory

Loads Config Adapter class using `adapter` option, if no extension is provided it will be added to `filePath`

```php
<?php

use Phalcon\Config;

$config = new Config(
    [
        'test' => [
            'parent' => [
                'property'  => 1,
                'property2' => 'yeah',
            ],
        ],  
    ]
);

echo $config->get('test')->get('parent')->get('property');  // displays 1
echo $config->test->parent->property;                       // displays 1
echo $config->path('test.parent.property');                 // displays 1
```

<a name='factory'></a>

## Factory

Loads Config Adapter class using `adapter` option, if no extension is provided it will be added to `filePath`

```php
<?php

use Phalcon\Config\Factory;

$options = [
    'filePath' => 'path/config',
    'adapter'  => 'php',
 ];

 $config = Factory::load($options);
 ```

<a name='native-arrays'></a>
## Native Arrays
The first example shows how to convert native arrays into `Phalcon\Config` objects. This option offers the best performance since no files are read during this request.

```php
<?php

use Phalcon\Config;

$settings = [
    'database' => [
        'adapter'  => 'Mysql',
        'host'     => 'localhost',
        'username' => 'scott',
        'password' => 'cheetah',
        'dbname'   => 'test_db'
    ],
     'app' => [
        'controllersDir' => '../app/controllers/',
        'modelsDir'      => '../app/models/',
        'viewsDir'       => '../app/views/'
    ],
    'mysetting' => 'the-value'
];

$config = new Config($settings);

echo $config->app->controllersDir, "\n";
echo $config->database->username, "\n";
echo $config->mysetting, "\n";
```

If you want to better organize your project you can save the array in another file and then read it.

```php
<?php

use Phalcon\Config;

require 'config/config.php';

$config = new Config($settings);
```

<a name='file-adapter'></a>

## Adaptéry pro soubory

The adapters available are:

| Class                            | Description                                                                                             |
| -------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `Phalcon\Config\Adapter\Ini`  | Použivá INI soubory jako úložiště nastavení. Tento adaptér interně využívá PHP funkci `parse_ini_file`. |
| `Phalcon\Config\Adapter\Json` | Používá JSON soubory jako úložiště nastavení.                                                           |
| `Phalcon\Config\Adapter\Php`  | Používá vícerozměrné PHP pole jako úložiště nastavení. Tento adaptér nabízí nejlepší výkon.             |
| `Phalcon\Config\Adapter\Yaml` | Používá YAML soubory jako úložiště nastavení.                                                           |

<a name='ini-files'></a>

## Čtení INI souborů

Ini files are a common way to store settings. `Phalcon\Config` uses the optimized PHP function `parse_ini_file` to read these files. Files sections are parsed into sub-settings for easy access.

```ini
[database]
adapter  = Mysql
host     = localhost
username = scott
password = cheetah
dbname   = test_db

[phalcon]
controllersDir = '../app/controllers/'
modelsDir      = '../app/models/'
viewsDir       = '../app/views/'

[models]
metadata.adapter  = 'Memory'
```

You can read the file as follows:

```php
<?php

use Phalcon\Config\Adapter\Ini as ConfigIni;

$config = new ConfigIni('path/config.ini');

echo $config->phalcon->controllersDir, "\n";
echo $config->database->username, "\n";
echo $config->models->metadata->adapter, "\n";
```

<a name='merging'></a>

## Slučování konfigurace

`Phalcon\Config` can recursively merge the properties of one configuration object into another. New properties are added and existing properties are updated.

```php
<?php

use Phalcon\Config;

$config = new Config(
    [
        'database' => [
            'host'   => 'localhost',
            'dbname' => 'test_db',
        ],
        'debug' => 1,
    ]
);

$config2 = new Config(
    [
        'database' => [
            'dbname'   => 'production_db',
            'username' => 'scott',
            'password' => 'secret',
        ],
        'logging' => 1,
    ]
);

$config->merge($config2);

print_r($config);
```

The above code produces the following:

```bash
Phalcon\Config Object
(
    [database] => Phalcon\Config Object
        (
            [host] => localhost
            [dbname]   => production_db
            [username] => scott
            [password] => secret
        )
    [debug] => 1
    [logging] => 1
)
```

There are more adapters available for this components in the [Phalcon Incubator](https://github.com/phalcon/incubator)

<a name='nested-configuration'></a>

## Vnořené konfigurace

Also to get nested configuration you can use the `Phalcon\Config::path` method. This method allows to obtain nested configurations, without caring about the fact that some parts of the path are absent. Let's look at an example:

```php
<?php

use Phalcon\Config;

$config = new Config(
   [
        'phalcon' => [
            'baseuri' => '/phalcon/'
        ],
        'models' => [
            'metadata' => 'memory'
        ],
        'database' => [
            'adapter'  => 'mysql',
            'host'     => 'localhost',
            'username' => 'user',
            'password' => 'passwd',
            'name'     => 'demo'
        ],
        'test' => [
            'parent' => [
                'property' => 1,
                'property2' => 'yeah'
            ],
        ],
   ]
);

// Using dot as delimiter
$config->path('test.parent.property2');    // yeah
$config->path('database.host', null, '.'); // localhost

$config->path('test.parent'); // Phalcon\Config

// Using slash as delimiter
$config->path('test/parent/property3', 'no', '/'); // no

Config::setPathDelimiter('/');
$config->path('test/parent/property2'); // yeah
```

<a name='injecting-into-di'></a>

## Konfigurace jako služba

You can inject your configuration to the controllers by adding it as a service. To be able to do that, add following code inside your dependency injector script.

```php
<?php

use Phalcon\Di\FactoryDefault;
use Phalcon\Config;

// Create a DI
$di = new FactoryDefault();

$di->set(
    'config',
    function () {
        $configData = require 'config/config.php';

        return new Config($configData);
    }
);
```

Now in your controller you can access your configuration by using dependency injection feature using name `config` like following code:

```php
<?php

use Phalcon\Mvc\Controller;

class MyController extends Controller
{
    private function getDatabaseName()
    {
        return $this->config->database->dbname;
    }
}
```