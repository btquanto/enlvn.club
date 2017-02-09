---
title: Permutation Algorithm
category: Programming Techniques
tags: ["programming", "algorithm", "python"]
descriptions: An algorithm for calculating the permutation of an array
keywords: Permutation Algorithm, Permutation, Python
---

# Permutation

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

def func(permutation):
    print "".join(permutation)

permute(list("ABCDE"), func)
```

# Partial permutation _(k-permutation)_

Applying a function to all arrrangements of k distinct elements selected from a list

``` python
def _partial_permute(prefix, suffix, k, func):
    if len(prefix) == k:
        func(prefix)
    else:
        for i in xrange(len(suffix)):
            _prefix = prefix + [suffix[i]]
            _suffix = suffix[:i] + suffix[i + 1:]
            _partial_permute(_prefix, _suffix, k, func)

def partial_permute(sequence, k, func):
    _partial_permute([], sequence, k, func)

def func(permutation):
    print "".join(permutation)

partial_permute(list("ABCDE"), 3, func)
```
