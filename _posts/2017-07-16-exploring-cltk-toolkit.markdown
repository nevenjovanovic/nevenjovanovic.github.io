---
layout:     post
title:      "Exploring the Classical Language Toolkit (CLTK)"
subtitle:   "How we tokenized Latin sentences using XQuery, Python, and XQuery again"
date:       2017-07-16 23:00:00
author:     "Neven Jovanović"
header-img: "img/pexels-photo-172133.jpeg"
permalink: /exploring-cltk/
---

# What is the Classical Language Toolkit (CLTK)?

This July, thanks to a series of the [Global Philology project](http://www.dh.uni-leipzig.de/wo/projects/global-philology-project/) workshops in Leipzig, I had a chance to learn what is the [Classical Language Toolkit (CLTK)](http://cltk.org/), how it works, and what are ambitions of its developers. They are working hard to offer natural language processing tools for the historical languages of Europe and Asia (at the moment). Their approach and their set of tools is inspired by the [Natural Language Toolkit (NLTK)](http://www.nltk.org/) (an excellent idea -- how many times I've sighed, looking at NLTK, "if only would someone do this for Latin..."). As with the NLTK, to use the CLTK we write Python scripts.

# Natural language processing starts from corpora

Linguists need two things to explore language using computers: a corpus of texts, and analytic tools to collect and generate information about it. CLTK offers both corpora and tools. At the moment, I was able to [count 24 languages](http://docs.cltk.org/en/latest/) for which there are corpora we can work with in CLTK. Greek and Latin seem to have the largest number of individual text collections.

# Latin and the CLTK

Having [CroALa](http://croala.ffzg.unizg.hr), we are not in much need of textual corpora (not until the comparison phase, of course), but we need linguistic tools. I was pleased to see that the most accomplished set of tools CLTK has is the one for Latin.

But, before tackling the whole 5 million words of Croatian Latin, we wanted to start small, to experiment (and to remember how to write Python). There is a polemical treatise by [Nicholas, bishop of Modruš](http://viaf.org/viaf/23432692/) (ca. 1427–1480) called *Defensio ecclesiasticae libertatis* (1479); because my colleague Luka Špoljarić is editing the treatise, and I am assisting with the [digital part of the undertaking](https://github.com/nevenjovanovic/modruski-temrezah) in the framework of the [TeMrežaH project](http://temrezah.ffzg.unizg.hr/), we decided to test the CLTK on our text of the Defensio.

# Tokenizing Defensio ecclesiasticae libertatis

The step I was most interested in was tokenizing (dividing) the text into sentences. For this, CLTK has a sentence tokenizer which is sufficiently described in their documentation. My tasks were as follows:

+ Transform the XML edition of Defensio into a plain text file, separating paragraph-like segments with new lines
+ Read the text file with Python
+ Feed each segment of the file into the CLTK tokenizer
+ Export the result and create another XML file, this time with the text segmented not only into paragraphs, but into sentences as well (to be marked with the TEI s element)

Because I am only rudimentarily acquainted with Python, and because working with XML is much easier with an XML tool, for the first and the last task I used XQuery and BaseX. Moreover, I decided to export the tokenized paragraphs as JSON -- and I realized why the programmers love JSON so: it is easy to get the stuff out of Python with. A bit of reading of the [XQuery 3.1 specification](https://www.w3.org/TR/xpath-functions-31/) -- which I really should have read much earlier -- introduced me to the immensely useful [json-to-xml function](https://www.w3.org/TR/xpath-functions-31/#func-json-to-xml), after which a very simple XQuery script produced what I wanted.

# The works

Enough bragging. Here is what I did this Sunday (I found the Python script template on the internet, so it probably retains commands which I don't need):

```python

#!/usr/bin/env python
import sys, os, traceback, optparse
import time
import re

import json
from cltk.tokenize.sentence import TokenizeSentence

def main ():

    global options, args
    
    # Read in the path to file as argument
    i = sys.argv[1]
    m_dict = []
    tokenizer = TokenizeSentence('latin')
    with open(i) as txt:
        data=txt.read()
        # Each newline marks a paragraph-like object
        modr_list = data.splitlines()
    # Tokenize each paragraph-like object
    # Put the result into the m_dict dictionary
    for c in modr_list:
        t = tokenizer.tokenize_sentences(c)
        m_dict.append(t)
    # Export JSON to file
    f = open('modr-sent.json', 'w')
    f.write(json.dumps(m_dict))
    f.close()
if __name__ == "__main__":
    main()

```

Definitely not rocket science. The result is then imported as XML and marked up with p and s element tags, numbering them in the process:

```xquery

let $f := "modr-sent.json"
let $json := file:read-text($f)
for $p in json-to-xml($json)//*:array/*:array
let $n := count($p/preceding-sibling::*:array) + 1 
return element p { 
attribute n { "p" || $n },
  for $s in $p/*:string
  let $ns := count($s/preceding-sibling::*:string) + 1
  return element s { 
  attribute n { "s" || $ns },
  $s/string() } }

```

As usual, most of the work went into preparation: "cleaning up" the original XML file, removing unnecessary elements, and checking the result, which is now [included in our repository](https://github.com/nevenjovanovic/modruski-temrezah/blob/master/data/nikolamodr01/croala1394999/nikolamodr01.croala1394999.croala-lat3text.txt). But, thanks to that, I found several typos in the text. There are few things a philologist enjoys more than finding typos, believe me.

The result, an XML version of the Defensio edition in which sentences have been automatically marked, is [now in the repository](https://github.com/nevenjovanovic/modruski-temrezah/blob/master/data/nikolamodr01/croala1394999/nikolamodr01.croala1394999.croala-lat4sents.xml) too. It needs to be included into the CTS catalog file and, of course, to be checked by the editors.
