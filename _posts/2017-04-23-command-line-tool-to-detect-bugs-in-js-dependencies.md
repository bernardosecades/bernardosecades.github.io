---
layout: post
title:  "NPM, Node.js - Command line tool to detect bugs in your js dependencies"
date:   2017-04-23 15:00:36 +0200
categories: node.js
---

# Introduction

Command-line tool to detect bugs in your dependencies versions: `sos-dep-checker`

Detect bugs in your dependencies based on [Semver](http://semver.org).

The ideal case is you update your dependencies to last version but sometimes
is hard because include breaking changes and maybe you do not want include
new features. Anyway you should update your dependencies at least to last patch version.

Note: PATCH version when you make backwards-compatible bug fixes.

I have created a package in `npm`, you can see here: [sos-dep-checker](https://www.npmjs.com/package/sos-dep-checker)


# Install

{% highlight bash startinline linenos=table %}
npm install sos-dep-checker -g
{% endhighlight %}

# Usage

Go to the root folder of your project, where your package.json is and run:

{% highlight bash startinline linenos=table %}
sos-dep-checker
{% endhighlight %}

Example Output:

![sos-dep-checker](https://image.ibb.co/jehxbQ/screen1.png)

See more details in github repository: [sos-dep-checker](https://github.com/bernardosecades/sos-dep-checker)





