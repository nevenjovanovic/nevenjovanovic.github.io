---
layout:     post
title:      "Reading supported by dependency trees, a prototype"
subtitle:   "Treebank annotations transformed into a reading page which shows dependencies between words in sentences."
date:       2020-01-02 12:00:00
author:     "Neven Jovanović"
header-img: "img/pan.jpg"
---

# Using dependency tree fragments: a prototype

The hypothesis is that we can understand a Greek sentence easier if we can check any word in it and see which word it is dependent on, and which word is governed by it.

Here is a prototype of such fragmentary display of dependency tree annotations in something resembling a reading environment (with tooltips showing lemmata):

* [Xenophon, Hellenica 1, 1-4](http://croala.ffzg.unizg.hr/grcfrag/xen-hell-1-1-4.html)
* [Josephus Flavius, Bellum Judaicum 1, 11-15](http://croala.ffzg.unizg.hr/grcfrag/josephus-BJ-1-11-15.html)
* [Aeschines 1, 51-100](http://croala.ffzg.unizg.hr/grcfrag/aeschines1-51-100.html)
* [Aristoteles Politica, 2](http://croala.ffzg.unizg.hr/grcfrag/arist-pol-2.html)

![A commented selection from Josephus Flavius, BJ 1, 11]({{ site.url }}/img/treebank-fragments2020-01-02 16-36-52.png "A selection from Josephus Flavius, Bellum Judaicum, 1, 11")


The treebank annotations I have used come from the wonderful [treebank collection of Vanessa Gorman](https://zenodo.org/record/3596076). The texts are published under a CC license by the Perseus Digital Library.

The ALDT XML was transformed by the XQuery script [transformALDTtoHTMLwithHighlight.xq](https://github.com/nevenjovanovic/explore-treebanks-xquery/blob/master/xq/transformALDTtoHTMLwithHighlight.xq). 

Some jQuery was used for (very simple) highlighting functions. 

Lemmata are shown by the [PowerTip jQuery plugin](https://stevenbenner.github.io/jquery-powertip/). 

The presentation was inspired by [Christopher Blackwell's work](http://folio2.furman.edu/readingKit/).

