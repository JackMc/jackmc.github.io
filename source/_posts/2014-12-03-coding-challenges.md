---
title: Day 10 - Coding Challenge Sites
date: 2014-12-03 23:59:59
layout: post
---

Today I tried out some problems from two different coding challenge sites:
[Project Euler](https://projecteuler.net) and
[Hacker Rank](https://hackerrank.com).

One of the major deficiencies I have that I will admit is that my competitive
coding/coding challenge abilities are not great. I can write a good app or
utility, but with straight programming problems with a lot of algorithmic
thinking I'm not the best. That's what I'm trying to fix with trying some
problems.

So first let's talk about my experiences with Project Euler. Now I'm
embarrassingly early in Project Euler (problem 17). Today I finished this
problem. It involves a lot of string manipulation, and I decided to write it in
Python. The basics of the problem is that you have to write code that will turn
an integer into an English representation of that number (for example 156
becomes one hundred and fifty-six). I'm not going to post the code here so as to
encourage competition and working it out yourself, but the basic idea I used is
to have a couple dictionaries which you can index into to get the string
representation of a set of digits with a one-word representation (for example 10
= ten, 20 = twenty, 2 = two, 13 = thirteen) and then to index into those
dictionaries and do a recursive call to find out the rest of the digits' naming
if necessary.

The second site I tried out was Hacker Rank. This site seems to be significantly
smaller than Euler, but definitely contains some quality problems. The site is
divided into four main "domains", which are sets of programming skills which the
challenges in that domain are themed around. The Linux/bash scripting domain
deserves a special mention even though it isn't the one I ended up picking. I
tried some of the challenges and it teaches constructs of bash scripting and the
standard GNU/Linux toolset (coreutils) that allow you to do some uniquely
powerful stuff with it by piping and chaining the various tools together.

The domain on this site that I ended up choosing was the Artificial Intelligence
domain. I am currently in the first section (sections comprise of quite a few
problems) of this domain. The first few problems were pretty easy, telling you
to guide a little robot through a whole bunch of different tasks such as
cleaning a floor and saving a princess. The way that this teaches AI principles
is pretty clever in that you learn a general idea of how best to do pathfinding
pretty quickly.

The one I'm currently stuck on is the two-player maze solving game. You can find
it [here](https://www.hackerrank.com/challenges/maze-escape). See if you can
beat me to it!
