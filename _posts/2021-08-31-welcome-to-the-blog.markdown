---
layout: post
title:  "Welcome to the IATI Developer Blog!"
author: Alex Miller
date:   2021-08-31
categories: blog
---
Hello and welcome to the first post on our brand new IATI Developer Blog. We're intending this blog to be a place for us to document the challenges we've faced as we build your favourite IATI tools, as well as the solutions we've designed to overcome them.

Fittingly, I'm going to first describe how we built this simple website. As you may have already noticed, it's a static website that has been written in Markdown, processed by a Ruby package called Jekyll, and served on Github Pages. But how does that all work?

You’ll notice that this post lives in the `_posts` directory of this repository. After cloning the repository locally, you may need to install Ruby, bundle, and the prerequisite Gems to render the site locally. I was able to do so with `sudo apt install jekyll` to install Jekyll and Ruby, as well as `bundle install` to install the Gems.

Then, go ahead and edit this post and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `bundle exec jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

To add new posts, simply add a file in the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.markdown` and includes the necessary front matter. Take a look at the source for this post to get an idea about how it works. After that, simply commit and push your new post and hey-presto, you've got a new post that appears not only on the [home page](/), but also in the [blog archive](/blog/).

Jekyll also offers powerful support for code snippets:

{% highlight xml %}
<reporting-org ref="AA-AAA-123456789" type="40" secondary-reporter="0">
   <narrative>Organisation name</narrative>
   <narrative xml:lang="fr">Nom de l'organisme</narrative>
</reporting-org>
<activities></activities>
{% endhighlight %}

Lastly, Jekyll also has some useful plugins. For example, I've enabled 'jekyll-feed' which automatically creates an Atom (RSS-like) feed at [/feed.xml](/feed.xml). These can be enabled by specifying the version number in the `Gemfile`, and then including the plugin name in `_config.yml`.

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh].

P.S. I might also recommend that anyone writing blogs in VS Code look into the Spell Checker extension. In VS Code, simply hit CTRL+P and then paste in `ext install streetsidesoftware.code-spell-checker`, which will automatically install the extension. Otherwise, it's very easy to accidentally misspell words!

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
