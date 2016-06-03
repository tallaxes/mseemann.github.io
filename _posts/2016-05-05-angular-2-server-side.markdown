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

A main feature of [Angular 2][angular2]{:target="_blank"} is the separation of DOM rendering from the rest of the framework. This makes it possible to run Angular 2 in more environments then only the browser. 

More environments? Think about the worker in the browser where you can run javascript code in a separate thread but where you don't have access to the DOM. Or think about the new ServiceWorker to create progressive apps. Another use case is the prerendering of the initial view of your app on the server. This will not only speed up the start time of your app rather makes it also possible to provide a representation of your app for search engines.

In this post I'll provide an example how to use angulars dependency injection framework on the server side.

<!-- more -->

To get started with angular 2 on the server side you need to import some 3rd party libs. In my case I am using TypeScript, ES6 and ES7 features that are not already present in node.js. At least the following imports are necessary for angular 2.0:

```javascript

import 'es6-shim';
import 'reflect-metadata';
import 'rxjs';

```

If you have done this you need to import some classes from the angular 2 core:

```javascript
import { ReflectiveInjector, provide, Type, Provider } from 'angular2/core';
```

Now we can setup all providers we want to use. Please have a look at [this excellent blog post][angular2-di]{:target="_blank"} for a detailed description how you can configure your providers.

```javascript

let restServices: Array<Type> = [
        GitHubLoginService,
        UserService,
        LandingPage,
        ErrorPages
      ];

let providers: Array<Type | Provider | any[]> = [];

providers.push(SecurityContextFactory);
providers.push(restServices);
providers.push(provide('redisClient', { useValue: redisClient }));

var injector = ReflectiveInjector.resolveAndCreate(providers);
	  
```
This examples uses an array named `restServices`. Here is an example of such an service. As you can see it is a simple ES6 class which requires an instance of an object that is available by the name 'redisClient'. The used decorators are provided by [js-restful-express][js-restful-express]{:target="_blank"}. In this example they are responsible to create the index page for an express app that is available under `/`.

```javascript
import { Path, GET, Context, ContextTypes, RenderWith } from 'js-restful-express';
import { Inject, Injectable } from 'angular2/core';
import * as redis from 'redis';
import { Promise} from 'es6-shim';
import * as express from 'express';

@Injectable()
@Path('/')
export class LandingPage {


  constructor(@Inject('redisClient') private redisClient:redis.RedisClient){
  }

  @GET()
  @RenderWith('index')
  index(@Context(ContextTypes.HttpRequest) req:express.Request){

    this.redisClient.incr('pageViews');

    return new Promise ((resolve, reject) => {

      this.redisClient.get('pageViews', (err:Error, pageViews)=> {

        if(err){
          reject(err);
        } else {
          resolve({
            pageViews: Number(pageViews),
          });
        }

      });
    });

  }
}
```

An instance of this class needs to be registered. But we do not instantiate this class by our self. The angular injector is responsible to create an instance of the services and hook up the required dependencies.

```javascript

var app = express();

... // configure the express app

restServices.forEach( (service)=>{
  ExpressServiceRegistry.registerService(app, injector.get(service));
})

```

### Conclusion

Angular 2 provides a dependency injection framework that is usable on the server side. It is easy to setup and use. You don't need to manage your dependencies manually, you can program against interfaces, you can provide pre instantiated objects and you can use mock implementations for your unit test as easy as possible.


[angular2]: https://angular.io
[angular2-di]:http://blog.thoughtram.io/angular/2015/05/18/dependency-injection-in-angular-2.html
[js-restful-express]:https://github.com/mseemann/js-restful-express