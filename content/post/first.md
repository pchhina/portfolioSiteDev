+++
date = "2017-06-18T05:38:19-05:00"
draft = false
title = "Welcome to the blog!"

+++

This is the test blog to see where it appears!
Skins are the files responsible for the look and feel of your site. It’s the CSS that controls colors and fonts, it’s the Javascript that determines actions and reactions. It’s also the rules that Hugo uses to transform your content into the HTML that the site will serve to visitors.

You have two ways to create a skin. The simplest way is to create it in the layouts/ directory. If you do, then you don’t have to worry about configuring Hugo to recognize it. The first place that Hugo will look for rules and files is in the layouts/ directory so it will always find the skin.

Your second choice is to create it in a sub-directory of the themes/ directory. If you do, then you must always tell Hugo where to search for the skin. It’s extra work, though, so why bother with it?

The difference between creating a skin in layouts/ and creating it in themes/ is very subtle. A skin in layouts/ can’t be customized without updating the templates and static files that it is built from. A skin created in themes/, on the other hand, can be and that makes it easier for other people to use it.

The rest of this tutorial will call a skin created in the themes/ directory a theme.

Note that you can use this tutorial to create a skin in the layouts/ directory if you wish to. The main difference will be that you won’t need to update the site’s configuration file to use a theme.