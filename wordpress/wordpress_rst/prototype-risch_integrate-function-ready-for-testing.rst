Prototype risch_integrate() function ready for testing!
#######################################################
:date: 2010-08-05 22:30
:author: asmeurer
:category: Uncategorized
:slug: prototype-risch_integrate-function-ready-for-testing

So today I finally finished up the prototype function I talked about
`last week`_. The function is called ``risch_integrate()`` and is
available at my `integration3`_ branch. Unlike the inner level functions
I have showcased in `previous`_ `blog posts`_, this function does not
require you to do substitution for dummy variables and manually create a
list of derivatives, etc. All you have to do is pass it a function and
the integration variable, and it will return the result, just like
normal ``integrate()``. I have spent the past few days working on a
monster of a function called ``build_extension()`` that does this
preparsing work for you. The reason that the function was so hard to
write is that the transcendental Risch Algorithm is very picky. *Every*
differential extension has to be transcendental over the previous
extensions. This means that if you have a function like $latex e^x +
e^{\\frac{x}{2}}$, you cannot write this as $latex t\_0 + t\_1$ with
$latex t\_0=e^x$ and $latex t\_1=e^{\\frac{x}{2}}$ because $latex t\_0$
and $latex t\_1$ will each be algebraic over the other ($latex
t\_0=t\_1^2$). You also cannot let $latex t\_0=e^{x}$ and rewrite the
whole integral in terms of $latex t\_0$ because you will get $latex t\_0
+ \\sqrt{t\_0}$, which is an algebraic function. The only way that you
can do it is to let $latex t\_0=e^{\\frac{x}{2}}$, and then your
function will be $latex t\_0^2 + t\_0$.

Now, fortunately, there is an algorithm that provides necessary and
sufficient conditions for determining if an extension is algebraic over
the previous ones. It's called the Risch Structure Theorems. My first
order of business this week was to finish implementing these. This is
actually the reason that I we had to wait until now to get this
prototype function. The Structure Theorems are at the very end of
Bronstein's book, and the integration algorithm is not correct without
them (namely, it is not correct if you add an algebraic extension). I
just recently got to them in my reading. Actually, I skipped some work
on tangent integration so I could get to them first. I hope to talk a
little about them in a future "Risch Integration" blog post, though be
aware that they require some extremely intense algebraic machinery to
prove, so I won't be giving any proofs.

Even though these algorithms can tell me, for example, that I shouldn't
have added $latex t\_0=e^x$ above because it makes $latex
e^{\\frac{x}{2}}=\\sqrt{t\_0}$, that means that I have to go back and
restart my search for an extension so that I can try to get $latex
t\_0=e^{\\frac{x}{2}}$ instead. So I wrote a simple function that takes
the arguments of the exponentials and determines the lowest common
factor. This heuristic saves a lot of time.

I also noticed (actually, Chris Smith inadvertently pointed it out to
me; super thanks to him), that the Structure Theorem algorithms only
tell you if the terms are the same as monomials. It would tell you that
$latex e^x = e^{x + 1}$ because both satisfy $latex Dt=t$. Therefore, I
had to also modify the structure theorem algorithms to pull out any
constant term.

It can still be necessary to restart building the extension even with
the above heuristic. For example, if you have $latex e^x + e^{x^2} +
e^{\\frac{x}{2} + x^2}$, and start with $latex t\_0=e^x$ and $latex
t\_1=e^{x^2}$, then the structure theorems will tell you that $latex
e^{x/2 + x^2} = \\sqrt{t\_0}t\_1$, which we cannot use because of the
radical. The solution it uses is to split it up as $latex e^x + e^{x^2}
+ e^{\\frac{x}{2}}e^{x^2}$ (the structure theorems tell you exactly how
to do this so you are splitting in terms of the other exponentials) and
then restart the extension building entirely. This can be an expensive
operation, because you have to rebuild $latex t\_0$ and $latex t\_1$,
but this time, the heuristic function I wrote from above handles the
$latex e^{\\frac{x}{2}}$ correctly, making $latex
t\_0=e^{\\frac{x}{2}}$, with the final answer $latex t\_0^2 + t\_1 +
t\_0t\_1$. I could have probably made it smarter by only going back to
before the conflicting extensions, but this was quite a bit more work,
and adds more difficulties such as non-trivial relationships, so I just
took the lazy way and restarted completely. It doesn't take *that* much
time.

Of course, sometimes, you cannot add a new exponential, no matter how
you add the extensions. The classic example is $latex
e^{\\frac{\\log{(x)}}{2}}$, which you can see is actually equal to
$latex \\sqrt{x}$, an algebraic function. Therefore, I had to implement
some tricky logic to keep the ``build_extension()`` function from trying
again infinitely. I hope I did it right, so that it never infinite
loops, and never fails when it really can be done. Only time and testing
will tell.

It is exactly the same for logarithms, except in that case, when a new
logarithm is algebraic in terms of old ones, it can be written as a
linear combination of them. This means that there are never any radicals
to worry about, though you do also have to worry about constants. For
example, $latex \\log{(x)}$ looks the same as $latex \\log{(2x)}$
because they both satisfy $latex Dt=\\frac{1}{x}$. An example of a
logarithm that is algebraic over old ones is $latex \\log{(x^2 - 1)}$
over $latex \\log{(x + 1)}$ and $latex \\log{(x - 1)}$, because $latex
\\log{(x^2 - 1)}=\\log{((x + 1)(x - 1))}=\\log{(x + 1)} + \\log{(x -
1)}$.

The parallels between exponentials and logarithms are amazing. For the
structure theorems, the exponential case is exactly the same as the
logarithmic case except replacing addition with multiplication and
multiplication with exponentiation. For the exponential case, you need
the arguments of the already added logarithms to find the algebraic
dependence, and the arguments of the already added exponentials to find
the constant term. For the logarithmic case, you need the arguments of
the already added exponentials to find the algebraic dependence, and the
arguments of the already added logarithms to find the content term.
Everything else is exactly the same, except for the shift in operators.
Of course, I realize why these things are, mathematically, but the
symmetry still amazing to me. I will hopefully explain in more detail in
my future Structure Theorems post.

So onto the ``risch_integrate()`` function. Here is the text that I have
basically put in my `commit message`_, the `aptly numbered issue`_ that
I have created for it, and the `post to the mailing list`_ (it's not so
much that I am lazy as that I was really excited to get this out there).

    I have ready in my integration3 branch a prototype
    risch\_integrate() function that is a user-level function for the
    full Risch Algorithm I have been implementing this summer. Pull from
    h\ `ttp://github.com/asmeurer/sympy/tree/integration3`_.

    This is NOT ready to go in. It is a prototype function that I am
    making available so people can try out the new algorithm and
    hopefully help me to find the bugs in it. Please pass it your
    favorite non-elementary integrals and see if it can determine that
    they are not elementary. If you try to pass it a very crazy function
    at random, the chances are pretty high that it will not be
    elementary. So a better way to test it is to come up with a crazy
    function, then differentiate it. Then pass the derivative and see if
    it can give you your original function back. Note that it will
    probably not look exactly the same as your original function, and
    may differ by a constant. You should verify by differentiating the
    result you get and calling cancel() (or simplify(), but usually
    cancel() is enough) on the difference.

    So you can review the code too, if you like, but just know that
    things are not stable yet, and this isn't strictly a branch for
    review.

    | So far, this function only supports exponentials and logarithms.
    |  Support for trigonometric functions is planned. Algebraic
    functions are
    |  not supported. If the function returns an unevaluated Integral,
    it means
    |  that it has proven the integral to be non-elementary. Note that
    several
    |  cases are still not implemented, so you may get
    NotImplementedError
    |  instead. Eventually, these will all be eliminated, and the only
    |  NotImplementedError you should see from this function is
    |  NotImplementedError("Algebraic extensions are not supported.")

    | This function has not been integrated in any way with the already
    |  existing integrate() yet, and you can use it to compare.

    | Examples:
    |  [code language="py"]
    |  In [1]: risch\_integrate(exp(x\*\*2), x)
    |  Out[1]:
    |  ⌠
    |  ⎮ ⎛ 2⎞
    |  ⎮ ⎝x ⎠
    |  ⎮ ℯ dx
    |  ⌡

    | In [2]: risch\_integrate(x\*\*100\*exp(x), x).diff(x)
    |  Out[2]:
    |  100 x
    |  x ⋅ℯ

    | In [3]: %timeit risch\_integrate(x\*\*100\*exp(x), x).diff(x)
    |  1 loops, best of 3: 270 ms per loop

    | In [4]: integrate(x\*\*100\*exp(x), x)
    |  ... hangs ...

    | In [5]: risch\_integrate(x/log(x), x)
    |  Out[5]:
    |  ⌠
    |  ⎮ x
    |  ⎮ ────── dx
    |  ⎮ log(x)
    |  ⌡

    | In [6]: risch\_integrate(log(x)\*\*10, x).diff(x)
    |  Out[6]:
    |  10
    |  log (x)

    | In [7]: integrate(log(x)\*\*10, x).diff(x)
    |  Out[7]:
    |  10
    |  log (x)

    | In [8]: %timeit risch\_integrate(log(x)\*\*10, x).diff(x)
    |  10 loops, best of 3: 159 ms per loop

    | In [9]: %timeit integrate(log(x)\*\*10, x).diff(x)
    |  1 loops, best of 3: 2.35 s per loop
    |  [/code]

    | Be warned that things are still very buggy and you should always
    verify
    |  results by differentiating. Usually, cancel(diff(result, x) -
    result)
    |  should be enough. This should go to 0.

    So please, please, PLEASE, try out this function and report any bugs
    that you find. It is not necessary to report NotImplementedError
    bugs, because I already know about those (I put them in there), and
    as I mentioned above, they are all planned to disappear. Also, I am
    continually updating my branch with fixes, so you should do a "git
    pull" and try again before you report anything.

    Also, I am aware that there are test failures. This is because I had
    to hack exp.\_eval\_subs() to only do exact substitution (no
    algebraic substitution). It's just a quick hack workaround, and I
    should eventually get a real fix.

    Finally, I'm thinking there needs to be a way to differentiate
    between an unevaluated Integral because the integrator failed and an
    unevaluated Integral because it has proven the integral to be
    non-elementary. Any ideas?

Also, looking at the integral from the previous blog post, you can get
the different results by using the ``handle_log`` argument to
``risch_integrate()``:

If ``handle_first == 'log'`` (the default right now), then it will
gather all logarithms first, and then exponentials (insomuch as it can
do it in that order). If ``handle_first='exp'``, it gathers exponentials
first. The difference is that the Risch Algorithm integrates
recursively, one extension at a time, starting with the outer-most one.
So if you have an expression with both logarithms and exponentials, such
that they do not depend on each other, ``handle_first == 'log'`` will
integrate the exponentials first, because they will be gathered last (be
at the top of the tower of extensions), and ``handle_first == 'exp'``
will integrate the logarithms first. Right now, I have defaulted to
'log' because the exponential integration algorithm is slightly more
complete. If you get ``NotImplementedError`` with one, it is possible
(though I don't know for sure yet) that you might get an answer with the
other.

Also, they can give different looking results, and at different speeds.
For example:

| **Hover over the code and click on the left-most, "view source" icon
(a paper icon with ``< >`` over it) to view without breaks. Opens in a
new window.**
|  [code language="py"]
|  In [1]: f = (x\*(x + 1)\*((x\*\*2\*exp(2\*x\*\*2) - log(x +
1)\*\*2)\*\*2 +
|  ...: 2\*x\*exp(3\*x\*\*2)\*(x - (2\*x\*\*3 + 2\*x\*\*2 + x +
1)\*log(x + 1))))/((x +
|  ...: 1)\*log(x + 1)\*\*2 - (x\*\*3 + x\*\*2)\*exp(2\*x\*\*2))\*\*2

| In [2]: f
|  Out[2]:
|  ⎛ 2 ⎞
|  ⎜⎛ 2⎞ 2⎟
|  ⎜⎜ 2 2 2⋅x ⎟ ⎛ ⎛ 2 3⎞ ⎞ 3⋅x ⎟
|  x⋅(1 + x)⋅⎝⎝- log (1 + x) + x ⋅ℯ ⎠ + 2⋅x⋅⎝x - ⎝1 + x + 2⋅x + 2⋅x
⎠⋅log(1 + x)⎠⋅ℯ ⎠
| 
──────────────────────────────────────────────────────────────────────────────────────────
|  2
|  ⎛ 2⎞
|  ⎜ 2 ⎛ 2 3⎞ 2⋅x ⎟
|  ⎝log (1 + x)⋅(1 + x) - ⎝x + x ⎠⋅ℯ ⎠

| In [3]: risch\_integrate(f, x, handle\_first='log')
|  Out[3]:
|  ⎛ ⎛ 2⎞⎞ ⎛ ⎛ 2⎞⎞
|  ⎜log(1 + x) ⎝x ⎠⎟ ⎜ log(1 + x) ⎝x ⎠⎟ ⎛ 2⎞
|  log⎜────────── + ℯ ⎟ log⎜- ────────── + ℯ ⎟ 2 ⎝x ⎠
|  ⎝ x ⎠ ⎝ x ⎠ x ⋅ℯ ⋅log(1 + x)
|  x + ─────────────────────── - log(1 + x) - ─────────────────────────
+ ──────────────────────────
|  2 2 2
|  2 3 2⋅x
|  - x⋅log (1 + x) + x ⋅ℯ

| In [4]: risch\_integrate(f, x, handle\_first='exp')
|  Out[4]:
|  ⎛ ⎛ 2⎞⎞ ⎛ ⎛ 2⎞⎞ ⎛ 2⎞
|  ⎜ ⎝x ⎠⎟ ⎜ ⎝x ⎠⎟ ⎝x ⎠
|  log⎝log(1 + x) + x⋅ℯ ⎠ log⎝log(1 + x) - x⋅ℯ ⎠ x⋅ℯ ⋅log(1 + x)
|  x + ───────────────────────── - log(1 + x) -
───────────────────────── - ──────────────────────
|  2 2 2
|  2 2 2⋅x
|  log (1 + x) - x ⋅ℯ

| In [5]: %timeit risch\_integrate(f, x, handle\_first='log')
|  1 loops, best of 3: 1.49 s per loop

| In [6]: %timeit risch\_integrate(f, x, handle\_first='exp')
|  1 loops, best of 3: 1.21 s per loop

| In [7]: cancel(risch\_integrate(f, x, handle\_first='log').diff(x) -
f)
|  Out[7]: 0

| In [8]: cancel(risch\_integrate(f, x, handle\_first='exp').diff(x) -
f)
|  Out[8]: 0
|  [/code]

So go now, and pull my `branch`_, and try this function out. And report
any problems that you have back to me, either through the mailing list,
IRC, issue 2010, or as a comment to this blog post (I don't really care
how).

.. _last week: http://asmeurersympy.wordpress.com/2010/07/31/integration-of-primitive-functions/
.. _integration3: http://github.com/asmeurer/sympy/tree/integration3
.. _previous: http://asmeurersympy.wordpress.com/2010/07/31/integration-of-primitive-functions/
.. _blog posts: http://asmeurersympy.wordpress.com/2010/07/12/integration-of-exponential-functions/
.. _commit message: http://github.com/asmeurer/sympy/commit/e3cd5f18f86fd6377836f33f726182c8bd4dc1a0
.. _aptly numbered issue: http://code.google.com/p/sympy/issues/detail?q=2010
.. _post to the mailing list: http://groups.google.com/group/sympy/browse_thread/thread/2464fa764f6f47aa
.. _`ttp://github.com/asmeurer/sympy/tree/integration3`: //github.com/asmeurer/sympy/tree/integration3
.. _branch: //github.com/asmeurer/sympy/tree/integration3
