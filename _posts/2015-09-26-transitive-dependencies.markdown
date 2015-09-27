---
layout: post
title:  "Transitive dependencies"
date:   2015-09-26 22:23:18
categories: scala
---

Transitive dependencies look to me a lot like multiple inheritance, and
with the same pitfalls.

(In case you're wondering why suddenly three posts - I came back
from a [Scala conference](http://scalaupnorth.com) and started
blabbering during dinner with [Bill Venners](http://artima.com) and
[Jon Pretty](http://rapture.io) about what I consider the biggest issue
with codebases I currently work with)

C++ gave us multiple inheritance, and we saw that it was bad,
especially coupled with the deep inheritance hierarchies that were
considered proper OO back in the times. Even though conceptually
multiple inheritance was thought to be advantageous, the practical
problems overwhelmed these and best practices moved to single
inheritance and flat class hierarchies. Only recently, a way around
the problem in the form of [traits](http://scg.unibe.ch/research/traits)
seems to have solved the diamond problem part of multiple inheritance,
basically by strictly only inheriting behavior, not state.

Java gave us .jar files as a very simple way to "inherit" code.
Then Maven came by and made it trivial to add more of these .jar
files to a project, simply by adding their name in a configuration
file. Suddenly, problems could be solved by just adding dependencies
instead of writing code, and thus the default way to write new
functionality almost has become to Google for a library, add it to
the build configuration, and wrap it in some code.

For any non-trivial system, people add logging, date/time libraries,
then libraries to do telemetry/statistics, network servers and
clients, talking to service registries, circuit breakers, etcetera.
What people call "microservices" often consists of 10 lines of code,
20 lines of boilerplate and wiring, and hundreds of supporting
libraries in a huge hierarchy that looks suspiciously like multiple
inheritance hierarchy, but then much bigger than even the heavyweight
"OO" methods of the early '90s managed to produce.

New efforts start small, people mix in more libraries, new services
are created, the list of libraries gets duplicated, people get
annoyed, and thus a "common" library gets born. The common library
is often just a list of libraries and not code, but of course that
quickly changes and it morphs into a collection of random code
snippets and a huge list of jars (either directly or, through
transitive dependencies, indirectly).

This is all convenient, of course. A new service is made by starting
an empty project, the common library is added, and everything is there
to start a spanking new service.

Until a bug is found in one of the support libraries. Or you want
to move from Scala 2.10 to 2.12, or JDK 1.6 to 1.7, or change
logging. Suddenly, the whole thing turns out to be one big hairball
of dependencies, stuff cannot be upgraded all at once and slow
upgrades require teams to keep multiple forks, etcetera.

I don't know what the solution is. I do know that I have seen this
occur multiple times and it hurts. A part of the solution is to
stop grabbing a library for every simple problem -
[unclever](http://github.com/cdegroot/unclever) and
[hardcoded](http://github.com/cdegroot/harcoded) are examples of
small, very opiniated and focused libraries that probably took less
time to develop than reading up and learning about their "big
library" counterparts would take. They are focused and thus more
stable; they are opiniated so won't suffer from bloat. Moreover,
they don't introduce new dependencies.

So, I set myself a challenge. Recognizing the causes as being similar
to what caused us to toss deep and multiple inheritance out of the
window should help in finding the solution - flat, single inheritance,
and what would be the equivalent of traits when you talk about jar
files?

In any case, software that is 90% support and support and 10%
functionality - I am not so sure that that is the solution we want.
