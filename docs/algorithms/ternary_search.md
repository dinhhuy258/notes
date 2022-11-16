# Ternary Search

We are given a function $f(x)$ which is unimodal on an interval $[l, r]$. By unimodal function, we mean one of two behaviors of the function: 
1. The function strictly increases first, reaches a maximum (at a single point or over an interval), and then strictly decreases.
2. The function strictly decreases first, reaches a minimum, and then strictly increases.

The task is to find the maximum of function $f(x)$ on the interval $[l, r]$.

## Algorithm


Consider any 2 points $m_1$, and $m_2$ in this interval: $l < m_1 < m_2 < r$. We evaluate the function at $m_1$ and $m_2$, i.e. find the values of $f(m_1)$ and $f(m_2)$. Now, we get one of three options:

-   $f(m_1) < f(m_2)$

    The desired maximum can not be located on the left side of $m_1$, i.e. on the interval $[l, m_1]$, since either both points $m_1$ and $m_2$ or just $m_1$ belong to the area where the function increases. In either case, this means that we have to search for the maximum in the segment $[m_1, r]$.

-   $f(m_1) > f(m_2)$

    This situation is symmetrical to the previous one: the maximum can not be located on the right side of $m_2$, i.e. on the interval $[m_2, r]$, and the search space is reduced to the segment $[l, m_2]$.

-   $f(m_1) = f(m_2)$

    We can see that either both of these points belong to the area where the value of the function is maximized, or $m_1$ is in the area of increasing values and $m_2$ is in the area of descending values (here we used the strictness of function increasing/decreasing). Thus, the search space is reduced to $[m_1, m_2]$. To simplify the code, this case can be combined with any of the previous cases.

How to pick $m_1$ and $m_2$

$$m_1 = l + \frac{(r - l)}{3}$$

$$m_2 = r - \frac{(r - l)}{3}$$

## Implementation

```cpp
double ternary_search(double l, double r) {
  double eps = 1e-9; // set the error limit here

  while (r - l > eps) {
    double m1 = l + (r - l) / 3;
    double m2 = r - (r - l) / 3;
    double f1 = f(m1); // evaluates the function at m1
    double f2 = f(m2); // evaluates the function at m2
    if (f1 < f2)
      l = m1;
    else
      r = m2;
  }
  return f(l); // return the maximum of f(x) in [l, r]
}
```

## Time complexity

$$T(n) = T({2n}/{3}) + 1 = \Theta(\log n)$$

It can be visualized as follows: every time after evaluating the function at points $m_1$ and $m_2$, we are essentially ignoring about one third of the interval, either the left or right one. Thus the size of the search space is ${2n}/{3}$ of the original one.

Applying [Master's Theorem](https://en.wikipedia.org/wiki/Master_theorem_(analysis_of_algorithms)), we get the desired complexity estimate.
