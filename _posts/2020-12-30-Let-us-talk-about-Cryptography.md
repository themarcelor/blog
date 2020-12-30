---
layout: post
author: Marcelo Costa
---
Let us talk about Cryptography

# Overview

Cryptography is awesome. It is such a fascinating subject that I had to get out of my ~2-year blogging hiatus just to type a bit about it. Here we go:

# How the whole thing started

The etymology behind the word Cryptography is based on its Greek roots: "kryptos" is a super cool word that means "hidden" and the not-so-cool one "graphein", means "to write". Due to the human nature and its dissimulative urges, the ability to hide information became very important over time.

## The Caesar Cipher

The very first example that is mentioned in every Cryptography 101 class is the Caesar cipher, which is basically a method to shift letters according to a 'key' number that dictates how many steps we should move across the alphabet to hide the original letter. For example, assuming we decide the key is going to be "3", then we transform the plain text letter "C" into an "F" (stepping forward 3 alphabet positions: C +1 -> D +1 -> E +1 -> F). The same method can be applied to full words and sentences, e.g.:

```
         +3
       encrypt
         -> 
"CIPHER"     "FLSKHU"
         <-
       decrypt
         -3
```

So this encryption technique was named after Julius Caesar as it was utilized by Roman soldiers to secretely exchange messages between military camps. Only the generals would know the key, which was picked during high rank gatherings and then utilized to encrypt and decrypt messages later on. If any soldier was captured by the enemy, only cipher text would be found in their scrolls.

This idea of having a plain text that is converted to a cipher text using a single key is known as Symmetric Encryption, but we should talk about this later as there are other cool examples to talk about. Things start to get slightly more interesting (and complex) from here :D. 

## The Vignere cipher

Named after some rich French dude called Blaise de Vigenère, this cipher is a pumped-up version of the Caesar's one. It works with a "block of keys" (think of an array of integers) to shift letters simmilarly to the Caesar cipher's approach. Here's an example:

```
key = [ 2, 3, 4] (block size 3)

              _  _  _  _  _  _
plain text -> C  I  P  H  E  R
              ↓  ↓  ↓  ↓  ↓  ↓
             +2 +3 +4 +2 +3 +4
              E  L  T  J  H  V  -> cipher text
```

So how many keys are possible? That would ~(26)^3, assuming the 26-letter alphabet.
As you can see, with this approach, it is a bit harder to "brute-force" the decryption process by trying to guess / infer the original block of keys.

## The Permutation cipher

In a quick glance my research did not show any specific historical information around the origin of such technique. All sources indicate that, just like the other ones, it was elaborated to exchange secret messages between troops during war times. The permutation cipher sets the foundation for other techniques that were utilized to build encryption devices, such as the famous Enigma Machine.
(watch "The Imitation Game" to know more about the history around the Enigma Machine and Alan Turing. Definitely recommended).

But anyway, this one works like this:

Assuming a key is now a Permutation that works in groups of 5 characters (π{1..5}):

```
              x | 1 | 2 | 3 | 4 | 5 |
            ____| _ | _ | _ | _ | _ |
            π(x)| 3 | 4 | 5 | 1 | 2 |
```

A plain text "HELLO" would be converted to "LLOHE".

```
                  1  2  3  4  5
                  H  E  L  L  O
                  L  L  O  H  E
                  3  4  5  1  2
```

What if the sentence does not fit the 5-column pattern for this permutation? Then we can just apply some "padding". For example Let us encrypt the plain text "HELLO YOU". As the 2nd 5-character block is missing letters, it can be padded with X's:

```
                    1st block       2nd block
                  1  2  3  4  5   1  2  3  4  5
                  H  E  L  L  O   Y  O  U  X  X
                  L  L  O  H  E   U  X  X  Y  O 
                  3  4  5  1  2   3  4  5  1  2
```
So, as you can see, the universe of possible keys is now way more vast. In a 32-letter alphabet the number of possibilities are expressed as 32! , which, as per the good ol' factorial calcualtion n! = n * (n-1) * (n-2) ... 1), that results in 2.6313083693369E+35 or 2.6313083693369 * 10^35. Anyway, that's just an awful lot of possible keys.

Here's some Python code to illustrate this method:

```
>>> password = 'marcelo'
>>> key = 'zabcdefghijklmnopqrstuvwxy'
>>> table = {ord('a') + i : ord(k) for i,k in enumerate(key)}
>>> print(table) # illustrate the mapping of unicode values of each character
{97: 122, 98: 97, 99: 98, 100: 99, 101: 100, 102: 101, 103: 102, 104: 103, 105: 104, 106: 105, 107: 106, 108: 107, 109: 108, 110: 109, 111: 110, 112: 111, 113: 112, 114: 113, 115: 114, 116: 115, 117: 116, 118: 117, 119: 118, 120: 119, 121: 120, 122: 121}
>>> password.translate(table)
'lzqbdkn'
```

# Multiplication and prime numbers

The essence of cryptography lies on the idea that is very hard to reverse engineer the result of a multiplication. The inverse of 31 is -31, so it is easy to figure out how to reverse addition operations, but if 31 is the result of a multiplication, it is not easy to figure out how we got there.

But what's up with prime numbers? There are a lot of interesting conjectures regarding prime numbers, there is actually a famous one called "The Goldbach's conjecture", that says that an even number N greater than 2 is the result of a sum of two primes (positive integer M > 2 = <prime> + <prime>). Later another guy called Chen Jingrun stated that a large multiple M is expressible as either the sum of two primes or the sum of a prime and a semiprime (the product of primes). 

```
    9 = 3^2
    15 = 3^5
    30 = 2 * 3 * 5
```

When we express a composite value M as a product of primes, the representation is essentially unique.

```
    15 = 3 * 5
    15 = 5 * 3
```

There are some major epiphanies around this :D

## Euclid's Division Algorithm for positive integers

The Euclidean algorithm, or Euclid's algorithm, is an efficient method for computing the greatest common divisor (GCD) of two integers (numbers), the largest number that divides them both without a remainder.

How it works? Given two positive numbers A and B where A <= B, then we can divide A into B through the following calculation and feed its output to the same process utilizing the remainder, this cycle will repeat iself until the remainder is no longer greater than or equals zero:

```
      In the first round:
      Number B (Divisor) is equal to Q (Quotient) times number A (Dividend) plus the Remainder.
         B = Q * A + R0
         A = Q * R0 + R1
         R0 = Q * R1 + R2
         R1 = Q * R3 + R4
              ...  # repeat while 0 <= R < A
               n

  e.g., 56 / 77 => 77 = 1(56) + 21
        56 / 21 => 56 = 2(21) + 14
        14 / 21 => 56 = 1(14) + 7 # <- here
         7 / 14 => 14 = 2(7) + 0
```

77 and 7 are divisible by 7. Hence, this is the largest divisor of both numbers.

Another example: A = 107 and B = 541. Let us calculate the Greatest Common Divisor (GCD) utilizing the Euclid's algorithm:

```
        541 = 5(107) + 6
        107 = 17(6) + 5
          6 = 1(5) + 1
          5 = 5(1) + 0

        Therefore, GCD(A, B) = 1, where A = 107 and B = 541. When the GCD = 1, that means A and B are relatively prime (to one another).
```

Cool, isn't it? But where are we going with this? We shall connect the dots later. Let's proceed to the next section.

## Clock Arithmetic and modular calculation

Consider the following universe of numbers {0, 1, 2, 3, 4} (that can be expressed as ℤ5), that means that every calculation in this universe will be handled in mod5. That means 3 + 3 = 1 because there's no 6 in the  {0, 1, 2, 3, 4} universe, once it goes beyond 4 we go back to the beginning and, like a clock :). Let us analyze two tables that represent the distribution of all elements from the point of views of "Addition" and "Multiplication".

The addition table:

```
  + | 0 | 1 | 2 | 3 | 4
  _____________________
  0 | 0   1   2   3   4
  1 | 1   2   3   4   0
  2 | 2   3   4   0   1
  3 | 3   4   0   1   2
  4 | 4   0   1   2   3
```

So 2 + ? = 0, which means that 5 - 2 = 3, therefore, 3 is the corresponding negative number. 2 and 3 are reciprocal between each other. If A + B = 0, that means that B is the "additive inverse" of A.

The take away from the addition table is just the idea that there's always an additive inverse, but the same is not true for the discoveries around the multiplication table.

Now, here is the multiplication table (The first index 0 is ignored):

```
  x | 1 | 2 | 3 | 4 
  __________________
  1 | 1   2   3   4
  2 | 2   4   1   3
  3 | 3   1   4   2
  4 | 4   3   2   1
```

Every non-zero value has a corresponding reciprocal. If A * B = 1, that means that B is the "multiplicative inverse" of A (aka: m. inverse). Like 2 * 3 = 1, so 3 is the m. inverse of 2.

Now let us try the same with `ℤ6 = {0, 1, 2, 3, 4, 5}`:

```
  x | 1 | 2 | 3 | 4 | 5 
  ______________________
  1 | 1   2   3   4   5
  2 | 2   4   0   2   4
  3 | 3   0   3   0   3
  4 | 4   2   0   4   2
  5 | 5   4   3   2   1
```

In this scenario, 2, 3 and 4 are considered "pathologicals", but 1 and 5 have multiplicative inverses (1 * 1 = 1 & 5 * 5 = 1).

So, again, Multiplication introduces some interestingly crazy scenarios which are less predictable than the ones based on simple addition.

### Finding the multiplicative inverse of numbers
 
It is time to combine some of the knowledge gathered in the previous sections: Let's say we need to find the multiplicative inverse of 7 in ℤ15, how can we do that without laying down all the elements in this silly table?

Answer: Using the Euclid's algorithm! :) The multiplicative inverse of a given number (K) can be obtained (if it actually exists / if there's no pathology) by calculating the GCD (Greatest Common Divisor) against its universe of numbers (N) and making sure it results in "1". This can be expressed as GCD(K, N) = 1.

```
  GCD(7, 15)
    15 = 2(7) + 1
     7 = 7.1 + 0

  Therefore GCD(7, 15) = 1, which means 7 has a multiplicative inverse.
  This took us only one step / equation so there's not too much complexity in this example.

  This equation can be rewritten as:
    15 - 2(7) = 1
       -2 * 7 = 1 => The m. inverse of 7 is -2
                  => -2 = 13 mod 15
```

Here is another example (slightly more complex):
 - Calculate the multiplicative inverse of 7 in ℤ18 (if it exists):

```
   18 = 2(7) + 4
    7 = 1(4) + 3
    4 = 1(3) + 1 # <- Oh, we have a "1" here, so that means the m. inverse does exist.
    3 = 3(1) + 0
```

Rewrite as:

```
1st equation    18 - 2(7) = 4
2nd equation     7 - 1(4) = 3
3rd equation     4 - 1(3) = 1
```

Now apply this super crazy backwards substitution procedure:

```
     apply the 2nd equation into the 3rd equation (replacing the 3's):
     if 3 = 7 - 1(4), then 4 - 1(3) = 4 - 1 (7 - 1 * 4) = 1
                                      2 * 4 - 1 * 7 = 1

     now apply the first equation into the equation produced above (replacing the 4's):
     if 4 = 18 - 2(7), then 2 * 4 - 1 * 7 = 2 (18 - 2 * 7) - 1 * 7 = 1
                                            2 * 18 - 4 * 7 - 1 * 7
                                            2 * 18 - 5 * 7

                                            - 5 = 1 / 7 = 7 ^-1
                                            since we are working with ℤ18
                                            18 - 5 = 13
                                            The m. inverse of 7 in ℤ18 is 13
```

This is definitely very trippy, so there is a shortcut to help us out.
To calculate the multiplicative inverse of 7 in ℤ18, just try the following brute-force approach:

Calculating 7 ^-1 (multiplicative inverse) in ℤ18:

    7 * 1 = 7
    7 * 2 = 14
    7 * 3 = 3
    7 * 4 = 10
    7 * 5 = 17 # <- this is the same as -1 mod18 (-1 in ℤ18)
    7 * 6 = 6
    7 * 7 = 13
    7 * 8 = 2
    7 * 9 = 9
    7 * 10 = 16
    7 * 11= 5
    7 * 12 = 12
    7 * 13 = 1 # <- Here! That means the multiplicate inverse of 7 in ℤ18 is 13!

Ok, that is enough for this first part.
I guess we have gathered some cool information that will help us understand how all these crazy calculations are actually applied when we utilize encryption in our day-to-day lives. In our next post we should try to put all this knowledge together and dive a bit more into RSA encryption (Rivest–Shamir–Adleman) and other protocols.

C ya! :)
