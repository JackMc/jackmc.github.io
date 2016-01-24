---
layout: post
title: Day 5 - Embeddable Code Editors
date: 2014-08-08 23:59:59
---
So today wasn't very exciting in terms of web development learning, but it took
up two hours. This, as it seems, is a common problem in programming. You can
spend hours and hours looking for soutions to a single problem, and there
probably exist enough solutions for you to spend hours and hours looking through
the various solutions to the problem. Generally, I'll take the "good enough"
solution, but it seems for the problem I was having there really wasn't one.

So, my problem was finding a code editor that I could embed into my form for
adding a new snippet to Syntag. There exist two major solutions to this in
the current landscape, as far as I can tell: CodeMirror and ACE. Neither of
which seem to be particularly architected towards people who want to use
them in forms or want to dynamically change the mode.

CodeMirror seems to be better built for inclusion in forms (it has a JS function
called fromTextArea which does exactly what it says on the tin), and ACE seems
to be better at dynamically loading languages from a string. I cannot confirm
this for ACE because I simply couldn't get it working. CodeMirror I got working.
Below is the (small amount) of code I had to add to my CoffeeScript file for
CodeMirror to replace my text area:

{% highlight coffeescript %}
editor = CodeMirror.fromTextArea($("#snippet_contents")[0], {value: "testfunction\n", mode: "ruby"}))
{% endhighlight %}

Currently I don't have it working for syntax highlighting for anything other
than Ruby. I am going to look into dynamically loading modes for CodeMirror.

In addition, I wanted to make CodeMirror dynamically expand depending on the
form content. So I added this to my CSS:

{% highlight css %}
.CodeMirror {
    height: auto;
}

.CodeMirror-scroll {
    overflow-x: auto;
    overflow-y: hidden;
}
{% endhighlight %}

So here's what the code editor ends up looking on Syntag:

![Syntag With Code Editor](/images/syntag_new_snippet-2014-08-08.png)
