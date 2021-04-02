+++
title = "Working with SASS on Bluehost"
date = 2013-03-23

[taxonomies]
tags = ['linux', 'bluehost', 'sass', 'wordpress']
+++

> Note: This page is out of date, and I've gotten reports that Bluehost SSH no longer permits these steps. I've switched off Bluehost for a while now, so I don't have a way to provide updated instructions. Please use this information as a historical reference only!


I’ve recently switched to WordPress since I was spending too much time configuring Drupal and not enough time making content.  I still prefer Drupal for sites that justify a CMS, but for a blog, why not use blogging software.  My biggest complaint with wordpress (other than The Loop) is how obviously WordPress most of the themes look.  I’m under no impressions that I can revolutionize WordPress aesthetics, but I figure I can make an obviously derivative theme on my own.

After a quick search, it seems like the big names in starter themes are Underscores and Bones.  I chose to go forward with Bones because it provides a little more default styling than Underscores.  Rather than styling with CSS, Underscores heavily recommends that you compile your stylesheets from LESS or SASS.  Since I have more familiarity with SASS, I’m going to go down that road.  However, either language should be compiled server side.  This blog is a low traffic learning experience, so I’m going to do all of my theme editing live (or on a testing subdomain) and not hosted locally.  It’s not great practice.

Anyway, lets work with SASS on Bluehost.  Our first step is to get SSH access to the Bluehost server.  You can request SSH access from the Bluehost cPanel. Once we have access to our account over SSH, we’ll need to install some gems.  Fortunately, Bluehost has the RubyGem package manager installed.

```sh
gem install sass
gem install –version ‘~> 0.9′ rb-inotify # provides better performance when polling the filesystem for changed files
```

Now we watch the Bones library/scss folder for changes and automatically compile a new stylesheet:

```sh
cd
/wp-content/themes/
sass –watch library/scss:library/css
```

If we’re successful, we can edit `_base.scss`, change the body background, and watch sass recompile.  Then refresh your demo site and see your changes.  For more information on Sass, check out the tutorial.
