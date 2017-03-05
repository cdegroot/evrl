---
layout: post
title:  "Mocking in Elixir"
date:   2017-03-05 13:44
categories: elixir, tdd, mocking
---
[Elixir](http://elixir-lang.org) is a pretty new language and as such is
still developing patterns and adapting existing patterns to its sometimes
idiosyncratic ways. One of these concerns mocking.

[José Valim](http://blog.plataformatec.com.br/2015/10/mocks-and-explicit-contracts/)
and [Lasse Ebert](https://medium.com/@lasseebert/mocks-in-elixir-7204f8cc9d0f#.u0v24e397)
both rally against mocking libraries that basically fiddle around with the global
mapping of module names to code, and I think that that is a very good thing. However,
their solution leaves to be deserved for several reasons. A minor reason is that with
Lasse's solution, you will not be able to see in your test code that a mock is used. It
is stowed away in configuration and the test helper script. I like languages that
have everything out in the open, and this smells for me.

The bigger question, however, is what are you mocking? I'm from the OO world, and I like
to mock objects - individual instances of classes in the OO world. In an OO language,
you have an interface and two implementations (the mock one and the real one). Only in
the individual test case you make the decision which one you need and instantiate the
chosen one. For a unit test, you will want to mock implementation, but for an integration
test the real one might just suit you well.

With the mock module plus config setup, you can't do that. There's effectively a singleton
and the decision is made for all tests. I'd like to do better - have real mock objects and
make it clear in the tests what we're using.

So what is the equivalent of an object in Elixir? "That which encapsulates state and behavior",
which means a specific process and its associated code (callbacks, etcetera). Most simply,
an object in Elixir is usually a Module (for the behavior) and a pid (for the state). If
you use that as your object pointer, you can mock as you want. Here's an example, say
we have a persistence subsytem (and we only look at writing here to keep it simple):

```elixir
defmodule Persistence do
  defmodule Behaviour do
    @callback persist(pid :: pid, key :: String.t, data :: Map.t) :: :ok
  end
end
```

So far, nothing special. Now, the API to persistence is going to look almost the same,
but it takes a module as well as a pid to find out where to send to:

```elixir
  ...

  def persist(_persister = {module, pid}, key, data) do
    :ok = Kernel.apply(module, :persist, [pid, key, data])
  end
```

So if we call `Perstence.persist(...)` then this function will basically forward the call
to the module. An example is the mock, which I often implement right in the module as mock
behavior is often simple and shared so we might as well offer up a default one. Here's a
possible mock implementation:

```elixir
defmodule Mock do
  @behaviour Persistence.Behaviour

  def start_link do
    {:ok, pid} = Agent.start_link(fn -> %{} end)
    {:ok, {__MODULE__, pid}}
  end

  def persist(pid, key, data) do
    Agent.update(pid, fn cur -> Map.put(cur, key, data) end)
  end

  def get({_module, pid}) do
    Agent.get(pid, fn state -> state end)
  end
end
````

Two things of note here - we don't return the standard `{:ok, pid}` tuple, but rather we combine
module and pid and return that. There's also a `get` function that we can use in tests to
see whether data got persisted. Talking about tests, let's see how this is used in a tests:

```elixir
defmodule SomeTest
  use ExUnit.Case, async: true # shouldn't that be the default in Elixir anyway?

  test "My Beautiful test of Some" do
    {:ok, persister} = Persister.Mock.start_link()
    {:ok, my_thing_under_test} = Some.start_link(persister)
    ... do some stuff with my_thing_under_test ...
    persisted_stuff = Persister.Mock.get(persister)
    ... assert that things are in persisted_stuff
  end
end
```

I think that that's reasonably decent code. It clearly shows we're mocking Persister, it
injects the mock into Some (whatever that may be), and when we done with our call, we
simply ask the mock for its status and assert on that. In the production code it's as
simple as calling the API's method:

```elixir
defmodule Some

  ...

  def handle_call(:dump_your_state, _from, state) do
    Persister.persist(state.persister, state.key, state.data)
  end
end
```

assuming that `persister`, `key` and `data` are, of course, corresponding fields in Some's
state structure. Note that the fact that `persister` is not your usual pid, but it is
completely transparent for the code that gets it injected, as is should be.

## Conclusion

While module-based switching may work well, and has the advantage of no compile time overhead,
I think it is too coarse and too opaque. The alternative presented here isn't much harder
to implement, comes with minimal run-time overhead, and has the advantage that test code
becomes easier to follow and more flexible, as implementation-switching can be done on a
case-by-case basis.
