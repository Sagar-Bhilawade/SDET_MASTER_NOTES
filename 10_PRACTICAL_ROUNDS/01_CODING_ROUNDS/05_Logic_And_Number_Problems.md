# Logic & Number Problems — The Bank

## What it is

The warm-up and logic category: FizzBuzz and its variants, prime checks, factorials, Fibonacci, palindrome numbers, Armstrong numbers, digit manipulation. These are often the *filter* problem — quick to solve, but interviewers watch whether you handle them cleanly and can generalize when they add a twist.

---

## Why it matters

These look trivial, and that's the trap. A messy FizzBuzz or a prime check that loops to `n` instead of `√n` signals you don't think about the obvious optimizations. Conversely, doing the trivial one crisply and then handling the twist ("make it configurable," "now do it for a range") in seconds shows composure and the ability to generalize — which is what they're really checking with a warm-up.

---

## 1. FizzBuzz (still asked as a filter)

```java
for (int i = 1; i <= n; i++) {
    String s = "";
    if (i % 3 == 0) s += "Fizz";
    if (i % 5 == 0) s += "Buzz";
    System.out.println(s.isEmpty() ? String.valueOf(i) : s);
}
```
*The clean version concatenates rather than nesting if/else — that structure generalizes. The twist: "make the divisors and words configurable." Pass a `Map<Integer,String>` and loop it. Being ready for the twist is the actual test.*

---

## 2. Check if a number is prime

```java
static boolean isPrime(int n) {
    if (n < 2) return false;
    for (int i = 2; i * i <= n; i++)   // only up to √n
        if (n % i == 0) return false;
    return true;
}
```
*The optimization: loop to `√n`, not `n` — a factor above √n has a partner below it. Looping to `n` works but reveals you didn't think. Edge: n < 2 is not prime, 2 is prime.*

---

## 3. Factorial (iterative — mention recursion + overflow)

```java
static long factorial(int n) {
    long r = 1;
    for (int i = 2; i <= n; i++) r *= i;
    return r;
}
```
*Iterative avoids stack overflow for large n. But `long` overflows past 20! — mention `BigInteger` for arbitrary size. Edge: 0! = 1.*

---

## 4. Fibonacci (iterative beats recursive)

```java
static long fib(int n) {
    if (n < 2) return n;
    long a = 0, b = 1;
    for (int i = 2; i <= n; i++) { long t = a + b; a = b; b = t; }
    return b;
}
```
*Naive recursion is O(2ⁿ) — exponential, recomputes everything. Iterative is O(n). If they want recursion, mention memoization to make it O(n). This trade-off is the whole point of the question.*

---

## 5. Reverse an integer

```java
static int reverse(int n) {
    int r = 0;
    while (n != 0) { r = r * 10 + n % 10; n /= 10; }
    return r;
}
```
*Digit manipulation with `% 10` and `/ 10`. Edge: negatives, trailing zeros (120 → 21), overflow (check before `r * 10 + digit`).*

---

## 6. Palindrome number (without converting to string)

Reverse it (above) and compare, or two-pointer on digits. *"Do it without string conversion" is the common constraint — reverse-and-compare handles it.*

---

## 7. Sum / count of digits

```java
static int digitSum(int n) {
    int sum = 0;
    n = Math.abs(n);
    while (n > 0) { sum += n % 10; n /= 10; }
    return sum;
}
```
*Edge: negatives (`abs`), zero.*

---

## 8. Armstrong number

*A 3-digit number equal to the sum of cubes of its digits (153 = 1³+5³+3³).* Extract digits, sum the powers, compare. Tests digit extraction + a loop.

## 9. GCD (Euclid's algorithm)

```java
static int gcd(int a, int b) { return b == 0 ? a : gcd(a, b % a == 0 ? 0 : b % a); }
// cleaner: while (b != 0) { int t = b; b = a % b; a = t; } return a;
```
*Euclid's algorithm is elegant and expected — the `a % b` recursion. LCM = `a * b / gcd(a,b)`.*

## 10. Count vowels / consonants, swap case, etc.

Simple loops with a `switch` or a lookup set. Quick to write — the point is clean, bug-free, edge-handled code (empty, mixed case, non-letters).

---

## The number gotchas they probe

| Gotcha | The catch |
|--------|-----------|
| Integer overflow | `int` maxes ~2.1B; factorials/big sums overflow silently. Use `long`/`BigInteger`. |
| Integer division | `5 / 2 == 2` in Java (truncates). Cast to double for real division. |
| Modulo of negatives | `-7 % 3` is `-1` in Java (sign follows dividend), differs from math. |
| `√n` loop bound | `i * i <= n` beats `i <= Math.sqrt(n)` (avoids repeated sqrt + floating point). |
| Off-by-one | Loop bounds `<` vs `<=` — the eternal bug. Trace the boundary. |
| Recursion depth | Deep recursion → StackOverflowError. Prefer iterative for large inputs. |

---

## Common interview Q&A

**Q: "Do it recursively / iteratively."**
Know both for factorial and Fibonacci. Volunteer the trade-off: recursion is cleaner to read, iteration avoids stack overflow and (for Fib) exponential blowup.

**Q: "Now make it handle a range / be configurable."**
The twist. Warm-ups almost always get a follow-up. If your first version was clean and general, the twist is a 30-second edit. If it was a tangled nest of ifs, you're rewriting.

**Q: "What's the complexity?"**
Prime: O(√n). Fibonacci iterative: O(n). Naive recursive Fib: O(2ⁿ) — call that out as the reason not to use it.

---

## Weak areas to watch

- **Don't loop to `n` for primes** — √n or you look like you don't optimize the obvious.
- **Watch overflow** on factorial/big sums — `long` or `BigInteger`, and say why.
- **Integer division and negative modulo** bite in Java — know the exact behavior.
- **Write the generalizable version first.** The twist is coming; clean structure makes it trivial.
- **Trace the boundary** — off-by-one in loop bounds is the most common bug here.

---

## Key takeaway

> **These are composure tests: do the trivial one crisply, handle the twist in seconds, and volunteer the one optimization (√n primes, iterative Fibonacci, overflow guards).** A clean, generalizable first version turns the inevitable follow-up into a quick edit — that readiness is what the warm-up actually measures.
