---
layout: post
title: Day 7 - Nice Assets
date: 2014-08-10 22:30:00
---
So today I spent most of the day wrestling with the Rails asset pipeline, and
this didn't involve any, to quote myself, "typey-typey where the words are all
nice and glowy," but it is the thing that I learned the most from today, so
it'll take up the bulk of this post.

But first, let's talk quickly about the code I wrote. I changed the CSS around
so that it automatically expands even on the snippet selection page so as to
stop it from having a scroll box without bars. I also fixed a bug where the
JavaScript for the snippet selection screen would actually pass in an ID of a
snippet which the set_language function would then try and interpret as a
language ID. It's a happy accident that this had worked out in my favor so far
because I would add a language and then add an associated snippet and they
would end up having an associated snippet.

So, back to the asset pipeline. It turns out that when the server is in
production on a Heroku Dyno, the only assets you can access which are in the
vendor/ and app/ directories are the generated application.js and
application.css files which add together all of the CSS and JS assets which you
wanted to have included in your pages. The problem arises when you want to
dynamically load these assets. After a large amount of experimentation, I
installed the Heroku rails\_12factor gem and sticking the code into the public
directory. For example, I keep a codemirror directory under an assets\_static
directory in public. This contains all the modes and themes to be dynamically
loaded.

You can see this project up on Heroku for a demo [here](http://syntag-web.herokuapp.com/snippets).
