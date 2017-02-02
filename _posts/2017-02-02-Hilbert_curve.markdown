---
layout: post
title:  "What is a Hilbert curve?"
date:   2017-02-02 10:12:20 +0100
---

# Construction

Before I would rigorously define the Hilbert curve I show how to draw the Hilbert curve by example.
To be a little more precise I describe an infinite series of Hilbert curves for every natural number (starting from 0).

Some (not defining) properties of the Hilbert curve:

* The nth Hilbert curve is defined on a 2<sup>n</sup>✕2<sup>n</sup> grid.
* The nth Hilbert curve is an ordered series of 2<sup>2n</sup> points, each points on the grid is visited exactly once.
* Subsequent points are neighbours.
* Each Hilbert curve's first point is the very bottom left point of the grid, the last point is the very bottom right point of the grid.

So by these properties the Hilbert curve is not actually a curve,
	but a series of grid points.
One can visualize it as a curve though by connecting the subsequent grid points with a line.
For example the 2nd iteration of the Hilbert curve looks like this:

![2nd iteration][2nd iter]
{: style="text-align: center;"}

But at this point we don't actually know, how the 2nd iteration looks like, I just pulled that out from thin air.
But we know the four properties above, I represent these buy this "black box":

![2nd iteration black box][2nd box]
{: style="text-align: center;"}

I use an arrow to represent the first and the last point on the curve.
Similarly the "black box" of the 3rd Hilbert curve looks like this:

![3rd iteration black box][3rd box]
{: style="text-align: center;"}

Now I show how to get the 3rd Hilbert curve assuming we already know how to draw the 2nd one.
It basically looks like this:

![3rd iteration from 2nd][3 from 2]
{: style="text-align: center;"}

The idea is that we draw a Hilbert curve in each quadrant in a way that the 1st quadrant's last point and the 2nd quadrant's first point are next to each other.
Note that we have to reflect the 2nd iteration for the 1st and 4th quadrant.
For the 1st quadrant the axis of reflection is the bottom left → top right diagonal, for the 4th quadrant it's the other diagonal.
Why we use reflection and not rotation?
Because the direction of the blue arrows matter.

Now how it looks like if we substitute the 2nd iteration that I showed you before:

![2nd iteration substituted to 3rd receipt][3 substituted] → ![3rd iteration][3rd iter]
{: style="text-align: center;"}

Now instead of pulling the 2nd curve from thin air we can actually do the same for the 2nd curve and the 1st curve. 
The 0th "curve" is defined on a 2<sup>0</sup>✕2<sup>0</sup> (1✕1) grid, it's a single point.
One can start from there.

# Definition

At this point we can try to rigorously define the Hilbert curve.
For that we need to address the grid points.
I choose a coordinate system where the bottom left corner is the origin.

For every $$ n \in \mathbb{N} $$, $$H_n$$ is a function, $$ H_n : \{0,1,\dots,2^{2n}-1\} \rightarrow \{0,1,\dots,2^{n}-1\}^2 $$. We can define $$ H_n $$ recursively.

So for ever $$ i \in {0,1,...,2^{2n}-1} $$, $$ H_n(i)=(x,y) $$ is a coordinate pair, where $$ x,y \in \{0,1,\dots,2^{n}-1\} $$. Notation: $$ H_n(i)_1 = x $$ and $$ H_n(i)_2 = y $$.

We can define $$ H_n $$ recursively.

$$ H_0 : \{0\} \rightarrow \{(0,0)\} $$, $$ H_0(0) = (0,0) $$. 

$$
	\scriptsize
	H_n(i) = 
	  \begin{cases}
			\left(
				H_{n-1}(i)_2,
				H_{n-1}(i)_1
			\right) &
			0 \le i < 2^{2(n-1)}
			\\
			\left(
				H_{n-1}(i-2^{2(n-1)})_1,
				H_{n-1}(i-2^{2(n-1)})_2 + 2^{n-1}
			\right) &
			2^{2(n-1)} \le i < 2 \cdot 2^{2(n-1)}
			\\
			\left(
				H_{n-1}(i-2\cdot2^{2(n-1)})_1 + 2^{n-1},
				H_{n-1}(i-2\cdot2^{2(n-1)})_2 + 2^{n-1}
			\right) &
			2 \cdot 2^{2(n-1)} \le i < 3 \cdot 2^{2(n-1)}
			\\
			\left(
				- H_{n-1}(i-3\cdot2^{2(n-1)})_2 + 2^{n}-1,
				- H_{n-1}(i-3\cdot2^{2(n-1)})_1 + 2^{n-1}-1
			\right) &
			3 \cdot 2^{2(n-1)} \le i < 4 \cdot 2^{2(n-1)}
			\\

		\end{cases}
$$

A little bit dry, isn't it? But it can be straightforwardly implemented as a recursive algorithm.

# Remarks

Actually there are multiple, conflicting, but closely related definitions of the Hilbert curve.
It can be useful to rescale the resulting curves to fit inside a unit square and actually connecting the resulting series of grid points to get a parametric curve  $$ \hat{H}_n: [0,1] \rightarrow [0,1]^2 $$.
What is nice about this series of curves is that the limit $$ \hat{H}(t) = \lim_{n\rightarrow\infty}\hat{H}_n(t) $$ exists, and the resulting curve can be rightfully called *the* Hilbert curve, where $$ \hat{H}_n(t) $$ are just approximations.
 $$ \hat{H}(t) $$ also fully covers the unit square.

[2nd iter]: /assets/2nd_iter.svg
[2nd box]:  /assets/2nd_iter_box.svg
[3rd box]:  /assets/3rd_iter_box.svg
[3 from 2]: /assets/3rd_iter_from_2nd.svg
[3 substituted]: /assets/3rd_iter_from_2nd_substituted.svg
[3rd iter]: /assets/3rd_iter.svg

