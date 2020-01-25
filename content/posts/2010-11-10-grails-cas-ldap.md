---
layout: post
title: Using Grails with CAS and LDAP
author: Dominik Schürmann
date: 2010-11-10
slug: grails-cas-ldap
aliases:
    - "/2010/11/10/grails-cas-ldap.html"
---

This is a guide how to use Grails with CAS as the authentication server and LDAP as the backend user directory. It uses Spring Security Core and CAS Plugin in conjunction with the LDAP Plugin, that maps LDAP directories into Java classes. The ZIP-file contains the tutorial (PDF) and the sources. The config values in the attached example should be replaced with config values that work in your environment, especially the server-URLs. The Source-Code is released under the LGPL license.

WARNING: It’s written very quick and dirty and only provides a simple example for CAS+LDAP authentication. It is not intended for production use, cause it likely contains security holes.

[Grails Workshop (ZIP)](/files/Grails-Workshop.zip)
