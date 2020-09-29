The first step is to make your code deprication free;

Remove the following code in `web/app.php` and `web/app_dev.php` files

```
if (PHP_VERSION_ID < 70000) {
    $kernel->loadClassCache();
}
```

If you encounter a message like this, don't panic :

`Not quoting the scalar %cache_type% starting with the "%" indicator character is deprecated since Symfony 3.1.`

All you have to do is the put quotes around your parameters values (before % sign).

After that, you can migrate to Symfony 4.0, you might get some compatibility issues with other dependencies, in that case you will have to put the compatible version with Symfony 4.0

initial composer.json

```
{
    "name": "symfony/framework-standard-edition",
    "license": "MIT",
    "type": "project",
    "description": "The \"Symfony Standard Edition\" distribution",
    "autoload": {
        "psr-4": {
            "AppBundle\\": "src/AppBundle"
        },
        "classmap": [ "app/AppKernel.php", "app/AppCache.php" ]
    },
    "autoload-dev": {
        "psr-4": { "Tests\\": "tests/" },
        "files": [ "vendor/symfony/symfony/src/Symfony/Component/VarDumper/Resources/functions/dump.php" ]
    },
    "require": {
        "php": ">=7.1",
        "doctrine/doctrine-bundle": "1.11.2",
        "doctrine/doctrine-migrations-bundle": "1.3.2",
        "doctrine/orm": "2.6.3",
        "friendsofsymfony/rest-bundle": "2.5.0",
        "gedmo/doctrine-extensions": "2.4.37",
        "google/recaptcha": "1.2.3",
        "incenteev/composer-parameter-handler": "2.1.3",
        "jms/serializer-bundle": "3.4.1",
        "nelmio/api-doc-bundle": "3.4.0",
        "nelmio/cors-bundle": "1.5.6",
        "sensio/distribution-bundle": "5.0.25",
        "sensio/framework-extra-bundle": "5.4.1",
        "stof/doctrine-extensions-bundle": "1.3.0",
        "symfony/monolog-bundle": "3.4.0",
        "symfony/polyfill-apcu": "1.12.0",
        "symfony/swiftmailer-bundle": "2.6.7",
        "symfony/symfony": "3.4.30",
        "twig/twig": "^1.0||^2.0"
    },
    "require-dev": {
        "doctrine/doctrine-fixtures-bundle": "3.1.1",
        "sensio/generator-bundle": "^3.0",
        "symfony/phpunit-bridge": "^3.0"
    },
    "scripts": {
        "symfony-scripts": [
            "Sensio\\Bundle\\DistributionBundle\\Composer\\ScriptHandler::buildBootstrap",
            "Sensio\\Bundle\\DistributionBundle\\Composer\\ScriptHandler::installAssets",
            "Sensio\\Bundle\\DistributionBundle\\Composer\\ScriptHandler::installRequirementsFile",
            "Sensio\\Bundle\\DistributionBundle\\Composer\\ScriptHandler::prepareDeploymentTarget"
        ],
        "post-install-cmd": [
            "@symfony-scripts"
        ],
        "post-update-cmd": [
            "@symfony-scripts"
        ]
    },
    "config": {
        "platform": {
            "php": "7.2"
        },
        "sort-packages": true
    },
    "extra": {
        "symfony-app-dir": "app",
        "symfony-bin-dir": "bin",
        "symfony-var-dir": "var",
        "symfony-web-dir": "web",
        "symfony-tests-dir": "tests",
        "symfony-assets-install": "relative",
        "branch-alias": {
            "dev-master": "3.4-dev"
        }
    }
}

```

Modified composer.json (dependencies versions)

```
"require": {
        "php": ">=7.1",
        "doctrine/doctrine-bundle": "^1.6",
        "doctrine/doctrine-migrations-bundle": "1.3.2",
        "doctrine/orm": "2.6.3",
        "friendsofsymfony/rest-bundle": "2.5.0",
        "gedmo/doctrine-extensions": "2.4.37",
        "google/recaptcha": "1.2.3",
        "jms/serializer-bundle": "3.4.1",
        "nelmio/api-doc-bundle": "3.4.0",
        "nelmio/cors-bundle": "1.5.6",
        "sensio/framework-extra-bundle": "5.2.4",
        "stof/doctrine-extensions-bundle": "1.3.0",
        "symfony/monolog-bundle": "3.4.0",
        "symfony/polyfill-apcu": "1.12.0",
        "symfony/swiftmailer-bundle": "3.2.4",
        "symfony/symfony": "4.0.15",
        "twig/twig": "^1.0||^2.0"
    },
    "require-dev": {
        "doctrine/doctrine-fixtures-bundle": "^2.3",
        "symfony/phpunit-bridge": "^4.0"
    }

```

As you might notice, some packages have been removed such as :

- incenteev/composer-parameter-handler
- sensio/distribution-bundle
- sensio/generator-bundle

Some has been updgraded/downgraded due to compatibility issues:

- doctrine/doctrine-bundle (upgraded)
- symfony/swiftmailer-bundle (upgraded)
- doctrine/doctrine-fixtures-bundle (downgraded)
- symfony/phpunit-bridge (upgraded)
- sensio/framework-extra-bundle (upgraded)

You should remove the declarations in AppKernel for the removed bundles:

```
$bundles[] = new Sensio\Bundle\DistributionBundle\SensioDistributionBundle();
$bundles[] = new Sensio\Bundle\GeneratorBundle\SensioGeneratorBundle();
```

Now you are all set and you can install Symfony flex :)

Let's get started :

```
composer require symfony/flex
```

After that, you will have to add / update some parts of your composer.json

- Update the require section, remove the symfony/symfony declaration and replace it by

```
"symfony/console": "4.4.*",
"symfony/dotenv": "4.4.*",
"symfony/framework-bundle": "4.4.*",
"symfony/yaml": "4.4.*",
```

- Update the config, replace, scripts, conflicts and extra sections with the following :

```
"config": {
        "preferred-install": {
            "*": "dist"
        },
        "sort-packages": true
    },
    "replace": {
        "paragonie/random_compat": "2.*",
        "symfony/polyfill-ctype": "*",
        "symfony/polyfill-iconv": "*",
        "symfony/polyfill-php71": "*",
        "symfony/polyfill-php70": "*",
        "symfony/polyfill-php56": "*"
    },
    "scripts": {
        "auto-scripts": [
        ],
        "post-install-cmd": [
            "@auto-scripts"
        ],
        "post-update-cmd": [
            "@auto-scripts"
        ]
    },
    "conflict": {
        "symfony/symfony": "*"
    },
    "extra": {
        "symfony": {
            "allow-contrib": true,
            "require": "4.4.*",
        }
    }
```

Don't forget to modify the autoload section since we still have our code in the AppBundle :

```
"autoload": {
        "psr-4": {
            "App\\": "src/",
            "AppBundle\\": "src/AppBundle"
        },
        "classmap": [ "app/AppKernel.php", "app/AppCache.php" ]
    },
    "autoload-dev": {
        "psr-4": {
            "Tests\\AppBundle\\": "tests/AppBundle",
            "App\\Tests\\": "tests/"
        }
    },
```

You have a composer.json that look like this :

```
{
    "name": "symfony/framework-standard-edition",
    "license": "MIT",
    "type": "project",
    "description": "The \"Symfony Standard Edition\" distribution",
    "autoload": {
        "psr-4": {
            "App\\": "src/",
            "AppBundle\\": "src/AppBundle"
        },
        "classmap": [ "app/AppKernel.php", "app/AppCache.php" ]
    },
    "autoload-dev": {
        "psr-4": {
            "Tests\\AppBundle\\": "tests/AppBundle",
            "App\\Tests\\": "tests/"
        }
    },
    "require": {
        "php": ">=7.1",
        "doctrine/doctrine-bundle": "^1.6",
        "doctrine/doctrine-migrations-bundle": "1.3.2",
        "doctrine/orm": "2.6.3",
        "friendsofsymfony/rest-bundle": "2.5.0",
        "gedmo/doctrine-extensions": "2.4.37",
        "google/recaptcha": "1.2.3",
        "jms/serializer-bundle": "3.4.1",
        "nelmio/api-doc-bundle": "3.4.0",
        "nelmio/cors-bundle": "1.5.6",
        "sensio/framework-extra-bundle": "5.2.4",
        "stof/doctrine-extensions-bundle": "1.3.0",
        "symfony/console": "4.4.*",
        "symfony/dotenv": "4.4.*",
        "symfony/flex": "^1.9",
        "symfony/framework-bundle": "4.4.*",
        "symfony/monolog-bundle": "3.4.0",
        "symfony/polyfill-apcu": "1.12.0",
        "symfony/swiftmailer-bundle": "3.2.4",
        "symfony/yaml": "4.4.*",
        "twig/twig": "^1.0||^2.0"
    },
    "require-dev": {
        "doctrine/doctrine-fixtures-bundle": "^2.3",
        "symfony/phpunit-bridge": "^4.0"
    },
    "config": {
        "preferred-install": {
            "*": "dist"
        },
        "sort-packages": true
    },
    "replace": {
        "paragonie/random_compat": "2.*",
        "symfony/polyfill-ctype": "*",
        "symfony/polyfill-iconv": "*",
        "symfony/polyfill-php71": "*",
        "symfony/polyfill-php70": "*",
        "symfony/polyfill-php56": "*"
    },
    "scripts": {
        "auto-scripts": [
        ],
        "post-install-cmd": [
            "@auto-scripts"
        ],
        "post-update-cmd": [
            "@auto-scripts"
        ]
    },
    "conflict": {
        "symfony/symfony": "*"
    },
    "extra": {
        "symfony": {
            "allow-contrib": true,
            "require": "4.4.*"
        }
    }
}
```

Now, delete you vendor directory and install the dependencies again :

```
rm -rf vendor/*
composer update
```

We did this because Symfony Flex uses recipes for auto configurating packages/bundles, you will see a numerous files generated for each bundle you have in your projects plus the `Entity, Controller, Repository` folders and `Kernel.php` file in the `src` folder

Next step is to modify your `bin/console` file in order to use the new Kernel

Copy and paste the following code in your `bin/console` file.

```
#!/usr/bin/env php
<?php

use App\Kernel;
use Symfony\Bundle\FrameworkBundle\Console\Application;
use Symfony\Component\Console\Input\ArgvInput;
use Symfony\Component\Debug\Debug;

if (!in_array(PHP_SAPI, ['cli', 'phpdbg', 'embed'], true)) {
    echo 'Warning: The console should be invoked via the CLI version of PHP, not the '.PHP_SAPI.' SAPI'.PHP_EOL;
}

set_time_limit(0);

require dirname(__DIR__).'/vendor/autoload.php';

if (!class_exists(Application::class)) {
    throw new LogicException('You need to add "symfony/framework-bundle" as a Composer dependency.');
}

$input = new ArgvInput();
if (null !== $env = $input->getParameterOption(['--env', '-e'], null, true)) {
    putenv('APP_ENV='.$_SERVER['APP_ENV'] = $_ENV['APP_ENV'] = $env);
}

if ($input->hasParameterOption('--no-debug', true)) {
    putenv('APP_DEBUG='.$_SERVER['APP_DEBUG'] = $_ENV['APP_DEBUG'] = '0');
}

require dirname(__DIR__).'/config/bootstrap.php';

if ($_SERVER['APP_DEBUG']) {
    umask(0000);

    if (class_exists(Debug::class)) {
        Debug::enable();
    }
}

$kernel = new Kernel($_SERVER['APP_ENV'], (bool) $_SERVER['APP_DEBUG']);
$application = new Application($kernel);
$application->run($input);
```

You might face some issues with Validation and twig after testing a controller since they are not automatically installed with symfony anymore, all you have to do is to install the missing components by adding them directly in the `composer.json` and running `composer update`

```
"symfony/twig-bundle": "4.4.*",
"symfony/validator": "4.4.*",
"twig/extra-bundle": "^2.12|^3.0",
```

Or by installing them via the CLI:

```
composer require validator symfony/twig-pack
```

After all this, you will have to migrate your configurations from `app/config/config.yml` to the dedicated files in the new `config` directory.
You will find a yaml file for each bundle.

For example the doctrine section in `app/config/config.yml` should be moved to `config/packages/doctrine.yaml`

If you already have some environment specific configurations (for staging maybe), you should move the doctrine configuration from `app/config/staging/config.yml` or `app/config/config_staging.yml` to `config/packages/staging/doctrine.yaml`

For the parameters, you can do the same thing, your default values should be placed in the parameters sections of the `config/services.yaml` file.

If you want to override those values for specific environments (staging maybe), you can create a `services_staging.yaml`

Now, if you attempt to call an endpoint, you might get an error that says that `The autoloader exepects .... to be defined in AppBundle.php, but it was not found` or something like . What causes this error is the fact that we a Flex app with some autowiring and auto discovery and it tries to autoregister our code which live inside the `AppBundle` directory.

Our code should be in the AppBundle, but for now we will make the app works and then we will move to files to the right place.

First things first, let's exclude everything inside the `AppBundle` to remove the error.

Modify the below section in `config/services.yaml` file.

```
App\:
    resource: '../src/*'
    exclude: '../src/{DependencyInjection,Entity,Migrations,Tests,AppBundle,Kernel.php}'
```

Add these declarations in the `config/services.yaml` file the code that lives inside the AppBundle can be loaded.

```
    AppBundle\:
        resource: '../src/AppBundle/*'
        # you can exclude directories or files
        # but if a service is unused, it's removed anyway
        exclude: '../src/AppBundle/{Entity,Tests,Utils}'

    # controllers are imported separately to make sure they're public
    # and have a tag that allows actions to type-hint services
    AppBundle\Controller\:
        resource: '../src/AppBundle/Controller'
        public: true
        tags: ['controller.service_arguments']
```

If you have any other explicit service declarations in `app/config/service.yml`, add them to in this file.
