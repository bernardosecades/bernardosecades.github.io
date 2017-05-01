---
layout: post
title:  "Runtime environment variables in symfony"
date:   2017-04-16 18:18:36 +0200
categories: symfony
---

# Introduction

Environment variables allow developers to extract sensitive credentials from their source code and to use different configuration variables based on their working environment.

When dealing with sensitive options, like database credentials, it is a good practice that you store them outside the Symfony project and make them available
through environment variable.

In symfony the file `config.yml` is used to options application level and `parameters.yml` is used to options in infrastructure level.

It is really bad practice put sensitive options like credentials directly in config.yml file. Usually you put in your repository a template
`parameters.yml.dist` with default values (usually values from dev environment) so in your deploy you will do something like this:

{% highlight bash %}
cp machine-with-credentials/parameters.prod.yml app/config/parameters.yml
# ...
cp machine-with-credentials/parameters.ci.yml app/config/parameters.yml

{% endhighlight %}

You can generate value options in parameters.yml from environment variables o directly you can avoid use parameters.yml file.


# Option 1: Use env-map in composer.json to package incenteev/parameterHandler can to create parameters.yml from environment variables



{% highlight bash startinline linenos=table %}
{
    "extra": {
        "incenteev-parameters": {
            "env-map": {
                "database.user": "DATABASE_USER",
                "database.password": "DATABASE_PASSWORD"
            }
        }
    }
}
{% endhighlight %}

So in the `scripts` section in your composer.json you can see:

{% highlight bash startinline linenos=table %}
    "scripts": {
        "post-install-cmd": [
            "Incenteev\\ParameterHandler\\ScriptHandler::buildParameters"
        ],
        "post-update-cmd": [
            "Incenteev\\ParameterHandler\\ScriptHandler::buildParameters"
        ]
    },
{% endhighlight %}

The script `buildParameters` will be called when you execute `composer install` or `composer update`,  this script
will create the file parameters.yml will from template parameters.yml.dist with values from your
environment variables.


# Option 2: Do not use parameters.yml file, only use environment variables

In file `config.yml` you will have something like this:

{% highlight bash startinline linenos=table %}
imports:
    - { resource: parameters.yml }
    - { resource: services.yml }
    # ...
{% endhighlight %}

Remove the import of parameters.yml file and create a template to environment variables with standard name `.env.dist`

*Note: Your .env file should not be committed to your application's source control, since each developer using your application could require a different environment configuration.*

So in environment dev, developers can import easily environment variables only executing the command:

{% highlight bash startinline linenos=table %}
source .env.dist
{% endhighlight %}

# Symfony < 3.2

The content file `.env.dist` will be something like this:

{% highlight bash startinline linenos=table %}
export SYMFONY__DATABASE_USER=root
export SYMFONY__DATABASE_PASSWORD=null
{% endhighlight %}

Symfony then automatically sets all $_SERVER variables prefixed with SYMFONY__ as parameters in the service container.

You can now reference these parameters wherever you need them.

{% highlight yaml startinline linenos=table %}
# app/config/config.yml
doctrine:
    dbal:
        driver:   pdo_mysql
        dbname:   symfony_project
        user:     '%database.user%'
        password: '%database.password%'
{% endhighlight %}

*Note: In symfony 3.3 deprecated the special `SYMFONY__` variables*

# Symfony >= 3.2

In symfony 3.2 you do not need put prefix `SYMFONY__`, it is not make sense because those environment variables
can be used by others applications no necessarily built with symfony.

So the name of variables will be more standard to share with other apps, example:

Content file `.env.dist`:

{% highlight bash startinline linenos=table %}
export DATABASE_USER=root
export DATABASE_PASSWORD=null
{% endhighlight %}

To load these environment variables you should execute the next command:

{% highlight bash startinline %}
source .env.dist
{% endhighlight %}


To use the environment variables in your `config.yml` file you can put something like this:

{% highlight yaml startinline linenos=table %}
# app/config/config.yml
doctrine:
    dbal:
        # ...
        password: "%env(DATABASE_PASSWORD)%"
{% endhighlight %}


# Environment "Dev"

To avoid execute in each session the command `source .env.dist` or persist environment variables in dev environment we can modify next symfony files:

- bin/console
- web/app_dev.php

**bin/console** file:

{% highlight php startinline %}
#!/usr/bin/env php
<?php

// ...
use Symfony\Component\Dotenv\Dotenv;

// ...

// New!! Load environment variables with dotenv component
if ('dev' === $env) {
    $dotenv = new Dotenv();
    $dotenv->load(__DIR__.'/../.env.dist');
}

// ...
{% endhighlight %}

**web/app_dev.php** file:

{% highlight php startinline %}
<?php

use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Debug\Debug;
use Symfony\Component\Dotenv\Dotenv;
// ...

// Note: we do not check environment because we are in app_dev.php entry.
$dotenv = new Dotenv();
$dotenv->load(__DIR__.'/../.env.dist');

// ...
{% endhighlight %}

To phpunit.xml.dist you can do something similar loading bootstrap file,
you can call bootstrap_dev.php and load in your xml phpunit.xml.dist:

{% highlight xml startinline %}
<?xml version="1.0" encoding="UTF-8"?>

<!-- https://phpunit.de/manual/current/en/appendixes.configuration.html -->
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="http://schema.phpunit.de/4.8/phpunit.xsd"
         backupGlobals="false"
         colors="true"
         bootstrap="bootstrap_dev.php"
>
{% endhighlight %}



# Conclusion

Whatever option is valid but I prefer the option 2 because you do not depend of third parts like `incenteev\parameterHandler` to
set up in right way options from infrastructure layer and looks a solution more standard.