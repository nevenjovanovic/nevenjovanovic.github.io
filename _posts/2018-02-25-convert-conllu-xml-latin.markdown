---
layout:     post
title:      "Convert Quintilian's sentences from CONLL-U to XML"
subtitle:   "Use Giuseppe Celano's lemmatized Latin corpus to prepare teaching materials"
date:       2018-02-25 23:00:00
author:     "Neven Jovanović"
header-img: "img/cocoon-butterfly-insect-animal-39862.jpeg"
permalink: /convert-conllu-xml-latin/
---

# The lemmatized Quintilian

The Latin Stylistics course at the University of Zagreb has a reading list which includes, among other texts, Quintilian's Book 10 of the Institutio oratoria. To prepare a number of exercises and a word list for the text, we wanted to use (and explore) the [lemmatized corpus of Latin texts](https://github.com/gcelano/LatinUD) prepared by Giuseppe Celano and published on Github.

The lemmatized texts, however, are not quite straightforward to use and process, because they are published in the [CONLL-U format](http://universaldependencies.org/docs/format.html), not XML but a specialised linguistic format (with metadata solutions) from which even CSV has to be derived first. There are no ready-made tools for converting from CONLL-U to more general formats such as CSV or XML. The CONLL-U format itself is well documented, though.

## The plan

Our initial plan was: extract Book 10 from the `phi1002.phi001.perseus-lat2.xml` file of the LatinUD repository; split the book into smaller files, a separate one for each sentence; remove the commented fields (starting with `#`), and convert the rest, which is in CSV format using tabs as field separators, into XML using the Basex's CSV module.

As usual, the process involved manual and automatic actions.

## The process

First, we located the first and the last sentence of Quintilian's Book 10, and discarded everything else. The resulting file (`phi1002.phi001.perseus-lat2:10.txt`) was split using the [csplit command](https://www.gnu.org/software/coreutils/manual/html_node/csplit-invocation.html) from GNU Coreutils. The file was split on empty lines:

```bash

csplit --prefix=phi1002.phi001.perseus-lat2:10. --digits=4 phi1002.phi001.perseus-lat2:10.txt /^$/ {*}

```

Now, we had a directory of 754 text files, each holding an individual sentence from the Book 10 (nice information for my students, who have to read 754 sentences of Quintilian). BaseX will read the files one by one, convert them to XML, and import into an XML database. The program will be in XQuery.

Before writing the program, we had to remove the comments (in lines starting with `#`). This bash one-liner did that, calling `sed`:

```bash

for f in `ls -1`; do sed -i '/^#/d' ${f}; done

```

This is how BaseX converts a single file from CSV into XML:

```xquery

let $f := "/home/neven/Documents/documents/skola/latstil/lektira/quint10/phi1002.phi001.perseus-lat2:10.0740"
let $csv := csv:parse(file:read-text($f), map { 'separator': 'tab' })
return $csv

```

Everything did not go smoothly. Because Quintilian uses a certain number of Greek words, and these were not transmitted well into the ISO-8859-1 encoding the UD files use, we had to locate and transliterate the Greek adverb τροπικῶς. The rest of Quintilian's Greek is garbled too (though it seems to be correctly interpreted morphologically!), but it did not choke BaseX.

This is the XQuery which finds all CSV files and returns XML of their data:

```xquery

declare function local:conllu($f){
  let $csv := csv:parse(file:read-text($f), map { 'separator': 'tab' })
return $csv
};

let $dir := "/home/neven/Documents/documents/skola/latstil/lektira/quint10/"
for $f in file:list($dir, xs:boolean(0), 'phi1002.phi001.perseus-lat2:10.*')
return local:conllu($dir || $f)

```

Note that, for elegance and readability, we have moved the conversion from CSV to XML to a local function `local:conllu($f)`.

Then we create "by hand" a `quint10` database and add to it each XML node produced by the script:

```xquery

declare function local:conllu($f){
  let $csv := csv:parse(file:read-text($f), map { 'separator': 'tab' })
return $csv
};

let $dir := "/home/neven/Documents/documents/skola/latstil/lektira/quint10/"
for $f in file:list($dir, xs:boolean(0), 'phi1002.phi001.perseus-lat2:10.*')
let $csv := local:conllu($dir || $f)
return db:add("quint10", $csv, $f)

```

The XQuery script above produces an XML database holding lemmata, morphological, and dependency annotations for the fragment of Quintilian we have selected -- Quintilian's Book 10. We have lost the `sent id` metadata from the original file (we lost them when we discarded the comments); they could have been preserved as part of the file name. There is also a problem with the Greek words, which seem to have not taken well the travel through the Universal Dependency annotation pipeline.

In any case, the database is now ready to be queried.

