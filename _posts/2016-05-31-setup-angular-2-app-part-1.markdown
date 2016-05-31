---
layout: post
title:  "Setup Continuous Integration for an Angular 2 app hosted on GitHub - Part I"
date:   2016-05-31 13:41:26 +0200
categories: frontend
tags: 
  - angular 2
  - angualr cli
  - GitHub
  - TravicCI
---

In this post I'll describe how you can setup continuous integration for an angular 2 app that is hosted on github with [Travic-CI][travis-ci]{:target="_blank"}, [Coveralls][coveralls]{:target="_blank"} and [Sauce Labs][saucelabs]{:target="_blank"}. After that you have a nearly complete pipeline for your agile project. I am using [roamlrs][roamlrs]{:target="_blank"} as an example project.

To get started you need to create a GitHub repository that you have initialized with ng init from the [Angular CLI][angular-cli]{:target="_blank"} project. How to do this is described on the Agular CLI website so I leave it up to you following they're instructions.

<!-- more -->

The first thing we want to do is running our builds at [Travis-CI][travis-ci] after every commit to the master branch. To achieve this we need to create a `.travis.yml` file in our repository and connect our GitHub account with Travis-CI.

Connect your GitHub account with your Travic-CI account is very easy. Navigate to [Travis-CI][travis-ci]{:target="_blank"} and sign in with your GutHub-Account. After that you need to give Travis-CI read access to your GitHub repositories and just need to activate the repositories for which you want to setup automatic builds.

The next step is to configure how Travis-CI should build your code and run your unit tests. This is done by the `.travis.yml` file. The generated project uses Chrome to run our test so we need to install Chrome before we run our tests. You'll find a detailed description how to do that in this [blog post][travis-ci-chrome]{:target="_blank"}. Because Chrome needs a newer ubuntu version than Travis-CI provides by default we need to configure [trusty][travis-trusty]{:target="_blank"} as the runtime environment.

Our complete `.travis.yml` is:

```yaml
language: node_js
sudo: true
dist: trusty

node_js:
  - '5.6.0'

branches:
  only:
  - master

before_install:
 - export CHROME_BIN=/usr/bin/google-chrome
 - export DISPLAY=:99.0
 - sh -e /etc/init.d/xvfb start
 - sudo apt-get update
 - sudo apt-get install -y libappindicator1 fonts-liberation
 - wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
 - sudo dpkg -i google-chrome*.deb
```

If we commit and push our changes to our master branch Travis-CI should grab our sources from GitHub and start buildung and testing our app. But we'll see that the build never stops. The reason is that in the `package.json` of our project the script `test` runs `ng test`. This command will wait for any changes in our sources to rerun the test. A perfect solution for our local test but really a bad one for our Travis-CI build. To solve this we need to change the command from `ng test` to `ng test --watch false`. Commit, push and we are done.

The first step is done. One last thing: let visitors of our GitHub project know that our build passes. Usually the build-badge is included in our readme.md file:

```
[![CI Status]
   (http://img.shields.io/travis/roamlrs/roamlrs.svg?style=flat)]
   (https://travis-ci.org/roamlrs/roamlrs)
```

[roamlrs]:https://github.com/roamlrs/roamlrs
[angular2]: https://angular.io
[angular-cli]: https://github.com/angular/angular-cli#usage
[travis-ci]: https://travis-ci.org
[travis-ci-chrome]: http://blog.500tech.com/setting-up-travis-ci-to-run-tests-on-latest-google-chrome-version/
[travis-trusty]: https://docs.travis-ci.com/user/trusty-ci-environment#tl%3Bdr---Using-Trusty
[coveralls]:https://coveralls.io/
[saucelabs]:https://saucelabs.com/