Automating the SymPy release process
####################################
:date: 2013-07-07 03:13
:author: asmeurer
:category: Uncategorized
:slug: automating-the-sympy-release-process

So I have just published `SymPy 0.7.3.rc1`_. I'll write a blog post
about the release itself when we release 0.7.3 final, but for now, I
wanted to write about how we managed to automate our release process.

Our story begins back in October of 2012, when I wrote a long winded
`rant`_ to the mailing list about how long it was taking to get the
0.7.2 release out (it took over a month from the time the release branch
was created).

The rant is fun, and I recommend reading it. Here are some quotes

The intro:

    | Now here's a timeline: 0.7.1 was released July 29, 2011, more than
    a year and two months ago. 0.7.0 was released just over a month
    before that, on June 28. 0.6.7 was released March 18, 2010, again
    over a year before 0.7.0. In almost two year's time, we've had three
    releases, and are struggling to get out a fourth. And it's not like
    there were no changes; quite the opposite in fact. If you look at
    SymPy 0.6.6 compared to the current master, it's unbelievable the
    amount of changes that have gone forward in that time. We've had
    |  since then the new polys, at least four completely new submodules
    (combinatorics, sets, differential geometry, and stats), massive
    improvements to integration and special functions, a ton of new
    stuff in the physics module, literally thousands of bug fixes, and
    the list goes on. Each of these changes on it's own was enough to
    warrant a release.

    So in case I didn't make my point, le me state it explicitly: we
    need to release more often. We need to release \*way\* more often.

My views on some of the fundamental (non-technical) issues:

    I think that one other thing that has held back many releases is the
    feeling of "wait, we should put this in the release". The use of a
    release branch has helped keep master moving along independently,
    but there still seems to be the feeling with many branches of, "this
    is a nice feature, it ought to go in the release." My hope is that
    by making the release process smoother, we can release more often,
    and this feeling will go away, because it won't be a big deal if
    something waits until the next release. As far as deprecations go,
    the real issue with them is time, not release numbers. So if we
    deprecate a feature today vs. one month from today, it's not a big
    deal (as opposed to today vs. a year from today), regardless of how
    many versions are in between.

    I read about what GitHub does for their Windows product regarding
    releasing often on their blog:
    https://github.com/blog/1271-how-we-ship-github-for-windows (they
    actually have this philosophy for all their products). One thing
    that they said is, "And by shipping updates so often, there is less
    anxiety about getting a particular feature ready for a particular
    release. If your pull request isn’t ready to be merged in time for
    today’s release, relax. There will be another one soon, so make that
    code shine!" I think that is exactly the point here. Another thing
    that they noted is that automation is the key to doing this, which
    is what I am aiming for with the above point.

My vision:

    Once we start releasing very often (and believe me, this is way down
    the road, but I'm trying to be forward looking here), we can do away
    with release candidates. A release candidate lives in the wild for a
    week before the full release. But if we are capable of releasing
    literally every week, then having release candidates is pointless.
    If a bug slips into a release, we just fix it and it will be in the
    next release.

...

    We should release \*at least\* once a month. I think that if the
    process is automated enough, that this will be very possible (as
    opposed to the current situation, where the release branch lasts
    longer than a month). In times of high activity, we can release more
    often than that (e.g., after a big pull request is merged, we can
    release).

That was October. Today is July. Basically, our release process was way
too long. Half of it was testing stuff, half of it was tedious releasing
stuff (like making tarballs and so on), and half of it was updating
websites.

We have moved all our testing to Travis CI. So now every pull request is
tested, and we can be pretty much assured that master is always passing
the tests. There is still some work to do here (currently Travis CI
doesn't test with external dependencies), but it's mostly a solved
problem.

For updating websites, we conceded that we are not going to update
anything that we don't own. That means no attempting to make Debian or
Sage packages, or updating Wikipedia or Freshmeat. Someone else will do
that (and does anyone even use Freshmeat any more?).

That leaves the releasing itself. It's still a pain, because we have to
make a source tarball, Windows installer, html docs, and pdf docs, and
do them all for both Python 2 and Python 3.

So Ondrej suggested moving to fabric/vagrant. At the SciPy 2013 sprints,
he started working on a fabfile that automates the whole process.
Basically vagrant is a predefined Linux virtual machine that makes it
easy to make everything completely reproducible. Fabric is a tool that
makes it easy to write commands (in Python) that are run on that
machine.

Building the basic stuff was easy, but I want to automate *everything*.
So far, not everything is done yet, but we're getting close. For
example, in addition to building the tarballs, the fabric script checks
the contents of the tarball against ``git ls-files`` to make sure that
nothing is included that shouldn't be or left out accidentally (and,
indeed, we caught some missing files that weren't included in the
tarball, including the README).

You can run all this yourself. Checkout the 0.7.3 branch from SymPy,
then cd into the release directory, and read the README. Basically, you
just install Fabric and Vagrant if you don't have them already, then run

| [code]
|  vagrant up
|  fab vagrant prepare
|  fab vagrant release
|  [/code]

Note that this downloads a 280 MB virtual machine, so it will take some
time to run for the first time. When you do this, the releases are in
the \`release\` directory.

Finally, I uploaded 0.7.3.rc1 to GitHub using the new releases feature.
This is what the release looks like on GitHub, from the user point of
view

|SymPy 0.7.3.rc1|

This is what it looks like to me

|SymPy 0.7.3.rc1 Edit|

GitHub has (obviously) the best interface I've ever seen for this. Of
course, even better would be if there were an API, so that I could
automate this too. But since Google's `announcement`_ that they are
discontinuing downloads, we can no longer upload to Google Code. Our
plan was to just use PyPI, but I am glad that we can have at least one
other location, especially since PyPI is so buggy and unreliable (I
can't even log in, I get a 502).

So please download this release candidate and test it. We espeically
need people to test the Windows installer, since we haven't automated
that part yet (actually, we are considering not making them any more,
especailly given the existence of people like Christoph Gohlke who `make
them`_ for SymPy anyway, but we'll see). The only thing that remains to
be done is to finish writing the `release notes`_. If you made any
contributions to SymPy since the last release, please add them there. Or
if you want to help out, you can go through our pull requests and make
sure that nothing is missing.

.. _SymPy 0.7.3.rc1: https://github.com/sympy/sympy/releases/sympy-0.7.3.rc1
.. _rant: https://groups.google.com/d/msg/sympy/UfNhyFv-oMg/PkwIz32K-lsJ
.. _announcement: http://google-opensource.blogspot.com/2013/05/a-change-to-google-code-download-service.html
.. _make them: http://www.lfd.uci.edu/~gohlke/pythonlibs/#sympy
.. _release notes: https://github.com/sympy/sympy/wiki/release-notes-for-0.7.3

.. |SymPy 0.7.3.rc1| image:: http://asmeurersympy.files.wordpress.com/2013/07/screenshot-2013-07-06-22-05-31.png
   :target: http://asmeurersympy.files.wordpress.com/2013/07/screenshot-2013-07-06-22-05-31.png
.. |SymPy 0.7.3.rc1 Edit| image:: http://asmeurersympy.files.wordpress.com/2013/07/screenshot-2013-07-06-22-08-19.png
   :target: http://asmeurersympy.files.wordpress.com/2013/07/screenshot-2013-07-06-22-08-19.png
