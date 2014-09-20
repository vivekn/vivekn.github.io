---
layout: post
title: "An algorithm for trending topics"
date: 2014-09-20 23:04
comments: true
categories: nlp machine-learning
---

In this post I will describe a really simple algorithm for identifying trending items or posts in your application. TF - IDF (term frequency, inverse document frequency) is a technique that was used to rank search results in early search engines. 

Assume that you have a large corpus of text documents and want to search a document containing a certain phrase or set of keywords. With TF-IDF, you need to calculate two quantities for each keyword :- 
1. term frequency - the number of times the keyword appears in a particular document.
2. inverse document frequency - the inverse of the number of documents containing the keyword. 

For each document sum up the TF-IDF values of each of the keywords in the query and then rank them. The reason this works is because the IDF part of it helps filter out common words that are present in tons of documents. So a document containing a rare term is given more weight in the search results. Why term frequency is needed is much more obvious as a document containing more occurrences of a keyword is more likely to be the document you are looking for.

Now, we can extend this algorithm for identifying trending items in a dataset. First, partition the data into two sets, one the target data set containing posts/items in the timeframe/geography for which you want to find the trending items and the rest of the data. Some preprocessing like removing stop words and stripping punctuation would be useful. Now for each of the terms in the target set, find its TF-IDF score and rank the terms. In this case the term frequency will be the number of occurrences in the target set and the IDF will be the number of documents in which the term appears in the other set. Instead of doing this exercise for each and every term, you may also do this on tags/hashtags to save memory usage.

Here is an example script on a dataset containing tweets:

<script src="https://gist.github.com/vivekn/9dfd1f23ce111b12c8ef.js"></script>
