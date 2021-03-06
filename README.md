FastGaussQuadrature.jl
=========
[![Build Status](https://travis-ci.org/ajt60gaibb/FastGaussQuadrature.jl.svg?branch=master)](https://travis-ci.org/ajt60gaibb/FastGaussQuadrature.jl) [![codecov](https://codecov.io/gh/ajt60gaibb/FastGaussQuadrature.jl/branch/master/graph/badge.svg)](https://codecov.io/gh/ajt60gaibb/FastGaussQuadrature.jl)

A Julia package to compute `n`-point Gauss quadrature nodes and weights to 16-digit accuracy and in `O(n)` time. So far the package includes `gausschebyshev()`, `gausslegendre()`, `gaussjacobi()`, `gaussradau()`, `gausslobatto()`, `gausslaguerre()`, and `gausshermite()`. This package is heavily influenced by <a href="http://www.chebfun.org">Chebfun</a>.

An introduction to Gauss quadrature can be found <a href="http://en.wikipedia.org/wiki/Gaussian_quadrature">here</a>. For a quirky account on the history of computing Gauss-Legendre quadrature, see <a href="http://math.mit.edu/~ajt/papers/QuadratureEssay.pdf">[6]</a>.

## Our Aims

* The fastest Julia code for Gauss quadrature nodes and weights (without tabulation).

* Change the perception that Gauss quadrature rules are expensive to compute.

## Examples
Here we compute `100000` nodes and weights of the Gauss rules. Try a million or ten million.

```
@time gausschebyshev( 100000 );
0.002681 seconds (9 allocations: 1.526 MB, 228.45% gc time)

@time gausslegendre( 100000 ); 
0.007110 seconds (17 allocations: 2.671 MB)

@time gaussjacobi( 100000, .9, -.1 );
1.782347 seconds (20.84 k allocations: 1.611 GB, 22.89% gc time)

@time gaussradau( 100000 );
1.849520 seconds (741.84 k allocations: 1.625 GB, 22.59% gc time)

@time gausslobatto( 100000 );
1.905083 seconds (819.73 k allocations: 1.626 GB, 23.45% gc time)

@time gausslaguerre( 100000 )
.891567 seconds (115.19 M allocations: 3.540 GB, 3.05% gc time)

@time gausshermite( 100000 );
0.249756 seconds (201.22 k allocations: 131.643 MB, 4.92% gc time)
```

The paper <a href="http://epubs.siam.org/doi/abs/10.1137/140954969">[1]</a> computed a billion Gauss-Legendre nodes. So here we will do a billion + 1. This is (probably) a world record:
```
@time gausslegendre( 1000000001 );
131.392154 seconds (17 allocations: 26.077 GB, 1.17% gc time)
```
(The nodes near the endpoints coalesce in 16-digits of precision.)

## The algorithm for Gauss-Chebyshev
There are four kinds of Gauss-Chebyshev quadrature rules, corresponding to four weight functions:

1. 1st kind, weight function `w(x) = 1/sqrt(1-x^2)`

2. 2nd kind, weight function `w(x) = sqrt(1-x^2)`

3. 3rd kind, weight function `w(x) = sqrt((1+x)/(1-x))`

4. 4th kind, weight function `w(x) = sqrt((1-x)/(1+x))`

They are all have explicit simple formulas for the nodes and weights <a href="http://books.google.com/books?id=8FHf0P3to0UC&lpg=PP1&pg=PA180#v=onepage&q&f=false">[4]</a>.
## The algorithm for Gauss-Legendre
Gauss quadrature for the weight function `w(x) = 1`.

* For `n<=5`: Use an analytic expression.

* For `n<=60`: Use Newton's method to solve `Pn(x)=0`. Evaluate `Pn` and `Pn'` by 3-term recurrence. Weights are related to `Pn'`.

* For `n>60`: Use asymptotic expansions for the Legendre nodes and weights <a href="http://epubs.siam.org/doi/abs/10.1137/140954969">[1]</a>.  

## The algorithm for Gauss-Jacobi
Gauss quadrature for the weight functions `w(x) = (1-x)^a(1+x)^b`, `a,b>-1`.

*  For `n<=100`: Use Newton's method to solve `Pn(x)=0`. Evaluate `Pn` and `Pn'` by three-term recurrence.

*  For `n>100`: Use Newton's method to solve `Pn(x)=0`. Evaluate `Pn` and `Pn'` by an asymptotic expansion (in the interior of `[-1,1]`) and the three-term recurrence `O(n^-2)` close to the endpoints. (This is a small modification to the algorithm described in <a href="http://epubs.siam.org/doi/abs/10.1137/120889873">[3]</a>.)

* For `max(a,b)>5`: Use the Golub-Welsch algorithm requiring `O(n^2)` operations. 

## The algorithm for Gauss-Radau
Gauss quadrature for the weight function `w(x)=1`, except the endpoint `-1` is included as a quadrature node.

The Gauss-Radau nodes and weights can be computed via the `(0,1)` Gauss-Jacobi nodes and weights <a href="http://epubs.siam.org/doi/abs/10.1137/120889873">[3]</a>.

## The algorithm for Gauss-Lobatto
Gauss quadrature for the weight function `w(x)=1`, except the endpoints `-1` and `1` are included as nodes.

The Gauss-Lobatto nodes and weights can be computed via the `(1,1)` Gauss-Jacobi nodes and weights <a href="http://epubs.siam.org/doi/abs/10.1137/120889873">[3]</a>.

## The algorithm for Gauss-Laguerre
Gauss quadrature for the weight function `w(x) = exp(-x)` on `[0,Inf)`

* For `n<128`: Use the Golub-Welsch algorithm. 

* For `method=GLR`: Use the Glaser-Lui-Rohklin algorithm. Evaluate `Ln` and `Ln'` by using Taylor series expansions near roots generated by solving the second-order differential equation that `Ln` satisfies, see <a href="http://epubs.siam.org/doi/pdf/10.1137/06067016X">[2]</a>.

* For `n>=128`: Use a Newton procedure on Riemann-Hilbert asymptotics of Laguerre polynomials, see [5], based on [8]. There are some heuristics to decide which expression to use, it allows a general weight `w(x) = x^alpha exp(-q_m x^m)` and this is O(sqrt(n)) when allowed to stop when the weights are below the smallest positive floating point number.

## The algorithm for Gauss-Hermite
Gauss quadrature for the weight function `w(x) = exp(-x^2)` on the real line.

* For `n<200`: Use Newton's method to solve `Hn(x)=0`. Evaluate `Hn` and `Hn'` by three-term recurrence.

* For `n>=200`: Use Newton's method to solve `Hn(x)=0`. Evaluate `Hn` and `Hn'` by a uniform asymptotic expansion, see <a href="http://arxiv.org/abs/1410.5286">[7]</a>.
*
The paper <a href="http://arxiv.org/abs/1410.5286">[7]</a> also derives an `O(n)` algorithm for generalized Gauss-Hermite nodes and weights associated to weight functions of the form `exp(-V(x))`, where `V(x)` is a real polynomial.

## Example usage


```
@time nodes, weights = gausslegendre( 100000 );
0.007890 seconds (19 allocations: 2.671 MB)

# integrates f(x) = x^2 from -1 to 1
@time dot( weights, nodes.^2 )
0.004264 seconds (7 allocations: 781.484 KB)
0.666666666666666
```

## References:
[1] I. Bogaert, <a href="http://epubs.siam.org/doi/abs/10.1137/140954969">"Iteration-free computation of Gauss-Legendre quadrature nodes and weights"</a>, SIAM J. Sci. Comput., 36(3), A1008-A1026, 2014.
       
[2] A. Glaser, X. Liu, and V. Rokhlin. <a href="http://epubs.siam.org/doi/pdf/10.1137/06067016X">"A fast algorithm for the calculation of the roots of special functions."</a> SIAM J. Sci. Comput., 29 (2007), 1420-1438.

[3] N. Hale and A. Townsend, <a href="http://epubs.siam.org/doi/abs/10.1137/120889873">"Fast and accurate computation of Gauss-Legendre and Gauss-Jacobi quadrature
       nodes and weights"</a>, SIAM J. Sci. Comput., 2012.

[4] J. C. Mason and D. C. Handscomb, <a href="http://books.google.com/books?id=8FHf0P3to0UC&lpg=PP1&dq=Mason%20and%20Handscomb&pg=PP1#v=onepage&q=Mason%20and%20Handscomb&f=false">"Chebyshev Polynomials"</a>, CRC Press, 2002.

[5] P. Opsomer, (in preparation).

[6] A. Townsend, <a href="http://math.mit.edu/~ajt/papers/QuadratureEssay.pdf"> The race for high order Gauss-Legendre quadrature</a>, in SIAM News, March 2015.  

[7] A. Townsend, T. Trogdon, and S. Olver, <a href="http://arxiv.org/abs/1410.5286">"Fast computation of Gauss quadrature nodes and weights on the whole real line"</a>, to appear in IMA Numer. Anal., 2014.

[8] M. Vanlessen, "Strong asymptotics of Laguerre-Type orthogonal polynomials and applications in Random Matrix Theory", Constr. Approx., 25:125-175, 2007.
