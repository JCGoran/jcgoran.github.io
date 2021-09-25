---
layout: post
title: "Projectile motion: what is the longest trajectory?"
tags: physics
mathjax: true
---
{% include mathjax_nlb.html %}

<style type="text/css">
kbd.key {
  border-radius: 3px;
  padding: 1px 2px 0;
  border: 1px solid black;
  #background-color: #f2f2f2;
  box-shadow: 0px 1px 2px;
  font-family: "Noto Mono", sans-serif;
}
img.center {
  display: block;
  margin-left: auto;
  margin-right: auto;
}
img + em {
  display: block;
  text-align: center;
}
p {
  page-break-inside: avoid;
}
.MathJax_Display, .MJXc-display, .MathJax_SVG_Display {
    overflow-x: auto;
    overflow-y: hidden;
}
</style>

This is an interesting problem I've stumbled upon while reading up on classical mechanics the other day.
The question goes as follows:

> Suppose that we fire a projectile from the ground with initial velocity $v_0$, at an angle $\varphi$ with the horizontal plane.
> What is the initial angle at which the projectile has the longest trajectory?

## The solution(s)

Glancing at the wording of the problem, we'd conclude that the quantity which we need to maximize is the horizontal distance, but it turns out, it's not, which is why I'll go through two solutions which maximize _different_ quantities.
To keep things simple, we're going to assume our trajectory is parametrized in the following way:

$$
x(t) = v_0 \cos \varphi \, t\\
y(t) = -\frac{1}{2} g t^2 + v_0 \sin \varphi \, t
$$

This is simple enough to prove, assuming that gravity acts in the $-y$ direction, and the initial conditions are such that $x(0) = y(0) = 0$.

### Solution 1: longest horizontal distance (range)

This is the "usual" solution, that is, it's so common that it has its own [Wikipedia article][wiki-range].
Let's briefly go through the derivation.

As a start, we know that the projectile should at some point in time, let's call it $T$, come back to the ground, that is, $y(T) = 0$.
Hence, we first need to solve the equation:

$$
-\frac{1}{2} g T^2 + v_0 \sin \varphi \, T = 0
$$

for $T$.
This is straightforward and yields:

$$
T = \frac{2 v_0 \sin\varphi}{g}
$$

Plugging this back into $x(T)$ gives:

$$
D \equiv x(T) = \frac{2 v_0^2 \sin\varphi \cos\varphi}{g}
=
\frac{v_0^2 \sin (2\varphi)}{g}
$$

where in the second equality I've just used the [double angle formula][wiki-angle].
Now, since we want the greatest range, we need to maximize $x(T)$ w.r.t. the angle $\varphi$, which we obtain by solving:

$$
\frac{\partial x(T)}{\partial \varphi} = 0
$$

which just gives us the equation $\cos (2\varphi_0) = 0$, whose solution in turn is simply $\varphi_0 = \pi / 4 + k \pi / 2$, $k \in \mathbb{Z}$, and, since we want $0 \leq \varphi_0 \leq \pi/2$, we conclude that the only viable solution is $\varphi_0 = \pi / 4$, that is, a $45^\circ$ angle.
We can of course plot the function $f(\varphi) \equiv \sin(2\varphi)$ and its derivative to see this explicitly.

<img src="/assets/plot_f.svg" class="center" width="100%">

Unfortunately, this $45^\circ$ angle is _not_ the angle which maximizes the _length_ of the entire trajectory, so let's solve that problem instead.

### Solution 2: longest total trajectory

The length of some parametric curve $\gamma(t) = \lbrace x(t), y(t)\rbrace$ between $t_1$ and $t_2$ is given by[^arclength]:

$$
L
\equiv
\int_{t_1}^{t_2}
\mathrm{d}t \,
\sqrt{
\left(\frac{\mathrm{d} x}{\mathrm{d} t}\right)^2
+
\left(\frac{\mathrm{d} y}{\mathrm{d} t}\right)^2
}
$$

In our case, we have:

$$
L
=
\int_{0}\limits^{\frac{2 v_0 \sin\varphi}{g}}
\!\!\!\!
\mathrm{d}t \,
\sqrt{
\left(v_0 \cos\varphi\right)^2 + \left(v_0 \sin\varphi - g t\right)^2
}
\\
=
g
\int_{0}\limits^{\frac{2 v_0 \sin\varphi}{g}}
\!\!\!\!
\mathrm{d}t \,
\sqrt{
\left(\frac{v_0}{g} \cos\varphi\right)^2 + \left(t - \frac{v_0}{g} \sin\varphi\right)^2
}
$$

The last integral can be reduced to one of the form:

$$
\int
\mathrm{d}t \,
\sqrt{
t^2 + a^2
}
=
\frac{1}{2}
\left[
a \sqrt{t^2 + a^2}
+
a^2 \log \left(\sqrt{t^2 + a^2} + t\right)
\right]
$$

After some fairly tedious-but-straightforward algebra, we obtain the simple-ish result:

$$
L
=
\frac{v_0^2}{g}
\left[\sin \varphi +\cos ^2\varphi \tanh ^{-1}(\sin
\varphi)\right]
$$

If we maximize this w.r.t. $\varphi$, we get the equation:

$$
-2 \cos \varphi  \left[\sin \varphi  \tanh ^{-1}(\sin \varphi)-1\right]
=
0
$$

Turns out, this equation has two solutions: a seemingly trivial one, $\varphi_1 = \pi / 2$, and a nontrivial one, which must be computed numerically, and has a value $\varphi_2 \approx 	0.985515 = 56.5^\circ$.
We can see what's going on here more clearly if we plot the quantity $h(\varphi) \equiv g\, L / v_0^2$ and its derivative:

<img src="/assets/plot_h.svg" class="center" width="100%">

As we can see from the figure above, the point $\varphi_1 = \pi / 2$ is actually a local minimum while $\varphi_2$ is clearly a global maximum.
Curiously, we _cannot_ apply the second derivative test to $\varphi_1$ since $h''(\varphi_1)$ diverges, but since $h'(\varphi_1 - \varepsilon) < 0$ and $h'(\varphi_1 + \varepsilon) > 0$ for any value $\varepsilon > 0$, we conclude that it's a minimum.

How much shorter is the trajectory that maximizes the horizontal distance than the one that maximizes the total length?
The answer is:

$$
\frac{L(\varphi_0)}{L(\varphi_2)}
\approx
0.956
$$

so by about 5%.
And how much shorter is the horizontal distance (range) of the maximized trajectory?

$$
\frac{D(\varphi_2)}{D(\varphi_0)}
\approx
0.921
$$

so by roughly 8%.
We see that in both cases the difference is less than 10%.

## Conclusion

The takeaway from this short exercise is: always read the problem statement carefully.

### Footnotes

[wiki-range]: https://en.wikipedia.org/wiki/Range_of_a_projectile
[wiki-angle]: https://en.wikipedia.org/wiki/List_of_trigonometric_identities#Double-angle_formulae
[wiki-metric]: https://en.wikipedia.org/wiki/Metric_tensor#Arclength_and_the_line_element
[^arclength]: in general, the length of a parametrized curve on an $n$-dimensional (pseudo-)Riemannian manifold with metric $g$ is [given by][wiki-metric] the following integral: $\int \mathrm{d}t \, \sqrt{g_{ij} \frac{\mathrm{d} x^i}{\mathrm{d} t}\frac{\mathrm{d} x^j}{\mathrm{d} t}}$, which just reduces to the expression above for $n=2$ and $g_{ij} = \delta_{ij}$
