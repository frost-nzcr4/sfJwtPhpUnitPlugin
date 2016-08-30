# Installation
To install the plugin into a Symfony project:

## Two ways of downloading sources and handling autoloading

### Manual

1. Install the plugin files into `plugins/`.
2. Install PHPUnit 3.6.12 if necessary. Make sure it is accessible from PHP's
    `include_path`.

### Composer

1. Add to your composer.json the following:

    ```json
    "require-dev": {
        "JWT-OSS/sfJwtPhpUnitPlugin": "1.0.12.1"
    },
    "repositories": [
        {
            "type": "package",
            "package": {
                "name": "JWT-OSS/sfJwtPhpUnitPlugin",
                "version": "1.0.12.1",
                "type": "symfony1-plugin",
                "require": {
                    "composer/installers": "^1.0.0",
                    "sebastianbergmann/phpunit": "3.6.12.1",
                    "zf1/zend-http": "1.12.*"
                },
                "source": {
                    "url": "https://github.com/frost-nzcr4/sfJwtPhpUnitPlugin.git",
                    "type": "git",
                    "reference": "1.0.12.1-dev"
                }
            }
        },
        {
            "type": "package",
            "package": {
                "name": "sebastianbergmann/phpunit",
                "version": "3.6.12.1",
                "type": "library",
                "require": {
                    "ext-dom": "*",
                    "ext-pcre": "*",
                    "ext-reflection": "*",
                    "ext-spl": "*",
                    "phpunit/php-file-iterator": "1.3.2",
                    "phpunit/php-text-template": "1.1.2",
                    "phpunit/php-timer": "1.0.3",
                    "sebastianbergmann/php-code-coverage": "1.1.4.1",
                    "sebastianbergmann/phpunit-mock-objects": "1.1.1.1"
                },
                "source": {
                    "url": "https://github.com/sebastianbergmann/phpunit.git",
                    "type": "git",
                    "reference": "a96b0d5814d6dae7a950a9e6b168b2909da3c3f8"
                },
                "autoload": {
                    "files": ["PHPUnit/Autoload.php"]
                },
                "include-path": [
                    ""
                ]
            }
        },
        {
            "type": "package",
            "package": {
                "name": "sebastianbergmann/php-code-coverage",
                "version": "1.1.4.1",
                "type": "library",
                "require": {
                    "phpunit/php-file-iterator": "1.3.2",
                    "phpunit/php-text-template": "1.1.2",
                    "phpunit/php-token-stream": "1.1.4"
                },
                "source": {
                    "url": "https://github.com/sebastianbergmann/php-code-coverage.git",
                    "type": "git",
                    "reference": "9fdb8274727c763b1674a8901d83c34ac7b96b6f"
                },
                "autoload": {
                    "files": [
                        "PHP/CodeCoverage/Autoload.php"
                    ]
                },
                "include-path": [
                    ""
                ]
            }
        },
        {
            "type": "package",
            "package": {
                "name": "sebastianbergmann/phpunit-mock-objects",
                "version": "1.1.1.1",
                "type": "library",
                "require": {
                    "ext-reflection": "*",
                    "ext-spl": "*",
                    "phpunit/php-text-template": "1.1.2"
                },
                "source": {
                    "url": "https://github.com/sebastianbergmann/phpunit-mock-objects.git",
                    "type": "git",
                    "reference": "d7c478dd488e8e2e3d6a3332700458da49e6f1f3"
                },
                "autoload": {
                    "files": [
                        "PHPUnit/Framework/MockObject/Autoload.php"
                    ]
                },
                "include-path": [
                    ""
                ]
            }
        }
    ],
    "extra": {
        "installer-paths": {
            "/path/to/symfony/plugins/{$name}": [
                "type:symfony1-plugin"
            ]
        }
    }
    ```

2. At the root of your symfony install, run:

  ```ShellSession
composer update
composer install -o
  ```

3. Make sure Symfony is including Composer autoloder (search Internet to get
  various ways to achieve that), or find in your
  `config/ProjectConfiguration.class.php` this string:

  ```php
sfCoreAutoload::register();
  ```

  and change to this snippet: 

  ```php
sfCoreAutoload::register();  // Symfony 1.4 autoloader.
require_once '/path/to/vendor/autoload.php';  // Composer autoloader.
  ```

## Remaining common installation steps

3. Add a `test` entry to `config/databases.yml` or disable `use_database` in
  `apps/*/config/settings.yml`.
4. Remove the `error_reporting` and `no_script_name` entries for the `test`
    environment in `settings.yml` for each application in your project.
5. Add the following code to `ProjectConfiguration::setup()` in
  `config/ProjectConfiguration.class.php`:

  ```php
if ( PHP_SAPI == 'cli' ) {
    $this->enablePlugins('sfJwtPhpUnitPlugin');
}
  ```

  *Note: Because this plugin only provides Symfony tasks and should have no effect
    upon the normal operation of your project, it only needs to be loaded when in
    CLI mode.*

Now you're ready to start writing tests!

# Performance Tips

For best results, consider adding the following additional configurations.

*Note: these changes could affect the execution of your application logic
  and so are not turned on by default. Use caution!*

## Doctrine

### Turn off profiling for test environment

By default, `sfDoctrineDatabase` creates a `Doctrine_Profiler` instance
  to measure the performance of queries.  It is particularly useful in dev mode
  so that you can review queries in Symfony's Web Debug Toolbar, but in test
  mode, it does little more than take up valuable system resources and memory.

To disable the profiler, add the following to your `databases.yml` file:

```yaml
# sf_config_dir/databases.yml
test:
  <connection_name>:
    class:  sfDoctrineDatabase
    param:
      ...
      profiler: false
```

### Automatically free() query objects

When you are done with a `Doctrine_Query` object, it is a good idea to `unset()`
  it to reclaim system resources.

This is especially critical during unit tests, where unreleased objects can hang
  around in memory long after the test that created them finishes.  This can
  cause memory leaks that only manifest during testing - very annoying!

Ultimately the solution to this problem is to practice good memory management,
  but if you are frequently encountering out-of-memory errors when running
  tests, try adding the following code to your `ProjectConfiguration` class:

```php
    # sf_config_dir/ProjectConfiguration.class.php

    public function configureDoctrine( Doctrine_Manager $manager )
    {
      /* Automatically free() query objects to keep memory usage low. */
      $manager->setAttribute(Doctrine_Core::ATTR_AUTO_FREE_QUERY_OBJECTS, true);
    }
```

### General strategies

See [Improving Performance][1] from the Doctrine documentation for more information.

[1]: http://www.doctrine-project.org/projects/orm/1.2/docs/manual/improving-performance/en
