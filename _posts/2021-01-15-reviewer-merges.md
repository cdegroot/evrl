---
layout: post
title:  "Less handoffs: reviewer merges"
date:   2021-01-15 01:02:03
description: A little writeup on why I think that having the PR reviewer merge/deploy is a good thing.
categories: agile
---
When developing software in teams, there are always seemingly conflicting
forces at work. You want to move as quickly as possible, but to do that
in the long term, you need to make sure that the full team knows what is
going on. A lot of people accept that this tiny slow down in the short
term has a huge payoff in the long term, and one of the practices that
seems to have become entrenched is the practice of code reviews.

When I started, code reviews were rarely done, or at best very informally
done. You committed your work and went on to the next item on the list,
and sometimes, while making changes, you happened to see code that
was written in a way to make you angry, sad or very happy and you gave
feedback. Since then, through a lot of intermediate steps, most teams
seem to have settled on what is now the de facto standard workflow on
Github: you branch, create a pull request, and before the pull request
is merged back into the main branch a colleague reviews.

There's some divergence here, though. Often, "a colleague reviews" is
translated as "a senior colleague reviews" which I don't think is a good
thing for various reasons: I believe in teams that are flexible and where
everybody can do anything that the team needs to be done to at least a
certain extent. The best way of teaching junior members how to do code
reviews is to let them do it. In a similar vein, "a colleague reviews"
often means "a colleague does the whole review and the rest of the team
does a drive-by review", essentially having the whole team pile on. It's
not nice to the coder, it is inefficient, and it breaks my rule where
any two people in a team should feel empowered to make decisions. The
rule is there to make sure that when there is work to be done, any two
people can jump in and you don't need to wait for "the right person"
(read: the lead developer or the manager) to be available. Tickets
stalling in the pipeline is enemy number one of the state of flow
you are hopefully trying to achive.

So, let's establish that "a colleague reviews" means: one, and only one, 
team member reviews and decides whether the code meets the team's definition
of done, whatever it may be. A reviewed PR is a PR that is up to par and
can be pushed out to production (I am assuming continuous delivery here, 
which really should be your default in 2021). What happens next might surprise 
you.... 

The reviewer slaps a thumbs up on the pull request, moves the ticket to 
the "Ready to Deploy" lane, and goes back to their own work. 

What? We have a ticket stalled. Why? Because the "original developer" should 
do the merge and pamper it through the deploy pipeline? I've seen this behaviour
regularly, and it seems to be the default until I bring it up, and I cannot 
understand why. Let's pull the virtual andon cord and discuss this. 

Why is the ticket stalled? There is a double hand-off: first, it stalled waiting
for a reviewer. That is a necessary wait, because we agreed to do reviews and
this is the price we pay for it. Then, there is a second hand-off: it stalls 
waiting for the original coder complete whatever is currently in progress, and
then go back to the "old ticket" that is waiting to be merged. This hand-off
does not add anything to the team, it is just a delay, so purely by the rules
of kanban/lean/whatyoucallit, it needs to be removed. Removing the hand-off
means that the reviewer gets the ticket to "Done."

And why would the original developer need to merge in the first place? Is
the reviewer unsure that the review was thorough enough? Is it to shift
blame when deployment breaks something on whoever write the code and
not share it between coder and reviewer? My stance has been for quite
some time now that if you, as a reviewer, do not feel completely up
to speed with what the ticket was about, how the coder implemented it,
and what the potential risks of a merge/deploy are, your review has not
finished yet. It is not just a check to see whether the curly braces are
in the right position - a review is a full share of knowledge of, and
responsibility for the work. It is essentially some form of asynchronous
pair programming. 

As such, a reviewer should also feel empowered to make
small fixes, remove that extra whitespace, and so on. No need to do
a hand-off back - just make sure that the coder gets feedback at some
point but please write that test you feel should be added. A hand-back
should be an exception, a flag to the team that something went not according
to plan, and probably a item worth of debate with the whole team. If reviewers
constantly need to ask questions then the definition of Done is not robust
enough to deal with this virtual hand-off, all the knowledge needed is not
embodied in code and docs, and therefore the team should think on how to 
improve. That is the exception, not the rule.

_Therefore_, the reviewer merges and, if necessary, checks that deployment
to production went fine. No need for another context switch with the
original coder, and a good way to ensure that knowledge is actually
getting shared: you're not just the person that tells the coder that a
trailing comma is missing, you're part of the pair that got the ticket
from conception to completion. Feel proud to be able to move that card
to "Done". High five. Next task.
