---
layout: post
title: "Roll your own autocomplete solution using Tries"
date: 2012-02-25 17:48
comments: true
categories: algorithms python
---
You might have come across many websites with autocomplete suggestions, most notably Google.

![Google Autocomplete](http://storage.googleapis.com/support-kms-prod/SNP_67CE1E2EEBBDA24AC636DF84A911964629C8_3207874_en_v1)

Adding such an option to your site or application might seem daunting but there is a very simple recursive data structure that solves the problem. There is a ton of literature on the net on how to do this using black box approaches like Lucene, Solr, Sphinx, Redis etc. But all these packages require a lot of configuration and you also lose flexibility. Tries can be implemented in a few lines of code in any language of your choice.

<img align="middle" alt="Trie" height="280" src="http://upload.wikimedia.org/wikipedia/commons/thumb/b/be/Trie_example.svg/400px-Trie_example.svg.png" width="300" />

A trie is basically a tree, with each node representing a letter as illustrated in the figure above. Words are paths along this tree and the root node has no characters associated with it.

<ol><li>The value of each node is the path or the character sequence leading upto it.</li>&#13;
<li>The children at each node are ideally represented using a hash table mapping the next character to the child nodes.</li>&#13;
<li>Its also useful to set a flag at every node to indicate whether a word/phrase ends there.</li>&#13;
</ol>Now we can define methods to insert a string and to search for one.

<script src="https://gist.github.com/1906905.js?file=trie.py"></script>

The insertion cost has a linear relationship with the string length. Now lets define the methods for listing all the strings which start with a certain prefix. The idea is to traverse down to the node representing the prefix and from there do a breadth first search of all the nodes that are descendants of the prefix node. We check if the node is at a word boundary from the flag we defined earlier and append the node's value to the results. A caveat of the recursive approach, is you might run into stack depth problems when the maximum length of a string reaches tens of thousands of characters.

<script src="https://gist.github.com/1906922.js?file=trie.py"></script>

To delete an item,  first traverse down to the leaf of the keyword and then work backwards, checking for common paths.

And there you have it, an efficient way of retrieving strings with a common prefix in less than 40 lines of python code. I also wrote a C++ version using the STL data structures in about 90 lines, the time complexity for this version however is <strong>O(n log n)</strong> as the STL uses a red-black tree implementation for an associative array. Colin Dean has wrote a Ruby version and Marcus McCurdy, a Java one. You can find them all over here - <a href="https://github.com/vivekn/autocomplete">https://github.com/vivekn/autocomplete</a>. Read more about tries at <a href="http://en.wikipedia.org/wiki/Trie" title="Wikipedia" target="_blank">Wikipedia</a>.

<strong>Update: </strong>Thanks to bmahler on reddit for pointing out that the wordlist approach was unnecessary and space inefficient. It has been corrected.

<div style="display: none;">
	trie data structure <br>
	trie algorithm <br>
	trie autocomplete <br>
	autocomplete <br>
	autosuggest <br>
	autocomplete algorithm<br>
	autosuggest algorithm<br>
	trie implementation <br>
	input autocomplete <br>
	autocomplete algorithm <br>
</div>
