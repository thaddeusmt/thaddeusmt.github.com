---
layout: post
title: "Converting my blog from Textpattern to Jekyll/Octopress"
date: 2013-01-05 12:16
comments: true
categories: [Meta]
keywords: octopress,jekyll,textpattern,github pages,blog,blogging
published: false
---

For 2013 I redid the Chili Pepper Design blog (the one you are reading right now). My goals for the revamp were as follows:

* Keep all blog posts, ditch the old web design portfolio and news posts
* Keep blog post URL structure
* Keep blog post comments
* RSS feed (with same URL as old one)
* Responsive HTML5 theme
* SASS or LESS CSS for theme
* Beautiful code snippet formatting
* Good SEO
* Fast and easy to set up
* Not WordPress

Things Octopress provided:
* Default post link URL structure using dates was the same
* Default HTML5 responsive theme
* CSS using SASS

How it works:
You need Ruby locally, to get the Octopress Gem, to run the rake commands to build your blog.
Originally I thought it would be cool just to use Jekyll, and let GitHub Pages build it for me
But I wanted some of the extra Octopress features, and it's nice to preview it locally,

One problem I ran in to was that pointing my main domain to GitHub Pages caused some issues with my email, which was still being routed through my regular webhost. I figured I could work around this, but then I realized I wanted to be able to do some 301 rewrites too. So I decided to still host the new blog on my old host, instead of GitHub.

Blog content and navigation structure

Permalink structure

Blog style/theme

Converting posts from database to static files
-suggested (https://github.com/mojombo/jekyll/wiki/blog-migrations)
-- didn't work on Windows, couldn't compile something

Comments migration
-options? pretty much just Disqus (link to static Jekyll comment plugins)
-using Textpattern views to generate Disqus import format
-sign up for Disqus and import
-need to have EXACT same post URL for the Disqus comments to appear
-How could I have fixed the encoded HTML entities?

RSS

http://www.chilipepperdesign.com/rss/?section=blog

/sigh Pygments on Windows issues
