---
layout: post
title:  "Uderzo vs Scenic"
date:   2018-07-08 20:24
categories: programming elixir
---
At ElixirConf 2018, [Boyd Multerer](https://github.com/boydm) finally announced
[Scenic](https://www.youtube.com/watch?v=1QNxLNMq3Uw&list=PLqj39LCvnOWaxI87jVkxSdtjG8tlhl7U6&index=51),
his Elixir-native UI. Since I've been
[dabbling](https://github.com/cdegroot/uderzo_poncho) in the same space,
I thought it'd be interesting to write up a quick post.

First, let's make one thing clear: Scenic is a pretty much finished,
ready to go product, and looking at the company Boyd is setting up, I
wouldn't be surprised if he plans to make his living that way. Uderzo is
a half-baked, unfinished idea. I'm very much comparing apples and onion
seeds here. If you want something that's ready to roll, use Scenic and stop
reading. The biggest difference between Scenic and Uderzo is that Scenic
has been created by someone competent and experienced in this particular
area :)

Having said that... some random remarks:

Both frameworks try to be OTP native and both don't trust OpenGL/NanoVG
in that the rendering code lives in a separate executable. Both indeed
use NanoVG.

..and the similarities pretty much stop there. I think there are two major
differences between the two systems:

1. Uderzo's use of Clixir makes it very fluid where your code goes - in
the port executable or in native Elixir; there's no difference between
"privileged" native code and user code, and that's on purpose as I wanted
to created something malleable and extensible. Clixir enables me to do
something akin to what Squeak Smalltalk does for the whole VM: write
it in Smalltalk but a subset that can be compiled to C code. Clixir,
frankly, is the more important bit of the system to me and most likely
where I'm gonna primarily focus my efforts as it is applicable in more
areas than just graphics.

2. Scenic is a high-ish-level "retained mode" library where you define
scenes and let some machinery work out what needs to happen; Uderzo is
(again, on purpose) very low level and close to the metal with just NanoVG
(optionally) in between; "immediate mode" is apparently the label to
apply there (there's some room for confusion there as OpenGL uses the
words slightly differently from how I think Boyd uses them; in any case,
in Scenic, the library manages and renders the model; in Uderzo, that is
your job).

Summarizing: Uderzo is neat for low level tinkerers who want to have full
control and don't mind some hardware dependencies - I'm not a graphics
programmer, but I guess that the low level will also make it more suitable
for games; Scenic seems to me perfect for people who just need to get
things working and where the UI is more a means than an end. Both play
nice with OTP and therefore should be robust; given that at the end of
the day, the "scary" native bits are handled similarly, my hunch is that
they should be equally robust. I for one am happy that Scenic is there
- there's now a good option for Elixir-native UIs on small devices and
one that looks like it'll get a decent amount of support and community.
