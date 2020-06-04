---
layout: post
title:  "Using Knuth-Morris-Pratt Algorithm for pattern matching"
date:   2020-06-04 19:00:00 +0200
categories: algorithms
---

Given a pattern, our problem is to find out if the pattern exists within a main string. This kind of problem is also known as a needle in a haystack problem and it's a very important and common in the field of computer science.  

The first solution that might come to mind when trying to solve this problem is the brute-force approach, also called the "Naive" algorithm.

## Naive approach

To illustrate this, let's assume we have a main string `"abcdabefg"` and we want to find out if the pattern `"efg"` occurs inside the string. Simply enough, we can keep two pointers, one that's going to iterate over the main string and another to iterate over the pattern string:

```cpp
// Leetcode's Implement StrStr()
// https://leetcode.com/problems/implement-strstr/
int strStr(string haystack, string needle) {
    // needle is empty
    if (needle.size() == 0)
        return 0;
    
    int needleIdx = 0;
    
    for (int i = 0; i < haystack.size(); i++) {
        if (haystack[i] != needle[needleIdx]) {
            needleIdx = 0;
        } else {
            needleIdx++;
            if (needleIdx == needle.size())
                return i - needleIdx + 1; // returns the index of the first ocurrence
        }
    }
    
    // pattern not found
    return -1;
}
```

This algorithm checks for equality of the first characterm, if it finds a match, then it keeps successively checking for matches, starting over whenever it fails to find a match. However **there is a problem here**. Imagine we have the following input:

```
main string(s): "mississippi"
pattern (p): "issip"
```

The algorithm starts matching on the position `1` (`"i"`) of the main string and finds a match until the position `5` (`"s"`) where it fails to match it with the position `5` of the pattern (`"p"`), it then continues trying to find a match, completely disregarding that it has already passed the beginning of **an actual match** on the position `4`. Lucky for us, we can easily modify the algorithm to solve this issue:

```cpp
int strStr(string haystack, string needle) {
    if (needle.size() == 0)
        return 0;
    
    int needleIdx = 0;
    
    for (int i = 0; i < haystack.size(); i++) {
        if (haystack[i] != needle[needleIdx]) {
            if (needleIdx != 0)
                i = i - needleIdx;
            needleIdx = 0;
        } else {
            needleIdx++;
            if (needleIdx == needle.size())
                return i - needleIdx + 1;
        }
    }
    
    return -1;
}
```

Again, this algorithm checks for equality of the first character, if it finds a match, then it keeps successively checking for matches, however, this time when the algorithm fails to obtain a full match the pointers return to the position where the first matching character was found and continue it from there.

This seems to work, and indeed it does, however, imagine what would happen for the following input:

```
s: aaaaaaaaaab
p: aaab
```

For this input, our algorithm would start matching the main string on every position, only to fail and then resume again, resulting in a runtime complexity of `O(mn)`, in which `n` is the length of the main string and `m` the length of the pattern string.

We can do better than that.

## Solving the problem with the KMP Algorithm

The [Knuth-Morris-Pratt algorithm](https://en.wikipedia.org/wiki/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm) searches for ocurrences of a "word" within a main "text string"


With this approach we end up with a runtime complexity of `O(n)` in the worse case, significantly better than the `O(mn)` of the naive approach.

## Other Resources

Some leetcode exercises about pattern matching that can be solved with the KMP Algorithm:

- [Implement strStr()](https://leetcode.com/problems/implement-strstr/)