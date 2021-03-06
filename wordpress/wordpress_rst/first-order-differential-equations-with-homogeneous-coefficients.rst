First Order Differential Equations with Homogeneous Coefficients
################################################################
:date: 2009-05-31 00:10
:author: asmeurer
:category: Uncategorized
:slug: first-order-differential-equations-with-homogeneous-coefficients

So here is what I have done.  Homogeneous has `several meanings`_, but
when I use it in this post, I will mean `homogeneous functions`_.  We
only care about functions of two variables, so I will also only talk
about them.  A function $latex F(x,y) $ is called homogeneous of order
$latex n$ if $latex F(tx,ty)=t^nF(x,y)$ for some $latex n$.  Also, if we
"pull" an $latex x$ out, we get $latex
F(x,y)=x^nF(1,\\frac{x}{y})=x^nG(\\frac{y}{x})$, which is why the
definition is often stated that the function can be written as a
function of $latex \\frac{y}{x} $.  Of course, we could do the same with
$latex y$ and get a function of $latex \\frac{x}{y}$.  Here is an
example of a homogeneous function: $latex
x^2+y\\sqrt{x^2+y^2}\\sin{\\frac{x}{y}}$.  You can see that $latex
(xt)^2+yt\\sqrt{(xt)^2+(yt)^2}\\sin{\\frac{xt}{yt}}=t^2\\left(x^2+y\\sqrt{x^2+y^2}\\sin{\\frac{x}{y}}\\right)$.

Now, if the coefficients $latex P$ and $latex Q$ in the first order
differential equation $latex P(x,y)dx+Q(x,y)dy=0$ are both homogeneous
functions of the same order, then the substitution $latex x=uy$ or
$latex y=ux$ will make the equation separable.  I will show how to do it
for $latex y=ux$.  We have $latex dy=xdy+udx$ by the product rule.  We
also have $latex P(x,y)=P(x,xu)$ and $latex Q(x,y)=Q(x,xu)$.  Because
these functions are homogeneous, we can "pull out" an $latex x^n$ from
each, giving us $latex x^nP(1,u)dx+x^nQ(1,u)(xdu+udx)=0$.  Because we
made the substitution $latex u=\\frac{y}{x}$, we already have had to
assume $latex x\\neq0$.  Thus, we can divide the whole thing by $latex
x^n$.  Doing this, and expanding the remaining terms, we get $latex
P(1,u)dx+xQ(1,u)du+uQ(1,u)dx=0$.  Refactoring the $latex dx$ and $latex
dy$ terms, we get $latex (P(1,u)+uQ(1,u))dx=-xQ(1,u)du$.  The equation
is separable!  Separating variables and integrating, we get $latex
\\int{\\frac{dx}{x}}=\\int{\\frac{-Q(1,u)du}{P(1,u)+uQ(1,u)}}+C$.  If we
had used $latex x=uy$ instead, we would have gotten $latex
\\int{\\frac{dy}{y}}=\\int{\\frac{-P(u,1)du}{uP(u,1)+Q(u,1)}}+C$.  This
is exactly how I was able to solve these equations in SymPy.  Each
homogeneous equation has two possible integrals, and often the right
hand side of one equation is a much harder integral than the right hand
side of the other.  Therefore, I did them both, and applied a
little heuristic on which one to return.  It prefers expressions that
can be solved for y (f(x) in the case of SymPy), expressions that have
evaluated integrals, and if neither of them are solvable, the shortest
one, which tends to be the simplest.

| Here is an example from my text book.  $latex 2xydx+(x^2+y^2)dy=0$.
 Astute readers of this blog may have noticed that this equation is
`exact`_.  It is easier to solve that way, but we will try using the
methods outlined above.  First, we notice that the coeficients of $latex
dx$ and $latex dy$ are both homogeneous of order 2, so we can make the
substitution $latex x=uy$ or $latex u=yx$ and the equation would become
seperable.  If I were doing this by hand or for a homework assignment, I
would make one of those substitutions and work it through, but we have
the exact form above, so let's use it just like SymPy would.  $latex
P(x,y)=2xy $ and $latex Q(x,y)=x^2+y^2$.  So we get $latex
\\int{\\frac{dx}{x}}=\\int{\\frac{-(1+u^2)du}{2u+u(1+u^2)}}+C$ with
$latex u=\\frac{y}{x}$ and $latex
\\int{\\frac{dy}{y}}=\\int{\\frac{-2udu}{u(2u)+(1+u^2)}}+C$ with $latex
u=\\frac{x}{y}$ (here and for the remainder of this derivation, the
arbitrary constants in the two equations do not necessarily equal each
other until the last step).  Both integrals can be solved  with
a substitution of the denominator, giving, with $latex u$ back
substituted, $latex
\\ln{x}=\\frac{-\\ln{(\\frac{y^3}{x^3}+\\frac{3y}{x})}}{3}+C$ and $latex
\\ln{y}=\\frac{-\\ln{(\\frac{3x^3}{y^3}+1)}}{3}+C$.  If we make $latex
C=ln(C\_1)$, and pull the constants in front of the logs on the right
hand sides in as powers, we can combine everything into logarithms.
 After doing that, we then take the antilog of both sides and get $latex
x=C\_1{(\\frac{y^3}{x^3}+\\frac{3y}{x})}^{-\\frac{1}{3}}$ and $latex
y=C\_1{(\\frac{3x^3}{y^3}+1)}^{-\\frac{1}{3}}$.  We then raise the whole
equation to the $latex -3$ power and make $latex C\_1^{-3}=A$.  We get
|  $latex \\frac{1}{x^3}=A(\\frac{y^3}{x^3}+3\\frac{y}{x})$
|  $latex \\frac{1}{x^3}=A\\frac{y^3+3x^2y}{x^3}$
|  $latex K=y^3+3x^2y$
|  and
|  $latex \\frac{1}{y^3}=A(\\frac{3x^2}{y^2}+1)$
|  $latex \\frac{1}{y}=A(3x^2y+y^2)$
|  $latex K=y^3+3x^2y$,
|  where $latex K=\\frac{1}{A}$

Verify as an exercise that you get the same solution by the exact
differential equation methods (or `don't... you know, depending on which
you'd rather do`_).

Of course, if that was all there was to it, I would have finished in a
day.  There was more.  First, I had to write a function that determines
if an expression is homogeneous and what order if it is. That was about
150 lines of code right there plus tests (150 lines is a lot in Python).

I also wanted it to recognize that $latex \\ln{x}-\\ln{y}$ is
homogeneous because $latex \\ln{x}-\\ln{y}=\\ln{\\frac{x}{y}}$. SymPy
was incapable of combining logarithms like this, so I had to write a
logcombine function, which was another 100 lines of code plus tests.
This came in handy, because I was also able to use it in the part above
where I combined the logarithms of my answer to eliminate all of them.

Lastly, you may have noticed that my math looks much nicer in this post.
It turns out that WordPress supports math using $latex <math>$.
Unfortunately, I did not discover this until I had already uploaded
images of almost all the math expressions in the post, so I had to go
back and fix them.

Also, I was having a problem where the math wouldn't render, but it
turns out that this was an issue with me copying and pasting "$latex"
and the expressions into the WordPress rich text editor. I changed my
editor settings to plain text. I'm sorry, but web rich text editors do
not work. And I've grown accustomed to seeing the source of stuff that I
write having done everything in $latex \\LaTeX$ for a year and having
edited Wikipedia for a while.

I wish there were a free blogging site like WordPress or Blogger that
used MediaWiki or $latex \\LaTeX$ as its rendering engine. It would look
nice, the formatting would be a standard thing that isn't confusing
html, and $latex \\LaTeX$ math would come out much nicer.

.. _several meanings: http://en.wikipedia.org/wiki/Homogeneous_(mathematics)
.. _homogeneous functions: http://en.wikipedia.org/wiki/Homogeneous_function
.. _exact: http://asmeurersympy.wordpress.com/2009/05/16/work-started-exact-differential-equations/
.. _don't... you know, depending on which you'd rather do: http://comics.com/brevity/2006-02-16/
