---
layout: post
title: "Facebook Page Optimization for Graph Search"
date: 2013-01-18 15:48
comments: true
categories: [Facebook, SEO]
keywords: facebook,graph search,facebook page,brand page,seo,facebook search,open graph search,graph search rank
description: "How to Optimize your Facebook Page for the new Graph Search"
---

Facebook is currently working on their new [Graph Search](https://www.facebook.com/about/graphsearch) product. It's in invite-only Beta right now, but it will be rolled out to all users eventually. Facebook has a crack team of [ex-Google employees](http://techcrunch.com/2013/01/15/here-is-the-ex-googler-dream-team-that-led-facebooks-new-graph-search-tool/) working on it, and the demos are impressive.

Currently, Graph Search does not complete directly with Google and is not intended for general "informational", keyword-based web searches. Rather, the idea is to tease interesting data out of social graph connections and relations, such as "who as likes to mountain bike where I work", "what games do my friends play" and "best pizza place near me". The Google+ "Search Plus Your World" product has a similar goal, enhancing search results with information from Google+, but it doesn't support relational queries. The number of brands and restaurants on Facebook actually positions Graph Search more as a competitor to Yelp and Foursquare than Google.

Facebook has not announced a monetization strategy for Graph Search yet (to the [disappointment of shareholders](http://beta.fool.com/mariecabural/2013/01/18/facebook-graph-search-fails-impress-threatens-yelp/21787/)), but one obvious route looks to be selling search ads that are targeted by combining "search intent" - which is what Google has built it's business on - with Facebook's valuable social data.

Privacy is an immediate concern with Graph Search, so much so that Facebook tried to allay in their [initial announcement](http://newsroom.fb.com/News/562/Introducing-Graph-Search-Beta). The effectiveness of social search relies of the amount of public data users share, putting it at odds with user's privacy needs. It will be interesting to see how this plays out.

It is early days still and speculation about Graph Search is rampant. But it will be a valuable discovery tool for businesses with Facebook Pages and Applications, and because of that Facebook has already released a [few](https://developers.facebook.com/blog/post/2013/01/16/platform-updates--operation-developer-love/) [resources](http://www.facebook-studio.com/news/item/introducing-graph-search-help-people-discover-your-business) that provide a basic outline of how the search will work, and how businesses can optimize for it.

 The goal of this post if for me to synthesize the changes for my own understanding, but I thought I would share it with the world as well. I will hopefully update this page as more information becomes available and my understanding improves.

 <!-- more -->

## What results does Graph Search return?

 Facebook says Graph Search can find "[people, photos, places and interests](http://newsroom.fb.com/News/562/Introducing-Graph-Search-Beta)". In technical terms, the goal of the Graph Search is to return Open Graph Object results, based on their connections to other objects. There are many [core Facebook Graph Object types](https://developers.facebook.com/docs/reference/api/#objects), but the ones Facebook is saying it will focus search on are:

* Pages (Brand Pages and Place Pages)
* Facebook Apps
* Groups
* Photos
* Videos

There are also [general App and website Open Graph Objects](http://developers.facebook.com/docs/concepts/opengraph/objects/). These are the objects outside of Facebook which have been created through the Graph API, or with [Open Graph meta tags](http://developers.facebook.com/docs/opengraphprotocol/) on HTML pages. These built-in types are:

* Article
* Blog
* Book
* Profile (External)
* Movie
* TV Episode
* TV Show
* Video
* Website

Presumably Facebook will also index [custom Graph Objects](http://developers.facebook.com/docs/technical-guides/opengraph/using-online-object-tool/), although that will be more challenging than indexing the clearly defined built-in types. There is currently no information about this.

If Facebook can't find any relevant Object results, it will fall back to [show web results from Bing](http://techcrunch.com/2013/01/15/if-its-not-in-graph-search-facebook-hands-your-query-off-to-bing/).

## What Facebook Page data is indexed?

Details are still scarce on this, but [we at least know](http://www.facebook-studio.com/news/item/introducing-graph-search-help-people-discover-your-business) high-level object attributes like 'name', 'about', 'description' and 'category' will be the most important search data. You can read more about the key Page attributes in the [documentation](http://developers.facebook.com/docs/reference/api/page/), but here is a summary of the key Page metadata:

* Name
* Vanity URL
* Category
* Location/Address (for Place Pages)
* About
* Description
* General Info

Good general SEO practices should suffice for this content. Make sure you clearly describe your Page using relevant keywords, and categorize it properly. If it's a Place Page with a physical location, make sure the address is correct so location or "geo" searches will work properly.

## What Facebook App data is indexed?

If you have a Facebook Application, the [Developer blog recommends](https://developers.facebook.com/blog/post/2013/01/16/platform-updates--operation-developer-love/) that you make sure the following "App Info" fields are up-to-date in the  "App Details" settings:

* Display Name
* Tagline
* Description
* Detailed Description
* Category

## What other Open Graph Object data is indexed?

Each type of graph object has different attributes, so it will vary. For instance the [Facebook Group Object](http://developers.facebook.com/docs/reference/api/group/)is much simpler than the Facebook Page, with just two key fields:

* Name
* Description

If you are optimizing content on your website or blog for Graph Search, refer to the documentation for the [built in Graph Objects](http://developers.facebook.com/docs/concepts/opengraph/objects/) and define your [meta tags](http://developers.facebook.com/docs/opengraphprotocol/) appropriately. Remember that Facebook has a [graph object debug tool](http://developers.facebook.com/tools/debug) you can use to double-check your content.

## How are the results ranked?

[Facebook has stated that](http://newsroom.fb.com/News/562/Introducing-Graph-Search-Beta) "all results are unique based on the strength of relationships and connections" in a user's social graph. This means the search rank incorporates the data shared with a user by their friends, plus any public data from users who are outside of the user's personal graph. So in addition to relevance from matching meta data on Facebook Graph objects (described above), other social indicators are taken in to account. It will be a similar algorithm to [EdgeRank](http://whatisedgerank.com/) I imagine, including Likes, Check-ins and other social actions. The implication is that Objects with many social interactions will be ranked higher, and Objects with social interactions from the searching user's _friends_ will rank even higher.

## What does this mean for optimization?

It means the buzzword "GSO" (Graph Search Optimization) has already been coined! But in all seriousness, Facebook will rank results by _overall_ social engagement: check-ins, page views, comments, etc. Optimizing a Page will mean filling out the metadata, and running an active Page that engages users with fresh, relevant content and frequent social interactions. These are the same recommendations that have always been made for running an effective Facebook Page, no surprises here. I would start by following techniques for improving [EdgeRank](http://whatisedgerank.com/) until more specific data about Graph Search ranking comes out.

## What if no Open Graph Objects are found?

As mentioned above, if Graph Search fails to find enough results in the Graph, it will fall back to Bing search results from the web. This implies that it will also be beneficial to make ensure related web properties (e.g. a website) are properly [optimized for Microsoft's Bing search engine](http://www.bing.com/community/site_blogs/b/webmaster/archive/2009/09/03/search-engine-optimization-for-bing.aspx). The Bing blog has an official post with [examples of Facebook/Bing integration](http://www.bing.com/community/site_blogs/b/search/archive/2013/01/15/sof.aspx).