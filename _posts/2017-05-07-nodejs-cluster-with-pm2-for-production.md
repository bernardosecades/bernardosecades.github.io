---
layout: post
title:  "Node.js cluster with PM2 for production"
date:   2017-05-07 09:51:36 +0200
categories: node.js
---

# Introduction

PM2 or Process Manager 2, is an Open Source, production Node.js process manager
helping Developers and Devops manage Node.js applications in production environment.

# Installation

`sudo npm install pm2 -g`

# Usage

We going to see how run in cluster mode the next worker with `PM2`:

{% highlight javascript startinline linenos=table %}
// File: app.js
const uncaughtExceptionHandler = require('./lib/error/uncaughtExceptionHandler');
const worker = require('./lib/worker');

worker.run();

process.on('uncaughtException', uncaughtExceptionHandler);
{% endhighlight %}

Environment **Dev**:

We can create js file to pm2 can read the configuration, so we won´t use parameters in the command line,
only will tell to PM2 load settings from js file. See in the settings to `dev` we put `watch` to true, this
is because in development environment we want PM2 restart our cluster if we modify the source code (this value should be false in production).

As well we put in environment `dev` the environment variables to run easily our service, in `prod` environment, machines
should provide these environment variables with right values to execute in production.

{% highlight javascript startinline linenos=table %}
// File: dev_ecosystem.config.js
module.exports = {
  /**
   * Application configuration section
   * http://pm2.keymetrics.io/docs/usage/application-declaration/
   */
  apps : [
    {
      name      : "MyService-DEV",
      script    : "app.js",
      watch     : true,
      exec_mode : "cluster",
      instances : 4,
      max_memory_restart: "3G",
      env: {
        NODE_ENV: "dev",
        NODE_DEBUG_REQUESTER: true,
        RABBITMQ_HOST: "myRabbitHost",
        RABBITMQ_PORT: 5672,
        RABBITMQ_USER: "myUser",
        RABBITMQ_PASSWORD: "myPassword",
        RABBITMQ_VHOST: "myVhost",
        STATSD_HOST: "localhost",
        STATSD_PORT: 8125
      }
    }
  ]
}

{% endhighlight %}

Environment **Prod**:


{% highlight javascript startinline linenos=table %}
// File: prod_ecosystem.config.js
module.exports = {
  /**
   * Application configuration section
   * http://pm2.keymetrics.io/docs/usage/application-declaration/
   */
  apps : [
    {
      name      : "MyService-PROD",
      script    : "app.js",
      watch     : false,
      exec_mode : "cluster",
      instances : 8,
      max_memory_restart: "3G",
    }
  ]
}

{% endhighlight %}

To run our worker in cluster mode only will need execute commands:

`pm2 start pm/dev_ecosystem.config.js` (Dev Environment)

or

`pm2 start pm/dev_ecosystem.config.js` (Prod Environment)

# Screenshots with useful commands

Command `pm2 start pm/dev_ecosystem.config.js`:

![pm2 start pm/dev_ecosystem.config.js](https://image.ibb.co/dHt51Q/screenshot_start.png)

Command `pm2 kill`:

![pm2 kill](https://image.ibb.co/hR7nFk/screenshot_kill.png)

Command `pm2 monit`:

![pm2 monit](https://image.ibb.co/e6G3o5/screenshot_monit.png)

Command `pm2 reload all`:

![pm2 reload all](https://image.ibb.co/kmYio5/screenshot_reload.png)

Command `pm2 scale MyService-DEV 6`:

![pm2 scale MyService-DEV 6](https://image.ibb.co/njqk1Q/screenshot_scale_app.png)

Command `pm2 show 0`:

![pm2 show 0](https://image.ibb.co/e2dG85/screenshot_show_process_id.png)

Command `pm2 status`:

![pm2 status](https://image.ibb.co/b6U1ak/screenshot_status.png)

Command `pm2 stop all`:

![pm2 stop all](https://image.ibb.co/jDSb85/screenshot_stop.png)

Command `pm2 stop 0`:

![pm2 stop 0](https://image.ibb.co/mJfJMQ/screenshot_stop_process_id.png)

