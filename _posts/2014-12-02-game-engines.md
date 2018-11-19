---
title: Day 9 - Game Engines Are Hard!
date: 2014-12-02 00:29:00
layout: post
---

So today I restarted work on a project which I've left be for a while: A generic
C++ wrapper around SDL2. It turns out that abstracting all this functionality is
not as easy as I would think, and that it takes a lot of time, effort, and
patience to make it work.

I present to you, the results of one hour's work on this game engine.

<center><img src="/images/plasmas_circleberry.png">Plasmas Game Engine drawing circle</img></center>

Now which one was harder to draw? Actually the circle. Turns out that SDL
doesn't conatin functions for drawing primatives past a rectangle, so I had to
write my own.  Now recalling the unit circle from grade 12 maths (ew, trig) I
wrote this little function which creates a circle out of a set of points given a
particular number of points to draw in the circle:

{% highlight C++ %}
void SDLWindow::DrawEllipse(const Rect &rect)
{
    // This is how often points will be drawn
    const static double granularity = 0.001;
    // We roll our own ellipse...
    double angle = 0;
    int x = rect.x, y = rect.y, w = rect.w, h = rect.h;
    while (angle <= M_PI*2) {
        SDL_RenderDrawPoint(this->write_to, x + (w*cos(angle)), y + (h*sin(angle)));
        angle += granularity;
    }
}
{% endhighlight %}

This taught me a little bit about how maths can have application in graphics. I
think I will continue to work on drawing primatives tomorrow. I will also
probably write more on the abstraction structure

Anyway, back to the point of this post. The main point of this project is to get
familiar with writing large(r) software projects in class-based C++. My goal is
that eventually I will not have to stare at linker errors for a long time before
I figure out what the heck they are talking about.

While working on this and my other C++ project (a robot simulator), I have
definitely found that I have gotten better at both writing compiler- and
linker-error free C++ (marginally, I've not had a class compile first time yet,
but it is getting to the point where it's just small basic fixes), along with
getting better at using GDB to diagnose my problems instead of
printf-debugging. C++ is a challenging language, but I definitely think I'm
getting better.

If you want to see the source and maybe try compiling this code on your system
(currently works only on Linux due to crazy Windows linkage I'm trying to
understand) go [Here](https://github.com/JackMc/CPlasmas). Unfortuantely I have
no idea how the strawberry image used above is licensed so you're gonna have to
take your own image and call it 'one.png' in the build directory.
