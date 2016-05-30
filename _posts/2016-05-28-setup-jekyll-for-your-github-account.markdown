---
layout: post
title:  "Setup a jekyll site for your github account."
date:   2016-05-28 18:41:26 +0200
categories: general
tags: 
  - jekyll 
  - site 
  - github
---

May be you have heard about [GuthubPages][github_pages]{:target="_blank"}. If not, it is a quite easy way to create a website that is hosted at GitHub. And if you want you can make this website available under your own domain. Let's get started:

First of all you need to create a GitHub repository with a name that will match your GitHub Pages website url. My github username is mseemann. So the name of the repository must be `mseemann.github.io`. Every file that is present in this repository will be part of your
website. Try it out and create a simple `index.html` in your repository, commit, push and access your website `http://[username].github.io`. 

Ok that works. But this is not perfect. You have to do a lot of things manually or generate your website with a seperate content 
management system. A content management system that is a real timesaver for developers is [jekyll][jekyll]{:target="_blank"}. You can write your pages in markdown, you can work with templates and you may extend the capabilities of the cms with plugins. GitHub Pages supports jekyll 
out of the box.

To get startet with jekyll you need to install jekyll by gem:

```bash
gem install jekyll
```` 
After that you need to run 

```bash
jekyll new .
````
to create a scaffolded version of your website in the current folder. Want to see your new website? run

````bash
jekyll serve
````
and point your browser to `http://localhost:4000`. Any changes you now make in your sources will be regenerate your website immediately and
is instantly available under `http://localhost:4000`.

Now you need to commit and push your new website in your gitHub repository and a few seconds later the content is abvailable under  `http://[username].github.io`.

To make the things perfect it would be nice to publish this website under your own domain. This is also possible. You need to create a `CNAME` file in your repository. This is a simple text file that must contain the domain name under which you want to access your website. After that you need to configure the DNS A record for your website. The ip adresses you need to use is documented here: [DNS A-Record configuration][github_dns]{:target="_blank"}. Thats it! Your website is now avaialable under your own domain for free.

  
[github_pages]: https://pages.github.com/
[jekyll]: https://github.com/jekyll/jekyll
[github_dns]:https://help.github.com/articles/setting-up-an-apex-domain/
