---
layout: post
title:  "Erlang stuff that Elixir developers should know"
date:   2019-09-24 01:02:03
categories: programming elixir
---
A lot of people are learning Elixir these days. Just at [my employer](http://www.pagerduty.com),
more than 150 people have taken [Dave Thomas' excellent video course](https://codestool.coding-gnome.com/courses/elixir-for-programmers). Learning Elixir is simple, but you'll also be embracing a new ecosystem,
the Erlang/OTP ecosystem; treating it as a black-box is just going to make life harder, so
let's open it.

There's a lot to be said to learning Elixir, even if you don't need it
for work:
* Given how hard it is to make anything mutable, it's a great way to force yourself into
  good functional programming habits;
* There's something in how the OTP actor system makes distribution and concurrency easy
  to reason about;
* [Phoenix](https://phoenixframework.org/), and especially [LiveView](https://dockyard.com/blog/2018/12/12/phoenix-liveview-interactive-real-time-apps-no-need-to-write-javascript) is a great way to build websites that scale;
* [Nerves](https://nerves-project.org/) is mindblowing if "large embeded", like Raspberry Pi, is your thing. Trust
  me---try it and then over-the-air update some firmware.

The ecosystem is not yet as rich as, say, Python's or Ruby's, but mostly what you want is there. On top of
that, the language is much cleaner,
development is probably faster, and the end result will actually run at speed and scale well. Furthermore,
as I said, it is simple to learn; there are some harder areas like macros (I
think that [Saša Jurić' introduction](https://www.theerlangelist.com/article/macros_1) is an excellent
one), but overall it is mostly "nothing to see, no magic, no sleigh of hand".

However, the way Elixir achieves all that goodness is by literally standing on the shoulders of a giant: the
Erlang/OTP ecosystem. Elixir is an extremely well-designed language, but José Valim had a relatively
easy job of creating a run-time library given that most of it is a thin wrapper around the already
excellent code that Erlang and OTP provide. Needless to say, there is a point where you have to look
down, notice the giant, and interact with it.

Therefore, given my proclivity for advocating "machine affinity", here's a list of stuff you should
do to learn more about the system you're working on. Elixir does, in most places, an excellent job
of hiding Erlang and OTP from you, but there are times where it makes sense to remove the veneer
and look at the raw material underneath. The list here is roughly in order of importance/complexity
and all but complete.

## Learn some Erlang

Erlang is a language from a different time. Whereas languages these days let themselves be inspired
by Python or Ruby, Erlang comes from a time where things like Prolog where what you looked at. As such,
it's a bit "funny": variable names start with an uppercase, statements are separated by commas,
atoms are just lowercase words and thus don't stand out like `:atom` does, and so on. [Learn you
some Erlang](https://learnyousomeerlang.com/introduction) and other introductions abound; work
through one until you are at least comfortable reading and maybe patching Erlang code. A lot of libraries
rely on Erlang libraries and there will be a time you will have to dive in.

## Learn some OTP

A lot of the power of Elixir directly derives from the power of the famous Open Telecom Platform, whether
you write telco software or not. The Elixir docs mostly tell you how to use OTP elements like GenServers
and Supervisors and Applications, but are thin on details. For good reason: why replicate documentation
that already exists? The [OTP Design Principles User's Guide](http://erlang.org/doc/design_principles/users_guide.html) goes into details and I consider it mandatory reading for everybody who is serious about Elixir.

## Learn about releases

Since Elixir 1.9, `mix release` is a freebie. Before that, [Distillery](https://github.com/bitwalker/distillery)
did a great job. However, here is one of the spots where Elixir and Mix muddy the waters a bit too much: by
going out of their way to be user friendly, a lot of the underlying details are hidden and that often
results in confusion and black-box prodding (try something and see whether it works in production, say).

Here's a small exercise. Open up a command line, and check that elixir and mix are version 1.9. If not,
make it so (I use [asdf-vm](https://asdf-vm.com/) for that). Then:

```
cd /tmp
mix new releases
cd releases
mix release
cd _build/dev/rel
find . -print
```

Questions you now want to answer:
* What files are here? What are their roles? What are, specifically, the '.script' and '.boot' files?
* What happens if you bump the mix.exs version number and run `mix release` again?
* What happens if you add a dependency and run `mix release` again?
* What happens if you add `:observer` to `extra_applications` and run `mix release` again?

You just read the OTP guide, including the [chapter on Releases](http://erlang.org/doc/design_principles/release_structure.html), so what is in there does make some sense. Note especially how files you would manually create
and maintain when coding in Erlang are mostly generated for you with Elixir.

For bonus points, create or copy a simple Erlang project and build a release. Compare with an Elixir release.

## Learn about the Erlang standard libraries

People joke that Erlang is not a language, but an operating system. To some extent, that is true because the
system comes with an enormously broad list of libraries. All these are pretty much free to use (they're
already installed) and rock solid (they run a good chunk of the world's telecom operations), so should be
a first option to use. Some pointers:
* The [ERTS documentation](http://erlang.org/doc/apps/erts/users_guide.html) contains the basic library
  documentation, including distribution (the chapters on building custom distribution and discovery
  protocols are very interesting) and the [`:erlang`](http://erlang.org/doc/man/erlang.html) module which
  has a pretty random collection of sometimes very handy utilities.
* Browse through the [reference documentation](http://erlang.org/doc/man/) of the bundled applications; there is
  a lot there, and what you want to skim through is most likely going to be influenced by your interests. I'm
  sure there is someone out there who is going to jump with enthusiasm when discovering that there is a
  bundled [ASN.1 to Erlang compiler](http://erlang.org/doc/man/asn1ct.html) in there.

## Learn about ETS and Tracing

ETS is the main way to share state in the Erlang/Elixir ecosystem, and it is [well-documented](http://erlang.org/doc/man/ets.html). Tracing is something completely different, a way to inspect a running VM. It is documented
[here](http://erlang.org/doc/man/erl_tracer.html), [here](http://erlang.org/doc/man/erlang.html#trace_pattern-3),
and probably some pages that lead from there.

Now why would I bunch these two together?

The commonality is called ["match specifications"](http://erlang.org/doc/apps/erts/match_spec.html) and it is
an important topic to pick up when you go deeper; it enables programmatic tracing and advanced ETS operations and
it is in fact a small, primitive mini language you write using ASTs. Erlang has a compiler available to compile
functions to match specifications, and a neat exercise would be to create one for Elixir too.

## Dive into the OTP source code

I'm out of ideas here but if you completed the above, you're well underway on the path towards expertise; what's
more, you can probably find where things are now. There's always a point, though, where you want or need to
know _exactly_ what is happening. To that end, I always have a checked out copy of the [OTP source code](http://erlang.org/doc/apps/erts/match_spec.html) lying around, built and ready to go if I want to add some debugging
`printf()` statements or just want to lookup what is happening (I also have the [Elixir source code](http://erlang.org/doc/apps/erts/match_spec.html) always checked out, but it is simple enough to jump from documentation to source code on Hexdocs so there's a bit less need for that).

Want to know what happens if you write `Kernel.send/2`? Grab the OTP code, and dive right in. There are
two main directories:
* `erts/` has the "emulator" (the BEAM VM and associated stuff) and the base libraries (so you'll find
  the `send` call there, most likely as a built-in function (BIF));
* `lib/` has the bundled Erlang libraries, mostly a mixture of Erlang code and native interface functions (NIFs)

I've found the code well-readable and reasonably commented, so to me it has been a fallback that, while not
needed often, has proven an invaluable resource during that one or two times I really needed to know what
was going on.
