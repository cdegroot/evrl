---
layout: post
title:  "Love thy product owner..."
date:   2018-12-14 20:24
categories: programming
---
People regularly find me quoting a Goethe poem called "Art and Nature".
It is the only Goethe poem I've read
and remembered (except probably during German classes in highschool,
but that's repressed memory), and the bit I quote is at the far end;
I like to quote it because it contains such a cardinal truth in such
a succinct statement that I'm jealous at the power of poetry.

Here's the poem translated by David Luke:

>Nature and Art, they go their separate ways,
>
>It seems; yet all at once they find each other.
>
>Even I no longer am a foe to either;
>
>Both equally attract me nowadays.
>
>Some honest toil's required; then, phase by phase,
>
>When diligence and wit have worked together
>
>To tie us fast to Art with their good tether,
>
>Nature again may set our hearts ablaze.
>
>All culture is like this; the unfettered mind,
>
>The boundless spirit's mere imagination,
>
>For pure perfection's heights will strive in vain.
>
>To achieve great things, we must be self-confined:
>
>Mastery is revealed in limitation
>
>And law alone can set us free again.
>

The last three lines are the interesting ones, and I keep coming
up at situations where they apply, especially at work.

A lot of people, especially those newer to the field, are awed by
the fast pace of development and the constant stream of new toys. And
people, techies first, want to play with these new toys: new languages,
new databases, this whole cloud thing, surely there must be a problem
that's just dying to be solved with a new toy.

However, if you've been around for long enough, you notice patterns and
repeat cycles. After some thirty years of doing paid coding work, there
is the realization that these new toys are invariably pushed by very
skilled sales people and that they never present a silver bullet, and
that the _actual_ hard parts of our work are more mundane and less
prone to change:

* Write really good code, which is made hard because good code wants to
  be simple and business requirements usually don't have that characteristic;
* Work with Peter L Deutsch [Fallacies of Distributed Computing](https://en.wikipedia.org/wiki/Fallacies_of_distributed_computing)
  and know your systems' limitations. Hint: a website is a distributed system,
  so there's a fair chance that you're working on a distributed system
  right now.
* Know design patterns and when and where to apply them (I'm a card-carrying
  [Hillside Group](https://hillside.net/) member because I think that patterns
  are a very powerful tool to distill and share knowledge).

There's probably more to that list, but this is what pops into my mind right
now as things that haven't really changed since I started typing away on my
first paid gig. As the French say, "plus ça change, plus c'est la même chose".
As I progressed from deployment through a 5.25" floppy disk handed out to
my first customer the tools changed, but the business problems I was solving
and the way in which we went about it did not change a lot. And the shinies
that sales people and enthusiasts told me to use sometimes helped, sometimes
not. As I progressed, I started focusing more and more on becoming really
good at these core skills and less on a language, a library, or the newest
way to send messages from A to B reliably (something solved in the '80s with
store and forward style mail delivery systems, by the way).

What does this have to do with "Love thy Product Owner?"

A PO will let you focus. A PO has a list of business problems, ideas, and
whatnot that is good for four years of work for your team, everything must
be done yesterday because a competitor popped up, and can we please stop
talking and start typing?

As the prime representative of the business you're working in, your product
owner will rub your nose in the dirty every day reality of doing business
and will make it clear that playing in the sandbox with new toys is not
something that will make them, or by extension the business, happy.

Your product owner restricts you. They are the "limitation" in Goethe's
poem. Your company's architecture, chosen stack, tools, and production
platform is the law. They work together to allow you to shine, set you
free, and show your mastery. Mastery in solving business problems by
focusing at the domains I listed above, and becoming really good at
them. Not mastery by getting a new tool from the shelf and applying it
because a) it's more fun that way and b) maybe it _will_ actually solve
your problem this time. There's a time and space for that, of course,
but make "innovation" the exception, not the norm.

The norm is good, honest, writing code. The best code you can deliver
and that makes your business happy now and in the future. Sure, learn new
languages, toy with new tech - it's important to see what's out there because
some day, your PO will come with a new idea and one of the things you toyed with
8 months ago seems to fit the bill _exactly_. But it is a side gig, and frankly
mostly an optimization; your real leverage sits in your core skills. Writing
good code is hard, but delivering good code is extremely fulfilling. And it
is more satisfying than chasing new shinies, toying with tools that come and
go over time, and - worst case - being in a development team that has to spend
half its time on maintaining three language stacks, two relational databases,
four messaging systems, and knowledge about half AWS' product catalog just because
your predecessors tried to solve relatively simple business problems by just
constantly hunting for new tools.

Chose a stack, stick with it. Embrace the law it lays down, as it lets you
focus; love thy product owner, because a decent one will put the pressure on
your team that will help you excel.
