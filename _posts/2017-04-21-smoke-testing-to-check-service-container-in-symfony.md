---
layout: post
title:  "Smoke testing to check service container in symfony"
date:   2017-04-19 18:18:36 +0200
categories: smoke-tests
---

# Introduction

Some times happen you only are working with unit test or not provide enough functional or
integration test cases to cover if all your services are OK in symfony container of your
application.

It was really useful for my team use a smoke test to check all our services loaded in the container,
here you have an example:


{% highlight php startinline linenos=table %}
# my-project/tests/SmokeTesting/CheckServicesContainerTest.php

<?php

namespace Tests\SmokeTesting;

use Symfony\Bundle\FrameworkBundle\Test\KernelTestCase;
use Symfony\Component\DependencyInjection\Exception\ServiceNotFoundException;
use Symfony\Component\DependencyInjection\Exception\ServiceCircularReferenceException;
use Exception;

class CheckServicesContainerTest extends KernelTestCase
{
    /**
     * @inheritDoc
     */
    protected function setUp()
    {
        static::$kernel = static::createKernel();
        static::$kernel->boot();
    }

    /**
     * @return \Symfony\Component\DependencyInjection\ContainerInterface
     */
    protected function getContainer()
    {
        return static::$kernel->getContainer();
    }

    /**
     * We are checking all custom services (Service Name: "prefixService_.....")
     * in the container.
     *
     * @test
     * @group smoke_testing
     */
    public function check()
    {
        $environment = $this->getContainer()->get('kernel')->getEnvironment();
        $classNameDebugContainer = sprintf('app%sDebugProjectContainer', ucwords($environment));
        $reflect = new \ReflectionClass($classNameDebugContainer);
        $reflectionPropertyMethodMap = $reflect->getProperty('methodMap');
        $reflectionPropertyMethodMap->setAccessible(true);
        $services = array_keys($reflectionPropertyMethodMap->getValue($this->getContainer()));

        foreach ($services as $key => $serviceName) {
            if (0 !== strpos($serviceName, 'prefixService')) {
                continue;
            }
            try {
                $this->getContainer()->get($serviceName);
            } catch (ServiceNotFoundException $exception) {
                $this->fail(sprintf('Service Not Found: %s', $exception->getMessage()));
            } catch (ServiceCircularReferenceException $exception) {
                $this->fail(sprintf('Service Circular Reference: %s', $exception->getMessage()));
            } catch (Exception $exception) {
                $this->fail(sprintf('Service Error: %s', $exception->getMessage()));
            }
        }

        $this->assertTrue(true);
    }
}
{% endhighlight %}

This test try to get all our custom services. If for some reason the container can not get the service maybe is because you have some
wrong namespace in the definition, bad dependencies, circular dependencies or something like that.

See the example in github, [smoke-testing](https://github.com/bernardosecades/tips-and-tricks/blob/master/smoke-testing/CheckServicesContainerTest.php).
