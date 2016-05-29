---
layout: post
title:  "Using Angular 2 on the server side."
date:   2016-05-05 18:41:26 +0200
categories: server
tags: 
  - angular 2
  - node.js
  - express
  - dependency injection
---

A main feature of [Angular 2][angular2]{:target="_blank"} is the separation of dom rendering from the rest of the framework. That makes it possible to use Angular 2 code on the serverside. 

to be continued...

{% highlight javascript %}

import 'es6-shim';
import 'reflect-metadata';
import 'rxjs';

import app from './lib/app';

app.start();

{% endhighlight %}


{% highlight javascript %}

let restServices:Array<Type> = [
        GitHubLoginService,
        BookService,
        UserService,
        LandingPage,
        ErrorPages
      ];

let providers:Array<Type | Provider | any[]> = [];

providers.push(SecurityContextFactory);
providers.push(restServices);
providers.push(provide('redisClient', { useValue: redisClient }));

var injector = ReflectiveInjector.resolveAndCreate(providers);
	  
{% endhighlight %}

[angular2]: https://angular.io