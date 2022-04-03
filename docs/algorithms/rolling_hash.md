# Rolling hash

Hashing is used for efficient comparison of strings by converting them into integers and then comparing those strings on the basis of their integer values.

Rolling hash is used to prevent rehashing the whole string while calculating hash values of the substrings of a given string.
In rolling hash,the new hash value is rapidly calculated given only the old hash value.Using it, two strings can be compared in constant time.

## Formula

In general,the hash H can be defined as:-

H = (c<sub>1</sub>a<sup>k-1</sup> + c<sub>2</sub>a<sup>k-2</sup> + c<sub>3</sub>a<sup>k-3</sup> +...+ c<sub>k</sub>a<sup>0</sup>) % m
where:

- a is a constant
- c<sub>1</sub>, c<sub>2</sub>, ... c<sub>k</sub> are the input characters
- m is a large prime number (eg: 1e9 + 7)

The probability of two random strings colliding is about ≈ 1/m.

The hash value of next substring H<sub>nxt</sub> using rolling hash can be defined as:-

H<sub>nxt</sub> = ((H - c<sub>1</sub>a<sup>k-1</sup>) \* a + c<sub>k+1</sub>a<sup>0</sup>) % m

## Example

Consider the string `abcd` and we have to find the hash values of substrings of this string having length 3 ,i.e.,`abc` and `bcd`.

For simplicity let us take 5 as the base but in actual scenarios we should mod it with a large prime number to avoid overflow.

So,the hash value of first substring,H(abc) :-

```
H(abc) => a*(5^2) + b*(5^1) + c*(5^0) = 97*25 + 98*5 + 99*1 = 3014
```

And the hash value of the second substring,H(bcd) :-

```
H(bcd) => b*(5^2) + c*(5^1) + d*(5^0) = 98*25 + 99*5 + 100*1 = 3045
H(bcd) =(H(abc)-a*(5^2))*5 + d*(5^0)=(3014-97*25)*5 + 100*1 = 3045
```

## Time Complexity

The rolling hash takes **O(n)** time complexity to find hash values of all substrings of a length k of a string of length n.

Computing hash value for the first substring will take **O(k)** as we have to visit every character of the substring and then for each of the n-k characters we can find the hash in **O(1)** so the total time complexity would be **O(k+n-k)**
