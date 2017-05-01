---
layout: post
title:  "Functional/Integration tests for standalone symfony bundles"
date:   2017-05-01 11:30:36 +0200
categories: symfony
---

# Introduction

When your are working in a new bundle to share with other projects sometimes
you create new service in your bundle with infrastructure dependencies (doctrine, redis, mysql, rabbitMQ, file system ...) or
include controllers.

In some cases we want to test real behaviour without mocking the dependencies (integration/functional tests) to check if our
services are working fine with real dependencies and symfony container.

So to can do functional/integration tests in standalone using the symfony container in our bundle we can add symfony kernel
in our `Tests` folder, we will add `FrameworkBundle` and own bundle to simulate integration bundle in the final symfony application.

# Usage

Create `AppKernel.php` file inside of `Tests` folder and add the next code:

{% highlight php startinline linenos=table %}
<?php

namespace Tests;

use BernardoSecades\FunctionalTestBundle\FunctionalTestBundle;
use Symfony\Bundle\FrameworkBundle\FrameworkBundle;
use Symfony\Component\Config\Loader\LoaderInterface;
use Symfony\Component\HttpKernel\Kernel;

class AppKernel extends Kernel
{
    /**
     * @return array
     */
    public function registerBundles()
    {
        $bundles = array();

        if (in_array($this->getEnvironment(), array('test'))) {
            $bundles[] = new FrameworkBundle();
            $bundles[] = new NameBundleWeAreWorkingBundle();
        }

        return $bundles;
    }

    /**
     * @param LoaderInterface $loader
     */
    public function registerContainerConfiguration(LoaderInterface $loader)
    {
        $loader->load(__DIR__.'/config_'.$this->getEnvironment().'.yml');

    }

    /**
     * {@inheritdoc}
     */
    public function getCacheDir()
    {
        return sys_get_temp_dir().'/NameBundleWeAreWorkingBundle/cache';
    }

    /**
     * {@inheritdoc}
     */
    public function getLogDir()
    {
        return sys_get_temp_dir().'/NameBundleWeAreWorkingBundle/logs';
    }
}
{% endhighlight %}

Create the file `config_test.yml` with next lines to load our services in test environment:

{% highlight yml startinline linenos=table %}

imports:
    - { resource: "@NameBundleWeAreWorkingBundle/Resources/config/services.yml" }

framework:
    secret: "Three can keep a secret, if two of them are dead."
    test: ~
{% endhighlight %}


This configuration will be loaded by `AppKernel.php` we added before. Check this piece of code from
kernel file:

{% highlight php startinline linenos=table %}
<?php

    // ...

    /**
     * @param LoaderInterface $loader
     */
    public function registerContainerConfiguration(LoaderInterface $loader)
    {
        $loader->load(__DIR__.'/config_'.$this->getEnvironment().'.yml');

    }

{% endhighlight %}

And now you can load the class `AppKernel` in your test putting like first parameter
the environment, in this case, environment `test`:

{% highlight php startinline linenos=table %}

<?php

namespace Tests\Functional;

use Tests\AppKernel;
use BernardoSecades\FunctionalTestBundle\Lib\MyService;
use Symfony\Bundle\FrameworkBundle\Test\KernelTestCase;
use Symfony\Component\DependencyInjection\ContainerInterface;

class MyTest extends KernelTestCase
{
    /** @var  ContainerInterface */
    protected $container;

    protected function setUp()
    {
        parent::setUp();

        $kernel = new AppKernel('test', true);
        $kernel->boot();
        $this->container = $kernel->getContainer();
    }

    /**
     * @test
     */
    public function checkMyService()
    {
        $myService = $this->container->get('bernardosecades.my_service');

        $this->assertInstanceOf(MyService::class, $myService);
        $this->assertTrue($myService->myAction());
    }
}

{% endhighlight %}

See the example in github, [functional-tests-standalone-bundle](https://github.com/bernardosecades/tips-and-tricks/tree/master/functional-test-standalone-bundle).
