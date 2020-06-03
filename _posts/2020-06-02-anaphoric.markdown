---
layout: post
title:  "Fun with macros: anaphoric lambda"
date:   2019-09-24 01:02:03
categories: programming elixir lisp
---
In ["On Lisp"](https://en.wikipedia.org/wiki/On_Lisp), Paul Graham introduced the concept of "anaphoric macros",
a term he borrowed from linguistics. We can have them in Elixir as well, and while they're dangerous because code
written in them might confuse people, they are fun to play with.

So what is an anaphoric macro exactly? Let's start by example: Common Lisp has the loop macro which is pretty much
`for` with everything but the kitchen sink. I think books have been written about it. From the
[Wikipedia article](https://en.wikipedia.org/wiki/Anaphoric_macro) on anaphoric macros:

```lisp
(loop for element in '(nil 1 nil 2 nil nil 3 4 6)
       when element sum it)
 ;; ⇒ 16
```

Like an anaphora in linguistics, `it` refers to something that came before, in this case the result of
the conditional clause. It just magically appears, without warning, poking up its head like a jack-in-the-box
and you need to know about the loop macro to know that you can use `it` here. But, for its expensive and
posh sounding name, it's a simple concept: a macro that on expansion introduces variables the user didn't
specify.

A more interesting example is when you make an anaphoric lambda macro:

```lisp
(defmacro alambda (parms &body body)
   `(labels ((self ,parms ,@body))
      #'self))
```

This macro packs a punch: it introduces the lambda expression as an anaphoric variable in the macro, which
means that you can refer to it from the lambda expression. The result is that you can write:

```lisp
(alambda (n)
   (if (= n 0)
     1
     (* n (self (1- n)))))
```

Look - an anymous function that can call itself recursively supported by just three lines! Much nicer than
the [Y combinator version](https://rosettacode.org/wiki/Anonymous_recursion#Using_the_Y_combinator) which is too
long to list here. On the same page, the Elixir Y combinator solution is much shorter:

```elixir
fib = fn f -> (
      fn x -> if x == 0, do: 0, else: (if x == 1, do: 1, else: f.(x - 1) + f.(x - 2))	end
	)
end

y = fn x -> (
    fn f -> f.(f)
  end).(
    fn g -> x.(fn z ->(g.(g)).(z) end)
  end)
end

IO.inspect y.(&(fib.(&1))).(40)
```

It works, but it is hardly elegant. Can we have an anaphoric lambda in Elixir?

Well, yes, but it is somewhat harder. While
Elixir has a pretty powerful macro system it has a very clear and compile time distinction between "data" and "code" and
the sort of on-the-fly function definition thing that the Lisp version leans on using `labels` won't cut it. The simplest
I got to work is this:

```elixir
defmodule AnaphoricLambda do
  defmacro alambda(fun) do
    r = Macro.var(:self, nil)

    quote do
      unquote(r) = unquote(fun)
      fn (x) -> unquote(fun).(x, unquote(r)) end
    end
  end
end

defmodule Test do
  import AnaphoricLambda
  def test do
    fib = alambda fn
      x, _ when x < 0 -> raise "Stay positive!"
      0, _ -> 0
      1, _ -> 1
      n, self -> self.(n - 1, self) + self.(n - 2, self)
    end
  end
end

IO.inspect Test.test()
```

This is short but not very elegant. We inject the function into itself to enable the recursion, but that means that
we need to write the single-argument function as a two-argument one where the second argument is the anaphoric
variable.

Now, I'm not sure that this is an optimal solution but I puzzled over it for a couple of hours and this
is the best I could come up with (I'm writing this post partially in the hope that someone will correct me and come
up with something nice - I'm hardly a macro specialist). Note the

```elixir
r = Macro.var(:self, nil)
```

which creates a variable `self` in an empty context. If you start digging through the ASTs during macro hacking then
you'll see that you need this nil context for such local variables. It expands to just `{:self, [], nil}` but writing
it this way shows intent better.

What we want is a simple one-argument `fn` and the macro then has to tweak things. Without
further ado:

```elixir
defmodule I do
  defmacro alambda(block) do
    r = Macro.var(:self, nil)
    block = Macro.prewalk(block, fn
      {:when, meta, elems} ->
	{:when, meta, [r | elems]}
      b = {:->, meta, [lhs, rhs]} ->
        case lhs do
	  [{:when, _, _} | _] ->
	    b
	  _ ->
	    {:->, meta, [[r | lhs], rhs]}
	end
      { {:., meta, [{:self, _, _}]}, _, args} ->
        { {:., meta, [r]}, [], [{:self, [], nil} | args]}
      other ->
	other
    end)
    block = quote do
      r = unquote(block)
      fn x -> r.(r, x) end
    end
    IO.puts Macro.to_string(block)
    block
  end
end

defmodule Test do
  import I
  def test() do
    fib = alambda fn
      x when x == 1 or x == 0 ->
        x
      x ->
        self.(x - 2) + self.(x - 1)
    end
    IO.inspect fib.(20)
  end
end

Test.test()
```

We made the _invocation_ quite nice: the `alambda` expression now has just a single argument and the `self` is
magically injected. But that macro is enough to give you a headache (and it is hardcoded to single argument
functions, although that shouldn't be _too_ hard to fix). We use `Macro.prewalk/2` to walk through the function
body and we match for three clauses in the AST:

* `{:when, meta, elems}` matches a `when` clause. `elems` are the arguments to `when` and this is a flat list
of parameters and then the when expression as the last element. We simply prepend `self` to the parameters so that
`x when x == 1` now reads `self, x when x == 1`.

* `{:->, meta, args}` matches a regular function head. We need to exclude a `when` clause in therei because it gets
processed in the first clause but otherwise we do the same trick: `x ->` is changed to read `self, x`.

* Finally, `{ {:., meta, ....` matches a function call. When we recurse from the function body, we need to rewrite
`self.(x - 2)` into `self.(self, x - 2)` and that completes the expansion.

There is debugging code in the macro code to print the expansion, and this is what we create:

```elixir
 r = fn
    (self, x) when x == 1 or x == 0 ->
      x
    self, x ->
      self.(self, x - 2) + self.(self, x - 1)
  end
  fn x -> r.(r, x) end
```

Pretty much what we wrote manually before, and this compiles and runs. The first clause causes an "unused" warning because
the `self` there really should be a `_self` but as the author here I can leave that as an exercise to the reader.

Anaphoric lambda, yay! Now, are we loosing a lot of performance? The reason I switched from factorial to fibonacci is that
the trivial recursive implementation is bad. It is even O(2<sup>n</sup>) bad, people on StackOverflow tell me (and it is true
because that answer got upvotes). Using `:timer.tc/1` and a trivial regular function definition, we can compare:

```elixir
def fib(x) when x == 0 or x == 1, do: x
def fib(x), do: fib(x - 1) + fib(x - 2)
...
IO.inspect :timer.tc(fn ->
  fib.(20)
end)
IO.inspect :timer.tc(fn ->
  fib(20)
end)
```

This very scientific benchmark shows 1.4ms for the anaphoric lambda version and 1.2ms for the regular function. Not bad,
the overhead for the anaphoric lambda doesn't seem to be very high, but O(2<sup>n</sup>) starts showing. Let's push it a bit
and calculate the 40th number instead of the 29th (the `tc/1` function returns a tuple with the time in microseconds and
the function return value):

```
{21719438, 102334155}
{18201677, 102334155}
```

Ouch. 2<sup>n</sup> hits hard. And I wanted to know the 400th Fibonacci number! Can we do better? Yes we can, with an
all-time favorite macro writers tool, "memoization". We sneak in some time-for-memory tradeoff by expanding the macro
so that we only calculate any given fibonacci number once. The whole variable injection stays the same, but we now
introduce a unique key for this particular macro and then return code that stores results in ETS so they only need to
be created once:

```elixir
    ...
    memo_key = :erlang.unique_integer()
    retval = quote do
      r = unquote(block)
      m_r = fn self, x ->
	case :ets.lookup(Memoize, {unquote(memo_key), x}) do
	  [] ->
	    v = r.(self, x)
            :ets.insert(Memoize, { {unquote(memo_key), x}, v})
`	    v
	  [{_key, val}] ->
	    val
	end
      end
      fn x -> m_r.(m_r, x) end # TODO more than one arg
    end
	...
```

This now results in the following expansion:

```elixir
r = fn
    (self, x) when x == 1 or x == 0 ->
      x
    self, x ->
      self.(self, x - 2) + self.(self, x - 1)
  end
  m_r = fn self, x -> case(:ets.lookup(Memoize, {-576460752303423327, x})) do
    [] ->
      v = r.(self, x)
      :ets.insert(Memoize, { {-576460752303423327, x}, v})
      v
    [{_key, val}] ->
      val
  end end
  fn x -> m_r.(m_r, x) end
```

We first lookup in ETS, and if there's a value we return that. We smash O(2<sup>n</sup>) to O(n) and that shows:

```
{14, 102334155}
```

Note that the really nice thing here is that we did not have to touch our code. The `alambda fn ...` is untouched
but we sped it up by, well, a lot of orders of magnitude. Memoization truly can be a miracle!

If you want to use memoization for more serious purposes than this code diversion, there are multiple packages on
Hex that will help you out. The version here
is lacking because there is no way to clean out unused values (you probably want something with a TTL, for example).

Also, if you want to generate fibonacci numbers quickly, here's an O(1) version:

```elixir
def Fibonacci do
  @φ (1 + :math.sqrt(5)) / 2
  @sqrt_5 :math.sqrt(5)
  def fib(x) do
    round(:math.pow(@φ, x) / @sqrt_5)
  end
end
```

It probably shows that math trumps recursion and certainly shows that tricks like anaphoric macros and
memoization are best left in the toolbox until you're sure you need them. But I had fun with them, and that counts for something :)
