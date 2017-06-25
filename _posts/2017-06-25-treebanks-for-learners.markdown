---
layout:     post
title:      "Treebanks for learners"
subtitle:   "Finding simple phrases with XQuery and dependency trees"
date:       2017-06-25 23:00:00
author:     "Neven Jovanović"
header-img: "img/typing-vintage-technology-keyboard.jpg"
permalink: /trreebanks-for-learners/
---

# Treebanks as a learners' corpus

Morphologically annotated corpora are usually seen -- at least in Greek and Latin philology -- as a scholarly resource. But they can be used to teach and learn Greek and Latin as well. We can use *parts* of annotated sentences as material for exercises.

We want to start simple, very simple: let's offer our learners some prepositional phrases and some predicate - object (V - O) sentence fragments. First, we have to find them in the treebank.

My experiment has been done with the PROIEL treebank, for two reasons. First, it is one of the two Greek and Latin treebanks available under a CC license. Second, it contains New Testament in Greek and Latin. Regardless of the religious and moralistic part of the experience, I have to agree with [Dwane Thomas](https://dwanethomas.com/the-vulgate-latin-course/) -- the Vulgate is a great place to find real, but stylistically simple Latin; "start with the easy books before you jump into the hard ones" is always a great advice.

PROIEL has annotated a lot of simple Greek and Latin phrases. And how do we find them? In my case, by writing several XQuery functions. Two, basically identical, XQuery scripts are in the [Discipulus working repository](https://bitbucket.org/nevenjovanovic/discipulus) on Bitbucket -- one to find [prepositional phrases](https://bitbucket.org/nevenjovanovic/discipulus/src/18eb4ecae815001619d661be295287841954360d/scripts/PROIELGetPrepositionalPhrasesRecursive.xq?at=master) and one to find [verbs with objects](https://bitbucket.org/nevenjovanovic/discipulus/src/18eb4ecae815001619d661be295287841954360d/scripts/PROIELGetPredObj.xq?at=master). Here I'll comment on more interesting parts of the scripts.

The structure of PROIEL XML files makes it possible to limit searches to specific books and chapters of the New Testament. We retrieve prepositions using an XPath like this: 

```xquery

db:open("proiel-biblia-tb","latin-nt.xml")/proiel/source/div[@alignment-id="41"]/div[@alignment-id="7"]/sentence/token[@part-of-speech="R-"]

```

Retrieving of words dependent on the preposition is then done by a recursive function `down_tree`:

```xquery

declare function local:down_tree($root){
  let $id := $root/@id
  let $down_t := $root/../*:token[@head-id=$id]
  (: check if the dependent word is itself a head of some other word :)
  return if ($down_t[@head-id]) 
  (: if it is, the function should recursively call itself :)
         then ( $down_t/@form/string() , local:down_tree($down_t) )
  else $down_t/@form/string()
};

Running this function against the Latin part of our `proiel-biblia-tb` database, we learn something about the limitations of annotated treebanks:

```xml

<phr citation-part="MARK 7.21">
  <w>ab</w>
  <w>intus</w>
  <w>de</w>
  <w>corde</w>
  <w>hominum</w>
</phr>
<phr citation-part="MARK 7.1">
  <w>ad</w>
  <w>eum</w>
</phr>
<phr citation-part="MARK 7.31">
  <w>ad</w>
  <w>mare</w>
  <w>Galilaeae</w>
</phr>

<phr citation-part="MARK 7.24">
  <w>in</w>
  <w>fines</w>
  <w>et</w>
  <w>Tyri</w>
  <w>Sidonis</w>
</phr>

<phr citation-part="MARK 7.19">
  <w>in</w>
  <w>non</w>
  <w>cor</w>
  <w>eius</w>
</phr>

<phr citation-part="MARK 7.13">
  <w>per</w>
  <w>traditionem</w>
  <w>vestram</w>
  <w>tradidistis</w>
  <w>quam</w>
</phr>

```

While the results such as the first three are immediately useful, the last three should be modified before we can integrate them in an exercise. The word order is not right. Here I can only agree with James Tauber's insight -- the [treebanks are not a tool for everything we do with language](https://jktauber.com/2017/05/24/comparing-analyses-herodotus/). They simply cannot capture some linguistic features -- for example, those connected with word order: informational structure and stylistic effects.

Now, however, we have a brief and relatively simple XQuery that can pull out mostly meaningful fragments from a dependency treebank. And the same script should work for Greek and Old Church Slavonic, but more about that experiment later.


```
