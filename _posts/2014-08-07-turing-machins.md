---
layout: post
title: Day 4A - Turing Machines
date: 2014-08-07 17:22:00
---
<p align="center">
<img src="http://imgs.xkcd.com/comics/candy_button_paper.png" alt="XKCD comic about Turing Machines"></img>
<span class="image-credits">Image Credit: <a href="http://xkcd.com/205/">Randall Munroe - xkcd</a></span>
</p>

I've always been a practical guy. The reason I got into computers was because I
loved the way I could make whatever the hell I wanted whenever the hell I
wanted. But, today, I'm trying something different. I'm learning about
theoretical computer science. Now fear not, dear 7 readers. For the next three days,
the "day of code" will be split into two parts: theory, and practical. The practical
half will still be approximately two hours of me coding and reporting on what I learned.
The theoretical segment will be pretty much me writing notes to myself 

So, let's talk about Turing Machines. Turing Machines are the simplest theoretical
model of a modern computer that you can devise. The visualization given by Alan
Turing (incidentally one of my heros/role models) in his seminal essay
<ins>Intelligent Machinery</ins> is of a machine (technically a human in the original
paper) with an infinitely large string of segmented tape which can contain any of
a finite set of symbols in each segment. The Turing Machine also has a single register
in which it keeps it's "state", which is part of another set of symbols which pertain
to the operation of the machine when it reads a given symbol from the tape.

The machine operates through a set of rules or instructions which it uses to determine
what to do next. These rules can be defined in a five-tuple, i.e. a set of five pieces
of information:

1. The state of the machine in which this rule applies;
2. The symbol which must be under the head of the machine for this rule to apply;
3. The symbol to write to the segment **before** moving performing the movement operation
   if any;
4. The direction to move after writing the given symbol to the segment;
5. The state to place the machine in after the operation is complete.

These rules are applied after the machine is put into a given "starting state" which consists
of an initial position and an initial state. This continues until the machine reaches
a state for which there is no finite transition (there are either zero or more than one
ways which the machine could transition from it's current state/symbol combination).

This appears to be a fairly good way to think about Turing Machines, but unfortunately,
as I found out today, things aren't ever this simple. My final statement above about
an "initial position" turns out to be rather tricky to specify when you're dealing
with an unbounded, infinitely large tape (how do you say where something is without
something to make a relative comparison to?) For this reason many have suggested
an alternate model for the Turing Machine which says that the tape has an end to the
left but extends infinitely to the right.

Another complication introduced by the Turing Machine's nature is that there has to
exist a state for the machine's segments to be in when they have not been written to.
This is called the empty symbol/state (I like to use "empty symbol" in my head because
it gets confusing with the use of 'state' for describing the contents of the sole
register of the Turing Machine) and can also be written by the Machine.

So that's all, folks. This is the basics of what I learned today, but this blog post
is already too long so I'll cut it off here.

N.B. I just researched and learned this today so don't fault me if it's wrong :-) But
do tell me!

---

References:

- [Turing Machine, Wikipedia](http://en.wikipedia.org/wiki/Turing_machine)
- [Shunichi Toida's wonderful notes](http://www.cs.odu.edu/~toida/nerzic/390teched/tm/definitions.html)
- [Stanford Encyclopedia of Philosophy](http://plato.stanford.edu/entries/turing-machine/)
- [Busy Beaver, Wikipedia](http://en.wikipedia.org/wiki/Busy_beaver)
- [Entscheidungsproblem, Wikipedia](http://en.wikipedia.org/wiki/Entscheidungsproblem)
