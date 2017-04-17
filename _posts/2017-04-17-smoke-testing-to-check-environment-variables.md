---
layout: post
title:  "Smoke testing to check environment variables"
date:   2017-04-17 14:10:36 +0200
categories: smoke-tests
---

# Introduction

Smoke testing is preliminary testing to reveal simple failures severe enough to
(for example) reject a prospective software release.

So if our application are working with environment variables, a possible smoke test can be checking
if in the machine where we are trying to do deploy are ready all environment variables need by our application.

# Test to check environment variables

We can use the component [symfony/dotenv](https://packagist.org/packages/symfony/dotenv) to check our environment variables
from our template file `.env.dist` (See more in related post [runtime-environment-variables-in-symfony](/symfony/2017/04/16/runtime-environment-variables-in-symfony.html))

{% highlight php startinline linenos=table %}
# my-project/tests/SmokeTesting/CheckEnvironmentVariablesTest.php

<?php

namespace Tests\SmokeTesting;

use Symfony\Component\Dotenv\Dotenv;

/**
 * Note that existing environment variables are never overridden.
 *
 * @author bernardosecades
 * @link http://symfony.com/doc/master/components/dotenv.html
 */
class CheckEnvironmentVariablesTest extends \PHPUnit_Framework_TestCase
{
    /**
     * @test
     */
    public function check()
    {
        $path = __DIR__.'/../../.env.dist';

        $dotenv = new Dotenv();
        $vars = $dotenv->parse(file_get_contents($path), $path);
        $varNames = array_keys($vars);

        foreach ($varNames as $varName) {
            $value = getenv($varName);
            if (false === $value) {
                $this->fail(
                    sprintf(
                        'Environment variable "%s" does not exist'
                        , $varName
                    )
                );
            }
            if (empty($value)) {
                $this->fail(
                    sprintf(
                        'Environment variable "%s" is empty'
                        , $varName
                    )
                );
            }
        }

        $this->assertTrue(true);
    }
}
{% endhighlight %}

See the example in github, [smoke-testing](https://github.com/bernardosecades/tips-and-tricks/blob/master/smoke-testing/CheckEnvironmentVariablesTest.php).
