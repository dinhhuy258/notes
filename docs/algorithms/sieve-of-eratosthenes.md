# Sieve of Eratosthenes

Sieve of Eratosthenes is an algorithm for finding all the prime numbers in a segment [1..n] using O(nloglogn).

**Algorithm Steps:**

```cpp
std::vector<bool> primes(n, true);
primes[1] = false;
for (int i = 2; i * i < n; ++i) {
  if (!primes[i]) {
    continue;
  }

  for (int j = i * i; j < n; j = j + i) {
    primes[j] = false;
  }
}
```

**Time Complexity:** O(nloglogn)
