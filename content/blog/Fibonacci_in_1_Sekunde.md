+++
title = 'Fibonacci in one Second'
date = 2025-03-02
draft = false
math = true
+++

Some time ago, YouTube suggested a [video](https://youtu.be/LXm6ygZ3h7A?si=2RG1Io6w2f44w2tR) in which the creator [Sheafification of G](https://www.youtube.com/@SheafificationOfG) attempted to compute the largest possible Fibonacci number within one second. The video was part of a series exploring different approaches and programming languages. I highly recommend watching these entertaining and engaging videos. Inspired by this, I decided to try the same, but in Java. Not because Java is a widely used language for scientific computing, but simply because I hadn't written Java in a while. My implementation can be found on [GitHub](https://github.com/chriscomputing/fib-java).

## The Naive Approach
The Fibonacci sequence is defined as $fib(n) = fib(n-1) + fib(n-2)$, with $fib(0)= 0, fib(1) = 1$. It is therefore natural to implement Fibonacci recursively. This results in the following pseudocode:

```
fib(n):
  if n == 0:
    return 0
  if n == 1:
    return 1
  return fib(n-2) + fib(n-1)
```

It is easy to see that this algorithm is inefficient. For every computation, all Fibonacci numbers from 0 onwards must be determined, leading to a runtime of $O(exp(n))$.

## Dynamic Programming
The obvious improvement to the recursive variant is to store $fib(n-2)$ and $fib(n-1)$ to save many computational steps. This is a trick that many people intuitively use when calculating a Fibonacci number by hand. In general, the technique of reusing previous results is called dynamic programming.

```
fib(n):
  a = 0
  b = 1
  for i from 1 to n:
    tmp = a + b
    b = a
    a = tmp
  return a
```

One might think this algorithm computes Fibonacci in $O(n)$, as the addition takes place within a simple for-loop. However, this ignores the fact that addition on the bit level also takes linear time. Therefore, this algorithm (when applied to a computer) has a runtime of $O(n²)$.

## Fast Squaring
The previous approach already offers a significant improvement and exhausts the potential for improving asymptotic runtimes in this article. However, this does not mean that we cannot achieve a better result. Fortunately, the following statement holds:

$$
\begin{bmatrix}
    0 & 1 \\ 1 & 1
\end{bmatrix}^n =
\begin{bmatrix}
    F_{n-1} & F_n \\ F_n & F_{n+1}
\end{bmatrix}
$$

Thus, when we exponentiate the matrix $\begin{bmatrix}0 & 1 \\ 1 & 1\end{bmatrix}$ by $n$, we obtain the $n$-th Fibonacci number in the top right and bottom left positions. A proof for this statement can be found [here](https://usaco.guide/plat/matrix-expo?lang=cpp). In particular, combined with [Exponentiation by Squaring](https://en.wikipedia.org/wiki/Exponentiation_by_squaring), it is possible to compute Fibonacci even faster.

```
fib(n):
  M = {{0,1}, {1,1}}
  M = exp_by_squaring(M, n)
  return M[0][1]
```

Exponentiation by Squaring has the advantage that the number of required operations is reduced from $O(n)$ in the dynamic algorithm to $O(log(n))$. However, since the final multiplication still requires $O(n^2)$ bit operations, the asymptotic runtime (for computers) remains the same.

## Implementation in Java
In true OOP tradition, I created a `BaseScheduler` and `BaseCalculator` interface so that concrete implementations can be easily swapped. Schedulers are responsible for finding an $n$ that brings the program's runtime to one second within a threshold of 0.002 seconds in as few steps as possible. This is necessary because I did not want to wait indefinitely for my laptop to test every number from $1$ to $n$. Calculators implement the previously discussed algorithms to compute Fibonacci.

### The Schedulers
I implemented three schedulers. The first, called "Graph Scheduler," increments $n$ by a predefined `STEP_SIZE` until the one-second mark is reached. The second, called "Binary Scheduler," performs a kind of binary search to find the correct $n$ as quickly as possible. This works surprisingly well but can still be improved. That is where the "Ratio Scheduler" comes in. Here is its code:

```java
public class RatioScheduler implements BaseScheduler {

    private final double THRESHOLD;

    public RatioScheduler(double threshold) {
        this.THRESHOLD = threshold;
    }

    public long schedule(double time, long previous, double target) {
        if (this.THRESHOLD > Math.abs(time - target))
            return -1;

        double dampener = 1;
        double factor = Math.max(Math.min(target / time, 2.5), 0.5);

        if (factor < 1) {
            dampener = 1.02;
        } else {
            dampener = 0.98;
        }

        return (long) Math.ceil(factor * previous * dampener);
    }
}
```

### The Calculators
The code for the calculators is either too trivial or too long, and the algorithms have already been discussed. Therefore, I refer again to the [GitHub repository](https://github.com/chriscomputing/fib-java). I had to use `BigInteger` for the implementation since Java’s largest data type, `long`, overflows after the 93rd Fibonacci number. I first implemented the recursive and dynamic algorithms before turning to Fast Squaring. For the latter, I used the [Apache Commons Math Library](https://commons.apache.org/proper/commons-math/), which provides numerous matrix operations. Unfortunately, the library's matrices do not support `BigInteger`, so I had to use `BigFraction` instead, which led to an awkward conversion to `BigInteger`.

## Results
Now for the exciting part—how does Java compare to C? I used [Sheafification of G’s implementations](https://github.com/SheafificationOfG/Fibsonisheaf) for C. Here are the results:

| Algorithm | n | Java | C |
| ----------- | --- | ---- | - |
| Recursive | 39 | 1.096616593s | 0.161663483s |
| Dynamic | 302,463 | 1.630596675s | 0.507222650s |
| Fast Squaring | 2,743,397 | 1.816605545s | 1.511263398s |

It is surprising how much better Fast Squaring performs compared to the dynamic algorithm, despite having the same asymptotic runtime. This once again shows that runtime complexity does not tell the whole story about an algorithm. Unsurprisingly, C outperforms Java across the board.

## Possible Improvements
There are many possible improvements, both in experimental setup and implementation. For example, using a uniform data type instead of a mix of `BigInteger` and `BigFraction` would make comparisons more precise. Furthermore, using a pre-allocated memory block for Fibonacci computations could eliminate significant overhead and allow for even larger $n$ values.

