---
layout: post
title:  "PokemonGoBot Installations Note New Version"
date:   2016-08-07
categories: Study-Notes
excerpt: git
comments: true
---

* content
{:toc}

## Steps 
Goto https://github.com/PokemonGoF/PokemonGo-Bot and download the lastest version. If you have the old version, just git reset --hard origin/master

In PokemonGo-Bot directory, run pip install -r requirements. choose wipe option to remove the "old" pgoapi.

All config files go to configs/. Note that only config.json works so far. If you have release_json, just copy the content to the 
new config.json: the old keyword "under" should be replaced by "below". 

go to http://pgoapi.com/ download the tar file, extract, then:
cd src  --> make -> mv libencrypt.so ..../PokemonGo-Bot/encrypt.so

Back to main folder, run python pokecli.py. Enjoy.

