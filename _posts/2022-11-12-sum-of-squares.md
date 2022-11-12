---
layout: post
title: Another way to prove the sum of squares formula
mathjax: true
tags: math
---

{% include mathjax.html %}
<style type="text/css">
p {
  page-break-inside: avoid;
}
.MathJax_Display, .MJXc-display, .MathJax_SVG_Display {
    overflow-x: auto;
    overflow-y: hidden;
}
</style>

One of the things you learn may have learned in your undergrad studies (or sometimes in high school) is the formula for the sum of first $n$ numbers is given by:

$$
\sum\limits_{k = 1}^n k = \frac{n (n + 1)}{2}
$$

The usual proof goes along these lines:

1. observe that the first and the $n$th term have the same sum as the second and the $n-1$st term, and is equal to $n + 1$
2. we can pair up all of them[^pairing], and there are $n/2$ such terms
3. therefore, the total result is $(n + 1) \times n / 2$

So far so good, but what happens if we need to compute the sum of _squares_ of the first $n$ numbers, that is:

$$
\sum\limits_{k = 1}^n k^2 = ?
$$

There seems to be a [myriad of ways][proof-squares] to prove the final result;
however, none of them are very satisfactory from my point of view, and usually involve invoking some _other_ theorem or having a "clever" insight which is a bit hard to make up yourself without already seeing it before.
Therefore, in the below, I will outline a simple proof which just requires a hint of calculus[^remark].

### The idea

The proof will basically rely on the fact that an integral and a sum are closely related; more precisely, that:

$$
\int x^n \mathrm{d}x \propto x^{n + 1}
$$

or in words, that the integral of a polynomial of degree $n$ is a polynomial of degree $n + 1$.
Therefore, it seems reasonable that an analogous statement holds for its discrete couterpart[^polynomial-proof], that is:

$$
\sum\limits_{k=1}^n k^m = \sum\limits_{i = 1}^{m + 1} a_i n^i
$$

### Example 1: sum of first $n$ numbers

To see that this actually works, let's apply it to our sum of first $n$ numbers:

$$
\sum\limits_{k = 1}^n k = A n^2 + B n
$$

There is no constant term in the above since then the sum on the left for $n = 0$ be zero, while the part on the right would be non-zero.
How do we proceed from here, that is, find the actual values of $A$ and $B$?
Easy, assume that mathematical induction works, and go directly to the induction step, that is, expand both sides for $n \rightarrow n + 1$:

$$
\sum\limits_{k = 1}^{n + 1} k
=
\sum\limits_{k = 1}^{n} k + n + 1
=
A (n + 1)^2 + B (n + 1)\\
\Rightarrow
\sum\limits_{k = 1}^{n} k + n + 1
=
An^2 + 2 A n + A + B n + B\\
\Rightarrow
\sum\limits_{k = 1}^{n} k + n + 1
=
(A n^2 + B n) + (2 A n + A + B)
$$

Now we just equate the terms with the various powers of $n$ to get the following system of equations:

$$
2 A = 1, \quad A + B = 1
$$

from which we get the only solution $A = 1/2$, $B = 1/2$.
We can rearrange the terms a bit to get the standard result:

$$
\frac{1}{2} n^2 + \frac{1}{2}n
=
\frac{1}{2}n(n + 1)
$$

### Example 2: sum of squares of first $n$ numbers

This time, we write out our sum as:

$$
\sum\limits_{k = 1}^n k^2
=
A n^3 + B n^2 + C n
$$

Going again to the induction step gives us:

$$
\sum\limits_{k = 1}^{n + 1} k^2
=
\sum\limits_{k = 1}^{n} k^2 + (n + 1)^2
=
A (n + 1)^3 + B (n + 1)^2 + C (n + 1)\\
\Rightarrow
\sum\limits_{k = 1}^{n} k + n^2 + 2n + 1
=
An^3 + 3 A n + 3 A n^2 + A + B n^2 + 2 B n + B + Cn + C\\
\Rightarrow
\sum\limits_{k = 1}^{n} k + n^2 + 2n + 1
=
(A n^3 + B n^2 + C) + [3A n^2 + (3A + 2B + C)n + A + B + C]
$$

From this we obtain the system of equations:

$$
3A = 1\\
3A + 2B = 2\\
A + B + C = 1
$$

which is solved by $A = 1/3$, $B = 1/2$, and $C = 1/6$.
Rearranging the terms again gives:

$$
\frac{1}{3}n^3 + \frac{1}{2}n^2 + \frac{1}{6}n
=
\frac{1}{6}n(2n^2 + 3n + 1)
=
\frac{1}{6}n(n^2 + 2n + 1 + n^2 + n)
\\
=
\frac{1}{6}n[(n + 1)^2 + n(n + 1)]
=
\frac{1}{6}n(n + 1)(n + 1 + n)\\
=
\frac{1}{6}n(n + 1)(2n + 1)
$$

which, by comparing to the [reference][proof-squares], is indeed the desired result.

### Why this works

You may be wondering why this works at all.
Well, the reason is simple: polynomials and sums are both additive, and closed under addition (more technically, polynomials of order $n$ make up a vector space of dimension $n + 1$).
This is why this technique can in principle be used to compute $\sum_{k} k^p$ , where $p$ is any positive integer[^wiki-faulhaber], but fails if $p$ is not an integer.
A simple example is $p = 1/2$; we cannot expand $\sqrt{k + 1}$ in a _finite_ sum of powers in $k$ (we can however do it with a _series_), so the right-hand side also needs to have infinitely many terms, and matching them, then finally putting them back together, becomes an impossible task.

#### Footnotes

[^pairing]: I'm using a bit of hand-waving here, since this fails when $n$ is odd, but the result still works, see for instance [the proofwiki proof](https://proofwiki.org/wiki/Closed_Form_for_Triangular_Numbers#Direct_Proof)
[proof-squares]: https://proofwiki.org/wiki/Sum_of_Sequence_of_Squares
[wiki-faulhaber]: https://en.wikipedia.org/wiki/Faulhaber%27s_formula
[^wiki-faulhaber]: for the curious, the formula for arbitrary integer $p$ is called [Faulhaber's formula][wiki-faulhaber], and involves Bernoulli numbers
[^polynomial-proof]: apparently, the [proof of this statement](https://math.stackexchange.com/q/18983) requires some elements of linear algebra and analysis
[^remark]: whether or not this makes it _simpler_ I leave up to the reader :)
