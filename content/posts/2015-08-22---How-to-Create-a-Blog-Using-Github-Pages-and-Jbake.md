---
title: How to Create a Blog Using GitHub Pages and JBake
date: "2015-08-22T15:45:35+00:00"
template: "post"
draft: false
slug: "how-to-create-a-blog-using-github-pages-and-jbake"
category: "Software Engineering"
tags:
  - "Blog"
  - "JBake"
  - "GitHub"
  - "GitHub Pages"
  - "Static Site"
  - "Site Generator"
description: "Recently I was looking into creating a blog in order to write down some thoughts. In looking for simple ways to create one, but being a bit more technical than the average user, I started by discarding Blogger, WordPress, Medium and the like. I wanted to keep content under my control (let’s pretend for a second that GitHub is under my control) and still have a workflow that allows me to write, save and publish with simplicity and flexibility."
---
Recently I was looking into creating a blog in order to write down some thoughts. In looking for simple ways to create one, but being a bit more technical than the average user, I started by discarding [Blogger](https://www.blogger.com), [WordPress](https://wordpress.com/), [Medium](https://medium.com/) and the like. I wanted to keep content under my control (let’s pretend for a second that GitHub is under my control) and still have a workflow that allows me to write, save and publish with simplicity and flexibility.

As a software engineer, a very simple solution that I found is using Git, which I already use on a daily basis for development work.

So without further ado let’s get started with the details of what we need to get writing and publishing.

#### GitHub Pages setup

The first step is creating a regular [GitHub](https://github.com/) account. There is nothing fancy here and I will not go into any details on how to do this, although it should be pretty straightforward.

Once you have your repository created you should install [Git](https://git-scm.com/) locally in your machine. It’s worth mentioning that the repository name must obey the rule ***username*.github.io** where *username* is your username on GitHub.

After you have Git locally set up you can clone the repository by copying the clone URL of your repository GitHub page and using it on your terminal.

`$ git clone https://github.com/jdmartinho/jdmartinho.github.io.git`

For user repositories [GitHub Pages](https://pages.github.com/) requires that you put your site in the `master` branch. For project repositories, you should use the `gh-pages` branch. For the remainder of this article, we’ll assume we’re interested in creating a page for the user and so we’ll use the `master` branch.

GitHub Pages will pick up whatever content you have in the `master` branch and it will serve it in the http://username.github.io URL. This means that if you just add a simple HTML document to your repository, you instantly get a page on that URL. Despite not being extremely interesting it’s a start.

#### Domain setup

If you have a domain name bought and paid for you can use it to point to your new page with some more configuration steps that I’ll explain now.

You’ll need access to your hosting provider (e.g. GoDaddy) typically to your control panel like cPanel or a custom interface. Here you need to add a CNAME or A record, depending on your goal.

The first step is creating a file called `CNAME` and adding it to the root of our repository. On this file, the only thing we need to add is the domain name we’ll be using to access this site, for instance `jdmartinho.com`.

After having this file in place in the repository, we go to our hosting provider control panel and we look for the option to zone records or alias. Here we’ll add a CNAME record with `jdmartinho.github.io` pointing to the subdomain that we want to use like `www.jdmartinho.com` or `blog.jdmartinho.com`. If we want to use the actual top level domain we’ll need to add an `A` or `ALIAS` record for `jdmartinho.com` to `192.30.252.153` and another `A` record also for `jdmartinho.com` to `192.30.252.154`. These IP addresses belong to GitHub and are subject to change (they have changed in the past). Typically GitHub help will contain articles on the latest IP addresses that you should use so [check here for the latest news on this](https://help.github.com/articles/my-custom-domain-isn-t-working/).

You can quickly check that the domain is correctly set up by running the [dig command online here](http://www.digwebinterface.com/). You should see something like this:

```
www.jdmartinho.com.    14399    IN    CNAME    jdmartinho.com.
jdmartinho.com.        14399    IN    A    192.30.252.153
jdmartinho.com.        14399    IN    A    192.30.252.154

```

#### JBake setup

[JBake](http://jbake.org/) is a great project that takes the [Jekyll](http://jekyllrb.com/) static site generator and brings it to the JVM world via Java. JBake allows us to edit pages using HTML, Markdown or AsciiDoc, process these pages with templates defined using Freemarker, Groovy or Thymeleaf and style them using CSS frameworks such as Bootstrap or Foundation.

Jekyll is what GitHub Pages actually uses behind the scenes to serve the content that you are uploading to the repository. If you want to use Jekyll locally you’ll have to install [Ruby](https://www.ruby-lang.org/en/) and [RubyGems](https://rubygems.org/). In order to use JBake you’ll just need [Java 6+](https://java.com/en/download/) and since I’m quite lazy and I happen to use a Java/JVM environment on a daily basis I decided to give it a try.

We’ll start by going to the [JBake](http://jbake.org/) website and downloading the latest release. Unpack the zip to a location of your choice and add that location to your Path environment variable so that we can launch JBake from any location.

Now let’s take a look at the structure of the site with JBake. I’m borrowing the structure from the official documentation to exemplify:

```
.
|-- assets
|   |-- favicon.gif
|   |-- robots.txt
|   |-- img
|   |   |-- logo.png
|   |-- js
|   |   |-- custom.js
|   |-- css
|   |-- style.css
|
|-- content
|   |-- about.html
|   |-- 2013
|   |-- 01
|   |   |-- hello-world.html
|   |-- 02
|   |-- weekly-links-1.ad
|   |-- weekly-links-2.md
|
|-- templates
|   |-- index.ftl
|   |-- page.ftl
|   |-- post.ftl
|   |-- feed.ftl
|
|-- jbake.properties

```

On the `assets` directory, we have the typical static files that we’ll need to load the pages, such as CSS, JavaScript and image files. Usually, we’ll edit this folder once at the beginning and then let it be unless we’re adding a new JavaScript library or editing our CSS in order to get a new look on the site.

On the `content` directory is where we will keep our data that will be used by JBake to craft the site. This is where we put our HTML, Markdown or AsciiDocs, which means this is where we’ll spend most of the time after our initial setup. After all, the whole point of this is getting us to write more and spend less time fiddling with the process of publishing.

On the `templates` directory, we keep the template files to edit with the language of our choice. Below I have a simple example in Freemarker for the post page that comes with JBake.

```html
<#include "header.ftl">    
    <#include "menu.ftl">    
    <div class="page-header">
        <h1><#escape x as x?xml>${content.title}</#escape></h1>
    </div>
    <p><em>${content.date?string("dd MMMM yyyy")}</em></p>
    <p>${content.body}</p>
    <hr />    
<#include "footer.ftl">

```

We can easily create an example site by issuing the command

`$jbake -i`

It’s very useful to run this command the first time as it will provide us with the structure that we can then adapt to our needs. Finally when we have some content and we are ready to generate the site we can do

`$jbake -b . blog`

This will generate the site from the root of the repository and put it into the `blog` directory. Bear in mind that the `blog` directory will be overwritten by this. An easy way to bypass having to issue this command to generate the site with the source and output directories is to configure the default in `jbake.properties` such as

`destination.folder=blog`

Having configured this, we can just do `$jbake -b` from our root directory.

#### Workflow

Now that we have the site structure in place a typical workflow for me goes like this:

- Pull the latest code available because I’ll use multiple laptops to work on. I’ll do this with `$git pull origin master` or any other branch name that you might be working on.
- Navigate to my content directory and create a new file for a new blog post. This usually takes the form of `year-month-day-post-name` and I’ll usually write in Markdown which is starting to become ubiquitous.
- Save the work on and commit it with `git commit -m "My message."`.
- If it’s a work in progress then we have two choices: either I’ll push the commit to a branch different than `master` and later when it’s finished I’ll merge that branch or I’ll push it to `master` now but I’ll keep the header options of the post as `status=draft`. In the latter, I’ll then change the status to `status=published` when I’m done with the blog post so that JBake will pick it and publish it.
- Run the `$jbake -b` command, pushing the content to the remote repository and see it published.

Here are the links for the official documentation. Go explore!

JBake Documentation – <http://jbake.org/docs/>

Freemarker Manual – <http://freemarker.org/docs/index.html>