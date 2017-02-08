---
layout: page
title: Parens 2 - Setup and atom
---

Before writing code, first get a development environment setup (that's the law. Except when you work in Smalltalk, but that's a topic for a different article). I was already playing with parsers and had some test code lying around, so I used that to quickly get started. A simple build.sbt file:

{% highlight scala %}
name := "Parens"

version := "0.00000000000001"

scalaVersion := "2.9.1"

libraryDependencies ++= Seq(
    "org.scalatest" %% "scalatest"        % "1.6.1" % "test",
    "org.jmock"      % "jmock"            % "2.5.1" % "test",
    "org.jmock"      % "jmock-legacy"     % "2.5.1" % "test",
    "cglib"          % "cglib-nodep"      % "2.1_3" % "test",
    "org.objenesis"  % "objenesis"        % "1.0"   % "test",
    "junit"          % "junit"            % "4.8.2" % "test"
)
{% endhighlight %}

One thing I don't understand about sbt is that then I need to put my SBT eclipse plugin definition in `project/build.sbt`, but when I include it above, it doesn't work... So:

{% highlight scala %}
resolvers += Classpaths.typesafeResolver

addSbtPlugin("com.typesafe.sbteclipse" % "sbteclipse-plugin" % "2.0.0")
{% endhighlight %}

lets met generate an Eclipse project, and then I complete the skeleton with some [borrowed code](https://github.com/cdegroot/Parens/blob/6d2c13dbde0ec9b2997f1474a97a374804bebb48/src/test/scala/parsing/FlatSpecForParsers.scala) serve to act as the basis for getting started. The base class makes it simple to test individual parsers, which is exactly what we're about to do.

The book starts with a definition of an atom, which is basically a chunk of uppercase characters. A first test is quickly created:

{% highlight scala %}
package parsing
import parsing.SymbolicExpressionAst._

class ParsingSymbolicExpressions extends FlatSpecForParsers with SymbolicExpressionParsers {

  "The Sexp parsers" should "parse an atom" in {
    implicit val parserToTest = atom

    parsing("ABCDE") should equal(Atom("ABCDE"))
  }
}
{% endhighlight %}

The implicit is a nice way to set the default parser to test during the test case. As parser combinators are functions, it is very easy to test them step by step - here we just exercise the to-be-created `atom` parser.

At this point, I made one design decision - the introduction of an Atom case class to capture the parsed result. This will likely lead to a big hierarchy forming an AST, so accept the inevitable right away:

{% highlight scala %}
package parsing

object SymbolicExpressionAst {
    sealed abstract class Token
    case class Atom(name: String) extends Token
}
{% endhighlight %}

Not sure why I put the class hierarchy-to-be in an object, but, hey, we can always change it. The test still does not compile, because `atom` is undefined. The book definition is quickly converted to a parser:

{% highlight scala %}
package parsing
import scala.util.parsing.combinator.RegexParsers
import parsing.SymbolicExpressionAst._

trait SymbolicExpressionParsers extends RegexParsers {

  def atom = regex("[A-Z][A-Z0-9]*"r) ^^ { new Atom(_) }
}
{% endhighlight %}

and the test is green. Time to git commit and celebrate!

The next installment will add basic [S-Expression parsing](Parens-3-S-expressions.html).

_this article discusses [this](https://github.com/cdegroot/Parens/tree/6d2c13dbde0ec9b2997f1474a97a374804bebb48) version of the code_
