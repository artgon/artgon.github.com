--- 
layout: post
title: New Design and Jekyll Migration
meta-description: Relaunching my site with a new design and a new host. This post details the migration from Wordpress to Jekyll, on Github Pages.
---

Welcome to the new iteration of my site. I finally spent some 
time thinking about and designing the new version of the site. Unlike the previous
layout, I can honestly say I'm proud of how it looks and I'm excited to write articles on it. The design is inspired by a template from [John O'Nolan](http://john.onolan.org/), but I changed the colors, fonts and most of the flow. Looks, aside,  a lot of the excitement also comes from the migration of the blog to Github Pages, from Wordpress.

Wordpress is a fantastic platform for blogging and has a huge community for support and plugins. As much as I enjoyed the ease of use, I really wanted more control. I wanted a site that's more nicely suited to technical blogging and ideally, one where I don't have to pay for hosting. An option I considered was using Blogger (via Google Apps), but there is a lack of control there, and it's a massive pain to get code highlighting working correctly -- ditto for Tumblr. I also wanted a space where I could host some images, CSS and JS with my site. Luckily the solution that fit all the criteria was [Github Pages](http://pages.github.com "Github Pages").

<!--more-->

Github Pages is a utility that Github provides for making sites to represent (and ideally document) a repository. However, one of the nice features provided is the [Jekyll](http://jekyllrb.com "Jekyll") framework for building the site. Jekyll is a file-based CMS, where code written is used to generate static HTML pages. Although the pages are not dynamic, you can use [Liquid](http://liquidmarkup.org/ "Liquid") for templating and [Markdown](http://daringfireball.net/projects/markdown/) for simple text to HTML conversion. The templating frameworks are applied once, when the site is built, and the end result is the static HTML. 

This turns out to be really convenient, because now I write all my blog posts in vim, preview them on my local Jekyll setup, commit them using git, and they magically show up on the site, courtesy of Github. Furthermore, since all my posts are files, I can use all the handy file-based tools, such as *find*, *grep*, *less*, and so on. Best of all, I won't have to pay for hosting fees anymore.

Enough about that, the really interesting part is how to do the migration. Fair warning: if you don't like long blog posts, now would be the time to stop reading... :)

#### Step 1 - Creating and Deploying a Jekyll Layout

As with any framework the first step is to install the framework and create a new project. This is fairly easy if you already use Ruby.

{% highlight bash %}
$ gem install jekyll
{% endhighlight %}

Also you'll want to install [RDiscount](http://github.com/rtomayko/rdiscount/tree/master) for Markdown support and [Pygments](http://pygments.org/) for code highlighting.

{% highlight bash %}
$ gem install rdiscount
{% endhighlight %}
{% highlight bash %}
$ sudo port install python25 py25-pygments (OS X)
$ sudo apt-get install python-pygments (Debian/Ubuntu)
{% endhighlight %}

The next step is create the proper directory structure and to configure it the way you want. The documentation can be found [here](https://github.com/mojombo/jekyll/wiki/Usage).

For an example of what my code looks like, feel free to visit [artgon.github.com](http://github.com/artgon/artgon.github.com).

Now all you have to do is commit the code to a repository labeled *&lt;username&gt;.github.com* and your code will show up at that URL.

#### Step 2 - Exporting Wordpress Posts to Markdown

If you already have an existing Wordpress blog, you'll want to migrate that content over. Github Pages provides some scripts that let you do this. 

You can find the scripts [here](https://github.com/mojombo/jekyll/wiki/Blog-Migrations), but it's a little tricky to get it all set up. The way I did it was the as follows:

1. Create an *_import* directory
2. Export your Wordpress blog to an XML file using Wordpress' *Export* feature and place the file in the *_import* directory, with the name *wordpress.xml*
3. From the scripts link above, copy *cvs.rb* and *wordpressdotcom.rb* to *_import*
4. Run the following command

{% highlight bash %}
$ ruby -r 'wordpressdotcom.rb' -e 'Jekyll::WordpressDotCom.process("wordpress.xml")'
{% endhighlight %}

When finished, you should potentially have *_posts*, *_pages* and *_attachments* directories in *_import*. If, like me, you used a funky plugin for code highlighting in Wordpress, you'll have to do a search-replace through your posts to use Jekyll's Liquid extension for code highlighting.

More info on code highlighting and other extensions can be found [here](https://github.com/mojombo/jekyll/wiki/Liquid-Extensions).

#### Step 3 - Tweaks and Plugins

Once you've got all your posts in place, it's time to start tinkering! 

##### Atom XML Feed

You'll probably want to syndicate your blog, here's how I did my [Atom feed](https://github.com/artgon/artgon.github.com/blob/master/atom.xml). It's based on this [Gist](https://gist.github.com/1237173) by [Paul Stamatiou](http://paulstamatiou.com/). Once done, you'll want to place this file in your root directory.

##### Sitemap XML File

The sitemap is a little trickier to do. At deploy time, you need to walk all the pages of your site and create an XML file representing the links. This can be done using a Ruby plugin in the *_plugins* directory. However, when running Jekyll in Github Pages you are not allowed to use plugins, as they don't want arbitrary Ruby code being executed server-side. Unfortunately, the solution I've come up with is running the plugin locally and then copying the generated file to my root directory, before I check it in. 

The plugin itself can be found [here](https://github.com/recurser/jekyll-plugins/blob/master/generate_sitemap.rb), written by [Dave Perret](http://recursive-design.com/about/).

##### Post Previews

If you've worked with Wordpress, you're familiar with the "more" tag. It's a little html snippet you can put in so that your index page only shows a preview of your post and not the whole thing. In order to support this functionality, I had to get a little creative with the Liquid templating engine. 

You can see the result [here](https://gist.github.com/1361497). Essentially, before printing the content, it iterates through the paragraphs of a post until it finds the "more" keyword. It then sets it to a variable and when actually printing the content it stops at that variable. 

##### Archive Page

Another feature that I found useful in Wordpress was the ability to view an archive with all my old posts, easily accessible. In order to create something similar, I forked the code from a [blog post](http://mikerowecode.com/2010/08/jekyll_archives_grouped_by_year.html) by Michael Rowe, and added a few tweaks for my layout. You can see it in all its glory [here](/archive/), or by clicking the Archive link in the header. 

You can find the code in my [Github repo](https://github.com/artgon/artgon.github.com/blob/master/archive/index.html).

#### Step 4 - Using a Custom Domain

This step is probably at once the most simple but most useful. It's important to have control of the name of your site and to have your domain decoupled from any given service. If I were to sign up for a *wordpress.com* blog, they charge you for using your own domain. The biggest advantage Github Pages has is that you can use your own custom domain if you want.

It's fairly simple. Just add a *CNAME* file into your root directory, with the name of your domain.

{% highlight bash %}
arthur.gonigberg.com
{% endhighlight %}

And then modify the CNAME record in your domain's DNS settings, to point to your Github URL. For example, in my Dreamhost panel, I had to set my CNAME record to point at *artgon.github.com*. It may take up to 48 hours or so, but once the DNS is updated, it works like a charm.

#### Final Thoughts

As you can see this type of blogging setup is really not for laymen. However, being a developer, a tinkerer and at one with revision control (<3 git), I feel right at home.
