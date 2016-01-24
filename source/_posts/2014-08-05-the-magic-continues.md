---
layout: post
title: Day 2 - The Magic Continues
date: 2014-08-05 11:15
---
Today I attended 'Project Night' at [Ottawa Ruby](ottawaruby.ca). This was my second
Experience in a 'meetup' or programmer gathering environment, and both times it was
**awesome**! I met people working on a variety of projects. One of these was
an interface to a new Google API or service that I don't understand (I hope to talk
to the author again at the next meetup to try and understand \:)) called
[Diatropikon](https://github.com/orospakr/diatropikon). Another project I
saw was from a friend, an application called [Gradster](https://github.com/isyed867/Gradster)
he was making to keep track of grades with a web interface written in Rails. Some of the
things people are capable of doing with software are crazy. The creativity of other
programmers never ceases to amaze me. 

Anyway, at Project Night I worked on Syntag, which is a Rails app I'm writing to
try and crowdsource code snippets. I learned about how easy it is to make JSON APIs
in Rails using the respond_to helper function. Here is an example of one of my
controllers before and after adding JSON API capabilities.

{% highlight ruby %}

### Before
def show
    @snippet = Snippet.find(params[:id])
end

### After
def show
    @snippet = Snippet.find(params[:id])

    respond_to do |format|
        format.html
        format.json { render json: @snippet }
    end
end
{% endhighlight %}

This blows my mind. Now, with this modification to all my controller methods (no
extra code/cost), I can do a request like /snippets/0.json and it will give me
the snippet with the ID 1, or /snippets.json to get all the snippets! :o

That's all for today. I know I forgot a couple days (I was watching TV and being
lazy :p), but I am back on the straight and narrow now!

You can see the progress on Syntag at [It's GitHub repository](https://github.com/JackMc/Syntag).
