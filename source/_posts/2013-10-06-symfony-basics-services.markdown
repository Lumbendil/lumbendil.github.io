---
layout: post
title: "Symfony basics: Services"
date: 2013-10-06 21:23
comments: true
categories: symfony
---
I'll start this blog talking about the framework in which I currently do most of my work, [Symfony](http://symfony.com/). It's a framework which has a really interesting component, the **Dependency Injection** component. This component is used to define services. But that brings up the question of *what is a service?*

A service is nothing else than an object which has it's creation already handled by the **Dependency Injection component**. This is to avoid having to re-type configuration everywhere, but instead keeping configuration in a different place than code usage. For instance, in most applications you need a database connection, which is always configured the same way. This database connection is a service. But looking at the dependency injection component in Symfony, one can't avoid to ask himself *should it be this complicated? Why is this approach good?*

### Advantages of the Dependency Injection Component

The dependency injection component as implemented by Symfony offers a lot of functionality over a custom solution:

 * Built to be optimal in production: The process of building a service container in this component is quite big, considering it has to parse files, remove unused dependencies and many other things, but most of the overhead is removed in production. This is thanks to code generation. The component builds a class which can build the services which can be required, and nothing else. This means a single PHP file is going to be read when having to use the container. Since it's likely that you use APC on your server, this means close to zero overhead from creating the classes manually.
 * Rich set of features: It has many features, such as configuration files, tagging services to be able to find them afterwards, or conditional extension.
 * Greatly backed up: It's backed up by both [Sensiolabs](http://sensiolabs.com) and the Symfony community, thus it's bugs are fixed promptly.
 
If after reading this you'd be interested in trying it, give it a go. You can read [it's documentation](http://symfony.com/doc/current/components/dependency_injection/index.html), go to [it's repository](https://github.com/symfony/DependencyInjection) and play arround with it a bit. After all, you can't know if something is interesting for you until you've tried it yourself.