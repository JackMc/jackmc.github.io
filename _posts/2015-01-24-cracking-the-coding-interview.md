---
---
title: Technical Internship Interviews
date: 2016-03-29 23:59:59
layout: post
---
title: Technical Internship Interviews
date: 2016-03-29 23:59:59
layout: post
---

Hello! So recently I've been going through the hoops of getting hired for the
2016 Summer internship season, and I've remembered just how much you can get to
suck at interview problems if you don't practice.

The point of this post is to give a quick introduction and some resources on how
to do at least okay in a coding interview. I won't say I'm the best (I had an
interview recently where I quite literally fumbled when asked basic networking
questions), but I've gone through a decent amount of these things, and seen a
decent amount of questions. Of course, this means everything here is my opinion
(specifically not the opinion of any employer).

_Quick note on phone interviews! This all applies there too._

## Before

Before your interview, your goal should be to cement your existing knowledge
unless you have months and months of notice to develop new skills. You caught
their eye based on who you are, so make you the best you you can be.

This means review your data structures, specifically the non-fancy ones.
Maps, lists, linked lists, sets (maybe not even that last one).
Review your algorithms, sorts specifically (quicksort is such
a common interview question and such a common programmer pain
point).

## Impressions

Next, let's talk about the first part of an interview: first impressions.
The typical advice regarding first impressions is to shake hands, make eye contact,
etc. This is important, but there's some even more basic stuff that coder types
don't do (and I didn't do until recently). Ensure that you look nice, smell nice,
and have a smile on your face as you go in the door. This is important,
and sets the tone for the rest of the interview. Your first sentence should make you
seem excited about the company and about being there. I always start with a "Hi!"
(note the exclamation marks!) and move on to talking with the interviewer with the
typical small talk as we move to the location of the interview. Typically, since
you're both technical people, this will turn technical pretty fast. You'll end
up talking about your past jobs or where you're working now, school, etc. Go with
this!

## The Meat: Technical Problems

Technical problems are the thing that's the most talked about with interviews,
and also are why the company has brought you into their office in the first
place (mostly). So let's work through one (note that this is a pretty well
known problem, with a twist at the end):

**Given a list of size N-1 containing the numbers 1...N inclusive except for two
numbers, find the missing numbers.**

Depending on how much maths training you have, the answer may jump out on
you right away. But that's not the point. This is a twist on a fairly common
problem.

**Step 1**: Think. Your interviewer is not gonna think less of you because you are
thinking a little bit about how to do the question. A good pause time is
probably around 15 seconds, so they know you're thinking but it's not awkward!

**Step 2**: Ask questions. After your thinking above, you're going
to undoubtably have some questions. How is the input specified?
What function signature are they looking for? The interviewer can't
tell you're thinking of this simple thing until you actually ask
them or mention it.

So, in this case, I would ask the interviewer if the input is an
array. This may seem pretty obvious, but with a set this would be
much easier. Assume the interviewer says that yes, the input is
an array.

I would also ask what the function signature should be, giving an
example. In Java, the following seems reasonable:

{% highlight java %}
// Returns the two missing numbers from "numbers", in an array.
public int[] twoMissingNumbers(int numbers[]);
{% endhighlight %}

**Step 3**: Brute force. Now that you've thought a little bit, you probably can see a brute force
solution, and that should be the first thing you write out in your document
or on the whiteboard. I'm going to use Java, as it's a really well
understood language.

So what does this problem actually want us to do? We want to find
the two missing numbers. How can we do this in the easiest and most
intuitive way? Well we can go through the lists and find one which
isn't in the other list. Let's code that up.

{% highlight java %}
public int[] twoMissingNumbers(int numbers[]) {
    // Since there are two missing numbers, we know that the
    // length of the array plus 2 is the number.
    int n = numbers.length + 2;
    // The list of numbers to return, -1 indicating no value.
    int[] results = {-1, -1};
    // The current number we are on (starts with 0, could be 1)
    int currentNumber = 0;

    // For each number, we test if it is in the array.
    for (int i = 1; i <= n; i++) {
        boolean found = false;
        for (int j = 0; i < n - 2; j++) {
            // in this statement, i is the number we are looking
            // for and numbers[j] is the number we are testing in
            // the list
            if (numbers[j] == i) {
                found = true;
                break;
            }
        }

        if (!found) {
            results[currentNumber++] = i;
        }
    }
}
{% endhighlight %}

**Step 4**: Damage report. Your solution right now is likely to be
pretty inefficient. Time to communicate to your interviewer that
you think you have a correct brute force solution and want to move
on to optimizing. First you should probably figure out exactly
how inefficient your code is and where so you don't spend time
analyzing the unimportant bits.

In the code above, the hard part for the computer is the two loops.
This is because the computer must do n computations, followed by n-2.
This means that the code is O(n(n-2)) = O(n*n) because of the
double loops.

**Step 5**: Look for common inefficiencies and develop an optimized solution. What are you doing
every time in the loop that you could do once and make it faster
for sufficiently large input? Look for unnecessary repeated work.
This is the killer in most questions.

Another common improvement is to use a data structure that is
slightly more sophisticated than a list.

In this case, what is our most expensive operation? We check every
single time if the element is in the array. Is there a data
structure which allows for checking if something is in a list? Yes.
We can use a set for this. This is "precomputing" in addition to
using a data structure so it's a pretty nice example. Let's see
how this looks in practice (note I assume a couple imports here,
that's totally cool for you to do on an interview):

{% highlight java %}
public int[] twoMissingNumbers(int numbers[]) {
    Set<Integer> numbersSet = new HashSet<>();
    // Since there are two missing numbers, we know that the
    // length of the array plus 2 is the number.
    int n = numbers.length + 2;
    // The list of numbers to return, -1 indicating no value.
    int[] results = {-1, -1};
    // The current number we are on (starts with 0, could be 1)
    int currentNumber = 0;

    // Hash the integers so we can do O(1) lookup on them
    for (int i : numbers) {
        numbersSet.add(i);
    }

    // For each number, we test if it is in the Set.
    for (int i = 1; i <= n; i++) {
        if (!numbersSet.contains(i)) {
            results[currentNumber++] = i;
        }
    }
}
{% endhighlight %}

By the properties of HashSets we know that generally (unless you
give it a _very_ bad, specially selected data set) have O(1)
expected insertion and O(1) expected "contains". This means that
we have brought down our nested O(n*n) loop into one O(n) (expected) loop.
As a tradeoff, we add the step of adding all of our integers to the
set. This takes (expected) O(n) time.

Therefore, we have found an O(n) solution. This is in fact the
optimal available (at least that I can think of) asymptotic
performance here. There is a solution that is faster non-asymptotically and doesn't
depend on the Set precompute, which involves getting the sum and
the product of every element in the array. We can then do some
fancy math with the system of equations presented to find the
missing numbers. This is left as an exercise to the reader 😀.

## Questions

You will probably be given a chance to ask questions about the
company. Even if you had all your questions answered previously,
put in the 15 minutes to research the technology and culture of the company.

I like to use the following formula for questions. I ask one
technical question and one more humanizing question. The first
question generally follows the format of "I notice you use
technology X. In my experience, this has a couple pitfalls like Y,
Z, and A. What was your technical reason for using X instead of
an alternative like B?". This will require a bit of research on
your part, like ensuring that you know one or two technologies that
they use.

For the second question, I attempt to humanize myself by asking
something about culture or how their team works. This is really
effective in letting you know something about the company and
to convince them that you are a good fit for their company.

## Final Thoughts

Lastly, I want to say that technical interviews should be exciting
if really really scary. This means that you can totally be able to
come out of an interview curious and excited to hear back. It doesn't
always have to be this nerve wracking experience you want to have
over right away. If there's one piece of advice I could give it's
to treat your interviewer like a human who _wants to hire you_!
It's not like they wasted all this money on bringing you in for no
reason. And it costs money. Think about the hour of the engineer's
time you took, plus HR, plus costs of bringing you there if they
paid for that, what the engineer could have been working on could have been working on, stuff like that,
and you'll see exactly how much interviewing/hiring costs a
company.

## Resources

Here are some resources that can help you prepare for technical
interviews (in order of how much I like them).

1. Cracking the Coding Interview (Book)/careercup.com - interview problems and interview processes at several large companies.
1. Maksim Noy's [programming interview problems](http://jackmc.github.io/public/interviews.html)
1. [Hackerrank](https://www.hackerrank.com)
1. [Leetcode](https://leetcode.com)
1. TopCoder
1. [InterviewCake](https://www.interviewcake.com/)
