---
layout:     post
title:      "A simple SPARQL query for LiLa"
subtitle:   "How to find the label of a given URI in a collection of Latin lemmata."
date:       2022-03-08 15:30:00
author:     "Neven Jovanović"
header-img: "img/pexels-rodnae-productions-8224406.jpg"
---

The [LiLa (Linking Latin)](https://lila-erc.eu/sparql/) is a ERC project that constructed an important part of infrastructure for computational analysis of Latin: they created a publicly accessible, well designed bank of Latin lemmata and lexical information about them. In LiLa, each lemma is uniquely described by a URI, an address of the entry in their database.

We want to use a particular URI to retrieve its label, to actually know which word the URI points to.

The URI is: [http://lila-erc.eu/data/id/lemma/4450].

There are two interfaces for humans to search the LiLa lemma bank. One of them is very intuitive, but the other is better suited for complex computational tasks. That other interface uses the query language SPARQL. In it, the task "Find word which the URI points to" translates to:

```sparql

PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

# Given a URI (e.g., ""http://lila-erc.eu/data/id/lemma/4450""), retrieve label

SELECT ?lemma
{
      <http://lila-erc.eu/data/id/lemma/4450> rdfs:label ?lemma .
}
```

Try it yourself on the [LiLa SPARQL interface](https://lila-erc.eu/sparql/).

(Image: RODNAE Productions on Pexels.)
