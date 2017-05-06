---
layout: post
title:  "RabbitMQ Setup Fabric"
date:   2017-05-06 13:51:36 +0200
categories: composer
---

# Introduction

Sometimes we need setup exchanges, queues and binding in RabbitMQ to environment dev/test/stage or production.

If you are working with php and symfony, probably you will use a bundle like [RabbitMqBundle](https://github.com/php-amqplib/RabbitMqBundle), this
bundle provide you commands to setup exchanges and queues reading your settings from your consumers and producers.

The problem is when you are working with services in differents languages, it will be missing some queues or exchanges are not defined in your symfony application, probably your system is using a producer from symfony but
the consumer is implemented in other technology like scala, node.js, ...

So if you want use a command to setup all queues and exchanges for your platform maybe the next command can be useful for you.

# Usage

Full your settings in the next file (exhanges, queues and binding):

```yaml
# .rabbit-mq-setup-fabric.yml
exchanges:
  - name: my-exchange
    type: 'topic'
    passive: false
    durable:  false
    auto_delete: false

queues:
  - name: my-queue-1
    exchange: my-exchange
    routing: my-routing-1
    passive: false
    durable: false
    exclusive: false
    auto_delete: false

  - name: my-queue-2
    exchange: my-exchange
    routing: my-routing-2
    passive: false
    durable: false
    exclusive: false
    auto_delete: false

  - name: my-queue-3
    exchange: my-exchange
    routing: my-routing-3
    passive: false
    durable: false
    exclusive: false
    auto_delete: false
```

So now you can execute the next command:

{% highlight php startinline linenos=table %}
bin/rabbit-mq-setup-fabric
{% endhighlight %}

After execute the command, in your rabbit server you will have all exhanges and queues ready to be used
by your services.

# Implementation

{% highlight php startinline linenos=table %}

#!/usr/bin/env php
<?php

/** @var Composer\Autoload\ClassLoader $loader */
$loader = require __DIR__.'/../vendor/autoload.php';

use Symfony\Component\Yaml\Yaml;
use PhpAmqpLib\Connection\AMQPStreamConnection;

/*
 * By default: ennvironment 'dev'
 *
 * Examples:
 *
 * ./bin/rabbit-mq-setup-fabric
 * ./bin/rabbit-mq-setup-fabric -e=dev
 * ./bin/rabbit-mq-setup-fabric -e=prod
 */
$options = getopt("e::", []);
$environment = !isset($options['e']) ? 'dev': strtolower($options['e']);

if ('dev' === $environment) {
    $dotenv = new Symfony\Component\Dotenv\Dotenv();
    $dotenv->load(__DIR__.'/../.env.dist');
}

$rabbitSetUp = Yaml::parse(file_get_contents(__DIR__.'/../.rabbit-mq-setup-fabric.yml'));

$connection = new AMQPStreamConnection(
        getenv('RABBITMQ_HOST'),
        getenv('RABBITMQ_PORT'),
        getenv('RABBITMQ_USER'),
        getenv('RABBITMQ_PASSWORD'),
        getenv('RABBITMQ_VHOST')
);
$channel = $connection->channel();


if (!isset($rabbitSetUp['exchanges']) || !isset($rabbitSetUp['queues'])) {
    throw new \Exception('Undefined exchanges or queues');
}

try {

    foreach ($rabbitSetUp['exchanges'] as $exchange) {
        $channel->exchange_declare(
            $exchange['name'],
            $exchange['type'],
            $exchange['passive'],
            $exchange['durable'],
            $exchange['auto_delete']
        );
    }

    foreach ($rabbitSetUp['queues'] as $queue) {
        $channel->queue_declare(
            $queue['name'],
            $queue['passive'],
            $queue['durable'],
            $queue['exclusive'],
            $queue['auto_delete']
        );

        $channel->queue_bind($queue['name'], $queue['exchange'], $queue['routing']);
    }

    echo 'All OK' . PHP_EOL;
    return 0;

} catch (\Throwable $exception) {
    echo sprintf('RabbitMQ error: %s%s', $exception->getMessage(), PHP_EOL);
    return 1;
}
{% endhighlight %}

# Dependencies

```json
{
  "name": "bernardosecades/tips-and-tricks-rabbit-mq-setup-fabric",
  "description": "RabbitMQ Setup Fabric",
  "type": "project",
  "license": "MIT",
  "authors": [
    {
      "name": "bernardosecades",
      "email": "bernardosecades@gmail.com"
    }
  ],
  "require": {
    "php-amqplib/php-amqplib": "2.6.3",
    "symfony/yaml": "3.2.8"
  },
  "require-dev": {
    "symfony/dotenv": "dev-master"
  }
}
```

See the example in github, [rabbit-mq-setup-fabric](https://github.com/bernardosecades/tips-and-tricks/tree/master/rabbit-mq-setup-fabric).