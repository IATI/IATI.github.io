---
layout: post
title:  "The IATI Unified Platform is Launched!"
author: Alex Lydiate
date:   2021-09-10
categories: blog
---
### Today we launched the IATI Unified Platform!

For the last decade or so, IATI software as cared for by the core IATI technical team has been a smorgasbord of tools contributed by all sorts, each operating independently on innumerable servers, each repeating a number of things in different and exotic ways - most presciently, downloading the 8000+ plus IATI documents to occupy many duplicated gigabytes of storage space.

[DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) I hear you cry, and quite right too. To whit, here's a pretty picture of the architecture of the all-new Unified Platform:

![IATI Unified Platform](/assets/IATIUnifiedPlatformArchitecture.png)

The first product to set sail aboard the Unified Platform is Version 2 the [IATI Validator](https://iativalidator.iatistandard.org). The architecture of the platform has allowed pretty huge performance benefits over the previous version:

* __The Validation microservice allows on-demand validation of IATI documents__ using a pure Javascript implementation on a serverless platform, meaning publishers can now get validation reports for their documents in seconds, where before they'd be sat in a queue with every other document from every other publisher, waiting for a BASH script to pick them up and put them through a relatively slow Java application. __There is also a public [Validation API](https://developer.iatistandard.org/api-details#api=iati-validator-v2&operation=post-pub-validate-post) for this, API fans__.

* __The centralised IATI document management operates with a configurable number of parallel processes to download many documents simultaneously__, and polls the registry every five minutes to get the state of play. This means new documents are available to the validator extremely quickly. Indeed, the limiting factor is the environment - we don't want to be a burdon to the publisher servers, and so we keep the parallelisation relatively low, presently 10 processes. Even so, we can identify and download all known documents within an hour.

The next product we're going to be delivering on the platform will be the next iteration of the IATI Datastore, which allows a fast search of all 1 million+ IATI documents. Much of that work is well underway, with the system already capable of flattening and indexing all of IATI into Solr perfectly dynamically, by recursively parsing each document and also by generating its Solr schema directly from the IATI schema. The next stage will be to define the public API contract, which will to a greater extent be a subset of the Solr API.

Onwards and upwards!

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
