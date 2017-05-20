---
layout:     post
title:      "Up and down, prev and next with CTS URNs"
subtitle:   "Thinking about navigation in a scholarly edition"
date:       2017-05-20 12:00:00
author:     "Neven Jovanović"
header-img: "img/navigation.jpg"
permalink: /navigation-xml/
---

# Up and down, prev and next with CTS URNs

If we design a scholarly edition as a set of parallel documents (versions) in which each node is identified by a unique URN, how do we move around in this universe?

Even the standard concepts of "next" and "previous" are not so simple there, because we, as editors, have to decide *what comes next*, actually: is it the next node in the XML document (which can be on the same nesting level of the XML tree, or one level down, or even one level up)? or is it the next [sibling](http://www.xml.com/pub/2000/10/04/transforming/trxml5.html) in the XML document? Not to mention the really intriguing question of the third dimension -- how do we move to the equivalent segment in another version of the text?

Starting with the simpler cases, I have tried to see how to extend the XML index to our XML database (containing different versions of our texts) to include locations of the first preceding and the first following sibling of each node, as well as locations of the parent node and of the first child node (if there are such). The index, recording the last part (the XPath) of the CTS URN and the node ID of the XML database, looks like this:

````xml

<idx>
  <doc n="urn:cts:latinLit:phi0472.phi001.heidelberg-ita1">
    <textgroup>Catullus</textgroup>
    <work>Carmen 53 (in Italian by Mario Ramous)</work>
    (...)
    <cts>
      <urn>div1.53</urn>
      <dbid>308</dbid>
      <ctsprev/>
      <dbidprev/>
      <ctsfol/>
      <dbidfol/>
      <ctsparent>div1</ctsparent>
      <dbidparent>298</dbidparent>
      <ctschild>div1.53.endecasillabo</ctschild>
      <dbidchild>313</dbidchild>
    </cts>
    <cts>
      <urn>div1.53.endecasillabo</urn>
      <dbid>313</dbid>
      <ctsprev/>
      <dbidprev/>
      <ctsfol/>
      <dbidfol/>
      <ctsparent>div1.53</ctsparent>
      <dbidparent>308</dbidparent>
      <ctschild/>
      <dbidchild/>
    </cts>
    <cts>
      <urn>div1.53.head2</urn>
      <dbid>317</dbid>
      <ctsprev/>
      <dbidprev/>
      <ctsfol/>
      <dbidfol/>
      <ctsparent>div1.53</ctsparent>
      <dbidparent>308</dbidparent>
      <ctschild/>
      <dbidchild/>
    </cts>
    <cts>
      <urn>div1.53.1</urn>
      <dbid>321</dbid>
      <ctsprev/>
      <dbidprev/>
      <ctsfol>div1.53.2</ctsfol>
      <dbidfol>325</dbidfol>
      <ctsparent>div1.53</ctsparent>
      <dbidparent>308</dbidparent>
      <ctschild/>
      <dbidchild/>
    </cts>
    <cts>
      <urn>div1.53.2</urn>
      <dbid>325</dbid>
      <ctsprev>div1.53.1</ctsprev>
      <dbidprev>321</dbidprev>
      <ctsfol>div1.53.3</ctsfol>
      <dbidfol>329</dbidfol>
      <ctsparent>div1.53</ctsparent>
      <dbidparent>308</dbidparent>
      <ctschild/>
      <dbidchild/>
    </cts>
    <cts>
      <urn>div1.53.3</urn>
      <dbid>329</dbid>
      <ctsprev>div1.53.2</ctsprev>
      <dbidprev>325</dbidprev>
      <ctsfol>div1.53.4</ctsfol>
      <dbidfol>333</dbidfol>
      <ctsparent>div1.53</ctsparent>
      <dbidparent>308</dbidparent>
      <ctschild/>
      <dbidchild/>
    </cts>
(...)

````

The XQuery script which creates this structure is called[indexRecordPathsPrevFol.xq](https://github.com/nevenjovanovic/catullus-cts/blob/master/scripts/indexRecordPathsPrevFol.xq) in our experimental database with Catullus's poems. When run with a command such as `../../../Apps/basex/bin/basex -v indexRecordPathsPrevFol.xq` (it should be run from the directory in which the script resides), it produces the XML index file. From the file, the XML database indexing the main database can be created. There is also a schema to validate the XML file: [ctsindex.rng](https://github.com/nevenjovanovic/catullus-cts/blob/master/schemas/ctsindex.rng)

This, of course, again adds redundancy to the XML index (and the database created from it, and the whole process...), but makes basic navigation easier and, I hope, faster.

