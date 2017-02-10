---
title: Calculating permutations with recursion
category: Programming Techniques
tags: ["programming", "algorithm", "python"]
descriptions: An algorithm for calculating the permutation of an array
keywords: Permutation Algorithm, Permutation, Python
---

# The problem

I happen to come accross this programming problem, which I find a bit challenging.

>Tom is playing his favorite adventure game and got stuck in a puzzle. This puzzle requires him
>to press a​ sequence of buttons to unlock a door. However, the game does not tell Tom the
>button sequence directly but via a encoded string.
>
>Tom knows that one unique character​ of the encoded string must correspond to ​one button​.
>However, he has no further clues on the mapping, i.e. which character in the string is which
>button. As a result, Tom has to try all the possible mappings to find the correct button sequence.
>
> **For example:**
>
> The sequence is **AABBABAB**, and there are 3 buttons, labelled **1**, **2**, **3**
>
> There are 2 uniques characters: **A**, and **B**. There are 3 buttons **1**, **2**, and **3**. So, there are only 6 possible mappings:
>
> * 11221212 # A = 1, B = 2
> * 11331313 # A = 1, B = 3
> * 22112121 # A = 2, B = 1
> * 22332323 # A = 2, B = 3
> * 33113131 # A = 3, B = 1
> * 33223232 # A = 3, B = 2

At first, I thought _"Ok, this is easy. I have to try all the cases. This is just brute-forcing, so I just need a few for loops"_, but then I realized it wasn't that simple.

The number of possible mappings is the _k-permutation_ of the buttons, in which _k_ is the number of unique characters in the sequence.

While it's totally possible to implement the permutation algorithm without recursion, I find the recursion algorithm much easier to understand.

# Algorithm

The problem was an example of a interview question for senior software engineer, in which the candidate is required to write the test in C/C++. However, I long have forsaken C/C++, thus I am going to write the algorithm this in Python.

Personally, I find Python much more friendly to algorithm learning than C/C++.

## Partial permutation _(k-permutation)_

Applying a function to all arrrangements of k distinct elements selected from a list

``` python
def _partial_permute(prefix, suffix, k, func):
    if len(prefix) == k:
        func(prefix)
    else:
        for i, c in enumerate(suffix):
            _prefix = prefix + [c]
            _suffix = suffix[:i] + suffix[i + 1:]
            _partial_permute(_prefix, _suffix, k, func)

def _partial_permute(sequence, k, func):
    _partial_permute([], sequence, k, func)
```

**Example usage**

``` python
def func(permutation):
    print "".join(permutation)

partial_permute(list("ABCDE"), 3, func)
```

## Full permutation

Applying a function to all permutions of elements in a list

``` python
def _permute(prefix, suffix, func):
    if not suffix:
        func(prefix)
    else:
        for i in xrange(len(suffix)):
            _prefix = prefix + [suffix[i]]
            _suffix = suffix[:i] + suffix[i + 1:]
            _permute(_prefix, _suffix, func)

def permute(sequence, func):
    _permute([], sequence, func)
```

**Notes:** The full permutation can as well be written as the _k-permutation_ of the list, in which _k_ is the length of the list.

``` python
def permute(sequence, func):
    partial_permute(sequence, len(sequence), func)
```

**Example usage**

``` python
def func(permutation):
    print "".join(permutation)

permute(list("ABCDE"), func)
```

# The solution

``` python
def _partial_permute(prefix, suffix, k, func):
    if len(prefix) == k:
        func(prefix)
    else:
        for i, c in enumerate(suffix):
            _prefix = prefix + [c]
            _suffix = suffix[:i] + suffix[i + 1:]
            _partial_permute(_prefix, _suffix, k, func)

def partial_permute(sequence, k, func):
    _partial_permute([], sequence, k, func)

def print_possible_mapping(sequence, n):
    sequence = list(sequence)
    uniques = list(set(sequence))

    if n < len(uniques):
        print "There are more buttons than the number of unique characters in the sequence"
        return

    buttons = range(1, n + 1)

    def func(buttons_permutation):
        charmap = {}
        for i, c in enumerate(uniques):
            charmap[c] = buttons_permutation[i]
        _sequence = "".join([str(charmap[c]) for c in sequence])
        print _sequence

    partial_permute(buttons, len(uniques), func)

# Test
print "------ Test 1 ------"
print_possible_mapping("AABBABAB", 2)
print "------ Test 2 ------"
print_possible_mapping("AABBABAB", 3)
print "------ Test 3 ------"
print_possible_mapping("EFEBBBFF", 3)
```
