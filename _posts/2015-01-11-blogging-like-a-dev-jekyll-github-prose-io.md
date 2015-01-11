---
layout: post
published: true
title: "Blogging like a dev: Jekyll + GitHub + Prose.io"
mathjax: false
featured: false
comments: false
---

You want a blog that is fully customizable and is sick of WordPress mess and insecurity? One solution that I found is to use Jekyll to build a blog where all posts are in markdown format, is hosted in github and I can edit using a browser with prose.io.

Summary of benefits:
- No database, only Markdown files
- Hosted for free by GitHub Pages
- Version controlled
- In browser rich editor with Prose.io
- Highly customizable and themeable
- Cool factor

## Step 1: Forking from another blog
The easiest way to do is to fork a jekyll theme in your GitHub account. Best ones I could find are:
- [Notepad](https://github.com/hmfaysal/Notepad) - used by this very blog
- [Clean Blog theme by Start Bootstrap](https://github.com/IronSummitMedia/startbootstrap-clean-blog-jekyll)

![howtoblog-githubsettings.png](/images/howtoblog-githubsettings.png)

Rename your forked repository to _yourusername_.github.io, so GitHub Pages get in action.

**Some theme repositories use gh-pages as the default branch and others use master. You must check in the theme docs which one is correct. For Notepad is master.**

## Step 2: Preparing your environment
Requirements are: Ruby, Gem and GIT.
From there you can install jekyll by running:

    gem install jekyll

Optionally you can create a blank site, by following [jekyll documentation](http://jekyllrb.com/docs/quickstart/), but that's not the method we will describe in this post.

## Step 3: Cloning your repository
If you forked correctly, you will have a repository on github.com/_yourusername_/_yourusername_.github.io

Clone it in your dev machine:

    git clone git@github.com:adenot/adenot.github.io.git
    cd adenot.github.io
    
_replace adenot with your github username_

Now run your blog locally with:

    jekyll serve
    
If all goes right, you should see the following page:
![howtoblog-notepadinitial.png](/images/howtoblog-notepadinitial.png)


    

