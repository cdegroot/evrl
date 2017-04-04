---
layout: post
title:  "Packaging Elixir"
date:   2017-04-04 11:24
categories: elixir deployment
---
You're all down and now you want to deploy your awesome new Elixir project. Question
one is - how to package it up so you can ship it off? There are tons of solutions
and there's little guidance out there. Here's my (quick) take on it.

First and foremost, I think you want for all but the simplest cases a proper Erlang
release for deployment. [Distillery](https://github.com/bitwalker/distillery/) is your
friend here.

The next question is - how to set up distillery "environments", Mix environments, etcetera?
Mix environments are notiriously risky - they are working mostly at compile time, Mix is
not a proper part of your release (so `MIX_ENV` won't work there either), and in my opinion
the default way should be to stick as close to [12 factor](http://12factor.net) as possible.
That means that variations between your various deployment targets should sit in your
environment, not in your code.

In short, the recommendation is to have three, and exactly three, Mix environments:

* `dev` - Your default development environment; it will point to all sort of localhost stuff for when you want to play with your code (your local database, a local RabbitMQ instance, etcetera).
* `test` - The Mix environment when running tests. Don't rely on it too much but for starters, having less logging in your test environment often makes for nicer reading of test reports.</dd>
* `deploy` - The Mix environment, and the _only_ Mix environment, you use for deployment. This is where you point to your actual database, search engine, and whatnot - but given that you hopefully have some pre-production environments as well, not by directly hardcoding them but by using Distillery's `REPLACE_OS_VARS` functionality.

Similarly, you should stick with only one Distillery environment, also named `:deploy`. This way, there's no confusion - no matter how many different environments you deploy to, the same single release will work and all the variation between your environments is documented in the list of environment variables that will be expanded in the generated `sys.config` file. It's simple, flexible, and there's a reason that it's recommended in 12FA.
