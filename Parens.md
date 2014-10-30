---
layout: page
title: Parens
permalink: /Parens/
---

In 1960, John McCarthy published a paper called "<a href="http://www-formal.stanford.edu/jmc/recursive/">Recursive Functions of Symbolic Expressions and Their Computation by Machine</a>" that described a programming language called Lisp. The rest is history :)

The formal description of the language culminates in a set of functions that together define "apply" - basically the Lisp interpreter in itself - which Alan Kay once called "the Maxwell equations of software":

>SF If nothing else, Lisp was carefully defined in terms of Lisp.
>
>AK Yes, that was the big revelation to me when I was in graduate school—when I finally understood that the half page of code on the bottom of page 13 of the Lisp  1.5 manual was Lisp in itself. These were “Maxwell’s Equations of Software!” This is the whole world of programming in a few lines that I can put my hand over.

(from <a href="http://queue.acm.org/detail.cfm?id=1039523">A conversation with Alan Kay</a>).

I have always been intrigued by Lisp, but - apart from toying with Emacs configuration and some DSSSL - never had the chance to really use the language. I still don't have, but when perusing the above mentioned <a href="http://www.softwarepreservation.org/projects/LISP/book/LISP%201.5%20Programmers%20Manual.pdf">manual</a> it dawned on me that I could use the manuals definitions as specifications for tests and that it'd be an interesting exercise to implement the language described in the book TDD style.

Which is what I set out to do with the <a href="https://github.com/cdegroot/Parens">Parens</a> project. This is no rock-shattering open source software, it's an intellectual (and, to a certain point, academic) exercise. I will use its Git history in the coming time to write some articles on this site to detail how I went about - at least I hope it will stimulate some discussion (to that end, I will add discussion boxes to relevant pages).

For Parens I chose Scala, mostly because this project is about parsing and I like its parser combinators library a lot. The next article will describe setup (I like sbt a lot less) and the first basic test, parsing atoms.

Next: [Parens 2: Setup and atom](/Parens-2%3A-Setup-and-atom.html)
