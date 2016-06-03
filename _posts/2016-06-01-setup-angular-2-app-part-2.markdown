---
layout: post
title:  "Setup Continuous Integration for an Angular 2 app hosted on GitHub - Part II"
date:   2016-05-31 21:30:26 +0200
categories: frontend
tags: 
  - angular 2
  - karma
  - coveralls
  - GitHub
  - Travis-CI
---

In the first part of this series we have seen how to run our tests on [Travis-CI][travis-ci]{:target="_blank"}. This is great because we now know when our code is broken. But we don't know how much of our code is covered by our tests. In this part we'll going to add coverage reporting for our tests.

<!-- more -->
Before we get started some thoughts about testcoverage:

- Our code is written in TypeScript and gets transpiled to JavaScript before we run the tests by karma. So we'll see the coverage of the JavaScript code not the TypeScript code. You'll notice that it is really easy to understand which part of your code is not perfectly covered by your test but reaching a 100% test coverage may be difficult to reach because of a little bit of meta code that is added by the transpiler (for example the decorator code).
- We only want to know how much of our app code is tested not how much of the testcode has been run. So we need to exclude all *.spec.js files from our coverage measurement.

To add test coverage reporting to our Angular 2 app we need two additional npm modules:

```bash
npm install karma-coverage karma-coveralls --save-dev
```
The module `karma-coverage` is responsible for code instrumentation and coverage reporting. After the reports have been written to the filesystem the module `karma-coveralls` will help us to transmit the report to [Coveralls][coveralls]{:target="_blank"}.

After these modules have been installed we need to customize our karma configuration. You'll find the configuration in `config/karma.conf.js`. First of all we need to require the module `karma-coverage` in the plugins array: `require('karma-coverage')`.

The next step is to configure a pre process step:

```javascript
preprocessors: { 'dist/app/**/!(*spec).js': ['coverage'] },
````
Here we include all js files from the `dist/app`folder but exclude the `spec`files.

The last change results in coverage reports written to the file system. We configure `html` (for ourself) and `lcov` for coveralls:

```javascript
coverageReporter: {
  dir : 'coverage/',
    reporters: [
      { type: 'html' },
      { type: 'lcov' }
    ]
},
```
If we now run `ng test --watch false` we'll find the reports in the `coverage` folder. The complete `karma.conf.js` file can be found [here](https://github.com/roamlrs/roamlrs/blob/538288a4b188154ed950455295a19eaf5210ede0/config/karma.conf.js){:target="_blank"}.

To get a nice report view and a badge for our GitHub project we need to transfer the coverage results to Coveralls. To achieve this we need to modify the `.travis.yml` file and add some script code after the build/test has finished. With help from the `karma-coveralls` module two lines of code are enough:

```bash
after_script:
  - cat ./coverage/*/lcov.info | ./node_modules/coveralls/bin/coveralls.js
``` 
For sure we need an account at Coveralls. This is also very easy. Just login with your GitHub account, grant access to Coveralls and activate Coveralls for the desired project.

Done!

Now we can ad our second badge to our Angular 2 app:

```
[![Coverage Status](https://coveralls.io/repos/github/roamlrs/roamlrs/badge.svg?branch=master)]
   (https://coveralls.io/github/roamlrs/roamlrs?branch=master)
```



[roamlrs]:https://github.com/roamlrs/roamlrs
[angular2]: https://angular.io
[angular-cli]: https://github.com/angular/angular-cli#usage
[travis-ci]: https://travis-ci.org
[travis-ci-chrome]: http://blog.500tech.com/setting-up-travis-ci-to-run-tests-on-latest-google-chrome-version/
[travis-trusty]: https://docs.travis-ci.com/user/trusty-ci-environment#tl%3Bdr---Using-Trusty
[coveralls]:https://coveralls.io/
[saucelabs]:https://saucelabs.com/