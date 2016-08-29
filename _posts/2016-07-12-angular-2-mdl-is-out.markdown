---
layout: post
title:  "The first release candidate of Angular 2 Material Design Lite (angular2-mdl) is out"
date:   2016-07-12 18:30:26 +0200
categories: frontend
tags: 
  - angular 2
  - Material Design Lite
---
The first release candidate of my angular 2 component library has been released ([demo][angular2-mdl-demo]{:target="_blank"} [github][mdl2-github]{:target="_blank"}). This library provides components and directives that can be used in angular 2. The comppnents are based on material design lite (see [getmdl.io][getmdl]{:target="_blank"} for more information about this ui framework).

This post covers why i have written such a component library, how to use it and what components are offered by this npm module.

<!-- more -->

*Update (08/24/2016): angualr2-mdl is finally released and supports now angular 2 (rc5) and material design lite version 1.2.*

### Why a component library for mdl
Material design lite was written as a collection of scss files and a little bit of javascript to add dynamic behavior like ripple effects, focus and blur handling and so on. It was not written with a specific javascript framework in mind. And it was also not written to match the needs of dynamic ui generation. So for example: you will not have the possibility to add menu items to a menu after the menu was created.

Until now there is no complete library for angular 2 that provides material design components. The official implementation [Material Design components for Angular 2][material2]{:target="_blank"} is still in the alpha state.

That's why i decided to write an implementation of material design components for angular 2 based on material design lite. After one month of work the npm package is now nearly feature complete and version 1.0.0-rc1 is released. For sure: any help/contribution is appreciated!

A side effect is that you don't need to write a lot of css classes and html structures to create material design lite components.

Instead of 

```html
<div class="mdl-textfield mdl-js-textfield mdl-textfield--expandable">
    <label class="mdl-button mdl-js-button mdl-button--icon" for="sample6">
      <i class="material-icons">search</i>
    </label>
    <div class="mdl-textfield__expandable-holder">
      <input class="mdl-textfield__input" type="text" id="sample6">
    </div>
  </div>
``` 
you just write:

```html
<mdl-textfield type="text" [(ngModel)]="theText" icon="search"></mdl-textfield>

```
to create an expandle text field and get two-way-databindig for free!

### How to use angular2-mdl

To get started you need to install the npm package and integrate it in your build system:

```bash
npm install angular2-mdl --save
```

I use angular cli with system.js as my build system. If you do the same, you have to do the following steps to integrate angular2-mdl in your project:
1. Extend the angular-cli-build.js file to include angular2-mdl as a vendor package:

```javascript
return new Angular2App(defaults, {

  vendorNpmFiles: [
    ...
    'angular2-mdl/**/*'
  ]
});
```

Next you need to configure your system-config.js file:

```javascript
const map: any = {
  'angular2-mdl': 'vendor/angular2-mdl'
};

/** User packages configuration. */
const packages: any = {
  'angular2-mdl': { main: 'dist/components/index.js'}
};
```

After that you may use the angular2-mdl directives in your components:

```javascript
import { MDL_DIRECTIVES } from 'angular2-mdl';

@Component{
   ...
   directives: [ MDL_DIRECTIVES ]
}
```

You may include the material-deisgn-lite css in your html and you're done!

```html
<link rel="stylesheet" href="https://code.getmdl.io/1.1.3/material.indigo-pink.min.css" />
```

Under [https://getmdl.io/customize/index.html][getmdl-config]{:target="_blank"} you'll find a customizing tool to change the theme colors.

But there is also another way. This package includes the scss files from material-design-lite. With these files you are able to change the colors and other variables in your own scss files:

First of all you need to install node-sass for your project:

```bash
npm install node-sass --save-dev
```

After that you need to configure the sass compiler to use the sass files from the angular2-mdl package. For that the file angular-cli-build.js needs to be extended:

```javascript
return new Angular2App(defaults, {

    sassCompiler: {
      includePaths: [
        `${__dirname}/node_modules/angular2-mdl/src/scss-mdl`
      ]
    },
    vendorNpmFiles: [
      ...
    ]
  });
```

Now you can use the sass sources form angular-material-lite and change the used colors in your app:

```scss
@import "color-definitions";

$color-primary: $palette-blue-500;
$color-primary-dark: $palette-blue-700;
$color-accent: $palette-amber-A200;
$color-primary-contrast: $color-dark-contrast;
$color-accent-contrast: $color-dark-contrast;

@import 'material-design-lite';
```

### Which components are available right now

The following components are included in the version 1.0.0-rc1:

* Badges
* Buttons
* Cards
* Icons
* Loading
* Shadow
* Toggle (Checkbox, Radio, Icon Toggle, Switch)
* Lists
* Slider
* Snackbar
* Table
* Tooltips
* Menu
* Layout (standard, scroll, waterfall, tabs)
* Tabs
* Textfields (multiline, expandable)

You'll find demos and examples on how to use these components within the [e2e test application][angular2-mdl-demo]{:target="_blank"}

My next steps are: polish the api and components/directive selectors and extend the documentation.


[getmdl]: https://getmdl.io
[getmdl-config]:https://getmdl.io/customize/index.html
[material2]: https://github.com/angular/material2
[angular2-mdl-demo]:http://mseemann.io/angular2-mdl/
[mdl2-github]:https://github.com/mseemann/angular2-mdl
