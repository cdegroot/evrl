---
layout: post
title:  "The Programming Language Conundrum"
date:   2019-03-28 01:02:03
categories: programming
---
A lot has been written about "hyperproductive teams", and a lot
has been written how programming languages help here. [Beating the
Averages](http://www.paulgraham.com/avg.html) is probably the most famous
one. I myself worked in Smalltalk full-time, and can share the same kind
of story. So why aren't we all using either of these two languages?

When I created my own startup, an Internet hosting company, I went
down the same path as Paul Graham: two coders, me and a full time one,
a VisualWorks Smalltalk license, and we managed to develop something
that was utterly impressive given that we had 1.5FTE on the job. Hosting
provisioning, accounting, all sorts of really advanced stuff around
whole-sale packaging, multiple currencies, and so on. Later on, I worked
at a larger Smalltalk shop and between 5-6 developers we completely
beat large companies employing ten times as many people. So why aren't
we all using either of these two languages?

I can recite many of such stories; of people keeping up with their ten
person team by coding whatever the team did during the day in 2-3 hours
at night using Squeak Smalltalk; about an old Lisp hacker working in a
research setup with mostly Java colleagues and always estimating to need
10% of the time (and then actually making the estimate). So why aren't
we all using either of these two languages?

The answer is multifaceted, of course. There's a fear, often more with
management than actual techies, of choosing a non mainstream language
because hiring for that language will be too hard (wrong). Tooling for
niche languages often is not as good as for mainstream stuff (quick, find
a way to deploy a Pharo Smalltalk image on EC2 or in a container). The
list goes on and on and usually every point can receive the same reaction:
yes, that might be true, but if the payoff is that big, why not make
the investment?

However, there's one thing in common between all the stories I've heard
and the successes experienced first hand: small teams.  Every single time,
you are looking at a software development shop with a grand total of,
say, less than a dozen developers in a pretty stable setup.

And that, I think, is the only way to use these very powerful languages
(and do not forget: they _are_ that powerful). Because, each in its own
way, these languages encourage you to solve your problem in two steps:

1. Create a language to solve your problem;
2. Solve your problem.

Lisp is more explicit about this, but I opened an old Smalltalk image
and the "application" language ended up being very high level, very
domain specific, and just as hard to grok as your average macro-ridden
Lisp system.

These languages are very powerful, but they don't scale. So even when
you get ten times the productivity, the mostly tribal complexity will
not scale beyond your "pizza-sized team" and therefore you now have a
hard upper limit on your total software development capacity.

Harsh, isn't it? But I cannot find counterexamples. Whereas I can find
plenty of examples of teams using "normal" programming languages (Java,
PHP, Python, Ruby, or newer entrants like Elixir or even Golang) that
breeze through the 100, 200, 500 developer marks without much trouble. So
I must conclude that there seems to be a scaling limit.

Why? My theory, completely made up and untested, is that these powerful
Lisp/Smalltalk systems weave code and brains together in a way that
amplifies both. It's very organic, very hard to document, and thus it
becomes very hard to absorb new brains into the system. When I joined the
small Smalltalk group it was very hard to get started in that codebase. It
was complex, and completely different from the previous thing I worked
on. Both were using the same VM (VisualWorks) but both had their IDEs
adapted to a point that they became new and very well-honed tools and
new things in their own right. In Smalltalk, there's no difference
between writing "business" code and writing "tooling" code, and even no
difference between that and changing your IDE, so everything happens
interchangeably and fluidly and you end up with an IDE that's adapted
for the problem at hand, not for everybody's generic problems. It's hard
to ramp-up in these environments and certainly in the light of the short
retention times these days, likely won't work.

And on that terrible disappointment... I think I'll end while hoping
that someone out there on planet Earth reads this and proves me wrong :)
