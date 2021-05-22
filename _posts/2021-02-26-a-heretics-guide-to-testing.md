---
layout: post
title:  "A heretic's guide to testing"
date:   2021-01-15 01:02:03
description: Like most of our practices, testing should not be done in a dogmatic way.
categories: agile, tdd
---
Our industry is largely driven by fads: someone comes up with a great idea, and people then
lift that idea out of context, run with it, and declare it "the true and only way." In this
post, I want to talk about testing and especially about the dogma that tests should be written
first and they always should be there. In other words, about Test Driven Development (TDD)
being the true and only way.

Let's start with what I think our goals should be.

## Quality, accessibility, pick two.

"Quality" is to me mostly captured by the notion of code being fit for purpose. It is contextual,
if the purpose of your code is to impress an interviewer your idea about quality is going to be
different than if you, say, write a system for a country's air traffic control. Your code should
do what it is supposed to do, and do it well.

"Accessibility" is a term I introduce here to capture a bunch of human factors aspects of code that
I feel are often somewhat neglected. To me, the primary interaction with code is the interaction
that people have with it, not the interaction that machines have with it. I write code for humans,
whether it is my collaborators or my later self, and pretty much on equal footing with "it should
be fit for purpose" is "it should be accessible." Code needs to be readable, navigable, and
malleable. The latter aspect cannot be stressed enough: if I cannot go back to my code
and change it in a safe way, knowing that neither quality nor accessibility will be impacted,
it is to me quite useless code.

## TDD, as I understand it

I was taught TDD in Java. I _learned_ TDD in Smalltalk. The idea of writing, compiling, and
running a test that talks to non-existent classes or methods might be strange but is totally
natural in Smalltalk - if you refer to a class that does not exist, you get a polite ask
whether you want to define it, if you refer to a method that does not exist, you get dropped
in the debugger and get a chance to create it while having the actual arguments that resulted
from your test run right there.

It is very interactive and all but guarantees that you will only write code that is covered
by a test. It is so efficient, that the running joke-with-a-truth is that you can recognize
a seasoned Smalltalker by the fact that they spend most of their time in the debugger. In
other languages, the feedback cycle is slower, sometimes much slower, as expensive VMs need
to be started up for every test run. Still, even in these cumbersome languages it is a very nice
way to develop software. Write a test, make it green, commit, rinse, repeat.

It is also limited in scope, and that is up next

## Where TDD breaks down

TDD is extremely good at two things: teaching about unit testing and developing algorithms,
like tricky calculations or somewhat complex data traversal.

The interesting thing is: at least in my industry that is a small part of my job. A lot of
my work consists of talking to APIs. Take in some data, do some very simple logic work, and
send it off again. Sometimes, the logic is so simple, it is what I call "too simple to test."
TDD breaks down here, because it inevitably would take you down the path of integration testing.
The complexity lives in the APIs and your code does, well, not much. A much heard critique
of Smalltalk is "everything happens elsewhere", which addresses the issue that your logic is
often smeared out over lots of collaborating classes; well, we now have the distributed
version of that where your logic is smeared out over lots of collaborating services, partially
owned by your employer, partially owned by third parties.
