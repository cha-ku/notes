# WEEK 3

## Greedy algorithms

### Celebration Party problem

> Divide the children into groups such as the age difference between any 2 children is at most one year

**Naive Implementation** - Make all groups, check (maxAge - minAge), verify.

<p>if number of children is <em>n</em>,  the lower bound for running time is &#937;(2<sup>n</sup>)</p>

**Efficient Implementation** - Make a safe move, remove points covered by the safe move, repeat
Implementation for grouping of children - select child of least age, Add 1 to age of child, Move all children in that range to a group, repeat

This has a running time of _O(n)_  if children's ages are sorted

If unsorted, we need to sort the children by age first, which takes a time of _O(n log n)_

Combine that with _O(n)_ we get a running time of  _O(n log n)_
