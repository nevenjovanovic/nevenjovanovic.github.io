---
layout:     post
title:      "Lemmatize Neo-Latin with LemLat"
subtitle:   "Testing the LemLat (CIRCSE)"
date:       2017-07-19 23:00:00
author:     "Neven Jovanović"
header-img: "img/pexels-photo-248559.jpeg"
permalink: /lemmatize-neo-latin-lemlat/
---

# Lemmatization and its tools

We *lemmatize* a word (or try to) each time we have to look it up in a dictionary, where it is listed in its canonical form. In Latin, canonical forms are traditionally nominatives (for nominals), first persons singular (for verbs); words without inflected forms are easier to look up... unless we need to look up *cum* or *quod*, of course.

Lemmatization is also a basic natural language processing task, one of the first operations we want to do if we want computers to "know" anything about a text in a flective language such as Latin. At the moment, there are several Latin lemmatizers around: the late William Whitaker's [WORDS](http://mk270.github.io/whitakers-words/index.html) (currently [maintained by Martin Keegan](https://github.com/mk270/whitakers-words) on GitHub), Perseus's [Morpheus](https://github.com/PerseusDL/morpheus), the [LEMLAT by CIRCSE](https://github.com/CIRCSE/LEMLAT3), lemmatizers provided by the [CLTK](http://cltk.org/), [Collatinus](https://github.com/biblissima/collatinus), [PREPRO's Latin parser](https://prepro.hucompute.org/) (Goethe-Universität, Frankfurt/Main), and probably some others I have missed (Uli Bubenheimer has written a [thesis on YALL Java morphological analyzer for Latin](http://athenaeum.libs.uga.edu/handle/10724/20053), but I haven't been able to track down its actual implementation).

# LEMLAT 3.0

Marco Passarotti, leader of the Index Thomisticus Treebank Project, recently suggested I should test how their LEMLAT lemmatizer performs on our Neo-Latin texts. I thought this was a great idea, and here are my notes on the setup and on the very first results.

LEMLAT exists as a [web service](http://www.ilc.cnr.it/lemlat/lemlat/index.html), and it can be queried by a script issuing requests such as `http://www.ilc.cnr.it/lemlat/cgi-bin/LemLat_cgi.cgi?wordform=iuveni`. But it turned out to be too slow -- lemmatization of some 2100 entries from [Nicholas of Modruš's funerary speech for Pietro Riario](https://github.com/nevenjovanovic/modruski-temrezah) (XML DB `modr-riar-texts` in my computer) took over 20 minutes. I used this XQuery to try that out:

```xquery

(: query LemLat's webservice, retrieve just lemmata :)
declare function local:lemlat($form){
  let $base := "http://www.ilc.cnr.it/lemlat/cgi-bin/LemLat_cgi.cgi?wordform="
let $s := $base || $form
return element p {
  element form {
  $form },
  fetch:xml($s, map {
  'parser': 'html' } )//u
}
};

(: send each wordform to the lemmatizer only once :)
let $forms := distinct-values (
for $w in db:open("modr-riar-texts","nikolamodr01.croala1394919.croala-lat2w.xml")/*:TEI/*:text/*:body//*:w
let $f := $w/string()
(: transform capitalized words to lower case :)
return lower-case($f)
)
for $f in $forms
return local:lemlat($f)

```

# Standalone version

Luckily, there is also a [version of LEMLAT (3.0) on CIRCSE's Github](https://github.com/CIRCSE/LEMLAT3) whose binary was very easy to run on my Debian laptop as a command line standalone application. It is well documented, offers a batch mode, and can output plain text, CSV, or XML.

This version lemmatized 2159 bishop Nicholas's word forms in 20.9 seconds; the command in terminal was simply `./lemlat -i riario.txt -x riariolem.xml`. LEMLAT did not recognize 26 forms. Here they are:

```

modrusiensi
gratitudinis
saona
francisci
franciscus
tyrocinii
vicheriae
ticinium
memoriter
nudiustertius
seipsum
cardinalatus
nabuchodonosoris
nosse
floccipendebat
nescierit
symoniacae
quamobrem
norunt
imolam
imolae
bosti
quotannis
taruixii
maioli
viterbiensem

```

Some of the words are proper names or adjectives derived from them (Modrusiensis, Saona, Franciscus, Vicheria, Ticinium, Nabuchodonosor, Symoniacus, Imola, Bosti, Taruixium, Maiolus, Viterbiensis); others would perhaps have been lemmatized if they were written as two words (nudiustertius, seipsum, floccipendebat) or if they were spelled differently (tyrocinii). Cardinalatus is obviously a cultural word connected with Christian church as institution -- although, because LEMLAT was prepared for a large medieval corpus, its absence is somewhat surprising; does it signal a specifically Early Modern theme? 

# Unknown to LEMLAT, how about others?

There remain, however, seven Latin word forms which I would expect to have beeen lemmatized: *gratitudinis, memoriter, nescierit, nosse, norunt, quamobrem, quotannis*. All of them except for *gratitudo* are lemmatized correctly by the CLTK LemmaReplacer lemmatizer (invoked in a Python script with `from cltk.stem.lemma import LemmaReplacer`). [Whitaker's Words Online](http://latin.ucant.org/) lemmatizes them all (*gratitudo* as well!) and provides correct information on their part of speech features. [Collatinus (web)](http://outils.biblissima.fr/fr/collatinus-web/) cannot lemmatize *gratitudinis, memoriter*, and *norunt*. [PREPRO](https://prepro.hucompute.org/) lemmatizes correctly only *memoriter* and *quotannis*, provides wrong part of speech information for *quamobrem*, and fails on other forms (perhaps it expects to be given a sentence? Also, the large lexicon compiled from various sources, accessible at [collex.hucompute.org](http://collex.hucompute.org/), seems to be much better place to start lemmatizing -- if only I knew how to batch query it). Perseus's Morpheus, accessible as a web service through requests such as `http://morph.perseids.org/analysis/word?lang=lat&engine=morpheuslat&word=memoriter`, does not recognize *gratitudo*, similarly to the CLTK lemmatizer, but lemmatizes correctly everything else.

So, the top list of lemmatizers for words unknown to LEMLAT from the Nicholas of Modruš's speech for Riario:

1. Whitaker's WORDS
2. Morpheus and CLTK
3. Collatinus (web)
4. PREPRO

This was, of course, just a quick and dirty exploration. A real [product testing](https://en.wikipedia.org/wiki/Product_testing) comparison of lemmatizers would include feeding all of them exactly the same list of words and comparing the results carefully. Ease of preparatory operations such as installing and batch processing would also have to be assessed. But still, I have to note that, contrary to what is claimed in literature for other linguistic software ("In accordance with Moore’s law describing scientific/technological progress over time, we find that more recent [tools] substantially outperform their predecessor generation"), for Latin, two of the oldest lemmatizers -- Whitaker's and Morpheus -- still do the best job (alongside the CLTK). At least to the philologist's eye.
