---
layout: page
title: Parens 3 - S-expressions
---

The [Lisp book](http://www.softwarepreservation.org/projects/LISP/book/LISP%201.5%20Programmers%20Manual.pdf) introduces two languages with the expression formats - the language itself, comprised of symbolic expressions, and a meta language employing meta expressions. My plan is to implement both languages and bootstrap the whole thing from the definitions in the meta language on page 13. 

I stay with the book structure, which follows the definition of Atoms with one for Symbolic Expressions, or S-Exps or sexps - the famous Lisp grammar with all the parenthesis.  However, first I need to finish some tests for the atoms - I committed (and cpublished the previous article ;-)), but the book had more examples and one of the idea is to have all the examples from the book be used as test input. So, quickly add some testcases:

{% highlight scala %}
parsing("APPLE") should equal(Atom("APPLE"))
parsing("EXTRALONGSTRINGOFLETTERS") should equal(Atom("EXTRALONGSTRINGOFLETTERS"))
parsing("A4B66XYZ2") should equal(Atom("A4B66XYZ2"))
{% endhighlight %}

Everything green, so work on symbolic expressions can start. First, make sure that atoms can be parsed.

{% highlight scala %}
// "All S-expressions are built out of atomic symbols and the punctuation marks "(", ")", and ".".
they should "parse an sexp" in {
  implicit val parserToTest = sexp
          
  parsing("ATOM") should equal(Atom("ATOM"))
}
{% endhighlight %}

(To make it easier to connect book and test code, I am quoting from the book before I write a test. Yes, I'm tempted to go all-out literal programming here, but typing in the whole thing would slow me down too much :-))

The test runs green with a very simple parser: 

{% highlight scala %}
def sexp: Parser[BaseSexp] = atom
{% endhighlight %}

and replacing the AST with:

{% highlight scala %}
sealed abstract class BaseSexp
case class Atom(name: String) extends BaseSexp
{% endhighlight %}

On to the meat - we need to parse a sexp. The simplest test (again, from the book), looks like:

{% highlight scala %}
parsing("(A . B)") should equal(Sexp(Atom("A"), Atom("B")))
{% endhighlight %}

and in order to satisfy it, the parser is extended:

{% highlight scala %}
def sexp: Parser[BaseSexp] = "(" ~> sexp ~ "." ~ sexp <~ ")" ^^ { case x~dot~y => Sexp(x, y) } | atom
{% endhighlight %}

This compiles when we create a case class to hold S-expressions, giving:

{% highlight scala %}
sealed abstract class BaseSexp
case class Atom(name: String) extends BaseSexp
case class Sexp(car: BaseSexp, cdr: BaseSexp) extends BaseSexp
{% endhighlight %}

The recursive parser definition is powerful enough to parse the other book examples, so just adding them completes the basic S-exp parser:

{% highlight scala %}
parsing("(A . (B . C))") should equal(Sexp(Atom("A"), Sexp(Atom("B"), Atom("C"))))
parsing("((A1 . A2) . B)") should equal(Sexp(Sexp(Atom("A1"), Atom("A2")), Atom("B")))
parsing("((U . V) . (X . Y))") should equal(Sexp(Sexp(Atom("U"), Atom("V")), Sexp(Atom("X"), Atom("Y"))))
parsing("((U . V) . (X . (Y . Z)))") should 
    equal(Sexp(
            Sexp(Atom("U"), Atom("V")), 
            Sexp(Atom("X"), Sexp(Atom("Y"), Atom("Z"))))) 
{% endhighlight %}
                
It's fun to look back at the full parser definition at this point:

{% highlight scala %}
trait SymbolicExpressionParsers extends RegexParsers {

  def sexp: Parser[BaseSexp] = "(" ~> sexp ~ "." ~ sexp <~ ")" ^^ { case x~dot~y => Sexp(x, y) } | atom
  
  def atom: Parser[Atom]  = regex("[A-Z][A-Z0-9]*"r) ^^ { new Atom(_) }
}
{% endhighlight %}

Just two lines of code implement a working Lisp parser!

The next installment will start with the meta language by implementing the basic function for constructing lists.

_this article discusses [this](https://github.com/cdegroot/Parens/tree/ba6d9470e7df55f32b07077c18c18ca945040789) version of the code_
