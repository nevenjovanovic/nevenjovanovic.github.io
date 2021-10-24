---
layout:     post
title:      "Learning TEI Publisher"
subtitle:   "Impressions from learning to transform TEI XML with a very powerful tool, somewhat underdocumented at more advanced levels."
date:       2021-10-24 17:30:00
author:     "Neven Jovanović"
header-img: "img/pexels-spencer-davis-4388289.jpg"
---

To learn how to use the [TEI Publisher](https://teipublisher.com/), a system for displaying XML (and other) documents, I set myself three tasks:

1. Publish a TEI edition of a poem in several cantos, with annotations in attributes (enjambements and verse numbers) and with marginal notes
2. Color-code each enjambement type and display the verse in respective color
3. Display verse numbers on the left side of each verse

Briefly, about the poem. It is the first epic poem in Croatian, the Judita by Marko Marulić, written in 1501 and published in 1521; it has six cantos and 2126 verses. This year, Croatia celebrates the 500th anniversary of the first edition of Judita. In April 2021, I presented a paper on the annual [Marulić Days conference](https://www.knjizevni-krug.hr/novosti/novost-4/) in Split, the Colloquium Marulianum XXXI, comparing enjambements in Judita and in Marulić's Latin epic, the Davidias. The XML files provided the data for the paper; currently I am reviewing the enjambement annotations.

The TEI Publisher is built on [eXist](http://exist-db.org/), an XML database which uses XQuery as its query language. The TEI Publisher is an ambitious attempt to use the mechanisms provided by the TEI standard to transform XML into presentation formats for different media, from HTML to LaTeX and PDF. TEI Publisher is now in its version 7.1.0, and it is a very complete system that just works. For experimenting and learning, I used its [docker image](https://teipublisher.com/exist/apps/tei-publisher/doc/documentation.xml?root=3.7.7.11&action=search&view=div&odd=docbook.odd#3.7.7.11.3).

# 1. Publish a TEI edition

TEI Publisher via docker made the first task ridiculously easy. You choose the Playground collection (for testing), login as the demo user (the information is provided on the page), and upload the TEI XML file. That is all, and it is how it should be.

![Davidias and Judita in TEI Publisher]({{ site.url }}/img/juditateipublisher2021-10-24-18-11-57.png "Davidias and Judita in TEI Publisher")

# 2. Color-code the verses to show the enjambement type

The second task was more complicated. It required changing the ODD document, which controls transformations of TEI elements in the XML. Following the instructions, I eventually reached the satisfying solution – not without making a few wrong turns.

At first I wrote four models for four different l[@enjamb] element types, including the one which does not have an @enjamb attribute. In the models I changed the CSS rules for color. The procedure was relatively simple (once I started reading the documentation carefully), but the better solution – because I want to do four different things to l elements – turned out to be to use the modelGrp feature. It is a kind of nest into which I put my four l models. The solution is more elegant, but I had a feeling that it is somewhat laconically described in the documentation, and I had to find out the harder way that you cannot have an separate model (outside the modelGrp group) and the modelGrp at the same time; when I tried to save the ODD, an error was reported, not saying where the error was.

The eventual solution:

```xml

<modelGrp>
    <model predicate="@enjamb='2b'" behaviour="block">
        <desc>Enjambement 2b, Dark Pastel Blue #779ecb</desc>
        <param name="n" value="@n"/>
        <pb:template xmlns="" xml:space="preserve"><span n="[[n]]">[[content]]</span></pb:template>
        <outputRendition xml:space="preserve">
            color: #779ecb;
margin-left: 1em;
	    </outputRendition>
    </model>
    <model predicate="@enjamb='2a'" behaviour="block">
        <desc>Enjambement 2a, Laurel Green #a9ba9d</desc>
        <param name="n" value="@n"/>
        <pb:template xmlns="" xml:space="preserve"><span n="[[n]]">[[content]]</span></pb:template>
        <outputRendition xml:space="preserve">
                margin-left: 1em;
color: #a9ba9d;
        </outputRendition>
    </model>
            <model predicate="@enjamb='1'" behaviour="block">
                <param name="n" value="@n"/>
                <pb:template xmlns="" xml:space="preserve"><span n="[[n]]">[[content]]</span></pb:template>
                <outputRendition xml:space="preserve">
                margin-left: 1em;
color: #8f0000;
                </outputRendition>
            </model>
            <model behaviour="block">
                <param name="n" value="@n"/>
                <pb:template xmlns="" xml:space="preserve"><span n="[[n]]">[[content]]</span></pb:template>
                <outputRendition xml:space="preserve">
                margin-left: 1em;
                </outputRendition>
            </model>
        </modelGrp>

```

# 3. Display verse numbers

Task 3 turned out to be hard. It took me two or three days to solve. The problem is that I can transform the `@n` value (showing the verse number) into a separate element, but then the element is on the same level as the element holding the verse text. It seemed impossible to wrap the two sibling elements into a new, higher-level one (the parent in XPath).

TEI Publisher has a function for that, and it was clearly visible in its own field in the model form, but the language describing it was a little unspecific to me. The field said: "Define code template to apply to content". I guess the reason for avoiding specificity is that "code" means not only HTML, but LaTeX, Markdown, a lot of other things. Still, it did not help that in the documentation only a LaTeX example was given. If there were a HTML transformation example as well, it would not have taken me so long to achieve that.

The solution from the same ODD (you can see it at the end of the previous section as well):

```xml

<param name="n" value="@n"/>
<pb:template xmlns="" xml:space="preserve"><span n="[[n]]">[[content]]</span></pb:template>

```

![Judita, color coded and numbered verses in TEI Publisher, beginning of Canto 3]({{ site.url }}/img/juditalargercolorcode2021-10-24-18-20-00.png "Judita in TEI Publisher, Canto 3")

# Postproduction

Once I understood how to pass the `@n` attribute (and its value) to the HTML to be displayed, displaying attribute values was easily done on the CSS level for the whole document; TEI Publisher ODD has well-documented places for CSS instructions (one of them is in the TEI header of the ODD). I learned some more CSS in the process.

To display the `@n` values, I use these CSS instructions in the ODD TEI header (position: absolute enabled me to move the values in front of the verse line):

```xml

<tagsDecl>
  <rendition selector="span[n]::before">
  content: "[" attr(n) "]\2004";
  color: #bebebe;
  visibility: visible;
  position: absolute;
  left: -3em;
  </rendition>
</tagsDecl>
```

The marginal notes had the unwanted way of taking on the color of the verse line they described. I gave the notes their own font color with this CSS:

```xml

<rendition selector="span.margin-note">
  color: #004225;
  font-size: small;
</rendition>

```

# Conclusion

TEI XML is an immensely expressive standard. A whole universe of things that people do with texts (and, to some extent, images) can be annotated with its elements, attributes and rules. But what to do with this rich expression once we have produced it? One approach is to analyze it; an annotated XML is in itself a database of sorts, and we have XQuery to get information out of it. The other approach, however, is to show other people what we have done. If they are not XQuery or XML experts, we have to transform the XML in many ways, for many purposes.

TEI Publisher offers a quite mature way to achieve this transformation. A lot of groundwork has been done, and quite thoughtfully at that. It is very easy to start working with TEI Publisher and to start transforming XML documents with it. The beginner stage is also the best documented.

To go past that stage, however, is not easy. TEI Publisher has excellent mechanisms for achieving quite complex transformations of the TEI. That is in itself an engineering feat. But to use these mechanisms, thorough understanding of TEI, ODD and XQuery is not enough. We have to learn the TEI Publisher itself. What I thought to be a simple task – show a verse number alongside its verse – baffled me. The documentation at this, slightly advanced level seems too general, too few examples are provided, at least for the task I had. Trial and error eventually taught me that the task is indeed relatively simple. This is why I wrote this report, to provide some material for better advanced level documentation of this wonderful TEI XML tool.



(Main photo by Spencer Davis from Pexels.)
