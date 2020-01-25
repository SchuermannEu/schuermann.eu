---
layout: post
title: Removing Restrictions like Printing from PDFs on Linux
author: Dominik Schürmann
date: 2011-03-29
slug: pdf-restrictions
aliases:
    - "/2011/03/29/pdf-restrictions.html"
---

Some PDF files are restricted in such a way that some PDF viewers do not allow to print, edit or extract parts of the PDF content. PDF viewers like Okular or Evince used on Linux are ignoring these restrictions, but Windows viewers like the Adobe Reader are obeying these limitations. In my case I wanted to remove these restrictions fast using a command line tool.
A handy tool named [QPDF](https://github.com/qpdf/qpdf) comes to rescue to save the world from [evil DRM restrictions](http://www.defectivebydesign.org/).

1. Install QPDF:  
``sudo aptitude install qpdf``

2. Remove restrictions:  
``qpdf --decrypt input.pdf output.pdf``

3. To do this with many PDFs use the following one-liner:  
``for file in *.pdf; do qpdf --decrypt $file ${file/.pdf/_rescued.pdf}; done``

You can check the DRM restrictions of the input and output PDFs using the free demo version of [GuaPDF](http://www.guapdf.com/), which is also available as a linux command line tool, but restricted to PDFs with a small file size.

Another option is to use [http://freemypdf.com](http://freemypdf.com/) to remove these restrictions using a web site just for this purpose. The downside is the slow uploading of every PDF and possible privacy implications when uploading a PDF to such a website.
