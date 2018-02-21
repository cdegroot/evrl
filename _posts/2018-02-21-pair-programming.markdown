---
layout: post
title:  "Pair Programming"
date:   2018-01-25 10:24
categories: programming agile
---
One of the mantras of agile is that Pair Programming is universally
good; some teams even extended this into "mob programming" where the
whole team shares a keyboard. After practicing this on and off for
two decades, I'm not too convinced of its general applicability or 
advantageousness. Note that this will be a longread ;-)

Pair programming, as a formal discipline, was probably first described
in '98 or so by [Beck](https://en.wikipedia.org/wiki/Kent_Beck) in 
"Extreme Programming Explained". The book has a publication date of '99
but I still remember getting my hands on a PDF preview, printing a copy
for all our colleagues at the small shop I worked at, and us rapidly
adopting it - with some success, I must say. Some bits that were lacking
were later, much later, provided by Scrum and then Kanban. A really
good book that bridges Scrum and XP, incidentally, is [Scrum and XP from the trenches](https://www.infoq.com/minibooks/scrum-xp-from-the-trenches-2)
by [Kniberg](https://www.crisp.se/konsulter/henrik-kniberg).

XP consisted of a set of practices, and later on diagrams appeared
on the corresponding website to show how the practices related to 
each other:

![XP Development Practices](http://www.extremeprogramming.org/map/images/development.GIF)

You'll notice that pairing is "just" one of a collection of disciplines
that make Collective Code Ownership possible by learning and communicating
a lot. The importance of the arrow is not really up for discussion, of 
course, the practices are listed as a set of things that can make learning
happen.

After the book, a whole cottage industry sprung up around it where all the
different aspects of XP were explored. XP was developed during the 
"Chrysler Comprehensive Compensation Project", and there was a lot 
of detail and subtlety lacking in Beck's initial description, which 
prompted him to write more books and many, many others to write and
reflect and discuss and write more... This was the time that blogging started
happening and the blogs were overflowing with meticulously detailed 
explanations of every aspect and practice of XP, often losing out of
sight what place a practice took in XP and where it was developed. 

Pair Programming became a mantra, and what people conveniently forgot
was how careful things were built up during the C3 project to make sure
that all the various practices could thrive. I couldn't dig up the image (so
I'm reciting this from memory) but the room was described in word and
picture as having small individual workspots against the wall, and shared
pairing workstations around a large central table.

Shared Pairing Workstations

Let that sink in for a while. And think about the implications of having
an Actual Room (not an open plan office) with all of the team in the
room (not half of them remote or working from home) and _specific
workstations to pair on_. So you don't need to type on your colleague's
"interesting" configuration of Emacs which uses Wordstar keybindings 
because that was such a nice editor way back. 

XP was developed as a set of practices that were reinforced and guided
by its environment - they had hugely complex business requirements, 
and a setup where all the team was in the same room, and that shaped 
their thinking of how they should tackle the project. Beck (and friends, 
I'm sure) invented a snappy title, published some books on it which 
maybe should have been marketed more as experience reports than as
"and this is how thou shalt henceforth develop thy software", and
the rest is history. 

Enough context. Let's start with some pair programming bashing. 

## By the numbers

One of the claims made about pair programming is that it is not
more expensive than individual programming, but scant proof is
ever delivered. Of course, asserting stuff but not doing the research
is the main thing that guides our fashion-driven industry, and I'm 
guilty of the same, but sometimes it's nice to at least have _some_
research backing up claims. I went looking for indications that 
pair programming was actually cheaper, and came up empty-handed. All 
that I _did_ come up with, after an email exchange with [Capers
Jones](https://en.wikipedia.org/wiki/Capers_Jones) was a very hefty
file full of counter evidence; Mr. Jones even provided me with 
a simple spreadsheet that let you do some cost calculations. The
spreadsheet is what convinced me that the cost argument doesn't
hold. 

Why? Without going into details - you can do the math yourself - the
premise of pair programming is that bugs are expensive to fix, that
by having more eyes on the code you will have less bugs, and that
the lower cost of bugfixing will make the higher up-front investment
go away. Superficially, that all seems to make sense.

There are a couple of things that don't fly here. The obvious thing, 
to me, is that you double the cost of fixing bugs (you're pairing
all the time, of course). So you need to slash the bugfixing work
by half if you just want to break even on bug fixing time, but 
you've still spend twice the amount of time writing the code, so you
probably need to really trounce the number of bugs by a factor of
ten or so. By that reasoning alone, you need to be very convinved
of the benefits of pair programming to go at it 100% of the time. 

But there's a more important aspect: in the diagram above, 
pair programming is listed as one of four practices that promotes
shared code ownership. I assert that XP - and even if you don't
XP, you do a lot of XP these days - is literally plastered with 
good practices that promote lower defect rates, and this was
all before we had nice code analysis tools, continuous integration
systems, etcetera. In the '90s, there was a large cost of software
distribution to be factored in but for most companies in the SaaS/Web
field these days, that cost is close to zero. If I have a bug in production, 
I fix it, hit merge, and five minutes later it is deployed. The
argument that bugs are expensive to fix simply doesn't hold anymore, 
which removes one of the major reasons for coping with the large
up-front cost of pair programming in the first place.

My hypothesis is that all these practices together already lower defect
rates and repair cost to such an extent that doubling up will _never_
be able to recoup the investment. Defect rates are simply too low and
caught too early, fixed too cheaply.

Economically, Pair Programming does not make sense in the modern
day of SaaS/Webapp development.

## By the practices

The XP practice map goes deeper to explain how daily coding works:

![XP Collective Code Ownership](http://www.extremeprogramming.org/map/images/code.GIF)

Pair programming now takes a central position, which seems to 
invalidate my take on that pairing is just one of the practices that
promotes collective code ownership and learning. But remember, these
people were sitting in a single room _and_ were very disciplined about
their pairing practice. Over the years, I practiced "good pairing" a 
lot and it is great fun - I'm not saying it makes economic sense, but
it certainly is a fun way to work. It's very intense as well, and hard
to keep up for long.

It goes something like this:

* We walk to a shared workstation, either specifically setup, or a
new user login with a simple IDE (and let's hope that you both agree
on Qwerty vs Dvorak). In Smalltalk, this was easy as the IDE and the
runtime are pretty much inseparable and teams would heavily modify the
IDE during a project but this would be run through the same mechanism as
"production" code; everybody therefore ran the exact same IDE. Not having
to hunt for keyboard shortcuts and menu locations is hugely important
when you're trying to write code;
* We look at the task at hand, and probably break it down in little steps;
* The first little step gets implemented as a test by (say) me - the
test fails, of course;
* I smile and hand the keyboard to you, and you go to work on fixing the
test, thinking out loud; I reflect on what we're doing. One person is 
thinking tactically ("are we doing the thing right?"), the other person more
strategically ("are we doing the right thing?");
* You fix the test, we decide the code is ok, nothing to refactor, so you
write the next test, when it shows red, bounce the keyboard back to me
and I start writing code;
* Ad Infinitum. Usually I'd like to couple this with a [Pomodoro](https://francescocirillo.com/pages/pomodoro-technique) (25 minute
code time, 5 minute break) work rhythm with longer breaks every two or three
pomodori. This is an intense way of working and regular breaks help.

The requirements here are: we have a common shared IDE, we have the
same worktimes, and we can bounce the keyboard between us. From my
experience, these are the only circumstancese where pairing as a coding practice
makes sense. Furthermore, not just "a pair" needs to have the same 
worktimes, the whole team pretty much has to: you want to typically switch
pairs around lunch and at the start of the day, and if only two or three people
are aligned in terms of work time, then the very important practice of "Move People Around"
gets killed. Two people will pair for days and then be surprised if a third person
looks at the code and goes all ballistic. The method of pairing described here makes
the hardest part of pairing - changing drivership at times - a natural and simple
thing to do and therefore in my experience it's the only way that really works well.

Summarizing:
* If you pair with the same person all day or for days, you're doing it wrong;
* If you pair in my environment or your environment, you're likely doing it wrong unless we use the same IDE;
* If you pair remotely using some screensharings where only one person can effectively drive, you're _certainly_
doing it wrong;
* If you pair without TDD, you're probably doing it wrong, too. 

Note that I'm still not advocating that you _should_ write code in pairs,
just that
if you insist on doing it you need to be mindful about why pairing is
supposed to work: the switching between driver and follower
ensures that people alternate between learning and reflecting, that
design issues are captured early by reflecting, and that you don't fall
into antipatterns like the subject matter expert typing and the junior
or intern just staring at the screen in ever increasing bewilderment,
leaving it up to the assertiveness of the latter person to pull the
emergency brake (hint: the emergency brake will usually not be pulled).

In other words: don't pair on coding, it's too expensive, but if for some reason you
do want to pair please check all the boxes that make it work and make it a fun and
productive, albeit expensive, experience. And do have a good explanation about why
you think spending the extra money is worth it :)

## Other ways to do the same

Given that:
* we don't have shared understandings about what's a good IDE (me Spacemacs, 
you Emacs, she Vim, he VSCode, they Atom, anyone on Sublime here? Hey, there's an IntelliJ
user!); 
* we more and
more want to work remote and please at our own preferred times and in our own preferred
timezones; 

what _else_ can we do to promote the mixing of knowledge and early detection
of problems?

Sometimes, a synchronous meet-up is still nice: I want to explain something, or demonstrate
something, or show someone where I'm stuck in the debugger; open a Hangout or Slack 
screenshare, and go at it. Do note, however, that if you're coding and you're stuck that
[Rubber Duck Debugging](https://rubberduckdebugging.com/) is quite effective and does not
require you to interrupt a co-worker. Interruptions are productivity killers!

A good way to reap most of the benefits and way less costly than pairing is the 
workflow I've used during the last years:

* Break down work into _small_ tickets - a day or so is ideal, three days is pretty much the max;
* Do TDD as much as possible; it'll help you drive the design by forcing you to actually _use_ it
in your tests. In a sense, TDD makes you flip between tactical and strategic thinking all the time
even without pairing;
* Do CI and often push to the CI system so you get feedback about potential environment issues (my
personal practice is to not start a task unless both the CI and my dev env can run all the tests).

So far, that's probably just good advice regardless on how many people sit behind the keyboard. However,
it enables the following:

* Grab work from the top of the To Do list whether you think you're up for the task or not. If not, 
you force yourself to learn, to review documentation, complain about it, get told by your colleagues
to fix it, etcetera. Stepping out of your comfort zone promotes knowledge sharing. Grabbing from the
top of the To Do will mix up tasks between team members, further promoting knowledge sharing;
* Small tasks limit the blast radius of your work. Inevitably, someone will start painting the floor
and end up in the corner. Preventing-by-pairing is too expensive, and frankly, preventing mistakes
is a mistake in itself; you need them to learn. Small tasks gives you a safe learning environment;
* Hand over your work by submitting it for code review. I like to use a "ready for review" and 
"in review" lane. A team member looking for work will pull reviews before new work (according to
the mantra "stop starting and start finishing"), review your code, and assuming it's all good
do the pull request merge and deploy (if not using CD). Only if major issues pop up you'll be pulled
from your work and you should hook up to review the design (I would, frankly, rather not do that
through Github or Gerrit comments alone - they serve as useful annotations but, if possible, do
have a 1:1 chat on the issues before going into a long back-and-forth about a design issue). 

Small tasks enable rotation (the "Move People Around" practice of XP), code review enables
multiple eyes on the code. As you move people around, you don't need more than one extra set
of brains. Adding two, three, four reviewers just burns time with little payback - rotation
guarantees that everybody, in time, will be exposed to everything. 

## When to pair

I'd like to reserve the term "pairing" for the actual practice of XP-style disciplined Pair Programming, 
but that ship has long sailed. There are still circumstances where it is useful to pair up, but it
should be the exception rather than the rule and used for specific reasons, not because of some abstract
idea that "pairing is good". Here are a couple:

* Ramp-up of a new team member. Lots of setup needs to be done, and
that is never documented to the extent that a newcomer can go at it
alone. Also, ramp-up work can be less synthetic if you start collaborating
on real work - the new member will feel part of the team quicker and
get an early sense they're contributing, which is typically not the case
if your first month consists of fixing P4 tickets. Having the first two weeks
of onboarding consist of a lot of planned collaborative work can really speed
up the process. Investment here pays off, so the arguments on cost don't hold. 
* Specific learning. A team member lacks a skill, maybe for the specific ticket
that just got pulled in, and is the only member that has this knowledge gap. Do a 1:1 knowledge
dump session where a more experienced person can guide. However, if more than 
one team member is lacking knowledge or skills, 1:1 sessions don't really scale
well so start with a tech talk, a group session, writing docs, collecting 
materials for self-guided study, etcetera.

People often argue that spikes and other off-the-beaten-path work should
be done in pairs, but I disagree: spikes are tentative, throw-away work,
often involve a lot of reading and puzzling and surfing the web (an
activity which used to be called "desk research" but now is basically just
hanging around in Google), and so on. I've tried that in pairs, but it's
inconvenient (reading speeds differ too much, meaningful collaboration on
finding docs is hard) and by doubling the investment in a spike you're
not necessarily get a better outcome. Let one person go at it for one
or two days and have them report back to the team during stand-up.

## Team bonding

Often, people just want to pair because "team bonding" or similar reasons. Teams, 
depending on their composition, have various needs for specific activities directed
at some form of cohesion, and I think a good scrum master or agile coach should be 
primarily responsible for identifying the average need, working with outliers to 
bring them to the average (or at least convince them to "play along"), and take it
from there. Pairing _may_ be a tool in the toolbox, 
but it's not a panacea and certainly not a tool that should just be deployed
nilly-willy because "team cohesion". Some teams will be sufficiently cohesive
without any work, some just need some ticket shuffling, and some need actual work; my
preference is to let the agile coach figure out what's needed and work with the team
on explicit and tailored practices rather than just waste a lot of the company's 
money on pairing. 

## TL;DR

Pairing has one known downside: doubling the cost of development time
and thus potentially halving a team's velocity. The advertised benefits
usually are based on gut feelings rather than evidence; there is little
research to support pair programming. Furthermore, reasoning from
first principles does not support the advertised benefits like deficit
reduction as the past two decades have seen huge quality improvements
through other means and lower repair costs for defects; therefore, pair programming
now has a steeper hill to climb to make sense. Pairing also does not
mesh well with the current practice of individualized workstation
setups, remote work, and flexible work times. 

Overall, I recommend that development teams don't pair as a general
practice - daily standups, code reviews and mixing up small tasks should
provide ample fertile ground for knowledge sharing. There are various
specific situations where collaborative "synchronous" sessions make sense,
but they should be called out explicitly.
