---
layout: post
title: The curious case of 3D projectile motion
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
video.center {
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

_Alternatively, how there's appears to be a "phase transition"-like phenomenon for a special case of 3D projectile motion_

What do projectile motion and phase transitions have in common?
More than meets the eye it seems!

Motivated by my [last post][projectile-motion-post], I was kind of curious to see what happens when we try to solve the same problem in 3D.
Since the results are quite ugly and long, I won't post them here, but just the general idea on how to approach the problem.

As a start, we again consider a projectile with velocity $v_0$, but this time thrown in some arbitrary direction $\hat{\mathbf{n}}$, and we have an additional force in the $y$ direction.
A (probably difficult to setup) physical scenario this problem can model would be a particle in mutually orthogonal electric and gravitational fields.
Since we're given the absolute value of the initial velocity, and a direction, it's simplest to work in spherical coordinates.
Furthermore, the initial conditions are such that the projectile is launched from the origin.
The equations of motion are of course:

$$
\ddot x = 0, \quad \ddot y = g_y, \quad \ddot z = - g_z
$$

All of the above gives us the following equations for the trajectory:

$$
x(t) = v_0 \cos\phi \sin\theta\, t
\\
y(t) = \frac{1}{2} g_y t^2 + v_0 \sin\phi \sin\theta\, t
\\
z(t) = - \frac{1}{2} g_z t^2 + v_0 \cos\theta \, t
$$

As the equation for the $z$ component remains identical (save for the definition of the angle, which is complementary to the one in the previous post), the time of flight actually stays the same, $T = 2 v_0 \cos\theta / g_z$.
The longest trajectory is given by maximizing the following integral:

$$
L(\theta, \phi)
=
\int_0^T
\mathrm{d}t\,
\sqrt{\dot x^2 + \dot y^2 + \dot z^2}
$$

The integral above is of exactly the same type as in the previous post, albeit the solution is now much more cumbersome to write down.
There's an interesting simplification though: if we take define $|\mathbf{g}| = \sqrt{\sum_i g_i^2}$, $R_y = g_y / |\mathbf{g}|$, and $R_z = g_z / |\mathbf{g}|$, it turns out that $L$ can be factorized in the following way:

$$
L(\theta, \phi) = f(v_0, |\mathbf{g}|) \, h(\theta, \phi, R_y, R_z)
$$

Additionally, we have the constraint $R_y^2 + R_z^2 = 1$, so in reality $h$ depends only on $R_z$.  
From here on, I will just focus on one particular case, namely $\phi = 0$, which has a very interesting property.
Since we only care about the angular dependence of the above, I'll write down just the rescaled distance, $h(\theta, R_z) = 2 |\mathbf{g}| L /v_0^2$, here:

$$
h(\theta, R_z) = \frac{1}{R_z} \bigg[-R_z^2 \cos (\theta ) \sqrt{\left(\frac{4}{R_z^2}-4\right) \cos ^2(\theta )+1}+2 \cos (\theta ) \sqrt{\left(\frac{4}{R_z^2}-4\right) \cos ^2(\theta )+1}+R_z^2 \cos
   (\theta )+R_z \left(R_z^2 \cos ^2(\theta )-1\right) \left(\log (1-R_z \cos (\theta ))-\log \left(\sqrt{\left(\frac{4}{R_z^2}-4\right) \cos ^2(\theta
   )+1}+\left(\frac{2}{R_z}-R_z\right) \cos (\theta )\right)\right)\bigg]
$$

This enormous equation reduces to a similar-looking one from the previous post for $R_z = 1$, though in this case for the complementary angle.
What's interesting is to see what happens when we vary $R_z$, which is limited to the interval $[0, 1]$:

<img src="/assets/edc9d81c9ec84f4d97037380b8a7456d.svg" class="center">

So, what's so special about the above behavior?
Well, there seems to be a threshold value, $R_z^\text{crit.}$, above which the largest trajectory is achieved by throwing the projectile at some nonzero angle $\theta^\textrm{max.}$, while below this value, the longest trajectory will always be achieved at an angle $\theta = 0$, that is, a completely vertical throw!  
This is very reminiscent of a phase transition (see for instance [the Wiki article][landau-wiki] on Landau theory for an overview), where now $R_z$ plays a role analogous to temperature, $h$ can be considered as the equivalent of the free energy function, and $\theta^\textrm{max.}$, the angle at which the trajectory is maximal, is the order parameter of the system.  
To show that this is indeed very much like a second order phase transition, we can plot the angle $\theta^\textrm{max.}$ as a function of $R_z$:

<img src="/assets/4a25796a7d5849dbbce1d3eeebf15fe6.svg" class="center">

As we can see, while $\theta^\textrm{max.}$ is a continuous function of the "temperature" parameter $R_z$ (so it cannot be a first order transition), there is a "bump" in the graph around some value of $R_z$, at which point $\partial \theta^\textrm{max.} / \partial R_z$ diverges, indicating the existence of a second order (or continuous, since $\theta^\textrm{max.}(R_z)$ is continuous) phase transition.
We can compute this critical value by solving the equation:

$$
\frac{\partial h}{\partial \theta} = 0
$$

by setting $\theta = 0$ and solving for $R_z$, which gives us $R_z^\textrm{crit.} \approx 0.903186$.
I leave the calculation of the [critical exponents][critical-wiki] as an exercise for the reader :)  
Overall, I was quite surprised to see a phase transition-like phenomenon showing up in a in problem involving projectile motion!

As a bonus, here's a video of the largest trajectories for various values of $R_z$;
as you can see, as soon as $R_z > R_z^\textrm{crit.}$, we have a nonzero initial throwing angle (with respect to the $z$ axis):

<video autoplay muted controls loop width="100%" class="center">
 <source src="/assets/trajectories_3d.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video> 
<center><i>Click on the video if it doesn't play automatically</i></center>

<p style="margin-top:0.7cm;">Note that as $R_z \rightarrow 1$, we must have $R_y \rightarrow 0$, and consequently we get a smaller and smaller displacement in the $y$ direction, and for $R_z = 1$ we recover the 2D problem.</p>

[projectile-motion-post]: /2021/09/15/projectile-motion.html
[landau-wiki]: https://en.wikipedia.org/wiki/Landau_theory
[critical-wiki]: https://en.wikipedia.org/wiki/Critical_exponent
