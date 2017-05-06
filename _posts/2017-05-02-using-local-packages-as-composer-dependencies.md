---
layout: post
title:  "Using local packages as composer dependencies"
date:   2017-05-02 13:51:36 +0200
categories: composer
---

# Introduction

Composer is a tool for dependency management in PHP. It allows you to declare the libraries your project
depends on and it will manage (install/update) them for you.

Sometimes we need to work in packages to share in different projects, for this goal we can use composer to
include dependencies in our projects.

If you are working in a package and you want to check how works in your project, yo can setup that dependency like local
in your composer, later when the package will be stable you can put tag and upload your repository, but while you are
working in environment dev can be useful only work in local.

We will see how setup your local dependency to work faster.

# Usage

We going to suppose we are working in package `myPackage` and we want to share with our project:

*Example composer.json in our project*
```json
{
  "name": "bernardosecades/tips-and-tricks-composer",
  "description": "Example local packages",
  "type": "project",
  "license": "MIT",
  "authors": [
    {
      "name": "bernardosecades",
      "email": "bernardosecades@gmail.com"
    }
  ],
  "autoload": {
    "psr-4": {
      "BernardoSecades\\TipsAndTricks\\Composer\\": "src/"
    }
  },
  "repositories": [
    {
      "type": "path",
      "url": "../../myPackage",
      "options": {
        "symlink": true
      }
    }
  ],
  "require": {
    "bernardosecades/myPackage": "dev-master"
  }
}
```

Instead of put the url of repository we put the repository will be in local path (`"type":"path"`) indicate
our local path.

To easily the development we can put options `"symlink": true`, so when we will modify our package `myPackage`
changes will be ready in our proyect without execute again `composer install`.

See the example in github, [local-packages-as-composer-dependencies](https://github.com/bernardosecades/tips-and-tricks/tree/master/composer).



