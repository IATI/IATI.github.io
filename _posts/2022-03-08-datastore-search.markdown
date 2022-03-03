---
layout: post
title:  "Datastore Search"
author: alex_lydiate
date:   2022-03-08
categories: blog
---
Today we released [Datastore Search](https://iatidatastore.iatistandard.org).

Over the lifecycle of previous iterations of IATI Datastore interfaces we noted two themes - they had only a very small number of users, and we'd receive messages from potential users to say, I want to explore IATI data, but I don't wish to have to learn the ins and outs of the standard to do so. An example of a group well represented in these messages were journalists - not the traditional users of IATI data, but one which could gain great value from over a million published IATI activities, absolutely rich with natural language content of interest.

So, the Datastore Search landing page is designed to be as simple and intuitive as possible, using a ubiquitous convention - it is emminently and very clearly a search engine:

[![IATI Datastore Search](/assets/dssearch.png)](https://iatidatastore.iatistandard.org)

It shares all the familiar search-enginy qualities with which anyone who has ever used a search engine will be familiar. Where as the landing pages and interfaces of previous Datastore iterations offered an immediate depth of text or complex forms, here we invite new users to enter a search term and dive in.

For those who wish to dive deeper, [Advanced Search](https://iatidatastore.iatistandard.org/advanced) offers the potential to dive deeper than ever before, with the ability to chain filters on any and all IATI elements together in complex and powerful ways. These chains can be saved to file, reloaded and edited. Search results can be downloaded in JSON, in CSV, or in valid IATI XML format.

If that level of depth still isn't deep enough to float your boat, the [API](https://developer.iatistandard.org/api-details#api=datastore&operation=query) is available, and very, very powerful indeed.

And that about completes the first delivery of the IATI Unified Platform.

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
