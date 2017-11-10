---
layout: post
title:  "Scala, part II"
date:   2017-04-04 11:24
categories: programming scala
---
Eight years ago, I wrote a [very enthusiastic post](http://www.artima.com/weblogs/viewpost.jsp?thread=260478) about Scala on Artima. It is
2017 now, and I'm a little bit less happy with the language, to say the least.
Consider this a retraction :-)

In 2009, we were in a large Java migration at [Marktplaats](http://www.marktplaats.nl) and
it was not fun. We chose Java because in the group and the parent company, there was
plenty of knowledge, it was _clearly_ beter than the old PHP stuff (believe me, no
bad Java could be as bad as that PHP codebase), we could get decent training, hiring
skilled Java developers was easy, our outsourcing partner [Lohika](http://www.lohika.com)
was happy with it, etcetera.

All good reasons, but actually rewriting the site from scratch was not fun at all. Java
was repetitive, had no higher order functions and other goodies that enable you to mold
the language to your problem instead of the other way around, and we were plodding on,
grumbling.

Scala promised a fresh wind in the JVM ecosystem: for the first time, a language that
was close to Java (like Groovy, unlike Clojure) and would run at Java speeds (like
Clojure, unlike Groovy ;-)), _and_ had all the higher order goodies, type inference,
etcetera that was promising as a set of tools to remove some of the repetition. Read
the old article, and I was enthusiastic about Scala for all the right reasons.

Then we started coding in it. I worked mainly in Scala for the next 4-5 years, then
switched to a Java-only [sister site](http://www.kijiji.ca) of Marktplaats, where I
introduced bits of Scala - as it was still better than just Java, and finally landed
at [PagerDuty](http://www.pagerduty.com) where Scala, certainly back then, reigned
supreme.

In ever case, the same thing happened - Scala has flaws, and they are serious enough
to make me recommend to not use the language. In the Java ecosystem, frankly, I do
not see any language I would like to use; Clojure comes close, but Java shines through
everywhere and if I want to write Lisp, there are [excellent](https://www.cons.org/cmucl/)
[free](http://www.sbcl.org/) Lisp systems out there. The JVM may still have an edge
in performance here and there, but I do not work in industries where this last 5-10%
matters.

So, on to why I think you should not use Scala, item by item, in no particular order:

## sbt

Sbt is probably the best example of what happens when you match academic thinking
with actual code - it's a theoretically very powerful build system, but it is
sufficiently different from regular Scala that it can be considered as a completely
different language, so most developers just poke a bit at it. The Scala language
is complex enough as it is, so as soon as you want to do something that is not "compile
Scala with some libraries into a JAR file", you'll be scratching your head, copying
code you don't understand from StackOverflow, and swearing a year later because an
update broke it. It is also atrociously slow - again, it has some theoretical niceties
like the console mode, but unless you completely adapt your workflow and live in the
console (problem is, you can't, because code reloading ain't perfect in Java) you will
be drinking a lot of coffee (just a clean start takes 5 seconds and triple that in CPU
time on my laptop...).

The JVM generally has a problem with build systems - I think I tried them all, from
GNU Make in the '90s (all we had) through ant, Maven, Gradle, sbt, and I forgot a
couple, but they're all slow. It's a red flag to me that something is wrong in that
ecosystem. Sbt makes things worse, and I guess you're probably better of with Maven
or Ant, because these tools are bad as well but at least I never had trouble understanding
an Ant file.

## How many languages are in your project?

Well, at least two - the selling point of Scala is that it interacts well with Java
code, except that it doesn't. You want, at the very least, wrapper libraries to
hide some of the nastier Java design patterns, so every time you add a Java dependency
you will either have to find out its API and wrap the parts you need in some Scala code
(which I strongly urge you to do every single time), or you'll have non-idiomatic Scala
code and "fun" conversions between Java and Scala types everywhere in your code. Either
solution slows you down - the first one mostly at the beginning, the second one mostly
all the time.

But that's not even the biggest issue - because Scala is highly non-opinionated, and
there seem to be many factions with opinions on what "proper" Scala is, from the very
practical "a mildly better Java" to the less practical "a really nicer Haskell", you
get a ton of sub-idioms to the point where you have entirely separate languages. If
the type system is Turing-complete (which is great if you're a CS post-doc and need a thesis subject), no wonder that people can write code a mere mortal like me simply is not equipped to understand. In "Let a thousand flowers bloom", [Peter Seibel](http://www.gigamonkeys.com/flowers/) eloquently summarized it as:

> Soon we had three kinds of Scala written at Twitter: Scala written by people who wished it was Ruby, Scala written by people who wished it was Java, and Scala written by people who wished it was Haskell. Let a thousand flowers bloom.

People will counter me with "yes, but we have coding standards that prevent this". That is not the point. Or maybe it is. In any case, your coding standards don't count. Try to build a moderately complex project without inadvertently, through some intermediate dependency, pulling in,
say, [Scalaz](https://github.com/scalaz/scalaz). Now your IDE eagerly shows you Scalaz
stuff for autocompletion. Have fun untangling the mess later. Or, at least, trying to understand the library code that pulled in Scalaz and probably is written in the purely functional idiom of Scala that academics admire but mostly reminds me of Perl and other line-noise oriented programming languages. Any sufficiently complex Scala project will carry this burden around, and if your projects aren't complex, I don't think you need Scala.

## Refactor me! and other static typing fun

Type inference was, and I think is, the biggest promise of the static typing people. Scala
was, and I think still is, the pinnacle of type inference in a practical, widely adopted
language. Time and time again, the static typing people have lured me into their traps
with promises of "this time it won't hurt, we solved type inferencing!", and Scala is
just the latest attempt (and, as far as I'm concerned, the last one that tricked me).

The problem is - type inference is hard. And that it is hard shows up in multiple places:

* It doesn't work, so you have type annotations left and right to
aid the compiler or just outright inform the compiler what you want.
Given the complexity of Scala's type system, this is not as easy
as it sounds, especially not if you happen to get some Java typed
data in there as well, or a library that likes Scalaz and comes
with some _really_ interesting types (interesting if I read a paper
on type systems, not when I'm trying to make my product owner happy
with a bit of new functionality, that is). You don't know until the
compiler has checked it what you need, so after a couple of trial
and error rounds, you're done. Problem is, the compiler is slow
(even the incremental one) and at least my attention span is shorter
than that. Now, what was I doing again? Let me ask my colleagues
on Slack...

* This code isn't nice, let me refactor it. Extract some stuff,
and.. Oh, darn, the compiler does not like this intermediate step,
let me see where the type issues are. Ah, I need to have this
temporary type to make sure this step compiles, let me apply that
through these ten source files, yup, it's all happy again I think...
Oops, missed a spot.  So, fixed that. Where was I again?

Refactoring is, in itself, already an interruption of your flow
because you're "supposed" to work on your user story, wrote or found
some iffy code, and are now cleaning up which is all good but you're
now a further stack level away from your "actual" work. With Scala,
over the eight years I've worked with it, the story is always the
same: you refactor in small steps, like you were taught in refactoring
class, you probably even have a plan of how the steps will result
in nicer code, but you do want to make sure and thus compile and
test every intermediate step. More often than not, especially if
it's a refactoring worth doing, the Scala compiler stops you dead
in its tracks by demanding that everything typechecks. In the best
case, you just fix up some type annotations (you should not be
needing, because "type inference", but, hey, that didn't work); in
the worst case, it demands you introduce temporary types to make
the compiler happy, even if the plan of attack won't have these
types in the end result. Every step of the way, it more feels like
the system is working against you instead of with you.

## There's more, but...

This blogpost is already getting too long. In general, Scala makes
me feel like I'm back in early C++ days; a language that makes it
extremely easy to do the wrong thing, a community that somehow
encourages doing the wrong thing (I really think that stuff like
Scalaz has no place outside academia), and a system that does it
best to work against you by slowing down feedback loops - the most
important aspect of a good programming system is quick feedback -
and distracting you with all sorts of stuff that rips you out of
flow. It goes counter to every little bit of experience we, as an
industry, have with powerful programming systems like Lisp and
Smalltalk, that try their utmost best to work with the human being
at the keyboard and forego, as a workable trade-off, some platonic
ideals about programming languages and types; inevitably, the results
of the sort of trade-offs that Scala does not or cannot make is
very high productiviy.

Summarizing: please do not use Scala, and certainly not if not using
the JVM is an option in your organization. There are nicer ways to
spend your life. Type inference, in my opinion, starts to look more
and more like a hard AI problem and therefore it won't be fixed.
In the meantime, if you really want a compiler to check your work,
use something that allows you to opt-in; various languages support
optional typing and allow you to, for example, run your tests for
the intermediate refactoring steps that wouldn't typecheck and then
let you fix up things later. To me, type annotations are mostly
useful documentation and as such, I consider them to be one of the
tools that I have to convey meaning to other humans; I could not
care less about the computer's wishes in this area (especially
because in ~3 decades of programming in static and dynamic languages,
I have found the run-time type error in the latter a fairly rare
occurrence and not worth the trade-off of having to live with a
retarded assistant in the form of a mandatory type-checking compiler).

