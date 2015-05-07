---
layout: post
title:  "First Post"
date:   2015-05-07 07:47
---

It took me a few days, but I've got a barebones blog up.  It doesn't get any simpler than [GitHub Pages][gh-pages], but Jekyll didn't come as naturally to me.  A [very helpful blog post][veithen] seemed to clear up how Jekyll expects the SASS files to be laid out and linked together on the filesystem, leading me to get it working today.

I tied my .github.io page to [this domain][alecroy] I won during [Gandi][gandi]'s 15th anniversary.  I really wanted to use an apex domain (*example.com*, not *www.*example.com), which made setup a bit different than most the guides I came across.  Just in case I wake up an amnesiac, let me break it down (for me):

1.  Log into GitHub
1.  Create a repository *your-user-name*.github.io
1.  `$ cd ~/GitHub/`
1.  `$ git clone github.com/your-user-name/your-user-name.github.io`
1.  `$ cd your-user-name.github.io`
1.  `$ echo 'hello, world!' > index.html`
1.  `$ echo 'your-custom-domain.tld' > CNAME`
1.  Go to your DNS provider and update your zone file to point to GitHub's servers.  Trim your zone file down to 3 lines as [demonstrated here][spector]:

    | Name | Type  | Value |
    |:----:|:-----:|:----- |
    | @    | A     | 192.30.252.153 |
    | @    | A     | 192.30.252.154 |
    | www  | CNAME | your-custom-domain.tld. |

Now commit your 'hello, world!' site to GitHub:

1.  `$ git add .`
1.  `$ git commit`
1.  `$ git push`

At this point you should see 'hello, world!' at *your-custom-domain.tld*.  It might take a while for your DNS provider to propagate the changes, so give it a while.  You can leave that up, like I did, while you set up Jekyll:

1.  `$ cd ~/GitHub/your-user-name.github.io`
1.  [Edit your Gemfile][jekyll-ghpages] to contain:

{% highlight ruby %}
source 'https://rubygems.org'

require 'json'
require 'open-uri'
versions = JSON.parse(open('https://pages.github.com/versions.json').read)

gem 'github-pages', versions['github-pages']
{% endhighlight %}

1.  `$ bundle install`
1.  `$ bundle exec jekyll new --force ./`
1.  Edit `_config.yml`, `about.md`, and the posts in `_posts/`.


At this point, you're pretty much done.  The default styling isn't bad (at Jekyll v2.4.0), responding to all sorts of window sizes, so I'll keep that around.  Clearly I'm not a virtuoso designer.  I'll probably change more of the styling later, but for now I'll just adjust the default fonts to have a little bit more character.  I'll use 2 webfonts hosted by Google, Fira Sans & Source Code Pro:

1.  Go to [Google Fonts][goog-fonts], add the fonts to your collection, add all the weights/styles you need, and grab the `<link ..>` under the Use tab.  Add that `<link ..>` to `_includes/head.html` before the other `<link>` elements.
1.  Create `css/style.scss` and fill it with:

{% highlight css %}
---
---

body {
  font-family: 'Fira Sans', sans-serif;
}

code {
  font-family: 'Source Code Pro', monospace;
  font-weight: 400;
}
{% endhighlight %}

Those first two lines with triple-dashes are important.  Commit the changes so far to GitHub:

1.  `$ git add .`
1.  `$ git commit`
1.  `$ git push`

That just about wraps it up.  Happy hacking.

[gh-pages]: https://pages.github.com
[veithen]: https://veithen.github.io/2015/03/26/jekyll-bootstrap.html
[gandi]: https://www.gandi.net
[alecroy]: http://alecroy.me
[spector]: http://spector.io/how-to-set-up-github-pages-with-a-custom-domain-on-gandi/
[goog-fonts]: https://www.google.com/fonts
[jekyll-ghpages]: http://jekyllrb.com/docs/github-pages/
