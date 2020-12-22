---
layout: post
title:  "Back to the '70s with Serverless"
date:   2020-12-18 01:02:03
description: Serverless, AWS Lambda, and mainframes versus the good old times of personal computing and simple client/servier systems.
categories: devops cloud
---
I've now worked with "The Cloud" for long enough to see that there's still a long way to
go before it becomes materially better than, say, the oldschool method of renting a
couple of servers with a co-location company and running your software there. The
latest fad is Serverless, which makes me feel a lot like we've arrived in 1970.

A long, long time ago, I was pretending to study business administration
while teaching myself to code in C. The university was in two worlds:
we had a lab with "personal computers" but also terminals to the mini
computer and assignments had to be made on either one depending on the
professors' preferences. But both systems were at least "on-line" systems--interactive, with immediate feedback. A friend of mine was not so lucky:
he had an assignment for one of his courses in civil engineering that
had to be completed in Pascal and had to be handed in as a print-out
from the university's mainframe.

He called me in for help, because he never wrote a line of code before. So I hopped
over to his place, a Turbo Pascal floppy disk in hand, and assumed that we would
make short work of the assignment. And we did, even though I never coded in Pascal
before: the IDE, even back then, was marvellous and a quick succession of trial-and-error
got us the results we wanted (they were some simple engineering calculations, much
quicker done on even the most primitive of calculators, but such are course assignments. At
least it was easy for us to verify the program was correct).

We hopped into the car and drove over the the university's computer center, a '70s brutalist
concrete bunker. We sat behind a terminal, typed in the code, surrounded it with the required
IBM Job Control Language that the teaching assistant helpfully added to the assignment, and
hit "Enter" (not "Return", actually "Enter" - Enter The Job). A couple of minutes later, the
printer started and spit out a couple of pages. Between gibberish in all caps, there it was: a
compiler error. Back to square one - Turbo Pascal and IBM Mainframe Pascal clearly had some
differences. A couple of hours and reams of wasted paper later, we had what my friend wanted:
the results of a good run. The numbers matched our notes, and we could retreat to the nearest
bar for a well-earned beer.

Ever since that experience, I have stressed the value of feedback and especially of quick
feedback. Even in this minimal example, we spent more time tweaking the code for the dialect
than we spent on writing it in the first place. Slow feedback loops kill performance, and if
you don't believe me, find an online version of [The Beer Distribution Game](https://en.wikipedia.org/wiki/Beer_distribution_game) and play it. You'll be surprised.

The pinnacle of interactivity was, and still is, Smalltalk. I worked in that language for a
couple of years and having your system compile and run most of your tests in less than a second
is addictive. It is incredible how much your performance increases on a fully interactive
programming system but it takes first-hand experience to fully appreciate it. It's a strange
system, a strange language, a tough sell; therefore Smalltalk is still in a very undeserved
state of limbo.

After my Smalltalk years, I met a lot of large Java systems. They started out as horrible
hairballs, but when the idea of "dependency injection" got a foothold, things started to get
better. Except for the return of the Job Control Language, this time disguised in XML and
carrying friendlier names like "Spring". Internally, the code was reasonably clean and modular,
but telling the computer how to run it took pretty much the same amount of typing, this time
not in a programming language but in a structured markup language. That language lacked all
the facilities that apply to writing good code, so principles like "Don't Repeat Yourself" went
overboard and copy/paste programming ensued. New business logic? New controller, twenty lines of
Java boilerplate, 10 lines of Java business logic, 50 lines of XML to wire it to the rest of
the system.

At least, in hindsight, XML was a blessing: your editor would tell you whether your "this is
how the code is wired up" description had the right structure, and later on even whether you
used names that existed in your Java codebase.

These systems took ages to compile, hours to run all the unit tests, and clearly interactivity
went down the drain. So "split it all up" was not a bad idea. It was an unnecessary idea, mostly
driven by the shortfalls of mainstream programming languages, but necessity is the mother of
invention - even though your average Java business app was much simpler than the Smalltalk
IDE before you added a single line of code to write your app, it was already too complex to
maintain and "divide and conquer" was the answer. Service Oriented Architecture, and later
microservices, was born. Split your codebase, split your teams, create a lot of opportunities
for mediocre coders to grow into mediocre engineering managers, everybody was happy. Including
your hardware vendor, because suddenly you needed much more hardware to run the same workloads. Because
networks are slow, and whereas it can be argued whether a three tier system is actually a distributed
computing system, a microservices system certainly qualifies for that label.

The return of the Job Control Language this time was in the form of,
again, configuration data on how to run your microservice. Microservices
were somewhat fatter than the very fine-grained objects of the old days,
so there was less of it, but still - it was there. The feedback cycle
became worse as well: in the monolith-with-XML days, your XML editor
would get you mostly there and a quick local compile and run would
leave you all but certain that your configuration was working.
XML, however, was universally rejected in favour of things like JSON, Yaml, HCL,
Toml - all free of structure, with zero indication whether a computer
would find your prose gibberish or the next Shakespeare play until
you actually pushed your code to some test cluster. It suddenly felt
a lot like being back at that university interacting with a mainframe,
but at least you still owned the hardware and could introspect all the
way down, especially if you were doing "DevOps" meaning mostly that you
had administrative access to your hardware.

Microservices have trouble scaling, and they are very complex. Most companies that employ them
have no need for them, but the systems and programming languages they employ are sufficiently
lacking that this stacking of complexity on top of complexity becomes a necessity. It seems that,
contrary to what everybody is saying, software developers are in plentiful and cheap supply so
wasting a lot of their talent makes economic sense over the perceived cost of adapting more
powerful programming systems. C'est la vie. The feedback cycle is truly broken - testing a
microservice is merely testing a cog in a machine and no guarantee that the cog will fit the
machine - but we just throw more bodies at the problem because Gartner tells us this is the future.

So, we're now at the next phase of this game: maintaining these very complex systems is
hard and not your core business, so outsource it (simplifying it would cost too many managers'
jobs, so that is not an option). The Cloud was born, first as a marketing label for an old
business model (offering "virtual private servers" to the public), but more and more as a marketing
label for an even older business model (the mainframe - we run it, we own it, you lease capacity).

Apparently, [Worse Is Better](https://www.dreamsongs.com/WorseIsBetter.html) and you can do worse than Virtual Private Servers, so through a short-lived detour through
containerizing microservices and deploying them on a distributed scheduler like Mesos, Nomad,
or Kubernetes, we have arrived at "Serverless". You deploy individual stateless functions. But
not inside a Java monolith, that is old, but on top of a distributed system. You would have been
laughed out of the door if you had proposed that in 2000, and you should be laughed out of the
door right now, but such is the power of marketing. So what have we now? A "mono repo" codebase,
because clearly a Git repository per function in your system would be too much, a large deployment
descriptor per fine-grained component, which Spring maybe called "Controller" but is now called
"Function", and instead
of combining them all on your desktop, you send them off to someone else's mega-mainframe. You
deploy, get an error message, and login to CloudWatch to see what actually happened - it's all
batch-driven, just like the bad old days, so progress is slow. At least we're not having to
walk to the printer on every try, but that pretty much sums up the progress of the last half
century. Oh, and a "Function"
will be able to handle a single request concurrently, so here's your AWS hosting bill, we needed
a lot of instances, I hope you won't have a heart attack. Yes, we run workloads that your nephew
can run on his Raspberry Pi 4, but this is the future of enterprise.

History will repeat itself, of course. Conceptually, hiding a lot of the scaling and coordination
machinery is not a bad idea; programming systems like Erlang and OTP have shown for decades how
well that can work and Elixir is giving the platform a well-deserved surge in popularity. But there's
a big difference here: a platform like OTP handles pretty much everything that a platform like
AWS Lambda handles, but it does it in a single programming language. Tools of the trade are
available: you can refactor Erlang code, you can write Elixir macros, all to keep the system
clean, malleable, and free of [accidental complexity](https://en.wikipedia.org/wiki/No_Silver_Bullet).

It is called "Configuration as Code", and it truly is a great idea (but hardly a new one). But XML
is not code, and neither is JSON, nor YAML, nor HCL. There is an essential quality lacking in all
of these markup languages, and that causes all the copy/paste programming I am currently seeing
everywhere, whether a team deploys on Nomad or uses Terraform or CloudFormation to whip up mega
complex clusters to run a marketing site. This level of complexity is not sustainable, and my
fear is that it will be solved the way our industry likes to solve the problems they have
created themselves: by adding more complexity on top if it.

Not by going back through our history and figuring out what caused the Personal Computer
Revolution. Which is a shame.
