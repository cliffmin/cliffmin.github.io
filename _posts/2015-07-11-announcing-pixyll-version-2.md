---
layout:     post
title:      Biplanar Crossing Number
summary: overview
---

# Definition

_Let cr(G) denote the standard crossing number of a graph G, i.e. the minimum number of crossings of its edges over all possible drawings of G in the plane. For k⩾2, define the k-planar crossing number as..._

![]({{ site.url }}/images/11.png)

_where the minimum is taken over all unions G = G1 ∪ G2._

_And let the thickness of the planar graphs G be expressed as..._

![]({{ site.url }}/images/12.png)

_where {G1,···,Gk} are planar._

So from k, we trivally we have

![]({{ site.url }}/images/two.png)
![]({{ site.url }}/images/3.png)
![]({{ site.url }}/images/4.png)

However..

![]({{ site.url }}/images/5.png)
![]({{ site.url }}/images/6.png)
![]({{ site.url }}/images/7.png)
![]({{ site.url }}/images/9.png)

The (graph-theoretical) thickness is now known for all complete graphs [1,
2, 3, 12, 13], and is given by the following formula:

![]({{ site.url }}/images/10.png)

From this, we can infer that any k with greater than 8 nodes, will have edge crossings in a biplanar graph. 

## Core of the problem

Applications have risiing relevance in a computer network, such a graph can be laid out on a circuit board so that communication channels do not cross, so no insulation is needed to avoid electrical shorts. 

Despite the simple nature of the problem not much is known about this parameter. For example, neither the crossing number of the complete graph cr(Kn)

## Current State

There is known conjecture to be held true for n ≤ 12 and is known to be an upper bound for general n, except for the special cases n = 6,7 where we find the upper bound. Saaty further verified that the upper bound is achieved for p ≤ 10 and Pan and Richter showed that it also is achieved for p = 11 and finally 12.

![]({{ site.url }}/images/conjecture.png)

However, for an applicable amount of nodes, it remains elusive as to the asymptotic trend. 

