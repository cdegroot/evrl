---
layout: post
title:  "Don't use libraries"
date:   2018-01-25 10:24
categories: programming
---
If there is one thing that changed in writing software since I started, it is how easily accessible other people's code is. Open
Source code repositories (started by SourceForge, enhanced by Github), package managers (I guess Perl's CPAN gets the credits there),
it is currently way easier to just borrow code than write it. But that doesn't mean you should do it. On the contrary, most of
the time you shouldn't.

One of our repositories at work contains our Ember-based UI and the assorted Node-based tools to compile the ember DSL into
plain Javascript, build compressed assets, and so on. I'm not a Javascript person but I do know that JS comes with an interesting
packaging concept which makes it really easy to mix and match packages. After the `npm install` routine, the `node_modules` directory
is populated with some 1275 packages, over 300 of them have nested `node_modules` directories, and to me, as a relative layman,
that's an overwhelming amount of "libraries" that composes our UI. It's one of the more extreme examples I can find, but it does
seem that "code reuse" has been taken to the max not only in the Node communities, but also in other places where it is easy
to share and reuse code (like Ruby).

"Code reuse" seems, to me, to have been promoted from a means to an end, and that's usually a sign something is amiss. Especially
because libraries always come with drawbacks, but these drawbacks are longer-term and often overlooked in the race to finish
a task within the targeted time. Here are a couple:

* You pollute your intellectual property with all sorts of unknown licenses. Pull in one Node module that has been published
under, say, the Affero Public License and you might be in hot legal waters. Did you talk to your lawyers about permissible
licenses? If so, did you setup a proxy server like Artifactory with mandatory screening? Or are you just winging it, hoping
you're never get sued?

* You pollute your codebase with a lot of unknown security risks - especially Node/NPM seem to have a rich history there. In the
Ember application above, the application-specific code is fairly simple; it's around 40KLOC of Javascript. However, it pulls in
2.5 _million_ lines of code of Javascript (plus a million more of JSON, Markdown, CSS, HTML, etcetera). Take a low number like
[5 defects per 1000 lines of code](https://www.mayerdan.com/ruby/2012/11/11/bugs-per-line-of-code-ratio): your own codebase
has 200 defects. Your addiction to code reuse added another 10,000. Some of these will be crashes, some of these will be security
risks, you don't know. And you won't, until it is too late.

* That package does everything for everybody and almost what you need for your own problem. [Nobody stunts their
frameworks](https://www.artima.com/weblogs/viewpost.jsp?thread=8826) so you will typically pull in a ton of code you don't need and the API will
be almost what you need but cluttered with options and configuration to make it as generic as possible. It will cost you time
to figure out what to do, how to use the dependency, and you will have to accept that the code will likely run slower than
something you crafted yourself because it is also catering to the 500 slightly different use cases of other users of the
library.

* Deep dependency hierarchies are akin to deep class hierarchies with multiple inheritance. Inevitably, the "diamond pattern" will
come back and bite you. Maybe not now, but then in half a year when you upgrade an entirely unrelated dependency you will get
the dreaded version conflict from your package manager and you know you're in for a couple of hours of puzzling over how to reconcile
the conflicting dependency specifications, probably by upgrading that other dependency as well, and then you find out that it changed
its API so you can redo your code. You'll be doing this pretty much all the time if you want to make sure that you don't depend,
directly or indirectly, on packages that have known security holes. Or you can just stay on Rails 2 and hope for the best, of course...

"Code reuse" is not an end. It's a tool in your toolbox, and there are alternatives to reusing code. Often, it's just simpler
to write the code yourself. Yes, you will write the same code as someone else on this planet. I bet you're writing the same code
as developers at your competitors as well, and I don't think that trying to avoid double work on a planetary scale is a good
basis for decision making.

Dave "PragDave" Thomas coined his ["rule of design"](https://www.infoq.com/articles/increasing-agility-dave-thomas) which isn't original
probably because it is so true in software development: "Good design makes future change easier than bad design." In 1999, I called
it [malleability](http://cdegroot.com/articles/1999-organic-software/) and others who spent more thought on the subject than I have
came up with [similar advice](https://www.martinfowler.com/ieeeSoftware/accChange.pdf). It does seem to be _the_ major principle for software design.

That rule is much closer to being a goal by itself than "code reuse". If anything, this is an overarching design strategy that, when you think
through it, will let you derive strategies like Single Responsiblity Principle, Dependency Injection, etcetera. Given that code reuse through
including packages has a bunch of trade-offs, I guess it is useful to look at "shall I add a dependency on package X?" as a design decision
and consider the trade-offs in the light of the overall "keep your code malleable" trade-off.

What is easier to change in the future? Code that freely flows in one idiom of your chosen language, or code you have to adapt in order to
pull in a library with 20 subroutines, 2 of which are useful to you? What is easier to adapt to your changing requirements? Your own code
or something that needs to be changed by forking and sending pull requests to total strangers? Is including an npm package that has
[a longer usage blurb than actual code](https://www.npmjs.com/package/home-or-tmp) really making your life easier? Is this code really
that hard to write? Weren't you trained to write code, not cobble it up by borrowing from strangers, competitors even? What time are
you saving, amortized over the useful life of your software package and is that worth all the extra risk, both legally and technically?

I think that if you honestly ask yourselves these questions, the answer should be, most of the time, that it is fine to just implement
the functionality, make it nice, go a bit slower but save yourself a ton of hassle in the future. You will essentially be [harvesting
a framework](https://martinfowler.com/bliki/HarvestedFramework.html) that is 100% geared towards your problem domain, and I think that,
done correctly, this is fine. It is similar to the Lisp philosophy of "first build a language specific to your problem, then solve
your problem in it". Not having to track tons of dependencies, fight diamond patterns during upgrades, etcetera is well worth the
initial investment.

So, should you _never_ use a library? Of course not. There are problem domains that require a lot of attention to get things right,
and I think that choosing to use someone's library to get their subject matter expertise (not to save some typing) is completely
legitimate:

* Date/time functions. If you can even get "is this a leap year" right, congrats. For the rest of us, please use a library to figure
out how many days there were between today and January 1st.

* Distribution. Sure, Raft is simple. Even I [can implement it](https://github.com/cdegroot/palapa/tree/master/erix). Now implement
it with all the "optional" features and in production strength.

* Databases and transactions. I only need to glance at my bookshelves to pick out Gray's "Transaction Processing", because it's such a
fat tome. Again, on the surface it's simple - I once wrote a serviceable [transactional persistence system](http://jdbm.sourceforge.net/)
but that one doesn't scale, or does HA, or whatever. I refused all these extensions after reading the aforementioned article by
Feathers on stunting your frameworks ;-).

I guess there are more examples. I think the biggest decision maker is: am I borrowing subject matter expertise I don't have, nor
should have? That's a very good indicator you should use a library.

But apart from that, give your decisions a good thought. Be ok with copying some code around, it is
not the end of the world and may be a surprisingly effective solution. Don't feel ashamed about going either way, as long as you made
a conscious decision and you thought about and weighed all the consequences of the alternatives. You should be able to use less
libraries and go much quicker later on.
