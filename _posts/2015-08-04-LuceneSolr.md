---
layout: post
title:  "Lucene Solr"
date:   2015-08-04
categories: Study-Notes
excerpt: study notes
comments: true
---

![alt text](https://cloud.githubusercontent.com/assets/5607138/9074327/56a73278-3abd-11e5-9fb1-3773d7196a12.png)
![alt text](https://cloud.githubusercontent.com/assets/5607138/9074326/56a58928-3abd-11e5-88c4-4ccb04523ae5.png)

* content
{:toc}

** Notes

analyzer->tokenizer->filter

An analyzer examines the text of fields and generates a token stream. Analyzers are specified as a child of the <fieldType> element in the schema.xml configuration file (in the same conf/ directory as solrconfig.xml).
In normal usage, only fields of type solr.TextField will specify an analyzer. The simplest way to configure an analyzer is with a single <analyzer> element whose class attribute is a fully qualified Java class name. The named class must derive from org.apache.lucene.analysis.Analyzer.

Beider-Morse Phonetic Matching (BMPM) is a "soundalike" tool that lets you search using a new phonetic matching system. BMPM helps you search for personal names (or just surnames) in a Solr/Lucene index, and is far superior to the existing phonetic codecs, such as regular soundex, metaphone, caverphone, etc.

filter: solr.EnglishPorterFilterFactory: Porter stemming algo to merge words hugged hugging -> hug

Solr includes a simple command line tool for POSTing various types of content to a Solr server. The tool is bin/post. The bin/post tool is a Unix shell script; for Windows (non-Cygwin) usage, see the Windows section below.

bin/post -c gettingstarted example/films/films.json

