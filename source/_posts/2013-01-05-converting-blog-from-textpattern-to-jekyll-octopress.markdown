---
layout: post
title: "Converting my blog from Textpattern to Jekyll/Octopress"
date: 2013-01-05 12:16
comments: true
categories: [Meta,Web-Development]
keywords: [octopress,jekyll,textpattern,github pages,blog,blogging]
published: false
---

For 2013 I redid the Chili Pepper Design blog (which you are reading right here). My goals for the revamp were as follows:

* Keep all blog posts, ditch the old protfolio and news posts
* Keep post URL structure the same
* Keep the comments on the old blog posts
* RSS feed - same url? (old: http://www.chilipepperdesign.com/rss/?section=blog)
* HTML5 responsive theme
* CSS using SASS/LESS
* Nice code formatting
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

Blog content and navigation structure

Permalink structure

Blog style/theme

Converting posts from database to static files
-suggested (https://github.com/mojombo/jekyll/wiki/blog-migrations)
-- didn't work on Windows, couldn't compile something

Comments!
-options? pretty much just Disqus (link to static Jekyll comment plugins)
-using Textpattern views to generate Disqus import format
-sign up for Disqus and import
-need to have EXACT same post URL for the Disqus comments to appear
-How could I have fixed the encoded HTML entities?

RSS

