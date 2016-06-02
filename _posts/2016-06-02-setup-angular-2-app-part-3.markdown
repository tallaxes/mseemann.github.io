---
layout: post
title:  "Setup Continuous Integration for an Angular 2 app hosted on GitHub - Part III"
date:   2016-06-02 18:30:26 +0200
categories: frontend
tags: 
  - angular 2
  - protractor
  - Sauce Lab
  - GitHub
  - Travis-CI
---
The previous two parts of this series covered how to setup the build process with [Travis-CI][travis-ci]{:target="_blank"} and how we can publish our test coverage on [Coveralls][coveralls]{:target="_blank"}.

This post covers how to setup the e2e testing for our project. We will use [Sauce Labs][saucelabs]{:target="_blank"} for our e2e testing. You'll need at least a test account to get started without any fees. If you have an open source project you may create an account on [Open Sacue][opensauce]{:target="_blank"} it's also free with unlimited testing time.


<!-- more -->

### Deploy the app on GitHub pages
The first problem we need to solve is where should our app be published so that the e2e tests can access the app. Angular CLI provides a task that makes it possible to publish the app on GitHub pages. GitHub pages are created from the branch `gh-pages` in our GitHub repository. The web address will be `http://[github user].github.io/[github project]`. 

To run this task during the Travis-CI build we need to make sure that `git` commands during the build have access to our GitHub repository. There are a lot of ways to achieve this. I'll present here a simple straight forward way to solve this problem.

First of all we need an access token for our GitHub project. An access token can be created from your profile/settings page ([https://github.com/settings/tokens](https://github.com/settings/tokens){:target="_blank"}). This token needs to be set as an environment variable `github_token` for our Travic-CI project.

After that we must create a `.netrc` file in the users home directory and tell the system that it should use a specific username and password for any access to github.com. The script will be:

```bash
#!/bin/bash
set -e
echo "machine github.com" >> ~/.netrc
echo "   login <your github email>" >> ~/.netrc
echo "   password $github_token" >> ~/.netrc
```
This script must be called during the `before_install` step:

```yml
- ./scripts/setup-github-access.sh
```

After the build process has finished it is possible to run `npm run deploy-gh-pages`. But there is a little problem. This will only work if the branch `gh-pages` did not yet exist. To solve this we run another script before this step that deletes the remote branch `gh-pages` if the branch already exist:

```bash
#!/bin/bash
set -e
git ls-remote --heads | grep gh-pages > /dev/null
if [ "$?" == "0" ]; then
  git push origin --delete gh-pages
fi
```
After the commit and push to GitHub Travis-CI will now build our app and at the end the script is going to publish our app on GitHub Pages and we are done: our app is running on a public server and can be accessed by our e2e tests.

### Run e2e tests with protractor

Now we need to configure [Protractor][protractor]{:target="_blank"}. Protractor is the swiss knife for e2e testing for angular apps. Angualr CLI has created a complete configuration for us but the generated one is only suitable for local test with a single browser. So we create a new `protractor.sauce.conf.js` file in the `config`folder:

```javascript
/*global jasmine */
var SpecReporter = require('jasmine-spec-reporter');

var buildNumber = 'travis-build#'+process.env.TRAVIS_BUILD_NUMBER;

exports.config = {
  sauceUser: process.env.SAUCE_USERNAME,
  sauceKey: process.env.SAUCE_ACCESS_KEY,
  allScriptsTimeout: 11000,
  specs: [
    '../tmp/e2e/**/*.e2e.js'
  ],
  multiCapabilities: [
    {
      browserName: 'safari',
      platform: 'OS X 10.11',
      name: "safari-osx-tests",
      shardTestFiles: true,
      maxInstances: 5
    }
  ],
  sauceBuild: buildNumber,
  directConnect: false,
  baseUrl: 'http://roamlrs.github.io/roamlrs/',
  framework: 'jasmine',
  jasmineNodeOpts: {
    showColors: true,
    defaultTimeoutInterval: 30000,
    print: function() {}
  },
  useAllAngular2AppRoots: true,
  beforeLaunch: function() {
    require('ts-node').register({
      project: 'e2e'
    });
  },
  onPrepare: function() {
    jasmine.getEnv().addReporter(new SpecReporter());
  }
};
```
Some notes about the changes:

- The `buildNumber` is initialized with the bild number from Travis-CI. So build and Sauce Labs test runs are connected to investigate potential problems.
- The original `protractor.conf.js` file uses the TypeScript sources to run the tests. For now this only works with one browser - but we want to test against multiple browswers. To makes this possible we need to compile the `*.ts`files into JavaScript. This must be done before the e2e tests are performed: `tsc --project e2e`. The output is written to `/tmp/e2e` (have a look at tsconfig.json `"outDir": "../tmp/e2e"`).
- To tell protractor to use Sauce Lab we need to add `sauceUser`and `sauceKey` to the config file. The values are readed from the Travis-CI environment. You'll find these information in you Sauce Labs account settings.
- In the array `multiCapabilities` we can specify on which browsers our tests should be processed. To add more configurations we can use the [Sauce Labs platform configurator][plattformconfigurator]{:target="_blank"}.

### Run the e2e tests in the last build step

Now we can extend our `.travis.yml` to run the e2e test. In the `after_success` block we extend:

```yml

  # remove branche gh-pages
  - ./scripts/delete-gh-pages.sh
  # deploy to github pages if build succeds
  - git status; npm run deploy-gh-pages
  # turn back to master
  - git checkout master
  # give github a little bit of time to update the github page
  - sleep 10
  # run e2e test
  - npm run e2e config/protractor.sauce.conf.js
  
``` 
The comments should explain what is going on in all of these steps. If you run into problems that the GitHub pages are not the just generated version it could help to increase the sleep time from 10 seconds to a higher value.

You'll find the complete version after the three parts as a tagged version at GitHub: [Tag: complete-artice][projectresult]{:target="_blank"}.

The last thing we have to do is to create the badge for our e2e tests. We use the matrix badge from Sauce Labs. This creates an image with the status of all browsers that our app has tested against:

```
[![Sauce Test Status](https://saucelabs.com/browser-matrix/sacuemseemann.svg)]
  (https://saucelabs.com/u/sacuemseemann)
```

<img src="/assets/img/saucelab_browser_matrix.png" alt="Browser Matrix" style="width: 100%;"/>



[travis-ci]: https://travis-ci.org
[coveralls]: https://coveralls.io/
[saucelabs]: https://saucelabs.com
[opensauce]: https://saucelabs.com/opensauce/
[projectresult]: https://github.com/roamlrs/roamlrs/tree/complete-article
[plattformconfigurator]: https://wiki.saucelabs.com/display/DOCS/Platform+Configurator#/
[protractor]: http://www.protractortest.org/#/