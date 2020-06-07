---
layout: post
title:  "Using Knuth-Morris-Pratt Algorithm for pattern matching"
date:   2020-06-07 17:00:00 +0200
categories: algorithms
---

> "Given a pattern, find out if this pattern exists within a main string."

 This kind of problem is very common and important in the field of computer science and there are many ways of solving it. In this post I'm going to talk about two solutions for this problem: the "naive" approach and the Knuth-Morris-Pratt algorithm.

## Naive approach

The first solution that came to my mind when trying to solve this problem was the brute-force approach, also called the "Naive" algorithm. This approach involves iterating over the main string and the pattern at the same time, checking for any possible matches.

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

This algorithm checks for equality of the first character, if it finds a match, then it keeps successively checking for matches, starting over whenever it fails to find a match. However **there is a problem here**. Imagine we have the following input:

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

Again, this algorithm checks for equality of the first character, if it finds a match, then it keeps successively checking for matches, however, this time when the algorithm fails to obtain a full match the pointers backtrack to the position where the first matching character was found and continue it from there.

This seems to work, and indeed it does, however, imagine what would happen for the following input:

```
s: aaaaaaaaaab
p: aaab
```

For this input (and generally for inputs with a small alphabet), our algorithm would start matching the main string on every position, only to fail and then resume again, resulting in a worst case runtime complexity of `O(m*n)`, in which `n` is the length of the main string and `m` the length of the pattern string.

There is, however, a way to solve this linearly.

## Solving the problem with the KMP Algorithm

The [Knuth-Morris-Pratt algorithm](https://en.wikipedia.org/wiki/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm) solves this set-back that happens when our pattern has identical sub-patterns that repeat within it. The idea behind the algorithm is to pre-process our string, identify those repeating sub-patterns and then avoid having to backtrack the whole pattern string when a mismatch occurs.

### The pre-processing

The key to avoiding the backtracking that happens in the naive approach is pre-processing the pattern string and identifying the patterns that happen inside of it. 

What does this mean? Imagine a pattern string _`"abcabdea"`_:

```
pattern: | a | b | c | a | b | d | e | a |
index:   | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 
```

And let's imagine we want to find a match for this pattern on the string _`"abcabcabdea"`_.

We are able to find character matches until our iterator reaches position `5` (_`"d"`_ in the pattern, _`"c"`_ in the main string), that's where we have our first mismatch.

If we take a look at the pattern to the left side of the mismatch _`"abcab"`_, it's possible to identify that the prefix _`"ab"`_ (indexes 0 to 1) is the same as the suffix _`"ab"`_ (indexes 3 to 4), this means we don't have to backtrack to the beginning of the pattern string, only to the latest repetition and then continue from there.

So we can continue the algorithm from the position `2` of the pattern string (letter _`"c"`_).

Knowing that, we can create a table to help us identify those repetions in suffixes and prefixes.

```
pattern: | a | b | c | a | b | d | e | a |
index:   | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 
value:   | 0 | 0 | 0 | 1 | 2 | 0 | 0 | 4 |
```

This table illustrates what we just did, if a mismatch occurs at a certain index, check the value of the index just behind it and continue trying to match from there.

We can easily build this table with the following algorithm:

```cpp
vector<int> preProcess(string s) {
        vector<int> lps(s.size(), 0);
        
        for (int i = 1, j = 0; i < s.size();) {
            if (s[i] == s[j]) {
                lps[i++] = ++j;
            } else if (j != 0) {
                j = lps[j - 1];
            } else {
                lps[i++] = 0;
            }
        }
        
        return lps;
    }
```

This piece of code builds the **longest prefix-suffix** (lps) table, also refered to prefix table, and it does so by iterating two pointers over the string (`i` and `j`). `i` starts from the position `1` and `j` from the position `0`.
We then advance checking for equality, if `s[i]` and `s[j]` are equal, we increment both pointers and save in the lps table the value of `j`.

By doing so, what we are doing is saving the index we will have to return to if a mismatch occurs.

However, if s[i] and s[j] are not equal, we first check to see if there's a pattern we can continue from with `j`, if not we then save the value 0 in the table (meaning the matching algorithm will have to start from the beginning of the pattern string).

### Solving the strStr() problem with the KMP Algorithm

```cpp
int strStr(string haystack, string needle) {
        if (needle.size() == 0) {
            return 0;
        }
        
        vector<int> lps = preProcess(needle);
        
        for (int i = 0, j = 0; i < haystack.size();) {
            if (haystack[i] == needle[j]) {
                i++;
                j++;
                if (j == needle.size())
                    return i - needle.size();
            } else if (j != 0) {
                j = lps[j - 1];
            } else {
                i++;
                j = 0;
            }
        }
        
        return -1;
    }
```

With this approach we end up with a runtime complexity of **`O(m+n)`** in the worse case, in which `m` is the size of the pattern string and `n` the size of the main string. However, it's worth noting that as a drawback for the reduced runtime complexity, the KMP algorithm raises the space complexity to `O(m)` (since we now have to store the pre-processed pattern string).

## Other Resources

Some leetcode exercises about pattern matching that can be solved with the KMP Algorithm:

- [Implement strStr()](https://leetcode.com/problems/implement-strstr/)
- [Shortest Palindrome](https://leetcode.com/problems/shortest-palindrome/)