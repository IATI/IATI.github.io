---
layout: post
title:  "Search Engine Optimization in the Age of Single-page"
author: alex_miller
date:   2022-03-14
categories: blog
---

How can one combine the advantages of modern single-page applications with traditional search-engine optimization (SEO)? Here's the lessons we learned while implementing SEO on the IATI Datastore Search.

## Project requirements

It was clear from the start of the project that two things stood in the way of new users finding and using IATI data: speed and indexing. If pages take too long to load, users will quickly get frustrated and leave. Likewise if they don't know the information they're seeking is contained in IATI in the first place, they don't have a good avenue to be connected to it in the first place. Our site needed to be as fast as static content, while still being easily discoverable by search engines and new users.

On site speed, a website is only as fast as its slowest-loading component. While IATI XML itself can be relatively compact, it quickly expands when flattened into more user-friendly formats. As measured at the beginning of this year, the average XML file used about 11 kb per IATI activity. Meanwhile, a sample CSV export of every possible field expanded to the budget-level clocks in at 475 KB per activity. That's over a 4000% expansion in size due to the necessity to repeat fields. A lot of effort and server-capacity was expended making the [IATI Unified Platform](/blog/2021/09/13/unified-platform-launch.html) as fast as possible to serve these flattened formats and [download them](/blog/2022/03/09/scalable-downloading-feature.html) as efficiently as possible. The last thing we wanted was a bloated web framework acting as the bottleneck, so we turned to the static, single-page generating features of VueJS.

## Advantages and disadvantages of a static single-page application

With the launch of the [Datastore Search](https://iatidatastore.iatistandard.org), it's safe to say that we've learned a lot about static single-page applications in the process. Most importantly, it can't be understated how quick they can be. At the time of writing, the page completely finishes loading (from an external page) in an average of 1.1 seconds (intra-page view changes render even faster). For comparison, the [IATI Standard Website](https://iatistandard.org/), based in Wagtail, averaged 6.4 seconds per page load over the last month. Why is our Wagtail site so much slower? It's because rendering is fully coupled to a Postgres database. Each and every one of the ~2,700 pages on the Standard website first needs to be fetched from the database before the framework can begin rendering it. And as the site grows, each additional page incrementally slows down the loading of others. Shortly after launch in late 2019, the Wagtail site settled into an average load time of about 2.4 seconds, but has gradually slowed as the number of pages has grown. Meanwhile, the Datastore Search, with the Unified Platform API acting as a backend, is capable of searching and displaying a page for every one of over 1 million IATI activities.

![Google Analytics showing an average of 1.1 second loads over 191 users](/assets/fast.png)

The main disadvantage of static single-page applications, on the other hand, is the loss of control over the server-side processes. You want to serve a `robots.txt` file so bots can scrape your content? You need to physically write a `robots.txt` in your files or via your deployment process, otherwise it won't exist. And there's absolutely no way for a single-page application to fully imitate a static file. By definition, the single-page application lives within the HTML DOM; the bots employed by search engines to index content won't be fooled just by printing text in the HTML tags or re-writing the URL. `robots.txt` needs to have a `text/plain` content-type header, and sitemaps will need to have an `application/xml` content-header. So without control over your web server's ability to write content types, your single-page application cannot create sitemaps and cannot serve them on the fly. And given that our VueJS Datastore Search had a million pages of rich content created via JavaScript, we needed to be able to tell browsers how to access and index it in order to fulfill the indexing project requirement.

## How we fit dynamic SEO into a static site

First, even though we lost control of our server-side processes, we still retained control over DNS. Especially useful is Cloudflare's "Page rule" feature, where you can implement a number of helpful functions like enforcing HTTPS or rewriting specific endpoints. So we had an avenue to rewrite `/sitemap-{id}.xml` to any arbitrary server of our choosing.

With a rewrite in hand, we could retain all of the advantage of our static website while using whatever technology we needed to serve the sitemap. Most accessible for us was an Azure consumption-tier Function that would only run when it was pinged by the Google bots. So we implemented an extensible [sitemapper](https://github.com/IATI/sitemapper) Function that would be able to accept redirects from any given static site, and serve dynamic sitemap content to the indexing bots. From that point, we could make use of the same APIs that fed the static website to create dynamic sitemap XML based upon IATI identifiers.

![Google Search Console showing the HTML of a crawled page with a title and meta description tag](/assets/crawled_seo.png)

After Google has an index of your dynamic pages, the Googlebot is fully capable of waiting for JavaScript-enabled content to load. As you can see above, in the pages Google has already indexed, the dynamic page title and meta description tag are populated by the API prior to the indexing snapshot. Now, only one week after launch, about 13% of our traffic is already coming from organic search, and that number is sure to increase further as Google indexes more of the IATI text within our content.
