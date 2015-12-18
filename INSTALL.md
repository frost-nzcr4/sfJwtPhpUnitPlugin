# Installation
To install the plugin into a Symfony project:

## Two ways of downloading sources and handling autoloading

### Manual
1. Install the plugin files into `plugins/`.
2. Install PHPUnit 3.6 if necessary.  Make sure it is accessible from PHP's
    `include_path`.

### Composer
1. Add to composer.json the following:

    ```
    "require-dev": {
        "JWT-OSS/sfJwtPhpUnitPlugin": "1.0.12",
        "phpunit/phpunit": "4.*.*",
        "zf1/zend-http": "1.12.*"
    },
    "repositories": [
        {
            "type": "package",
            "package": {
                "name": "JWT-OSS/sfJwtPhpUnitPlugin",
                "version": "1.0.12",
                "type": "symfony1-plugin",
                "require": {
                    "composer/installers": "~1.0"
                },
                "source": {
                    "url": "https://github.com/JWT-OSS/sfJwtPhpUnitPlugin.git",
                    "type": "git",
                    "reference": "tags/1.0.12"
                },
                "extra": {
                    "installer-paths": {
                        "plugins/{$name}": ["type:symfony1-plugin"]
                    }
                }
            }
        }
    ]
    ```

2. At the root of your symfony install, run:

    ```
    composer update
    composer install -o
    ```

3. make sure Symfony is including Composer autoloder (search Internet to get various ways to achieve that)

## Remaining common installation steps
3. Add a `test` entry to `config/databases.yml` or disable `use_database` in
  `apps/*/config/settings.yml`.
4. Remove the `error_reporting` and `no_script_name` entries for the `test`
    environment in `settings.yml` for each application in your project.
5. Add the following code to `ProjectConfiguration::setup()` in
  `config/ProjectConfiguration.class.php`:

        if( PHP_SAPI == 'cli' )
        {
          $this->enablePlugins('sfJwtPhpUnitPlugin');
        }

Note:  Because this plugin only provides Symfony tasks and should have no effect
  upon the normal operation of your project, it only needs to be loaded when in
  CLI mode.

Now you're ready to start writing tests!

# Performance Tips
For best results, consider adding the following additional configurations.

*Note: these changes could affect the execution of your application logic and so
  are not turned on by default.  Use caution!*

## Doctrine
### Turn off profiling for test environment.
By default, `sfDoctrineDatabase` creates a `Doctrine_Profiler` instance
  to measure the performance of queries.  It is particularly useful in dev mode
  so that you can review queries in Symfony's Web Debug Toolbar, but in test
  mode, it does little more than take up valuable system resources and memory.

To disable the profiler, add the following to your `databases.yml` file:

    # sf_config_dir/databases.yml
    test:
      <connection_name>:
        class:  sfDoctrineDatabase
        param:
          ...
          profiler: false

### Automatically free() query objects.
When you are done with a `Doctrine_Query` object, it is a good idea to `unset()`
  it to reclaim system resources.

This is especially critical during unit tests, where unreleased objects can hang
  around in memory long after the test that created them finishes.  This can
  cause memory leaks that only manifest during testing - very annoying!

Ultimately the solution to this problem is to practice good memory management,
  but if you are frequently encountering out-of-memory errors when running
  tests, try adding the following code to your `ProjectConfiguration` class:

    # sf_config_dir/ProjectConfiguration.class.php

    public function configureDoctrine( Doctrine_Manager $manager )
    {
      /* Automatically free() query objects to keep memory usage low. */
      $manager->setAttribute(Doctrine_Core::ATTR_AUTO_FREE_QUERY_OBJECTS, true);
    }

### General Strategies
See [Improving Performance][1] from the Doctrine documentation for more information.

[1]: http://www.doctrine-project.org/projects/orm/1.2/docs/manual/improving-performance/en
