---
layout: post
title:  "Tackling software complexity with the CELP stack"
date:   2021-01-15 01:02:03
description: Scaling software is easy; scaling software is hard. Elixir and some libraries can help
categories: agile, elixir
---
The last couple of months I've been deeply immersed in the combination of [Elixir](https://elixir-lang.org)
with [Phoenix LiveView](https://github.com/phoenixframework/phoenix_live_view) for the Web UI,
[Commanded](https://github.com/commanded/commanded) as a CQRS/ES implementation, and [PostgreSQL](https://www.postgresql.org/) for persistence - I think I heard someone call it the CELP stack,
so let's call it the CELP stack. I think this stack is a great way to tackle software complexity.

_[Note: the timestamps in this post are off, but given that they are part of this post's URL, not simply unfixable. The
actual date this got published was 23 May 2021_

Some may disagree with [Fred Brooks'](https://en.wikipedia.org/wiki/No_Silver_Bullet) distinction between
accidental and essential complexity, but my experience indicates that he is correct. I have worked in what is
now called SaaS for almost 25 years (and "regular" software another decade before that), I've seen the gamut between horrible contraptions and trivially simple systems,
and the interesting thing is that hardly ever, the extra complexity was introduced by any sort of requirements: it
just happened, people thought it was normal, "this is what Gartner adviced", and so on. In my experience, accidental
complexity is one of the biggest sources of waste in our industry, and tackling software complexity therefore
logically means reducing that as much as possible. We need simpler systems with less layers that human beings
can reason about, [not more](https://aws.amazon.com/certification/certified-solutions-architect-associate/).

Simple software scales. It scales in both ways of scaling that are important: you can put more load on it in
production and you can grow it (both in terms of LOC and team size).

If you talk about software scaling, most will think you're talking about the first kind of scaling, but, frankly,
that's a solved problem. I've worked with pretty terrible systems that scaled by just throwing hardware at it. Is
it "elegant" to run 300 webservers because your PHP app cannot deal with more than a couple of dozen requests
per second per server? Nope, but it works. And the sort of companies I've worked at invariably saw the idiotically
high hosting bills that resulted as rounding errors. If your revenue is €100million and your growth rate is 20-30%,
it's hard as CFO to get excited about some techie that tells you there's a savings of €2m in the architecture if we
just can stop building new product and to a rewrite.

No, business wakes up when the other kind of scalability starts hurting, is my experience. When they cannot onboard
more developers for all the new great plans that product has for even more growth, or "velocity" slows down because
the software and the infrastructure around it just got too complex: these are the costly things that will make
your CFO take the executive elevator down into the coding dungeons and ask what's up. Tackling this is invariably very costly and very
disruptive to the business, so I've learned to do a little less of "You Ain't Gonna Need It" and a little more
of thinking about how to prevent this kind of pain in the future. The best solution? Ruthlessly hunt down and erase
any kind of accidental complexity. Make "KISS" your religion. Funnily enough, that'll often also help you
scale in the other way: simpler systems are easier to reason about and therefore scaling bottlenecks are likely
simpler to find and resolve.

> Everything Should Be Made as Simple as Possible, But Not Simpler
>
>   -- <cite><a href="https://quoteinvestigator.com/2011/05/13/einstein-simple/">Maybe Albert Einstein</a></cite>

As simple as possible would be - in the Elixir nook of this world - a straight Phoenix app. Phoenix comes with a
very powerful set of generators that make whipping up a quick form trivial. But it also comes with the known
weaknesses of "old school" CRUD apps: read/write mixing makes it hard to scale your RDBMS - often done by adding
read replicas and sending read traffic there - and there's some additional complexity because Phoenix, with its
server-side templates, does not address a lot of the modern expectations of web applications around interactivity,
liveliness, and so on. I think that's why a lot of teams find themselves these days writing a Vue or React application
and then using Phoenix to power the API. Now you have two languages, two deployment pipelines, two production
environments. Often, two teams. It is modern. KISS, it is not.

Read/write scaling has been a focus for me since 2005. The typical scenario is: you buy/lease/cloud-provision a
database server and all is well. You are successful and the DBA comes complaining. You buy/lease/cloud-provision a
bigger database server and all is well. That goes fine for a well - at one company we "joked" that all we would need to
do is survive until the next AWS re:Invent conference where the next bigger piece of kit would be announced, and
I heard rumours that Paypal at one time had a dedicated team hunting for the more powerful hardware for their central
Oracle RAC db. Of course, then the horizontal scaling people start yelling that this is nonsense and that you
should have read replicas because - usually - for every write transaction you have ten reads and half of them
with overly slow queries. Vertical scaling isn't nonsense, of course: it is super cheap, compared to pausing half
your development work (which we assume powers your growth) and splitting reads and writes. Trust me, I've done it.
The more magic your CRUD framework is (looking at you, Rails), the more painful it is.

Split your reads and writes early on, even if you don't need it. It is one of these things that is very simple
to do when you start, very simple to keep up as you grow, and will make future you very happy when the "how do we
scale the DB" question comes up. It's an insurance premium, where in this case the thing you're insured against
is your own success :).

[CQRS/ES](https://www.youtube.com/watch?v=JHGkaShoyNs) is one way of forcing you to split your reads and your writes,
while getting extra benefits along the ride. The extra benefits are quite likely to make business people happy and
thus will make it easier to sell doing it early:

* You force yourself to think about stuff that happens in your system in terms of behaviours, not data. That alone
  will make things more understandable;
* You have this event stream that lets you trivially walk through history, links events together, and is a veritable
  gold mine for the data scientists that management hired without knowing what to do with them.
* It is trivially easy to create new projections, replay the past, and create whatever dashboards management wants
  without having to tell them that they'll be useful in 6 months because you weren't collecting the data.

Greg Young advices not to use Event Sourcing unless there's a proven need, but he's a C#/.Net person and I get it that
when you start with an overly complex and baroque system, you want to be careful adding more ;-). However, over here in
Elixirland it's a
somewhat different story: we start with a simple system and Event Sourcing is a natural extension to a system
that is already driven by functional thinking and data pipelines. I am very close here to inverting Greg's advice:
use it unless you have reasons to not do it. With Commanded, Elixir has top-notch support for the collection of
patterns that go under this label and adopting it is simple.

That leaves the other thing: our two systems/teams for front-end and back-end. Phoenix LiveView is the answer. It's an
answer that will make a lot of "I only know Javascript" type of front-end developers unhappy, but if you're truly
"full stack" then adopting to it should be pretty easy. I can be short about it: I've used pretty much everything
under the sun since my first forays on the web (through X.25 from the DECUS Germany server to CERN) and this,
so far, is the only thing that has truly worked both in terms of productivy and long-term cleanliness. It is not
perfect yet, I'm hoping that some component system will get added, but as far as web application frameworks go
it is very close to optimal. One language, and one running server on your system, seamless refactoring of logic from
your UI down to your business logic when you want it: it is all there and thanks to Elixir's excellent incremental
compilation system and Phoenix' pushes, your page is refreshed in the time it takes you to blink after you save
an IDE buffer. Pro tip: add TailwindCSS for even more fun, but that is strictly speaking not part of this story.

The true magic comes when you combine these two concepts. Because Phoenix LiveView is also all about read/write
spitting the two fit perfectly: user writes get turned into messages to the server process (every active LiveView has an active
OTP "gen_server" process), and view updates/reads - after the initial load - are pushed down by messages from the server
process back to the view in the browser. This is what you end up with:

* User presses "submit" to, say, add something to their shopping cart;
* the active LiveView server receives event, and converts it into an "AddItemToCart" command which it dispatches
  to the Commanded application;
* Commanded takes the command and routes it to your "ShoppingCart" aggregate root, hydrating it if necessary;
* Your aggregate root has an `execute` method that checks whether the item wasn't already in the cart, etc, and
  if all is well, emits an "ItemAddedToCart" event;
* Commanded persists the event;
* PostgreSQL fires a trigger which prompts a listener to wake up;
* That listener, another Commanded building block, gets the event and sends them out to anyone who is interested;
* One of the interested parties is your Ecto projector, which takes the "ItemAddedToCart" event and translates
  it into a row add in a database that has current shopping carts (let's assume that shopping carts must be
  persistent here);
* After your listener is done, Commanded calls the module's `after_update` handler which puts the events on a
  Phoenix PubSub topic;
* That view that received the event subscribed when the cart was created/retrieved, so receives the event as
  a `handle_info` standard OTP message;
* Your event handler converts the event into a simple structure and assigns it to the socket;
* LiveView notices the updated assigns, calculates the deltas with the current assigns, and pushes a diff to
  the browser;
* The browser patches the DOM, the user sees the extra cart item.

Complex, not? Well... not really. Mostly a lot of typing :). But look at the finegrained separation of concerns
here - that is what will matter as you grow. Not how many LOC you have, but how little spaghetti.

Your Command cleanly defines what data is required to perform an action. It also is the entry point if you
ever want to create an alternate UI - just send the already implemented commands. It is a specification:

```elixir
typedstruct module: AddItemToCart do
  field :id, String.t()
  field :product_id, String.t()
end
```

Your `execute` handler does contract checking: is all the data there? Is it possible?

```elixir
def execute(cart, c = %AddItemToCart{}) do
  if has_item(cart, c.product_id) do
    raise "You cannot add the same item more than once"
  else
    %ItemAdded{id: cart.id, product_id: c.product_id}
  end
end
```

Your `apply` is responsible for effecting the state change (if any) on the aggregate root. And that completes the model side
of things - notice the complete absence of external dependencies here, making for trivial testing.

```elixir
def apply(cart, %ItemAdded{product_id: product_id}) do
  %{cart | items: [product_id | cart.items]}
end
```

Your projection (normally you'll have an Ecto projection - you could argue that a shopping cart may not belong in the
database and that is fine, with projections you can always change your mind later) converts the data into something
that is easy to read on the initial load of a page.

```elixir
project(e = %ItemAdded{}, _metadata, fn multi ->
    Ecto.Multi.insert(multi, :project, %Projections.CartItem{
      id: e.id,
      product_id: e.product_id,
    })
  end)
```

Finally, on the UI side, your event handler is just concerned with creating a command in reaction to user actions...

```elixir
def handle_event("add-item", %{product_id: product_id}, socket) do
  App.dispatch(%AddItemToCart(id: socket.assigns.cart_id, product_id: product_id))
  {:noreply, socket}
```

...and in the PubSub handler with updating the view state by updating the socket state (LiveView will then figure
out the differences and send out a push).

```elixir
def handle_info({:ecto_projection_completed, %ItemAdded{id: cart_id, product_id: product_id}}, socket) do
  socket = if cart_id == socket.assigns.cart_id do
    assign(socket, items, [product_id | socket.assigns.items])
  else
    socket
  end
  {:noreply, socket}
```

Whereas with a normal CRUD app, a lot of these concerns get mashed up in a single controller method, the CELP stack
forces you to separate everything, which will force you to write short, clean, and reusable code. Absorbing new changes
is just as trivial. Say that product says that you can have more than one item of a product in a cart.

* Your command handler checks whether there's already an item with that product id in the cart. If so, it will emit
  an, say, "ItemCountIncreased" event instead of "ItemAdded".
* A new event handler just increases the item count for an existing item (the cart state will now probably be a map
  or keyword list containing (product_id, count) elements).
* A second projection method handles that event, just emitting an update. The insert and update are nicely separated
  without conditional logic.
* You can now add a command to set the item to a certain count and hook it up to the UI, reusing a lot of the pipeline
  already created for adding an item.

In essence, CELP forces you to build small, reusable blocks. It's more typing (although macro libraries are already
being brewed in this space), but the extra code is clean, concise,
and small enough to form a bucket of reusable blocks. In the meantime, you're gathering a history of everything that
happened in your web app for when management decides that Data Science is the next big thing ;-). Yes, you can achieve
the same end result using TDD and refactoring, but sometimes it is nice to know when to skip ahead.

"You Ain't Gonna Need It" is very good advice. Every time you pull in more than the bare minimum to get things done,
you need to see it as an actuarial thing: are you insuring against something that could conceivably happen? Is the
premium small enough to not get in the way of the important stuff? If so, you should consider pulling in the little
extra thing. In my experience, it is exceptional that such a move is warranted. In my experience, pulling in the CELP stack
is such an exception.
