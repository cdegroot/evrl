---
layout: post
title:  "Rantware: Hardcoded"
date:   2015-09-26 22:15:42
categories: scala
---

More rantware. Bloated configuration libraries that let you store
configs all over the place, fetch them from URLs or S3 or whatever,
while the smart people behind [the twelve-factor app](http://12factor.net/)
are quite right in simply insisting that configs go through the
environment.

Hence, the second installment of rantware,
[hardcoded](http://github.com/cdegroot/hardcoded). It's an attempt
to minimize what you need for a 12 factor app that is prepared to
run in different environments.

Like [unclever](//evrl.com/scala/2015/09/26/rantware-unclever.html),
hardcoded is a work in progress, details in Github at the link above.
Feedback is very welcome, as pull requests or as constructive
comments in issues.
