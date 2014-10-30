---
layout: post
title:  "How to skin a snake using docker"
date:   2014-10-29 19:59:15
categories: docker
---

The snake being Python, of course.

The situation is as follows: I write some code in Python, on Mac
OS X, and the target system is Debian Squeeze. Now, how to distribute
my code to Squeeze? Needless to say, I'm spoiled by all the libraries
available in Python, so my code quickly grows a list of dependencies,
which may or may not be available as Debian packages and may or not
come as native extensions. We don't want to pull stuff from PiPy
in production (security reasons), and we're not a very big Python
shop so setting up our own Python-specific infrastructure is a bit
expensive.  Hence, the stated target is Debian packages and versioning
through Puppet.

_The art of the doable_ is accepting the given and making the best
of it. Preferably through a makefile, because I am getting old and keep
forgetting arcane command line invocations. And preferably portable, so
we don't depend on a specific system to do builds. So, thinking about a
portable, predictable system I quickly landed on Docker. 

(In case you haven't heard of [Docker](https://docs.docker.com/)
yet, please crawl out of your cave and google for it. I'll wait... ;-))

So, my thinking is to have a Dockerfile that mimicks the target
environment and can build Python packages. Instead of Squeeze I opt
for Ubuntu 10.04 (binary compatible. I just happen to prefer Ubuntu),
and the file just installs everything I think I'll need for creating
Debian packages from Python:

{% highlight bash %}
FROM ubuntu:10.04

ENV DEBIAN_FRONTEND noninteractive

RUN sudo apt-get update
RUN sudo apt-get install -y python-stdeb git-core python-pip
python-virtualenv python2.6-dev ruby1.9.1 rubygems1.9.1 ruby1.9.1-dev
libhttpclient-ruby1.9.1
ENV RUBYOPT -E utf-8
RUN gem1.9.1 install fpm && ln -s /var/lib/gems/1.9.1/bin/fpm /usr/bin/

VOLUME ["/data"]

CMD "/bin/bash"
{% endhighlight %}

(The RUBYOPT bit was necessary because of issues around installing FPM -
Ruby 1.9 still thinks ASCII rules the world and FPM speaks UTF-8 in the Gem). 

Build it, tag it as, say, casedeg/python-dist:10.04-1, and a push later we're
the proud owners of a Squeeze-compatible package building container.

That was simple - we now have a Docker container which should, in
theory, be able to create debian packages out of thin air, mostly thanks
to the awesomeness of [FPM](https://github.com/jordansissel/fpm). The
next step, of course, is to use it. As I said, I keep forgetting things
so everything goes into a Makefile for my projects. First, make sure we
have a proper list of dependencies, up-to-date all the time. 

{% highlight makefile %}
test:
	pip freeze >requirements.txt  # this is what we tested with...
	python -m unittest discover
{% endhighlight %}

This keeps requirements.txt up-to-date as I make a habit of running
tests before a release (and, of course, we can "force" that if we want
to). Requirements.txt ends up containing a list of all dependencies,
fully recursed down, which nicely describes the packages we need. 

Now comes the magic. We run Docker in a Vagrant VM, and one of the
things we have in our Vagrantfile is:

{% highlight ruby %}
    config.vm.synced_folder ENV['HOME'], ENV['HOME']
{% endhighlight %}

In this way, the home directory on Mac OS X is mirrored in the Linux VM
and thus you can pretend you can mount "local" Mac OS X directories
directly in Docker containers. This allows us to do the following:

{% highlight makefile %}
PACKAGER_DOCKER=casedeg/python-dist:10.04-1
VERSION:=$(shell python setup.py --version)

dist: test
    # Package dependencies
    docker run -i -t --rm -v ${PWD}:/data ${PACKAGER_DOCKER} /bin/bash \
        -c 'cd /data; cat requirements.txt |grep ==|while read p; do fpm -s python -t deb $$p;done'
    # Package this
    docker run -i -t --rm -v ${PWD}:/data ${PACKAGER_DOCKER} /bin/bash \
        -c "cd /data; fpm -s python -t deb -v ${VERSION} setup.py"
{% endhighlight %}

The first statement goes through all the requirements that have a
double equals - essentially skipping over the line that pip freeze emits
to represent the local source code. It spits out Debian packages in the
current directory - /data - which happens to match the source code
directory we're invoking this from through a) the Vagrant mapping, and
b) the '-v' volume mapping given to the Docker. It ends with a .deb
dropped in our source directory for each dependency, recursively. 

The second one fires up the container again, now invoking FPM to
make a .deb out of the source code and setup.py. The end result is
a Debian package from the source tree. FPM will copy all the
dependencies into the Debian packages, so all you need to do now
is throw the whole bunch into your APT repository and apt-get install
the package representing your source code - the rest will be pulled
in automatically.

The really cool thing is that in essence, you instantiate the
equivalent of a virtual machine for each step in that Makefile snippet,
without any of the overhead. A sample run:

{% highlight bash %}
$ time gnumake dist
# Package dependencies
docker run -i -t --rm -v /Users/casedeg/dev/counters:/data casedeg/python-dist:10.04-1 /bin/bash -c 'cd /data; cat requirements.txt |grep ==|while read p; do fpm -s python -t deb $p;done'
Created package {:path=>"python-argparse_1.2.1_all.deb"}
Created package {:path=>"python-ecdsa_0.11_all.deb"}
Created package {:path=>"python-paramiko_1.15.1_all.deb"}
Created package {:path=>"python-pycrypto_2.6.1_amd64.deb"}
Created package {:path=>"python-pyjavaproperties_0.6_all.deb"}
Created package {:path=>"python-pysftp_0.2.8_all.deb"}
Created package {:path=>"python-requests_2.4.3_all.deb"}
Created package {:path=>"python-scp_0.8.0_all.deb"}
Created package {:path=>"python-wsgiref_0.1.2_all.deb"}
# Package this
docker run -i -t --rm -v /Users/casedeg/dev/counters:/data casedeg/python-dist:10.04-1 /bin/bash -c "cd /data; fpm -s python -t deb -v 1.0.0 setup.py"
Created package {:path=>"python-counters_1.0.0_all.deb"}

real    0m43.312s
user    0m0.119s
sys 0m0.058s
{% endhighlight %}

More than acceptable for a make dist spitting out 10 Debian packages, I'd say :)

### Conclusion

There is more than one way to skin this particular snake. What I like
about the Docker solution is that, in effect, the Dockerfile describes
the standardized environment you usually setup in an integration
environment (e.g., "distributions can only be created on this Jenkins
machine"), but now it becomes fully portable - I can run this command on
Linux, Windows, Mac OS X, no matter what versions, the packages will be
built on exactly the same environment and thus can be relied upon to be
correct. Furthermore, no hardware needs to be dedicated - it gets fired
up when we need it, and forgotten milliseconds afterwards. 

I think building distributions using Docker is a huge step forwards, and
after this pretty succesful experiment I'm sure I'll be pushing for more
of this in our company.
