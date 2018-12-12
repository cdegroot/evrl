---
layout: post
title:  "Integration testing considered harmful"
date:   2018-07-08 20:24
categories: programming testing
---
I just watched [J.B. Rainsberger](https://www.jbrains.ca/)'s talk
["Integrated Tests Are a Scam"](https://vimeo.com/80533536) and I urge you to do so as well (I'm
gonna refer to him as JBR down below in line with the good old "name people by their TLA" tradition).

It is a pattern that hits me over and over again, and I feel like groundhog day already
because this is the fourth gig in a row where I enthusiastically start coding away, then
ask "what is this?", get the answer "oh, our legacy monolith" and commence work on getting
rid of it. I guess I'm getting old.

Integration tests. Or, as JBR probably correctly calls them, integrated tests. Tests that
don't test whether the 10 lines of code you just wrote are correct, but also the 10,000 lines
below that and a couple of million lines in, say, MySQL. These tests are a scourge.

Just yesterday I changed some code which essentially added two lines to an Elixir "application" (in
Elixir/Erlang/OTP, an "application" is basically a coarse-grained component and I'm gonna use the
word "component" going forward to not make this language/platform specific). One line to grab
a number from a new component I wrote, and one line to actually insert that number in an outgoing
Kafka message.

`make test` showed three components that failed. The component I just added the single line to (I was expecting
that, of course), and two components that had that one as a dependency. Graphically:

```
{C,D}
  \-- B
      \-- A
```

I added A, B depended on that and I expected some test breakage there, but C and D?

It turns out that C and D had "integration" tests, just like B. Basically test that treat everything as
a black box, make real connections to Kafka and MySQL (but arbitrarily stub out others because "too hard
to get running in Circle", like our Rails monolith that this service also talks to), and it is easy to see
that if you add behaviour in A, use that to change how B operates, then C and D will suddenly see a differently
operating black box and will have failing tests.

I tried to make the tests run which was actually trickier than I thought. After an hour of trying, I filed a
pull request instead to just remove these tests. Tests that play dominoes and are hard to fix have no place
in good software. Then I remembered some blog posts by JBR, found the video, and watched that instead. Way
better use of my time than trying to fix tests that shouldn't be there in the first place.

Integration tests are usually harmful and deliver no positive value. That is, they do deliver some value, but
it is offset by the high cost of maintaining them and therefore they don't make sense, economically. They are just
too darn expensive.

So, what is the solution? Starting with your test suite, if you have a language that so clearly and
easily componentizes like Elixir, stub liberally. So, in the above dependency tree:

* A tests itself. It tests whether its external API abides with its contract, it may use the stacked testing
  that JBR proposes, or actually have integrated tests, I don't care. I have my preferences, but I don't
  care that much. One nice thing of componentizing your software is that you can stop caring at component
  boundaries.
* B tests itself, but stubs out A. Always. Again, I'm more than happy to treat B as a black box, but it is
  not allowed to instantiate A. I also don't care whether A provides a stub or B creates one, as long as
  the code in A doesn't get invoked during B's test suite.
* C and D, likewise, never invoke B but stub it out. Given that B has two modules using it and we're looking
  at a single service/repo here, DRY says that B's stub might end up with B but it is not clear cut - for all
  I know C and D use B in non-overlapping ways and thus can each have a partial stub.

Oh, and I'm not a linguist either, so I don't really care whether you call it a "stub", or a "mock", or a
"test double". To quote the great Richard Feynman:

> “You can know the name of that bird in all the languages of the world, but when you're finished, you'll know
> absolutely nothing whatever about the bird. You'll only know about humans in different places, and what they
> call the bird. ... I learned very early the difference between knowing the name of something and knowing something.”

I care about what it _is_, not about what you call it. Yes, one helps identifying the other but the terminology
is so tangled up that I'll look past your code's name and into its structure to see what it is :).

JBR spends quite some time about explaining about how testing two sides of the contract should prevent issues
when you actually integrate code. I think he knows that there will be gaps here, and I hope that he'll have the
same opinion as I do: "this is fine". The goal of your test suite is not to eliminate all errors, it is to
give a good return on investment on writing test code versus fixing production issues. Test suites are economically
defined, not by some absolutes. If the cost of failure is low, risk more. Besides, there is more you can - and
probably should - do besides testing:

* Deploy your branch to a staging environment and fire a "tracer bullet" through your code. Preferably automated, one
  button. "Integration" tests in your CI environment typically don't test the actual database, the actual message queue,
  the actual monolith-you-don't-want-to-talk-to-but-have-to, and so on. Your staging environment probably has these
  and thus deploying and testing there is a way better method of checking whether things properly fit together.

* Have components emit health check data and have an integrated "all good" health check API endpoint in your service. It
  will tell you whether indeed it can talk to Kafka, MySQL, that API you depend on, and so on. Again, not in some
  synthetic CI environment but in the real deal, which is way more useful information.

* Deploy your final code to production as a canary; the aforementioned health check will tell you whether all the
  fiddly (and largely untestable) bits around configuration are correct before any traffic is sent to it. Then, when
  that is all green, you trickle some production traffic through it for a couple of minutes and when that is all green,
  there's a good chance that you can ramp up the new version and all will be good; things you would classically catch
  with integration tests usually aren't the sort of subtle things that fly under the radar but more likely the big
  "oh, well, we didn't look at _that_!" stuff that explodes right in your face the first time you hit the service.

* Be able to roll back in a minute. Sometimes, a thing _does_ fly under the radar, you roll back, figure out a test for
  the issue that happened, and try again. In most industries, the one or two minutes of irregular behaviour are not
  very costly; normally, this is less costly than carrying around a 1000-test integration suite that breaks all the time.

Note that this requires a pretty flexible environment, where you can run multiple versions of a service side by side,
have automated health checks, and so on. All this newfangled stuff around "DevOps" is not there to make deployment
simple. Deployment was simple enough when I built Win16 installers to be burned on a "Golden Master" CD in the '90s
when working for an ISV. It was one script and a cup of coffee. I don't need no stinkin' buzzwords to accomplish that.

No, all this stuff is there to make deployment _low risk_.

You reduce risk by lowering impact and cost of bad deployments and you can use that reduction to offset some of the
heavy-handed testing that people still thinks are needed. There's no test like actual production traffic.

"Testing" is a means, not an end. And it is part of a whole chain: writing code, reviewing it, having automated tests and a CI
environment, deploying it through a CD environment with automated canary testing, even manual production testing after
deployment: they are all small steps in a large chain that just has one goal: add value to your business. Locally optimizing
one bit of that chain by focusing on "we have to integration test everything" while not seeing that other parts of the
chain are so good they allow you to do _less_ testing is folly. Optimize the whole system and know how to make trade-offs
to that the whole chain adds maximum value at minimum cost. __That_ is the art of software development, if you'd ask me.
