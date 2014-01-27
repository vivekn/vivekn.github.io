---
layout: post
title: "Efficient spelling correction using BK Trees"
date: 2014-01-27 22:14
comments: true
categories: 
---

I came across an interesting data structure called the BK Tree today. It is useful in coming up with suggestions for spelling correction from a dictionary.

A simple solution for coming up with spelling corrections would be to iterate through all the words in the dictionary and computing the Levenshtein distance with the search term and filtering those words less than some bound. But this would have a complexity of **O(n * m^2)** where **n** is the number of words and **m**, the average length. But words are not randomly distributed across the alphabet space, hence a large number of these computations are useless. The idea behind the BK tree is to construct a k-ary tree, such that only nodes that are within a particular edit distance are searched. This exploits the fact that the Levenshtein distance forms a metric space. A really nice explanation of constructing and searching through a BK tree is given [here](http://nullwords.wordpress.com/2013/03/13/the-bk-tree-a-data-structure-for-spell-checking/). It is not necessary that the metric used should be Levenshtein distance, you can use any other metric as you see fit.

It has been observed that for edit distance 2, using a BK tree you need to search only 5-10% of the nodes of the tree. I hacked up a BK Tree [implementation](https://github.com/vivekn/autocorrect) in Scala, and was able to load the `/usr/share/dict/words` file which contains about 250k words in 5 seconds on my MacBook Air. A search query with edit distance 2 takes about 0.06 seconds. Still, I feel there is room for improvement on the performance front. Suggestions welcome.

<script src="https://gist.github.com/vivekn/8652998.js"></script>